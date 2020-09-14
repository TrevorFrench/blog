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
