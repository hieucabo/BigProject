# 🚢 Titanic - Machine Learning from Disaster

*** 

## 📅 **Data source**

I get the data from **Kaggle**. 

The data has been split into two groups:

- training set (train.csv)
- test set (test.csv)

The training set should be used to build your machine learning models.

The test set should be used to see how well your model performs on unseen data. 
For each passenger in the test set, use the model you trained to predict whether or not they survived the sinking of the Titanic.

- [Titanic - Machine Learning from Disaster](https://www.kaggle.com/competitions/titanic/data)

***

## 🏁 **Project objectives**

- Use machine learning to create a model that predicts which passengers survived the Titanic shipwreck. 

## 🧐 **What I learned**

I'm using R Studio for this project and I learned so much 🤯

- Dealing with missing data. 
- Use ````mice```` package for fill missing data. 
- Use ````random forest```` for predicting which passengers survived.

## 👟 **Let's get started**

### Load the necessary library 

````r
library(dplyr) #manipulation
library(ggplot2) #visualization
library(tidyverse) #manipulation
library(skimr) #summarystatistics
library(mice) #datamissingfill
library(randomForest) #dataprediction
````

### Import the dataset

````r
train <- read.csv("~/RStudio File/Titanic/train.csv")
test <- read.csv("~/RStudio File/Titanic/test.csv") 
gender <- read.csv("~/RStudio File/Titanic/gender_submission.csv")
````
Let's combine train and test into the same data frame, however the survivied variable on test data will be NA. 

````r
test$Survived <- NA 
full <- bind_rows(train,test)
````

### Exploring data 

````r
str(full)
````

````r
'data.frame':	1309 obs. of  12 variables:
 $ PassengerId: int  1 2 3 4 5 6 7 8 9 10 ...
 $ Survived   : int  0 1 1 1 0 0 0 0 1 1 ...
 $ Pclass     : int  3 1 3 1 3 3 1 3 3 2 ...
 $ Name       : chr  "Braund, Mr. Owen Harris" "Cumings, Mrs. John Bradley (Florence Briggs Thayer)" "Heikkinen, Miss. Laina" "Futrelle, Mrs. Jacques Heath (Lily May Peel)" ...
 $ Sex        : chr  "male" "female" "female" "female" ...
 $ Age        : num  22 38 26 35 35 NA 54 2 27 14 ...
 $ SibSp      : int  1 1 0 1 0 0 0 3 0 1 ...
 $ Parch      : int  0 0 0 0 0 0 0 1 2 0 ...
 $ Ticket     : chr  "A/5 21171" "PC 17599" "STON/O2. 3101282" "113803" ...
 $ Fare       : num  7.25 71.28 7.92 53.1 8.05 ...
 $ Cabin      : chr  "" "C85" "" "C123" ...
 $ Embarked   : chr  "S" "C" "S" "S" ...
 ````
 
- PassengerID: unique ID of each passenger
- Survived: Survived(1) or died (0)
- Pclass: Passenger’s class 
- Name: Passenger’s name
- Sex: Passenger’s sex
- Age: Passenger’s age 
- SibSp: Number of siblings/spouses onboard
- Parch: Number of parents/children onboard
- Ticket: Ticket number 
- Fare: Ticket price
- Cabin: Cabin
- Embarked: Port of embarkation

### Data Preparation and Cleaning

````r
colSums(is.na(train))
````

````r
PassengerId    Survived      Pclass        Name         Sex         Age       SibSp       Parch 
          0         418           0           0           0         263           0           0 
     Ticket        Fare       Cabin    Embarked 
          0           1           0           0
````

- 418 missing values in Survived column are those in the test data frame.

- There are 177 values missing about the ages of the passengers. I will try to fill this in later.

- Besides, maybe a value is not missing but named as ‘ ‘ (nothing). So I should check on it.

````r
sapply(full, function(x) sum(x == ''))
````

````r
PassengerId    Survived      Pclass        Name         Sex         Age       SibSp       Parch 
          0          NA           0           0           0          NA           0           0 
     Ticket        Fare       Cabin    Embarked 
          0          NA        1014           2
````

Age is NA because it’s the num data type. We noticed that there are 1014 non-value for Cabin and 2 for Embarked. 
Our data only has 1309 rows. 
So Cabin variable should be dropped for too much data missing. Let’s drop the Cabin column.

### Data Analyze

- Please note that all the analysts here used only the observation who already had the value on Survived observation. 

- Any additional column based on other observations still included those passengers on the test data frame.

**TITLE**

````r
full <- full %>% 
  mutate(title = gsub('(.*, |\\..*)','',Name))

full %>% 
  group_by(title) %>% 
  summarise(count = n()) %>% 
  arrange(-count)
````

````r
title        count
   <chr>        <int>
 1 Mr             757
 2 Miss           260
 3 Mrs            197
 4 Master          61
 5 Dr               8
 6 Rev              8
 7 Col              4
 8 Major            2
 9 Mlle             2
10 Ms               2
11 Capt             1
12 Don              1
13 Dona             1
14 Jonkheer         1
15 Lady             1
16 Mme              1
17 Sir              1
18 the Countess     1  
````

- There are many rare titles such as Capt, Col, Don,… 
- Mlle should be known as Miss but written in French.  
- The same goes for Mme as Mrs. 
- Ms can be written as Mrs. 
- Master is a title for under 18 boys.

````r
full <- full %>% 
  mutate(title = case_when(
    title == "Mlle" ~ "Miss",
    title == "Mme" ~ "Mrs",
    title == "Ms" ~ "Mrs",
    title == "Miss" ~ "Miss",
    title == "Mrs" ~ "Mrs",
    title == "Mr" ~ "Mr",
    title == "Master" ~ "Master",
    .default = "Rare_Title"
  ))
````

````r
title      count
  <chr>      <int>
1 Mr           757
2 Miss         262
3 Mrs          200
4 Master        61
5 Rare_Title    29
````

- Mr: Married or unmarried men
- Master: Men under 18
- Mrs: A married women
- Miss: Unmarried mowen

````r
full %>% 
  filter(!is.na(Survived)) %>% 
  group_by(title) %>% 
  summarise(survived_rate = sum(Survived)/n()*100) %>% 
  arrange(-survived_rate)
````

````r
title      survived_rate
  <chr>              <dbl>
1 Mrs                 79.5
2 Miss                70.1
3 Master              57.5
4 Rare_Title          34.8
5 Mr                  15.7
````

- Well, for a man over 18 survival rate is only 15.7, a young man is 57.5, married women is 79.5% and unmarried women is 70.1%. 
- When you have a rare title, your survival rate is 34.8%. Not so low.

**CLASS**

````r
class_survived_rate <- full %>%
  filter(!is.na(Survived)) %>% 
  group_by(Pclass,Survived) %>% 
  summarise(num_pp = n())

ggplot(class_survived_rate,aes(x=as.character(Pclass),y=num_pp,fill = as.character(Survived))) + 
  geom_col(position=position_dodge()) +
  theme(axis.title.x = element_blank(),
        axis.title.y = element_blank(),
        legend.title = element_blank()) +
  scale_fill_discrete(labels = c('Died','Survived'))
````

Because Pclass and Survived is an integer, I have to use **as.character()** to change to the character for graphing.  

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222917033-cfd0aa30-5d11-4ea5-93e9-0c9f8cfa5e30.png">
</p>

You should know that there were 3 classes on Titanic: 

- First class (the most expensive)
- Second class (cheaper than first class but still had more comfort compared to third class)
- Third class (had the lowest price, lived on the bottom decks of the ship). Most people in class 3 were dead.

````r
class_survived_rate %>% 
  group_by(Pclass) %>% 
  summarise(survive_rate = sum(num_pp*Survived)/sum(num_pp))
````

````r
Pclass survive_rate
   <int>        <dbl>
1      1        0.630
2      2        0.473
3      3        0.242
````

The higher class, the better chances of survival. 

- 1st class: 63%
- 2nd class: 47%
- 3rd class: 24%

**FAMILY**

- SibSp is the number of siblings; and spouses. Which is the number of brothers, sisters and a wife or husband.
- Parch is the number of family members, including children.
- So the family size is the total number of SibSp and Parch plus 1 (the passenger)

````r
full <- full %>% 
  mutate(family_size = SibSp + Parch + 1)
````

````r
full %>% 
  group_by(family_size) %>% 
  summarise(n())
````

````r
family_size `n()`
        <dbl> <int>
1           1   790
2           2   235
3           3   159
4           4    43
5           5    22
6           6    25
7           7    16
8           8     8
9          11    11
````

There were so many passengers who went by themself and lots of families with more than 4 members. 

````r
ggplot(family_size_rate,aes(x=family_size,y=num_pp,fill = as.character(Survived))) + 
  geom_col(position=position_dodge()) +
  scale_x_continuous(labels=0:11,breaks=0:11) +
  theme(axis.title.x = element_blank(),
        axis.title.y = element_blank(),
        legend.title = element_blank()) +
  scale_fill_discrete(labels = c('Died','Survived'))
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222917643-89a3c746-16f7-45ab-b1d6-b9ef582c1686.png">
</p>

- Passengers who went with more than 8 family members had no one survive.
- Most survival went alone or had less than 5 members.

````r
family_size_rate %>% 
  group_by(family_size) %>% 
  summarise(survive_rate = sum(num_pp*Survived)/sum(num_pp))
````

````r
family_size survive_rate
        <dbl>        <dbl>
1           1        0.304
2           2        0.553
3           3        0.578
4           4        0.724
5           5        0.2  
6           6        0.136
7           7        0.333
8           8        0    
9          11        0
````

- For those who had less than 5 family members, their survival rate increased when the family size increased.
- In contrast, those who had 5 and higher had a very low chance of survival.
- The passenger who was in a family with 8 members and 11 members died with their family.

### Filling missing data

There are missing values in the Age column, with about 20%. If we can fill this, it will be a big help for analysts. 
There are many methods for filling in missing data, but I will use **multiple imputation** using the **mice** package. 

This method fills missing values with values from other observations that are similar 
to the missing observation. The similarity can be determined based on other variables in the dataset.

There are many features that can help us to fill the missing value of age: 

- Title
- Pclass
- Family size
- Sex

Let’s group by Title by those having Age first.

````r
title_age <- full %>% 
  filter(!is.na(Age))

ggplot(title_age,aes(y=title,x=Age,fill=title)) + 
  geom_boxplot() + 
  theme(
    axis.title.x = element_blank(),
    axis.title.y = element_blank(),
    legend.title = element_blank())
  )  
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222917782-65af498f-a402-4861-aaf2-ab9c6ad17b2b.png">
</p>

