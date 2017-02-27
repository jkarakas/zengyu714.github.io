---
author: kimmy
layout: post
title:  "Backpropagation Equations"
date:   2017-02-27 16:14:58 +08:00
categories: Deep Learning
tags:
- Deep Learning
- Notes
- Backpropagation

---


* content
{:toc}



###  $$\delta_j^l$$ of neuron $$j$$ in layer $$l \\ $$

$$
\delta_j^l\equiv \dfrac{\partial C}{\partial z^l_j}
$$

###  $$\delta_j^L$$ of neuron $$j$$ in the output layer $$L \\ $$
$$
\delta_j^l = \dfrac{\partial C}{\partial z^L_j} \ \sigma'(z^L_j)\tag{BP1}
$$

###  $$\delta^l$$ in terms of the error in the next layer,  $$\ \delta^{l+1}\\ $$
$$
\delta^l = (w^{l+1})^T\delta^{l+1})\ \odot\ \sigma'(z^l)\tag{BP2}
$$

###  $$\partial C$$ with respect to any $$b$$ <br>
$$
\dfrac{\partial C}{\partial b^l_j}= \delta_j^l \tag{BP3}
$$

###  $$\partial C$$ with respect to any $$w$$ <br>
$$
\dfrac{\partial C}{w_{jk}^l}= a^{l-1}_k \delta^l_j \tag{BP4}
$$

<br><br>
> See full article, [How the backpropagation algorithm works](http://neuralnetworksanddeeplearning.com/chap2.html)
