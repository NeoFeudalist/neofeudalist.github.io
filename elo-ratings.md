<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script>
MathJax = {
  tex: {
    inlineMath: [              // start/end delimiter pairs for in-line math
      ['$', '$'],
      ['\\(', '\\)']
    ]
  }
};
</script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>


# How Not To Calculate Average Elo Rating

## TL;DR (for the math-phobic)
If you are in charge of a team in some team game where each of your players have Elo ratings, please don't calculate your team's average rating the usual way: don't just sum their ratings and divide by the number of players. To get a more accurate rating, take the average you would usually get, add the rating of the team member with the highest rating, and divide by 2! 

For example, let's say your team has three players with ratings: 1600, 1500, and 2300. The average would be 1800 (1600 + 1500 + 2300 = 5400, divide by 3). Then average it with the highest rating of 2300, resulting in a final rating of 2050. This is a way fairer way of calculating average Elo ratings.

## Elo ratings are on a log scale
Elo ratings can be misleading because they are on a log scale, which basically means that you cannot determine the chance of player A winning over player B by just dividing player A's rating by the sum of both player A and B's ratings. For example, a player with an Elo rating of 3000 would win against a player with an Elo rating of 1500 not two-thirds of the time, as

$$ P(\text{player A beats player B}) \neq \frac{3000}{3000 + 1500} = \frac{2}{3} $$

Instead, the formula that determines the chance of winning is, given player ratings $ r_a $ and $ r_b $ of player A and B respectively,

$$ P(\text{player A beats player B}) = \frac{1}{1 + 10^{(r_b - r_a)/400}} $$

Note that 400 is a constant which might vary depending on how the system is setup. The choice of 400 just means that if a player has 400 more Elo points than an opponent, they will be approximately 10 times more likely to win than their opponent. This makes sense as otherwise the ratings would quickly become too large. Often the gap between beginners and experts in a game is huge -- for example, a person who just learned how to play chess two days ago would have no chance against the world champion Magnus Carlsen. The expert would win against the beginner nearly 100% of the time. Let's say the beginner has an Elo rating of 800, which is already very low when it comes to Elo ratings. Suppose that the expert has a 99.99% chance of beating the beginner, which might actually be generous in some games where players spend their entire lives learning it. Then the expert's Elo rating according to the model, assuming that the mentioned constant is set to 400,

$$ \begin{align}
r_\mathrm{beginner} = 800 \\
P(\text{expert beats beginner}) = 0.9999 \\
\Rightarrow 0.9999 = \frac{1}{1 + 10^{(800 - r_\mathrm{expert})/400}} \\
r_\mathrm{expert} \approx 2400
\end{align} $$

Then the expert's Elo rating would be 2400. So far so good. But assume that instead of putting Elo ratings on a log scale, we calculate the chance of the expert beating the beginner as $ \frac{r_\mathrm{expert}}{r_\mathrm{expert} + r_\mathrm{beginner}} $. Then,


$$ \begin{align}
r_\mathrm{beginner} = 800 \\
P(\text{expert beats beginner}) = 0.9999 \\
\Rightarrow 0.9999 = \frac{r_\mathrm{expert}}{r_\mathrm{expert} + 800} \\
r_\mathrm{expert} = 7999200
\end{align} $$

Yikes! Clearly the log scale is a sane decision when it comes to skill ratings. But the log scale leads to a problem...

## The right way to average Elo ratings
Normally, we average things by adding them and dividing by the number of things. But let's say you're playing a team game, and you need to find the average rating of a team based on each team member's individual rating. The problem with calculating the average rating the typical way can be illustrated by an example. Suppose you're playing a 6v6 team game against an average opponent. Your opponent is run-of-the-mill average, and all of their ratings are 1500. Meanwhile, most of your team is below average, with five players having ratings of 1200. But you've decided that enough was enough and decided to invite one of the world's foremost experts at the game with a rating of 2500. Clearly, just having the expert alone increases your chances greatly and more than makes up for the low ratings of your other players. Well, your team's "average" rating is still 1417 ($ \approx 8500/6 $) but your opponent's average rating is 1500. Big yikes!