The Master title should range from 0 to 18, however. The max age for this title is only 14.5. While the Mr title should be 18 and over but its min value is 11.

````r
title_age %>% 
  filter(Age < 18, title == "Mr") %>% 
  select(Name, Sex, Age, SibSp, Parch)
````

````r
Name  Sex Age SibSp Parch
1           Ford, Mr. William Neal male  16     1     3
2              Osen, Mr. Olaf Elon male  16     0     0
3                  Calic, Mr. Jovo male  17     0     0
4   Sunderland, Mr. Victor Francis male  16     0     0
5        Panula, Mr. Ernesti Arvid male  16     4     1
6        de Pelsmaeker, Mr. Alfons male  16     0     0
7  Vander Planke, Mr. Leo Edmondus male  16     2     0
8               Elias, Mr. Tannous male  15     1     1
...
22             Culumovic, Mr. Jeso male  17     0     0
23      Svensson, Mr. Johan Cervin male  14     0     0
24                 Dika, Mr. Mirko male  17     0     0
25              Davies, Mr. Joseph male  17     2     0
26       Deacon, Mr. Percy William male  17     0     0
27     Sweet, Mr. George Frederick male  14     0     0
28               Pokrnic, Mr. Mate male  17     0     0
29          Carrau, Mr. Jose Pedro male  17     0     0
````

