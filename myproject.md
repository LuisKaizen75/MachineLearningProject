---
title: "MyProject"
author: "Luis Carlos Garzón"
date: "05/08/2020"
output: 
  html_document:
    keep_md: true
---

# 1. Getting the data
In this section, I downloaded the data and made some adjusts to it like eliminating the columns with the user info and replacing the NA values.


```r
trainurl<- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

testurl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

download.file(trainurl,"traindata.csv")
download.file(testurl,"testdata.csv")

traindata<-read.csv("traindata.csv")
testdata<-read.csv("testdata.csv")

traindata<-traindata[-c(1,2)]
testdata<-testdata[-c(1,2)]
outid<-grep("classe",names(traindata))
traindata$classe<-as.factor(traindata$classe)
traindata[is.na(traindata)]<-0
```

# 2. Data selection  

In this section, because the dataset has 157 variables, I look for and select the more important ones to save computation time in the model creation process. Also, I created a test set from the training data provided to do cross validation.


```r
set.seed(333)
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
#obtaining an extra test data
inTrain = createDataPartition(traindata$classe, p=0.7)[[1]]
mytestdata<-traindata[-inTrain,]
traindata<-traindata[inTrain,]

# RFE

control <- rfeControl(functions=rfFuncs, method="cv", number=10)

results <- rfe(traindata[,-outid],traindata[,outid], sizes=c(5,10), rfeControl=control)

results
```

```
## 
## Recursive feature selection
## 
## Outer resampling method: Cross-Validated (10 fold) 
## 
## Resampling performance over subset size:
## 
##  Variables Accuracy  Kappa AccuracySD   KappaSD Selected
##          5   0.9996 0.9995  0.0003838 0.0004854        *
##         10   0.9991 0.9989  0.0005741 0.0007261         
##        157   0.9977 0.9971  0.0015139 0.0019147         
## 
## The top 5 variables (out of 5):
##    raw_timestamp_part_1, cvtd_timestamp, roll_belt, num_window, yaw_belt
```
In this process, I looked for the sets of 5 or 10 variables that can predict the outcome. As the results suggests, the set of the variables: *raw_timestamp_part_1, cvtd_timestamp, roll_belt, num_window, yaw_belt*. Can predict the outcome with high accuracy. These variables will be used in model training.
  
# 3. Model selection  
  With the previous variables, I trained a random forest model and test it results in a confusion matrix against the test sample I extracted from the training data.

```r
impvar<-c("raw_timestamp_part_1", "cvtd_timestamp", "roll_belt", "num_window", "yaw_belt","classe")
traindata <- traindata[impvar]


rfmodel<-train(classe ~.,method = "rf", data = traindata)

mytestdata<-mytestdata[impvar]
confusionMatrix(mytestdata$classe,predict(rfmodel,mytestdata))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674    0    0    0    0
##          B    1 1138    0    0    0
##          C    0    0 1024    2    0
##          D    0    0    0  964    0
##          E    0    0    0    0 1082
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9995          
##                  95% CI : (0.9985, 0.9999)
##     No Information Rate : 0.2846          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9994          
##                                           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9994   1.0000   1.0000   0.9979   1.0000
## Specificity            1.0000   0.9998   0.9996   1.0000   1.0000
## Pos Pred Value         1.0000   0.9991   0.9981   1.0000   1.0000
## Neg Pred Value         0.9998   1.0000   1.0000   0.9996   1.0000
## Prevalence             0.2846   0.1934   0.1740   0.1641   0.1839
## Detection Rate         0.2845   0.1934   0.1740   0.1638   0.1839
## Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
## Balanced Accuracy      0.9997   0.9999   0.9998   0.9990   1.0000
```
  
As the results show, this model predicted the test sample with 100% accuracy. Thus, I will use this model with the test data provided.  

# 4. Predicting the test data
  In this section, I extracted the relevant variables from the test data and predicted the *classe* based on the value of the variables using the model created in the previous section.  
  

```r
impvar2<-c("raw_timestamp_part_1", "cvtd_timestamp", "roll_belt", "num_window", "yaw_belt")

forquiz<-predict(rfmodel,testdata[impvar2])
#forquiz
```
I suppressed the output because that are the answers for the *Course Project Prediction Quiz*. I got a grade of 100%. Thus I can say that the model works.

# 5. Comments and conclusion
Here I will explain all the topics that the instructions say that the report should have and why I made those decisions.  

## How I build the model  

This topic was discussed in the report. In synthesis, the model was build in two steps. First, identifying the more relevant variables. I made that because the computation time required to train a model with 157 variables is so long for my standard computer, also, using all the variables can lead to overfitting as can be seen in the *RFE* results table. Second, training a random forest model. I made this decision because I think that the random forest is the best model for this case. Because the state of the variables can be used to predict the outcome as leaves and the outcome is a factor.  

## How I used cross-validation  

I used the cross-validation creating an extra test set. I made this decision because the test set provided don't have the outcome value. After all, it is meant to be used for the *Course Project Prediction Quiz*. With my test set, I tested my model to validate its accuracy.  

## The out sample error
I am surprised about getting a 100% grade in the *Course Project Prediction Quiz*. I do not know if maybe I selected a predictor that should not be selected. However, in the two tests, I made the model predict the outcome with 100% accuracy. I suppose that in this case there is no out sample error. But, maybe with different test data, the model can have out of sample error because it uses only 5 predictors.





