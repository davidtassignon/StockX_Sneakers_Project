# Predicting the ROI of Sneakers on StockX.com
My first Data Science Project where I used a dataset published by StockX in combination with an unofficial StockX API to predict the ROI for Yeezy and Off-White Sneakers.

## Table of Content

1. [Background and Research Goal](#Background-and-Research-Goal)
2. [Data Overview](#Data-Overview)
3. [Exploratory Data Analysis](#Exploratory-Data-Analysis)
4. [Modelling the Data](#Modelling-the-Data)
5. [Model Results](#Model-Results)
6. [Limitations and Further Deployment](#Limitations-and-Further-Deployment)


## Background and Research Goal
Over the past 10 years flipping shoes - buying new sneakers and reselling the sneakers - is evolved to what is now the sneaker culture. You can think flipping sneakers for profit is worthless, but this once niche market has become a 2 billion market and it is projected to reach 6 billion by 2025.
One of the largest sneaker resale platforms is called StockX. It operates like a stock exchange website where users can place a bidding or asking prices, and when there is a match, the deal can come through. As you can imagine, this online marketplace is a Walhalla for data scientists or analaysts as each day 7000 to 10000 deals are made.

The beautifull ting about this marketplace is that all kind of people can access a limited pair of sneakers. So this means that all kind of people can participate on the marketplace and make profit on reselling sneakers. The aim of my project was to identify which features boost the resale price of a pair of sneakers and give you as reseller the highest return on investment.   

## Data Overview
### StockX Dataset
For this project I used a [dataset](CSV_files/StockX-Data-Contest-2019.xlsx) that StockX published as a challenge where people could win 1000 dollar. The data that was given consisted of a random sample of all Off-White x Nike and Yeezy 350 sales between 9/1/2017 and 13/02/2019. In total there were 99956 observations, 27,794 Off-White, and 72,162 Yeezy. To create this sample, they took a random, fixed percentage X of deals on StockX for each colorway, on each day since September 2017. So for each day the Off-White Jordan 1 was on the market, they randomly selected X% of its sale from each day. 
Table 1 shows which variables were in the dataset.

![Screen Shot 2020-12-21 at 10 47 09](https://user-images.githubusercontent.com/70702631/102763476-ff675d00-4379-11eb-8e18-9722c5043a83.png)

###### Table 1: Variables in public StockX Dataset

### StockX Unofficial API
To get some extra information per sneakername I decided to use an unofficial StockX API. As shown in the code below, I iterated over each sneakername that I had in my initial dataset returning 37 extra variables. After closely investigating each variable, I decided to keep the variables shown in Table 2.
```powershell
from stockxsdk import Stockx
stockx_ = Stockx()

large_sneaker_dict = {}
for sneakername in sneakernames:
    large_sneaker_dict[sneakername] = stockx_.search(sneakername)
```

![Screen Shot 2020-12-21 at 11 22 45](https://user-images.githubusercontent.com/70702631/102766737-e57c4900-437e-11eb-9241-d89fe1ff93a9.png)

###### Table 2: Usefull variables for StockX API 

### Feature Engineering
After merging the two dataframe together, I decided to do some extra feature engineering. I created the following variables:
* ROI: the target variable calculated as the difference between the sale- and retail price divided by the retail price expressed as a percentage.
* Day since Release Date: a continuous variable which represents the number of days the order is made before or after the release date.
* Within 1 year: a binary variable which represent whether the order is made within 1 year from the release date.
* Month, Week, Day: some extra seasonal variables extracted from the order date.
* Colorway: this variable retrieved from the api was used to extract the single colors per sneaker resulting in a variable per color.
* Day since Release Date_2: the same variable as "Day since Release Date" but shifted with +29 days to be able to take this time dependence into account in the modelling stage. 

After feature engineering I had 125 variables and 79710 observations. For a closer look into the code used I would like to direct you to [my jupyter notebook](code_notebook.ipynb) section 2.4 and for the data to this [dataset](CSV_files/stockx_df.csv) which consists of all the sales.

## Exploratory Data Analysis
Before continuing with the EDA and visualizations I subsetted the dataset to only the orders within 1 year of the release date. This gave me the following [subsetted dataset](CSV_files/stockx_df1y.csv) which I used for this project. 

The following section will visualize some variables to give you an understanding of the data that was used for this project. For a closer look into the code used and other variables I direct you to the [my jupyter notebook](code_notebook.ipynb) section 3.

### Brand
This shows the distribution of the brand variable. As you can see, there are more Yeezy's than Off-White sneakers in the dataset. 
<details>
<summary>Figure 1</summary>
<img width="300" alt="Screen Shot 2020-12-21 at 11 39 07" src="https://user-images.githubusercontent.com/70702631/102768252-27a68a00-4381-11eb-919d-42e825905d1d.png">
</details>

### ROI per Sneaker name and Type
Figure 2 shows the average ROI per sneaker name and Figure 3 shows the number of sneaker types and the average ROI per sneaker type. What already can be seen from these visualizations is that the Jordan 1 Retro High, Nike Air Presto, and the Nike Blazer Mid are worth keeping an eye out for. 
<details>
<summary>Figure 2</summary>
<img width="879" alt="Screen Shot 2020-12-21 at 11 37 30" src="https://user-images.githubusercontent.com/70702631/102782405-94c61980-4399-11eb-95b3-3ee5dba7caa3.png">
</details>
<details>
<summary>Figure 3</summary>
<img width="879" alt="Screen Shot 2020-12-21 at 11 43 27" src="https://user-images.githubusercontent.com/70702631/102768642-c7fcae80-4381-11eb-96c8-333f374d88ab.png">
</details>

### ROI over Time
Figure 4 shows how the ROI changes over time before and after the release date of the sneaker. We see that before the release date the ROI is very high, the demand is high and sellers can ask a lot for these not even released pairs. After the release, the ROI drops whereafter it increases a bit. Figure 5 looks a bit closer to this. So what is happening is that before the release date the prices are high as everyone wants to have the first pair of those limited sneakers. The light color indicated the number of sneakers sold on the platform. The darker color shows us that a lot of sneakers are sold in the first week after the release. People are making a quick profit within this first week. Hereafter, the ROI decreases as the market gets saturated. So already advising; do not buy sneakers before the release date as pre-release sales account for the biggest buyer premium. Try to buy those sneakers for the retail price and quickly sell them within the first 7 days. After the first two weeks, try to find sneakers twice the retail price and keep them until somewhere between day 60 and 70 to sell them.

<details>
<summary>Figure 4</summary>
<img width="500" alt="Screen Shot 2020-12-21 at 11 56 00" src="https://user-images.githubusercontent.com/70702631/102769850-810fb880-4383-11eb-9477-e541611a0d88.png">
</details>
<details>
<summary>Figure 5</summary>
<img width="879" alt="Screen Shot 2020-12-21 at 11 46 37" src="https://user-images.githubusercontent.com/70702631/102768922-32adea00-4382-11eb-9dcf-3a7bf9baa788.png">
</details>

### Daily, Weekly, Monthly difference of ROI
Figure 6 shows the daily, weekly, and monthly difference of the ROI. As you can see, the number of orders is very equal over the days. However, the darker colors indicate that the ROI is higher on Wednesday, Tuesday, Monday accordingly. When looking at the months, between March and May, and between August and October there are fewer sneakers on the marketplace resulting in a higher ROI. The week figure just gives a closer look per week for people to use. 
<details>
<summary>Figure 6</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 11 52 23" src="https://user-images.githubusercontent.com/70702631/102769497-0181e980-4383-11eb-91c2-f343aa1ec68b.png">
</details>

### Size
Looking at Figure 7 it seems that there aren’t very big differences between the different shoe sizes. However, based on this figure I would suggest buying sneakers with sizes that have a whole size number as it seems that they are higher in demand and slightly higher in ROI. 
<details>
<summary>Figure 7</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 11 59 47" src="https://user-images.githubusercontent.com/70702631/102770208-098e5900-4384-11eb-8840-7f41e5bfb6a9.png">
</details>

## Modelling the Data
### Preparation
Before applying some models, I prepared the data in the following way. The variables that were used for the modelling were: 
```powershell
target_variable = 'ROI'
categorical_variables = ['brand', 'shoe_size', 'style_id','make', 'day', 'month', 'week']
continous_variable = ['day_since_release_2']
```
Hereafter I did the following:
1. Initialized the target- and predictor variables
2. Dummified the categorical variables
3. Setted up a Train-Test split with random state 34

### Modelling
I fitted the following linear and tree-based models using a cross-validation of 5 resulting in the R2 scores shown in the tables below.
## Linear-based Models
Model                  | Train Score     | Test Score        | R2 Score          | MSE               | RMSE            
--------------------   | -------------   | -------------     | -------------     | -------------     | -------------     
Lasso Regressor *      | 82.87%          | 81.17%            | 82.76%            | 5142.35           | 71.71        
Linear SVM Regressor * | 82.87%          | 81.17%            | 82.76%            | 5142.35           | 71.71        
###### * Hyperparameters tuned by GridSearch

## Tree-based Models
Model                  | Train Score     | Test Score        | R2 Score          | MSE               | RMSE            
--------------------   | -------------   | -------------     | -------------     | -------------     | -------------     
Random Forest Regressor| 99.58%          | 96.74%            | 97.45%            | 891.04            | 29.85        
HistGradientBoosting   | 95.83%          | 94.59%            | 95.05%            | 1478.60           | 38.45      
MLPRegressor           | 97.52%          | 96.36%            | 96.67%            | 995.34            | 31.55  

According to the modeling results, the Random Forest Regressor preforms best as it has the highest cross-validated accuracy (R2 score) and lowest values for the MSE and RMSE. For the deployment of the model into some sort of web application which could be used by people, the Random Forest Regressor would be prefered. The next section will however also show the results of the Lasso Regressor as they are easier to interpret. 

## Model Results
### Lasso Regressor
The variables with the highest coefficients are visualized below. For convenience, I also decided to visualize the coefficients per variable to make it easier to interpret.

<details>
<summary>Variables with Highest Coefficients</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 46 28" src="https://user-images.githubusercontent.com/70702631/102783562-65181100-439b-11eb-93e3-92ee39b001eb.png">
</details>

<details>
<summary>Type of Sneaker</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 48 50" src="https://user-images.githubusercontent.com/70702631/102783720-a8727f80-439b-11eb-819c-e9140a939e6d.png">
</details>

<details>
<summary>Style of Sneaker</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 49 42" src="https://user-images.githubusercontent.com/70702631/102783797-c809a800-439b-11eb-9f4e-6713a610ca79.png">
</details>

<details>
<summary>Day of Sale</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 50 40" src="https://user-images.githubusercontent.com/70702631/102783976-056e3580-439c-11eb-82e8-545c5c39dc52.png">
</details>

<details>
<summary>Week of Sale</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 50 50" src="https://user-images.githubusercontent.com/70702631/102784108-32224d00-439c-11eb-821f-9f6f49a29167.png">
</details>

<details>
<summary>Month of Sale</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 50 57" src="https://user-images.githubusercontent.com/70702631/102784154-45cdb380-439c-11eb-92d7-201ffa501ab1.png">
</details>

<details>
<summary>Size of Sneaker</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 51 03" src="https://user-images.githubusercontent.com/70702631/102784219-5e3dce00-439c-11eb-9570-872a802d532e.png">
</details>

<details>
<summary>Color of Sneaker</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 51 26" src="https://user-images.githubusercontent.com/70702631/102784254-6dbd1700-439c-11eb-959f-2c27896b40d8.png">
</details>

### Random Forest Regressor
<details>
<summary>Feature Importance</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 14 55 47" src="https://user-images.githubusercontent.com/70702631/102784406-a2c96980-439c-11eb-808c-883c4f5c4e08.png">
</details>


## Limitations and Further Deployment
### Limitations
* Only 50 different sneakers from the brand Yeezy and Nike Off-White in the dataset.
* Some sneakers are first generation sneakers and therefore are more expensive.
* Not including number of sales per day on the StockX Marketplace.
* Data covers sales from 2017 to 2019, the predictive power of the model will decrease over time.
* Proposing candidate based on hot features, I'm not able to give a predicted ROI.
### Further Deployment
* There is an idea of implementing the Random Forest Regressor model in some sort of user-friendly web application for people to use. Nevertheless, this is something I still need to explore.

Thank you for reading this readme, feel free to reach out if you have any questions.
