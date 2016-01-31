---
##                                 Practical Machine Learning

#### author: John Espitia


### Executive Summary

Six participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: 

     1. Exactly according to the specification (Class A)
     2. Throwing the elbows to the front (Class B)
     3. Lifting the dumbbell only halfway (Class C)
     4. Lowering the dumbbell only halfway (Class D) 
     5. Throwing the hips to the front (Class E)


This report will describe how the data captured are used to identify the parameters involved in predicting the movement involved based on the above classification, and then to predict the movement for 20 test cases.

The training data were divided into two groups, a training data and a validation data (to be used to validate the data), to derived the prediction model by using the training data, to validate the model where an expected out-of-sample error rate of less than 0.5%, or 99.5% accuracy, would be acceptable before it is used to perform the prediction on the 20 test cases - that must have 100% accuracy (to obtain 20 points awarded).

The training model developed using Random Forest was able to achieve   100% accuracy and predict the 20 test cases with the same accuracy.

### Code and Results
 
Loading libraries 

```{r}


library (lattice)
library(ggplot2)
library(caret)
library(randomForest)
```

#### Loading Data and Cleaning

First all, we want to load the data sets into R.

```{r }
 
Trainingset <- read.csv("C:/pml-training.csv", na.strings = c("NA","#DIV/0!",""))
Testingset <- read.csv("C:/pml-testing.csv", na.strings = c("NA","#DIV/0!",""))

```
Secondly, We remove columns that are irrelevant.Also, We remove either NA or empty colum values.
```{r }
#Irrelevant columns
Trainingset <- Trainingset[,-c(1:7)]
Testingset <- Testingset[,-c(1:7)]


#NA or Empty Values
Trainingset <- Trainingset[, names(Trainingset)[sapply(Trainingset, function (x)! (any(is.na(x) | x == "")))]]
Testingset <- Testingset[, names(Testingset)[sapply(Testingset, function (x)! (any(is.na(x) | x == "")))]]
``` 

###Cross Validation

The training data set contains 53 variables and 19622 obs.
The testing data set contains 53 variables and 20 obs.
In order to perform cross-validation, the training data set is partionned into 2 sets: subTraining (60%) and subTest (40%).
This will be performed using random subsampling without replacement.
```{r }
Intrain <- createDataPartition(Trainingset$classe, p = 0.6, list = FALSE)
Subtraining <- Trainingset[Intrain,]
Subtesting <- Trainingset[-Intrain,]
```

We can see that each level frequency is within the same order of magnitude of each other in the next graph


```{r }
plot(Subtraining$classe, col="Green", main=" Classe in Subtraining Data Set", xlab="classe levels", ylab="Frequency")
```


###Prediction model: Using random forest
#### Subtrainig set

```{r }

model <- randomForest(classe ~. , data=Subtraining, method="class")

# Predicting:
prediction <- predict(model, Subtraining, type = "class")

# Test results on Subtraining data set:
confusionMatrix(prediction, Subtraining$classe)
```
 
From the training subset and the testing subset the accuracy is very high

#### Subtesting set
```{r }
 
model2 <- randomForest(classe ~. , data=Subtesting, method="class")

# Predicting:
prediction2 <- predict(model2, Subtesting, type = "class")

# Test results on Subtesting data set:
confusionMatrix(prediction2, Subtesting$classe)

```

###Applying Model to Test Set

 When whe apply the prediction model to the testing data, the results are: 

```{r }
FinalPrediction <- predict(model, Testingset, type = "class")
FinalPrediction
```

Finally, we use the following code to generate the text file for submission.
```{r }
answers <- FinalPrediction

write_files <- function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("Prediction_for_case_",i,".txt")
    write.table(x[i], file=filename, quote=FALSE, row.names=FALSE, col.names=FALSE)
  }
}
write_files(answers)
```

