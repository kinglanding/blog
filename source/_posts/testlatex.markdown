---
title: Latex on hexo
date: 2016-04-27 14:09:21
tags: 
- hexo 
- math example
---

\begin{aligned}
   \phi(x,y) = \phi \left(\sum_{i=1}^n x\_ie\_i, \sum\_{j=1}^n y\_je\_j \right)
  = \sum\_{i=1}^n \sum\_{j=1}^n x\_i y\_j \phi(e\_i, e\_j)
\end{aligned}

$$\frac{\partial u}{\partial t}
= h^2 \left( \frac{\partial^2 u}{\partial x^2} +
\frac{\partial^2 u}{\partial y^2} +
\frac{\partial^2 u}{\partial z^2}\right)$$

{% math %}
\begin{aligned}
\dot{x} & = \sigma(y-x) \\
\dot{y} & = \rho x - y - xz \\
\dot{z} & = -\beta z + xy
\end{aligned}
{% endmath %}

\begin{aligned}
\dot{x} & = \sigma(y-x) \\
\dot{y} & = \rho x - y - xz \\
\dot{z} & = -\beta z + xy
\end{aligned}

Simple inline $a = b + c$.

{% math %}
f(x1, x2) = \frac{ 1 }{2\pi \sigma1\sigma2\sqrt{1-r^2}}e^{-\frac{1}{2(1-r^2)} 
[\frac{(x1-\mu\_{x1})^2}{\sigma\_{x1}^2} - \frac{2r(x1-\mu\_{x1})(x2-\mu\_{x2})}{\sigma\_{x1} \sigma\_{x2}} + \frac{(x2-\mu\_{x2})^2}{\sigma\_{x2}^2}]}
{% endmath %}

{% math %}
\begin{aligned}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{aligned}
{% endmath %}

{% math %}  
\min { \quad \frac { 1 }{ 2 } \left\| w \right\| ^{ 2 }+c\sum _{ i=1 }^{ l }{ { \xi }_{ i } } } \\ st.\quad y_{ i }(\omega ^{ T }\phi (x_i)-b)\ge 1-\xi _{ i },\left( \xi _{ i }>0 \right)  
{% endmath %}  