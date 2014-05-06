---
comments: true
date: 2013-08-05 15:00:00
layout: post
slug: fft-analog-to-digital
title: FFT Analog Representation
summary: "FFT to Analog Frequency Response"
image: 'fft-analog-digital/test-signal.png'
tags:
- fft
- dsp
---

Often, we want to find the spectral content of a sampled signal.
First, we think that we can quickly solve this problem by computing
the Fast Fourier Transform of the signal using a software package like
[Octave](http://www.octave.org), [Matlab](http://www.mathworks.com),
[Julia](http://www.julia.com), etc.  Let's take an example from the
Mathworks.

{% highlight matlab %}
Fs = 1000;                    % Sampling frequency
T = 1/Fs;                     % Sample time
L = 1000;                     % Length of signal
t = (0:L-1)*T;                % Time vector
% Sum of aHz sinusoid and a 120 Hz sinusoid
x = 0.7*sin(2*pi*50*t) + sin(2*pi*120*t); 
{% endhighlight %}

![Filter Frequency Response](/img/posts/fft-analog-digital/test-signal.png)

Now, let's take the aforementioned `fft` and see what we get.

![Filter Frequency Response](/img/posts/fft-analog-digital/test-fft.png)

Ok...So what is this?

What we often would rather have is an approximation of the analog frequency
response.  Instead, what we get is a Discrete Fourier Transform.

{% raw %}
<script type="math/tex; mode=display">
X_{k} = \sum_{n=0}^{N-1} x_{n}e^{-j\frac{2 \pi}{N}kn}
</script>
{% endraw %}

What we need is a relationship between the FFT and the Fourier Transform.
This will link the discrete frequency response to an approximation of the
continuous frequency response.  To transform the DFT, or FFT, to an analog
Fourier Transform, is not a trivial matter.  Thanks to 
[James McNames](http://bsp.pdx.edu/Reports/BSP-TR0201.pdf), we can synthesize
a simple transform to convert the DFT to a Fourier Transform equivalent.

First, let's look at the theory of the relationship between the Discrete
Fourier Transform and the Fourier Transform. The Fourier Transform is defined
as:

{% raw %}
<script type="math/tex; mode=display">
\mathcal{F}\{x(t)\} = \int_{-\infty}^{\infty} e^{-j \Omega t} dt
</script>
{% endraw %}

We'll use the uppercase {% raw %}<script type="math/tex">\Omega</script>{% endraw %}
to represent frequencies in the continuous time domain and the lowercase {% raw %}<script type="math/tex">\omega</script>{% endraw %} to
represent frequencies in the discrete frequency domain to prevent confusion.

Now, we must consider the sampling of the signal.  To represent a sampled signal
mathematically, it is common to use an infinite series of shifted Dirac Delta
functions, {% raw %}<script type="math/tex">\delta (t)</script>{% endraw %}.

{% raw %}
<script type="math/tex; mode=display">
p(t) = T \sum_{n=-\infty}^{\infty} \delta(t -nT)
</script>
{% endraw %}

This function will be our sampling function that must be mixed with our
continuous time signal in order to create a discrete sequence.  A discrete
sampled time sequence is defined as

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
x[n] & = x(t) |_{t=nT} \\
     & = \lim_{\epsilon \to 0} \int_{nT - \epsilon}^{nT + \epsilon} x(t) \cdot p(t) dt
\end{aligned}
</script>
{% endraw %}

We can represent our sampled time signal as the continuous time signal mixed with
the sampling function {% raw %}<script type="math/tex">x_{s}(t) = x(t)p(t)</script>{% endraw %}.  This will make up the foundation of our conversion from the discrete to
continuous domain.  We must take our time domain representation and find the frequency
domain representation.

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
X_{s}(\Omega) & = \int_{-\infty}^{\infty} x_{s}(t) e^{-j \Omega t} dt \\
& = \int_{-\infty}^{\infty} ( x(t) T \sum_{n=-\infty}^{\infty} \delta(t -nT) )  e^{-j \Omega t} dt \\ 
& = T \sum_{n=-\infty}^{\infty} x[n] e^{-j \Omega nT}
\end{aligned}
</script>
{% endraw %}

We see that we now have the continous transform {% raw %}<script type="math/tex">Xs</script>{% endraw %} in
terms of the discrete time signal {% raw %}<script type="math/tex">x[n]</script>{% endraw %}.  Yet, we must consider another feature of our sampled signal.  Our signal will
be finite, or windowed.  This means that we may truncate the infinite sum.  For
convience, we will replace the upper limit with `N-1` and begin our summation with `0`
giving us a sequence of `N` terms.  Now, we can compare the equation for {% raw %}<script type="math/tex">Xs(\Omega)</script>{% endraw %} with the equation for the FFT.

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
X[k] & = \sum_{n=0}^{N-1} x[n]e^{-j\frac{2 \pi}{N}kn} \\
X_{s}(\Omega) & = T \sum_{n=0}^{N-1} x[n] e^{-j \Omega nT}
\end{aligned}
</script>
{% endraw %}

As James McNames writes, all we must do now is implement a simple change
of variables, and we're done!

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
X[k] & = \sum_{n=0}^{N-1} x[n]e^{-j\frac{2 \pi}{N}kn} \\
     & = \frac{1}{T} X_{s}(\Omega) |_{\Omega=\frac{k}{N} \frac{2 \pi}{T} } \\
		 & \text{yields} \\
X_{s}(\Omega) & = T X[k] \text{ where } \Omega \in [0,\frac{1}{N}\frac{2 \pi}{T},...,\frac{N-1}{N}\frac{2 \pi}{T}]
\end{aligned}
</script>
{% endraw %}

So, let's return to our original example and use our new found conversion method.


{% highlight matlab %}
Fs = 1000;                    % Sampling frequency
T = 1/Fs;                     % Sample time
L = 1000;                     % Length of signal
t = (0:L-1)*T;                % Time vector
% Sum of aHz sinusoid and a 120 Hz sinusoid
x = 0.7*sin(2*pi*50*t) + sin(2*pi*120*t); 
Xk = fft(x);
f = [ -L/2 : (L/2 - 1) ];
A0 = 0.5*[1 0.7 0.7 1];
f0 = [-120 -50 50 120];
plot( f, fftshift(abs(X)),f0,A0,'*' )
title('FFT Fourier Transform Approximation')
xlabel('Frequency(Hz)')
ylabel('Amplitude')
legend('FFT','Theory')
{% endhighlight %}

Which yeilds the following plot:


![Filter Frequency Response](/img/posts/fft-analog-digital/test-convert.png)

