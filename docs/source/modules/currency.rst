Currency - Overview
===================

The currency module's purpose is to provide Exchange Rates for a given date using a base currency and another currency.

The API endpoint is the easiest and fastest way to access the exchange rate data. 

Choose your base currency and the endpoint will simply return the conversion rates from your base currency code to all 
the other supported in an easy to parse JSON format.

 **Business Rules**

  * Currencies need to use 3 alpha character codes as per `ISO 4217`_ on requests. We use ISO 4217 Three Letter Currency Codes - e.g. USD for US Dollars, EUR for Euro etc. Here are the codes we support.
  * Exchange Rates imported once a day (using external 3rd party service `ExchangeRate-Api`_) i.e. GET https://v6.exchangerate-api.com/v6/YOUR-API-KEY/latest/USD
  * If an exchange rate isn't available for a date, the closest previous date should be used.
  * If a currency is not supported on a request, an exception should be thrown.
  * Comparing Exchange Rates, use standard compare method on **effective dates** first, **from** currency next and **to** currency last.

Features
--------

 * gRPC services
 * MediatR
 * Fluent Validation

Specifics
---------

Supported Currencies
^^^^^^^^^^^^^^^^^^^^

Support Exchange Rate API requests for the currency codes listed below. Please note that `ISO 4217`_ Three Letter Currency Codes can be deprecated 
and replaced over time. An example would be Mexico - which used MXP prior to the 1993 currency revaluation but now uses MXN. Some online lists of 
currency codes still use old codes - so check here if you're uncertain about which codes to include in your currency conversion project.

You can read more about `ISO 4217`_ codes on `Wikipedia`_.

