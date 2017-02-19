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

First, some mathematics. This is called a summation, more specifically arithmetic progression:

<math xmlns="http://www.w3.org/1998/Math/MathML">
    <munderover>
      <mo>&sum;</mo>
      <mn>i=1</mn>
      <mn>n</mn>
    </munderover>
    <mo>=</mo>
    <mfrac><mi>n(n+1)</mi><mn>2</mn></mfrac>
</math>

because mathematical induction. Important to remember, as this means that the complexity of an algorithm that takes `i` steps with every iteration from `i=0..n` has complexity of `Θ(n^2)` (big theta). This is the case of **selection sort**. And the generified rule is:

<math xmlns="http://www.w3.org/1998/Math/MathML">
    <mi>S(n, p)</mi>
    <mo>=</mo>
    <munderover>
      <mo>&sum;</mo>
      <mi>i</mi>
      <mi>n</mi>
    </munderover>
    <msup>
      <mi>i</mi>
      <mi>p</mi>
    </msup>
    <mo>=</mo>
    <mi>Θ(</mi> 
    <msup>
         <mi>n</mi>
         <mi>p+1</mi>
    </msup>
    <mi>)</mi>
</math>

for `p>=1`.

Geometric series have the index of the loop involved in the exponent:

<math xmlns="http://www.w3.org/1998/Math/MathML">
    <mi>G(n, a)</mi>
    <mo>=</mo>
    <munderover>
      <mo>&sum;</mo>
      <mi>i=0</mi>
      <mi>n</mi>
    </munderover>
    <msup>
      <mi>a</mi>
      <mi>i</mi>
    </msup>
    <mo>=</mo>
    <mfrac>
        <mrow>
            <msup>
              <mi>a</mi>
              <mi>n+1</mi>
            </msup>
            <mo>-</mo>
            <mn>1</mn>
        </mrow>
        <mrow>
            <mi>a</mi>
            <mo>-</mo>
            <mn>1</mn>
        </mrow>
    </mfrac>
    <mo>=</mo>
    <mi>Θ(</mi> 
    <msup>
         <mi>a</mi>
         <mi>n+1</mi>
    </msup>
    <mi>)</mi>
</math>

for for `a>=1`.

What may be also relevant:

<math xmlns="http://www.w3.org/1998/Math/MathML">
    <munderover>
      <mo>&sum;</mo>
      <mi>i=1</mi>
      <mi>n</mi>
    </munderover>
    <mi>i</mi>
    <mo>×</mo>
    <mi>i!</mi>
    <mo>=</mo>
    <mi>(n+1)!-1</mi>
</math>
