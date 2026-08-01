# Floating Point

I've seen a few people struggle to solve problems that require understanding the intricacies of floating point.
I'd like to cover what I know with some [examples](##interesting-applications).

> [!IMPORTANT]  
> This page is still a work in progress, it may have incomplete or incorrect information

> [!NOTE]
> This article will constrain itself to the most common IEEE754 binary floats, other kinds of floating point numbers do exist, but it is rare to encounter anything other than the standard binary ones.

> [!NOTE]
> It will be important to show numbers in binary a fair bit, they will be marked with a subscript b ($\text{NUMBER}_b$), when not marked in this way a number will be decimal, or it won't matter for the particular number, such as 0 and 1

## The Basics
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it) ; the fundamental structure is the same, and could be described as “[scientific notation](#decimal-scientific-notation) in binary”.

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
It can be treated as an unsigned integer $E_{\text{storage}}$ from which the **actual** exponent `E` is calculated by applying a bias ($E_{\text{bias}}$, seen in the above table).

$$E = E_{\text{storage}} - E_{\text{bias}}$$

Not all values for $E_{\text{storage}}$ follow this scheme though; [all zeros](#sub-normal-numbers) and [all ones](#inf-&-nan) are special cases that behave differently. Every other value for $E_{\text{storage}}$ will fit this scheme though, and will be a "normal" floating point number.

$E_{\text{min}}$ is the smallest exponent a given floating point type can express, it corresponds to 1 in $E_{\text{storage}}$ which gives: $E_{\text{min}} = 1 - E_{\text{bias}}$

$E_{\text{max}}$ is likewise the largest exponent, with $E_{\text{storage}}$ being all ones *except* the lowest bit: $E_{\text{max}} = (2^{\text{E bit count}} - 2) - E_{\text{bias}}$

So for "normal" floating point numbers:

$$ 1 \leq E_{\text{storage}} \leq 2^{\text{E bit count}} - 2 $$

$$ E_{\text{min}} \leq E \leq E_{\text{max}} $$

The value of $E_{\text{bias}}$ for each type balances the ability to represent numbers with both very large and very small magnitudes. But there is also a [reason](#calculating-the-exponent-bias) in the specification for the values of $E_{\text{bias}}$ listed.

#### Mantissa: M

|Format|M bit count|
|:---|:---|
|`half`|10|
|`float`|23|
|`double`|52|

$$0 \leq M_{\text{storage}} < 2^{\text{M bit count}}$$

All remaining bits are used by the mantissa. Because the leading digit in binary scienticic notation [will always be 1](#binary-scientific-notation), it isn't stored so as to avoid wasting space. Instead every "normal" floating point number is treated as having an *implicit* leading 1.

The stored bits thus represent only the binary digits following the dot in the mantissa.

Another way to think of this is as storing the fractional part of the mantissa:

$$ M = 1 + \frac {M_{\text{storage}}} {2^{\text{M bit count}}}$$

A couple of examples may help; `half` has 10 bits for mantissa storage, so for `half` numbers:

$$ M  =  1 + \frac{M_{\text{storage}}} {1024}$$

|$M_{\text{storage}}$|$\text{Equation}$|$M_{\text{decimal}}$|$M_{\text{binary}}$|
|:---|:---|:---|:---|
|$0000000000_b$(0)|$M  = 1 + \frac{0} {1024}$|$1.0$|$1.0000000000_b$|
|$1000000000_b$(512)|$M  = 1 + \frac{512} {1024}$|$1.5$|$1.1000000000_b$|
|$0000000011_b$(3)|$M  = 1 + \frac{3} {1024}$|$1.0029296875$|$1.0000000011_b$|
|$1100000001_b$(769)|$M  = 1 + \frac{769} {1024}$|$1.7509765625$|$1.1100000001_b$|
|$1111111111_b$(1023)|$M  = 1 + \frac{1023} {1024}$|$1.9990234375$|$1.1111111111_b$|

Hopefully the $M_{\text{binary}}$ values illustrate just how uninteresting whats going on really is.

#### Putting the bits together

For any normal floating point number the equation we get by putting the above elements together is:

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

*Note that the last two examples have the largest exponent* ($E_{\text{max}}$) *with the largest possible mantissa and, the smallest exponent* ($E_{\text{emin}}$) *with the smallest possible mantissa. These combinations result in the largest and smallest possible magnitudes for a "normal"* `half` *respectively.*


## The Details

### Sub-normal numbers

#### Zero

### INF & NAN

### Representable numbers
### Precision loss

## Interesting applications

For each of these; I'd highly recommend thinking about the problem yourself for a bit first. You'll remember the solution and understand it better if you do.

### Order equivalent unsigned integer manipulated reinterpretation

The title is quite a mouthful, but the goal of this is to map every `float` (or `double`/`half`) (that isn't INF or NAN) to an unsigned integer of the appropriate size, in such a way that all comparisons (`<` `>` etc.) behave the same way for the unsigned integer as they would have for the original `float`.

#### Why would anyone want that though?

- On the GPU it can be a very handy tool when used used in conjunction with atomic operations. For example; you might want to find the bounding box that tightly contains the parts of your game world that are actually visible on screen. For that you want to record the min and max worldspace positions in X, Y and Z across all pixels on screen. There are ways to keep that data in float and reduce it in a complex post processing step, sure, but it would be more convenient to (at least in part) make use of the `atomic_min` and `atomic_max operations`. Alas, those operations only work for unsigned integers. You *could* rescale the numbers and apply a bias (to avoid negatives) then type cast your floats to unsigned ints, but you're going to have to pick a range of floats you want to handle **and** accept some loss of precision if you do it that way.\
Having a perfect 1:1 mapping for which `atomic_min` and `atomic_max` still work would make things so much simpler...
  - *At some point I may talk more about doing this efficiently and it's application in generating cascaded shadow maps.*

- When performing a radix sort you need to be able to split your data up into distinct ordered buckets; repeatedly moving all entries to their appropriate bucket, in multiple stages where the buckets for that stage correspond to some property that has more significance within the "complete" sort than the last stage.\
This is fairly simple in a 2 stage radix sort of unsigned integers. First you sort each element into one of $2^{16}$ buckets corresponding to the **bottom** (or *least significant*) 16 bits, then iterate through all those buckets in order and move their entries into a new set of buckets corresponding to the **top** (or *most significant*) 16 bits, then you stitch the buckets together in order and, hey presto, the resulting array is sorted.\
So, say you wanted to sort floats with a radix sort, it would be very useful if you could convert them to unsigned integers that had the same comparison order (so that the ordered buckets would still make sense).

- Use your imagination... This is definitely something useful to have in your toolbox.

#### How is it done?

I'll give the code in C, hopefully the bit casts aren't too unreadble or offensive:

To go from a `float` to a `uint`:
```
uint32_t float_to_uint(float f)
{
	uint32_t u = *(uint32_t*)&f;
	return u ^ ((u & 0x80000000) ? 0xFFFFFFFF : 0x80000000);
}
```

And to go from a `uint` back to the original `float` when you're done:
```
float uint_to_float(uint32_t u)
{
	u ^= ((u & 0x80000000) ? 0x80000000 : 0xFFFFFFFF);
	return *(float*)&u;
}
```

Not much to it is there?


#### But *WHY* does that work?

EXPLAIN

## Extras and "tidbits"

### Decimal scientific notation

Scientific notation is a way of representing both very large and very small numbers in a consistent way that is realtively easy to read (once you get used to it). In practice you are likely only ever encounter **decimal** scientific notation.
 
$$\large s M \times 10^E$$

- `s` the "sign" of the number; positive or negative. Very often only negative numbers will be marked, positive being the default.

- `M` the "mantissa" which must be greater than or equal to one and less than the decimal base (ten) $\{1 \leq M < 10\}$

*The mantissa is also sometimes called the "significand" or "coefficient".*

*The range of* $\{1 \leq M < 10\}$ *for the mantissa is for consistency purposes; any value less than one or larger than the base (ten) could be represented using a mantissa that **is** within this range but has a higher or lower exponent,* for example:  $0.9 = 9.0\times10^{-1}$ and $11 = 1.1\times10^1$ *choosing a mantissa with a single (nonzero) digit above the decimal place is the standard. That is; a mantissa greater than or equal to one and less than the base.*

- `E` the "expontent" which can be any positive or negative **integer**.

Keeping in mind that: $10^0=1 ,  10^{2}=100 ,  10^{-1}=0.1$ and so on.

Some examples:

|Standard notation|Scientific notation|
|:---:|:---:|
| $-9876.43$ | $-9.87643 \times 10^3$ |
| $6.13$ | $6.13 \times 10^0$ |
| $-0.0405$ | $-4.05 \times 10^{-2}$ |

As an aside: there are some benefits in separting out the exponent like this. For one, it makes judging the magnitude of numbers easier. Looking at:

$$ 12461100 \ , \ 3420200 \ , \ 269100 $$

It's perhaps a little difficult to judge which is larger, and by how much, but:

$$ 1.24611\times10^7 \ , \ 3.4202\times10^6 \ , \ 2.691\times10^5 $$

Hopefully makes it easier to compare and understand the magnitudes at a glance.

Operating on numbers in scientific notation is generally also easier, both to actually calculate and to approximate the result. Using multiplication as an example:

$$
\begin{split}
1.24611\times10^7 \ \ \times \ \ 3.4202\times10^6 & = 1.24611\times3.4202 \ \ \times \ \ 10^7\times10^6\\
& = 4.261945422 \ \times \ 10^{7+6}\\
& = 4.2619 \times 10^{13}\\
\end{split}
$$

And if the result mantissa is larger than ten; simply increase the exponent by 1 and shift the decimal place of the mantissa accordingly.

*Note that in this example I've chosen to round the result mantissa such that it's precision is commensurate with the operand that has the lowest precision* ($$3.4202\times10^6$$) *Though in practice error/uncertanty measures are a more useful metric in determining the appropriate number of decimal places to keep.*

### Binary scientific notation
Not much is different in base 2 (binary) scientific notation.

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
