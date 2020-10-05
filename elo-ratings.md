<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# How Not To Calculate Average Elo Rating

## TL;DR (for the math-phobic)
If you are in charge of a team in some team game where each of your players have Elo ratings, please don't calculate your team's average rating the usual way: don't just sum their ratings and divide by the number of players. To get a more accurate rating, take the average you would usually get, add the rating of the team member with the highest rating, and divide by 2!

## Elo ratings are on a log scale
Elo ratings can be misleading because they are on a log scale, which basically means that you cannot determine the chance of player A winning over player B by just dividing player A's rating by the sum of both player A and B's ratings. For example, a player with an Elo rating of 3000 would win against a player with an Elo rating of 1500 not two-thirds of the time, as

$$ P(\text{player A beats player B}) \neq \frac{3000}{3000 + 1500} = \frac{2}{3} $$

Instead, the formula that determines the chance of winning is, given player ratings \\( r_a \\) and \\( r_b \\) of player A and B respectively,

$$ P(\text{player A beats player B}) = \frac{1}{1 + 10^{(r_b - r_a)/400}} $$

Note that 400 is a constant which might vary depending on how the system is setup. The choice of 400 just means that if a player has 400 more Elo points than an opponent, they will be approximately 10 times more likely to win than their opponent. This makes sense as otherwise the ratings would quickly become too large. Often the gap between beginners and experts in a game is huge -- for example, a person who just learned how to play chess two days ago would have no chance against the world champion Magnus Carlsen. The expert would win against the beginner nearly 100% of the time. Let's say the beginner has an Elo rating of 800, which is already very low when it comes to Elo ratings. Suppose that the expert has a 99.99% chance of beating the beginner, which might actually be generous in some games where players spend their entire lives learning it. Then the expert's Elo rating according to the model, assuming that the mentioned constant is set to 400,

$$ \begin{align}
r_\mathrm{beginner} = 800 \\
P(\text{expert beats beginner}) = 0.9999 \\
\Rightarrow 0.9999 = \frac{1}{1 + 10^{(500 - r_\mathrm{expert})/400}} \\
r_\mathrm{expert} \approx 2400
\end{align} $$

Then the expert's Elo rating would be 2400. So far so good. But assume that instead of putting Elo ratings on a log scale, we calculate the chance of the expert beating the beginner as \( \frac{r_\mathrm{expert}}{r_\mathrm{expert} + r_\mathrm{beginner}} \). Then,


$$ \begin{align}
r_\mathrm{beginner} = 800 \\
P(\text{expert beats beginner}) = 0.9999 \\
\Rightarrow 0.9999 = \frac{r_\mathrm{expert}}{r_\mathrm{expert} + 800} \\
r_\mathrm{expert} = 7999200
\end{align} $$

Yikes! Clearly the log scale is a sane decision when it comes to skill ratings. But the log scale leads to a problem...

## The right way to average Elo ratings
Normally, we average things by adding them and dividing by the number of things. But let's say you're playing a team game, and you need to find the average rating of a team based on each team member's individual rating. The problem with calculating the average rating the typical way can be illustrated by an example. Suppose you're playing a 6v6 team game against an average opponent. Your opponent is run-of-the-mill average, and all of their ratings are 1500. Meanwhile, most of your team is below average, with five players having ratings of 1200. But you've decided that enough was enough and decided to invite one of the world's foremost experts at the game with a rating of 2500. Clearly, just having the expert alone increases your chances greatly and more than makes up for the low ratings of your other players. Well, your team's "average" rating is still 1417 (\\( \approx 8500/6 \\)) but your opponent's average rating is 1500. Big yikes!

You have to compensate for the fact that Elo ratings are on a log scale, which means you have to convert those Elo ratings into a linear scale first. Consider the formula that calculates the chance of winning,

$$ P(\text{player A beats player B}) = \frac{1}{1 + 10^{(r_b - r_a)/400}} $$

If we let \\( s_a = 10^(r_a / 400) \\) and \\( s_b = 10^(s_b / 400) \\) (for players A and B respectively), then some calculation results in the following equivalent formula,

$$ P(\text{player A beats player B}) = \frac{s_a}{s_a + s_b} $$

Define \\( f(x, c) = 10^{x/c} \\). \\( r \\) is the rating, and \\( c > 0 \\) is the constant I was mentioning before, usually set to 400. Then, suppose that there are \\( n \\) players with ratings \\( r_1, r_2, \dots, r_n \\). The right way to calculate the average is,

$$ A(r_1, r_2, \dots, r_n; c) = c \log_{10}\left(\frac{1}{n} \sum_{i=1}^n r_i \right) $$