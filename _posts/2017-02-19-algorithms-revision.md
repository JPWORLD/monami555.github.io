---
layout: post
title: Algorithms - revision
date: '2017-02-19'
author: Monik
tags:
- Algorithms
commentIssueId: 34
---
<div class="bg-info panel-body" markdown="1">
[under construction]<br/>
Revising the most important things about data structures and algorithms.
</div>

# Table of Contents
  * [Complexity analysis](#complexity)

## Complexity analysis <a id="complexity"></a>
<!-- http://docs.mathjax.org/en/latest/start.html -->

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>
<script type="text/javascript" async src="https://github.com/mathjax/MathJax/blob/master/MathJax.js?config=TeX-AMS_CHTML"></script>

First some mathematics:

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mi>a</mi><mo>&#x2260;</mo><mn>0</mn>
</math>,


$$
\begin{align*}
  \sum_{i=1}^n = \frac{n(n+1)}{2}
\end{align*}
$$


<p>When `a != 0`, there are two solutions to `ax^2 + bx + c = 0` and
they are</p>
<p style="text-align:center">
  `x = (-b +- sqrt(b^2-4ac))/(2a) .`
</p>


because mathematical induction. Important to remember, as this means that the complexity of something that takes i steps with every iteration from i=0..n has complexity of (bit theta)(n^2). And the generified rule is:

$$
\begin{align*}
  S(n,p) = \sum_{i}^n i^p = \theta n^(p+1)
\end{align*}
$$

sdfaklsdfjkasldjfkjdsahlfkhsdakjfhlasdkjfhksd
