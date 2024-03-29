---
title: "Prediction on WLE Dataset"
author: "CesarTC"
date: "02/11/2019"
output: 
  html_document: 
    fig_caption: yes
    keep_md: yes
---



## Introduction

In this project, we are going to explore the [Weight Lifting Exercise Dataset], a part of the Human Activity Recognition Project briefly described [here]. Our main objective is to create a Machine Learning strategy to accurately predict the way a group of testers are performing dumbbell biceps curls. As we can learn from the documentation, the researchers defined five possible ways of performing the exercise, one right and four wrong. That means we are dealing with a multi class classification problem.

To perform the prediction, we have decided to use a random forest algorithm. This technique performs better than other multi class classification algorithms in most cases, according to the literature and open competitions such as those hold by Kaggle. We used a k-fold strategy to estimate the out of sample error rate and, finally, performed a prediction on the test dataset.

## Preparing the dataset
We start by loading the necessary packages and the test and train datasets.


```r
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(stringr))
suppressPackageStartupMessages(library(caret))
suppressPackageStartupMessages(library(rattle))

download.file('https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv', 'training_set.csv')
download.file('https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv', 'test_set.csv')

train <- as_tibble(read.csv('training_set.csv'))
test <-  as_tibble(read.csv('test_set.csv'))
```

We should have a look at the test set, just to have an idea of what kind of data we have to predict on.


