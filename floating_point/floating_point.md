# Floating Point

I've seen a few people struggle to solve problems that require understanding the intricacies of floating point.
I'd like to cover what I know with some examples.

> [!IMPORTANT]  
> This page is still a work in progress, it may have incomplete or incorrect information

> [!NOTE]
> This article will constrain itself to the most common IEEE754 binary floats, other kinds of floating point numbers do exist, even decimal floats also exist, but it is rare to encounter anything other than the standard binary ones.

[The Basics](#the-basics)

[The Details](#the-details)

[Example: Order preserving unsigned integer conversion](#order-preserving-unsigned-integer-conversion)

## The Basics
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it); the fundamental structure is the same, and could be described as “scientific notation in binary”.
### Decimal scientific notation
You may be familiar with decimal scientific notation, but if not a quick refresher:

$$\large s M \times 10^E$$

`s` the "sign" of the number; positive or negative. Very often only negative numbers will be marked, positive being the default\
`M` the "mantissa" which must be greater than or equal to one and less than ten  $$\{1 \leq m < 10\}$$\
*the mantissa is also sometimes called the "significand" or "coefficient".*\
This range for the mantissa is for consistency purposes, a value above ten or below one could be represented with a number that **is** within this range but has a higher or lower exponent, for example:  $0.9 = 9.0\times10^{-1}$ and $11 = 1.1\times10^1$ ao the number with a single (nonzero) digit above the decimal place is the standard.\
`E` the "expontent" which can be any positive or negative **integer**.\
Keeping in mind that $10^0=1$, $10^{-1}=0.1$ and so on.


This allows any number to be constructed, for example:

$-9876.43 = -9.87643 \times 10^3$\
$6.13 = +6.13 \times 10^0$\
$0.0405 = +4.05 \times 10^{-2}$

### Binary scientific notation
Not much changes in base 2 (binary).\
$$\large s M \times 2^E$$

Though from here you can expect to see numbers in binary a fair bit, they will be marked approproiately; $NUMBER_b$ , as such:\
$$\large s M \times 10_b \, ^E$$

`s` the sign has exactly the same meaning as in decimal.\
`M` the mantissa must now be greater than or equal to one and less than two  $\{1 \leq m < 2\}$ which is $\{1_b \leq m < 10_b\}$ for the same reasons that the decimal range is constrained.\
 *Note that this means the leading binary digit of the mantissa will **always** be 1* \
`E` the expontent can again be any positive or negative **integer**.


### Storage layout
#### Sign bit
To represent the "sign" of the number, positive or negative just a single bit is needed.\
For all floats you will encounter this will be the top bit, 0 will signify the number is positive, 1 will signify it is negative.

#### Exponent bits
The exponent will be stored in the next few bits which form an unsigned integer $E_{storage}$ which is the actual exponent `e` after a bias $E_{bias}$ has been applied\
$$E = E_{storage} - E_{bias}$$

Note that the min ([all zeros](#sub-normal-numbers)) and max ([all ones](#inf-&-nan)) values for the stored exponent $E_{storage}$ have specicial meanings which don't conform to the above equation for the exponent `e`. As a result the min and max represntable exponents ($E_{min}$ & $E_{max}$) do not include values which would require all zeros or all ones in the exponent bits. The bias for the standard formats is the middle of representable values (rounded down). 

> [!NOTE]
> The min and max here form an *inclusive* range.

|format|bit count|bias($E_{bias}$)|$E_{min}$|$E_{max}$|
|---|---|---|---|---|
|`half`|5|15|-14|15|
|`float`|8|127|-126|127|
|`double`|11|1023|-1022|1023|

#### Mantissa bits


## The Details
### Sub-normal numbers
### INF & NAN

## Order preserving unsigned integer conversion

