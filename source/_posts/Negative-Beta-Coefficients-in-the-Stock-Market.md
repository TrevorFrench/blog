---
title: Negative Beta Coefficients in the Stock Market
date: 2020-09-14 16:43:05
tags:
---

!!!THIS POST IS CURRENTLY UNDERGOING FORMATTING ITERATIONS!!!

This report is a statistical analysis of the validity of using stocks with negative beta coefficients as hedges against economic downturns. The scope of this report does not include financial instruments which use short positions to artificially create inverse relationships.

**Research Question**

Does an equity with a negative beta coefficient in a bull market truly make a worthwhile hedge in a bear market?

Conventional stock market wisdom suggests that stocks which have inverse relationships with overall market performance may have a place in an investor’s portfolio if said investor believes the overall market is going to experience a moment of decline (Streissguth, 2019). Mathematically, this theory makes sense; however, it is unclear if the theory holds water in real-life applications. A majority of stocks with negative beta coefficients may simply be underperforming or non-performing assets that continue to perform poorly in a downturn.

The hypotheses and null hypothesis are as follows:

   **H<sub>0</sub>:** Stocks with negative beta coefficients perform as expected during distressed market conditions (there is no statistically significant change in beta coefficients).

   **H<sub>1</sub>:** Stocks with negative beta coefficients perform better than expected relative to market indices during distressed market conditions (beta coefficients decrease).

   **H<sub>2</sub>:** Stocks with negative beta coefficients perform worse than expected relative to market indices during distressed market conditions (beta coefficients increase).

**Data Collection**

Data collection for this research paper began by finding samples to test the hypotheses. Ideally, one would be able to analyze complete data of historical prices for every equity that has ever traded publicly; however, this paper will be analyzing sample sets instead.

A representative sample began to be gathered by manually scraping mentions of stocks with negative betas from the web. To obtain a valid representation, examples were taken from different time periods, different industries, and different beta ranges. Various search terms were used when scraping the web to try and mitigate confirmation bias. There was a total of 55 observations recorded with 49 of them being distinct.

As some of the observations were American Depositary Receipts (ADRs), it also became necessary to retrieve historical currency exchange rates to isolate for interactions between the equity and the index rather than interactions between the U.S. Dollar and the native currency. This data was retrieved from ofx.com (“Historical Exchange Rates Tool & Forex History Data”, 2020).

The next step was to retrieve historical price data for each of these observations so that beta coefficients could be calculated. To do this efficiently, a node.js script was developed which used the Alpha Vantage API. The code is as follows:

```javascript
const https = require('https');
const csv = require('csv-parser');
const fs = require('fs');
const rp = require('request-promise');
let URL = "https://www.alphavantage.co/query?function=TIME_SERIES_MONTHLY_ADJUSTED&symbol=";
let URLEnd = "{API_KEY}";
let output = [];
let tickers = [
{ Ticker: 'ARNA' },{ Ticker: 'FGP' },{ Ticker: 'AEM' },{ Ticker: 'DPS' },{ Ticker: 'TRMD' },{ Ticker: 'ENEVY' },
{ Ticker: 'ZM' },{ Ticker: 'GRSXY' },{ Ticker: 'GEGYY' },{ Ticker: 'COE' },{ Ticker: 'CCOEY' },{ Ticker: 'FLRAF' },{ Ticker: 'DRCSY' },
{ Ticker: 'VIRT' },{ Ticker: 'SAFE' },{ Ticker: 'FRHC' },{ Ticker: 'IHICY' },{ Ticker: 'CUYTY' },{ Ticker: 'PGENY' },{ Ticker: 'DHT' },
{ Ticker: 'PYPTF' },{ Ticker: 'AHCO' },{ Ticker: 'VIAAY' },{ Ticker: 'SLP' },{ Ticker: 'PLMR' },{ Ticker: 'AWR' },{ Ticker: 'CTWS' },
{ Ticker: 'TCO' },{ Ticker: 'CHCJY' },{ Ticker: 'OLCLY' },{ Ticker: 'KGDEY' },{ Ticker: 'HRL' },{ Ticker: 'DNLMY' },{ Ticker: 'WDFC' },
{ Ticker: 'PSO' },{ Ticker: 'DCMYY' },{ Ticker: 'VGZ' },{ Ticker: 'SO' },{ Ticker: 'SO' },{ Ticker: 'SO' },{ Ticker: 'SO' },
{ Ticker: 'SO' },{ Ticker: 'SO' },{ Ticker: 'SO' },{ Ticker: 'ASPS' },{ Ticker: 'GILD' },{ Ticker: 'GLPG' },{ Ticker: 'RHHBY' },
{ Ticker: 'HAS' },{ Ticker: 'ROST' },{ Ticker: 'WMT' },{ Ticker: 'AMGN' },{ Ticker: 'BUD' },{ Ticker: 'HRB' },{ Ticker: 'DLTR' }
];

function timer(ms) {
 return new Promise(res => setTimeout(res, ms));
}

	function make_api_call(Ticker){
	    return rp({
	        url : URL + Ticker + URLEnd,
	        method : 'GET',
	        json : true
	    })
	}

	async function processUsers(){
	    let result;
			for(let i = 0; i < tickers.length; i++){
	        result = await make_api_call(tickers[i].Ticker);
          outputTwo = result["Monthly Adjusted Time Series"];
          var keys = Object.keys(outputTwo);
          let csvText = "";
          let n = 0;
          for (x in outputTwo) {
            csvText += "<tr><td>" + tickers[i].Ticker + "</td><td>" + keys[n] + "</td><td>" + outputTwo[x]['5. adjusted close'] + "</td></tr>";
            n++;
          }
          output[i] = csvText;
          var index = i + 1;
          var time = 20 * (tickers.length - i + 1) / 60;

					console.log("Retrieving " + index + "/" + tickers.length);
          console.log("Estimated Time Remaining: " + time + " minutes");
					await timer(20000);
	    }
	    return output;
	}

	async function doTask(){
	    let result = await processUsers();
	    console.log(result);
      let html = "<table>" + result + "</table>";
			fs.writeFile('newfile.html', html, function (err) {
			  if (err) throw err;
			  console.log('Saved!');
			});
	}

doTask();
```
Finally, the data for the S&P 500 (NYSE: SPY) was retrieved using the node.js script.