```r
str(test)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	20 obs. of  160 variables:
##  $ X                       : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ user_name               : Factor w/ 6 levels "adelmo","carlitos",..: 6 5 5 1 4 5 5 5 2 3 ...
##  $ raw_timestamp_part_1    : int  1323095002 1322673067 1322673075 1322832789 1322489635 1322673149 1322673128 1322673076 1323084240 1322837822 ...
##  $ raw_timestamp_part_2    : int  868349 778725 342967 560311 814776 510661 766645 54671 916313 384285 ...
##  $ cvtd_timestamp          : Factor w/ 11 levels "02/12/2011 13:33",..: 5 10 10 1 6 11 11 10 3 2 ...
##  $ new_window              : Factor w/ 1 level "no": 1 1 1 1 1 1 1 1 1 1 ...
##  $ num_window              : int  74 431 439 194 235 504 485 440 323 664 ...
##  $ roll_belt               : num  123 1.02 0.87 125 1.35 -5.92 1.2 0.43 0.93 114 ...
##  $ pitch_belt              : num  27 4.87 1.82 -41.6 3.33 1.59 4.44 4.15 6.72 22.4 ...
##  $ yaw_belt                : num  -4.75 -88.9 -88.5 162 -88.6 -87.7 -87.3 -88.5 -93.7 -13.1 ...
##  $ total_accel_belt        : int  20 4 5 17 3 4 4 4 4 18 ...
##  $ kurtosis_roll_belt      : logi  NA NA NA NA NA NA ...
##  $ kurtosis_picth_belt     : logi  NA NA NA NA NA NA ...
##  $ kurtosis_yaw_belt       : logi  NA NA NA NA NA NA ...
##  $ skewness_roll_belt      : logi  NA NA NA NA NA NA ...
##  $ skewness_roll_belt.1    : logi  NA NA NA NA NA NA ...
##  $ skewness_yaw_belt       : logi  NA NA NA NA NA NA ...
##  $ max_roll_belt           : logi  NA NA NA NA NA NA ...
##  $ max_picth_belt          : logi  NA NA NA NA NA NA ...
##  $ max_yaw_belt            : logi  NA NA NA NA NA NA ...
##  $ min_roll_belt           : logi  NA NA NA NA NA NA ...
##  $ min_pitch_belt          : logi  NA NA NA NA NA NA ...
##  $ min_yaw_belt            : logi  NA NA NA NA NA NA ...
##  $ amplitude_roll_belt     : logi  NA NA NA NA NA NA ...
##  $ amplitude_pitch_belt    : logi  NA NA NA NA NA NA ...
##  $ amplitude_yaw_belt      : logi  NA NA NA NA NA NA ...
##  $ var_total_accel_belt    : logi  NA NA NA NA NA NA ...
##  $ avg_roll_belt           : logi  NA NA NA NA NA NA ...
##  $ stddev_roll_belt        : logi  NA NA NA NA NA NA ...
##  $ var_roll_belt           : logi  NA NA NA NA NA NA ...
##  $ avg_pitch_belt          : logi  NA NA NA NA NA NA ...
##  $ stddev_pitch_belt       : logi  NA NA NA NA NA NA ...
##  $ var_pitch_belt          : logi  NA NA NA NA NA NA ...
##  $ avg_yaw_belt            : logi  NA NA NA NA NA NA ...
##  $ stddev_yaw_belt         : logi  NA NA NA NA NA NA ...
##  $ var_yaw_belt            : logi  NA NA NA NA NA NA ...
##  $ gyros_belt_x            : num  -0.5 -0.06 0.05 0.11 0.03 0.1 -0.06 -0.18 0.1 0.14 ...
##  $ gyros_belt_y            : num  -0.02 -0.02 0.02 0.11 0.02 0.05 0 -0.02 0 0.11 ...
##  $ gyros_belt_z            : num  -0.46 -0.07 0.03 -0.16 0 -0.13 0 -0.03 -0.02 -0.16 ...
##  $ accel_belt_x            : int  -38 -13 1 46 -8 -11 -14 -10 -15 -25 ...
##  $ accel_belt_y            : int  69 11 -1 45 4 -16 2 -2 1 63 ...
##  $ accel_belt_z            : int  -179 39 49 -156 27 38 35 42 32 -158 ...
##  $ magnet_belt_x           : int  -13 43 29 169 33 31 50 39 -6 10 ...
##  $ magnet_belt_y           : int  581 636 631 608 566 638 622 635 600 601 ...
##  $ magnet_belt_z           : int  -382 -309 -312 -304 -418 -291 -315 -305 -302 -330 ...
##  $ roll_arm                : num  40.7 0 0 -109 76.1 0 0 0 -137 -82.4 ...
##  $ pitch_arm               : num  -27.8 0 0 55 2.76 0 0 0 11.2 -63.8 ...
##  $ yaw_arm                 : num  178 0 0 -142 102 0 0 0 -167 -75.3 ...
##  $ total_accel_arm         : int  10 38 44 25 29 14 15 22 34 32 ...
##  $ var_accel_arm           : logi  NA NA NA NA NA NA ...
##  $ avg_roll_arm            : logi  NA NA NA NA NA NA ...
##  $ stddev_roll_arm         : logi  NA NA NA NA NA NA ...
##  $ var_roll_arm            : logi  NA NA NA NA NA NA ...
##  $ avg_pitch_arm           : logi  NA NA NA NA NA NA ...
##  $ stddev_pitch_arm        : logi  NA NA NA NA NA NA ...
##  $ var_pitch_arm           : logi  NA NA NA NA NA NA ...
##  $ avg_yaw_arm             : logi  NA NA NA NA NA NA ...
##  $ stddev_yaw_arm          : logi  NA NA NA NA NA NA ...
##  $ var_yaw_arm             : logi  NA NA NA NA NA NA ...
##  $ gyros_arm_x             : num  -1.65 -1.17 2.1 0.22 -1.96 0.02 2.36 -3.71 0.03 0.26 ...
##  $ gyros_arm_y             : num  0.48 0.85 -1.36 -0.51 0.79 0.05 -1.01 1.85 -0.02 -0.5 ...
##  $ gyros_arm_z             : num  -0.18 -0.43 1.13 0.92 -0.54 -0.07 0.89 -0.69 -0.02 0.79 ...
##  $ accel_arm_x             : int  16 -290 -341 -238 -197 -26 99 -98 -287 -301 ...
##  $ accel_arm_y             : int  38 215 245 -57 200 130 79 175 111 -42 ...
##  $ accel_arm_z             : int  93 -90 -87 6 -30 -19 -67 -78 -122 -80 ...
##  $ magnet_arm_x            : int  -326 -325 -264 -173 -170 396 702 535 -367 -420 ...
##  $ magnet_arm_y            : int  385 447 474 257 275 176 15 215 335 294 ...
##  $ magnet_arm_z            : int  481 434 413 633 617 516 217 385 520 493 ...
##  $ kurtosis_roll_arm       : logi  NA NA NA NA NA NA ...
##  $ kurtosis_picth_arm      : logi  NA NA NA NA NA NA ...
##  $ kurtosis_yaw_arm        : logi  NA NA NA NA NA NA ...
##  $ skewness_roll_arm       : logi  NA NA NA NA NA NA ...
##  $ skewness_pitch_arm      : logi  NA NA NA NA NA NA ...
##  $ skewness_yaw_arm        : logi  NA NA NA NA NA NA ...
##  $ max_roll_arm            : logi  NA NA NA NA NA NA ...
##  $ max_picth_arm           : logi  NA NA NA NA NA NA ...
##  $ max_yaw_arm             : logi  NA NA NA NA NA NA ...
##  $ min_roll_arm            : logi  NA NA NA NA NA NA ...
##  $ min_pitch_arm           : logi  NA NA NA NA NA NA ...
##  $ min_yaw_arm             : logi  NA NA NA NA NA NA ...
##  $ amplitude_roll_arm      : logi  NA NA NA NA NA NA ...
##  $ amplitude_pitch_arm     : logi  NA NA NA NA NA NA ...
##  $ amplitude_yaw_arm       : logi  NA NA NA NA NA NA ...
##  $ roll_dumbbell           : num  -17.7 54.5 57.1 43.1 -101.4 ...
##  $ pitch_dumbbell          : num  25 -53.7 -51.4 -30 -53.4 ...
##  $ yaw_dumbbell            : num  126.2 -75.5 -75.2 -103.3 -14.2 ...
##  $ kurtosis_roll_dumbbell  : logi  NA NA NA NA NA NA ...
##  $ kurtosis_picth_dumbbell : logi  NA NA NA NA NA NA ...
##  $ kurtosis_yaw_dumbbell   : logi  NA NA NA NA NA NA ...
##  $ skewness_roll_dumbbell  : logi  NA NA NA NA NA NA ...
##  $ skewness_pitch_dumbbell : logi  NA NA NA NA NA NA ...
##  $ skewness_yaw_dumbbell   : logi  NA NA NA NA NA NA ...
##  $ max_roll_dumbbell       : logi  NA NA NA NA NA NA ...
##  $ max_picth_dumbbell      : logi  NA NA NA NA NA NA ...
##  $ max_yaw_dumbbell        : logi  NA NA NA NA NA NA ...
##  $ min_roll_dumbbell       : logi  NA NA NA NA NA NA ...
##  $ min_pitch_dumbbell      : logi  NA NA NA NA NA NA ...
##  $ min_yaw_dumbbell        : logi  NA NA NA NA NA NA ...
##  $ amplitude_roll_dumbbell : logi  NA NA NA NA NA NA ...
##   [list output truncated]
```

