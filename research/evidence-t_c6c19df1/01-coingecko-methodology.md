# Methodology

CoinGecko provides a wide variety of data on many different cryptoassets. All data that we receive/query from various sources are put through CoinGecko’s various algorithms that verifies the integrity of the data. Read on for a summary of how we calculate and evaluate the various metrics available on CoinGecko.

Trust Score

1.
    Trading Pairs





For each trading pair on CoinGecko, the Trust Score is calculated by considering the:





1. Web traffic stats of an exchange (by SimilarWeb)
2. Orderbook spread & ±2% depth
3. Overall trading volume
4. Trade frequency
5. Outlier checks

Trust Score is displayed in Green/Yellow/Red or None on under the “Trust Score” column. A set of screenshots taken below illustrates the difference between different Trust Score colors. You may also navigate directly to coin pages (eg. [Bitcoin](https://www.coingecko.com/en/coins/bitcoin), [Litecoin](https://www.coingecko.com/en/coins/litecoin)) to see it in action!

![](https://static.coingecko.com/s/methodology/trust_score_1-083ff6020fb04193c01c025e1dc45935d1c9433c00955e0c8867d3417f6c2857.png)![](https://static.coingecko.com/s/methodology/trust_score_2-2b1e86ee66ec4a91322ea3baa7cdcba46ef9337eb7979c32203f8aaea068e0c5.png)![](https://static.coingecko.com/s/methodology/trust_score_3-15988f19d128d4916a6fd18a3eccf0955dbb8a91c8b7534c123dbe29f6949bdd.png)![](https://static.coingecko.com/s/methodology/trust_score_4-debcc0b99ca4af0f77e60fa6bbed9c1eb0f09e339f5d3a6ff56e81a2bae09d9e.png)


Note that Trust Score calculated on each trading pair is not final and changes in real-time depending on market conditions. The detailed methodology of Trust Score is illustrated on CoinGecko’s Blog Post [here](https://blog.coingecko.com/trust-score/).

2.
    Crypto Exchanges (Spot)





Trust Score for spot crypto exchanges is displayed on a scale of 1 - 10 on the [exchange overview](https://www.coingecko.com/en/exchanges) page.



![](https://static.coingecko.com/s/methodology/trust_score_exchanges_1-77e0b4ef356d9cda349732f5652fefec8f1bf05e03cebdf7ef60f782aefa78e5.png)



Trust Score for Exchanges is evaluated based on a combination of the following:





   - Liquidity
   - Regulation
   - Cybersecurity
   - Past Incidents
   - Proof of Reserves

You can find the breakdown of an exchange’s Trust Score on the respective pages (eg. [Coinbase](https://www.coingecko.com/en/exchanges/coinbase-exchange)).

![](https://static.coingecko.com/s/methodology/trust_score_exchanges_2-81d8e3f48a4e179755d93ffb1cd865110476e8d87deee5a909c1fc34220df07d.png)


Each evaluation category is calculated based on several other indicators as shown by the breakdown above.

The detailed methodology can be found on CoinGecko’s blog in a [summarized format here](https://blog.coingecko.com/trust-score-explained), or as separate releases:

   - Trust Score 1 ( [May 2019](https://blog.coingecko.com/trust-score)) \- ranks trading pairs based on web traffic, liquidity & trading activity related metrics.
   - Trust Score 2 ( [Sep 2019](https://blog.coingecko.com/trust-score-2)) \- ranks exchanges based on web traffic, liquidity, scale & API coverage.
   - Trust Score Cybersecurity update ( [Jul 2020](https://blog.coingecko.com/trust-score-cybersecurity-update)) \- improves on Trust Score 2 through inclusion of Cybersecurity evaluation.
   - Trust Score Team Presence & Incidents update ( [Nov 2020](https://blog.coingecko.com/trust-score-team-presence-incidents-update)) \- inclusion of evaluation of operational factors such as Team Presence & Past Incidents.
   - Trust Score 3 Proof of Reserves ( [Jan 2023](https://blog.coingecko.com/trust-score-3-proof-of-reserves-asset-liabilities)) \- incorporation of Reserves data into Trust Score to be used as part of the algorithm for ranking purposes.
   - Trust Score Basilisk Update ( [May 2026](https://blog.coingecko.com/trust-score-basilisk-update)) \- rebuilds Trust Score by replacing web traffic with direct volume and order book depth evaluation, introducing Regulation Score, and grading exchanges on a curve relative to peers.

3.
    Crypto Exchanges (Derivatives)



    Trust Score is currently unavailable for [Derivative Exchanges](https://www.coingecko.com/en/exchanges/derivatives). Derivative exchanges are ranked based on their respective Open Interest and Trading Volume figures as reported.


Market Data

1.
    Market Price of Coins



    CoinGecko collects all available price, volume, and liquidity data across all tickers on all exchanges integrated to CoinGecko. By aggregating data across multiple tickers through a comprehensive algorithm, we calculate the market price of each coin. CoinGecko’s overall price aggregation methodology is divided into two distinct parts - a. Calculating the Bitcoin Price Index (BPI), and b. Aggregating the market price for all coins. You can find the complete methodology, as well as example calculations, in this [document](https://assets.coingecko.com/methodology/CoinGecko-Price-Aggregation-Methodology.pdf). Due to the importance of this Price Aggregation Methodology to CoinGecko, any changes or modifications to the methodology is governed by a multi-departmental Governance Forum. No changes can be made to the methodology without prior discussion and approval at the forum.







1. Calculating the Bitcoin Price Index (BPI)



       All ticker data on CoinGecko are stored in BTC base. To convert all ticker data to BTC, we utilize the BPI as the internal reference Bitcoin conversion rate to all fiat currencies.




      1. BPI Calculation

          We begin with calculating the BPIUSD, the reference BTC-USD conversion rate. This is calculated as the volume-weighted average price (VWAP) of selected BTC/USD, BTC/USDT, BTC/USDC, or BTC/EUR tickers across a selection of reputable exchanges. Prior to performing VWAP, USDT and USDC-paired BPI tickers are first converted to USD using a USD\_ONRAMP\_INDEX, while EUR BPI tickers are converted to USD using Open Exchange Rates. Once we obtain the BPIUSD, we apply the corresponding /USD exchange rate from [Open Exchange Rates](https://openexchangerates.org/) to obtain the BPI for all other fiat currencies supported on CoinGecko.

      2. Ticker Data Processing

          Once the BPI is calculated, we can process the tickers for all coins listed on CoinGecko. Tickers with BTC base are stored as-is in our database. Fiat currency tickers of all other coins are converted to a BTC base by applying the BPI. For crypto-to-crypto tickers, we apply an internal pathfinding algorithm to arrive at the BTC price. Once the price and volume of all tickers are denominated in BTC, they can be converted back to USD or any other fiat currency.
2. Aggregating the market price for all coins




      1. Initial Ticker Set Construction

          In addition to displaying data from individual tickers, CoinGecko also displays one aggregated market price for coins. For the purpose of price aggregation, an initial ticker set is constructed based on the top 600 tickers by volume of a particular coin. We then filter out tickers if it is detected to be anomalous or do not meet selected criteria.

      2. Price Outlier Detection and Exclusion

          One of the most important anomalous ticker filters we utilize is price outlier detection, where tickers that display an outlier price within the ticker set are automatically excluded. There are currently two different outlier detection algorithms depending on the number of available tickers in the ticker set. For coins with less than three tickers, any ticker price change that is greater than 100x from the previous price will be classified as an outlier. For coins with three tickers or more, CoinGecko applies an outlier detection algorithm by calculating the lower and upper bounds based on the [median absolute deviation (MAD)](https://eurekastatistics.com/using-the-median-absolute-deviation-to-find-outliers). If the price falls outside the bounds, we consider those tickers as outliers.

      3. Final Price Aggregation

          Once outliers have been removed, we calculate the VWAP of all remaining tickers in the ticker set to arrive at the final aggregated price. In certain cases, our operations team may intervene to exclude outliers if the team believes a certain ticker price to be anomalous but was not excluded by our outlier detection algorithm.

2.
    Trading Volume (Exchange)



    The trading volume of an exchange is the sum of the volume of all Trading Pairs available on a particular exchange. For example, assume exchange A has 2 different coins (ETH and LTC) and both have USD and BTC pairings:



    Taking ETH = USD $200 and LTC = USD $100 from CoinGecko’s global volume-weighted average price and with the following trading volume (rolling 24 hours) of each pair:



    ETH/BTC = 400 ETH

    ETH/USD = 100 ETH

    LTC/BTC = 3,000 LTC

    LTC/USD = 2,000 LTC



    Exchange A’s trading volume is then:

    400 ETH + 100 ETH + 3,000 LTC + 2,000 LTC = USD 600,000



    Note that CoinGecko’s calculation algorithm for an exchange volume excludes trading pairs that have been blacklisted for inconsistent data and have not been updated for over 3 hours. This is to ensure that the aggregated volume will reflect the market conditions as accurately as possible.

3.
    Trading Volume (Global)





The Global 24-hour Volume (top of the website, below the menu bar) is the sum of the trading volume of all exchanges as tracked by CoinGecko.



![](https://static.coingecko.com/s/methodology/trading_volume_1-a7eae140b3f197a93fd2cd8bd54c7dcd5e360e67bae5196ee1adb2be5331591a.png)

4.
    Circulating Supply





The Circulating Supply information displayed on CoinGecko is obtained from the various token teams and is verified by the CoinGecko team. If CoinGecko cannot verify the accuracy of such information, the Circulating Supply will be marked with a “-” symbol.



The Circulating Supply information is displayed on CoinGecko and the source of this calculation is shown by clicking on the  symbol besides the Circulating Supply text. The Circulating Supply information is used to calculate the Market Capitalization of cryptoassets.



For Proof-of-Work coins, an API endpoint from the coin’s block explorer is used by CoinGecko to query the amount of coins available. CoinGecko includes pre-mine coins in the Circulating Supply calculation as these coins can be sold by whoever that has control of these coins.




![](https://static.coingecko.com/s/methodology/circulating_supply_1-5a473a1174558035fafeb5421b176cddf391a74e61fe626b9acc00c06390c492.png)



For tokens issued on smart contract platforms (eg. ERC-20 tokens on [Ethereum](https://www.coingecko.com/en/coins/ethereum)), we calculate the Circulating Supply by deducting locked tokens from Total Supply. Locked tokens may include Foundation’s Fund, investors’ locked tokens, team’s locked tokens etc. The addresses of these locked tokens are obtained from the token teams.



![](https://static.coingecko.com/s/methodology/circulating_supply_2-2465ba6e45b2e43c34fd2718345ffc4738e593990959c53b37418325a75435f4.png)



The token balance of locked addresses is obtained automatically from the block explorer whenever an API is available. For 0x’s case, as it is an ERC-20 token, the token balances of the 0x addresses are obtained automatically from Etherscan and deducted from the total 0x supply.

5.
    Market Capitalization (Cryptoasset)



    The Market Capitalization of a cryptoasset is calculated using the following formula:



    Let:

    A = Current cryptoasset price in USD

    B = Circulating supply of an asset



    Market Capitalization = A \* B



    For example, the market capitalization of 0x is calculated by multiplying the Circulating Supply of ZRX with its price. Assuming the Circulating Supply of ZRX is 583,209,787 and the price of ZRX is USD 0.25, the market capitalization is then calculated as 583,209,787 \* USD $0.25 = USD 145,802,446.75.

6.
    Market Capitalization (Global)





The Global Market Capitalization (top of the website, below the menu bar) is the sum of the Market Capitalization of all projects tracked by CoinGecko.



![](https://static.coingecko.com/s/methodology/market_capitalization_1-4e7cb998249769f3df85e6d5b5fdbaad91fb019fc4bcd9d2915db94dfcf417d9.png)

7.
    All-Time High (ATH)



    CoinGecko calculates the ATH using following data:





1. ATH Price
2. ATH Date
3. Current Price
4. Current Date

% Drop since ATH= (ATH Price – Current Price) / (ATH Price) \* 100%

Date since ATH = Current Date – ATH Date

Project Listings

Getting your token or exchange listed on CoinGecko is free and no CoinGecko representative will ever ask you for any form of listing fees.

While we would gladly accept any donations (addresses are available at the CoinGecko website footer), please note that it does not guarantee a listing or expedite any part of the process. Listed below are some of the criteria that need to be fulfilled for a listing consideration on CoinGecko. However, do note that fulfilling these criteria does not automatically guarantee a listing as CoinGecko also evaluates many other factors prior to listing any project on our site.

CoinGecko reserves the rights to publish or unpublish any listed cryptoasset, ICO, or exchange on our site without prior notice if we feel that any of the presented information is inaccurate in any way. CoinGecko will not provide an update if a cryptoasset/ICO/exchange fails any part of the review process.

Listing Criteria:

1. Cryptoassets
   - Working, functional website that has sufficient information on cryptoasset that is being listed. Websites with no information on purpose, team or social media profiles will be considered as invalid.
   - Website must be owned by the project/maintaining team. Websites hosted on website builders (i.e Wix) will not be accepted.
   - Working block explorer
   - Listed on at least one (1) [active exchanges where CoinGecko is integrated with](https://www.coingecko.com/en/exchanges).

   - Projects traded only on self-serviceable centralized/decentralized exchanges may be rejected due to security concerns.
   - For market capitalization calculation, circulating supply must be communicated clearly (e.g. any company/foundation/team/developers/vesting/locked tokens).
2. Exchanges
   - A working exchange website with actual trading volume that matches information in API.
   - Fulfills [CoinGecko’s Crypto Exchange API Standards](https://docs.google.com/document/d/1v27QFoQq1SKT3Priq3aqPgB70Xd_PnDzbOCiuoCyixw).
   - A working REST API documentation.
   - Have a representative with whom the team can easily communicate for any issues.
   - Coin Info page with identification of each traded coin on the exchange
      Basic info required: Name, Ticker, Logo, Coin Website, Block Explorer.

Do’s and Don’ts for Listing Submission:


There are certain things that you can help us with to ensure that the listing process is as smooth as possible.

Do:

- Fill in as much accurate information in the [CoinGecko Request Form](https://www.coingecko.com/request-form?locale=en).
- Ensure that there are no dead links/bad references.
- Submit a ticket at support.coingecko.com if you have further enquiries.

Don’t:

- Submit multiple requests.
- Repeatedly ask for status updates.
- Offer financial gain to anyone for a listing.
- Accept any offer from others who claim that they can expedite or guarantee your listing.


Remember, CoinGecko representatives will never ask you for any form of listing fees.

Listing Process Flow:

Cryptoassets

1. Fill out [CoinGecko Request Form](https://www.coingecko.com/request-form?locale=en). (Refer to the guide [here](https://support.coingecko.com/hc/en-us/articles/7291312302617-How-to-submit-a-Coin-Token-Listing-Request-via-CoinGecko-Request-Form-))
2. CoinGecko reviews requests.
3. If it passes review, CoinGecko lists the cryptoasset.
4. Submitters can check the listing status [here](https://www.coingecko.com/request-form?locale=en).

Exchanges

1. Submit the [CoinGecko Request Form](https://www.coingecko.com/request-form?locale=en).
2. CoinGecko reviews requests.
3. If the exchange fulfills all criteria, a CoinGecko representative will be in touch via email to proceed with exchange listing agreement (applicable on CEX listing).
4. A CoinGecko developer will work on the exchange integration.
5. CoinGecko team will check the code again and if it passes all checks, the exchange API will be merged to CoinGecko’s internal Exchange library. Exchanges are required to follow [CoinGecko’s Crypto Exchange API Standards](https://docs.google.com/document/d/1v27QFoQq1SKT3Priq3aqPgB70Xd_PnDzbOCiuoCyixw) to facilitate the exchange listing.
6. CoinGecko team will check every single trading pair and match it against an existing cryptoasset listed on CoinGecko.
7. Exchange information will be populated (images, description, fees etc).
8. Exchange integration is completed and the exchange is listed on CoinGecko.

Select Currency

Suggested Currencies

USD

US Dollar

IDR

Indonesian Rupiah

TWD

New Taiwan Dollar

EUR

Euro

KRW

South Korean Won

JPY

Japanese Yen

RUB

Russian Ruble

CNY

Chinese Yuan

Fiat Currencies

AED

United Arab Emirates Dirham

ARS

Argentine Peso

AUD

Australian Dollar

BDT

Bangladeshi Taka

BHD

Bahraini Dinar

BMD

Bermudian Dollar

BRL

Brazil Real

CAD

Canadian Dollar

CHF

Swiss Franc

CLP

Chilean Peso

CZK

Czech Koruna

DKK

Danish Krone

GBP

British Pound Sterling

GEL

Georgian Lari

HKD

Hong Kong Dollar

HUF

Hungarian Forint

ILS

Israeli New Shekel

INR

Indian Rupee

KWD

Kuwaiti Dinar

LKR

Sri Lankan Rupee

MMK

Burmese Kyat

MXN

Mexican Peso

MYR

Malaysian Ringgit

NGN

Nigerian Naira

NOK

Norwegian Krone

NZD

New Zealand Dollar

PHP

Philippine Peso

PKR

Pakistani Rupee

PLN

Polish Zloty

SAR

Saudi Riyal

SEK

Swedish Krona

SGD

Singapore Dollar

THB

Thai Baht

TRY

Turkish Lira

UAH

Ukrainian hryvnia

VND

Vietnamese đồng

ZAR

South African Rand

XDR

IMF Special Drawing Rights

Cryptocurrencies

BTC

Bitcoin

ETH

Ether

LTC

Litecoin

BCH

Bitcoin Cash

BNB

Binance Coin

EOS

EOS

XRP

XRP

XLM

Lumens

LINK

Chainlink

DOT

Polkadot

YFI

Yearn.finance

SOL

Solana

Bitcoin Units

BITS

Bits

SATS

Satoshi

Commodities

XAG

Silver - Troy Ounce

XAU

Gold - Troy Ounce

Select Language

Popular Languages

EN

English

RU

Русский

DE

Deutsch

PL

język polski

ES

Español

VI

Tiếng việt

FR

Français

PT-BR

Português

All Languages

AR

العربية

BG

български

CS

čeština

DA

dansk

EL

Ελληνικά

FI

suomen kieli

HE

עִבְרִית

HI

हिंदी

HR

hrvatski

HU

Magyar nyelv

ID

Bahasa Indonesia

IT

Italiano

JA

日本語

KO

한국어

LT

lietuvių kalba

NL

Nederlands

NO

norsk

RO

Limba română

SK

slovenský jazyk

SL

slovenski jezik

SV

Svenska

TH

ภาษาไทย

TR

Türkçe

UK

украї́нська мо́ва

ZH

简体中文

ZH-TW

繁體中文

Welcome to CoinGecko

Welcome back!

Login or Sign up in seconds

.
Not you?

![](https://static.coingecko.com/s/google-167c1e093ccfc014420e14da91325a1f70c91e592f58164fefe84603d2fde02e.svg)

Continue with Google


![](https://static.coingecko.com/s/apple_black-6d9a519c69649e1a56774f4036ec552e412e73d5f9c2b0bab193732e47721533.svg)![](https://static.coingecko.com/s/apple_white-df0a614505190a8b2bc87fd16396160fed4680f62a69d5005cd2ae95562b2d2a.svg)

Continue with Apple


or


Continue with email


Continue with email

Check your email

We can't verify you right now

Please try again later.

Sign in with other methods


By continuing, you acknowledge that you've read and agree fully to our [Terms of Service](https://www.coingecko.com/en/terms) and [Privacy Policy](https://www.coingecko.com/en/privacy).

[**Get Price Alerts with CoinGecko App**![Apple App Store Button](https://static.coingecko.com/s/coingecko_logos/apple_app_store-558245a688cc13737dfb861fd82b252d75d5afbaf343c06e3067a454675bbe05.svg)\\
![Gecko Illustration](https://static.coingecko.com/s/mobile_login_banner_illustration-8033f10041126a2567d726da00e01841722bbf2ed9e334e5fe7dc795669094ef.svg)](https://coingecko.app.link/footer-ios) [**Get Price Alerts with CoinGecko App**![Google Play Store Button](https://static.coingecko.com/s/coingecko_logos/google_play_store-cb1f298b04afa7f74639a948d9b2e22e4aa6eea9486a2b0442c2cf9bdcda63e8.svg)\\
![Gecko Illustration](https://static.coingecko.com/s/mobile_login_banner_illustration-8033f10041126a2567d726da00e01841722bbf2ed9e334e5fe7dc795669094ef.svg)](https://coingecko.app.link/footer-android)

Get Price Alerts with CoinGecko App

[![Apple App Store Button](https://static.coingecko.com/s/coingecko_logos/apple_app_store-558245a688cc13737dfb861fd82b252d75d5afbaf343c06e3067a454675bbe05.svg)](https://coingecko.app.link/footer-ios)[![Google Play Store Button](https://static.coingecko.com/s/coingecko_logos/google_play_store-cb1f298b04afa7f74639a948d9b2e22e4aa6eea9486a2b0442c2cf9bdcda63e8.svg)](https://coingecko.app.link/footer-android)

![](https://static.coingecko.com/s/gecko_peek_right_qr_illustration-c062eafc49ff2d60d5b9fddbc2e42c6e795a9e50e2feaf221e0d3487a10d088c.svg)

Resend confirmation instructions

You will receive an email with instructions for how to confirm your email address in a few minutes.


Resend confirmation instructions

Get the CoinGecko app.

Scan this QR code to download the app now
![App QR Code](https://static.coingecko.com/s/landing_pages/mobile_app_qr_code-503a3393ccc355a8c41ea1a4026c31db469665b66c47847b347e2447c80feea3.svg)
Or check it out in the app stores


[![Google Play Store Button](https://static.coingecko.com/s/coingecko_logos/google_play_store-cb1f298b04afa7f74639a948d9b2e22e4aa6eea9486a2b0442c2cf9bdcda63e8.svg)](https://coingecko.app.link/footer-android)[![Apple App Store Button](https://static.coingecko.com/s/coingecko_logos/apple_app_store-558245a688cc13737dfb861fd82b252d75d5afbaf343c06e3067a454675bbe05.svg)](https://coingecko.app.link/footer-ios)

Add assets

**0** assets selected


Add Assets

Add NFT

![CoinGecko](https://static.coingecko.com/s/coingecko-logo-without-text-f20c41f84915a38a8daf9a09132e9c2eec56a926a02a61ace708d268bb5a793c.svg)

Better on the app

Real-time price alerts and a faster, smoother experience.


Not Now

[Download App](https://coingecko.app.link/Nhgleb44aVb)

![](https://static.coingecko.com/s/gekko_throne-565383f97a64423b31f84161099d180ea665c9352ad890ee8dc5c1853f07dfd8.png)![](https://static.coingecko.com/s/gekko_reach_coin-1c7e6668964fa7ca02467940e7cf0762dbf9c6e5e334c1111cdd3d297b19b8eb.png)

Maybe Later

![](https://static.coingecko.com/s/gekko_reach_coin-1c7e6668964fa7ca02467940e7cf0762dbf9c6e5e334c1111cdd3d297b19b8eb.png)

Remove Coins

![](https://cloudcdn-img.com/static/752c02be/spacer.gif)