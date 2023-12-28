### [H-1] The `enterRaffle` function can be DOS attacked

**Description:** 

The `enterRaffle` function loops through the players array to check for duplicates . 

As the length of ' players ' array increase, the gas cost and the number of checks a new player must carryout also increase .

This issue has the potential to deter players that enter later due to the remarkably higher gas cost .



**Impact:** 

The skyrocketing costs for users  entering the raffle at a later stage could deter participation . Furthermore, an attacker with large enough resources could monopolize the system , crowding out other potential participants .



**Proof of Concept:**

To demonstrate the vulnerability at hand, we could showcase the escalating gas costs with a simple comparison - taking two sets of 100 players each and observing the gas charges. Our projected surge in costs could look something like this:

```markdown
## Proof of Concept:

1st set of 100 players: ~70,000 gas
2nd set of 100 players: ~210,000 gas
Note: The second set of players face a gas cost more than 3 times that of the initial set.
```



**Recommended Mitigation:** 

We recommend altering the manner in which duplicate players are checked – switching from an iteration-based system to a mapping-based system – which would be a far more gas-efficient solution.