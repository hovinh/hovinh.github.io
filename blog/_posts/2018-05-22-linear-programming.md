---
#layout: post
title: Linear programming - A powerful problem solving method that works effectively in practice but is provably hard in principle
description: >
  A tutorial briefly introduces linear programming and its toolkits.
author: author1
comments: true
---

There was a time, scientists had to predict nature with pure reasoning and looked for affirmative from observations. There was a time, practitioners could barely scan through all the possible solutions to analyse the correctness of a problem, and thus had to ask for help from mathematicians. And, there was a time, when mathematics listed a method as hard to solve (in terms of polynomial time) in principle then it came out to be a good fit in practice. Among those, there is Linear Programming (LP).

This post will give you a brief overview of how LP works, suggest several useful solvers,  and assuredly to make you comfortable enough to use them, though it took, to say the least, some efforts to adapt it to your own problem. In other words, this is an introduction from engineering's point of view. Audiences who are interested in its underlying principle could easily look up for plenty helpful blog posts or Optimization-related textbook.

## Formulation
LP is an optimization method, in which given a set of variables together with an objective function, can help us to find the optimal combination of values of this set, with respect to some constraints on its domain.

More formally, given a vector of variables $$x = (x_1, x_2, x_3)$$, 2 vectors $$c$$,  $$b$$ and a matrix $$A$$ of fixed values, $$c^{\intercal}$$ is the transpose of c, we can express our problem [1] as

maximize $$\hspace{35pt}c^{\intercal}x$$ <br/> subject to $$\hspace{30pt}Ax \leq b$$ <br/> and $$\hspace{60pt}x \geq 0$$
{:.message}

Simply put, so long as you can formulate any problem in this particular form, then it is <em>likely</em> to be solved. The LP solver has the ability to identify a non-solvable problem, no possible values of $$x$$ satisfy the constraints, so the term <em>likely</em> here refers to the running time.

<span style="font-weight:bold; font-style: italic;">Example 1</span>: Given a bag of coins, if I allow you to take at most 40 coins, how can you maximize the total values of the selected coin. You know that there are 3 types of coins: Galleon ($6.64), Sickle ($0.39), and Knut ($0.01)[2], and you could not take more than 20 Galleons, 10 Sickles but there is no limitation on Knut. Let's call the number of the coins in decreasing order of value as $$g$$, $$s$$, $$k$$, the formulation is as follows

maximize: $$\hspace{16pt}6.64 \cdot g + 0.39 \cdot s + 0.01 \cdot k$$ <br/> subject to <br/> 
$$
\begin{aligned}
  \hspace{60pt} g &\leq 20 \\[0.5em]
  \hspace{60pt} s &\leq 10
\end{aligned}
$$
<br/> and <br/>
$$
\begin{aligned}
  \hspace{60pt} g \geq 0 \\[0.5em]
  \hspace{60pt} s \geq 0 \\[0.5em]
  \hspace{60pt} k \geq 0
\end{aligned}
$$
{:.message}

One does not need to sketch on paper to know that the optimal solution is $$(g, s, k) = (20, 10, 10)$$. However, this example serves the purpose to familiarize you with the formulating process rather than demonstrate the capacity of this method.

Note that, all variables must be first order to remain the linearity, in other words, can only be $$g$$, not $$g^2$$, $$g^3$$, or higher order.

When I first come across this method, instantly do I look for a connection to Machine Learning, since essentially it is an optimization problem. To who may not be familiar with Machine Learning, this is a field that studies the pattern of data. Provided a data point and its corresponding label (an image of a cat and the label says "Cat", for instance), we need to find a model that automatically learns the mapping from data to its true label. This task is called classification. Admittedly, this is an oversimplified explanation, but it does suffice for the following example.

Intuitively, for the classification task, we need to minimize the loss function, which is the number of misclassified data points. However, what are the variables we need to solve? It turns out a bit tricky to actually formulate into the standard format.

<span style="font-weight:bold; font-style: italic;">Example 2</span>: Given a set of $$n$$ 2-dimension data points $$X = \{(x_{11}, x_{12}), (x_{21}, x_{22}), ..., (x_{n1}, x_{n2})\}$$ and the set of corresponding binary labels $$Y = \{-1, 1\}^n$$, find the weight vector w that correctly classified every element of $$ X $$, i.e. $$ sign ( w^{\intercal} x_{i} \cdot y_i ) = 1 \cdot sign ( x )$$ is 1 if the sign of x is positive and -1 otherwise, for any data point $$x_i = (x_{i1}, x_{i2})$$. The formulation is

maximize $$\hspace{60pt} \sum_{i \in [1, n]} sign ( w^{\intercal} x_{i} \cdot y_{i} )$$ <br/> subject to
$$\hspace{60pt}w_1, w_2 \in \mathbb{R}$$ (or no constraint)
{:.message}

