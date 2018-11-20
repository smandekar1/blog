---
layout: post
title:  "Quantopian Trading Platform Analysis and Testing"
date:   2018-9-20 12:58:58 -0500
categories: [code, finance]
excerpt_separator: <!--more-->
---
[adapted from previous journal]

Quantopian is a backtesting platform for trading.  It is used by many to test high frequency trading strategies I have been working with it quite a bit lately looking for opportunties and seeing how they work.  I have been using a strategy that seems to get good returns during a bull market.  The strategy basically is looking for equities that have risen at least 8% during the last trading day and then hold those stocks for a period of time(ex. 6 days).  The strategy worked fairly well during certain years, but when tested during the 08'-09' financial crisis, the portfolio lost more than 85% of its value.  So that's not going to work.  

   