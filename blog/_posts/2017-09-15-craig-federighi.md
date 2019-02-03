---
#layout: post
title: Craig Federighi did not lie about FaceID and its face detection accuracy
description: >
  A brief discussion of FaceID in lens of confusion matrix.
author: author1
comments: true
---

After September 12 Apple-event, the news of the new iPhone X and its cutting-edge facial recognition technology, namely FaceID, run all over the place. Besides a big leap in price in mobile-market making people suddenly be aware of their internal organ value, an inevitable question pops up about the reliability of FaceID. On one hand, it's totally convenient to look at the screen and the phone automatically unlocks itself. On the other hand, such kind of privacy protection was proved to be imprecise from time to time and could be faked by a whole bunch of simple (or not) tricks: a photo of the owner, her video, or even a 3D model.

Things went crazier right after "Apple claims that Face ID is incredibly accurate and secure, and states that there’s a 1 in 1,000,000 chance of a passer-by on the street gaining access, compared to 1 in 50,000 with Touch ID"[1], Craig Federighi's demo did not go as planned when he failed to unlock his own phone[2].  As usual, the media never misses a chance to twist the trust, mislead people to a so-called "true" fact. Technically, Craig didn't lie, even if FaceID actually did not recognize him.

![Batman Rocky](/assets/blog/2017-09-15/batman_rocky.jpg){:data-width="1440" data-height="836"}
Source: https://9gag.com/
{:.figure}

As a scientist, I feel my responsibility to unveil the truth behind this with a simple tool, Confusion Matrix. Before getting your hands dirty, I want to make a statement: there is no <strong>contradiction</strong> between [1] and [2], hence what Craig did should not be an appropriate approximation of the <strong>true error</strong> of  FaceID. To be clearer, I want to put security matter all aside, and just focus on the claim [1], merely about accuracy/error.

To address this problem, the aforementioned terms should be more well-defined. Take the FaceID, for instance, it must decide the one facing the screen is its one-and-only owner or not. In this context, accuracy means it (a) correctly recognize its owner and (d) correctly detect strangers as well. On the contrary, the error means it (c) mistakes owner as strangers and (b) mistakes strangers as owner. At this point, [1] clearly implies (b), whereas [2] implies (c). In other words, they are two separate sets such that intersection is an empty set. Their union would be the total error of FaceID.

A more general and formal definition could be represented by Confusion Matrix. This is a $$n \times n$$ matrix, such that $$n$$ is the number of options a classifier (an algorithm which has an ability to make a decision/prediction) can choose. The simplest form would be a $$2 \times 2$$ matrix, that is classifying an object into type A or B, OR, detect an object is whether it belongs to type A or not. Apparently, FaceID falls into the second scenario.

Now, in order to construct this matrix, each column is corresponding to the ground truth and each row is the decision/prediction made by the algorithm. Imagine before making the claim [1], Apple conducted an experiment to test the performance of FaceID, and you are an observer who has the job to create the Confusion Matrix and gives the CEO the true accuracy/error. Ground truth means you know the real owner of the phone, whereas prediction is locking/unlocking the screen behavior of FaceID, implying if it recognizes owner or not. Once this is clear, I will introduce 4 new terms:

- True Positive: predict as type A and the ground truth is type A.
- False Positive: predict as type A and the ground truth is not-type  A.
- True Negative: predict as not-type A and the ground truth is not-type A.
- False Negative: predict as not-type A and the ground truth is type A.
I'm positive that the above list is self-explanatory, and we simply calculate accuracy/error by counting samples in below table.

$$
\begin{aligned} 
accuracy =& \frac{true positive \: + \: true negative}{total samples}\\[0.5em]
error =& \frac{false positive \: + \: false negative}{total samples}
\end{aligned}
$$

||**Actually Owner**|**Actually Stranger**|
|-------------------------|-------------------------|-------------------------|
|**Recognize as Owner**|(a)True Positive|(b)False Positive|
|**Recognize as Stranger**|(a)False Negative|(b)True Negative|

To push this a bit further, as a user, our concerns are preventing strangers from having access to the phone, therefore it does not matter much (besides user experience) if it fails to recognize you. In some problems, one false must be treated seriously than the other one. From the perspective of security, mistaking strangers as user causes much damage than the other way around, for example.

Other 2 True terms are not less important, in fact, people invent several terms constructed from these 4 cells. For a better understanding of Confusion Matrix, I suggest checking [3].

In conclusion, the point I want to make is that people should not make any judgments simply from 2 seemingly similar cases, but actually turns out to be 2 different things. And, you know how to verify that for such news in the future, don’t you?

## References

<ol> 
  <li>http://www.macworld.co.uk/feature/iphone/how-secure-is-face-id-3663992/</li>
  <li>http://www.bbc.com/news/technology-41266216</li>
  <li>https://machinelearningmastery.com/confusion-matrix-machine-learning/</li>
</ol>