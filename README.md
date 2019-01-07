# Practical Machine Learning Project

Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

Data

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv



## Getting Started

In this project, firstly loading required libraries. Then import and prepare data. and lastly present modeling and results.

### Loading libraries

```
> library(caret)
> library(rpart)
> library(rpart.plot)
> library(randomForest)
> library(corrplot)
```

### Importing data

```
> lft.train = read.csv('pml-training.csv')
> lft.test  = read.csv('pml-testing.csv')
```

### Explore imported data

```
> dim(lft.train)
[1] 19622   160

> dim(lft.test)
[1]  20 160
```

### Clean data

```
> sum(complete.cases(lft.train))
[1] 406

> sum(complete.cases(lft.test))
[1] 0

> lft.train <- lft.train[, colSums(is.na(lft.train)) == 0]
> lft.test <- lft.test[, colSums(is.na(lft.test)) == 0]
```


### Removing unnecessary columns

```
> classe <- lft.train$classe
> trainRemove <- grepl("^X|timestamp|window", names(lft.train))
> lft.train <- lft.train[, !trainRemove]
> trainCleaned <- lft.train[, sapply(lft.train, is.numeric)]
> trainCleaned$classe <- classe
> testRemove <- grepl("^X|timestamp|window", names(lft.test))
> lft.test <- lft.test[, !testRemove]
> testCleaned <- lft.test[, sapply(lft.test, is.numeric)]
```

### Split data

```
> set.seed(22519) 
> inTrain <- createDataPartition(trainCleaned$classe, p=0.70, list=F)
> trainData <- trainCleaned[inTrain, ]
> testData <- trainCleaned[-inTrain, ]
```

### Data modeling

```
> contRf <- trainControl(method="cv", 5)
> modelRf <- train(classe ~ ., data=trainData, method="rf", trControl=contRf, ntree=250)
> modelRf

Random Forest 

13737 samples
   52 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

No pre-processing
Resampling: Cross-Validated (5 fold) 
Summary of sample sizes: 10991, 10989, 10988, 10990, 10990 
Resampling results across tuning parameters:

  mtry  Accuracy   Kappa    
   2    0.9901722  0.9875668
  27    0.9900997  0.9874753
  52    0.9833295  0.9789091

Accuracy was used to select the optimal model using the largest value.
The final value used for the model was mtry = 2.
```

```
> predictRf <- predict(modelRf, testData)
> confusionMatrix(testData$classe, predictRf)

Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 1673    0    0    0    1
         B    4 1133    2    0    0
         C    0    5 1020    1    0
         D    0    0   22  941    1
         E    0    0    0    4 1078

Overall Statistics
                                          
               Accuracy : 0.9932          
                 95% CI : (0.9908, 0.9951)
    No Information Rate : 0.285           
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.9914          
 Mcnemar's Test P-Value : NA              

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            0.9976   0.9956   0.9770   0.9947   0.9981
Specificity            0.9998   0.9987   0.9988   0.9953   0.9992
Pos Pred Value         0.9994   0.9947   0.9942   0.9761   0.9963
Neg Pred Value         0.9991   0.9989   0.9951   0.9990   0.9996
Prevalence             0.2850   0.1934   0.1774   0.1607   0.1835
Detection Rate         0.2843   0.1925   0.1733   0.1599   0.1832
Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
Balanced Accuracy      0.9987   0.9972   0.9879   0.9950   0.9987
```

```
> accuracy <- postResample(predictRf, testData$classe)
> accuracy

Accuracy     Kappa 
0.9932031 0.9914019 
```

```
> oose <- 1 - as.numeric(confusionMatrix(testData$classe, predictRf)$overall[1])
> oose
[1] 0.006796941
```

### Prediction

```
result <- predict(modelRf, testCleaned[, -length(names(testCleaned))])
> result

[1] B A B A A E D B A A B C B A E E A B B B
Levels: A B C D E
```