**Data Extraction and Preparation**

The first step in the data-preparation process was to put all historical price for each ticker into a single, readable table where the data can be compared easily. This step exposed fourteen observations which did not contain enough data to be used in the analysis (DPS, ENEVY, GRSXY, GEGYY, DRCSY, PYPTF, CTWS, CHCJY, KGDEY, TRMD, ZM, FRHC, AHCO, PLMR). This was a relatively easy process, so it was performed manually in Excel.

The resulting table begins as follows and continues through 08/31/2000:

{% asset_img img1.png %}

The advantage of preparing this data in Excel is that it isn’t necessary to spend time creating a script to prepare the data; however, if the amount of data were larger, it may make sense to spend the time writing a script rather than spending time manually manipulating the data. Setting the data up in a table like this will allow for each column to be read in as a variable in R for regression to be performed. Additionally, the regression can be performed directly in Excel. This format also sets the data up nicely for Excel-based regression to be executed.

The data for the independent variable (S&P 500) was then appended to the table:

{% asset_img img2.png %}

For quick reference, ADRs were then color-coded by country. Each country was then given a corresponding Excel tab with a matching color which contains the historical exchange rates for the country’s native currency. Setting the data up this way allows for easy access to desired information through tools like Excel’s “VLOOKUP()” function.

These steps are illustrated by the following images:

{% asset_img img3.png %}

{% asset_img img4.png %}

{% asset_img img5.png %}

**Analysis**

While linear regression is how the beta statistic is commonly calculated within the industry, it can also be calculated simply by dividing covariance by variance (Nickolas, 2019). However, this analysis will be using complete linear regression models to dive more deeply into the strength of the relationships and evaluate more fully the null hypothesis that the difference between beta coefficients is actually zero. Additionally, linear regression is easier to use when evaluating models with multiple independent variables. Out all of the tools I’ve used throughout this program, I’ve found that R is the easiest to use when performing regression methods on larger data sets

This analysis will be focusing on two “inflection points” from which the market turned from a positive trend to a negative trend. Specifically, these two inflection points are October 31st, 2007 and December 31st, 2019. The next step was to calculate the Beta Coefficient (3Y monthly) from each of those points for every one of the dependent variables. This was done using linear regression in R with the following code:

