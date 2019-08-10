---
layout:     post
title:      Airbnb Insights - Final Project for DS Class
date:       2019-5-14 12:00:00
summary:    Part of the Final Exam for the IS 457 Introduction to Data Science class at UIUC, these are some insights I found in Airbnb data outside guided questions of the class. In short, I find that listings which specify childcare and recreational items have a higher price and rating, while listings mentioning smoking are constantly lower rated and less expensive. I will then discuss what  Airbnb could do with these insights.
categories: Projects
---

In the Spring of 2019 I was able to take IS 457 *Introduction to Data Science* at UIUC. It wasn't a required course and was geared for undergraduates, but I had only done data-sciencey work in self projects or self taught methods for research data analysis. It was also taught in R and I wanted to be able to say that I knew both R and Python, the two most used Data Science languages. Most of the work within the class was guided , but the some of final project did involve coming up with your own insights. I thought what I found was interesting, so I'm putting it up here.

We were given an data set of anonymized Airbnb users from Australia from 2009 to 2017. Unfortunately I don't know where it's from so I can't link it here. Each row of the dataset was an individual listing on Airbnb, with columns beings features of that listing. The features could be generalized into three categories; host identity, listing details, and user feedback.   

Within the guided final project, we had to find number of people accommodated based on property type, determined if being a "superhost" (a paid service for listing promotion) resulted in better ratings, and fit linear model to price vs. reviews per month. In short, smaller listings like apartments accommodated less people, superhosts had similar median ratings but a higher "floor"/less terrible ratings, and more expensive properties are minimally correlated with fewer reviews.

The last question section on the final was "generate your own Actionable insights". And that's what I want to talk about.

### Actionable Australian Amenity Amelioration

A big theme in this class was to "generate business actionable insights", meaning find something that can help your employer/client improve their business. Besides a nice introduction to buzzwords, this emphasized looking for relationships that can be exploited practically. With this in mind, I decided to look at the relationship between amenities and a listings price/rating. Amenities are desirable features of a place and are often upgradeable. You can't tell a townhouse in the country to move to the beach, but you can tell it get cable if it will increase their ratings.

The first step in making progress is to get the data in a format that is easy to deal with programmatically. The amenities data of a listing was originally stored as a general object, so I first turned it into a string. The string of amenities looked something like `{Bathroom, Cable, Wifi}`. So I needed to get rid of the punctuation to get separate words. Then I sorted the number of occurrences for the top words and decided to look at the 200 amenities that appear most often. The code for that is here:
```r  
airbnb_data$amenities=as.character(airbnb_data$amenities)
amn_words=unlist(strsplit(airbnb_data$amenities,"[[:punct:] ]"))
amn_words=amn_words[amn_words!=""]
top200_amn=names(sort(table(unlist(amn_words)),decreasing = T)[1:200])
cap_200=unlist(str_extract_all(top200_amn,"\\b[A-Z]\\w+"))
```

I now had the top 200 amenities in the data set. But I needed to know the average price and average rating of listings with each amenity listed. Previously in the project I had written a function called `getWordFreq` to get the number of times a word appears within a section of text. I used this to fill an ordered lists with the mean price/rating if that word occurred.
```r
for(i in 1:length(cap_200)){
  amn_prices[i]=mean(airbnb_data[getWordFreq(cap_200[i],airbnb_data$amenities)>0,]$price)
  amn_ratings[i]=mean(airbnb_data[getWordFreq(cap_200[i],airbnb_data$amenities)>0,]$review_scores_rating)
}
```

This gave me the data in a nice form I could plot. So I plotted amenity vs. price and amenity vs. rating. These are shown below.

![Amenity and Price](/_img/is457_price.png)

![Amenity and Rating](/_img/is457_rating.png)

A few things jump out initially. On the positive side, we see that having a Full Bedroom/Bath/Toilet are all highly correlated to a good rating and a high price. This indicates that people are willing to pay for not having to share a room or a bathroom with people they do not know. On the low side, smoking is low in both price and rating. Therefore, if you are a host and considering making your listing smoking friendly it may not be worth it.

Next I will plot both rating and price in the same plot. The features that Airbnb would want to promote are going to be in in the upper right portion of the next graph. These features are ones that people like (high rating) and are willing to pay for (higher price).

![Rating and Price](/_img/is457_price_rating.png)

The willingness to pay for privacy is evident by the cluster on the far right of the graph. But we also see a cluster of items associated with childcare in the top right of the graph. Words such as 'Children', 'Crib', "Changing", "Babysitter", and "Baby" are all associated with a high price and high rating. Words that have a high rating but low price are "Cat", "Dog", and "Breakfast". This would indicate people like having their pets brought along, but do not associate doing so with a higher price.

Using these results, a clear recommendation for Airbnb hosts would be to:
1. If possible, make sure the user has their own bedroom and bathroom.
2. Make the listing kid friendly, people are willing and happy to pay for convenience with kids.

Thanks for reading, hope you found it interesting. I don't think I'm going to make this code available, just in case the class uses the same final project again. Sorry about that.
