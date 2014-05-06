---
comments: true
date: 2013-08-01 15:00:00
layout: post
slug: nyquist-digital-filter-sampling-specification 
title: Nyquist, and Digital Filter Sampling Frequency 
summary: "Digital Filter Sampling"
image: 'digital-sampling/sinc.png'
tags:
- dsp
- nyquist
---

## Digital Filter Specification
When we develop digital filters, we need to specify the filter
cutoff as a fraction of the bandlimit.  Here is the `help` from
the `octave` Butterworth filter specification function `butter`:

{% highlight console %}
 Generate a butterworth filter.
 Default is a discrete space (Z) filter.
 
 [b,a] = butter(n, Wc)
    low pass filter with cutoff pi{% raw %}*{% endraw %}Wc radians
{% endhighlight %}

Although it's not that confusing, it's still not that clear what exactly
what the cutoff should specify.  First off, we need to know the sample
period of the digital filter we are going to design.  Let's call this
value {% raw %}<script type="math/tex">T_{s}</script>{% endraw %}.  For
a digital filter to be effective, the cutoff must be significantly below
the sampling frequency.  The sampling frequency must be significantly above
the frequency content of the signal.

## Sampling Frequency
If we have a signal, corrupted with noise or otherwise, that can be
approximately considered bandlimited to a certain frequency.  The sampling
frequency should be about ten times greater than that frequency.  This
may sound so rudimentary that it is not even worth mentioning, but I have
been quite surprised by engineers that don't seem to understand this fact.

### Nyquist Frequency
This mantra of digital signal processing is quite well know, but not as
well understood.

{% raw %}
<script type="math/tex; mode=display">
f_{B} < 2 \times f_{s} \text{ or }
\frac{1}{2} \times f_{B} < f_{s}
</script>
{% endraw %}

The bandlimit of the signal to be sampled must be strictly less than
twice the sampling frequency.  I have actually been told by a senior
engineer before that he had done signal analysis and was going to sample
the signal "at Nyquist."  This is fingernails on a chalkboard.  When I
hear that term, I know they are going to sample at twice the highest
frequency of the signal of interest.  Let's look at a classic example of
what can happen when we think of Nyquist's Theorem this way.

## Sampling Example

Here we show a plot of a 100 Hz sine wave.

![Original Sine Wave](/img/posts/digital-sampling/origsine.png)

Next we consider sampling of the sine wave at the so called Nyquist
frequency, and four times such a frequency.

![Sine Wave with Sampling](/img/posts/digital-sampling/samplingsine.png)

Finally, consider the way one might interpolate the signals which would
be constructed from the different sampling.

![Signal Sampled at Nyquist](/img/posts/digital-sampling/nyquistsine.png)

![Signal Sampled at 4 Times Nyquist](/img/posts/digital-sampling/nyquist4sine.png)

## Why doesn't this work?

The Nyquist Frequency is an absolute minimum.  Some people almost treat
it as a maximum.  I realize this may be hard to believe, but I have seen
it.  The sampling frequency must be greater than twice the fastest
frequency.  However, the original signal can not be reconstructed using a
simple zero order hold.  Instead, it requires a pulse train, the samples,
each mixed with a `sinc` function, which has the familiar form of:

{% raw %}
<script type="math/tex; mode=display">
sinc(x) = \frac{sin(x)}{x}
</script>
{% endraw %}

![Sinc Function](/img/posts/digital-sampling/sinc.png)

It can be shown that a signal sampled according to the Nyquist Frequency
can be reconstructed *completely*, meaning that any point between the
samples can be interpolated using the following equation.

{% raw %}
<script type="math/tex; mode=display">
f(t) = sinc(\frac{\pi}{T} t) * \sum_{n=-\infty}^{\infty} f(nT) \delta(t-nT)
</script>
{% endraw %}

Since the `sinc` function continues toward positive and negative infinity,
mixing with any causal signal is not practical.  Even a shifted truncated
series is still computationally costly.  If at all possible, it is far
easier and less error prone to "over" sample by a factor of 10.  We'll
continue our discussion of filter specification next time.