+-------------+------------------------------------+----------------------+
| Code        | Name                               | Country              |
+-------------+------------------------------------+----------------------+
| AED         | UAE Dirham                         | United Arab Emirates |
+-------------+------------------------------------+----------------------+
| ARS         | Argentine Peso                     | Argentina            |
+-------------+------------------------------------+----------------------+
| AUD         | Australian Dollar                  | Australia            |
+-------------+------------------------------------+----------------------+
| BGN         | Bulgarian Lev                      | Bulgaria             |
+-------------+------------------------------------+----------------------+
| BRL         | Brazilian Real                     | Brazil               |
+-------------+------------------------------------+----------------------+
| BSD         | Bahamian Dollar                    | Bahamas              |
+-------------+------------------------------------+----------------------+
| CAD         | Canadian Dollar                    | Canada               |
+-------------+------------------------------------+----------------------+
| CHF         | Swiss Franc                        | Switzerland          |
+-------------+------------------------------------+----------------------+
| CLP         | Chilean Peso                       | Chile                |
+-------------+------------------------------------+----------------------+
| CNY         | Chinese Renminbi                   | China                |
+-------------+------------------------------------+----------------------+
| COP         | Colombian Peso                     | Colombia             |
+-------------+------------------------------------+----------------------+
| CZK         | Czech Koruna                       | Czech Republic       |
+-------------+------------------------------------+----------------------+
| DKK         | Danish Krone                       | Denmark              |
+-------------+------------------------------------+----------------------+
| DOP         | Dominican Peso                     | Dominican Republic   |
+-------------+------------------------------------+----------------------+
| EGP         | Egyptian Pound                     | Egypt                |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Germany              |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Austria              |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Belgium              |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Cyprus               |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Estonia              |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Finland              |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | France               |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Greece               |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Ireland              |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Italy                |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Latvia               |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Lithuania            |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Luxembourg           |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Malta                |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Netherlands          |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Portugal             |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Slovakia             |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Slovenia             |
+-------------+------------------------------------+----------------------+
| EUR         | Euro                               | Spain                |
+-------------+------------------------------------+----------------------+
| FJD         | Fiji Dollar                        | Fiji                 |
+-------------+------------------------------------+----------------------+
| GBP         | Pound Sterling                     | United Kingdom       |
+-------------+------------------------------------+----------------------+
| GTQ         | Guatemalan Quetzal                 | Guatemala            |
+-------------+------------------------------------+----------------------+
| HKD         | Hong Kong Dollar                   | Hong Kong            |
+-------------+------------------------------------+----------------------+
| HRK         | Croatian Kuna                      | Croatia              |
+-------------+------------------------------------+----------------------+
| HUF         | Hungarian Forint                   | Hungary              |
+-------------+------------------------------------+----------------------+
| IDR         | Indonesian Rupiah                  | Indonesia            |
+-------------+------------------------------------+----------------------+
| ILS         | Israeli New Shekel                 | Israel               |
+-------------+------------------------------------+----------------------+
| INR         | Indian Rupee                       | India                |
+-------------+------------------------------------+----------------------+
| ISK         | Icelandic Krona                    | Iceland              |
+-------------+------------------------------------+----------------------+
| JPY         | Japanese Yen                       | Japan                |
+-------------+------------------------------------+----------------------+
| KRW         | South Korean Won                   | South Korea          |
+-------------+------------------------------------+----------------------+
| KZT         | Kazakhstani Tenge                  | Kazakhstan           |
+-------------+------------------------------------+----------------------+
| MVR         | Maldivian Rufiyaa                  | Maldives             |
+-------------+------------------------------------+----------------------+
| MXN         | Mexican Peso                       | Mexico               |
+-------------+------------------------------------+----------------------+
| MYR         | Malaysian Ringgit                  | Malaysia             |
+-------------+------------------------------------+----------------------+
| NOK         | Norwegian Krone                    | Norway               |
+-------------+------------------------------------+----------------------+
| NZD         | New Zealand Dollar                 | New Zealand          |
+-------------+------------------------------------+----------------------+
| PAB         | Panamanian Balboa                  | Panama               |
+-------------+------------------------------------+----------------------+
| PEN         | Peruvian Sol                       | Peru                 |
+-------------+------------------------------------+----------------------+
| PHP         | Philippine Peso                    | Philippines          |
+-------------+------------------------------------+----------------------+
| PKR         | Pakistani Rupee                    | Pakistan             |
+-------------+------------------------------------+----------------------+
| PLN         | Polish Zloty                       | Poland               |
+-------------+------------------------------------+----------------------+
| PYG         | Paraguayan Guarani                 | Paraguay             |
+-------------+------------------------------------+----------------------+
| RON         | Romanian Leu                       | Romania              |
+-------------+------------------------------------+----------------------+
| RUB         | Russian Ruble                      | Russia               |
+-------------+------------------------------------+----------------------+
| SAR         | Saudi Riyal                        | Saudi Arabia         |
+-------------+------------------------------------+----------------------+
| SEK         | Swedish Krona                      | Sweden               |
+-------------+------------------------------------+----------------------+
| SGD         | Singapore Dollar                   | Singapore            |
+-------------+------------------------------------+----------------------+
| THB         | Thai Baht                          | Thailand             |
+-------------+------------------------------------+----------------------+
| TRY         | Turkish Lira                       | Turkey               |
+-------------+------------------------------------+----------------------+
| TWD         | New Taiwan Dollar                  | Taiwan               |
+-------------+------------------------------------+----------------------+
| UAH         | Ukrainian Hryvnia                  | Ukraine              |
+-------------+------------------------------------+----------------------+
| USD         | United States Dollar               | United States        |
+-------------+------------------------------------+----------------------+
| UYU         | Uruguayan Peso                     | Uruguay              |
+-------------+------------------------------------+----------------------+
| ZAR         | South African Rand                 | South Africa         |
+-------------+------------------------------------+----------------------+


gRPC service
^^^^^^^^^^^^

The ExchangeRateService 

.. _`ISO 4217`: https://www.iso.org/iso-4217-currency-codes.html
.. _`ExchangeRate-Api`: https://app.exchangerate-api.com/dashboard
.. _`Wikipedia`: https://en.wikipedia.org/wiki/ISO_4217