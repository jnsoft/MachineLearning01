---
title: "Practical Machine Learning"
author: "Johan Nilsson"
date: "Sunday, November 23, 2014"
output: html_document
---

Init, loading required packages:

```r
library(ggplot2)
library(lattice)
library(caret)
library(randomForest)
```

Helper functions to write out predictions of test data and sample data

```r
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}

randomRows = function(df,n){
  return(df[sample(nrow(df),n),])
}
```

Read in data, remove columns witch are mostly NA:s or empty and remove the index column.


I'm removing the column with the name of the participant, even though the name is present in the test data, the participants name would have no meaning when predicting outcomes from other participants outside of this test.

Also, all timestamps and window inforamtion is removed as I consider those data have no value here as the activities are not time dependent series.
head(data[,1:10])

```r
not_usefull <- c(-1,-2,-3,-4,-5,-6)
data <- data[,not_usefull]
```
Divide data into train and cross validation subsets. I sample 2000 rows to train with which would be enough to get the out of sample error to about 0.95. If I hade more time and computer power, I would have used the whole traing set:

```r
data <- randomRows(data,2000)
inTrain <- createDataPartition(y=data$classe, p=0.7, list=FALSE)
training <- data[inTrain,]
CV <- data[-inTrain,]
```

For prediction I choose the random forrest method as it has preformed well in my earlier experiments with machine learning:

```r
set.seed(1000)
modFit <- train(classe ~ ., method="rf", data=training)
#modFit <- train(classe ~ ., method="rf", prox=TRUE,importance=TRUE,do.trace=TRUE, data=training)
```

Calculate Out of sample error, using the cross validation set:

```r
answer <- predict(modFit,CV)
mean(answer==CV$classe)
```

```
## [1] 0.9181
```

Read in test data, format the data the same as the training data but also remove the problem_id column:

```r
test=read.csv('pml-testing.csv')
test <- test[,no_na]
test <- test[,-1]
test <- test[,not_usefull]
test <- test[-53]
```

Predict outcomes of test data and write to files:

```r
answers = predict(modFit,test)
answers
```

```
##  [1] C A A A A E D D A A B C B A E E A B A B
## Levels: A B C D E
```

```r
pml_write_files(answers)
```











