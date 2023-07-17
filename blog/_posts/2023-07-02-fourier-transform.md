---
#layout: post
title: Fourier Transform
description: >
  Fourier Transform is a novel method that is widely used in signal processing. This post to explain the intuition behind its mathematical operations and celebrate the beauty of it. 
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

And because a sinusoid $$g_n(t)$$ can be expressed as a linear combination of a sine and cosine function, hence

$$f(t) = \sum_{n \in \mathbb{N}} C_n cos(w_nt) + D_n sin(w_nt)$$


Lastly, by this construction, **we can leverage on the orthogonal properties of basis function to obtain Fourier coefficients $$C$$ and $$D$$** in the above formula for each component sinunoid. Let's say we want to get coefficient $$C_m$$ of harmonics $$w_m = m \times w_1$$; this can be achieved by multiplying $$f(t)$$ by $$cos(w_mt)$$ and integrating over $$t$$ between $$-T/2$$ and $$T/2$$:


$$\int_{-T/2}^{T/2} f(t) cos(w_mt)dt 
= \int_{-T/2}^{T/2}[\sum_{n \in \mathbb{N}} C_ncos(w_nt) + D_nsin(w_nt)] cos(w_m)dt 
= \sum_{n \in \mathbb{N}} C_n \int_{-T/2}^{T/2} cos(w_nt) cos(w_m)dt + \sum_{n \in \mathbb{N}} D_n \int_{-T/2}^{T/2}sin(w_nt) cos(w_m)dt$$

Despite its intimidating expansion, because any pair $$\boldsymbol{sin(w_nt)cos(w_mt) = 0, (\forall m, n)}$$ and $$\boldsymbol{cos(w_nt)cos(w_mt) = 0, (\forall m \neq n)}$$, we can simplify the equation into

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

Even though they are examples, this does not lose generality for other similar pairs with varying frequencies, as long as their frequencies are harmonics. 

![Fig04](/assets/blog/2023-07-02/sin_cos2.png){:data-width="1440" data-height="836"}
![Fig04](/assets/blog/2023-07-02/cos_cos2.png){:data-width="1440" data-height="836"}
![Fig04](/assets/blog/2023-07-02/cos_cos.png){:data-width="1440" data-height="836"}
Fig. 4. **Row 1**: The integral of any $$sin(nwt) \times cos(mwt)$$ is zero. **Row 2**: The integral of any $$cos(nwt) \times cos(mwt)$$ is zero. **Row 3**: The integral of any $$cos^2(nwt)$$ is $$\frac{T}{2}$$. 
{:.figure}


## Discrete Fourier Transform Algorithm

With the background out of the way, I hope that the to-be-shown algorithm will come out in a positive light, such that the employed numerical operations accompanied by a great sense of intuition. The term "discrete" means the basis function takes whole-number multiples of a fundamental frequency.

Fig. 5. depicts the Fourier Transform at play when apply it on a synthetic curve $$f(t)$$. The curve is decomposed into 500 cosine waves whose fundamental frequency and harmonic $$n_2$$ having great amplitude and higher frequency $$w_3, w_4, ..., w_{500}$$ having amplitude approximately 0. Taking the summation of the 500 waves can lead to a near-perfect reconstruction $$f^\prime(t)$$. Lastly, by plotting each frequency $$w_n$$ in the sinunoidal basis functions on the x-axis and their amplitude $$A_n$$ on the y-axis, we can form the amplitude spectrum of the curve $$f(t)$$. This shows which frequency contributes the highest to the shape of $$f(t)$$, which is expectedly the $$w_1$$ and $$w_2$$.

![Fig05](/assets/blog/2023-07-02/fourier_transform.png){:data-width="1440" data-height="836"}
Fig. 5. **Column 1**: A synthetic curve $$f(t)$$. **Column 2**: Applying Fourier transform and compute the sinunoidal basis functions that approximate $$f(t)$$ in form of $$A_n cos(w_nt - \phi^\prime_n), \forall n \in \mathbb{N}$$. **Column 3**: Reconstruct $$f^\prime(t)$$ using the formula $$\sum_{n=1}^\mathbb{N} A_n cos(w_nt - \phi^\prime_n)$$. **Column 4**: The plot of $$A_n$$ to see the contribution of each frequency $$w_n = n \times w_1$$ in the basis functions. 
{:.figure}

I use the code below to produce the figure. 

```python
x, ft, T = generate_curve_ft()
fundamental_frequency = 2 * math.pi / T
n_frequencies = 500
freqs, Cs, Ds = fourier_transform(fundamental_frequency, n_frequencies)
As, thetas, sinunoids = compute_sinunoidal_basis_function(Cs, Ds, n_frequencies, x)
reconstructed_ft = np.sum(sinunoids, axis=0)

f, (ax0, ax1, ax2, ax3) = plt.subplots(nrows=1, ncols=4, figsize=(20, 5))

# synthetic curve ft
ax0.plot(x, ft, color='black')
ax0.set_title(r'random function $f(t)$')
ax1.set_xlabel(r'time $t$')

# obtain sinunoidal basis function
for sinunoid_n in sinunoids:
    ax1.plot(x, sinunoid_n)
ax1.set_title(r'sinunoidal basis function $A_n cos(w_nt - \theta^\prime_n)$')
ax1.set_xlabel(r'time $t$')

# reconstructed functions
ax2.plot(x, reconstructed_ft)
ax2.set_title(r'reconstructed $f^\prime(t) = \sum_{n=1}^N A_n cos(w_nt - \theta^\prime_n))$')
ax2.set_xlabel(r'time $t$')

# amplitude spectrum
ax3.plot(freqs,As)
ax3.set_xlabel('frequency (Hz)')
ax3.set_ylabel('amplitude')
ax3.set_title('amplitude spectrum')
plt.show()
```

