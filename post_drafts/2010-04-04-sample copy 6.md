---
layout: post
title: "Maximum Likelihood estimation for time series models"
author: "You!"
categories: journal
tags: [documentation,sample,blog]
image: mountains_trim.jpg
date: 2021-03-13 15:34:09 +0100
---

In this post ... $a=5$ asdf


$$ L(\boldsymbol{\theta})=f_{Y_{T},Y_{T-1},...,Y_{1}}(y_{T},y_{T-1},...,y_{1};\boldsymbol{\theta}) $$

This approach requires specifying a partiular distribution for the noise proccess. For example the Gaussian white noise:

$$\epsilon_{t}\sim i.i.d.\,N(0,\sigma^{2})$$

...

This has to be maximised with respect to $\boldsymbol{\theta}.$
It is often conditioned on the first $p$ observation (Conditional
Maximum Likelihood Estimation), therefore $y_{p},...,y_{1}$ are often
substituted with their observed or expected values. Thus the loglikelihood
can be simplified to