---
layout:     post
title:      Poker Starting Hands and Monte Carlo Simulation
date:       2019-8-9 12:00:00
summary:   In Texas Holdem, each player is given two starting cards. Then five community cards are revealed and each player makes the best five card hand from their two cards and the five community cards available. But how strong are your two starting cards? Using Monte Carlo simulations, I find the distribution of expected hands given two starting cards.
categories: Projects
---

### Monte Carlo and Poker Hands

I've been interested in Texas Hold'em poker recently, as it provides a nice example of a game relying on understanding of numbers and basic statistics. The rules of Texas Hold'em are relatively simple. Every player (usually five to six), is dealt two cards. Five community cards are then revealed face up to everyone. First, three cards called the "flop" are shown, then one card called the "turn", and then one last card called the "river". Each player then makes the best hand they can out of the seven cards available to them, the two they received initially and the five community cards.

As poker is a gambling game, there are several rounds of betting. The first round of betting happens after everyone gets their first two cards. So the first decision that a Hold'em player needs to make is based on the strength of their two cards. But how strong are any two cards in this game? It's not clear initially whether a pair of threes is better or worse than an Ace and a King. And what does better or worse mean?

To figure this out I wanted to calculate a probability distribution for the strength of the final hand, given two initial cards. I decided that the "strength" of a hand will be quantified from 0 to 1 as the percentage of random Hold'em hands it will beat. This calculation neglects the fact that poker hands are highly correlated since all players share five cards. In order to calculate this distribution, I decided to use a Monte Carlo simulation.

Monte Carlo simulations are processes that use random sampling to gain insights on numerical events. A classic Monte Carlo simulation example is the determination of \\( \pi \\) from the number of random points within a square that lie within a circle inscribed in that square. The generation of these random points are done using a psuedo-random process, meaning the numbers are generated deterministically but have the statistics of a random sample. Monte Carlo simulations are also used in Physics, where results or distributions may analytically intractable, but can be well approximated by careful random sampling.

Monte Carlo simulation can be used in poker for the random sampling of cards within a deck. Within `python` it is easy to use the numpy package. If using `import numpy as np`, the random sampling can be done in just a few lines. This needs to be run for every calculation of starting hands strength.

```python
deck=np.arange(0,52,1)
np.random.shuffle(deck)
```

### Strength of a Hand

The strength of a poker hand is given by how likely a given combination of cards to occur. A particular hand is usually defined by a pattern within the cards, like three of the same kind or if all of the cards have the same suit. A figure of the hand rankings is shown below.

<p align="center">
<img src="/_img/hand_strength.jpg" alt="drawing" width="350" />
</p>

First I need to define what cards are within the python environment. I could create a Deck object, but since cards within a deck can be mapped one to one with integers from 1-52 (or in python 0-51) I chose to do that instead. This can be done using modular arithmetic. When I divide a number from 0-51 by 13, I get a number 0-3 with a remainder 0-12. The first number is the suit of the card (hearts, diamonds, spades, clubs) and the remainder is the value of the card (from 2-Ace).

The next step is to write a function to take in seven cards and find the best five card hand. This was the most difficult part of this project, but was accomplished using the `np.unique` function, which returns all the unique occurrences of values and how many times they occurred. This made it easier to check for types of hands. For example, if a flush occurred then `np.unique` of the suits of the cards would have an instance with five counts. Similar approaches were used for all other hands, working in descending order from the strongest hand to the weakest hand.

Once I was able to identify a hand, I could see how often each type of hand happened if seven cards were chosen randomly. This would give me an approximation for the percentage of hands a given hand would beat. For a sanity check, I also looked at the distribution of hands for five randomly selected cards. The strength of hands from five cards should be worse than from seven since only the best five cards are selected to make a hand.

<p align="center">
<img src="/_img/five_v_seven.png" alt="drawing" width="500" />
</p>

This distribution sets a baseline for the strength of a hand of a certain type. Since a hand of a certain type will beat all the hands lower than that type, the base strength of a hand is the sum of probabilities of having a lower hand. Within that hand type, hand strength can be further divided by the high card. For most of the hand types this gradation is straightforward, as your equally as likely to have all pairs, sets, and four of a kinds and straights can have a high card from 5 to an Ace with equal likelihood.

However this is not true for a "High Card" hand. For these hands, the six other cards need to be all different without having a five in a row. Combinatorically, we need to chose six cards from all possible cards below and then subtract all possible straights. This looks like \\( {n-1 \choose 6} - (n-6) \\). Here n is the number 1-13 that represents the value of the card. We can compare the theoretical result to the simulated results (below) and see that they match pretty well.

