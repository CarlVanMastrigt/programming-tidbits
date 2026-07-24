# Floating Point

I've seen a few people struggle to solve problems that require understanding the intricacies of floating point.
I'd like to cover what I know with some examples.

> [!IMPORTANT]  
> This page is still a work in progress, it may have incomplete or incorrect information

> [!NOTE]
> This article will constrain itself to the most common IEEE754 binary floats, other kinds of floating point numbers do exist, even decimal floats also exist, but it is rare to encounter anything other than the standard binary ones.

> [!NOTE]
> It will be important to show numbers in binary a fair bit, they will be marked with a subscript b ($NUMBER_b$), when not marked in this way a number will be decimal (or it won't matter, such as 0 and 1)

[The Basics](#the-basics)

[The Details](#the-details)

[Example: Order preserving unsigned integer conversion](#order-preserving-unsigned-integer-conversion)

## The Basics
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it); the fundamental structure is the same, and could be described as “scientific notation in binary”.

### Decimal scientific notation
You may be familiar with decimal scientific notation, but if not a quick refresher:
 
$$\large s M \times 10^E$$

- `s` the "sign" of the number; positive or negative. Very often only negative numbers will be marked, positive being the assumed default.

- `M` the "mantissa" which must be greater than or equal to one and less than the decimal base (ten) $$\{1 \leq M < 10\}$$

*The mantissa is also sometimes called the "significand" or "coefficient".*\
*This range for the mantissa is for consistency purposes; any value less than one or larger than the base (ten) could be represented using a mantissa that **is** within this range but has a higher or lower exponent, for example:  $0.9 = 9.0\times10^{-1}$ and $11 = 1.1\times10^1$ choosing a mantissa with a single (nonzero) digit above the decimal place is the standard, that is a mantissa greater than or equal to one and less than the base.*

- `E` the "expontent" which can be any positive or negative **integer**.\
Keeping in mind that $10^0=1 ,  10^{2}=100 ,  10^{-1}=0.1$ and so on.


Some examples:

|Regular|Scientific notation|
|:---:|:---:|
| $-9876.43$ | $-9.87643 \times 10^3$ |
| $6.13$ | $+6.13 \times 10^0$ |
| $0.0405$ | $+4.05 \times 10^{-2}$ |

### Binary scientific notation
Not much changes in base 2 (binary) scientific notation.

$$\large s M \times 2^E$$

- `s` indicates the sign, positive or negative

- `M` the mantissa (as opposed to decimal) must be greater than or equal to one and less than two  $\{1 \leq M < 2\}$ once again between one and the "base" (two).\
 *Note that this means the leading binary digit of the mantissa will **always** be 1 unless the number is zero*

- `E` the expontent can again be any positive or negative **integer**.\
Keeping in mind that $2^0=1 ,  2^{3}=8 ,  2^{-2}=0.25$ and so on.


### Storage layout
#### Sign bit
To represent the "sign" of the number, positive or negative, a single bit is needed.\
For all floats you will encounter this will be the top (most significat in storage) bit, `0` signifies the number is positive, `1` signifies it is negative.

#### Exponent bits

|Format|Bit count|Bias($E_{bias}$)|$E_{min}$|$E_{max}$|
|:---|:---|:---|:---|:---|
|`half`|5|15|-14 ($\small 00001_b$)|15 ($\small 11110_b$)|
|`float`|8|127|-126 ($\small00000001_b$)|127 ($\small 11111110_b$)|
|`double`|11|1023|-1022 ($\small 00000000001_b$)|1023 ($\small 11111111110_b$)|

The exponent will be stored in the next few bits which form an unsigned integer $E_{storage}$ from which the *actual* exponent `E` is calculated by applying a bias ($E_{bias}$)

$$E = E_{storage} - E_{bias}$$

The bias for the standard formats is the middle of representable values (rounded down) such that $E_{min} = 1 − E_{max}$ is satisfied.

Note that the min ([all zeros](#sub-normal-numbers)) and max ([all ones](#inf-&-nan)) values possible for the exponent bits $E_{storage}$ have specicial meanings which don't conform to the above equation for the exponent. As a result the valid range of exponents $E_{min} \leq E \leq E_{max}$) does not include these values.

#### Mantissa bits

|Format|Bit count|
|:---|:---|
|`half`|10|
|`float`|23|
|`double`|52|

All remaining bits are used by the mantissa. Because the leading digit will [almost always](#the-details) be 1, it is not stored to avoid wasting space. Instead any floating point number that does not have all zeros or all ones in $E_{storage}$ is treated as having an implicit leading 1. 
The stored bits thus represent the binary digits following the dot in the mantissa, with the most significant digit having the highest bit.

Another way to think of it is as storing the fractional part of the mantissa.

$$ M = 1 + \frac {M_{storage}} {2^{bit\ count}}$$

A couple of examples may help; `half` has a 10 bit mantissa, so for `half` numbers:

$$ M  =  1 + \frac{M_{storage}} {1024}$$

|$M_{storage}$|$Equation$|$M_{decimal}$|$M_{binary}$|
|:---|:---|:---|:---|
|$0000000000_b$|$M  = 1 + \frac{0} {1024}$|$1.0$|$1.0000000000_b$|
|$1000000000_b$|$M  = 1 + \frac{512} {1024}$|$1.5$|$1.1000000000_b$|
|$0000000011_b$|$M  = 1 + \frac{3} {1024}$|$1.0029296875$|$1.0000000011_b$|
|$1100000001_b$|$M  = 1 + \frac{513} {1024}$|$1.7509765625$|$1.1100000001_b$|
|$1111111111_b$|$M  = 1 + \frac{1023} {1024}$|$1.9990234375$|$1.1111111111_b$|

Hopefully the $M_{binary}$ values reveal just how uninteresting whats going on really is.


## The Details
### Sub-normal numbers
### INF & NAN
### Representable numbers
### Precision loss

## Order preserving unsigned integer conversion

