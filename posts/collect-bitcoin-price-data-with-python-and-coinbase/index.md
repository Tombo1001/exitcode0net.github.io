# Collect bitcoin price data with python and Coinbase

> Collect live Bitcoin prices from the Coinbase API with Python 3, store the readings in InfluxDB and chart the trend in Grafana.

Source: https://exitcode0.net/posts/collect-bitcoin-price-data-with-python-and-coinbase/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-02-02
Updated: 2026-10-08
Tags: python, coinbase, influxdb, grafana, crypto



Cryptocurrencies have been back on the popular agenda lately, thanks mostly to the surging value of Bitcoin. I have been interested, but not massively invested in cryptocurrency for many years. If I had the courage to buy and hold all those years ago, I might be writing to you from a beach – I assure you, this is not the case! I have been [getting into InfluxDB lately](https://exitcode0.net/posts/writing-to-an-influxdb-server-with-python3/) and its fantastic ability to store heaps of time series data. Well, using python and Coinbase, I am going to show you how to collect bitcoin (and other crypto assets) price data and store it in InfluxDB… then we are going to make some neat panels in Grafana to help us realise our data. This is going to be a longer write up than most, so here is here are links to each section:

Contents Table:
---------------

1. [Prerequisites](#Prerequisites "#Prerequisites")
2. [Using the Coinbase API with python](#Using-the-Coinbase-API-with-python "#Using-the-Coinbase-API-with-python")
3. [Writing to InfluxDB with python](#Writing-to-InfluxDB-with-python "#Writing-to-InfluxDB-with-python")
4. [Creating Coinbase data panels in Grafana](#Creating-Coinbase-data-panels-in-Grafana "#Creating-Coinbase-data-panels-in-Grafana")
5. [Ways to support this blog](#Ways-to-support-this-blog "#Ways-to-support-this-blog")

Prerequisites
-------------

Before we can jump into programming, there are a number of boxes we need to check off. 

1. You must have a Coinbase account. If not, consider using my referral link to get $10 free when you sign up and buy/sell $100 of crypto: https://www.coinbase.com/join/cockin\_u
2. Your programming environment has python3 and pip3 installed – pip3 will be used to install the Coinbase module: `pip install coinbase`
3. You have InfluxDB and Grafana setup. There are a number of great tutorials out there on how to do this, here are some places to get started:
	1. Unraid – https://unraid.net/blog/ultimate-unraid-dashboard
	2. InfluxDB getting started guide, including docker setup – [https://docs.influxdata.com/influxdb/v2.0/get-started/](https://docs.influxdata.com/influxdb/v2.0/get-started/ "https://docs.influxdata.com/influxdb/v2.0/get-started/")
	3. Grafana setup guide – [https://grafana.com/docs/grafana/latest/installation/](https://grafana.com/docs/grafana/latest/installation/ "https://grafana.com/docs/grafana/latest/installation/")

Using the Coinbase API with python
----------------------------------

Whist you can use the Coinbase API without an account there are rate-limiting restrictions, so if you want more representative, ‘live’ data, you will need to create a Coinbase account and apply for an API key:

*Where to find API options in your Coinbase account*
For the purposes of what we are doing in this exercise, there is no need to give the API key anything more than read level access to our account:

*Coinbase API read permissions*
### Python and Coinbase

You can access the full Coinbase API reference here, but we will be focusing on getting price data: [https://developers.coinbase.com/api/v2#prices](https://developers.coinbase.com/api/v2#prices "https://developers.coinbase.com/api/v2#prices")

#### Basic Python Request – BTC Buy Price USD

```

from coinbase.wallet.client import Client
client: Client(<api_key>, <api_secret>)

price: client.get_buy_price(currency_pair: 'BTC-USD')

```

#### Basic Response – BTC Buy Price USD

```
{
  "data": {
    "amount": "1020.25",
    "currency": "USD"
  }
}
```

So now that we have the basics from the Coinbase API reference, let us flesh this out into something more reliable and error-proof. Now is also a great time to make this into a function and call it continuously to gather time-series data.

#### Improved Python Request

The main function, giving us a continuous loop and skipping sending data when a Coinbase API request fails (checking for BTC and ETH):

```
import requests
import sys
import os
import json
from time import sleep
from coinbase.wallet.client import Client

def main():
    timer: 0
    timer_stop: 120
    portfolio_total: 0
    coins: ["BTC", "ETH"]
    while True:
        for coin in coins:
            reqdvalue: func_req_price(coin, 'USD', 'sell')
            if reqdvalue != False:
                func_logPrice(coin, reqdvalue, 'USD-sell')

            reqdvalue: func_req_price(coin, 'USD', 'buy')
            if reqdvalue != False:
                func_logPrice(coin, reqdvalue, 'USD-buy')

        sleep(5)
```

A function to get the price data for a provided coin and kind (please note that we DON’T actually need an API key for this particular method):

```
def func_req_price(coin, currency, kind):
    try:
        price: float(requests.get(
                f'https://api.coinbase.com/v2/prices/{coin}-{currency}/{kind}'
                ).json()['data']['amount'])
        return price
    except:
        print("Error getting price for: {}".format(coin))
        return False
```

This returns a price value or false if the request failed for any reason.  
We now have three values – coin, kind (USD buy/sell) and price) – which we can throw into InfluxDB…

---

**IMPORTANT NOTE:** If you are requesting data more frequently from Coinbase or you are pulling data for far more than just BTC and ETH, you **will** run into rate limits and receive error 429 responses for your requests. This is way Coinbase suggest using the API supported method of getting prices:

```
from coinbase.wallet.client import Client
client: Client(<api_key>, <api_secret>)

price: client.get_buy_price(currency_pair: 'BTC-USD')
```

---

3 – Writing to InfluxDB with python
-----------------------------------

I have covered most of the groundwork for this section already in this post: [https://exitcode0.net/writing-to-an-influxdb-server-with-python3/](https://exitcode0.net/posts/writing-to-an-influxdb-server-with-python3/).

#### Generic InfluxDB example

```
from datetime import datetime
from influxdb import InfluxDBClient

def func_log(unit, kind, value):
	client: InfluxDBClient(host='127.0.0.1', port=8086)
	client.create_database('DBname')
	client.switch_database('DBname')
	json_body: [
		{
			"measurement": "MeasureName",
			"tags": {
				"unit": unit,
				"kind": kind

			},
			"fields": {
				"value": value
			},
			"time": f'{datetime.utcnow().isoformat()}Z'
		}
	]

	try:
		if client.write_points(json_body):
			success: True
		else:
			print("Error writing to InfluxDB")
	except:
		print("Error writing to InfluxDB")

#Call the above function:
func_log(unit, kind, value)
```

#### Coinbase specific InfluxDB example

```
def func_logPrice(coin, price, kind):
    client: InfluxDBClient(host='127.0.0.1', port=8086)
    client.switch_database('PyDCA')

    json_body: [
        {
            "measurement": "stability",
            "tags": {
                "coin": f'{coin}', # e.g. BTC
                "kind": f'{kind}' #e.g. USD-sell
            },
            "fields": {
                "price": price
            },
            "time": f'{datetime.utcnow().isoformat()}Z'
        }
    ]

    try:
        if client.write_points(json_body):
            #do nothing
            success: True
        else:
            print("ERROR writing to InfluxDB")
    except:
        print("ERROR writing to InfluxDB")
```

Combining all this code, we can pass any value that we retrieve from the Coinbase API and store it in InfluxDB with a time stamp. This will be very useful when we start creating panels in Grafana!

4 – Creating Coinbase data panels in Grafana
--------------------------------------------

In my last segment of code, you can see that I specify the PyDCA data group when writing to influxDX, remember to set this accordingly in your code and Grafana panels.

Here is a basic query to display BTC and ETH, USD sell prices as a time series graph; I have set ETH on the right-hand axis as the two values differ so greatly, bring the graphs into a tighter range:

*A basic Grafana time series query*
*Showing the USD sell price provided by the Coinbase API*
*API data flat spot*
As you can see from the above figure, there are some gaps in our data, this can be caused by a lack of response from the Coinbase API request, an API limit, or a failure to write to the InfluxDB. In my experience, this is a frequent occurrence for BTC and ETC prices in USD; most probably because it is the most popular API endpoint. I do not see correlating flat spots in the data for GBP. In Grafana we can make smooth the graph out by setting the `GROUP BY: fill(none)` tag. Grafana draws a neat line between data points where there is NULL data.

5 – Ways to support this blog
-----------------------------

As previously mentioned, if you are new to Coinbase, you can sign up with my referral link. If you do so, you will receive $10 and so will I (`You'll both get free Bitcoin when a friend buys or sells $100 of crypto` – Coinbase).  
[https://www.coinbase.com/join/cockin\_u](https://www.coinbase.com/join/cockin_u "https://www.coinbase.com/join/cockin_u")

Alternatively, you can fund my coffee misadventures by throwing a coin in the tip jar:

[](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")
**FREE BONUS:** Alternatively, you can share this post or leave a comment below to let the search algorithms know I’m not totally useless.

*See you on the moon HODLers!*



---
Markdown version of https://exitcode0.net/posts/collect-bitcoin-price-data-with-python-and-coinbase/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