I only showed 16 rows over 29 rows. There are different title called in each country, 
so in order to make the filling process more precise, I will use the title based on age.

````r
	title_age <- title_age %>% 
  mutate(title = case_when(
    Age < 18 & title == "Mr" ~ "Master",
    .default = title
  ))
  
  title_age %>% 
  group_by(title) %>% 
  summarise(min=min(Age),
            max=max(Age),
            median=median(Age))
````

````r
title        min   max median
  <chr>      <dbl> <dbl>  <dbl>
1 Master      0.33    17    9  
2 Miss        0.17    63   22  
3 Mr         18       80   30  
4 Mrs        14       76   35  
5 Rare_Title 23       70   47.5
````

This looks good now. Let’s have a look at another feature. 

````r
ggplot(title_age,aes(x=Age, y=as.character(Pclass),fill=as.character(Pclass))) +
  geom_boxplot() +
  theme(
    axis.title.x = element_blank(),
    axis.title.y = element_blank(),
    legend.title = element_blank())
)
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222917908-e5b3ccec-1dc4-48af-8228-9c286f55ff1c.png">
</p>

The passenger who had higher Age had a higher chance of buying an expensive ticket (first class), this make sense.

````r
ggplot(title_age,aes(x=Age, y=family_size,fill=as.character(family_size))) +
  geom_boxplot() +
  scale_y_continuous(labels=0:11,breaks=0:11) + 
  theme(
    axis.title.x = element_blank(),
    axis.title.y = element_blank(),
    legend.title = element_blank())
)
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222918077-80545d7d-3034-4230-9de0-1aea328384df.png">
</p>

