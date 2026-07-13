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
For binary floating point types; single precision `float`, double precision `double` and half precision `half` (when you have it). The fundamental structure is the same, and it could be described as “scientific notation in binary”.
### Scientific noatation
You may be familiar with decimal scientific notation:
$$`\pm m \times 10^e`$$ 
`m` the mantissa $`1 \geq m < 10`$ 
`e` the expontent; any number positive or negative
test `$1 \geq m < 10$` test
### Layout

## The Details
### Sub-normal numbers
### INF & NAN

## Order preserving unsigned integer conversion

