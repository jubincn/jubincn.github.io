# Programming Model Selected questions
## What is the value of Math.abs(-2147483648)?
Ans. -2147483648

-2147483648 is represented as 10000000000000000000000000000000 in binary. The negation in Java is do in two steps: 1. convert to its complementary format. 2. add 1. The negation of -2147483648 will be:
1. Convert to complementary: 01111111111111111111111111111111
2. Add 1: 10000000000000000000000000000000

## What is the result of division and remainder for negative integers?
The quotient a/b rounds toward 0; the remainder a % b is defined such that (a / b) * b + a % b is always equal to a. For example, -14/3 and 14/-3 are both -4, but -14 % 3 is -2 and 14 % -3 is 2.