The passenger who went alone or with only 1 family member had the highest median Age. As the number of family member increase, 
the median age decrease, and they may had more children.

````r
ggplot(title_age,aes(x=Age, y=Sex,fill=Sex)) +
  geom_boxplot() +
  theme(
    axis.title.x = element_blank(),
    axis.title.y = element_blank(),
    legend.title = element_blank())
)
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222918147-c846ad51-7cd0-4d78-b92a-b4961dc85d54.png">
</p>

The median age of men is higher than women. Men also have more outliers. These features look nice. Let’s begin filling in the missing Age values.

````r
imp <- full %>% 
  select(Age, Sex, title, family_size, Pclass) %>% 
  mice()

imputed_data <- complete(imp,1)

````

This looks good, there are actually no big differences between imputed data and original data(without missing values). 
Let's add the imputed data to the original data.

````r
full$Age <- imputed_data$Age
colSums(is.na(full))
````

````r
PassengerId    Survived      Pclass        Name         Sex         Age       SibSp       Parch 
          0         418           0           0           0           0           0           0 
     Ticket        Fare    Embarked       title family_size 
          0           1           0           0           0
````

There is still 1 NA value in Fare observation because there’s only 1 so let's replace it with the median value base on the passenger PClass and his/her Embarked.

````r
full %>% 
  filter(!is.na(Fare)) %>% 
  group_by(Pclass, Embarked) %>% 
  summarise(median = median(Fare))
````

````r
   Pclass Embarked median
<int><chr><dbl>
 1      1 ""        80
 2      1 "C"       76.7
 3      1 "Q"       90
 4      1 "S"       52
 5      2 "C"       15.3
 6      2 "Q"       12.4
 7      2 "S"       15.4
 8      3 "C"        7.90
 9      3 "Q"        7.75
10      3 "S"        8.05
````

````r
full %>% 
  filter(is.na(Fare)) %>% 
  select(Sex, Pclass, Embarked)
````

````r
Sex Pclass Embarked
1 male      3        S
````

The passenger who had a NA value in Fare was third class and embarked from S, so his price should be 8.05$. 

````r
full <- full %>% 
  mutate(Fare = case_when(
    is.na(Fare) ~ 8.05,
    .default = Fare
  )  
  )

colSums(is.na(full))
````

Now all the missing values on the Titanic dataset are replaced now. 
Except for the Cabin observation because their missing values are 
more than 75% of the collected data.

### Prediction 

There are many methods for data prediction, such as:

- Random Forest
- Gradient Boosting
- Neural Networks

However, I choose **random forest** for this project. Because of its high accuracy and robustness to outliers. 

Let’s split the data into 2 groups, train and test data. 
However, because our based data set is already named train and test. So I will name them train_full and test_full.

**Split the dataset**

````r
train_full <- full[1:891,]
test_full <- full[892:1309, ]
````

````r
rf <- randomForest(factor(Survived) ~ Pclass + Age + Sex + title
                                      + family_size, data = train_full)

plot(rf, ylim=c(0,0.36)) +
legend('topright', colnames(rf$err.rate), col=1:3, fill=1:3)
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/222918604-824c8721-979c-478f-b263-45c6d934bb36.png">
</p>

The black line shows that the error rate falls by about 18%. It’s more accurate to predict deadly than predict survived.

**Predict on the test set**

````r
predictions <- predict(rf, newdata = test_full)
````

**Save the solution into CSV file**

````r
solution <- data.frame(PassengerID = test_full$PassengerId, Survived = predictions)

write.csv(solution, file = 'rf_mod_Solution.csv', row.names = F)
````

### Conclusion

- The submission scores only 0.77990 on the public leaderboard, I know it’s not a great score but to me, it’s such a challenge. 
Including all the work that I have never done before, 
**data cleaning, filling missing values using multiple imputations, predict missing data using the Random Forest method.**

- I really enjoy working with this problem, if you have any advice or suggestions, I’d really appreciate it.

### Credit

- https://www.kaggle.com/code/dr1t10/surviving-the-titanic-step-by-step-with-groups#6.-Predictions-and-submission-
- https://www.kaggle.com/code/mrisdal/exploring-survival-on-the-titanic/report#feature-engineering
- https://data.library.virginia.edu/getting-started-with-multiple-imputation-in-r/