You have to compensate for the fact that Elo ratings are on a log scale, which means you have to convert those Elo ratings into a linear scale first. Consider the formula that calculates the chance of winning,

$$ P(\text{player A beats player B}) = \frac{1}{1 + 10^{(r_b - r_a)/400}} $$

If we let $ s_a = 10^{r_a / 400} $ and $ s_b = 10^{s_b / 400} $ (for players A and B respectively), then some calculation results in the following equivalent formula,

$$ P(\text{player A beats player B}) = \frac{s_a}{s_a + s_b} $$

Define $ f(x, c) = 10^{x/c} $. $ r $ is the rating, and $ c > 0 $ is the constant I was mentioning before, usually set to 400. Then, suppose that there are $ n $ players with ratings $ \mathbf{r} = r_1, r_2, \dots, r_n $. The right way to calculate the average is,

$$ A(\mathbf{r}; c) = c \log_{10}\left(\frac{1}{n} \sum_{i=1}^n f(r_i, c) \right) = c \log_{10}\left(\frac{1}{n} \sum_{i=1}^n 10^{r_i / c} \right) $$

Expressed in Python:

```python
from math import log10
def average_rating(ratings, c):
  linscale_ratings = [10 ** (rating / c) for rating in ratings]
  return c * log10(sum(linscale_ratings) / len(ratings))
```

Let's go back to our example. Our team with five noobs and one superstar now has an average rating of

$$ A(2500, 1200, 1200, 1200, 1200, 1200; c = 400) \approx 2189$$

which more accurately reflects the team's actual average rating. Meanwhile the team full of 1500 rated players would still have an average of

$$ A(1500, 1500, 1500, 1500, 1500, 1500; c = 400) \approx 1500$$

Define the following function for the naive way of calculating average rating:

$$ E(\mathbf{r}) = \frac{1}{n} \sum_{i=1}^n r_i $$

We can prove that this is always either an underestimate of or equal to the average calculated with our method.

**Theorem 1.** $ A(\mathbf{r}; c) \geq E(\mathbf{r}) $ with equality holding if and only if $ r_1 = r_2 = \dots = r_n $.

*Proof.* Jensen's inequality states that for any concave function $ g(x) $ and a list of real numbers $ \mathbf{r} $,

$$ g\left( \frac{\sum_{i=1}^n r_i}{n} \right) \geq \frac{\sum_{i=1}^n g(r_i)}{n} $$

with equality holding if and only if $ r_1 = r_2 = \dots = r_n $. Then per Jensen's inequality and the concavity of the $ \log_{10} $ function,

$$ \begin{align}
A(\mathbf{r}; c) = c \log_{10}\left(\frac{1}{n} \sum_{i=1}^n 10^{r_i / c} \right) \\
\geq \frac{c}{n} \sum_{i=1}^n \log_{10}(10^{r_i / c}) = \frac{c}{n} \sum_{i=1}^n r_i / c = \frac{1}{n} \sum_{i=1}^n r_i \\
= E(\mathbf{r})
\end{align} $$

Equality holds if and only if $ r_1 = r_2 = \dots = r_n $ as required. $\blacksquare$

**Theorem 2.** $ A(\mathbf{r}; c) \leq \max(\mathbf{r}) $, with equality holding if and only if $ r_1 = r_2 = \dots = r_n $

*Proof.* Without loss of generality, let $ \max(\mathbf{r}) = r_1 $. Then for all $i$, $r_i \geq r_1$. Then, noting that $10^{r_i / c}$ is a strictly increasing function of $r_i$,

$$ \begin{align}
r_1 = c \log_{10}\left( 10^{r_1 / c} \right) \\
\geq c \log_{10}\left( \frac{1}{n} \sum_{i=1}^n 10^{r_i / c} \right) = A(\mathbf{r}; c)
\end{align}
$$

Equality holds if and only if every rating is equal to the maximum, that is, $ r_1 = r_2 = \dots = r_n $. $\blacksquare$

## A simple approximation
But what if logarithms and math jazz make your head turn? Is there a simpler way to calculate average Elo ratings without having to pull out that TI-84 that's been rotting in your attic since high school? One way is to first calculate the average ( $ E(\mathbf{r}) $ ), take the highest rating in the list, and then average your calculated average with the highest rating. That is, 

