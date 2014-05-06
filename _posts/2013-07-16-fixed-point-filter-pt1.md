---
comments: true
date: 2013-07-16 15:00:00
layout: post
slug: fixed-point-single-pole-filter

title: Fixed Point Filtering 
summary: "Fixed Point Single Pole Digital Filter"
image: 'fixedpt-digital-filter/dfilt.png'
tags:
- dsp
- fixed-point
---

Recently, I had a project that needed to be implemented on a microcontroller
with a little bit of noise rejection.  The signal to be conditioned was
extrmemly low frequency in nature, so even an averaging filter would likely
work, but I could never lower myself to such standards; and niether should
you!  :-)

As a quick aside, averaging filters may have a linear phase, but they pass
a multitude of frequencies.  If system constraints are too great, such as
computing power or available memory, suitable attenuation can not be attained.
Not to mention the difficulty of determining the optimal averaging window.
The corner frequency will be relatively low, attenuating exciting signals which
may be unintentionally attenuated.  I'll illustrate by a simple example of
a discrete filter with a sampling frequency of 1 kHz.  We'll just compare a
single pole filter with a cutoff of 10 Hz and a averaging filter with a window
length of 10.

![Averaging Filter and Single Pole Frequency Response](/img/filtcmp.png)

Clearly, not a pretty picture, not the kind of spectral annihilation we'd like to see.

We can use [this here link](http://lorien.ncl.ac.uk/ming/filter/fillpass.htm) as a
starting point.  This is a **classic** single pole lowpass filter.  I would hazard
a guess that this is the most popular linear filter found in control applications.
Sure, there are all other kinds of filters, each with its pros and cons, but this
filter is very intuitive.  First order is often how we want and expect systems to
respond.

First, let's look at the starting point for our filter.
We'll use {% raw %}<script type="math/tex">y</script>{% endraw %} as the output
of our filter and {% raw %}<script type="math/tex">u</script>{% endraw %} as the
input signal.  Let's look at the Laplace Transform of a first order analog filter.

{% raw %}
<script type="math/tex; mode=display">
\frac{Y(s)}{U(s)} = \frac{1}{1 + \tau_{c}s}
</script>
{% endraw %}

Here the {% raw %}<script type="math/tex">\tau_{c}</script>{% endraw %} is the
reciprocal of the angular frequency of the cutoff for the first order filter.

{% raw %}
<script type="math/tex; mode=display">
\omega_{c} = 2 \pi f_{c} = \frac{1}{\tau_{c}}
</script>
{% endraw %}

Now, we'll do some algebra and put it in first order linear configuration.

{% raw %}
<script type="math/tex; mode=display">
Y(s) + \tau_{c}s Y(s) = U(s)
</script>
{% endraw %}

Now, let's perform a Laplace Transform and get the time domain representation
of the equation.

{% raw %}
<script type="math/tex; mode=display">
y(t) + \tau_{c} \frac{dy}{dt} = u(t)
</script>
{% endraw %}

Next, we'll transform the equation from the continuous time domain to the
discrete time domain using a backward Euler approximation.

{% raw %}
<script type="math/tex; mode=display">
\frac{dy}{dt} = \frac{y_{k}-y_{k-1}}{T_{s}} 
</script>
{% endraw %}

Where {% raw %}<script type="math/tex">T_{s}</script>{% endraw %} is the
period at which the signal is sampled.  Let's work through this, just to
be exhaustive.

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
y_{k} + \tau_{c} \frac{y_{k}-y_{k-1}}{T_{s}} & = u_{k} \\
y_{k}(1 + \frac{\tau_{c}}{T_{s}}) - y_{k-1} \frac{\tau_{c}}{T_{s}} & = u_{k} \\
y_{k} & = y_{k-1} \frac{\frac{\tau_{c}}{T_{s}}}{(1 + \frac{\tau_{c}}{T_{s}})} + \frac{u_{k}}{(1 + \frac{\tau_{c}}{T_{s}})} \\
y_{k} & = \frac{\tau_{c}}{T_{s} + \tau_{c}} y_{k-1} + \frac{T_{s}}{T_{s} + \tau_{c}} u_{k} \\
y_{k} & = \frac{\tau_{c} + T_{s} - T_{s} }{T_{s} + \tau_{c}} y_{k-1} + \frac{T_{s}}{T_{s} + \tau_{c}} u_{k} \\
y_{k} & = ( \frac{T_{s} + \tau_{c}}{T_{s} + \tau_{c}} - \frac{T_{s}}{T_{s} + \tau_{c}} ) y_{k-1} + \frac{T_{s}}{T_{s} + \tau_{c}} u_{k} \\
y_{k} & = ( 1 - \frac{T_{s}}{T_{s} + \tau_{c}} ) y_{k-1} + \frac{T_{s}}{T_{s} + \tau_{c}} u_{k}
\end{aligned}
</script>
{% endraw %}

Then we let,

{% raw %}
<script type="math/tex; mode=display">
\alpha = \frac{T_{s}}{T_{s} + \tau_{c}}
</script>
{% endraw %}

and we get

{% raw %}
<script type="math/tex; mode=display">
y_{k} = ( 1 - \alpha ) y_{k-1} + \alpha u_{k}
</script>
{% endraw %}

Now we have a formula in the time domain that is a function of our

* Sample time {% raw %}<script type="math/tex">T_{s}</script>{% endraw %} 
* Cutoff Frequency {% raw %}<script type="math/tex">\tau = \frac{1}{2 \pi f}</script>{% endraw %}
* Previous Output {% raw %}<script type="math/tex">y_{k-1}</script>{% endraw %}
* Current Input {% raw %}<script type="math/tex">u_{k}</script>{% endraw %} 

This form can now be used to write a short routine to run on a microcontroller.
However, we may also need to write the routine using only fixed binary point
operations, no floating point allowed!  We'll look at that some time in the
near future!

