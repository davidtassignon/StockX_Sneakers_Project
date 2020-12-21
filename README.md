# Predicting the ROI of Sneakers on StockX.com
My first Data Science Project where I used the sneaker data set of StockX in combination with an unofficial StockX API to predict the ROI for Yeezy and Off-White Sneakers.

## Table of Content

1. [Background and Research Goal](#Background-and-Research-Goal)
2. [Data Overview](#Data-Overview)
3. [Exploratory Data Analysis](#Exploratory-Data-Analysis)
4. [Modelling the Data](#Modelling-the-Data)
5. [Model Results](#Model-Results)
6. [Limitations and Further Deployment](#Limitations-and-Further-Deployment)


## Background and Research Goal
Over the past 10 years flipping shoes (buying new sneakers and reselling the sneakers) is evolved to what is now the sneaker culture. You can think flipping sneakers for profit is silly, but this once niche market has grown to become a 2 billion market and it is projected to reach 6 billion by 2025.
One of the largest sneaker resale platforms is called StockX. It operates like a stock exchange website where users can place a bidding or asking prices, and when there is a match, the deal can come through. As you can imagine, this online marketplace is a Walhalla for data scientists or analysts as each day 7000 to 10000 deals are made.

The beautifull ting about this new marketplace is that all kind of people can now access a limited pair of sneakers. So this means that all kind of people can also participate on the StockX marketplace and make profit on reselling sneakers. The aim of my project was to identify which features boost the resale price of a pair of sneakers and give you as reseller the highest return on investment.   

## Data Overview
### StockX Dataset
For this project I used a dataset that StockX (see CSV_files/StockX-Data-Contest-2019.xlsx) published as a Challenge where people could win 1000 dollar. The data that was given consisted of a random sample of all Off-White x Nike and Yeezy 350 sales between 9/1/2017 and 13/02/2019. In total there were 99956 deals, 27,794 Off-White, and 72,162 Yeezy. To create this sample, they took a random, fixed percentage X of deals on StockX for each colorway, on each day since September 2017.
So for each day the Off-White Jordan 1 was on the market, they randomly selected X% of its sale from each day. 
Table 1 shows which variables were in the dataset.

![Screen Shot 2020-12-21 at 10 47 09](https://user-images.githubusercontent.com/70702631/102763476-ff675d00-4379-11eb-8e18-9722c5043a83.png)

###### Table 1: Variables in public StockX Dataset

### StockX Unofficial API
To get some extra information per sneaker name I decided to use an unofficial StockX API. As shown below, I iterated over each sneaker name that I had in my initial dataset returning me 37 extra variables. After closely investigating each variable, I decided to keep the variables shown in Table 2.
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
* ROI: the target variable calculated as the difference between the sale and retail price divided by the retail price expressed as a percentage.
* Day since Release Date: a continuous variable which represents the number of days the order is made before or after the release date.
* Within 1 year: a binary variable which represent whether the order is made within 1 year from the release date.
* Month, Week, Day: some extra seasonal variables extracted from the order date.
* Colorway: this variable from the api was used to extract the single colors per sneaker resulting in a variable per color.
* Day since Release Date_2: the same variable as "Day since Release Date" but shifted with +29 days to be able to take this time dependence into account in the modelling stage. 

On top of the variables I already had, this feature engineering gave me a total of 125 variables. 
For a closer look into the code used to perform this feature engineering I would like to direct you to the "code_notebook.ipynb" section 2.4.

## Exploratory Data Analysis
Before continuing with the EDA and corresponding visualizations I subsetted the dataset to all the sales within 1 year. 

The following section will visualize some variables to give you an understanding of the data that was used for this project. For a closer look into the code used and other variables I direct you to the "code_notebook.ipynb" section 3.
### Brand
<details>
<summary>Figure 1</summary>
<img width="300" alt="Screen Shot 2020-12-21 at 11 39 07" src="https://user-images.githubusercontent.com/70702631/102768252-27a68a00-4381-11eb-919d-42e825905d1d.png">
</details>

### ROI per Sneaker name and Type
<details>
<summary>Figure 2</summary>
<img width="879" alt="Screen Shot 2020-12-21 at 11 37 30" src="https://user-images.githubusercontent.com/70702631/102782405-94c61980-4399-11eb-95b3-3ee5dba7caa3.png">
</details>

<details>
<summary>Figure 3</summary>
<img width="879" alt="Screen Shot 2020-12-21 at 11 43 27" src="https://user-images.githubusercontent.com/70702631/102768642-c7fcae80-4381-11eb-96c8-333f374d88ab.png">
</details>

### ROI over Time
<details>
<summary>Figure 4</summary>
<img width="500" alt="Screen Shot 2020-12-21 at 11 56 00" src="https://user-images.githubusercontent.com/70702631/102769850-810fb880-4383-11eb-9477-e541611a0d88.png">
</details>

<details>
<summary>Figure 5</summary>
<img width="879" alt="Screen Shot 2020-12-21 at 11 46 37" src="https://user-images.githubusercontent.com/70702631/102768922-32adea00-4382-11eb-9dcf-3a7bf9baa788.png">
</details>

### Daily, Weekly, Monthly difference of ROI
<details>
<summary>Figure 6</summary>
<img width="600" alt="Screen Shot 2020-12-21 at 11 52 23" src="https://user-images.githubusercontent.com/70702631/102769497-0181e980-4383-11eb-91c2-f343aa1ec68b.png">
</details>

### Size
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
