# How to calculate gaps between floating-point numbers

Say we take every possible value representable in a given floating-point implementation and name them from smallest to largest with $ v_0, v_1, v_2, ...$ .

The gap between $ v_i $ and $ v_{i+1} $, for any $ 2^n \le v_i < 2^{n+1} $, is going to be $ 2^{-s+n} $, where $ s = $ amount of bits in the significand.

## Example

IEEE 754 double-precision floating-point numbers are 64 bits long, of which 52 are for the significand/mantissa.

To know the gap between 1024 and 1025, we first find in which power-of-2 range 1024 is. 1024 is exactly $ 2^{10} $, so $ n = 10 $.

Plugging those values in the equation:

$$
2^{-s+n} =
2^{-52+10} =
2^{-42} = 2.2737368e-13
$$

## Going the other way

To find what is the smallest number for which the gap between consecutive values grows larger than some precision $ p $, we just run the numbers in reverse:

$ 2^{-s+n} = p $

$ -s+n = \log_2{p} $

$ n = \log_2{p} + s $

This $ n $ would give us our number, if the gaps increased linearly throughout the range of values. That's not the case, though. Because of the way the floating-point arithmetic work, the width of gaps between values only changes at powers of 2, so we need to search for the first power of two at which $ p $ grows larger than the specified precision.

Therefore, our number is going to be:

$ v = 2^{\lceil n \rceil } $