Firstly, I create a synthetic curve $$f(t)$$ whose value is the summation of two arbitrary sine waves, but I picked their frequencies such that the pattern of $$f(t)$$ will repeat every $$T=5$$ seconds. This is no means a constraint and you can replace with an arbitrary non-periodic function.
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

Secondly, I decided that I want 500 sinunoidal basis functions to approximate $$f(t)$$, whose fundamental frequency $$w_1 = 2 \pi/T = 2\pi/5$$ and subsequently $$w_2 = 2 \times w_1 = 4\pi/5$$, $$w_3 = 3 \times w_1 = 6\pi/5$$, and so on. For each frequency $$w_n$$, utilizing the orthogonal properties by multiplying $$f(t)$$ with $$cos(w_nt)$$ and $$sin(w_nt)$$, you can obtain $$C_n$$ and $$D_n$$ respectively. With this info, you essentially have decomposed successfully the curve into multiple sine and cosine waves.  
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

Thirdly, utilizing the premise of any sinunoid can be expressed as a summation of sine and cosine functions, we use $$C_n$$ and $$D_n$$ to compute the amplitude $$A_n$$ and initial phase $$\theta^\prime_n$$. Take note that we need to normalize each $$A_n$$ by dividing by 500. With this, the rest of the code is to take the summation of 500 sinunoids to compute $$f^\prime(t)$$, a reconstruction of $$f(t)$$. 
```python
def compute_sinunoidal_basis_function(Cs, Ds, n_frequencies, x):
    As = ((Cs**2 + Ds**2) ** 0.5) / n_frequencies
    thetas = np.arctan(Ds / Cs)

    sinunoids = list()
    for harmony_freq_n, theta_n, A_n in zip(freqs, thetas, As):
        T_n = 2*math.pi/harmony_freq_n
        sinunoid_n = get_cosinewave_y(A=A_n, T=T_n, theta=-theta_n, x=x)
        sinunoids.append(sinunoid_n)

    return As, thetas, sinunoids
```

The full code can be found in this <a href="https://github.com/hovinh/fourier-transform">Github</a>. Please note that the code favours explainability over performance, hence is not optimized. In practice, people commonly use fast Fourier Transform instead.

## Applications

To recap, the Fourier Transform enables the transformation of a signal (or function) $$f(t)$$ into a set of basis functions $$g_1(t), g_2(t),$$... on the same input domain, that in turn can compute the amplitude (or "contribution") of those functions onto the construction of $$f(t)$$. The near-perfect two-way representation hints at the ability to study a signal at a closer look by decomposing it into core components, and imagine, if you will, to retain certain components and get rid of the rest, a nicer form of the original signal can be revealed.

My argument may better illustrates with an example of a sound recording. Let's say the record is polluted by background noise, then by leveraging on Fourier Transform and plenty of trial-and-error to know which basis function is the source of noise, you can remove it from the equation and enhance the sound quality. Of course this is an extreme, but it's not so further away from denoising a time series of interest by retaining only frequencies with above-a-threshold "contribution".

Alternatively, Fourier Transform is a feature engineering process to transform any time-varying (with acceptable tolerance) signal into a fixed-size tabular format. This can be done with the amplitude spectrum, i.e. the contribution profile of each sinunoidal basis function to the original signal. Frame it differently, this is also a dimensionality reduction method should you use $$M$$ data points to represent the signal, then instead $$n << M$$ frequencies can approximately capture the same information.

The same can apply for 2D image: instead of the univariate time axis, we have the bivariate Cartesian coordinate in x and y axis to capture the "signal" $$f(x, y)$$. In the context of image processing, a filter is an image captures specific edges, corners, or textures of the original image, that is no different from the decomposition basis functions of Fourier Transform. One can, for example, explore the idea of initializing Convolutional Neural Network's kernels with these, potentially promise a more robust features and quicker convergence. 

## Summary

In this post, I have sketched out the intuition foundation behind the numerical operations employed in discrete Fourier Transform. I illustrate the algorithm with a synthetic curve and go through the code snippets that product it. Lastly, some ML applications have been discussed in the context of signal such as 1-D sound recording or 2-D image. I hope that this post make you understand better the idea behind Fourier Transform and can adapt its idea in the upcoming signal analysis or ML tasks.

My appreciation to James V Stone for the great book: <a href="https://www.amazon.com/Fourier-Transform-Tutorial-Introduction-ebook/dp/B0BN22Q8WW">The Fourier Transform: A Tutorial Introduction</a>, on which inspired me to write this post in the first place. 

---