```R
library(readxl)
output <- read_excel("C:/Users/Frenc/OneDrive/Desktop/output.xlsx")
#DETERMINE BETA (3Y MONTHLY) FROM INFLECTION POINT
inflectionOne <- output[8:44, ]
#DECLARE VARIABLES
spy <- inflectionOne$SPY
arna <- inflectionOne$ARNA
fgp <- inflectionOne$FGP
aem <- inflectionOne$AEM
coe <- inflectionOne$COE
ccoey <- inflectionOne$CCOEY
flraf <- inflectionOne$FLRAF
virt <- inflectionOne$VIRT
safe <- inflectionOne$SAFE
ihicy <- inflectionOne$IHICY
cuyty <- inflectionOne$CUYTY
pgeny <- inflectionOne$PGENY
dht <- inflectionOne$DHT
viaay <- inflectionOne$VIAAY
slp <- inflectionOne$SLP
awr <- inflectionOne$AWR
tco <- inflectionOne$TCO
olcly <- inflectionOne$OLCLY
hrl <- inflectionOne$HRL
dnlmy <- inflectionOne$DNLMY
wdfc <- inflectionOne$WDFC
pso <- inflectionOne$PSO
dcmyy <- inflectionOne$DCMYY
vgz <- inflectionOne$VGZ
so <- inflectionOne$SO
asps <- inflectionOne$ASPS
gild <- inflectionOne$GILD
glpg <- inflectionOne$GLPG
rhhby <- inflectionOne$RHHBY
has <- inflectionOne$HAS
rost <- inflectionOne$ROST
wmt <- inflectionOne$WMT
amgn <- inflectionOne$AMGN
bud <- inflectionOne$BUD
hrb <- inflectionOne$HRB
dltr <- inflectionOne$DLTR
#PERFORM REGRESSIONS
arna1 <-lm(arna ~ spy)
fgp1 <- lm(fgp ~ spy)
aem1 <- lm(aem ~ spy)
coe1 <- lm(coe ~ spy)
ccoey1 <- lm(ccoey ~ spy)
flraf1 <- lm(flraf ~ spy)
virt1 <- lm(virt ~ spy)
safe1 <- lm(safe ~ spy)
ihicy1 <- lm(ihicy ~ spy)
cuyty1 <- lm(cuyty ~ spy)
pgeny1 <- lm(pgeny ~ spy)
dht1 <- lm(dht ~ spy)
viaay1 <- lm(viaay ~ spy)
slp1 <- lm(slp ~ spy)
awr1 <- lm(awr ~ spy)
tco1 <- lm(tco ~ spy)
olcly1 <- lm(olcly ~ spy)
hrl1 <- lm(hrl ~ spy)
dnlmy1 <- lm(dnlmy ~ spy)
wdfc1 <- lm(wdfc ~ spy)
pso1 <- lm(pso ~ spy)
dcmyy1 <- lm(dcmyy ~ spy)
vgz1 <- lm(vgz ~ spy)
so1 <- lm(so ~ spy)
asps1 <- lm(asps ~ spy)
gild1 <- lm(gild ~ spy)
glpg1 <- lm(glpg ~ spy)
rhhby1 <- lm(rhhby ~ spy)
has1 <- lm(has ~ spy)
rost1 <- lm(rost ~ spy)
wmt1 <- lm(wmt ~ spy)
amgn1 <- lm(amgn ~ spy)
bud1 <- lm(bud ~ spy)
hrb1 <- lm(hrb ~ spy)
dltr1 <- lm(dltr ~ spy)
#CREATE LIST OF BETA COEFFICIENTS
x <- list(as.numeric(arna1$coef[2]),
          as.numeric(fgp1$coef[2]),
          as.numeric(aem1$coef[2]),
          as.numeric(coe1$coef[2]),
          as.numeric(ccoey1$coef[2]),
          as.numeric(flraf1$coef[2]),
          as.numeric(virt1$coef[2]),
          as.numeric(safe1$coef[2]),
          as.numeric(ihicy1$coef[2]),
          as.numeric(cuyty1$coef[2]),
          as.numeric(pgeny1$coef[2]),
          as.numeric(dht1$coef[2]),
          as.numeric(viaay1$coef[2]),
          as.numeric(slp1$coef[2]),
          as.numeric(awr1$coef[2]),
          as.numeric(tco1$coef[2]),
          as.numeric(olcly1$coef[2]),
          as.numeric(hrl1$coef[2]),
          as.numeric(dnlmy1$coef[2]),
          as.numeric(wdfc1$coef[2]),
          as.numeric(pso1$coef[2]),
          as.numeric(dcmyy1$coef[2]),
          as.numeric(vgz1$coef[2]),
          as.numeric(so1$coef[2]),
          as.numeric(asps1$coef[2]),
          as.numeric(gild1$coef[2]),
          as.numeric(glpg1$coef[2]),
          as.numeric(rhhby1$coef[2]),
          as.numeric(has1$coef[2]),
          as.numeric(rost1$coef[2]),
          as.numeric(wmt1$coef[2]),
          as.numeric(amgn1$coef[2]),
          as.numeric(bud1$coef[2]),
          as.numeric(hrb1$coef[2]),
          as.numeric(dltr1$coef[2])
          )
#PRINT OUT LIST OF BETA COEFFICIENTS
print(x)
```
