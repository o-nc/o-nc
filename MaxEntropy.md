
I built a [C++23 library](https://github.com/o-nc/MaxEntropy) to compute the probability distribution having maximal entropy over a domain and given a set of constraints.

### Why  
I never liked parametric statistics, as it is often employed without justifying why a family of distributions should match the data used. 
While a qualitative property sometimes justifies a distribution, e.g. the central limit theorem justifying normality, 
the choice of distribution often comes down to the ease of use and availability of a particular well-known technique.
For exploring data quickly, this is often fine. However, there are many problems where it is known that the data does not fit the distributional assumptions of the methods used.

While there are some nonparametric statistics methods that allow to bypass these unsupported assumptions, newer methods from machine learning using neural networks are by far the most popular. 
Such methods work because any function can be approximated by neural network given enough parameters and compute power. They lack human understandable explanatory power, however.

### Entropy
In information theory, entropy is a measure of the amount of information needed to describe, on average, a state in a system.
The larger the entropy, the more information is needed to describe an average state. In other words, a system is freer or less constrained as its entropy increases.
Of all the probability distributions satisfying a set of constraints, the distribution having maximal entropy is thus the one having the least amount of additional requirements past the given constraints.

Instead of using the logistic distribution "because I want something similar to the normal distribution, but having higher kurtosis", 
maximum entropy allows one to find the distribution having exactly the right characteristics (in this case: mean, variance and kurtosis) while forgoing all the unnecessary constraints that come with the logistic distribution. 

Examples of distribution having maximal entropy are plenty. In fact, most well known distributions have maximum entropy over a set of constraints. The uniform distribution has maximal entropy over all distributions using the same domain.
The bernoulli, geometric, exponential, binomial and poisson distributions all have maximal entropy given their domain and a constraint on the value of their expectation.
Most famous of all, the normal distribution has maximal entropy over all distribution defined over the real line, given constraints on its expectation and its variance.


### Formulation of the problem

Mathematically, it is known that a maximum entropy probability distribution has the following form: $$p(x|\lambda)=\int_D\exp{\left(-\sum_{i=1}^n\lambda_if_i(x)\right)}dx \tag{1}$$
Where the domain is $D$, the constraints are the functions $f_i$ and $\lambda_i$ are Lagrange multipliers. This distribution solves the multivariate root-finding problem given by
$$\begin{equation} F(\lambda) =
\begin{cases}
\int_Df_1(x)p(x|\lambda)dx= C_1\\
\dots\\
\int_Df_n(x)p(x|\lambda)dx = C_n
\end{cases}       \tag{2}
\end{equation}$$
In $(2)$, the $C_i$ are real constants. While it is also possible to solve the problem for inequality constraints, this library only works for equality constraints.
To find the Lagrange multipliers, one needs to use a numerical multivariate root-finding algorithm. (to be continued...)

### This library
From the mathematical formulation, we see that we will need two main parts: numerical integration and numerical root finding.
To integrate functions, I built an adapter to the GNU Scientific Library (GSL).
While this allowed me to dedicate my work to building features not available elsewhere, this library as its disadvantages:
1. It requires the constraints to be of type `double(*)(double)` (that is, function pointers). This prevents capturing lambdas (but lambdas that do not capture are fine, as they alone can be converted to function pointers) and member methods from being usable.
2. It prevents error messages from being handled straightforwardly. GSL often sends `SIGABRT` signals when something goes wrong. Such signals require the use of `setjmp` and `longjmp` standard C library calls.
3. It creates the largest bottleneck in performance (>90% of the time). As can be seen from equation (2) above, many very similar functions need to be integrated (even more functions for the Jacobian matrix). 
GSL can't reuse computed values across different integration calls. As such, the functions involved need to be evaluated over the whole domain for each requested integration, the work currently increases with the cube! of the number of constraints.
This is because the Jacobian matrix grows quadratically with the number of constraints and the number of evaluations for each element of the matrix increases linearly. 
A custom-built integration routine could reuse the functions evaluations across all integral so that the work increases only linearly with the addition of new constraints. Because the integrals are recomputed inside an iterative algorithm, the added complexity is even more felt.

## To be continued...
- Density
- Numerical root-finding
- Challenges: Jacobian matrices are badly conditioned. Line search methods: backtracking, quadratic-cubic. Trust-region method: LevenbergMarquardtFan.

### Example usage: stock returns distribution, fat tails, power laws


### Summary
