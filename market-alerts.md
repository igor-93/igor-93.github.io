---
layout: default
---

# Market Event Notification Tool

I've recently built the Market Event Notification Tool because I wanted to track few tickers without the need to check them up manually on daily or intra-daily basis.

So I bought a Raspberry Pi and wrote a bit of code that notifies my via email when something interesting happens.

It turned out to be pretty useful, so I kept working on it a bit... by mostly adding some more complex types of alerts.

## How to (un-)subscribe

### Quick start

To subscribe to an "all-time high or low" alert for Tesla stock, send the following email to [my service account](mailto:igor.service.acc@gmail.com):

Subject: `add alert`

Body:
```
ticker = "ADA-USD"
indicator = "all_time_hilo"
silence_for = "10h"
```

The response that the you would have received if you had this alert on the 2021-05-15 would be:

`Alerts: ADA-USD: all-time HIGH`

```
Triggered alerts:

ADA-USD reached all-time HIGH at 2.18
For more details on how to (un-)subscribe, please check the docs here.
```


To subscribe to multiple emails at once, adjust the email content as follows:
```
[[alerts]]
ticker = "UBER"
indicator = "all_time_hilo"
silence_for = "10h"

[[alerts]]
ticker = "TSLA"
indicator = "pct_last_close"
silence_for = "10h"
	[alerts.params]
	threshold = 0.05
```

The email body must be a parsable [TOML](https://toml.io) text.

### How to unsubscribe

Simply change the subject of the email from `add alert` to `remove alert` and keep the body the same as the one you sent when subscribing to that alert. 

Or if you subscribed to multiple alerts, but want to unsubscribe from only one of them, then include only the part that refers to the alert you want to unsubscribe from.

## Documentation

### Email format

Subject:
- it is case insensitive
- it must contain the word `alert` in it
- it must contain the word `add` in it if you want to subsribe to a new alert
- it must contain the word `update` or `change` in it if you want to update the `silence_for` parameter of an existing alert. If you want to update parameters, then please delete and then add a new alert. Remember, for that you need 2 separate emails!
- it must contain one of: `delete`, `stop`, `remove`, `unsubscribe` words to unsubscribe from an existing alert. In that case it is importand that ticker, indicator and its parameters (if any) correspond to the alert you added earlier.

Body:
- must be a valid [TOML](https://toml.io) format
- can either specify only 1 alert, in which case you just need to pass the folloowing keys:
	- `ticker`: ticker (aka symobl) of a stock or currency pair or crypto-currency pair you want to track. See "Data sources" section below for more details on supported tickers.
	- `indicator`: type of alert you want to apply (see below documentation for more)
	- `silence_for`: time duration for which it is guaranteed that you will not recieve an email for that alert. The format can be for example "10h", "1 day", "1:00:00", etc... For more details see [pandas docs](https://pandas.pydata.org/pandas-docs/stable/user_guide/timedeltas.html)
	- optional: `params`: set of parameters and its values that are necessary or optional for that indicator
- can specify multiple alerts to be added or removed. Then above every alert defintions, the `[[alerts]]` must be written. The alert defintion is then the same as described above. 

### Data sources

Currently the only data source that is supported is [Yahoo](https://finance.yahoo.com/). 

It means, that all data arrives with 15 minutes of delay.

However, it has a very good data coverage and data quality.

To find what ticker you can pass to the alert, go to [Yahoo](https://finance.yahoo.com/) and just search for what you need.

### Indicator types

As you probably realized, the tool supports various types of indicators that can notify you of various events that are happening on the market.
Althought the number of different indicators is constantly increasing, here is the list of what is currently available:

#### All-time high or low
- Indicator name: `all_time_hilo`
- Parameters: none

Notifies you if the ticker has reached all-time-high price or all-time-low price.

Note: currently, it only takes into account the available data that I have, not the whole history from yahoo. So if you look for some major index, it will be correct, but otherwise it might only take recent data for comparison

#### Pct change since last trading session
- Indicator name: `pct_last_close`
- Parameters:
	- `threshold`: (absolute) percentage change since last session close must be higher then this. E.g. `0.01` is 1%
	- `sign`: optional. If `le` (stands for less or equal than) then the alert triggers only if price drops by at least `-threshold`, if `ge` (greater or equal than) then the alert triggers only if price increases by at least `+threshold`. Otherwise the alert triggers in both directions.

Notifies if there are major changes in price compared to last trading session.

#### Absolute threshold
- Indicator name: `abs_threshold`
- Parameters:
	- `threshold`: alert triggers when threshold price is reached. E.g. 120.

Notifies you if the price of a given ticker has crossed the specifed price.

#### Bollinger Band
- Indicator name: `bollinger_band`
- Parameters:
	- `band_width`: describes how many standard deviations should the bands be away from the mean. Example values: 1,2,3.... Default is `2`.
	- `std_history`: time-range used to estimate the standard deviation. Example value `7d` means that last 7 days will be used to get the standard deviation of prices. Default is `7d`.

This indicator triggers when price crosses lower or upper bound of [Bollinger Band](https://en.wikipedia.org/wiki/Bollinger_Bands). Example of the Bollinger Bands on S&P 500 index:
![Bollinger Band Example](/images/bollinger_band.png)

#### Crossing Moving Averages (MA)
- Indicator name: `crossing_moving_average`
- Parameters:
	- `short_window`: time-range used to calculate more recent moving average. E.g. `7d`
	- `long_window`: time-range used to calculate more long-term moving average. E.g. `14d`
	- `threshold`: value by which long- and shor-window MAs must differ in order to trigger and alert.

This indicator triggers when short-window MA crosses the long-window MA and has a difference of at least threshold. Example plot of 7 and 14 days moving averages of S&P 500 index: 
![Crossing Moving Average](/images/crossing_moving_average.png)
