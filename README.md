---
title: 'Practical Machine Learning Course Project: Prediction Assignment Writeup'
author: "Catalina Barco Castillo"
date: "10/11/2020"
output:
  html_document: default
  pdf_document: default
---

***Background***

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.

***Data***

The training data for this project are available here: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv
The test data are available here: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv
The data for this project come from this source: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har. 

***Preparation***

```{r}
library(caret)
library(lattice)
library(rpart.plot)
library(rpart)
```

***The dataset***

```{r}
training_data <- read.csv('pml-training.csv', na.strings = c("NA", "#DIV/0!", ""))
test_data <- read.csv('pml-testing.csv', na.strings = c("NA", "#DIV/0!", ""))
```
*Training data has 19622 observations and 160 variables, while testing data provides 20 observation for 160 variables.*

```{r}
clean_column <- colSums(is.na(training_data))/nrow(training_data) < 0.95
clean_training_data <- training_data[,clean_column]
colSums(is.na(clean_training_data))/nrow(clean_training_data)
colSums(is.na(clean_training_data))
```
*All missing values (NA) were removed*

```{r}
clean_training_data <- clean_training_data[,-c(1:7)]
clean_test_data <- test_data[,-c(1:7)]
```
*Columns from 1 to 7 did not provide relevant information to the model, hence, were removed*

```{r}
inTrainIndex <- createDataPartition(clean_training_data$classe, p=0.75)[[1]]
training_training_data <- clean_training_data[inTrainIndex,]
training_crossval_data <- clean_training_data[-inTrainIndex,]
```
*Creation of cross validation data set*

```{r}
allNames <- names(clean_training_data)
clean_test_data <- test_data[,allNames[1:52]]
```

***Decision tree***

```{r}
decision_tree <- train(classe ~., method='rpart', data=training_training_data)
decision_tree_prediction <- predict(decision_tree, training_crossval_data)
confusionMatrix(table(training_crossval_data$classe, decision_tree_prediction)) 
rpart.plot(decision_tree$finalModel)
```

***Random forest***
```{r}
rfMod <- train(classe ~., method='rf', data=training_training_data, ntree=100)
rfPrediction <- predict(rfMod, training_crossval_data)
confusionMatrix(table(training_crossval_data$classe, rfPrediction))
```

***Prediction***
```{r}
predict(rfMod, clean_test_data)
```

***Conclusion***

Decision tree presents an accuracy of 48.1% [CI 46.7 - 49.51, p=1.000] vs. random forest 99.3% [CI 99.06-99.54, p=<0.002]
