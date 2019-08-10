---
layout:     post
title:      Decoding the Federalist Papers
date:       2019-7-21 12:00:00
summary:    The Federalist Papers were a series of essays that helped gain support for ratifying the U.S. constitution. Originally published anonymously, we now know that they were written by Alexander Hamilton, James Madison, and John Jay. Can the author of each essay can be determined by their other writings?
categories: Projects
---

A few months ago I was listening to the [Linear Digressions](http://lineardigressions.com/) podcast about natural language processing and the hosts mentioned how the authors of the Federalist Papers were determined using natural language processing (NLP). I was looking for small data science projects to do when I had time and this one seemed like a great idea. Could I determine the authors of the Federalist Papers using their previous writings in an afternoon? (The Answer: Sort Of)

For the uninitiated, the Federalist Papers are a series of 85 essays written in the late 1780's to garner support for the ratification of the U.S. Constitution.  Written by Alexander Hamilton, James Madison, and John Jay under the pseudonym **PUBLIUS** (more on that later). After the ratification of the constitution, these three were not shy about claiming ownership for the essays. Hamilton even claimed credit for "two thirds" of all the essays.

In 1964, a 303 page [book](https://books.google.com/books?id=LJXaBwAAQBAJ) was published on determining the authorship of the essays. It attributed 51 essays to Hamilton, 29 to Madison, and 5 to Jay. This post will not take 303 pages. But we will see what modern techniques (read scikit learn) can do.

The first step is getting the data. As the copyright is out, the Federalist Papers themselves can be obtained from [Project Gutenberg](https://www.gutenberg.org/ebooks/18). For this project I want the raw text, as it will be easier to work with. For writing samples to train our model on, the best source would be writing by the authors on government, excluding the Federalist Papers. For [Hamilton](https://archive.org/stream/worksalexanderh22hamigoog/worksalexanderh22hamigoog_djvu.txt) and [Madison](https://archive.org/stream/lettersotherwrit04madi/lettersotherwrit04madi_djvu.txt) I used `.txt` files of books of their political writings and took out the Federalist Papers essays by hand if present. Despite being the first Chief Justice of the Supreme court, there was not a similar source for John Jay. Instead I used [letters](https://oll.libertyfund.org/people/john-jay) that he wrote during his lifetime.

Once I got the data, I decided to use Python to study it. I choose Python because of the scikit learn library and that I am most comfortable with python. From my experience of R I assume a similar package exists there.

My plan for this project was:
1. Clean the `txt` files so I had just the writings.
2. Segment and attribute the essays to each author.
3. Train a model on writings excluding the Federalist papers
4. Predict the authors of the Federalist Papers.
5. Compare predictions to actual authors.

Within a more rigorous setting, I would take the results from step five and go back to step three until I had diminishing returns in model accuracy, but this was more of a proof of concept project for me.

### Clean the Text

To clean the files, I didn't want headers (all caps), newlines, page numbers, or punctuation. So I wrote a function that cleaned the text. This relies upon regular expressions (and stack overflow answers).

```python
def clean_source(txt):
    #replace newlines with space
    txt=txt.replace("\n"," ")
    #get rid of punctuation
    txt=re.sub(r'[^\w\s]','',txt)
    #remove strings containing numbers
    txt=re.sub(r'\w*\d\w*', '', txt).strip()
    #get rid of all caps
    txt=re.sub(r'\b[A-Z]+\b', '', txt)

    return(txt)
```

### Segment the Federalist Papers

With clean text, I could move on to number 2: author segmentation. Within the federalist papers, I needed to separate out the essays and the authors for each essay. I was lucky for the txt file to have the author name before each essay. This marked the start of the essay. And remember how each author signed the document **PUBLIUS**? Well that served as a perfect ending for each essay. While some extra cleaning was needed for multiple authorship and a few missing **PUBLIUS**'s, the code looked like:

```python
re.search("MADISON",fpapers_txt)
# and
re.search("PUBLIUS",fpapers_txt)
```
This search function only gave the first occurrence, so the location of a positive match was used to find the next word until no matches exist. Each author and their essay was then put into a pandas DataFrame.

### Train and test a model

By following a [tutorial](https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html) from the scikit learn webiste, I was able to analyze the data. Models need to be trained on some feature, so a natural choice for this kind of analysis is "TD-IDF" or **T**ext **F**requency times **I**nverse **D**ocument **F**requency. The idea behind this feature is that the best words to classify text are not the words that show up the most, like "the" or "and", but infrequent words that show up more often than normal.

Using this as the feature, I was then introduced to the Pipeline object within the tutorial. And man, abstraction is a powerful tool. So much goes into this one object, namely the vectorization of the words (text to a numerical matrix), conversion of words frequency to TF-IDF, and the training a model on a naïve Bayes classifier. The entirety of the train and predict process is:
```python
text_clf = Pipeline([('vect', CountVectorizer()),('tfidf', TfidfTransformer()),('clf', MultinomialNB())])
text_clf.fit(sources_df["Text"],sources_df["Author"])
predicted=text_clf.predict(fpapers_df["Text"])
```
Use of multinomial naïve Bayes is justified not only by its presence in the tutorial, but also in its simplicity. I was training just on one occurrence of each class (author). Therefore algorithms which require iterations over a deeper set of data compared to wider (many features, few instances) sets of data would perform worse.

### Evaluating the model

The easiest question to ask is "How many essays did it get right?" The simple answer is 73 out of 85 or about 86 percent. This is great, much better than I was expecting to be honest. The next step is to ask "How was I wrong?"

Looking at a confusion matrix (where diagonal elements are correct identifications and off diagonal element represent how an instance was misclassified) gives additional insight.

| Predicted Author | Hamilton | Jay | Madison |
|------------------|:---------:|:---:|:-------:|
| Actual Author    |          |     |         |
| Hamilton         | 50       | 1   | 0       |
| Jay              | 0        | 5   | 0       |
| Madison          | 11       | 0   | 18      |

This shows that a majority of the misclassifications are predicting Hamilton and the actual author is Madison. Is this a sign of mislabeled authorship? There is some controversy on whether Madison wrote essays 49-58 and 18-20 were a collaboration. However of the eleven misclassifications, only five were in this set of disputed essays.

Maybe Hamilton was right that he really did write two thirds. But a much more likely explanation is that this model is imperfect and overfits for Hamilton. A reason for this could be that the source writing I used for him was more political than Madison's, thus having a higher frequency of words that may appear in the Federalist Papers.

Anyways, hope you enjoyed this project. I found it pretty cool that you could (somewhat) predict something that took historians several years of dedicated work to decipher. If you want the data or code I used, it's up on my GitHub under "Python Projects". Thanks.
