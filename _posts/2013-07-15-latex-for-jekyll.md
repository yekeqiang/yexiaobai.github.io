---
comments: true
date: 2013-07-15 15:00:00
layout: post
slug: latex-and-jekyll

title: Render LaTeX in Jekyll with RDiscount
summary: "Time to enhance our blog with LaTeX Rendering!"
image: 'latex-jekyll-render/latex.png'
tags:
- latex
- jekyll
---

# LaTex and Jekyll

As I've written in a [previous post](http://www.liquidinertia.com/2013/07/05/jekyll-setup-basics-pt2/), I would like to be able
to render LaTeX within my blog posts.  Every once and a while it
seems that you want to describe something with a formula, and really,
LaTeX is the way to go.

RDiscount doesn't support LaTeX, but does support a lot of other really
nice features.  So, I looked for an extension or something that would 
allow LaTeX formatting.  I found this [blog post](http://cwoebker.com/posts/latex-math-magic) by a High School student in Boston and was properly impressed!  Check this guy out, unbelievable!

He found a method to use MathJax JavaScript and CSS.  This is a pretty
sweet solution.  I've added it to my blog, feel free to check out
the repo and peruse the delta between this post and the previous.  The
configuration is pretty minor, just cut'n'paste the Javascript and we're
good to go!  Now, let's test it out!

Examples taken from the MathJax [Demo Page](http://mathjax.org/demos/tex-samples/)

### The Lorentz Equations

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
\dot{x} & = \sigma(y-x) \\
\dot{y} & = \rho x - y - xz \\
\dot{z} & = -\beta z + xy
\end{aligned}
</script>
{% endraw %}

#### The Cauchy-Schwarz Inequality

{% raw %}
<script type="math/tex; mode=display">
\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)
</script>
{% endraw %}

#### A Cross Product Formula

{% raw %}
<script type="math/tex; mode=display">
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0
\end{vmatrix}
</script>
{% endraw %}

#### The probability of getting k heads when flipping n coins is

{% raw %}
<script type="math/tex; mode=display">
P(E)   = {n \choose k} p^k (1-p)^{ n-k}
</script>
{% endraw %}

#### An Identity of Ramanujan

{% raw %}
<script type="math/tex; mode=display">
\frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigr) e^{\frac25 \pi}} =
1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\frac{e^{-6\pi}}
{1+\frac{e^{-8\pi}} {1+\ldots} } } }
</script>
{% endraw %}

#### A Rogers-Ramanujan Identity

{% raw %}
<script type="math/tex; mode=display">
1 +  \frac{q^2}{(1-q)}+\frac{q^6}{(1-q)(1-q^2)}+\cdots =
\prod_{j=0}^{\infty}\frac{1}{(1-q^{5j+2})(1-q^{5j+3})},
\quad\quad \text{for $|q|<1$}.
</script>
{% endraw %}

#### Maxwell's Equations

{% raw %}
<script type="math/tex; mode=display">
\begin{aligned}
\nabla \times \vec{\mathbf{B}} -\, \frac1c\, \frac{\partial\vec{\mathbf{E}}}{\partial t} & = \frac{4\pi}{c}\vec{\mathbf{j}} \\   \nabla \cdot \vec{\mathbf{E}} & = 4 \pi \rho \\
\nabla \times \vec{\mathbf{E}}\, +\, \frac1c\, \frac{\partial\vec{\mathbf{B}}}{\partial t} & = \vec{\mathbf{0}} \\
\nabla \cdot \vec{\mathbf{B}} & = 0 \end{aligned}
</script>
{% endraw %}

