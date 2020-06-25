
# Case Study: ABC International

## Table of contents
- [Case Study: ABC International](#case-study-abc-international)
  - [Table of contents](#table-of-contents)
- [Case Study: ABC International <a name="case-study"></a>](#case-study-abc-international-1)
  - [Supplier Financial Data Analysis <a name="supplier-financial-data"></a>](#supplier-financial-data-analysis)
      - [Liquidity](#liquidity)
      - [Solvency](#solvency)
      - [Profitability](#profitability)
  - [Financial Data ETL<a name="background-of-organization"></a>](#financial-data-etl)
    - [Financial Data API<a name="financial-data-api"></a>](#financial-data-api)
    - [Web Crawler <a name="web-crawler"></a>](#web-crawler)
    - [Suppliers Profiling and Scoring<a name="profiling-and-scoring"></a>](#suppliers-profiling-and-scoring)
  - [Sentiment Analysis for ABC International<a name="sentiment-analysis"></a>](#sentiment-analysis-for-abc-international)
    - [News Feed API<a name="news-feed-api"></a>](#news-feed-api)
    - [Sentiment Analysis<a name="sentiment-analysis"></a>](#sentiment-analysis)

# Case Study: ABC International <a name="case-study"></a>

ABC International is a Fortune 500 company with presence in over 60 countries spread across the globe. It is a global conglomerate in true sense. With an annual revenue of over $50 Bn, ABC International figures in the top pharmaceutical companies. Majority of its revenue comes from Germany & USA. The company’s suppliers are largely concentrated in the Austria and China. With the current COVID situation, things became difficult for ABC International. While the company managed to stay afloat with the added pressure on its operations, the same couldn’t be said about its suppliers. The company needed to understand and segment the suppliers based on their liquidity risk.

## Supplier Financial Data Analysis <a name="supplier-financial-data"></a>

To accurately evaluate the financial health and long-term sustainability of a company, a number of financial metrics must be considered. Four main areas of financial health that should be examined are below.
 #### Liquidity
 Liquidity is the amount of cash and easily-convertible-to-cash assets a company owns to manage its short-term debt obligations.
 Following are the ratios, generally being used to understand liquidity power of a company
 

 - Current Ratio: Current Assets / Current Liabilities
 - Quick Ratio:  Current Assets (excluding inventory) / Current Liabilities

 #### Solvency
 Solvency is a company's ability to meet its debt obligations on an ongoing basis, not just over the short term. To evaluate solvency following is the ratio.
 
Debt/Equity ratio = Total Liabilities​ / Total Shareholders’ Equity
#### Profitability
The best metric for evaluating profitability is [net margin](https://www.investopedia.com/terms/n/net_margin.asp), the ratio of profits to total revenues.

## Financial Data ETL<a name="background-of-organization"></a>

![AWS Financial Data Extraction and Load](https://github.com/vaibhavmaurya/abc-case-study/blob/master/images/FiancialDataETL.png)

 1. AWS CloudWatch event is scheduled, which triggers Lambda function (Extract Financial Data) after every given interval. Lambda function either calls the rest api [financial data api](#financial-data-api) or run a [web crawler](#web-crawler).
 2. Lambda function (Extract Financial Data) extracts the financial data and push it to S3 bucket in JSON format.

```
{
	supplierId:
	FinancialReportDate
	netMargin : 
	currentRatio:
	quickRatio:
	debtEquitRatio:
}

```

3. This data can be analyzed using Amazon Athena by creating a table on that.
```
CREATE  EXTERNAL  TABLE  IF  NOT  EXISTS supplier_profile (
supplierId STRING, FinancialReportDate  Date, netMargin Decimal, currentRatio  Decimal, quickRatio Decimal, debtEquitRatio Decimal) ROW  FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'  WITH SERDEPROPERTIES ( "input.regex" = "^(?!#)([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+[^\(]+[\(]([^\;]+).*\%20([^\/]+)[\/](.*)$" ) LOCATION 's3://FinancialData-`MyRegion`/cloudfront/plaintext/';
```

4. Amazon Athena database can be connected to AWS Quick Insight for data visualization.

### Financial Data API<a name="financial-data-api"></a>
Following is the API can bes used to extract the Financial Ratios for a company.
 - [Financial Ratios API](https://financialmodelingprep.com/developer/docs/financial-ratio-free-api/)
 - [Financial Data API Doc](https://financialmodelingprep.com/developer/docs/)

Some additional public rest APIs to extract financial data are below.
 - [EOD Historical Data](https://eodhistoricaldata.com/?gclid=CjwKCAjw88v3BRBFEiwApwLeveHx3vXgJffrP7KSfAlsSTmLtUmbYV4ODzp4YxE5lIh93eeb_wk9qhoCwQUQAvD_BwE)
 - [Xignit](https://www.xignite.com/product/factset-fundamentals-financials#/DeveloperResources/Request/GetFinancialStatements)

### Web Crawler <a name="web-crawler"></a>
Another way to extract financial data is from websites like [MoneyControl](https://www.moneycontrol.com/financials/cityonlineservices/ratiosVI/COS%23COS).
In this approach an API needs to be created in python.

### Suppliers Profiling and Scoring<a name="profiling-and-scoring"></a>

![profiling](https://github.com/vaibhavmaurya/abc-case-study/blob/master/images/profiling.png)
The extracted ratios are different criterias to be evaluated for a supplier. There should be a mechanism to calculate the score combining all these criterias and give relative efficiency score.
One way to do that is [Analytic Hierarchy Process](https://www.pmi.org/learning/library/analytic-hierarchy-process-prioritize-projects-6608#:~:text=The%20multi%2Dcriteria%20programming%20made,the%201970s%20by%20Thomas%20L.).
Before AHP financial ratio data of all the suppliers need to be transformed, which can be done with [Business Rule Engine](https://github.com/venmo/business-rules). Following is the mechanism for profiling and scoring.
1. As soon data is updated to the S3 bucket (Financial data), it triggers Lambda function.
2. Lambda function calls AHP algorithm deployed in the ECS container service.
3. ECS container application generates the scores and push it to Simple Notification Service, which can be subscribed by any email service or mobile app and to S3 bucket.

## Sentiment Analysis for ABC International<a name="sentiment-analysis"></a>

![SentimentAnalysis](https://github.com/vaibhavmaurya/abc-case-study/blob/master/images/Sentiment.png)
 
### News Feed API<a name="news-feed-api"></a>
Following APIs can be used for news extraction for a given query.
   - [News API](https://newsapi.org/s/google-news-api) can be used to extract news using rest API
   - [Google News API](https://pypi.org/project/GoogleNews/) python library
    
### Sentiment Analysis<a name="sentiment-analysis"></a>

As per the above architecture diagram. The extracted text can be analyzed using AWS Comprehend and results can be pushed to S3 bucket.

 - Detect entities with predefined keywords. To learn specific keywords use following API
	[Create Entitiy Recognizer](https://docs.aws.amazon.com/comprehend/latest/dg/API_CreateEntityRecognizer.html)
	
 - Find language of the text.
	 [Dominant Language Detetction](https://docs.aws.amazon.com/comprehend/latest/dg/API_DetectDominantLanguage.html)
	 
 - Extract sentiment from the following API.
 [Detect Sentiment](https://docs.aws.amazon.com/comprehend/latest/dg/API_DetectSentiment.html)
