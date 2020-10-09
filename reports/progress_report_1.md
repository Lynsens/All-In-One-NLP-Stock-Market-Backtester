# CSDS 395 - Report 1: An All-In-One NLP Stock Market Backtester

> Shaochen (Henry) ZHONG `sxz517`
> Jiaqi Yu `jxy618`
> Mocun Ye `mxy293`

---
*Due and submitted 10/09/2020*

---


## Background

The background of the project remains largely unchanged from out perviously submitted proposal. So here we will provided a concentrated version of it.

Using NLP related approaches to do some kinds of prediction on stock market is nothing new among traders who want to develop profitable trading strategies, researchers who want to testify their models' performances, and also to every developer who wants to have some hand-on ML/DL experience.

Despite the popularity, we noticed that it is rather hard to verify a NLP-stock-prediction model's performance since the researcher will have to gather the plain text data, gather the company data, gather the stock market data, and categorize them in a way that is communicable with each other and the model; then the researcher will need to build a virtual trading platform that keeps track of all the trading signals generated by the model, and visualize them for evaluation.

To implement all these steps from ground up, it is required for a researcher to have certain level of proficiency on skills which are, from a research stand-point, fairly deviated from the nature of the NLP model itself (like scraping a website and understanding the fundamental mechanism of trading in stock market). Even though there are some very mature tools being developed in the subareas of this task (especially on the stock market backtesting area), it still requires a reasonably large amount of effort to couple them together, and to store necessary information in a way that are not only communicable with each other, but also suits to the design of each and every tool a research chose to use. It is our understanding this kind of preliminary work will distract a researcher from the essence of his/her work -- developing an SOTA model, and will also create a unnecessary barriers for researcher who wants to quickly testify an idea in a controlled manner, or to who are in the need of reproducing a published work.


We like to build a set of tools that may automate such process to a certain degree. The ideal workflow we visioned is that developers may import their plain text and company data in a certain format (or even use the build-in API to obtain such data, of course, with limitation on available channels), then we will have a set of functions (or parsers) available to execute and register the trading signals generated by a desired model; our toolkit will also able research to restructure data in a way that is suitable for his/her model. In our pervious proposal we said if time’s available, we may even built in some classic NLP model just to provide a benchmark reference on “the same playing field,” or develop my own HMM model I am researching right now for demo. **By re-evaluating of our team members' course load per TA's feedback, we will likely drop the idea of actually providing a technically significant model, but just to place a "dummy model" to demonstrate our project's capability.**

In our pervious proposal, we also researched couple related works regarding this task (mostly focusing on the fields of backtesting, obtaining stock market data, and obtaining plain text data). We analyzed the pros-and-cons of several existing public available libraries and described why are (or why aren't) we developing own our tools, and what functionalities were expected. Please do review out proposal if you'd expect more information regarding related works.

---

## Progress Report

Per our proposal, by the time this report is written, we should be working on:

* *Week of 09/28/2020: Refactor scraper / Technical selection on backtester / Learning for visualizer / Learning stock market basic*

In an overview, we have followed our plan to a very exact manner. Although due to having to write this report, to prepare for upcoming midterms, and to handle some heavy deadlines on 10/5, we foresee a postponement our future plan for around an 1.5 after this week. We remain positive on delivery our project due to the redundancy we left, and we can also scale down the front-end part if necessary.

### Regarding Text Input

As mentioned in the background session, one of the main focuses of our project is to increase the obtainability of high-quality, ease-to-use plain text data with reach metadata provided; so that a researcher may concentrate on the model without having to deal with the minute details of data obtaining/processing/basic cleaning.

As a proposed option and now and per TA's feedback, we decided to limited our provided "exemplary" plain text data API to be [The Wall Street Journal](https://www.wsj.com) as one of our group member (Henry) has perviously worked on it.

What we have in hand now it a set of scripts which capable of scraping almost all (there are couple of exceptions, i.e. [this comic](https://www.wsj.com/articles/SB10001424052970204394804577010393609610160) has no plain text information and our script will simply skip it) WSJ articles during a certain timeframe, and register mentioned companies market information accordingly. As a visual aid of the former achievement, the scripts we have are able to extract the below information from WSJ:


The name of the files a bit "random" because all WSJ article urls are structured with a prefix of `https://www.wsj.com/articles/` and a suffix of `SB...`. Like for [Delta Petroleum's Stake Sale Eases Need for Cash - WSJ](https://www.wsj.com/articles/SB119909991774359193?): the link is `https://www.wsj.com/articles/SB119909991774359193?`), where `SB119909991774359193?` is the suffix. So we named our scraped articles with such suffix as it is a unique ID for all WSJ articles. We understood this discovery is unique to WSJ, and we may therefore implement another layer of ID system to make it cross-distributor compatible.

And another key feature where our project thrive is the ability to gather rich and necessary metadata along with the plain text information from an article. One of the essential need of NLP-based trading is to be able to link a plain-text mention of a company to such company's market data information. For example, in this above mentioned article [Delta Petroleum's Stake Sale Eases Need for Cash - WSJ](https://www.wsj.com/articles/SB119909991774359193?), two significant companies were mentioned, which are:

```
Delta Petroleum
Tesoro
```

But which ticker name, exchange center, and legal full name (sometime the mentioned company name is entirely different to its registered name on stock markets) does `Delta Petroleum` associate to? WSJ provided a hyperlink but it is always out-of-date (like this time it links to https://www.wsj.com/market-data/quotes/DPTR, which is a page that is no longer available). We did some heavy engineering (in short, we scraped google results and get back to the new WSJ market data page of such company and obtain such information from it, in this case it will be: https://www.wsj.com/market-data/quotes/DLTA/company-people) on this and able to obtain most companies market information accurately. The result will be something like:

```
[
    {
        "ing groep": {
            "market_data_url": "https://www.wsj.com/market-data/quotes/ing",
            "ticker": "ING",
            "exchange": "(U.S.: NYSE)",
            "legal_full_name": "ING Groep N.V. ADR",
            "cap_name": "ING Groep"
        },
        "baidu.com": {
            "market_data_url": "https://www.wsj.com/market-data/quotes/bidu",
            "ticker": "BIDU",
            "exchange": "(U.S.: Nasdaq)",
            "legal_full_name": "Baidu Inc. ADR",
            "cap_name": "Baidu.com"
        },
        "barclays": {
            "market_data_url": "https://www.wsj.com/market-data/quotes/bcs",
            "ticker": "BCS",
            "exchange": "(U.S.: NYSE)",
            "legal_full_name": "Barclays PLC ADR",
            "cap_name": "Barclays"
        },
...
```

(Note the above demo is up-to-change during the course of developing.)


One other key metadata is about the article itself: the title, the author, the exact publishing time, which session does it belong to (extremely critical when it comes to politics, e.g. a new report v. a column piece). We will able to register such metadata on the fly when scraping the plain-text information of the article.

Note a lot of these work were done perviously by one of our group member (Henry) under the help of another group member (Mocun). However most of the scripts were written in an one-off fashion, thus very "hacky" and lacks of usability. So we had and are putting heavy effort to refactor these script and hoping to have something like:

```
obj.get_plain_text_data (<start_time>, <end_time>, publisher = 'WSJ', credential dict, restriction_dict = None, metadata_dict = None, output_path = <default_dir>).
```

We except to finish the refactoring / adding new features to this segment for around one or two weeks.





### Regarding Trading




### Regarding Visualization


---

## Future Plan


---

## Updated Management Plan


---
## Contribution Acknowledgment

