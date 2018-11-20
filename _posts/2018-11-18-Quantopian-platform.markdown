---
layout: post
title:  "Quantopian Long Trading Model"
date:   2018-11-18 14:58:58 -0500
categories: code, finance
excerpt_separator: <!--more-->
---

*includes: Chart, Code* 

**Summary**

 The model finds small cap equities that have dropped at least %8 during the previous trading day and holds them for 5 days.  The trading model beats the S&P 500 during some periods of time and can be used when the its relative performance has been poor.  It would not be effective if the strategy was implemented in all market environments.   

**Strategy**

1.  *Security Selection.* The trading model creates a pipeline of equities based on fundamental factors such as market capitilization.  Securities must have market capitalizations that are less than $1 billion and greater than $500 million.  The pipeline also screens for securities that are actually tradeable to investors.     

2.  *Portfolio Leverage.* The model uses up to %30 leverage.  If the maximum portfolio leverage is reached, the model does not make additional purchases.  

3.  *Fill Prices*.  Price slippage is accounted for on the Quantopian platform.  Because limit prices are used, if an order is not able to be filled by the end of the day, it is cancelled.  

4.  *Orders.*  If the security fell more than %8 in the previous trading day, a limit order is placed for that security at the previous day's closing price.  An allocation of %13 of the portfolio is made and and the position is completely closed after the 5 day period.   

**Performance**

During a one year period the trading model returned %17 verses %29 for the S&P 500.  Overall the model does not perform better than the benchmark on a consistent basis.   

**Model Implementation**

This model would be useful when its relative performance to the benchmark S&P 500 has been poor.  During these times, investors have an opportunity to beat the benchmark as the model "catches up" in relative terms.  Investors would not benefit from using the trading model consistently over time as it does not match benchmark returns and also has large drawdowns in portfolio values.     


Here's the beginning of the chart indicating performance:  



![performanc chart](/blog/assets/Quantopian.blog.shot.JPG)



{% highlight ruby %}

from quantopian.algorithm import attach_pipeline, pipeline_output
from quantopian.pipeline import Pipeline, CustomFactor
from quantopian.pipeline.data.builtin import USEquityPricing
from quantopian.pipeline.factors import SimpleMovingAverage, RSI, AverageDollarVolume
from quantopian.pipeline.data import Fundamentals
from quantopian.pipeline.filters import QTradableStocksUS
import talib
import numpy as np
import pandas as pd
from quantopian.pipeline import factors, filters, classifiers
from quantopian.pipeline.classifiers.fundamentals import Sector
import quantopian.algorithm as algo


def initialize(context):
    set_commission(commission.PerTrade(cost=1))
    context.maxlv = 1.3
    context.days_elapsed = {}
    context.can = []
    
    # Create and attach an empty Pipeline.
    pipe = Pipeline()
    pipe = attach_pipeline(pipe, name='my_pipeline')
    context.cannot = []
    context.rsis = {}
    context.buys = []
    
    # Available technical analysis factors ------------------------------------------ 
    # sma_10 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=10)
    # sma_30 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=30)
    # sma_1 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=1)
    # sma_50 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=50)
    # sma_200 = SimpleMovingAverage(inputs=[USEquityPricing.close], window_length=200)
    # dollar_volume = AverageDollarVolume(window_length=30)
    
    # Filter construction
    # previous greater than limit = 200000000000
    # sma_greater = sma_50 > sma_200
    # prices_under_5 = (sma_10 < 20)
    big_market_cap = Fundamentals.market_cap.latest < 10000000000
    tradable_stocks = QTradableStocksUS() 
    # can put this in - tech_sec =  Sector().element_of( [311])
    low_market_cap = Fundamentals.market_cap.latest > 500000000
    # volume = dollar_volume > 100000

    # fundamentals.asset_classification.morningstar_sector_code == 301)
    # Register outputs.
    # pipe.add(sma_10, 'sma_10')
    # pipe.add(sma_30, 'sma_30')
    # pipe.add(sma_1, 'price')

    pipe.set_screen(big_market_cap & tradable_stocks & low_market_cap)
    

    schedule_function(current_positions,date_rules.every_day(), time_rules.market_close())
    schedule_function(buy_sell_orders,date_rules.every_day(), time_rules.market_open(minutes=1))
    schedule_function(empty_can,date_rules.every_day(), time_rules.market_open(minutes=25))

    
def empty_can(context, data):
    context.can = []
    
   
def before_trading_start(context, data):

    results = pipeline_output('my_pipeline')

    # Store pipeline results for use by the rest of the algorithm.
    context.pipeline_results = results
    context.security_list = context.pipeline_results.index
    
    context.pos_for_day = 0


    if context.account.leverage < 1.1:    
        for security in context.security_list:
    
            hist = data.history(security, 'price', 14, '1d')

            price_2 = hist[-2]
            price_1 = hist[-1]
            
            
            
            price_dif = (price_1-price_2)/price_2

            if -.12 < price_dif < -.06 and 1 < price_1 < 6 and security not in context.cannot:
                context.can.append(security)
                # print(security, round(price_1,2), round(price_dif,2))   
        # print('Can:', context.can)  
   


    for stock in context.days_elapsed:
        context.days_elapsed[stock] += 1
    
    record(leverage = context.account.leverage)
    record(positions = len(context.portfolio.positions))
    
def buy_sell_orders(context, data):  
    
    # results = pipeline_output('my_pipeline')
    # # print results.price

    for security in context.can:
        hist = data.history(security, 'price', 2, '1d')

        price_1 = hist[-1]

        if (
            security not in context.portfolio.positions
            and context.account.leverage < context.maxlv
            # and len(context.can) < 50
            and context.pos_for_day < 15):

            hist = data.history(security, 'price', 20, '1d')
            # print(hist)
            rsi = round(talib.RSI(hist, timeperiod=14)[-1],0)
            
            if security not in context.rsis:
                context.rsis[security] = rsi
            # print('rsi_list', context.rsis)
            # print('sec:',security,'rsi',round(rsi,2))
            
            order_percent(security, .13, style=LimitOrder(price_1))
            
            context.days_elapsed[security] = 0
            context.pos_for_day += 1
                # print("Bot:", security, context.maxlv, round(context.account.leverage,3))
            

    if len(context.portfolio.positions) > 0:    
        for security in context.portfolio.positions:
            # print('sma', round(sma_10,2))
            if context.days_elapsed[security] > 5:
                # print(security, sma_10, 'sell - 10')
                # print(security, sma_30, 'sell-30')

                order_target(security, 0)
                # print("sold:", security)

def current_positions(context, data):
    if len(context.portfolio.positions) > 0: 
        
        # Avaialable RSI Factor (for price performance comparison)
        # for security in context.portfolio.positions:
            
            # prices = data.history(context.stocks, 'price', 20, '1d')
            # hist = data.history(security, 'price', 20, '1d')
            # print(hist)
        #     rsi = talib.RSI(hist, timeperiod=14)[-1]
        #     # print(rsi)
        #     context.rsis.append(rsi) 
        # avg_rsi = round(np.average(context.rsis),2)
        # record(RSI = avg_rsi)
    
        # print('average', avg_rsi)
        
        all_positions = "Current positions for %s : " % (str(get_datetime()))  
        for pos in context.portfolio.positions:  
            if context.portfolio.positions[pos].amount != 0:  
                all_positions += "%s -- %s shares, %s days, " % (pos.symbol,                                  
                                                                 context.portfolio.positions[pos].amount,
                                                                 context.days_elapsed[pos])  
        log.info(all_positions)

{% endhighlight %}

