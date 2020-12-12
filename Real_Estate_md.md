
---
title: "Stat3340_Final_Project"
author: "Nicole Torrie & Qinyan Jiang"
date: "07/12/2020"
output:
  html_document: default
  pdf_document: default
---

```{r setup, include=FALSE}
library(dplyr)
knitr::opts_chunk$set(echo = TRUE)
```
Use rmarkdown::render()

## 1.0	Abstract
The purpose of our project is to research the Real Estate dataset through different images and models. The final product will be a model which can be used to determine housing price based on variables determined to have high influence on housing price. First, we will introduce our topic to give background context on the Real Estate market. We then introduce the dataset and prep the data for analysis. Next we analyze the relationships between variables and build a linear model. Then, we will further validate our model by fitting it to training data and testing it on a separate test subset of the data. Finally, we will summarize the research conclusions based on the results of data experiments and our personal experience.

## 2.0	Introduction
With the development of the economy, people's demand for houses is increasing, so the housing price has become a very important topic to research. There are also many factors that affect the housing price, such as the age of the house, the market trend when buying the house, the acesss to transportation, the convenience of life, the geographical location of the house and so on. It is important to study the historical data to find the relationship between housing price and explanatory variables. For buyers, this can help them make a better decision when buying a house that is best suited for their lifestyle. For sellers, this can help them make an estimate of the value of the house that they should be selling at. For investors, this can help them understand the trend of the market and make appropriate choices to make profits. Our study use the dataset that consists of six variables and 414 houses. We will conduct data visualization on the dataset and study on different model or methods to summarize the conclusions. We will include our code throughout this analysis so that other users can follow along and reproduce our final model. The main question we hope to address through this analysis is: which variables have highest influence on housing price per unit area? 


## 3.0 Data Description
### 3.1 Downloading the data

Download the Real Estate dataset from the source: https://www.kaggle.com/quantbruce/real-estate-price-prediction, or find it attached to the github repository: https://github.com/nicoletorrie/STAT3340_Term_Project


Load the necessary packages:
```{r}
library(maps)
library(mapdata)
library(maptools)  #for shapefiles
library(scales)  #for transparency
library(ggplot2)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
library(rgeos)
library(ggspatial)
library(ggmap)
library(dplyr)
library(tidyr)

```


Attach and view the Real Estate Data
```{r}
Data <- read.csv("Real estate.csv")  
attach(Data)
head(Data)
```

### 3.2 Data Information

Study Area: We first plot the data on a map so that we can get a better understanding of the study area. The code for two mapping methods are below: 

Create a basic map with a scalebar to understand the proximity of data points:
```{r}
# gene world map
world <- ne_countries(scale = "medium", returnclass = "sf")
ggplot(data = world) +
  geom_sf() +
  labs( x = "Longitude", y = "Latitude") +
  coord_sf(xlim = c(121.460,121.58), ylim = c(24.92,25.023), expand = FALSE) +
  annotation_scale(location = "bl", width_hint = 0.5) +
  annotation_north_arrow(location = "bl", which_north = "true", 
                         pad_x = unit(0.02, "in"), pad_y = unit(0.3, "in"),
                         style = north_arrow_fancy_orienteering) +
  geom_point(data = Data, aes(x = X6.longitude, y =X5.latitude ),col="red", size=2)+
  theme_bw()

```

Create a more detailed street map:
```{r}
qmplot(X6.longitude, X5.latitude, data = Data, maptype = "toner-lite", color = I("red"))+
         labs( x = "Longitude", y = "Latitude") 

```

According to the latitude and longitude information, the dataset is from Taiwan, on the outskirts of the country's capital city, Taipei. Now that we know our study area, we can look at the variables within the context of the study area.


One limitation to understanding the variables included in this analysis is that there are limited variable descriptions included in the online database from which we sourced our data, thus our definitions of the variables below are based on inference. 


The independent variables to be included in this analysis are as follows:
X1.transaction.date: the transaction date that the house sold on
X2.house.age: the age of the house
X3.distance.to.the.nearest.MRT.station: The distance between the house and the nearest MRT station. MRT is inferred to mean "Mass Rapid Transit" station.
X4.number.of.convenience.stores: The number of convenience stores near the house, with proximity undefined
X5.latitude: The latitude of the house location
X6.longitude: The longitude of the house location

The dependent variable is:
Y.house.price.of.unit.area: The price of the house per unit area. This is presented as per unit area, instead of overall house price to reflect the fact that houses considered in this model are of different sizes. We scale them to price per unit area so that the value can be compared. 


