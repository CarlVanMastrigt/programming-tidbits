# Floating Point

I've seen a few people struggle to solve problems that require understanding the intricacies of floating point.
I'd like to cover what I know with some examples.

> [!IMPORTANT]  
> This page is still a work in progress, it may have incomplete or incorrect information

> [!NOTE]
> This article will constrain itself to the most common IEEE754 binary floats, other kinds of floating point numbers do exist, but it is rare to encounter anything other than the standard binary ones.

> [!NOTE]
> It will be important to show numbers in binary a fair bit, they will be marked with a subscript b ($\text{NUMBER}_b$), when not marked in this way a number will be decimal, or it won't matter for the particular number, such as 0 and 1

## The Basics
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it) ; the fundamental structure is the same, and could be described as “[scientific notation](#decimal-scientific-notation) in [binary](#binary-scientific-notation)”.

$$\large s M \times 2^E$$

#### Sign: s

To represent the "sign" of the number, positive or negative, a single bit is needed.\
For all floats you will encounter this will be the top (most significant in storage) bit.\
`0` signifies the number is positive, `1` signifies it is negative.\
This can alternatively be thought of as multiplying by 1 or negative 1.

$$
s=
\left\lbrace
	\begin{array}{lr}
		-1&, \text{if } s_{\text{bit}} = 1 \\
		+1&, \text{if } s_{\text{bit}} = 0
	\end{array}
\right\rbrace
$$

#### Exponent: E

|Format|E bit count|Bias $E_{\text{bias}}$|$E_{\text{min}}$|$E_{\text{max}}$|
|:---|:---|:---|:---|:---|
|`half`|5|15|-14 <br>$\small E_{\text{storage}} = 00001_b$|15 <br>$\small E_{\text{storage}} = 11110_b$|
|`float`|8|127|-126 <br>$\small E_{\text{storage}} = 00000001_b$|127 <br>$\small E_{\text{storage}} = 11111110_b$|
|`double`|11|1023|-1022 <br>$\small E_{\text{storage}} = 00000000001_b$|1023 <br>$\small E_{\text{storage}} = 11111111110_b$|

The exponent will be stored after the sign; in the next ($\text{E bit count}$) most significant bits.\
It can be treated as an unsigned integer $E_{\text{storage}}$ from which the **actual** exponent `E` is calculated by applying a bias ($E_{\text{bias}}$)

$$E = E_{\text{storage}} - E_{\text{bias}}$$

Not all values for $E_{\text{storage}}$ follow this scheme though; [all zeros](#sub-normal-numbers) and [all ones](#inf-&-nan) are special cases that behave differently. Every other value for $E_{\text{storage}}$ will fit this scheme though, and will be a "normal" floating point number.

$E_{\text{min}}$ is the smallest exponent a given floating point type can have, it corresponds to 1 in $E_{\text{storage}}$ which gives: $E_{\text{min}} = 1 - E_{\text{bias}}$

$E_{\text{max}}$ is likewise the largest exponent, with $E_{\text{storage}}$ being all ones *except* the lowest bit: $E_{\text{max}} = (2^{\text{E bit count}} - 2) - E_{\text{bias}}$

So for "normal" floating point numbers:

$$ 1 \leq E_{\text{storage}} \leq 2^{\text{E bit count}} - 2 $$

$$ E_{\text{min}} \leq E \leq E_{\text{max}} $$

The value for $E_{\text{bias}}$ for each format balances the ability to represent numbers with both very large and very small magnitudes. But there is also a [reason](#calculating-the-exponent-bias) in the specification for the values of $E_{\text{bias}}$ listed.

#### Mantissa: M

|Format|M bit count|
|:---|:---|
|`half`|10|
|`float`|23|
|`double`|52|

$$0 \leq M_{\text{storage}} < 2^{\text{M bit count}}$$

All remaining bits are used by the mantissa. Because the leading digit will [*almost*](#sub-normal-numbers) always be 1, it is not stored to avoid wasting space. Instead any normal floating point number is treated as having an implicit leading 1. 
The stored bits thus represent only the binary digits following the dot in the mantissa.

Another way to think of this is as storing the fractional part of the mantissa:

$$ M = 1 + \frac {M_{\text{storage}}} {2^{\text{M bit count}}}$$

A couple of examples may help; `half` has 10 bits for mantissa storage, so for `half` numbers:

$$ M  =  1 + \frac{M_{\text{storage}}} {1024}$$

|$M_{\text{storage}}$|$\text{Equation}$|$M_{\text{decimal}}$|$M_{\text{binary}}$|
|:---|:---|:---|:---|
|$0000000000_b$|$M  = 1 + \frac{0} {1024}$|$1.0$|$1.0000000000_b$|
|$1000000000_b$|$M  = 1 + \frac{512} {1024}$|$1.5$|$1.1000000000_b$|
|$0000000011_b$|$M  = 1 + \frac{3} {1024}$|$1.0029296875$|$1.0000000011_b$|
|$1100000001_b$|$M  = 1 + \frac{769} {1024}$|$1.7509765625$|$1.1100000001_b$|
|$1111111111_b$|$M  = 1 + \frac{1023} {1024}$|$1.9990234375$|$1.1111111111_b$|

Hopefully the $M_{\text{binary}}$ values illustrate just how uninteresting whats going on really is.

#### Putting the bits together

For any normal floating point number the equation we get by putting the above parts together is:

$$
\left\lbrace
	\begin{array}{lr}
		-1&, \text{if } s_{\text{bit}} = 1 \\
		+1&, \text{if } s_{\text{bit}} = 0
	\end{array}
\right\rbrace \times (1 + \frac {M_{\text{storage}}} {2^{\text{M bit count}}}) \times \Large 2^{E_{\text{storage}} - E_{\text{bias}}} $$

Some examples, again using `half` to keep them compact and remembering that for `half` $2^{\text{M bit count}} = 1024$ and $E_{\text{bias}} = 15$ :

|$\text{Storage}$|$\text{Equation}$|$\text{Value}$|
|:---|:---|:---|
|$1\ 01111\ 0000000000_b$|$-1 \times (1 + \frac{0} {1024}) \times 2^{15-15}$|$-1.0$|
|$0\ 10000\ 1000000000_b$|$+1 \times (1 + \frac{512} {1024}) \times 2^{16-15}$|$3.0$|
|$1\ 10001\ 0001010000_b$|$-1 \times (1 + \frac{80} {1024}) \times 2^{17-15}$|$-4.3125$|
|$0\ 11110\ 1111111111_b$|$+1 \times (1 + \frac{1023} {1024}) \times 2^{30-15}$|$65504.0$|
|$0\ 00001\ 0000000000_b$|$+1 \times (1 + \frac{0} {1024}) \times 2^{1-15}$|$0.00006103515625$|

*Note that the last two examples have the largest exponent* ($E_{\text{max}}$) *with the largest mantissa and the smallest exponent* ($E_{\text{emin}}$) *with the smallest mantissa, these result in the largest and smallest possible magnitudes for a "normal"* `half`


## The Details

### Sub-normal numbers

#### Zero

### INF & NAN

### Representable numbers
### Precision loss

## Interesting applications 

### Order preserving unsigned integer conversion

## Extras

### Decimal scientific notation

Scientific notation is a way of representing both very large and very small numbers in a realtively easy to read fashion (once you get used to it) that is consistent. In practice you will likely only encounter **decimal** scientific notation.
 
$$\large s M \times 10^E$$

- `s` the "sign" of the number; positive or negative. Very often only negative numbers will be marked, positive being the default.

- `M` the "mantissa" which must be greater than or equal to one and less than the decimal base (ten) $\{1 \leq M < 10\}$

*The mantissa is also sometimes called the "significand" or "coefficient".*

*The range of* $\{1 \leq M < 10\}$ *for the mantissa is for consistency purposes; any value less than one or larger than the base (ten) could be represented using a mantissa that **is** within this range but has a higher or lower exponent,* for example:  $0.9 = 9.0\times10^{-1}$ and $11 = 1.1\times10^1$ *choosing a mantissa with a single (nonzero) digit above the decimal place is the standard. That is; a mantissa greater than or equal to one and less than the base.*

- `E` the "expontent" which can be any positive or negative **integer**.

Keeping in mind that: $10^0=1 ,  10^{2}=100 ,  10^{-1}=0.1$ and so on.

Some examples:

|Regular|Scientific notation|
|:---:|:---:|
| $-9876.43$ | $-9.87643 \times 10^3$ |
| $6.13$ | $6.13 \times 10^0$ |
| $-0.0405$ | $-4.05 \times 10^{-2}$ |

### Binary scientific notation
Not much changes in base 2 (binary) scientific notation.

$$\large s M \times 2^E$$

- `s` the sign, positive or negative.

- `M` a binary mantissa must be greater than or equal to one and less than two  $\{1 \leq M < 2\}$ once again between one and the "base" (two).\
 *Note that this means the leading digit of a binary mantissa will be 1 unless the number is exactly zero*

- `E` the expontent can again be any positive or negative **integer**.\
Keeping in mind that $2^0=1 , 2^{3}=8 , 2^{-2}=0.25$ and so on.

### Calculating the exponent bias

First remember that:

$E_{\text{min}} = 1 - E_{\text{bias}}$

$E_{\text{max}} = (2^{\text{E bit count}} - 2) - E_{\text{bias}}$

The specification for floating point actually requires that $E_{\text{min}} = 1 − E_{\text{max}}$ is satisfied, which means there is a required $E_{\text{bias}}$ which can be calculated by substituting $E_{\text{min}}$ and $E_{\text{max}}$ in terms of the bias:

$$
\begin{split}
E_{\text{min}} & = 1 − E_{\text{max}}\\
1 - E_{\text{bias}} & = 1 - (2^{\text{E bit count}} - 2 - E_{\text{bias}})\\
E_{\text{bias}} & = 2^{\text{E bit count}} - 2 - E_{\text{bias}}\\
2 \times E_{\text{bias}} & = 2^{\text{E bit count}} - 2 \\
E_{\text{bias}} & = \frac{2^{\text{E bit count}}}{2} - 1 \\
E_{\text{bias}} & = 2^{\text{E bit count} - 1} - 1 \\
\end{split}
$$

This is the middle of representable values, rounded such that $E_{max}$ will be one larger. The reason to bias it in this way is that numbers smaller than the normal floating point range are actually still [representable](#sub-normal-numbers), just with a lower precision. Whereas numbers larger than the maximum exponent are not representable at all. 
