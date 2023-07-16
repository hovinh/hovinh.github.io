---
#layout: post
title: Fourier Transform
description: >
  Fourier transform one liner. This post to ...
author: author1
comments: true
---

**Code**: <a href="https://github.com/hovinh/fourier-transform">Github</a>

I'm fascinated by the ingenious idea of Fourier Transform - a signal can be expressed as a sum of a set of basis signals. Why is this exciting? By breaking down to simpler and more interpretable components, we can study the original signal's nature in a constructive lens, manipulate them as will to see their influence on the original form, and ultimately, transform the original signal itself under a control environment. An ML practitioner definitely sees the great potential of such method in the data analysis. 

Better yet, the aforementioned **signal** could be broadly applicable to almost any natural phenomena, be it a time series, a 2D image, an audio recording, or higher dimensional functions. 

In this post, I will explain the Fourier Transform algorithm in an intuitive manner with adequate mathematical reasoning to back up. The scope is limit to univariate time series in view of brevity's sake, but what shown definitely can extend to higher-dimension domain. At the end, I will briefly discuss common applications can derive from this method.


![Fig01](/assets/blog/2023-07-02/decomposition_sinusoid.png){:data-width="1440" data-height="836"}
Fig. 1. The function (or blue time series) $$f(t)$$ that is expressed in time axis $$t$$ is the summation of a set of sine functions that have different phases and amplitude. Each sinusoid have its "contribution", i.e. amplitude, indicated on the frequency axis $$w$$.  
{:.figure}

## Background knowledge

To make sense of how the algorithm works, certain concepts and characteristics to be defined and proved first. 

A **sinusoid** constitutes the basic elements of Fourier transform. Its mathematical form is a sine or cosine function, which is a smooth, repetitive oscillation that fluctuates symmetrically around a central value. It is characterized by its amplitude $$A$$, angular frequency $$w$$, and phase $$-\theta^\prime$$ (negative sign is not a typo but a mathematical trick to facilitate a different representation of sinusoid that is to be explained later). 

Fig. 2. depicts three such sinusoids in different value of said parameters. Amplitude $$A$$ determines the maximum height of the oscillation, for example the blue wave is spread four times further than the red and green wave. Phase $$-\theta^\prime$$ indicates the starting point of the wave, with $$-\theta^\prime=0$$ is the wave's peak and $$-\theta^\prime=\pi$$ radian the wave's trough. Lastly, if the wave takes $$T$$ seconds to complete one periodic cycle, and a complete cycle is 360 degree or $$2 \pi$$ in radian unit, then the frequency $$w=2 \pi/T$$ is the rate of radians per second. Therefore, we need both $$w$$ and $$-\theta^\prime$$ to figure out the position of waveform in its cycle at a specific point in time. 

![Fig02](/assets/blog/2023-07-02/cosine_function.png){:data-width="1440" data-height="836"}
Fig. 2. Three sinusoids illustrate the effect in wave shape that is determined by the three parameters amplitude $$A$$, angular frequency $$w$$, and initial phase $$-\theta^\prime$$.  
{:.figure}

We then proceed to show that **any sinusoid can be expressed as a linear combination of a sine and cosine function** in the form:

<br/> $$\hspace{10pt} g(t) = A cos(\phi_t -\phi^\prime) = C cos\phi_t + D sin\phi_t$$

Note that $$\phi_t = wt$$ and I have reordered $$A cos (-\phi^\prime + wt) = A cos(-\phi^\prime + \phi_t )$$ into $$A cos(\phi_t -\phi^\prime)$$. The proof goes as follows: 
<br/>
<br/> $$\hspace{25pt} A cos(\phi_t -\phi^\prime) = C cos\phi_t + D sin\phi_t$$
<br/> $$\hspace{10pt} \Leftrightarrow A (cos\phi_t cos\phi^\prime + sin\phi_t sin\phi^\prime) = C cos\phi_t + D sin\phi_t$$, given the trigonometric identity $$cos(\alpha - \beta) = cos\alpha cos\beta + sin\alpha sin\beta$$
<br/> $$\hspace{10pt} \Leftrightarrow \boldsymbol{[A cos\phi^\prime]}cos\phi_t  + \boldsymbol{[A sin\phi^\prime]}sin\phi_t  = \boldsymbol{C} cos\phi_t + \boldsymbol{D} sin\phi_t$$
{:.message}

As a result, we have $$\boldsymbol{C = A cos\phi^\prime}$$ and $$\boldsymbol{D = A sin \phi^\prime}$$. Furthermore, given $$sin^2 \phi^\prime + cos^2 \phi^\prime = 1$$, we have $$\boldsymbol{A=\sqrt{C^2+D^2}}$$ and $$\boldsymbol{\phi^\prime=arctan(D/C)}$$. This interestingly implies one can represent a sinusoid with either pair of parameters: the amplitude $$A$$ and initial phase $$\phi^\prime$$, or the amplitude $$C$$ of a cosine and amplitude $$D$$ of a sine whose summation construct the same sinusoid.

