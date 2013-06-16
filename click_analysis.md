URL Click Analysis
========================================================

This is an R Markdown document. Markdown is a simple formatting syntax for authoring web pages where you can embed R codes. 

The [source data](https://s3.amazonaws.com/coursera_intro_data_science_project/coursera.sanitized.csv). Special thanks to Nandita for coming up with an idea for processing URL and Refer URL variables.


Data inspection and cleanup
---------------------------

### load raw data

```r
# raw <- read.csv('coursera.sanitized.csv') sapply(raw[1, ], class)
```


### click: 
An integer representing whether or not a click occurred - this is the outcome we want to predict

```r
# table(raw$click) just those actually got any clicks clicked <-
# raw[raw$click==1,]
```


### datehour: 
A string describing the day and hour when the ad was served

```r
# head(raw$datehour) sum(is.na(raw$datehour)) convert string to datetime
# object pdata <- raw datehour <- strptime(raw$datehour, '%Y-%m-%d
# %H:%M:%S') pdata$wday <- datehour$wday pdata$hour <- datehour$hour
# pdata$datehour <- datehour
```


### v_id: 
An integer representing a channel in which a viewer has been served through. 

```r
# table(pdata$v_id) just those actually got any clicks table(clicked$v_id)
# check missing values sum(is.na(pdata$v_id))
```

### b_id: 
An integer representing a class of user based on their interests

```r
# length(unique(pdata$b_id)) just those actually got any clicks
# length(unique(clicked$b_id)) check missing values sum(is.na(pdata$b_id))
# inspect data head(pdata$b_id) check the number of NULL values
# sum(pdata$b_id=='\\N') rename '\\N' to 0 and convert the factor into
# integer levels(pdata$b_id)[levels(pdata$b_id) == '\\N'] <- '0'
# pdata$b_id <- as.numeric(as.character(pdata$b_id))
```

### t_id: 
An integer identifying a specific ad

```r
# length(unique(pdata$t_id)) just those actually got any clicks
# length(unique(clicked$t_id)) check missing values sum(is.na(pdata$t_id))
```

### seller: 
An integer representing the seller providing the wholesaler with the impression

```r
# length(unique(pdata$seller)) just those actually got any clicks
# length(unique(clicked$seller)) check missing values
# sum(is.na(pdata$seller)) check the value range
# min(pdata$seller[!is.na(pdata$seller)])
# max(pdata$seller[!is.na(pdata$seller)])
# pdata$seller[is.na(pdata$seller)] <- 3000 sum(is.na(pdata$seller))
```

### country: 
A string representing the clicker's country of origin

```r
# table(pdata$country) just those actually got any clicks
# table(clicked$country) check NULL values levels(pdata$country)
# levels(pdata$country)[levels(pdata$country) == ''] <- '00'
# levels(pdata$country)
```

### state: 
A string representing the clicker's state of origin

```r
# length(unique(pdata$state)) just those actually got any clicks
# length(unique(clicked$state)) check NULL values levels(pdata$state)
# levels(pdata$state)[levels(pdata$state) == ''] <- '00'
# levels(pdata$state)
```

### url: 
An encrypted string representing the URL the ad was displayed on

```r
# length(unique(pdata$url)) just those actually got any clicks
# length(unique(clicked$url)) check NULL values
# levels(pdata$url)[levels(pdata$url) == '']
```


Check the frequency of urls = views. Special thanks to Nandita for coming up with this metric. 

```r
# quantile(table(pdata$url))
# barplot(sort(table(pdata$url),decreasing=TRUE), col='blue')
# barplot(sort(table(clicked$url),decreasing=TRUE), col='blue') views <-
# as.data.frame(table(pdata$url)) names(views) <- c('url', 'views')
```


Check the number of traffic sources - this represents the popularity of a given URL similar to PageRank algorithm. This is also based on Nandita's idea.

```r
sourceCount <- function(page_url) {
    ref <- pdata$refer_url[pdata$url == page_url]
    return(length(unique(ref)))
}

# exetime <- system.time(views$srcs <- sapply(views[,1], sourceCount))
# exetime[3]/60
```


Add the views and source counts to the data

```r
# pdata <- merge(pdata,views,all=TRUE)
```


### refer_url: 
An encrypted string representing the referrer URL

```r
# length(unique(pdata$refer_url)) just those actually got any clicks
# length(unique(clicked$refer_url)) check NULL values
# levels(pdata$refer_url)[levels(pdata$refer_url) == '']
```


Check the frequency of refer_urls

```r
# quantile(table(pdata$refer_url))
# barplot(sort(table(pdata$refer_url),decreasing=TRUE), col='blue')
# barplot(sort(table(clicked$refer_url),decreasing=TRUE), col='blue')
```


Add the frequency to the data - this represents how frequently a given referrer sends traffic - the larger frequency, the more active. 

```r
# referrals <- as.data.frame(table(pdata$refer_url)) names(referrals) <-
# c('refer_url', 'referrals') pdata <- merge(pdata,referrals,all=TRUE)
```


### Check the cleanup result

```r
# pdata <- pdata[,c('datehour','wday','hour', 'v_id', 'b_id', 't_id',
# 'seller', 'country', 'state', 'views', 'srcs', 'referrals', 'url',
# 'refer_url', 'click')] summary(pdata) head(pdata) save(pdata,
# file='processed.rda')
load(file = "processed.rda")
```


Deal with the class imbalance
---------------------------------
### Reduce the majority class by subsumpling

Here we are making an assumption that we can ignore most of the cases where a click didn't occur without impeding our ability to detect the cases where it did occur.


```r
# first check the ratio between the two classes
table(pdata$click)
```

```
## 
##      0      1 
## 782725    217
```

```r

# drop url and refer_url variables
pdata <- pdata[, !(names(pdata) %in% c("datehour", "url", "refer_url"))]
# convert click to a factor
pdata$click <- as.factor(pdata$click)

# subample the super majority class to reduce the ratio to 10:1
maj <- pdata[pdata$click == 0, ]
subSampling <- sample(1:dim(maj)[1], size = 217 * 10, replace = FALSE)
subSampled <- rbind(pdata[pdata$click == 1, ], maj[subSampling, ])
table(subSampled$click)
```

```
## 
##    0    1 
## 2170  217
```

```r
summary(subSampled)
```

```
##       wday           hour           v_id           b_id     
##  Min.   :2.00   Min.   : 0.0   Min.   :0.00   Min.   :   0  
##  1st Qu.:2.00   1st Qu.: 4.0   1st Qu.:1.00   1st Qu.:   0  
##  Median :3.00   Median :13.0   Median :1.00   Median :   0  
##  Mean   :3.23   Mean   :11.5   Mean   :1.13   Mean   : 136  
##  3rd Qu.:4.00   3rd Qu.:17.0   3rd Qu.:1.00   3rd Qu.:  32  
##  Max.   :6.00   Max.   :23.0   Max.   :3.00   Max.   :1539  
##                                                             
##       t_id          seller     country       state          views      
##  Min.   :9595   Min.   :   0   00:  16   AB     :1420   Min.   :    1  
##  1st Qu.:9600   1st Qu.: 459   BR:   0   MB     : 829   1st Qu.:  324  
##  Median :9601   Median :1263   CA:2368   SK     : 101   Median : 2163  
##  Mean   :9602   Mean   :1177   EG:   0   00     :  16   Mean   : 6599  
##  3rd Qu.:9606   3rd Qu.:1362   ES:   0   ON     :   8   3rd Qu.: 8143  
##  Max.   :9612   Max.   :3000   US:   3   BC     :   7   Max.   :35898  
##                                          (Other):   6                  
##       srcs         referrals     click   
##  Min.   :  1.0   Min.   :    1   0:2170  
##  1st Qu.:  2.0   1st Qu.:  853   1: 217  
##  Median :  4.0   Median : 5843           
##  Mean   : 25.4   Mean   :18781           
##  3rd Qu.:  8.0   3rd Qu.:42085           
##  Max.   :539.0   Max.   :61337           
## 
```


### Setup Cross Validation

In order to test the predictive model, we will split the downsampled data into two subsets. We will also take a sampling from the original data without downsampling the majority class.


```r
# split data into two subsets - 2/3 training set, 1/3 test set
set.seed(333)
trainSamples <- sample(1:dim(subSampled)[1], size = (dim(subSampled)[1]/3 * 
    2), replace = FALSE)
train <- subSampled[trainSamples, ]
test <- subSampled[-trainSamples, ]
# we will also have a validation set from the original data
valSamples <- sample(1:dim(pdata)[1], size = dim(pdata)[1]/50, replace = FALSE)
validation <- pdata[valSamples, ]
# check the number of minority class - should be around 145:72
sum(train$click == 1)
```

```
## [1] 140
```

```r
sum(test$click == 1)
```

```
## [1] 77
```

```r
sum(validation$click == 1)
```

```
## [1] 4
```


Random Forest
--------------

We try to build a predictive model using a random forest, with parameters set to take 10 samples from each class per iteration to make sure we get a balanced result. 


```r
suppressPackageStartupMessages(library(randomForest))
table(train$click)
```

```
## 
##    0    1 
## 1451  140
```

```r
set.seed(1234)
rf.model1 <- randomForest(click ~ ., data = train, importance = TRUE, prox = TRUE, 
    strata = train$click, sampsize = c(10, 10))
rf.model1
```

```
## 
## Call:
##  randomForest(formula = click ~ ., data = train, importance = TRUE,      prox = TRUE, strata = train$click, sampsize = c(10, 10)) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 3
## 
##         OOB estimate of  error rate: 37.96%
## Confusion matrix:
##     0   1 class.error
## 0 918 533      0.3673
## 1  71  69      0.5071
```


The class error rate for click = 1 is 35.0% - which is not so great, but at least we are getting more than half of them right. 

Now let's see how much accuracy we get on the test set. 


```r
# make prediction from the test set
test.pred1 <- predict(rf.model1, test[, -12])
# compare it to the actual outcome
confusionMatrix <- table(observed = test$click, predicted = test.pred1)
confusionMatrix
```

```
##         predicted
## observed   0   1
##        0 434 285
##        1  28  49
```

```r
# class error rate of click=1
confusionMatrix[2, 1]/sum(confusionMatrix[2, ])
```

```
## [1] 0.3636
```


The class error rate is 29.9% - a bit better.

Now let's make prediction with validation set


```r
# make prediction from the validation set
validation.pred1 <- predict(rf.model1, validation[, -12])
# compare it to the actual outcome
confusionMatrix <- table(observed = validation$click, predicted = validation.pred1)
confusionMatrix
```

```
##         predicted
## observed    0    1
##        0 9886 5768
##        1    1    3
```

```r
# class error rate of click=1
confusionMatrix[2, 1]/sum(confusionMatrix[2, ])
```

```
## [1] 0.25
```


The class error rate is 25.0% - we called 3 out of 4 correctly.
 

```r
result <- importance(rf.model1, )[, "MeanDecreaseAccuracy"]
importance(rf.model1)[order(result, decreasing = TRUE), ]
```

```
##                0       1 MeanDecreaseAccuracy MeanDecreaseGini
## views     3.2771  4.4915               4.1200        1.6590082
## referrals 3.0914  1.3381               3.4903        1.4041509
## srcs      1.3740  2.7841               1.8055        1.0471955
## b_id      1.0697  1.8514               1.4313        0.6454228
## seller    0.7796  2.8447               1.3951        1.2557797
## v_id      1.2974 -0.2263               1.3550        0.2444988
## state     1.2817 -1.3130               1.2113        0.3107043
## country   1.0010  0.0000               1.0010        0.0002727
## hour      0.9406 -0.7737               0.7402        1.4177433
## t_id      0.3015 -2.5240              -0.1075        1.0677502
## wday      0.0538 -0.9365              -0.2385        0.8120070
```


The result rank the variable's importance by how much each contribute to reduce prediction error. However, the differences are pretty small, so I am not confident that this is a robust result.

Improving the model
-------------------

Wenjia points out that two variables that show high importance, seller and b_id, happen to contain a lot of NULL values. These NULL values may be artificially raising the apparent importance of those variables.

### Seller variable

Let's examine "seller" variable.

```r
# 94129 null values in 'seller' variable were re-coded with 3000
table(pdata$seller[pdata$seller == 3000], pdata$v_id[pdata$seller == 3000])
```

```
##       
##            3
##   3000 94129
```

```r
length(pdata$v_id[pdata$v_id == 3])
```

```
## [1] 94129
```


It looks none of the v_id==3 conains valid seller id. 
So what happens if we remove v_id==3?


```r
# New random forest without v_id==3
rf.model2 <- randomForest(click ~ ., data = train[train$v_id != 3, ], importance = TRUE, 
    prox = TRUE, strata = train$click[train$v_id != 3], sampsize = c(10, 10))
rf.model2
```

```
## 
## Call:
##  randomForest(formula = click ~ ., data = train[train$v_id !=      3, ], importance = TRUE, prox = TRUE, strata = train$click[train$v_id !=      3], sampsize = c(10, 10)) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 3
## 
##         OOB estimate of  error rate: 39.82%
## Confusion matrix:
##     0   1 class.error
## 0 784 505      0.3918
## 1  58  67      0.4640
```

```r
result <- importance(rf.model2, )[, "MeanDecreaseAccuracy"]
importance(rf.model2)[order(result, decreasing = TRUE), ]
```

```
##                  0       1 MeanDecreaseAccuracy MeanDecreaseGini
## srcs       3.79019  4.6311              4.52709           1.3089
## views      2.98890  2.8276              3.58901           1.7262
## referrals  1.75569  2.4379              2.36092           1.5154
## b_id       1.54055  2.6309              2.24424           0.5096
## seller     1.44731  2.5090              1.99971           1.4115
## state      1.95871 -1.4849              1.93269           0.2739
## wday       1.40661 -1.6918              0.85456           0.7444
## v_id       0.02666  0.9979              0.17227           0.1620
## t_id       0.29105 -2.0345              0.02757           0.8111
## country    0.00000  0.0000              0.00000           0.0000
## hour      -0.07618 -1.2618             -0.36988           1.4282
```


The importance of "seller" variable drops significantly once the null values are removed. For this reason, we can probably ignore this variable altogether. 


```r
# New random forest without seller variable
noseller <- train
noseller$seller <- NULL
rf.model3 <- randomForest(click ~ ., data = noseller, importance = TRUE, prox = TRUE, 
    strata = noseller$click, sampsize = c(10, 10))
rf.model3
```

```
## 
## Call:
##  randomForest(formula = click ~ ., data = noseller, importance = TRUE,      prox = TRUE, strata = noseller$click, sampsize = c(10, 10)) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 3
## 
##         OOB estimate of  error rate: 39.85%
## Confusion matrix:
##     0   1 class.error
## 0 883 568      0.3915
## 1  66  74      0.4714
```

```r
result <- importance(rf.model3, )[, "MeanDecreaseAccuracy"]
importance(rf.model3)[order(result, decreasing = TRUE), ]
```

```
##                0        1 MeanDecreaseAccuracy MeanDecreaseGini
## referrals 4.0099  3.84550               4.9054         1.626035
## views     2.8819  3.09193               3.5536         1.892680
## srcs      2.9171  0.93647               3.1836         1.547856
## v_id      1.3760 -1.29107               1.2738         0.350930
## state     1.1976 -2.69347               0.9432         0.301558
## wday      0.6662 -0.05231               0.6020         0.825901
## b_id      0.2783  1.49371               0.5828         0.688996
## t_id      0.5038 -1.80794               0.2509         1.049953
## country   0.0000  0.00000               0.0000         0.002945
## hour      0.4273 -2.21424              -0.1837         1.568812
```


### b_id variable

Let's now examine "b_id" variable. 546712 values "//N" variable were re-coded with 0.


```r
# how many clicks does that class contain?
table(pdata$click[pdata$b_id == 0])
```

```
## 
##      0      1 
## 546580    132
```

```r
# how many clicks do all other classes contain?
table(pdata$click[pdata$b_id != 0])
```

```
## 
##      0      1 
## 236145     85
```


It turned out Null value is a majority class for this variable. So what happens if we remove it?


```r
# New random forest without null values in b_id
nonulls <- noseller[noseller$b_id != 0, ]
rf.model4 <- randomForest(click ~ ., data = nonulls, importance = TRUE, prox = TRUE, 
    strata = nonulls$click, sampsize = c(10, 10))
rf.model4
```

```
## 
## Call:
##  randomForest(formula = click ~ ., data = nonulls, importance = TRUE,      prox = TRUE, strata = nonulls$click, sampsize = c(10, 10)) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 3
## 
##         OOB estimate of  error rate: 40.88%
## Confusion matrix:
##     0   1 class.error
## 0 262 167      0.3893
## 1  28  20      0.5833
```

```r
result <- importance(rf.model4, )[, "MeanDecreaseAccuracy"]
importance(rf.model4)[order(result, decreasing = TRUE), ]
```

```
##                 0       1 MeanDecreaseAccuracy MeanDecreaseGini
## b_id       2.8195  1.2521              2.99471         1.715363
## srcs       1.6089  3.6262              2.34689         1.204936
## hour       1.3956  1.0054              1.56410         1.349173
## wday       0.3277  0.8371              0.54904         0.753447
## referrals -0.7971  3.5902              0.05585         1.498934
## country   -0.4473  0.0000             -0.44730         0.004333
## t_id      -0.2942 -1.1810             -0.55761         1.079696
## v_id      -0.2082 -2.1696             -0.63077         0.422890
## views     -0.8187  1.2141             -0.63860         1.547472
## state     -1.3208 -1.9639             -1.69868         0.375755
```


b_id still remains high, and our class error rate worsened noticeably. So it is probably not good idea to remove the records with null values. We should keep those records, but we shouldn't use this variable because of the class imbalance. 


```r
# New random forest without b_id
noBID <- noseller
noBID$b_id <- NULL
rf.model5 <- randomForest(click ~ ., data = noBID, importance = TRUE, prox = TRUE, 
    strata = noBID$click, sampsize = c(10, 10))
rf.model5
```

```
## 
## Call:
##  randomForest(formula = click ~ ., data = noBID, importance = TRUE,      prox = TRUE, strata = noBID$click, sampsize = c(10, 10)) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 3
## 
##         OOB estimate of  error rate: 38.4%
## Confusion matrix:
##     0   1 class.error
## 0 912 539      0.3715
## 1  72  68      0.5143
```

```r
result <- importance(rf.model5, )[, "MeanDecreaseAccuracy"]
importance(rf.model5)[order(result, decreasing = TRUE), ]
```

```
##                  0       1 MeanDecreaseAccuracy MeanDecreaseGini
## srcs       3.68128  1.7910            4.137e+00         1.575414
## v_id       3.35143 -2.9727            3.076e+00         0.386244
## views      2.37803  3.0564            2.931e+00         2.083754
## referrals  1.29726  5.5803            2.510e+00         1.950164
## country    1.38943  0.0000            1.390e+00         0.003053
## wday      -0.17600  0.5731           -4.574e-05         0.878060
## hour       0.06016 -0.7144           -1.260e-01         1.621165
## t_id      -0.96362 -1.1880           -1.181e+00         1.121981
## state     -1.92901 -1.8976           -2.267e+00         0.269537
```


The important variables are now: views, t_id, srcs, referrals, and state. This seems to make more intuitive sense.

Now let's see how much accuracy we get on the test set. 


```r
# drop seller, b_id from the test set
test <- test[, !(names(test) %in% c("seller", "b_id"))]
# make prediction from the test set
test.pred5 <- predict(rf.model5, test[, -10])
# compare it to the actual outcome
confusionMatrix <- table(observed = test$click, predicted = test.pred5)
confusionMatrix
```

```
##         predicted
## observed   0   1
##        0 454 265
##        1  32  45
```

```r
# class error rate of click=1
confusionMatrix[2, 1]/sum(confusionMatrix[2, ])
```

```
## [1] 0.4156
```


Now let's make prediction with validation set


```r
# drop seller, b_id from the validation set
validation <- validation[, !(names(validation) %in% c("seller", "b_id"))]
# make prediction from the validation set
validation.pred5 <- predict(rf.model5, validation[, -10])
# compare it to the actual outcome
confusionMatrix <- table(observed = validation$click, predicted = validation.pred5)
confusionMatrix
```

```
##         predicted
## observed    0    1
##        0 9920 5734
##        1    1    3
```

```r
# class error rate of click=1
confusionMatrix[2, 1]/sum(confusionMatrix[2, ])
```

```
## [1] 0.25
```


So we didn't lose our predictive power, either. 
