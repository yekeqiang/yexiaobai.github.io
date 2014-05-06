---
comments: true
date: 2013-07-27 15:00:00
layout: post
slug: fixed-point-single-pole-filter-pt2

title: Fixed Point Filter Implementation
summary: "Fixed Point Math for Digital Filtering"
image: 'fixedpt-digital-filter/dfilt.png'
tags:
- dsp
- fixed-point
---

Many higher end microprocessors do not require fixed point math as they have
in hardware a floating point unit which is quite fast.  However, there are still
many 8 bit, 16 bit, and even 32 bit systems which do not have this luxury and
must take a sizable performance hit to emulate floating point operations in
software.  Rather than make the micro suffer each time the `RESET` line goes
low, we can implement fixed point software that will save untold CPU cycles.

Fixed pointing our math can be tricky business.  Overflows, round off errors,
etc. can cause severe problems.  Simulation is one of the best tools to combat
these problems and prevent the costly cycle of cross compile, on target test,
and on target debug.  If one has [Matlab/Simulink](http://mathworks.com),
simulation comes for free.  If ones pockets are not so deep, like ~$30k for
a basic embedded code generation package *eek!*, we can still compile our
routines for the host machine, in my case Ubuntu Linux.  We can write our
fixed point filter as a function and design it to be built for either an
embedded target or our host machine.

Without any further ado, here's the routine:
{% highlight cpp %}
/* dfilt_single_pole
 *  Parameters:
 *    input - current input to the filter
 *    output - previous time step output of the filter
 *    alpha - cutoff frequency parameter, scaled by left shifted 15
 *  returns:
 *    current time step output of the filter
 *  Calculates the current output of the digital filter for a two byte
 *  unsigned signal.
 */
uint16_t
dfilt_single_pole_u16(uint16_t input, uint16_t prev_output, uint16_t alpha)
{
  uint16_t output;                  /* current time step output */
  uint32_t beta = ((1<<15)-alpha);  /* coefficient of previous output */
  /* calculate the previous output term and scale to output scaling */
  uint16_t y1 = (beta*(uint32_t)prev_output) >> 15;
  /* calculate the input term and scale to output scaling */
  uint16_t u1 = ((uint32_t)alpha*(uint32_t)input) >> 15;
	/* calculate current filter output */
  output = y1 + u1;
	return output;
}
{% endhighlight %}

First, let's look at the `beta` variable which is chosen to be `1-alpha`, where
`alpha` is our only filter calibration constant.  Since the value of alpha is

{% raw %}
<script type="math/tex; mode=display">
\alpha = \frac{T_{s}}{T_{s} + \tau_{c}}
</script>
{% endraw %}

with {% raw %}<script type="math/tex">T_{s} > 0</script>{% endraw %}
and {% raw %}<script type="math/tex">\tau > 0</script>{% endraw %}
we see that `alpha` must always
be less than one.  Equal to one would indicate that the inverse of the
angular cutoff frequency is zero, meaning that the cutoff frequency is
approaching infinity.  If this were so, there would be no need for a
filter.  :-)

So we know the range of the `alpha` parameter is `[0,1)`, but we have
not specified the resolution.  If we choose to use two bytes as our standard
data type, we will be able to compute four byte products.  When working with
a 32 bit processor, this is quite convenient.  Even when working with
8 or 16 bit processors, two bytes quickly becomes a requirement since
1 byte will only have 255 discrete levels, often not granular enough to
yield an accurate fixed point representation.

If we use a two byte unsigned variable, since `alpha` must be strictly
positive, we can map the range of `[0,65536)` to `[0,1)`.  We can see
this is a simple linear transformation that has zero offset and a
positive slope.

We would like to do all of our scaling in powers of two since this can
be realized as bit shifting on the processor.  To give units to these
data we'll use `bits` for the fixed point number and `fraction` for the
engineering unit of `[0,1)`.  We can then to the following
to calculate the scaling:

{% raw %}
<script type="math/tex; mode=display">
\frac{65536\text{ bits}}{1\text{ fraction}} = 65536\text{ bits/fraction}
</script>
{% endraw %}

So our scaling is 65536 bits/fraction, if expressed as a power of two
we can use the formula:

{% raw %}
<script type="math/tex; mode=display">
R = \lfloor log_{2}(S) \rfloor
</script>
{% endraw %}

Where `S` is the desired scaling for the fixed point number and `R` is the
binary shift required for the closest representation which is less than or
equal to the desired scaling.  Using this equation we can see that the
required shifting for maximum resolution is `16`.  This yields a resolution
of `1.5259e-05`, which is plenty, we can back down the resolution to help
in implementation, by being less likely to overflow.  If we use `15` we still
have a resolution of `3.0518e-05`, which will be quite enough for our filter
and allow us to have a range of `[0,2)` which will simplify our computation
significantly by allowing us to represent unity.

So, if we want to represent any number with our `alpha` scaling it must be
scaled up, left shifted by 15 places.  We can assume in our function that
`alpha` is already scaled.  However, when we calculate `beta` we must compute
`1-alpha`.  Since `alpha` is scaled, we must scale the `1` as well.

{% highlight cpp %}
beta = (1<<15) - alpha /* now scaled by l15 */
{% endhighlight %}

Commonly used notation is the `lX` where `l` is the left shift operator
and `X` is the number of shifts.  Now we have our two coefficients.  Whew!
I hope this is worth it!

Next time we'll look at putting these values to work!