We ignore the first column with label "no". Because this is not a variable, and we will not include it in the analysis.

## 4.0 Methods

We began analysis by analyzing model variables and determining whether it makes sense to consider them in the Real Estate model. Based on these decisions, we re-formatted the dataset accordingly and added our new data point. Once our data was prepped and variables were formatted, we built a linear model with all variables from the new dataset included. This was to get an initial idea of model accuracy.

Crating diagnostic plots of the full model allowed us to identify influential leverage points and outliers in the data. We used cook's distance value to eliminate influential points from the dataset. In total there were 18 influential points identified. 

Upon removal of influential points, we re-fit the full model to the new dataset and re-created the diagnostic plots so as to test the following regression assumptions:

1. Linearity of the data. 
2. Normality of residuals. 
3. Homogeneity of residuals variance. 
4. Independence of residuals error terms.

Next we tested the correlation between variables using pairs plots. There did not appear to be strong linear correlation between any of the variables. We also tested for multicollinearity using VIF values. A VIF greater than 10 indicates multicollinearity, and suggests that a variable should be removed.

After confirming the variables and data points that should be considered in the final model, we used stepwise regression to create the strongest model, by eliminating any variables which had high associated AIC values. In addition, we compared the model output from stepwise regression to the model output created by backward regression. Both models were identical, which validated the model. We further validated the final model by splitting it into training and test data, creating a model using the training set, and comparing predicted and observed values of the test data run through the model. The RMSE values and the predicted vs. observed plots allowed us to draw conclusions on how accurately the model had predicted the test data.

The afore-mentioned methods are presented with R-code and analysis in the results section (5.0).



## 5.0 Results

### 5.1 Data prepping

We begin by prepping the dataset.

First, take a look at the model variables and decide if any variables or values should be re formatted or removed. Column 1, "No" can be removed, as the values in this column are merely a count of the rows in the dataset. 

Column 2, "X1.transaction.date" was divided to just include the transaction year, with the month and day values being eliminated. Year has influence on housing price for several reasons. First, the value of the dollar, (inflation or deflation) will vary by year, thus influencing a house's selling price. 

Column 3, "X2.house.age" is an important predictor of price, because house value can either decrease or increase with age. Older houses could be in some cases more desirable due to their antique character or historical value. On the other hand, these houses may be less desirable because of higher infrastructural upkeep costs or outdated features and style. 

Column 4, "X3.distance.to.the.nearest.MRT.station" is another important predictor of price. Proximity to public transit can cause house value to increase due to ease of transportation and accessibility that comes along with it. 

Column 5, "X4.number.of.convenience.stores" is similarly important. Access to convenience stores in close proximity means homeowners have the ability to pick up food items, or essential small household items last minute. Contrarily, a person living in a home far from any stores will have to rely on transportation to access these services. 

Column 6 and 7, "X5.latitude" and "X6.longitude" work together to provide the location of the house. For similar reasons as discussed with the previous variables, location does have impact on the retail price of a house. Often, houses in more desirable locations will be more expensive.

Reference: https://www.opendoor.com/w/blog/factors-that-influence-home-value

Split "X1.transaction.date" to just include the year of transaction date and remove column 1.
```{r}
#split X1.transaction.date
library(tidyr)
Data2<-separate(Data, X1.transaction.date, c("X1.transaction.date",NA,NA))
#make a new dataset with just columns 2-8 to eliminate column 1
Data3<-Data2[,2:8]
head(Data3)
#remove NA values
Data3<-Data3[complete.cases(Data3),]
```


Next, we add an additional data point to our dataset, to make the dataset unique for the purposes of this project. For this data point, we use the average value of each column to create the new value. In order to take the average of each column in the data frame, each column has to be converted from character to numeric type. 
```{r}
#check and convert the column types to numeric
Data3<-as.data.frame(Data3)
str(Data3)
Data4<-data.frame(lapply(Data3,as.numeric))
str(Data4)

#create values for the additional data point to add to the dataset
Newrow<-data.frame(t(colMeans(x=Data4, na.rm = TRUE)))
library(data.table)

#attach the additional data point to the end of the dataset. 
RE_Data <- rbind(Data4,Newrow)
str(RE_Data)
#RE_Data2<-as.data.frame(RE_Data)
head(RE_Data)
```



### 5.2 Linear Regression Model

Now that the data has been properly formatted, and the new datapoint is added we can start building the regression model. 

