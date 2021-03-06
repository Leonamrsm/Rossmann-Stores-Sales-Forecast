# Rossmann Stores Sales Forecast

## Predicting the income of the units of a chain store

![](img/cover.png)

## Business Problem

Rossmann is one of the largest drug store chains in Europe with around 56,200 employees and more than 4000 stores.[1](https://www.retail-index.com/sectors/personalcareretailersineurope.aspx) In 2019 Rossmann had more than €10 billion turnover in Germany, Poland, Hungary, the Czech Republic, Turkey, Albania, Kosovo and Spain. 

Store managers were tasked with predicting their daily sales for up to six weeks in advance, in order to define a budget for stores renovation. It is believed that sales are influenced by many factors, including promotions, competition, school and state holidays, seasonality, and locality. This task was assigned to the Data Science team of the whole chain, who must model the historical database in order to generate the desired forecasting. 

The [database](https://www.kaggle.com/c/rossmann-store-sales) spans around 2.5 years in time (between 2013 and 2015) and 1115 stores in total.

## Attribute List
- Id - an Id that represents a (Store, Date) duple within the test set
- Store - a unique Id for each store
- Sales - the turnover for any given day (this is what you are predicting)
- Customers - the number of customers on a given day
- Open - an indicator for whether the store was open: 0 = closed, 1 = open
- StateHoliday - indicates a state holiday. Normally all stores, with few exceptions, are closed on state holidays. Note that all schools are closed on public holidays and weekends. a = public holiday, b = Easter holiday, c = Christmas, 0 = None
- SchoolHoliday - indicates if the (Store, Date) was affected by the closure of public schools
- StoreType - differentiates between 4 different store models: a, b, c, d
- Assortment - describes an assortment level: a = basic, b = extra, c = extended
- CompetitionDistance - distance in meters to the nearest competitor store
- CompetitionOpenSince[Month/Year] - gives the approximate year and month of the time the nearest competitor was opened
- Promo - indicates whether a store is running a promo on that day
- Promo2 - Promo2 is a continuing and consecutive promotion for some stores: 0 = store is not participating, 1 = store is participating
- Promo2Since[Year/Week] - describes the year and calendar week when the store started participating in Promo2
- PromoInterval - describes the consecutive intervals Promo2 is started, naming the months the promotion is started anew. E.g. "Feb,May,Aug,Nov" means each round starts in February, May, August, November of any given year for that store

## Business Assumptions
- The days when stores were closed were removed from the analysis.
- Only stores with sales values bigger than 0 were considered.
- For stores which did not have Competition Distance information, it was considered that the distance should be the longest distance observed in the data set.

## Solution Strategy

The resolution of the challenge was carried out following the CRISP (CRoss-Industry Standard Process for data mining) methodology, which is a cyclical approach that streamlines the delivery of value.

![](img/crisp_cycle.png)


In order to solve this challenge, the work went along the following steps:
1. **Data Description:** understanding of the status of the database and dealing with missing values properly. Basic statistics metrics furnish an overview of the data.  
2. **Feature Engineering:** derivation of new attributes based on the original variables aiming to better describe the phenomenon that will be modeled, and to supply interesting attributes for the Exploratory Data Analysis.
3. **Data Filtering:** filtering of records and selection of attributes that do not contain information for modeling or that do not match the scope of the business problem.
4. **Exploratory Data Analysis (EDA):** exploration of the data searching for insights and seeking to understand the impact of each variable on the upcoming machine learning modeling.
5. **Data Preparation:** preprocessing stage required prior to the machine learning modeling step.
6. **Feature Selection:** selection of the most significant attributes for training the model.
7. **Machine Learning Modeling:** implementation of a few algorithms appropriate to the task at hand. In this case, models befitting the *regression* assignment - *i.e.*, forecasting a continuous value, namely sales.
8. **Hyperparameter Fine Tuning:** search for the best values for each of the parameters of the best performing model(s) selected from the previous step.
9. **Translation and Interpretation of the Model Performance:** conversion of the performance metrics of the Machine Learning model to a more tangible business result.
10. **Deployment of Model to Production:** publication of the model in a cloud environment so that the interested people can access its results to improve business decisions.

Eleven hypotheses were validated in the exploratory data analysis. Of these, the 3 most relevant hypotheses were:

- **Hypothesis 1:** stores with greater assortment have a bigger income.  
- 
   ***True:*** among the 3 possible assortments for each unit, those in the 'extra' category sell more in average than those in 'extended', which, by its turn, sell more in average than those in 'basic'.

![](img/H1.png)

- **Hypothesis 2:** stores perform better in terms of sales after the 10th day of each month.  
- 
   ***False:*** the average performance is better for the first 10 days of the month.

![](img/H10.png)

- **Hypothesis 3:** stores should sell less during school holidays.
- 
   ***False:*** in average, Stores sell more on school holidays.

![](img/H12.png)


## Machine Learning Modeling Performance

As the data varies over time, and the objective is to forecast sales in the next 6 weeks, time series cross validation was applied as shown in the image below.

![](img/ts_cross_validation.png)

In time series cross-validation, a reduced portion of the training database is separated in each iteration, where a final period is set aside for validation. In order to forecast sales for the next 6 weeks, at each new iteration of cross validation, the last 6 weeks of the training set have been separated for validation. The cross-validation performance was the average of each of these iterations.

Four different models (linear regression, regularized linear regression - lasso, random forest and XGBoost Regressor) were evaluated using cross-validation. 

Were used 5 folds to perform the cross validation.The results in terms of Mean Absolute Error (MAE), Mean absolute percentage error (MAPE) and Root Mean Square Error (RMSE), were:

|Model|MAE|MAPE|RMSE|
|-----------------------------|------------------|-------------|------------------|
|Random forest regressor      |837.80 +/- 214.98 |0.12 +/- 0.02|1256.26 +/- 310.60|
|Linear regression            |2078.09 +/- 295.43|0.3 +/- 0.02 |2946.66 +/- 468.06|
|Lasso                        |2117.99 +/- 342.74|0.29 +/- 0.01|3058.17 +/- 506.07|
|XGBoost regressor            |7049.73 +/- 589.3 |0.95 +/- 0.0 |7718.99 +/- 689.76|

	
Despite having the worst results, the XGB Regressor was chosen to continue the project, as it is a model that has been showing excellent results in several data science competitions and is lighter than the Random Forest Regressor, and can be more easily published in production. 

## Hyperparameter Tuning

Using the random search precedure, with different values for the parameters "n_estimators", "eta", "max_depth", "subsample", "colsample_bytree" and "min_child_weight", 10 different iterations of XGBoost were performed, all evaluated using cross-validation. The values of MAE, MAPE and RMSE are detailed in the notebook for all the iterations.

The performance of the chosen model was:

|Model|MAE|MAPE|RMSE|
|-----------------------------|------------------|-------------|------------------|
|XGBoost regressor            |770.787           |0.116        |1110.18|


And then the model was trained with the entire training data. The performance on the test data was:


|Model|MAE|MAPE|RMSE|
|-----------------------------|------------------|-------------|------------------|
|XGBoost regressor            |826.29 +/- 99,62  |0.11 +/- 0.01|1200.79 +/- 172.27|

## Business Perfromance	

Finally, with the model trained, it's time to translate model performance into business performance. Considering the MAE obtained in the forecast for each store, during the test period, the best and worst sales scenarios for each store were projected.

Below is the performance of 5 randomly sampled stores, showing the total sales prediction for the next 6 weeks:

|store       | predictions| worst scneario| best scenario|MAE|MAPE|
|------------|-----------|----------|-----------|--------|------|
|573         |324,304,34 |320,451,50| 322,157,19| 852,84 | 0,08 |
|863         |163,751.41	|163,430.16	|164,072.65	|321.24	|0.07|
|811         |226,083.36	|225,222.91	|226,943.81	|860.45	|0.18|
|178         |277,968.44	|277,191.01	|278,745.87	|777.43	|0.11|
|289         |411,175.59	|410,125.14	|412,226.05	|1,050.46	|0.08|

Being **worst scenario** = prediction - MAE and **best scenario** = prediction + MAE

### Total Performance

By adding the sales forecasts for all stores for the next 6 weeks, the following results were obtained:

| Scenario| Values|
|---------|-------|
|predictions| 286,746,880.00|
|worst scenarion| 286,746,880.00|
|best scenario  | 287,609,874.99|

### Machine Learning Performance


![](img/overall_performance.png)

In the end, the model had an average percentage error of 11%. If 11% error is not acceptable, it is possible to improve it following the CRISP methodology. It may be considered to train stores individually or even a smaller group of them, for example. Another possibility is to perform fine tuning for more iterations. And finally, other models could be used such as recurrent neural networks.
## Model in production

The model was finally put into production and operated via a Telegram chatbot. For this, in addition to the final trained model, a class in python was created with the entire data processing pipeline, an API handler and an application to manage the messages. All files were hosted on Heroku (https://www.heroku.com/); the production data was also stored in its cloud.

The following scheme represents all these files.


![](img/app_scheme.png)



A demonstration of the app: 

![](img/rossmann_bot.gif)

To access the bot via telegram, click on the icon below:

[<img alt="Telegram" src="https://img.shields.io/badge/Telegram-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white"/>](http://t.me/leonam_RossmanBot)

Notice that the bot might be idle most of the time, so be patient on your request (it should not take more than 1 minute to have a response).