As we can see, there is a lot of missing values on that dataset. So we are going to get rid of those columns and work only with the ones where we have information. After that, we are going to focus on preparing the train dataset to present only the columns with data in the test set.


```r
compl_test <- test[, !apply(is.na(test), 2, all)]
compl_train <- train[, !apply(is.na(test), 2, all)]
```

It is kind of a coincidence that the last column in the test set has values, because it allowed us to use the simple code above to obtain the train dataset ready without missing the so important 'classe' variable - the one we want to predict in the test set.

The first seven variables are just identifiers and should not have any explanatory power. All of the rest we are going to try and use to train our model.


```r
compl_train <- compl_train[,-c(1:7)]
```

## Estimating out of sample error

We used k-fold technique to estimate the out of sample error of our model. We have also decided to preprocess our data with Principal Component Analysis, to reduce the dimension of the dataset we're training on. Still, the next operation is going to take a while. Also, because we are using the function train, from caret, and preprocessing with pca, and the function doing the actual work is randomForest from the equally named package, we have a conflict in one particular parameter (mtry) that raises warning messages from the randomForest function. More on that in [this post on Stack Overflow]. That, and that alone, is why we are using suppressWarnings in the call to train.


```r
k <- 10
g_size <- round(length(compl_train$pitch_belt)/k, 0)
n_adjust <- length(compl_train$pitch_belt)%%k

k_group <- c(rep(1, times = g_size),
             rep(2, times = g_size),
             rep(3, times = g_size),
             rep(4, times = g_size),
             rep(5, times = g_size),
             rep(6, times = g_size),
             rep(7, times = g_size),
             rep(8, times = g_size),
             rep(9, times = g_size),
             rep(10, times = g_size + n_adjust))


set.seed(12345)

compl_train <- compl_train %>% 
    mutate(random_var = rnorm(length(compl_train$pitch_belt), 1, 10)) %>% 
    arrange(random_var) %>% 
    mutate(k_group = k_group)
    
fit_rf <- list()
acc_rf <- list()
confusion_rf <- list()
for (i in 1:k) {
    i <- 1
    ds_train <- compl_train %>% 
        filter(k_group != i) %>% 
        select(-random_var, -k_group)
    
    ds_test <- compl_train %>% 
        filter(k_group == i) %>% 
        select(-random_var, -k_group)
    
    fit_rf[[i]] <- suppressWarnings(train(classe~., data = ds_train, method = 'rf',
                         preProcess = 'pca', ntree = 300))
    
    predict_rf <- predict(fit_rf[[i]], newdata = ds_test)
    
    check_rf <- sum(predict_rf == ds_test$classe)
    
    acc_rf[[i]] <- check_rf/length(predict_rf)
    confusion_rf[[i]] <- fit_rf[[i]]$finalModel$confusion
}
```

