# Reasoning About Functional Programs: How?
Advocates of functional programming often tout how easy it is to reason about functional programs. Okay, we know that it is supposedly easy to reason about them, but how? Yes, I know how functional programming is about the "what" and not the "how", but a little bit of the latter often doesn't hurt. We will look at an actual example where we use reasoning to come up with an functional algorithm for the gambler's favorite card game: Blackjack!

## The Problem
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

In Python 3.4 or higher there is an enum type but it isn't needed for our purposes. Now how do we calculate the value of a hand, or a list of `Card`s? 