First, fit a regression model including all independent variables so we can gauge model accuracy.
```{r}
#library(MPV) 
#library(faraway)

#Fit the base model with all variables included
BaseModel<-lm(RE_Data$Y.house.price.of.unit.area~RE_Data$X1.transaction.date+RE_Data$X2.house.age+RE_Data$X3.distance.to.the.nearest.MRT.station+RE_Data$X4.number.of.convenience.stores+RE_Data$X5.latitude+RE_Data$X6.longitude)

summary(BaseModel)

```
The adjusted R-squared value of this full model is low, at 0.5748. This is indication that the base model is not well performing, and variables should be re-visited to determine if they should be removed from the model to make it more accurate. Before we re-assess the variables to include, check the plots to assess leverage points and outliers. 

```{r}
plot(BaseModel)
```

Based on the model diagnostic plots, it appears that observations 271, 313, and 114 and potentially a few others are outliers to the dataset. Removing these could improve model accuracy. 

Let's remove influential points:
```{r}
#use Cook's distance to eliminate outliers or variables with strong influence.
#for this analysis we eliminate values with a cook's distance value above 4/number of observations

cooksd=cooks.distance(BaseModel)
sample_size=length(RE_Data$X1.transaction.date)
influential<-as.numeric(names(cooksd)[(cooksd>(4/sample_size))])
RE_DataNEW2<-RE_Data[-influential,]

#Fit the base model with all variables included now that the outliers and leverage points have been removed
BaseModel2<-lm(RE_DataNEW2$Y.house.price.of.unit.area~RE_DataNEW2$X1.transaction.date+RE_DataNEW2$X2.house.age+RE_DataNEW2$X3.distance.to.the.nearest.MRT.station+RE_DataNEW2$X4.number.of.convenience.stores+RE_DataNEW2$X5.latitude+RE_DataNEW2$X6.longitude,Data=RE_DataNEW2)

summary(BaseModel2)
plot(BaseModel2)

```
18 variables were considered influential, and were removed from the dataset. The remaining values more closely follow a normal distribution, as shown by the Normal Q-Q plot. 

When the full model is fit with the new dataset, the adjusted R-squared value is 0.7215, which is much better than the model fit to the full dataset. Moving forward we will use RE_DataNEW2 as our dataset. 

Before we begin assessing which variables to include in the model, lets consider the regression assumptions. 

1. Linearity of the data. 
2. Normality of residuals. 
3. Homogeneity of residuals variance. 
4. Independence of residuals error terms.

To test these assumptions, re-visit the diagnostics plots:
```{r}
plot(BaseModel2,1)
```
Based on the Residuals vs. Fitted plot there is a slight pattern, but residuals are fairly evenly distributed around zero. For the purposes of continuing with this analysis we will say that this model meets the assumption #1 that the data is linear. 

```{r}
plot(BaseModel2,2)
```

The Normal Q-Q plot shows the points are distributed in a close line, meaning that the data is normally distributed. Thus this data meets assumption #2. 


```{r}
plot(BaseModel2,3)
```
The scale-location plot shows a mostly horizontal line with equally spread points, which indicates that there is homoskedasticity of the variables so we can assume that model assumption #3



```{r}
# Cook's distance
plot(BaseModel2, 4)
```
There are no additional leverage points with high Cooke's distance values, so it is confirmed that points of high leverage have been successfully removed from the model. 

Now we have confirmed that the data meets model assumptions and outliers and leverage points have been removed. To improve the model further, let's return to the variable selection and variable relationships in the following section.



### 5.3 Model Adequacy
Pairs plots are useful to look at the relationships (correlation) between variables. 
```{r}
#a ggpairs plot gives a bit more information about the correlation between variables
library(GGally)
ggpairs(RE_DataNEW2,title="Relationships Between Housing Variables")
```
```{r}
#(1) Scatterplot Matrix to show correlation
cor(RE_DataNEW2)
```
There does not appear to be strong linear relationships between any of the variables. 

Check for multicollinearity between variables
```{r}
library(car)
vif(BaseModel2)
```
None of these variables had VIF greater than 10. A VIF greater than 10 indicates multicollinearity, and suggests that a variable should be removed.



Next, we use stepwise regression to determine the best model. The stepwise model was also compared to the model output achieved from backward regression. Both methods created the same model output:

```{r}
library(tidyverse)
library(caret)
library(leaps)
library(MASS)

# Stepwise regression model from the full model
step.model2<-step(BaseModel2,direction=c("both"),trace=TRUE)
summary(step.model2)

```
Backward regression:
```{r}
#backward regression model from the full model
step.model.back<-step(BaseModel2,direction=c("backward"),trace=TRUE)
summary(step.model.back)
```