$$ B(\mathbf{r}) = \frac{E(\mathbf{r}) + \max(\mathbf{r})}{2} $$

Since the maximum value is always greater than or equal to the average, it follows that $ B(\mathbf{r}) \geq E(\mathbf{r}) $ with equality holding if and only if all ratings are equal, exactly the same case where equality holds for the inequality $ A(\mathbf{r}; c) \geq E(\mathbf{r}) $. Essentially, $ B(\mathbf{r}) $ is a correction for the underestimate of $E$. Another way to justify the approximation is to consider the form of $A(\mathbf{r}; c) $:

$$A(\mathbf{r}; c) = c \log_{10}\left(\frac{1}{n} \sum_{i=1}^n 10^{r_i / c} \right)$$

Without loss of generality, let $ \max(\mathbf{r}) = r_1 $ - rating lists can be considered as equivalence classes where two lists of ratings are equal if and only if they are permutations of each other. Take the derivative $ \frac{d}{dx} \log_{10}{x} = \frac{1}{x \ln(10)} $. Taking its first order Taylor series expansion around an arbitrary value of $ x $,

$$ \log_{10}(x + \delta) \approx \log_{10}(x) + \frac{\delta}{x \ln(10)} $$

Then,

$$
\begin{align}
A(\mathbf{r}; c) = c \log_{10}\left(\frac{1}{n} \sum_{i=1}^n 10^{r_i / c} \right) \\
\approx c \left(\log_{10}\left(\frac{10^{r_1 / c}}{n} \right) + \frac{\log_{10}\left(\frac{1}{n} \sum_{i=2}^n 10^{r_i / c} \right)}{\frac{10^{r_1 / c}}{n} \ln(10)} \right) \\
\approx c \left(\log_{10}\left( \frac{10^{r_1 / c}}{n} \right) \right) = c \left( \frac{r_1}{c} - \log_{10}(n) \right) = r_1 - c \log_{10}(n)
\end{align}$$

as if $r_1$ is much greater than the other ratings (in which case the naive average calculation method *really* fails), then the second term containing the remaining ratings would be insignificant as it is divided by the much larger $ \frac{10^{r_1 / c}}{n} $ term. So under the approximation, $A(\mathbf{r}; c) < r_1 = \max(\mathbf{r}) $, and $ B(\mathbf{r}) \leq \max(\mathbf{r}) $ as well. Another way to justify the approximation is to view it as the average of the bounds

$$
E(\mathbf{r}) \leq A(\mathbf{r}; c) \leq \max(\mathbf{r})
$$

We can see that the maximum error of the approximation is

$$
|B(\mathbf{r}) - A(\mathbf{r}; c)| \leq \frac{\max(\mathbf{r}) - E(\mathbf{r})}{2}
$$

## Properties of $ B(\mathbf{r}) $
**Theorem 3.** $B(\mathbf{r}) \geq E(\mathbf{r})$ with equality if and only if $r_1 = r_2 = \dots = r_n$.

*Proof.* Since for all $i$, $\max(\mathbf{r}) \geq r_i$, it follows that $ n \max(\mathbf{r}) \geq \sum_{i=1}^n r_i $ and $\max(\mathbf{r}) \geq \frac{1}{n} \sum_{i=1}^n r_i = E(\mathbf{r})$. Obviously equality holds if and only if $\max(\mathbf{r}) = r_i$ for all $i$, or in other words $r_1 = r_2 = \dots = r_n$. Then,

$$ \begin{align}
B(\mathbf{r}) = \frac{E(\mathbf{r}) + \max(\mathbf{r})}{2} \\
= \frac{E(\mathbf{r}) + E(\mathbf{r}) + \max(\mathbf{r}) - E(\mathbf{r})}{2} \\
= E(\mathbf{r}) + \frac{\max(\mathbf{r}) - E(\mathbf{r})}{2} \geq E(\mathbf{r})
\end{align} $$

Equality follows in the same conditions as mentioned. $\blacksquare$