We can now have a look on the expected out of sample error rate and the mean confusion matrix of our experiment. First, the accuracy measurement.


```r
acc_rf <- unlist(acc_rf)
mean(acc_rf)
```

```
## [1] 0.9852192
```

And now the mean confusion matrix, that took a little more coding.


```r
mean_conf <- matrix(nrow = dim(confusion_rf[[1]])[1], ncol = dim(confusion_rf[[1]])[2])
elements <- list()
for (i in 1:dim(confusion_rf[[1]])[1]) {
    for (j in 1:dim(confusion_rf[[1]])[2] - 1) {
        for (k in 1:length(confusion_rf)) {
            elements[[k]] <- confusion_rf[[k]][i,j]
        }
        mean_conf[i,j] <- mean(unlist(elements))
    }
}

mpe <- vector()
for (i in 1:dim(mean_conf)[1]) {
    mpe[i] <- mean_conf[i,i]/sum(mean_conf[i,], na.rm = TRUE)
}


mean_conf[,6] <- mpe
row.names(mean_conf) <- c('A', 'B', 'C', 'D', 'E')
colnames(mean_conf) <- c('A', 'B', 'C', 'D', 'E', 'mean_per_class_error')
mean_conf
```

```
##      A    B    C    D    E mean_per_class_error
## A 4995   12   17    3    3            0.9930417
## B   52 3326   25    0    7            0.9753666
## C    3   41 3018   23    4            0.9770152
## D    3    3  104 2770    3            0.9608047
## E    0   11   15   17 3205            0.9867611
```

## Training the final model and predicting on the test sample

Now that we have estimates of the out of sample error rate, we can use all the information available in the train dataset to train our model. After that, we'll perform the prediction in the test dataset, where we expect to get 98.5% of the classes right.


```r
ds_train <- compl_train %>% 
        select(-random_var, -k_group)
    
fit_rf <- suppressWarnings(train(classe~., data = ds_train, method = 'rf',
                         preProcess = 'pca', ntree = 300))

ds_test <- compl_test[,-c(1:7, length(compl_test))]

predict_rf <- predict(fit_rf, newdata = ds_test)

final <- as_tibble(cbind(ds_test, predicted_classe = predict_rf))
```

Our prediction is stored in the dataset called 'final'. This is the distribution of the predicted variable.


```r
table(final$predicted_classe)
```

```
## 
## A B C D E 
## 8 7 1 1 3
```
[Weight Lifting Exercise Dataset]:http://web.archive.org/web/20161224072740/http://groupware.les.inf.puc-rio.br/static/WLE/WearableComputing_weight_lifting_exercises_biceps_curl_variations.csv
[here]:http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har
[this post on Stack Overflow]:https://stackoverflow.com/questions/49186277/caret-method-rf-warning-message-invalid-mtry-reset-to-within-valid-rang
