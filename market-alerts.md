---
layout: default
---

# Market Alerts

I've recently built the Market Alerts tool because I wanted to track few tickers without the need to check them up regualry.

So I bought a Raspberry Pi and wrote a bit of code that notifies my via email when something interesting happens.

It turned out to be pretty useful, so I kept working on it a bit... by mostly adding some more complex types of alerts.

## How to (un-)subscribe

### Quick start

To subscribe to an "all-time high or low" alert for Tesla stock, send the following email to [my service account](mailto:igor.service.acc@gmail.com):

Subject: `add alert`

Body:
```
ticker = "TSLA"
indicator = "all_time_hilo"
silence_for = "10h"
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

TODO: Here describe alerts that are implemented and for each of them explain the parameters