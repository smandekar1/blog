---
layout: post
title:  "Postgres ETL in Python - A taste of database management"
date:   2018-10-11 14:58:58 -0500
categories: code
excerpt_separator: <!--more-->
---
After looking at an opportunity to work on ETL, I decided to go through a tutorial and work out a program that would do some extraction in Postgres.  The code here looks pretty good but there were some changes that needed to be made from Python 2 to 3. The setdefaultecoing method is no longer used in Python 3.  Someone said that this was an abuse in Python 2. 

What it does:  It takes 2 of the 3 tables from one server and puts them on another server.  It also checks if the tables already exists, and if it does then that table gets dropped.  

{% highlight ruby %}
#Simple Extract & Load Python Program


import petl as etl, psycopg2 as pg, sys
from sqlalchemy import *
from importlib import reload

reload(sys)
# sys.setdefaultencoding('utf8') was seen as an abuse Python 2

dbCnxns = {'operations':"dbname=operations user=postgres password=admin host=127.0.0.1",
'python':"dbname=python user=postgres password=admin host=127.0.0.1"}

# set connections and cursors
sourceConn = pg.connect(dbCnxns['operations']) #grab value by referencing key dictionary
targetConn = pg.connect(dbCnxns['python']) #grab value by referencing key dictionary
sourceCursor = sourceConn.cursor()
targetCursor = targetConn.cursor()

# # retrieve the names of the source tables to be copied
sourceCursor.execute("""select table_name from information_schema.columns where table_name in ('orders','returns') group by 1""")
sourceTables = sourceCursor.fetchall()

# # iterate through table names to copy over
for t in sourceTables:
    targetCursor.execute("drop table if exists %s" % (t[0]))
    sourceDs = etl.fromdb(sourceConn, 'select * from %s' % (t[0]))
    etl.todb(sourceDs, targetConn, t[0], create=True, sample=10000)

{% endhighlight %}

{% include share-bar.html %}
