<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
# Reasoning About Functional Programs: How?
Advocates of functional programming often tout how easy it is to reason about functional programs. Okay, we know that it is supposedly easy to reason about them, but how? Yes, I know how functional programming is about the "what" and not the "how", but a little bit of the latter often doesn't hurt. We will look at an actual example where we use reasoning to come up with an functional algorithm for the gambler's favorite card game: Blackjack!

## The problem
Blackjack is a card game where the house always wins. Pretty much everyone knows the basic rules, but here's a review just in case. Each card is assigned a value: for the cards from 2 to 10, it is their face value, and for Jacks, Queens, and Aces, the value is 10. But Aces are special - they may either be 1 or 11.

When calculating the value of a hand, if there are no Aces, just add the values of the cards. But if there are Aces, you have to be careful. Suppose you are dealt these two cards to your hand: 5, A. The value is 16, with the Ace being worth 11. Of course you want the largest value possible, so if you decide to stand, the value will remain 16.

Let's say the next card you draw is a 10. If the Ace retains its value of 11, your hand now has a value of 26 (5 + 11 + 10), which is a bust as it's over 21! But the Ace now unleashes its hidden ability and now has a value of 1, so your hand actually retains its value of 16 (5 + 1 + 10). Phew.

We will use Python, which although not a pure functional language, can still do functional programming. Let's create a simple `Card` class, which doesn't have any methods - it is just a way to enumerate the cards:

```python
class Card:
  ACE = 1
  TWO = 2
  THREE = 3
  FOUR = 4
  FIVE = 5
  SIX = 6
  SEVEN = 7
  EIGHT = 8
  NINE = 9
  TEN = 10
  JACK = 11
  QUEEN = 12
  KING = 13
```

In Python 3.4 or higher there is an enum type but this ain't broke so we don't need it. Now how do we calculate the value of a hand, or a list of `Card`s? Well, let's start with calculating the value of a single card. The easiest cards are from `TWO` to `TEN`:

```python
# Takes a Card and returns an int
def calc_card_value(card):
  if card >= Card.TWO and card <= Card.TEN:
    return card
```

Since each card from `TWO` to `TEN` has a value equal to its value in Blackjack, it's easy. The `JACK`, `QUEEN`, and `KING` each have a value of 10:

```python
# Takes a Card and returns an int
def calc_card_value(card):
  if card >= Card.TWO and card <= Card.TEN:
    return card
  elif card >= Card.JACK:
    return 10
```

That leaves us with the Aces. But hold up, they can either be 1 or 11, which presents a problem. `calc_value` was supposed to return an integer, but obviously the Ace presents us with a problem. Let's return 0 when the card is an Ace for now:

```python
# Takes a Card and returns an int
def calc_card_value(card):
  if card >= Card.TWO and card <= Card.TEN:
    return card
  elif card >= Card.JACK:
    return 10
 # else, card == Card.ACE is the only possible case
 return 0
```

## First steps
By pondering over the problem a little, we can divide the problem into two parts: calculating the value of the cards that aren't Aces, and calculating the value of the Aces. The total value of a Blackjack hand can be expressed as a function:

```python
def sum_hand_value(nonAceValue, aces1, aces11):
  return nonAceValue + aces1 + 11 * aces11
```

`nonAceValue` is the value of the non-Ace cards calculated by summing the values returned by `calc_value` on those cards. `aces1 + aces11` is equal to the number of Aces in the hand, where `aces1` are the number of Aces considered to be worth 1, and `aces11` are the Aces considered to be worth 11. Let `calc_hand_value` be the function that calculates the value of a list of `Card`s. First, we split the list into non-Ace cards and Ace cards:

```python
# Takes a list of Cards and returns an int
def calc_hand_value(hand):
  nonAces = [card for card in hand if card != Card.ACE] # Yields all the cards that are not Aces.
  numAces = len(hand) - len(nonAces) # All the cards remaining
  nonAceValue = sum([calc_card_value(card) for card in nonAces]) # Yields the total value of all the non-Ace cards
  aces1 = ?
  aces11 = ?
  return sum_hand_value(nonAceValue, aces1, aces11)
```

The problem remains: find the correct value of `aces1` and `aces11`.

## The goal
We want to find a function `distribute_aces` that:

- **given** the total value of all the non-Ace cards, `nonAceValue`, and the number of Aces, `numAces`,
- **returns** a pair `(aces1, aces11)` that tells us how many of the Aces should be worth 1 or worth 11.

The answer returned by `distribute_aces` must satisfy a property:distr

**Property:** If `distribute_aces(nonAceValue, numAces) == (aces1, aces11)`, then `sum_hand_value(nonAceValue, aces1, aces11)` must be as close to 21 as possible without going over. 

That is, for a pair `(aces1, aces11)` returned by `distribute_aces`, there must not be another pair `(other_aces1, other_aces11)` that may be returned by `distribute_aces` such that:

- `sum_hand_value(nonAceValue, other_aces1, other_aces11)` is greater than `sum_hand_value(nonAceValue, aces1, aces11)` and
- `sum_hand_value(nonAceValue, aces1, aces11)` is less than or equal to 21.

Properties are important because they give us a goal to achieve, and it is our job to write a function and *prove* that it meets the property. This is in contrast to the usual method of unit testing where we just write a bunch of input-output pairs and hope we didn't miss an edge or corner case. 

## Do the easy stuff first
Consider the case where `nonAceValue + numAces > 21`. Obviously, the player has already busted because even if all their Aces are considered to be worth 1, they'll still go over 21. The usual convention is to just make all the Aces worth 1 anyways to minimize their sorrow.

```python
def distribute_aces(nonAceValue, numAces):
  if nonAceValue + numAces > 21:
    return (numAces, 0)
```

## Simplify the problem
We have to choose two values `aces1` and `aces11`. But we can simplify the problem by noticing that `aces1` is dependent on the value of `aces11`, as `aces1 == numAces - aces11`. Since `numAces` is already known in advance, we only have to choose a value of `aces11` and we will get `aces1` for free.

```python
def distribute_aces(nonAceValue, numAces):
  if nonAceValue + numAces > 21:
    return (numAces, 0)
  # choose aces11:
  aces11 = ?
  # ezpz, (aces1, aces11) is equivalent to (numAces - aces11)
  return (numAces - aces11, aces11)
```

Furthermore, note that since the value `sum_hand_value(nonAceValue, aces1, aces11)` must not exceed 21 if possible,

$$ \begin{aligned} \mathrm{sumHandValue}(\mathrm{nonAceValue}, \mathrm{aces1}, \mathrm{aces11}) \leq 21 \\
\mathrm{nonAceValue} + \mathrm{aces1} + 11 * \mathrm{aces11} \leq 21
\end{aligned} $$