Indeed, Fig. 3. show that for any sinusoid $$A cos(\phi_t -\phi^\prime)$$ with known $$A$$ and  $$\phi^\prime$$, one can compute the counterpart pair $$C$$ and $$D$$ and the visualization intuitively verifies that mathematical property.

![Fig03](/assets/blog/2023-07-02/sinusoid_sine_cosine.png){:data-width="1440" data-height="836"}
Fig. 3. Sinusoids with three different parameter set $$[A=1, \phi^\prime=0 \text{ rad}]$$, $$[A=1, \phi^\prime=-\pi/2 \text{ rad}]$$, $$[A=1, \phi^\prime=0.393 \text{ rad}]$$, displayed in black, and their decomposition to sine and consine functions, blue and red respectively.  
{:.figure}

Next, we define **sinusoid basis functions a family of functions whose sum can approximate any function $$f(t)$$**. Note that here we meant *any* function, not just sinusoid one. 


Let's say given an arbitrary function $$f(t)$$ and its input time argument $$t \in [0, T]$$, then the first sinusoid basis will have its angular frequency $$w_1 = 2\pi/T$$ (rad/s). In other words, we say the first-order approximation of $$f(t)$$ is a sinunoid whose period is $$T$$ seconds. We call $$w_1$$ the **fundamental frequency**. Naturally, to better approximate, we need to fill in more intricate functions, namely **harmonics** of $$w_1$$ with successive value $$n \in \mathbb{N}$$ such that:

<br/> $$\hspace{60pt} w_1 = 1 \times w_1,$$
<br/> $$\hspace{60pt} w_2 = 2 \times w_1,$$
<br/> $$\hspace{60pt} w_3 = 3 \times w_1,$$
<br/> $$\hspace{60pt} ...$$
<br/> $$\hspace{60pt} w_n = n \times w_1$$

Therefore, the approximation of a function $$f(t)$$ with sinusoidal basis functions would be

$$f(t) = \sum_{n \in \mathbb{N}} A_n cos(w_nt - \phi^\prime_n) = \sum_{n \in \mathbb{N}} g_n(t) $$

And because a sinusoid can be expressed as a linear combination of a sine and cosine function, hence

$$f(t) = \sum_{n \in \mathbb{N}} C_n cos(w_nt) + D_n sin(w_nt)$$


Lastly, by this construction, **we can leverage on the orthogonal properties of basis function to obtain Fourier coefficients $$C$$ and $$D$$** in the above formula for each component sinunoid. Let's say we want to get coefficient $$C_m$$ of harmonics $$w_m = m \times w_1$$; this can be achieved by multiplying $$f(t)$$ by $$cos(w_mt)$$ and integrating over $$t$$ between $$-T/2$$ and $$T/2$$:


$$\int_{-T/2}^{T/2} f(t) cos(w_mt)dt 
= \int_{-T/2}^{T/2}[\sum_{n \in \mathbb{N}} C_ncos(w_nt) + D_nsin(w_nt)] cos(w_m)dt 
= \sum_{n \in \mathbb{N}} C_n \int_{-T/2}^{T/2} cos(w_nt) cos(w_m)dt + \sum_{n \in \mathbb{N}} D_n \int_{-T/2}^{T/2}sin(w_nt) cos(w_m)dt$$

Despite its intimidating expansion, because any pair $$sin(w_nt)cos(w_mt) = 0$$ ($$\forall m, n$$) and $$cos(w_nt)cos(w_mt) = 0$$ ($$\forall m \neq n$$), we can simplify into

$$\int_{-T/2}^{T/2} f(t) cos(w_mt)dt 
= C_m\int_{-T/2}^{T/2} cos(w_nt) cos(w_mt) dt
= C_m\int_{-T/2}^{T/2} cos^2(w_mt) dt
= C_m \frac{T}{2}
$$

Because all variables are known except for $$C_m$$, hence

$$C_m = \frac{2}{T} \int_{-T/2}^{T/2} f(t) cos(w_mt) dt$$

Likewise, by multiplying $$f(t)$$ by $$sin(w_mt)$$, we obtain

$$D_m = \frac{2}{T} \int_{-T/2}^{T/2} f(t) sin(w_mt) dt$$

I believe you have doubt about the simplification step, which I think can better explain in the Fig. 4. Each row shows two exampled waves whose frequencies are harmonics to a fundamental frequency $$w$$ and an example of their multiplication over the time $$t$$ axis. 

The first row, for example, given the two waves $$\boldsymbol{sin(\phi)}$$ and $$\boldsymbol{cos(2\phi)}$$, the integral of their multiplication, i.e. $$\boldsymbol{\int_{-T/2}^{T/2} sin(\phi) cos(2\phi) dt} $$, is the summation of shaded area in the $$t$$-interval $$[-T/2, T/2] = [-5, 5]$$. You could notice that the positive side and negative side cover symmetrically the same amount of area, hence results in a total sum of zero. This holds true for the second example where $$\boldsymbol{\int_{-T/2}^{T/2} cos(\phi) cos(2\phi) dt = 0}$$. Lastly, $$\boldsymbol{\int_{-T/2}^{T/2} cos^2(\phi) dt} = T/2$$, again, can be shown in the figure.

