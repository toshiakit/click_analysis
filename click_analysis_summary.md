Summary - URL Click Analysis
============================

This analysis was performed as a Real World Problem assignment in "Introduction to Data Science" course on [Coursera](https://www.coursera.org/) 

### Who:
A Canadian marketing firm [media6degrees](http://m6d.com/) is engaged in online ad serving business.

### Problem:
The firm is interested to find out what historical non-identifiable metrics collected fro server logs are associated with increase in click rate of an ad. They would like to use such information to improve the effectiveness of ad targeting. The firm has done previous work with clustering algorithms to explore this problem, but it resulted in inconclusive findings.

### The desired output:
The discovery of a set of metrics or combination of such metrics that show correlation to a click.

### The input:
The firm provided a sanitized (anonymized) log data from servers that have served ads and tracked their performance on a per user basis. Each entry represents an instance of an ad served. The non-identifiable metrics includes:

- **Time stamp** (date and time) of when the ad was served. (string)
- **User interest**: clasification of ad viewer based on their interest (numeric)
- **URL** - encrypted string of the URL on the destination website where the ad was served. (string)
- **Referrer URL** - encripted string of the URL on a website where the ad viewer came from before arriving at the destination website where the ad was served. (string)
- **Country** where the ad was served to a viewer (string)
- **State** where the ad was served to a viewer (string)
- **Ad ID** - unique identifier of an ad that was served to a viewer (numeric)
- **Vendor ID** - the vendors act as the wholesaler of destination websites where the ads are being shown (numeric)
- **Seller ID** - the seller is an intermediary that connects the advertisers to vendors (numeric)
- **Click** - a binary outcome - whether the ad was clicked "1" or not "0" (numeric)

There are 782942 records in the dataset, but only 217 was clicked. This represents a binary classification problem with a vastly imbalanced outcome classes, because only 0.28% of the ads are clicked.
The ads are distributed from **Origin** --> **Seller** --> **Vendor** --> **Destination**.

### Solution

Because the dataset is mainly comprised of categorical data rather than numerical data, I chose Random Forest algorithm to analyze the data.
- Random Forest works well with categorical as well as numeric data
- Suitable for binary classification problem
- Can produce the variable importance measure that can be used to deliver the answers to the question the firm seeks.
- The resulting model can be used for prediction.

One major problem I had with the dataset was the imbalance of the outcome - only 217 ad views out of 782942 resulted in clicks, so a classifier that simply predicts every case as no click will still achieve 99.97% accuracy rate. This problem was addressed this way.

- First downsampled the majority class (no click) to 2170 cases so that the ratio of majority class and minority class (clicked) is 10:1.
- This is further split randomly into two subsets - the training set that contains 2/3 of the downsampled data and the test set that contain the remaining 1/3.
- 1/50th of the original dataset, without downsampling, was randomly selected and set aside as a validation set. This contained 4 cases that resulted in a click.
- Random Forest model was first trained on the training set using a stratified sampling technique where equal propotion of samples are taken from both classes in growing individual trees.
- Use class classification error rate for the minority class only for evaluation, rather than using the overall misclassification error rate. This means the models that detects the cases that resulted in a click accurately are chosen over those that produced better overall prediction accuracy.

Another key consideration was feature selection. The following transformations were performed on the dataset.

- **Timestamp** was converted into the day of the week variable and 24-hour variable
-	**User interest** variable contained such a large number of null values that it became a dominant majority class. This variable was dropped for this reason.
-	**URL** variable was convered into frequency of the views for a particular URL. The more views you have, the more chance you get clicks.
-	**Referrer URL** was also converted into frequency metric. It represents how active a given referrer is as a traffic source.
- **URLs and Referrer URLs** were mapped to determine how many referrers each URL has. This metric represents a number of incoming links. It represent the popularity of a given page. This is similar to how PageRank agorithm caculates the weight of a given webpage.
- **Seller ID** was dropped because it also had a very large number of null values.
- Missing values in the variables were also cleaned up.

### Results

While the goal was not provide a predictive model for this problem, I needed some way to run a sanity check on the way I was dealing with the class imbalance issue, and I evaluated the class classification error for clicked class.
- **Training set** "clicked" class error rate: 35.0% - 91 out of 140 "clicked" cases were correctly classified.
- **Test set** "clicked" class error rate: 33.8% - 51 out of 77 "clicked" cases were correctly classified.
- **Validation set** "clicked" class error rate: 25.0% - 3 out of 4 "clicked" cases were correctly classified.

The error rates are probably too high for a predictive model, but it shows that the model does produce meaningful prediction to some extent, even after I downsampled quite a large portion of the original data and apply stratified sampling technique. Based on this model, I then produced the variable importance measure. Here is the first 5 variables ranked based on Mean Decrease Accuracy metric.

1. **Number of referral sources a URL has where the ad appeared** - indicating how popular and well linked that page is. *Mean Decrease Accuracy = 3.8784*
2. **Number referrals the source sends** - indicating how actively does the source page sent out traffic.*Mean Decrease Accuracy = 2.0870*
3. **Page views of the URL where the ad appeared** - the more page views you get, the more chance to get clicks. *Mean Decrease Accuracy = 1.9245*
4. **Ad ID** - which ad it was. *Mean Decrease Accuracy = 0.9382*
5. **State** - how people respond to ads many be different geographically. *Mean Decrease Accuracy = 0.8159*

One thing to keep in mind is that the score difference among the top 3 variables are pretty small, and they are all intercorrelated by nature - the more referral sources you have, the more referrals you get, and more page views you will have. Therefore the ranking of those variable could potentially change depending on how the data is sampled.

### Recommendations

- This analysis was performed on a data set that contain only 217 cases of clicks. That limits the accuracy of the model we can develop even though we had 782942 records. Therefore much larger dataset is needed for improving the analysis.
- Some of the variables had a very large number of null values became unusable. However, User Interest variable had shown a sign that it could be a valuable predictor when I tried it without null values. Unfortunately this resulted in poorer predictive performance, because I had to discard large chunk of otherwise meaningful data and further reduced the cases of clicks. Therefore it is probably important to reduce the null values for User Interest variable.
- How people access the websites also affects the probability of clicks. For example, mobile users are less likely to click on ads. Therefore adding the device variable should be part of the dataset. 

### Special Thanks

- Nandita Ramesh for sharing her exploratory analysis in the study group. Her idea for converting the URL and Referrers variables into quantitative metrics were vital in in applying Random Forest algorithm.
- Wenjia Zhao for also sharing her cluster analysis in the study group and critiquing my analysis. She identified the issues caused by the large quantity of null values in Seller ID and User Interest variables.