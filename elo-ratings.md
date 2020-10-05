<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# How Not To Calculate Average Elo Rating

## Elo ratings are on a log scale
Elo ratings can be misleading because they are on a log scale, which basically means that you cannot determine the chance of player A winning over player B by just dividing player A's rating by the sum of both player A and B's ratings. For example, a player with an Elo rating of 3000 would win against a player with an Elo rating of 1500 not two-thirds of the time, as

$$ P(\text{player A beats player B}) \neq \frac{3000}{3000 + 1500} = \frac{2}{3} $$