Even though they are examples, this does not lose generality for other such pairs with varying frequencies, as long as their frequencies are harmonics. 

![Fig04](/assets/blog/2023-07-02/sin_cos2.png){:data-width="1440" data-height="836"}
![Fig04](/assets/blog/2023-07-02/cos_cos2.png){:data-width="1440" data-height="836"}
![Fig04](/assets/blog/2023-07-02/cos_cos.png){:data-width="1440" data-height="836"}
Fig. 4. **Row 1**: The integral of any $$sin(nwt) \times cos(mwt)$$ is zero. **Row 2**: The integral of any $$cos(nwt) \times cos(mwt)$$ is zero. **Row 3**: The integral of any $$cos^2(nwt)$$ is $$\frac{T}{2}$$. 
{:.figure}


## Algorithm

A picture is worth a thousands words, let's see at play.
first 2 sinunoids capture, the rest has amplitude next to 0, hence 

![Fig05](/assets/blog/2023-07-02/fourier_transform.png){:data-width="1440" data-height="836"}
Fig. 5. **Column 1**: An arbitrary function $$f(t)$$. **Column 2**: Applying Fourier transform and compute the sinunoidal basis functions that approximate $$f(t)$$ in form of $$A_n cos(w_nt - \phi^\prime_n), \forall n \in \mathbb{N}$$. **Column 3**: Reconstruct $$f(t)$$ using the formula $$\sum_{n=1}^\mathbb{N} A_n cos(w_nt - \phi^\prime_n)$$. **Column 4**: The plot of $$A_n$$ to see the contribution of each frequency $$w_n = n \times w_1$$ in the basis functions. 
{:.figure}


```python
x, ft, T = generate_curve_ft()
fundamental_frequency = 2 * math.pi / T
n_frequencies = 500
freqs, Cs, Ds = fourier_transform(fundamental_frequency, n_frequencies)
As, thetas, sinunoids = compute_sinunoidal_basis_function(Cs, Ds, x)
reconstructed_ft = np.sum(sinunoids, axis=0)
```

```python
def generate_curve_ft():
    T = 5
    x = np.linspace(0, T, 1000)
    sin_y1 = get_sinewave_y(1, 2.5, 0, x)
    sin_y2 = get_sinewave_y(0.5, 5, 0.5, x)
    ft = plus_waves(sin_y1, sin_y2)

    return x, ft, T

def plus_waves(Y1, Y2):
    return Y1 +  Y2
```

```python
def fourier_transform(fundamental_frequency, n_frequencies):
    freqs, Cs, Ds = list(), list(), list()


    for n in range(1, n_frequencies+1, 1):
        harmony_freq_n = n * fundamental_frequency
        T_n = 2*math.pi/harmony_freq_n
        cosine_wave_freq_n = get_cosinewave_y(A=1, T=T_n, theta=0, x=x)
        C_n = compute_integral_of_wave_multiplication(ft, cosine_wave_freq_n)

        sine_wave_freq_n = get_sinewave_y(A=1, T=T_n, theta=0, x=x)
        D_n = compute_integral_of_wave_multiplication(ft, sine_wave_freq_n)

        freqs.append(harmony_freq_n)
        Cs.append(C_n)
        Ds.append(D_n)

    return freqs, np.array(Cs), np.array(Ds)

def get_sinewave_y(A, T, theta, x):
    y = [A * math.sin(theta + 2*math.pi*i/T) for i in x]
    return np.array(y)

def get_cosinewave_y(A, T, theta, x):
    y = [A * math.cos(theta + 2*math.pi*i/T) for i in x]
    return np.array(y)

def compute_integral_of_wave_multiplication(Y1, Y2):
    return np.dot(Y1, Y2) # inner product is equivalent to element-wise multiplication, then summation
```

```python
def compute_sinunoidal_basis_function(Cs, Ds, x):
    As = ((Cs**2 + Ds**2) ** 0.5)
    thetas = np.arctan(Ds / Cs)

    sinunoids = list()
    for harmony_freq_n, theta_n, A_n in zip(freqs, thetas, As):
        T_n = 2*math.pi/harmony_freq_n
        sinunoid_n = get_cosinewave_y(A=A_n, T=T_n, theta=-theta_n, x=x)
        sinunoids.append(sinunoid_n)

    return As, thetas, sinunoids
```

## Properties of Fourier Transform


marginalization: contribution to f(t)
wavelength: distance between adajance peaks

convolution theorem.

decompose structure, 

Outcome on 2D images are more exciting, but not within the scope of this study

compression, feature engineering, noise cancelling. extract feature and manipulate the original images.

how to get amplitude

## Summary

My appreciation to James V Stone for the great book: <a href="https://www.amazon.com/Fourier-Transform-Tutorial-Introduction-ebook/dp/B0BN22Q8WW">The Fourier Transform: A Tutorial Introduction</a>, on which inspired me to write the post in the first place. 

---


