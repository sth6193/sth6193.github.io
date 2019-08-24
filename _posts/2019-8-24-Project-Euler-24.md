---
layout:     post
title:      Recursion, C++, and Project Euler 24
date:       2019-8-24 12:00:00
summary:    Project Euler is a great site that hosts math problems that can often be solved using intuition and programming. Here I solve their 24th problem, finding the millionth ordering of [0,1,2,3,4,5,6,7,8,9], in a more general fashion using recursive functions in C++.
categories: Projects
---

### The Problem

Project Euler is a [website](https://projecteuler.net/about) that hosts mathematical problems which often can be solved using some math knowledge and computer programming. I was looking for a small project to do for a refresher in C++, so I decided to try and code up one of their problems. I was scrolling through and found the [24th problem](https://projecteuler.net/problem=24), which I found interesting since it reminded me of a combinatorics class I took during undergrad. The problem goes:

*A permutation is an ordered arrangement of objects. For example, 3124 is one possible permutation of the digits 1, 2, 3 and 4. If all of the permutations are listed numerically or alphabetically, we call it lexicographic order. The lexicographic permutations of 0, 1 and 2 are:*

<p align="center">
012   021   102   120   201   210
</p>

*What is the millionth lexicographic permutation of the digits 0, 1, 2, 3, 4, 5, 6, 7, 8 and 9?*

This problem is not particularly difficult, as we could just enumerate all the combinations, sort, and take the millionth element. However, this approach is not very elegant or efficient as creating a vector with \\(10! \\), or roughly 3.6 million, elements takes up a lot of computer memory and time. And if we want to solve this on a larger ordered set other than the digits 0-9, like say all hexadecimal digits, then this approach requires too much memory to solve. Is there a more sophisticated way of solving this problem?

### The Solution

This problem is fundamentally about the number of permutations of an ordered list. These kind of problems belong to a subfield of math known as combinatorics, which is more or less the study of counting. One of the first functions you learn about in combinatorics is the factorial. Factorials are written as a number with an exclamation point, like \\(10! \\). A factorial, say \\( n! \\), counts the number of orderings of \\(n \\) distinct items and is defined as the product of all integers less than or equal to \\( n \\).

To see why a factorial is defined this way, consider having to line 10 people up into a row. You have 10 choices for the first person, 9 choices for the second, 8 choices for the third, and so on until only one person is left. As these decisions rely on the previous choices, to count the number of ways you can order people you have to multiple the number of possibilities for each choice. This multiplication of 10 times 9 times 8 and so on down to one is exactly \\( 10! \\).

As there are 10 unique digits in this problem, there are \\(10! \\) different possible permutations. For choosing the most significant digit, we have ten choices, 0-9. For each of these choices, there are \\(9! \\), or 362,880, possible combinations of the remaining digits. However, if we choose 0 as the first digit, we are stuck between the first/lowest ordering and the 362,880th ordering. If we choose 1 as the first digit, we are stuck between the 362,881st ordering and the 725,760th ordering. Similar ranges of orderings are true for all of the other digits we can choose for the most significant digit.

If we know which ordering we want, in this case the millionth ordering, we can find which digit must appear first. If we choose 2 to be the first digit, then the range of orderings possible is from the 725,761st ordering to the 1,088,640th ordering. Therefore the first digit must be a 2. Selection of the next digit breaks this initial range, 725,761 to 1,088,640, into 9 smaller ranges. The range containing the millionth ordering indicates the next digit we need to use. However, as 2 is no longer an option since it cannot appear twice. Therefore, if we find the millionth ordering to be in the 7th bracket, we need to use the 7th element in the array of remaining digits [0,1,3,4,5,6,7,8,9].

With this method, we can perform the same process on the list of remaining digits until all digits have been selected and we found the millionth ordering. But how can turn this algorithm into code?

### The Code

##### Recursive Factorial
When applying the same process continually until reaching a stopping condition, like finding the correct digit for an ordering until there is only one digit left, the process can be described recursively. Recursive processes are self-referential, where a step in the process is to repeat the process again. This concept can often be confusing, but is very important in math and some programming applications.

In code, recursive functions are functions which contain calls to themselves within the function. One of the simplest recursive functions to code is actually the factorial:
```cpp
int factorial(int n){
  if (n<=1)
    return n=1;
  else
    return n*factorial(n-1);
}
```

Within this function, the factorial function is called within the return statement. It will continue to be called until the base case is reached, which for this function is when the input is equal to 1 or 0. To get to 1, from say 10, 10 will be multiplied by factorial(9), which will return 9 times factorial(8), and so on. Once reaching the base case, the nested returns will all be multiplied together and return the correct value for the factorial of 10.

##### Recursive Solution

Remarkably, the algorithm I described is also a repeated process and therefore can be coded as a recursive function. I'll put the `findDigits` function below and then describe it:
```cpp
long long int findDigits(vector<int> integerList, long long int nthElement){
  int listSize=integerList.size();
  long long int baseFactorial=factorial(listSize-1);

  if (listSize==1){
    return integerList[0];
  }
  else{
    for(int i=0; i<=listSize; i++){
      if(baseFactorial*(i+1)>nthElement){
        long long int digit=integerList[i];
        integerList.erase(integerList.begin()+i);
        return (long long) digit*tenToTheN(listSize-1) + findDigits(integerList,nthElement-baseFactorial*(i));
      }
    }
    return 0;
  }
```

First off, this function does not just find the millionth ordering of a permutation of the digits 0-9, but takes in any ordered subset of these digits and finds the nth element (given as the input `nthElement`).

I first get the number of digits in the list, which allows me to determine how many permutations there are for each choice of digit. Then I define the base case in an **if** statement, saying if there is only one digit left we can return that digit. If there is more than one digit left, the **else** block runs. The **else** block looks at the lowest ordering for each digit choice, and when it is larger than the `nthElement` we know that the previous digit is the one we need to use. That digit is stored in `digit` and removed from the list of possible digits `integerList`. Finally, I return the digit times the significance of that digit, which can be determined from the number of remaining digits, plus another function call of `findDigits` with the updated list of remaining digits and a new value of `nthElement` representing the fact that we at least have the lowest ordering associated with the digit just chosen.

Some technical notes: I created a function `tenToTheN` which allows me to get \\(10^n \\) as an integer. If I wanted to use a preexisting library, I would have to have used floats. And second, the number I end up returning is actually quite a large integer. Using 32-bit integers has a max value of 2,147,483,647 , which is smaller than the largest possible permutation 9,876,543,210. To fix this, I use long long ints, which forces 64-bit integers. There are definitely ways around this, like storing the digits as strings instead of integers, but this was the quick and easy fix.

### The Results

I first run some sanity checks on my functions to make sure they are working as I expect. The output of this is:
```console
Test factorial of 5! = 120
Test of 10 to power 5 = 100000
1st ordering of [0,1,2,3] = 123
Highest ordering of [0,1,2,3] = 3210
Twelfth ordering of [0,1,2,3] = 1320
```

Great! All of these results are as expected. The first and last orderings of [0,1,2,3] are easily confirmed. The 12th ordering is also as expected, since \\(3! \\)=6 which forces the highest ordering of three remaining digits after choosing 1 as the most significant digit.

Finally, running the code to solve the problem gives:

```console
Millionth ordering of [0,1,2,3,4,5,6,7,8,9] = 2783915460
```

With the added sanity check that we already know that 2 has to be the first digit.

Thanks for reading, hope you found this interesting!
