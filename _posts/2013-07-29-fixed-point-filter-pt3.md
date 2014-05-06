---
comments: true
date: 2013-07-29 15:00:00
layout: post
slug: fixed-point-single-pole-filter-pt3

title: Fixed Point Filter Math in Action
summary: "Fixed Point Math In Action"
image: 'fixedpt-digital-filter/dfilt.png'
tags:
- dsp
- fixed-point
---

Let's return to our fix point filter example code.

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

Where we have `alpha` and `beta` scaled by a radix of `15`, or left shifted by
`15`.  Ultimately, the equation we are looking to implement is:

{% raw %}
<script type="math/tex; mode=display">
y_{k} = ( 1 - \alpha ) y_{k-1} + \alpha u_{k}
</script>
{% endraw %}

So we must multiply `beta` and the previous output, `alpha` and the current
input to the filter and then sum the result.  The input and output of the filter
is defined to be an unsigned two byte number.  If we consider the maximum value for
the product, we can assume the maximum fixed point value for each input to be
`65535`.  Therefore, the maximum value of the product is:

{% raw %}
<script type="math/tex; mode=display">
65535 \times 65535 < 2^{32} - 1
</script>
{% endraw %}

This product snugly fits into an unsigned 4 byte value.  So, to prevent overflow from a
two byte value we can use a four byte variable to store the intermediate
calculation.  We can also, just cast the input values to four byte numbers
and then remove the radix point scaling to allow the value to fit into two
bytes.

{% highlight cpp %}
  uint16_t y1 = (beta*(uint32_t)prev_output) >> 15;
{% endhighlight %}

The above will calculate the intermediate value of the output term.  Now to sum the
two values.  Since we have removed the `15` bit radix scaling, the remaining scaling
is the scaling of the input variable.  No more scaling need be performed.  To calculate
the output, all that must be done is to sum the intermediate input and output terms.

{% highlight cpp %}
	/* calculate current filter output */
  output = y1 + u1;
{% endhighlight %}

For my application, the signal to filter was very slow and sampled at a period of 0.5 seconds.
I chose the cutoff frequency to be 0.1 Hz.  At this point, the filter should attenuate
the signal by about 3 dB.  It is very common to chose the -3 dB point as the cutoff frequency.
This makes the -3 dB point a key performance parameter.  Using the equation for the cutoff
parameter, we can calculate `alpha`.

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
\alpha & = \frac{T_{s}}{T_{s}+\tau_{c}} \text{with} \\
\tau_{c} & = \frac{1}{2 \pi f_{c}} \text{gives us} \\
\alpha & = \frac{T_{s}}{T_{s}+ \frac{1}{2 \pi f_{c}}} \\
\alpha & = \frac{0.5}{0.5+ \frac{1}{2 \pi \times 0.1}} \\
\alpha & = 0.23906 
\end{aligned}
</script>
{% endraw %}

### -3 dB Point
Here is a close up of the frequency response at the -3 dB point.
![-3 dB Point](/img/posts/fixedpt-digital-filter/dfilt3dB.png)
We can clearly see that the -3 dB point is almost dead on our design
goal of 0.1 Hz.

With the `alpha` parameter, we're ready to finish the implementation.  I wrote a simple
driver program for the host machine.  This program uses a step input corrupted with
Gaussian noise to test the filter for perfomance.  Coupled with some simple Octave
code, we can examine the input and output of the filter to ensure that it is performing
acceptably, with no overflows or round off errors.  Here is the noise corrupted input
step from 0 to 500 bits.
### Single Pole Filter Input
![Single Pole Filter Input](/img/posts/fixedpt-digital-filter/input.png)

### Single Pole Filter Output
Here is the output of the filter, showing that the noise has been significantly
reduced.
![Single Pole Filter Output](/img/posts/fixedpt-digital-filter/output.png)

### Single Pole Filter Input/Output
Here is a side by side comparison of input and output signals of the filter.  Even
though it does leave some to be desired, it is quite adequate for my needs.
![Single Pole Filter Input/Output](/img/posts/fixedpt-digital-filter/io.png)

