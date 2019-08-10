---
layout:     post
title:      Scraping Basketball Reference
date:       2017-3-14 12:00:00
summary:    The analytics movement has drastically changed the NBA in the past 20 years, going from slow play in the paint to fast play with lots of three pointers. But to justify strategic decisions you need access to the data. In this post I describe how I scraped NBA data from the HTML files accessible at basketball-reference.com
categories: Projects
---

The NBA has changed dramatically within the past 20 years. This change has been driven, at least in part, by leveraging data and mathematics to instruct strategic decisions. One example of a strategic decision is the emphasis on three point frequency. In basketball, shots behind a line of approximately 23 feet count for 3 points, while shots inside this line count for just 2 points. On average, a two point shot has about a 50 percent chance of going for an expected value of around 1 point. While a three point shot has a lower percentage of going in, around 35 percent, it has an higher expected value of 1.15 points. While this point is oversimplified, it is the basis behind the success of many recent NBA contenders such as the Houston Rockets and Golden State Warriors.

There many insights to be gained from data within basketball, as many contextual and numeric inputs influence a deceivingly simple question "Will you score more points than your opponent?". Before attempting to answer these questions, you need access to the data. Accessing the data can be done easily online, through sites like [Basketball Reference](https://www.basketball-reference.com/). However, getting it the data into a format that is easy to work within a programming language can be a challenge. Individual tables can be downloaded as `csv` files, but going to each webpage and downloading a `csv` is not most efficient or robust way to go about getting the data. Without an existing API, I think it is easier to scrape the data from the HTML.

### Scraping the data

The first step is to get the HTML from the website into a form that is easy to search and manipulate. For this exercise I'll be using Python and `BeautifulSoup`. `BeautifulSoup` is a library for python that allows you to take the HTML from a website as a long string, then compartmentalizes it into an object with methods making it easier to get just the relevant data. To get the actual HTML string to give to `BeautifulSoup`, I use the `requests` package. So getting the HTML into `BeautifulSoup` would look something like this:

```python
from bs4 import BeautifulSoup as BS
from requests import get as r_get

test_url='https://www.basketball-reference.com/leagues/NBA_2017_per_game.html'
test_HTML=r_get(test_url)
soupTest=BS(test_HTML.text,'html.parser')
```

Once in a `BeautifulSoup` object, I want to extract each row or instance of the statistics as a `dict` in python. Dictionaries in python act as key-value pairs, just like how each column in the basketball statistics table is a type of statistic and each row has a value for that statistic. This structure of the `dict` data type is similar to a row in a table of data. Because of this similarity, we can actually create a `pandas` DataFrame from a list of `dict`s, something that could be useful for storage and manipulation of data.

Every website will have the data appear in HTML differently, but the hope is that the data appears in a similar structure over all Basketball Reference pages. A quick check finds this to be true. Now we can run a `BeautifulSoup` method called `find_all(pattern)` and it will return all the instances in the HTML where this pattern occurs. Using the pattern `tr`, which stands for "table row", we can take a look at how the statistics are stored in the HTML.

```python
soupTest.find_all('tr')[1]
```
```html
<tr class="full_table"><th class="right " csk="1" data-stat="ranker" scope="row">1</th><td class="left " csk="Abrines,Alex" data-append-csv="abrinal01" data-stat="player"><a href="/players/a/abrinal01.html">Alex Abrines</a></td><td class="center " data-stat="pos">SG</td><td class="right " data-stat="age">23</td>.....
```

From an inspection of the return of `find_all('tr')`, the column value is represented in a variable called `data-stat`. Separate from the brackets is the value for the column in this row, which is seen between `<td>  </td>` brackets. Running `find_all('td')` should give all the values within the table. Here I will show one element of the list as an example.

```python
soupTest.find_all('tr')[1].find_all('td')[1]
```
```html
<td class="center " data-stat="pos">SG</td>
```

With this information, turning a row of the data into a `dict` is not that difficult. We just need to create a function that for each row in the table of statistics makes a `dict` object with key-value pairs for every "table data" or `<td>` instance of the text that appears and the `data-stat` or type of data it represents.

```python
def getPlayerDict(playerRow):
    playerDict={}
    for column in playerRow:
        playerDict[column.attrs.get("data-stat")]=column.text

    return playerDict

getPlayerDict(soupTest.find_all('tr')[1].find_all('td'))

RETURNS
{'age': '23',
 'ast_per_g': '0.6',
 'blk_per_g': '0.1',
 'drb_per_g': '1.0',
 'efg_pct': '.531',
 'fg2_pct': '.426',
 'fg2_per_g': '0.6',
 'fg2a_per_g': '1.4',
 'fg3_pct': '.381',
 ...}
```

Running this function on all the rows in `find_all('tr')` will create a list of `dict`s with each one being a row in the basketball statistics table. Then these functions can be called on different webpages within Basketball Reference, easily getting large amounts of data from the website with relative ease.

### Using the data

Once the data has been gathered, it is useful to put it into a `pandas` DataFrame. It can easily convert the entire dataset to numeric values and has convenient ways to deal with missing values.
```python
playerDF=pd.DataFrame(playerStatList).dropna(subset=['player'])
for column in playerDF:
            playerDF[column]=pd.to_numeric(playerDF[column],errors='ignore')
```
Here the `playerStatList` is a list of `dict`s for each row in to table of statistics. I also get rid of instances with don't have a `player` attribute. Since the statistics are numeric values, I use the pandas `to_numeric` method to change the data type when applicable.

With all the data in a nice form, we can now plot the data to try and gain insights. I'm interested in how the NBA changes over time, so I will create a plot that tracks league averages from the 1980 to today.

Take a look below to see how the NBA has changed!

{% include nba_evo.html %}