Based on stepwise regression, the most well-fitted model is: Y.house.price.of.unit.area=X1.transaction.date+X2.house.age+X3.distance.to.the.nearest.MRT.station+X4.number.of.convenience.stores+X5.latitude

A second model which could be considered is: Y.house.price.of.unit.area=X1.transaction.date+X2.house.age+X3.distance.to.the.nearest.MRT.station+X4.number.of.convenience.stores
```{r}
Simple_REmod<-lm(Y.house.price.of.unit.area~X1.transaction.date+X2.house.age+X3.distance.to.the.nearest.MRT.station+X4.number.of.convenience.stores,data=RE_DataNEW2)
summary(Simple_REmod)
```
This model has no consideration for latitude or longitude. It could be argued that latitude and longitude should not be considered in linear regression modeling, as there are other factors represented by location that aren't reflected in this analysis which merely uses the number values of latitude and longitude. However, excluding latitude causes the adjusted r-squared value to decrease, meaning that this model does not predict house price per unit area as accurately as the model which does include latitude.

We will continue using the step.model2 model outcome from the stepwise regression: Y.house.price.of.unit.area=X1.transaction.date+X2.house.age+X3.distance.to.the.nearest.MRT.station+X4.number.of.convenience.stores+X5.latitude


### 5.4 Model Validation

Validate stepmodel by splitting into training and test data
```{r}

#Train 5 times
set.seed(71168)

for(i in 1:5){
nsamp=ceiling(0.8*length(RE_DataNEW2$Y.house.price.of.unit.area))
training_samps=sample(c(1:length(RE_DataNEW2$Y.house.price.of.unit.area)),nsamp)
training_samps=sort(training_samps)
train_data  <- RE_DataNEW2[training_samps, ]
test_data <-   RE_DataNEW2[-training_samps, ]

train.lm <- lm(Y.house.price.of.unit.area~X1.transaction.date+X2.house.age+X3.distance.to.the.nearest.MRT.station+X4.number.of.convenience.stores+X5.latitude)

preds <- predict(train.lm,test_data)

plot(test_data$Y.house.price.of.unit.area,preds,xlim=c(0,80),ylim=c(0,80),xlab = "Observed",ylab = "Predicted")
abline(c(0,1))

RMSE<-sqrt(sum((preds-test_data$Y.house.price.of.unit.area)^2)/length(preds))


print(c(i,RMSE))

}


```
Lower values of RMSE indicate better fit. In general, points should ideally fall along the centre line of the residuals vs. fitted plot. All RMSE values were below 8. Because the range of the dependent variable in this case is large (roughly 11-70 house price per unit area), 8 is a relatively small RMSE, thus the trained model was a relatively good predictor of the test data.

 
## 6 - Conclusion

The final model included the following variables to predict house price per unit area: 
X1.transaction.date
X2.house.age
X3.distance.to.the.nearest.MRT.station
X4.number.of.convenience.stores
X5.latitude


The equation for predicting house price per unit area is:
```{r}
Y.house.price.of.unit.area=2.139e00*X1.transaction.date-3.207e-01*X2.house.age-3.889e-03*X3.distance.to.the.nearest.MRT.station+1.268e00*X4.number.of.convenience.stores+2.295e02*X5.latitude-9.995e03

```


The values of these coefficients is logical. For transaction date, larger value (more recent transaction) has a positive influence on the house price. For house age, larger value (older house) has a negative influence on the house price. For distance to MRT station, larger value (higher distance) has a negative influence on the house price. For number of convenience stores, larger value (more nearby stores) has positive influence on house price. For latitude, larger value (closer proximity to capitol city) has positive influence on house price. 

The final R squared value for this model was 0.7222. This means that the model can be used to predict house price per unit area, however it is not always extremely accurate. An R-squared value above 0.9 would be more ideal. There are a few reasons why this model may not be the best predictor for house price per unit area in Taiwan. Many factors influence housing price. This model did not consider factors such as updated appliances, condition of the house, lot size, proximity to schools, appealing views, and more. In addition, the housing market fluctuates greatly over time. House price per unit area could depend on how well the economy is doing in the month that the house sold, inflation or deflation of currency over time, the current value of currency in comparison to other countries, and demand for housing. In order to build a more accurate model to predict house price per unit area, a more robust dataset should be used, with a larger set of variables to consider. 

Source: https://www.opendoor.com/w/blog/factors-that-influence-home-value, https://www.toptal.com/finance/real-estate/real-estate-valuation


