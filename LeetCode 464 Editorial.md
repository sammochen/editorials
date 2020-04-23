# LeetCode 464: Can I Win
Original problem: [https://leetcode.com/problems/can-i-win/](https://leetcode.com/problems/can-i-win/)

## Problem Description

In the "100 game," two players take turns adding, to a running total, any integer from 1..10. The player who first causes the running total to reach or exceed 100 wins.

What if we change the game so that players cannot re-use integers?

For example, two players might take turns drawing from a common pool of numbers of 1..15 without replacement until they reach a total >= 100.

Given an integer maxChoosableInteger and another integer desiredTotal, determine if the first player to move can force a win, assuming both players play optimally.

You can always assume that maxChoosableInteger will not be larger than 20 and desiredTotal will not be larger than 300.

### Example

```
Input:
maxChoosableInteger = 10
desiredTotal = 11

Output:
false

Explanation:
No matter which integer the first player choose, the first player will lose.
The first player can choose an integer from 1 up to 10.
If the first player choose 1, the second player can only choose integers from 2 up to 10.
The second player will win by choosing 10 and get a total = 11, which is >= desiredTotal.
Same with other integers chosen by the first player, the second player will always win.
```

## Summary
At first glance, the question feels like a standard dynamic programming question, where we see if it is possible to reach some target with a set of choices. However, the restriction where we cannot choose repeated values means we have to introduce a way to remember which numbers are available. The small ```maxChoosableInteger``` value means that we can remember the **set** of which numbers are left.

### Tags
Dynamic Programming, Bit Manipulation, Minimax

## Solution (Dynamic Programming, with Bitmasks)

The question asks if a player can reach the **target** if they can choose from a **set of numbers**. When the player makes a choice, the target decreases and the set of numbers reduces, and the problem is asked from the other player's perspective.

For example, if the **target** is 5 and the **choices** are [1,2,3], the three choices that the player has are:  
```
Take the 1: (target: 5-1 = 4, choices: [2,3])  
Take the 2: (target: 5-2 = 3, choices: [1,3])  
Take the 3: (target: 5-3 = 2, choices: [1,2])
```
Notice that three subproblems were formed, with lower target values and smaller range of choices. The appearance of **subproblems** hints that this question should be approached using dynamic programming.

### Strategy
How do we know if the player will win or lose? In a two-player game, you can win if **any** of your moves **force** the opponent to lose; you will lose if you **cannot** force them to lose. Of course, in this game, you will also win if you can decrease the target to 0 or below.

Let's implement this approach in pseudocode by introducing a method that returns whether the current player can win or not, given the **target** (a number) and **choices** (a set of numbers). 

```
bool canWin(target, choices):
    for choice in choices:
        let nextChoices = choices \ choice # The set of all choices except choice
        let nextTarget = target - choice

        if (nextTarget <= 0): # If you can reduce the target to 0 or below, you win!
            return true

        if (canWin(nextTarget, nextChoices) == false): # If you can force the opponent to lose, you win!
            return true

    return false
```
The method is recursive and currently has an exponential complexity. We will improve this later on.
### Bitmasks
We can capture the available choices, our current state, using the binary representation of an integer. This is as there are only a maximum of 20 numbers to choose from, and each number can be available (1), or not available (0).

For example, here are some numbers and the set of choices that they represent:
```
 5: 0101 -> [3,1]
 7: 0111 -> [3,2,1]
 8: 1000 -> [4]
12: 1100 -> [4,3]
```

We will refer to the currently available choices as our **state**. To change and access the contents of the state, we will need to use some bit manipulation techniques.

Initially, we have the choice to choose **any number**: this is a number with all the choices set at '1'. If we have ```k``` choices, we would need ```k``` '1's, which is ```(1 << k) - 1```.

```
k = 3
  1<<3 = 1000
  1000 - 1 = 111

k = 5
  1<<5 = 100000
  100000 - 1 = 11111
```

We will need to check if a bit is on to see if we can **choose** it. With a ```state``` and ```num```, ```(1 << (num-1)) & state``` will be non-zero if it is on, and zero if it is off.

```
state = 110, num = 1
  (1<<(1-1)) = 001
  001 & 110 = 000 -> Num 1 bit is off

state = 110, num = 2
  (1<<(2-1)) = 010
  010 & 110 = 010 -> Num 2 bit is on
```

We will also need to turn a bit off to **remove** it from the choices. With a ```state``` and ```num```, ```~(1 << (num-1)) & state``` will return the state with the ```num``` bit off.

```
state = 111, num = 2
  (1<<(2-1)) = 010
  ~010 & 111 = 101 -> Removed the num 2 bit

state = 111, num = 3
  (1<<(3-1)) = 100
  ~100 & 111 = 011 -> Removed the num 3 bit
```

By using these bit manipultation techniques, our current C++ implementation of the ```canWin``` method is now below:

```c++
int maxChoice;

bool canWin(int target, int choices) {
    for (int choice = 1; choice <= maxChoice; choice++) { // Go through each of the possible choices
        if (((1 << (choice - 1)) & choices != 0) { // If the bit is on
            int nextChoices = ~(1 << (choice - 1)) & choices; // Turn this choice bit off
            int nextTarget = target - choice;

            if (nextTarget <= 0) return true;
            if (canWin(nextTarget, nextChoices) == false) return true; // If the opponent loses, you win
        }
    }
    return false; // If you cannot force a win, you lose
}
```

### Memoization
We need to memoize this function so we do not recalculate visited states. The key is that we **only** need to remember the choices that are available left. 
We can introduce a ```dp``` array, where ```dp[state]``` is -1 if ```state``` has not been calculated, 0 if the player loses, and 1 if the player wins. The size of this array needs to be at least ```2^20```.

### Gotchas
Don't forget about the edge cases! If the target is too high even with all the choices, the player cannot win.

## Complexity
As there are ```2^maxChoosableInteger``` states, the memory complexity is ```O(2^maxChoosableInteger)```.  
In each of the ```2^maxChoosableInteger``` states, there is a for loop of size ```maxChoosableInteger```, resulting to a total runtime complexity of ```O(maxChoosableInteger * 2^maxChoosableInteger)```


## Full Code Solution
```c++
class Solution {
public:
    int maxChoice;
    int dp[1<<20];

    bool canWin(int target, int choices) {
        if (dp[choices] != -1) return dp[choices]; // If the state has been visited, return the answer!
        
        for (int choice = 1; choice <= maxChoice; choice++) { // Go through each of the choices
            if (((1 << (choice - 1)) & choices) != 0) { // If the bit is on
                int nextChoices = ~(1 << (choice - 1)) & choices; // Turn this choice bit off
                int nextTarget = target - choice;

                if (nextTarget <= 0) {
                    dp[choices] = 1; // Memoize! 
                    return true;
                }
                if (canWin(nextTarget, nextChoices) == false) { // If the opponent loses, you win
                    dp[choices] = 1; // Memoize!
                    return true;
                }
            }
        }
        dp[choices] = 0; // Memoize!
        return false; // If you cannot force a win, you lose
    }
    
    bool canIWin(int maxChoosableInteger, int desiredTotal) {
        int totalAchievable = 0;
        for (int i = 1; i <= maxChoosableInteger; i++) {
            totalAchievable += i;
        }
        if (totalAchievable < desiredTotal) return false; // If the desiredTotal is larger than the sum of all the choices, you cannot win
        
        memset(dp, -1, sizeof(dp)); // Initialise the dp array to -1
        maxChoice = maxChoosableInteger; 
        return canWin(desiredTotal, (1 << maxChoosableInteger) - 1); // Start with a full bitmask of '1's
    }
};
```