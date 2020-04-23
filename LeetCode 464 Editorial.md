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
The question feels like a standard dynamic programming question, where we see if it is possible to reach a target with a set of choices. However, the restriction where we cannot choose repeated values means we have to introduce a way to remember which numbers are left to choose from. The small ```maxChoosableInteger``` value means that we can remember the **set** of which numbers are left.

### Tags
Dynamic Programming, Bit Manipulation, Minimax

## Solution (Dynamic Programming, with Bitmasks)

The question asks if a player can reach the **target** if they are provided with a **set of numbers** to choose from. When the player makes a choice, the target decreases and the set of numbers reduces, and the problem is asked from the other player's perspective.

For example, if the **target** is 5, and the **choices** are [1,2,3], the three choices that the player has are:  
```
Take the 1: (target: 5-1=4, choices: [2,3])  
Take the 2: (target: 5-2=3, choices: [1,3])  
Take the 3: (target: 5-3=2, choices: [1,2])
```
Notice that three subproblems were formed, with lower target values and smaller range of choices. The appearance of **repeated subproblems** hints that this question should be approached using dynamic programming.

### Minimax
How do we know if the player will win or lose? In a two player game, you can win if any of your moves **force** the opponent to lose; you will lose if you **cannot** force them to lose. Of course, in this game, you will also win if you can decrease the target to 0 or below.

Let's implement this approach in pseudocode by introducing a method/function that returns whether the current player can win or not, given those parameters. 

```
bool canWin(target, choices):
    for choice in choices:
        let nextChoices = choices \ choice # All the choices except choice
        let nextTarget = target - choice

        if (nextTarget <= 0): # If you can reduce the target to 0 or below, you win!
            return true

        if (canWin(nextTarget, nextChoices) == false): # If you can force the opponent to lose, you win!
            return true

    return false
```

### Bitmasks
How do we efficiently remember the states? Since there are only a maximum of 20 choices, and we have to remember whether we can use it (1) or not (0), we can use a bitmask.

We will use the binary representation of a number to represent which numbers we can choose: if the bit is on, we can choose it. 

For example:
```
 5: 0101 -> [3,1]
 7: 0111 -> [3,2,1]
 8: 1000 -> [4]
12: 1100 -> [4,3]
```

We will need to use some bit manipulation techniques.

Initially, we have the choice to choose **any number**: this is a number with all choices set at '1'. If we have ```k``` choices, we would need ```k``` '1's, which is ```(1 << k) - 1```.

We will need to check if a bit is on to check if we can **choose** it. With a ```state``` and ```num```, ```(1 << (num-1)) & state``` will be non-zero if it is on, and 0 if it is off.

We will also need to turn a bit off to **remove** it from the choices. With a ```state``` and ```num```, ```~(1 << (num-1)) & state``` will return the state with the ```num``` bit off.

By including these bitmasking techniques, our current C++ implementation of the ```canWin``` method is now below:

```c++
int maxChoice;

bool canWin(int target, int choices) {
    for (int choice = 1; choice <= maxChoice; choice++) { // Go through each of the possible choices
        if (((1 << (choice - 1)) & choices != 0) { // If the bit is on
            int nextChoices = ~(1 << (choice - 1)) & choices;
            int nextTarget = target - choice;

            if (nextTarget <= 0) return true;
            if (canWin(nextTarget, nextChoices) == false) return true; // If the opponent loses, you win
        }
    }
    return false; // If you cannot force a win, you lose
}
```

### Memoization
We need to memoize this function so we do not recalculate visited states. The key is to notice that we **only** need to remember the choices that are available left. 
We can introduce a ```dp``` array that is set to -1 if it has not been visited, 0 if the player loses, and 1 if the player wins. The size of this array needs to be at least 2^20.

## Complexity
As there are ```2^maxChoosableInteger``` states, the memory complexity is ```O(2^maxChoosableInteger)```.  
In each of the ```2^maxChoosableInteger``` states, there is a for loop of size ```maxChoosableInteger```, resulting to a total runtime complexity of ```O(maxChoosableInteger * 2^maxChoosableInteger)```

## Gotchas
Don't forget about the edge cases! If the target is too high, even with all the choices, the player cannot win.


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
                int nextChoices = ~(1 << (choice - 1)) & choices;
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
        return false;
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