This might look confusing at first glance, but let me break it down bit by bit. Basically, we assume that the data points have 2 classes, $$-1$$ and $$1$$, and there is a line that can separate them. The target line, or variables that need to be optimized, is in fact $$w$$, with $$w^{\intercal} x_{i}$$ is the prediction. Any point that lies above or on the line, $$w^{\intercal} x_{i} \geq 0 $$, have predicted label as $$1$$. Likewise, any point that lies below having $$w^{\intercal} x_{i} < 0 $$,  or predicted label as $$-1$$. From this observation, if we correctly classify a point, the multiplication of the actual value and prediction will always be a positive value, which has $$sign ( w^{\intercal} x_{i} ) = 1$$. There is no constraint on the value of $$w$$, hence we complete the formulation.

The point of this example is showing that it takes some analysis to deduce the appropriate form of LP. Later on, when learning of Support Vector Machine, I realize that this idea has been exploited already!!! Additionally, they modify elegantly to handle even the nonlinear separable data.

## Linear Programming Solvers
 
Okay, I now know how to formulate my problem, but how exactly do I solve them? Fortunately, there are plenty solvers out there can do the work once you feed them the formulation. For students, I recommend Gurobi, since it is commercial hence a good solver but also provides a free academic license. Other choices would be CPLEX, GLPK or standard library on your prefered programming language (MATLAB, Python).

### Gurobi
+ Website: http://www.gurobi.com/
+ Gurobi downloaded package provides mini problem sets in each programming language to help user get accustomed to.
+ Gurobi has API for most common-used languages: C, Java, Python, MATLAB.... Besides, it is possible to use as a software as well. In fact, first test out Gurobi with the software before proceeding to use API is a good idea. Try with Mip1 problem set.
+ All required information of installation and uses are <a href="http://www.gurobi.com/documentation/7.5/quickstart_mac.pdf">here</a>.
+ Issues can meet when installing on Window: <a href="https://www.urtech.ca/2014/12/solved-installation-error-code-2502-2503/">here</a> and <a href="https://www.urtech.ca/2016/02/solved-how-to-fix-error-2503-2502-on-windows-10-when-installing-software/">here</a>.
+ Window is easier to install than Ubuntu. However, when running on a server, usually there is no choice other than Ubuntu. A helpful referent <a href="http://abelsiqueira.github.io/blog/installing-gurobi-7-on-linux//">blog</a> of Abel. A few notes:
  - remember to check if <em>grbgetkey</em> file is executable or not: 
  ```s
  chmod +x file_name
  ```
  - check if a file is hidden or key file in hidden folder.
  - when I use Java to compile (command javac, java), you need to link to Gurobi folder. You can save this path to <em>bashrc</em> file. All declared path must be direct to folders, should not use relative path name like ~. A helpful <a href="https://groups.google.com/forum/#!topic/gurobi/eTtUNAw8Ed8">thread</a>.
  ```s
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/vinh/opt/gurobi752/linux64/bin/
  ```

+ Set <a href="http://www.gurobi.com/documentation/6.5/refman/parameters.html">parameters</a> for Gurobi <a href="http://www.gurobi.com/documentation/7.0/refman/java_parameter_examples.html#JavaParameterExamples">here</a>, though is is optimized by default.
+ If you install Gurobi with an academic license properly, you must have the following line printed out every time you call the library:
  ```s
  Academic license - for non-commercial use only
  ```

### Other Solvers
+ <a href="http://www.gnu.org/software/glpk/">GLPK</a>. All credit on the GLPK <a href="https://drive.google.com/drive/folders/1CIRzhYz5nh37BWiprCZW1sB4LKhfSIW0">tutorial</a> goes to Mithun Chakraborty.
+ Python: <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linprog.html">scipy.optimize.linprog</a>. Quite limited in terms of solvability.
+ PuLP: a helpful series of <a href="http://benalexkeen.com/linear-programming-with-python-and-pulp/">blog posts</a> written by Ben Keen.

## Conclusion
As stated previously, this post is for engineers, developers etc. who seek for a solution for their problem that matches the class of Linear Programming. It is noteworthy that at first, it might not cross your mind that they have any similarity, but the ability to formulate a problem is itself an art in my opinion. Next time, if you have an optimization problem, try this 😉.

Hopefully, you find this post useful in some ways. Please feel free to make any suggestion on this post or share your opinion on the pros and cons of solvers you have tried.

## References

<ol> 
  <li>https://en.wikipedia.org/wiki/Linear_programming</li>
  <li>http://harrypotter.wikia.com/wiki/Wizarding_currency</li>
</ol>
