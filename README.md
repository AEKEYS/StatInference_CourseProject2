# Assessing Tooth Growth in Guinea Pigs

*Synopsis:* This report compares the relative effects on guinea pig tooth growth of administering vitamin C via different supplements and at various dose levels. It was conducted as part of a class assessment for the Johns Hopkins University's ["Statistical Inference"](https://www.coursera.org/course/statinference) offered through Coursera.

####Loading Data, Initial Exploration
Data for this report are based on [a study published in 1947](http://jn.nutrition.org/content/33/5/491.full.pdf) that recorded the tooth length for each of 10 guinea pigs that had been administered vitamin C via orange juice or ascorbic acid and at three dose levels (0.5, 1, and 2 mg).  Data were loaded from R's "datasets" library.


```r
#Load the ToothGrowth data and perform some basic exploratory data analyses 
library(datasets)
data(ToothGrowth)
# Give levels more descriptive name
levels(ToothGrowth$supp)<-c("Orange Juice", "Ascorbic Acid")
ToothGrowth$dose <- as.factor(ToothGrowth$dose)
```

We begin by examining a simple descriptive summary of the data frame. This helps us understand the distribution of tooth length among all 60 guinea pigs and that equal numbers of guinea pigs were administered the different supplement-dose combinations.


```r
summary(ToothGrowth)
```

```
##       len                   supp     dose   
##  Min.   : 4.20   Orange Juice :30   0.5:20  
##  1st Qu.:13.07   Ascorbic Acid:30   1  :20  
##  Median :19.25                      2  :20  
##  Mean   :18.81                              
##  3rd Qu.:25.27                              
##  Max.   :33.90
```

Next, we plot the distribution of tooth growth but delineate by the type of supplement. Although inconclusive, a visual inspection seems to suggest that more guinea pigs experienced tooth growth when receiving orange juice than those receiving ascorbic acid.


```r
library(ggplot2)

a <- ggplot(ToothGrowth,aes(x=len,fill=supp))
a + geom_histogram(aes(y=..density..),colour="black",binwidth=4) +
    labs(title="Guinea Pig Tooth Growth by Supplement")+
    labs(x="Tooth Length (microns)",y="Percent of Guinea Pigs")
```

<img src="./StatInferencePart02_files/figure-html/unnamed-chunk-3-1.png" title="" alt="" width="500px" />

####Basic Data Summary
Our initial exploration suggested that orange juice may have corresponded with greater tooth growth than ascorbic acid.  We would like to investigate further by examining guinea pig tooth growth not only by supplement but also by dose level.

First, we reshape the data into a tidy data frame.


```r
#Provide a basic summary of the data.
library(reshape2); library(plyr)
ToothGrowth$lenGroup <- cut(ToothGrowth$len,quantile(ToothGrowth$len))
ToothGrowth$lenGroup[1]<-"(4.2,13.1]"
SixGroups <- data.frame(split(ToothGrowth$len,list(ToothGrowth$supp,ToothGrowth$dose)))
molten <- melt(SixGroups)
```

```
## No id variables; using all as measure variables
```

Next, we summarize the distributions of guinea pig tooth growth for each of the six supplement-dose combinations.  The box plots show the minimum, first quartile, median, third quartile, and maximum tooth growth for each combination.


```r
g <- ggplot(molten, aes(variable,value))
g+geom_boxplot()+
    labs(title="Guinea Pig Tooth Growth by Supplement & Dose")+
    labs(x="Supplement & Dose (mg)",y="Tooth length (microns)")
```

<img src="./StatInferencePart02_files/figure-html/unnamed-chunk-5-1.png" title="" alt="" width="500px" />

Finally, we calculate means and sample standard deviations for each supplement-dose combination.


```r
ds <- ddply(molten,.(variable),summarise,mean=mean(value),sd=sd(value)) #data summary
ds
```

```
##            variable  mean       sd
## 1  Orange.Juice.0.5 13.23 4.459709
## 2 Ascorbic.Acid.0.5  7.98 2.746634
## 3    Orange.Juice.1 22.70 3.910953
## 4   Ascorbic.Acid.1 16.77 2.515309
## 5    Orange.Juice.2 26.06 2.655058
## 6   Ascorbic.Acid.2 26.14 4.797731
```

####Hypothesis Tests To Compare Tooth Growth by Supplement & Dose
We would like to test whether the difference between tooth growth at each supplement-dose combination is larger than zero, which would imply that one supplement corresponded with greater tooth growth than the other for the same dose level.  

To test this, we conduct a two-sided test of our null hypothesis: tooth growth is the same at each dose level.  In other words, we would like to see whether the difference in the means between each supplement at each dose level is non-zero.  We will set the test to reject the null hypothesis if we can limit type 1 errors (false positives) to 5%. 


```r
# reject null if t-statistic larger than
qt(.975,10-1)
```

```
## [1] 2.262157
```

```r
# reject the null, the difference is non-zero. OJ grows bigger teeth.
t.test(SixGroups$Orange.Juice.0.5,SixGroups$Ascorbic.Acid.0.5)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  SixGroups$Orange.Juice.0.5 and SixGroups$Ascorbic.Acid.0.5
## t = 3.1697, df = 14.969, p-value = 0.006359
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  1.719057 8.780943
## sample estimates:
## mean of x mean of y 
##     13.23      7.98
```

```r
# reject the null, the difference is non-zero. OJ grows bigger teeth.
t.test(SixGroups$Orange.Juice.1,SixGroups$Ascorbic.Acid.1)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  SixGroups$Orange.Juice.1 and SixGroups$Ascorbic.Acid.1
## t = 4.0328, df = 15.358, p-value = 0.001038
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  2.802148 9.057852
## sample estimates:
## mean of x mean of y 
##     22.70     16.77
```

```r
# failed to reject the null, the difference might be zero.
t.test(SixGroups$Orange.Juice.2,SixGroups$Ascorbic.Acid.2)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  SixGroups$Orange.Juice.2 and SixGroups$Ascorbic.Acid.2
## t = -0.0461, df = 14.04, p-value = 0.9639
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -3.79807  3.63807
## sample estimates:
## mean of x mean of y 
##     26.06     26.14
```

####Conclusions & Assumptions
Based on these tests, we can reject the null hypothesis that the difference between tooth growth in guinea pigs administered orange juice versus those administered ascorbic acid at dose levels 0.5 and 1 mg is not zero.  This implies that orange juice, at least at these dose levels, corresponded with guinea pigs growing larger teeth.

We fail to reject the null hypothesis in the case of guinea pigs administered a dose of 2 mg.  We can see this both because of the small absolute value of the t-statistic but also because the confidence interval includes zero.  This implies that there might not be any difference between orange juice and ascorbic acid at this dose level.

Our analysis assumes that guinea pig tooth growth follows a Gossett's distribution, which is difficult to say since we have only 10 guinea pigs per supplement-dose combination. We assume samples were independently drawn from the same population. Any extension of our claims about the correlation of tooth growth and orange juice consumption at low doses to a larger population of guinea pigs assumes that the sample is a good approximation of that population.