<p align="center">
<img src="/_img/high_card_dist.png" alt="drawing" width="500" />
</p>

We see that it much more likely to have Ace High than all other high card values. In the combinatoric sense this is expected, since there is many more combinations of the lower six cards if the highest card is an Ace.

With this distribution, we have all the data we need to assign a strength to a given seven card hand.

### Starting Two Card Strength Distribution

The distribution of final hands from two starting hands can be calculated by randomly generating the five community cards over and over again and recording the results. There are 169 unique two card openings (counting for suited or not suited cards only), which is a small enough number to calculate the distribution for all possible combinations.

For example, the distribution for a pair of 3s, is shown below. Beforehand, it might not be clear whether a pair of threes is a good hand. While you start with a pair, most other pairs will beat your hand.

<p align="center">
<img src="/_img/Pair3s.png" alt="drawing" width="500" />
</p>

The first and largest peak in this distribution represents your final hand being a pair of threes with various high cards. As you start with a pair of threes, your hand cannot get worse and it represents a baseline. The next range of hands is two-pair. All pairs besides the threes you already have are equally possible, so the probability distribution is flat over this region. The next and second largest spike is for three of kind, in this case three threes. The final spike represents hands like a full house, which would happen when a three appears in the community cards along with a pair.

Now consider two starting cards of Ace-King suited. While your hand starts off low (Ace High), it has the possibility of getting a high pair, a high straight, and a flush.

<p align="center">
<img src="/_img/AceKing-s.png" alt="drawing" width="500" />
</p>

The largest peak in this distribution represents your hand staying at Ace High. While it is the most likely individual outcome, it is more likely that the final hand will be better than Ace high. The next small nonzero region of the distribution is that a pair besides Ace or King appears in the five community cards. The next largest bump occurs for straights and flushes, both of which are more likely for suited connecting cards.

### Interpreting Results: Percentile Heatmaps

So how should we compare a starting hand of Ace-King suited to a pair of 3s? I think the best way is in terms of percentile outcomes. A pair of 3s will have a high floor as a pair is guaranteed. On the other hand, Ace-King suited will have a higher hand for top ten percent of its distribution.

Which is better, having a high floor or a high ceiling? In actual poker the answer is that it depends. A high floor is often best when there are fewer people at the table, as the less likely high strength hands are more infrequent. But getting a very strong hand offers a tremendous hand with regards to betting, as the player can be confident they will win the money at stake.

To best visualize the strength of every starting hand, I've made some heatmaps shown below for the 25th percentile, 50th percentile, 75th percentile. The upper left portion of these graphs represents the less likely suited combination. So while there are 169 possible starting hands, they are not all equally likely to occur.

##### 25th Percentile Heatmap

<p align="center">
<img src="/_img/twentyfive.png" alt="drawing" width="500" />
</p>

When first analyzing this graph, I thought I did something wrong as it didn't seem to make a difference if you had a pair to start or not. But important in this is understanding that the distributions are not Gaussian/normal distributions. What we are really sampling is small regions of hands. So in the 25th percentile of final hands here, most of the low hands have made bottom pair and the paired hands have stayed the same.

If we instead look at the 10th percentile distribution, all of the unpaired hands finish as High Cards, while the paired hands are significantly better.

<p align="center">
<img src="/_img/ten.png" alt="drawing" width="500" />
</p>

##### 50th Percentile Heatmaps

<p align="center">
<img src="/_img/fifty.png" alt="drawing" width="500" />
</p>

In the fiftieth percentile of hands, all of the paired starting hands have made at least two pair. The heatmap is decently symmetric, but some asymmetry is noticeable near the bottom left of starting hands. I think this is due to suited cards having a larger portion of their distribution attributed to flushes, which brings up the distribution as a whole.

##### 75th Percentile Heatmaps

<p align="center">
<img src="/_img/seventyfive.png" alt="drawing" width="500" />
</p>

In the seventy fifth percentile of hands we have our straights, sets, and flushes. The difference between the upper left and bottom right is more noticeable, mainly due to the strength of a flush. Also, hands close to the diagonal are stronger than those further away, due to the increased probability of a straight.


### Final Thoughts

While this was a fun exercise for me, the actual game of poker is much more complex. Knowing how much to bet and when is extremely important, understanding the ranges of your opponents is crucial, and circumstantial bluffing is necessary. What I think I did here is show that given two starting cards, what is the likelihood that I will beat \\( x \\) random hands. I also enjoyed using Monte Carlo simulations in the past, so doing this again outside of the academic environment and on my own was fun.

Thanks for reading!
