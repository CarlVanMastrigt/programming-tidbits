---
katex: True
---
# Floating Point

I've seen a few people struggle to solve problems that require understanding the intricacies of floating point.
I'd like to cover what I know with some examples.
> [!NOTE]
> This discussion will constrain itself to binary floats, decimal floats do exist, but are very rare.

[The Basics](#the-basics)

[The Details](#the-details)

[Example: Order preserving unsigned integer conversion](#order-preserving-unsigned-integer-conversion)

## The Basics
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it); the fundamental structure is the same, and could be described as “scientific notation in binary”.
### Scientific notation
You may be familiar with decimal scientific notation, but if not a quick refresher:

$$\large \pm m \times 10^e$$

`m` the "mantissa" which must be greater than or equal to one and less than 10: $$\{1 \leq m < 10\}$$\
`e` the "expontent" which can be any **integer** positive or negative\
Keeping in mind that $10^0 = 1$, $10^{-1} = 0.1$ and so on.

This allows any number to be constructed, for example:\
$9876.43 = 9.87643 \times 10^3$\
$6.13 = 6.13 \times 10^0$\
$0.0405 = 4.05 \times 10^{-2}$

### Layout

## The Details
### Sub-normal numbers
### INF & NAN

## Order preserving unsigned integer conversion

