# Floating Point

I've seen a few people struggle to solve problems that require understanding the intricacies of floating point.
I'd like to cover what I know with some examples.
> [!NOTE]
> This discussion will constrain itself to binary floats and the, decimal floats also exist, but are very rare.

[The Basics](#the-basics)

[The Details](#the-details)

[Example: Order preserving unsigned integer conversion](#order-preserving-unsigned-integer-conversion)

## The Basics
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it); the fundamental structure is the same, and could be described as “scientific notation in binary”.
### Decimal scientific notation
You may be familiar with decimal scientific notation, but if not a quick refresher:

$$\large s \times m \times 10^e$$

`s` the "sign" of the number; positive or negative. Very often only negative numbers will be marked, positive being the default/\
`m` the "mantissa" which must be greater than or equal to one and less than 10: $\{1 \leq m < 10\}$ this is sometimes called the "significand" or "coefficient".\
`e` the "expontent" which can be any positive or negative **integer**.\
Keeping in mind that $10^0 = 1$, $10^{-1} = 0.1$ and so on.

This allows any number to be constructed, for example:\
$-9876.43 = -9.87643 \times 10^3$\
$6.13 = +6.13 \times 10^0$\
$0.0405 = +4.05 \times 10^{-2}$

### Binary scientific notation
Not much changes in base 2 (binary)
$$\large s \times m \times 2^e$$

`s` the sign has exactly the same meaning as in decimal.\
`m` the mantissa must now be greater than or equal to 1 and less than 2 $\{1 \leq m < 2\}$.\
*Note that this means the leading binary digit of the mantissa will **always** be 1*\
`e` the expontent can again be any positive or negative **integer**.\

### Storage layout
To represent the "sign" of the number, plus or minus $\pm$ a single bit is needed.\
For all floats you will encounter this will be the top bit, 0 will signify the number is positive, 1 will signify it is negative.\

The exponent will be stored in the next few bits (8 for `float`, 11 for `double` and 5 for `half`).\
Those bits form an unsigned integer E<sub>s</sub> which is the actual exponent `e` with a bias E<sub>b</sub> applied $e = E_{s} - E_{b}$\

## The Details
### Sub-normal numbers
### INF & NAN

## Order preserving unsigned integer conversion

