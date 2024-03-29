# Details on Janushash 

## Interpreting Hashes as Numbers

- Each hash (32 byte hex string) can be interpreted as a number in the interval $[0,1)$. 
    - For example the hash `FFFF...FFFF` would correspond to 1-2^256 which is almost 1 and 
    - the hash `0000...0000` would correspond to 0. 
    - **Note**: A hash cannot correspond to exactly 1 but almost 1. 
- For a block header $h$ we denote
    - the verus hash interpreted as a number in $[0,1)$ by $X(h)$ and
    - the triple sha256 hash interpreted as a number in $[0,1)$ by $Y(h)$.
- We define the Janushash number $J(h) = X(h)Y(h)^{0.7}$.
- Similar to above where we converted a hash to a number we can do the reverse, i.e. interpret $J(h)$ as a hash, this hash is the Janushash but we will never compute it or work with it, instead we will solely consider the number representation $J(h)$. All theory works with numbers, so implementation only needs to convert hashes to numbers, but not numbers to hashes For convenience we will call the Janushash number also *Janushash*.
- **Note**: To represent a small number as a hash, one might require more than 32 digits, there exist transcendental numbers which even require infinite digits to be represent exactly. However this is not of interest for us.


## Condition to solve a block

For target $\tau\in[0,1]$ we define the following two conditions a header $h$ must satisfy to have a valid Proof of Balanced Work:

1. The Sha256t must not be too small $Y(h) \ge c$ for some constant $c=0.005$
2. The Janushash must be below the threshold $J(h) < \tau$.

An equivalent formulation is to require that $(X(h),Y(h))$ must be an element of the *acceptance region* $A_{\tau}\subset [0,1]^2$ defined as

$$
A_{\tau} := \big\lbrace (x,y) \in [0,1]^2\big \vert  xy^{0.7} < \tau \land y>c\big\rbrace
$$


The target controls the difficulty, obviously if the target is decreased then the condition to solve a block is more difficult to satisfy.
### More insight on the log scale
If we apply the logarithmic transformation on the acceptance region $A_{\tau}$, the condition

$$ xy^{0.7} <\tau\land y>c $$
can be reformulated as
$$ -\log(x)+0.7(-\log(y)) >-\log(\tau)\land -\log(y)<-\log(c) $$

Recall that $x$ and $y$ are less than 1. This means $-\log(x)$ and $-\log(y)$ are positive. We can therefore visualize the acceptance region $A_{\tau}$ in the first [quadrant](https://en.wikipedia.org/wiki/Quadrant_(plane_geometry)) of a [Cartesian coordinate system](https://en.wikipedia.org/wiki/Cartesian_coordinate_system) representing $-\log(x)$ and $-\log(y)$ along its axes.

The following figure depicts the situation in log scale, the acceptance region $A_{\tau}$ is colored light blue:
<p align="center">
  <img src="./img/acceptance_region.svg" alt="Acceptance Region"/>
</p>

## Probability Theory
### Distribution of hashes
A proper hash function should be random in the sense that each output bit cannot be predicted from the input and also cannot be predicted from other bits in its output. Therefore with the interpretation of a hash as a number in $[0,1)$ we can model the Verushash v2.1 $X(h)$ and the Sha256t $Y(h)$ of a header as samples of [uniform](https://en.wikipedia.org/wiki/Continuous_uniform_distribution) [random variable](https://en.wikipedia.org/wiki/Random_variable) on $[0,1]$. Since we use two different hash functions we can model the vector $(X(h),Y(h))$ as two [independent](https://en.wikipedia.org/wiki/Independence_(probability_{\tau}heory)) realization of a uniform random variable on $[0,1]$. This means that the [joint distribution](https://en.wikipedia.org/wiki/Joint_probability_distribution) is the [product measure](https://en.wikipedia.org/wiki/Product_measure), i.e. the uniform distribution on $[0,1]^2$.

We therefore define the [random vector](https://en.wikipedia.org/wiki/Multivariate_random_variable) $`(X,Y)`$ to have this uniform probability distribution on $`[0,1]`$. Keep in mind that this random vector just models the Verushash v2.1 and Sha256t hashes (interpreted as numbers in $`[0,1]`$) of a block header in a probability-theoretic setting. 

### Pushforward measure on log scale.
On the log scale we consider the transformed vector $(-\log(X),-\log(Y))$. The probability distribution of this transformed vector is the [pushforward measure](https://en.wikipedia.org/wiki/Pushforward_measure) of $(X,Y)$ through the map $g: [0,1]^2 \to \mathbb{R}_{\ge0}^2, (x,y)\mapsto(-\log(x),-\log(y))$. Note that by independence of $X$ and $Y$

$$
\forall x,y\ge 0:\quad \mathbb{P}\big[-\log(X)\le x\land -\log(Y)\le y\big]=
\mathbb{P}\big[X\le e^{-x}\land Y\le e^{-y}\big]=e^{-(x+y)}.
$$

Since the [Borel $`\sigma`$-algebra](https://en.wikipedia.org/wiki/Borel_set) on $`\mathbb{R}_{\ge 0}^2`$ is generated by the sets $`[0,x]\times [0,y], x,y\in\mathbb{R}_{\ge0}`$ this proves that the pushforward measure is the product measure of two identical [exponential distributions](https://en.wikipedia.org/wiki/Exponential_distribution).

With this info we can do probability-theoretic calculations on the log scale.

### Probability to mine a block using filtered mining
Recall that a block is rejected if the Sha256t hash of its header is too small, i.e. if $Y< c$. Furthermore, the smaller $Y$ the easier it is to satisfy the second condition $XY^{0.7}< \tau$ because larger, and therefore easier-to-mine Verushash v2.1 hashes are accepted.

Now consider a specific mining setting. We denote the Verushash v2.1 hashrate by $\mathfrak{h}_X$ and the Sha256t hashrate by $\mathfrak{h}_Y$. For simplicity we will call $\mathfrak{h}_X$ the *CPU hashrate* and $\mathfrak{h}_Y$ the *GPU hashrate* because these are the devices that the respective algorithms are typically mined on at the moment. We will denote the $\frac{\mathfrak{h}_Y}{\mathfrak{h}_X}$ by $a$ and since GPU hashrate is usually greater than CPU hashrate $a$ will be greater than 1. We call this number the *hashrate ratio*.

To match CPU hashrate, hashes computed on GPU must be filtered, and from the discussion above a reasonable filter condition is to compute Verushash v2.1 on headers $h$ that satisfy

$$ c< Y(h) < c+1/a$$

This way we would select fraction 1/a of GPU hashes to check Verushash v2.1 on the corresponding headers. The fraction $1/a$ of GPU hashrate will exactly match the CPU hashrate such that filtered GPU results will come at the right rate to be processed by CPU. 

Note that this is only true for $a > (1-c)^{-1}$, for the small range between $1$ and $(1-c)^{-1}$ (which is only a tiny bit larger than 1) the above reasoning would need to treat the case where filtering cannot avoid that some GPU hashes are rejected for $Y$ being smaller than $c$ if we want to match CPU rate. In this case CPU and GPU hash rates are just too close to allow enough filtering. But we ignore this small range for now as usually GPU hash rate on Sha256t is orders of magnitude larger than CPU hashrate on Verushash v2.1.

For some number $d \in [c,1]$ it holds that

$$
\begin{align*} 
\mathbb{P}\big[(X,Y)\in A_{\tau}\land Y\in[c,d]\big]&=\int_{-\log(d)}^{-\log(c)}e^{-y}\int_{-\log(\tau)-0.7y}^{\infty}e^{-x}\textnormal{d}x\textnormal{d}y\\
&=\int_{-\log(d)}^{-\log(c)}e^{-(1-0.7)y}\tau\textnormal{d}y\\
&=
\tfrac{1}{1-0.7}\big(d^{1-0.7}-c^{1-0.7}\big)\tau\\
&=\tfrac{10}{3}\tau(d^{0.3}-c^{0.3}) \\
\mathbb{P}\big[Y\in[c,d]\big]&=\int_{\log(d)}^{-\log(c)}e^{-y}\textnormal{d}y=(d-c)\\
\mathbb{P}\big[(X,Y)\in A_{\tau}\vert Y\in[c,d]\big]&=\frac{\mathbb{P}\big[(X,Y)\in A_{\tau}\land Y\in[c,d]\big]}{\mathbb{P}\big[Y\in[c,d]\big]}\\
&=\tfrac{10}{3}\tau\frac{(d^{0.3}-c^{0.3})}{(d-c)}
\end{align*}
$$

We denote the conditional probability to mine a block for  $Y(h)$ filtered to be in the interval $[c, c+1/a]$ by $p_{\tau}(a)$. If we plug in $d=c+1/a$ above, we observe that

$$
p_{\tau}(a) = \mathbb{P}\big[(X,Y)\in A_{\tau}\vert Y\in[c,c+1/a]\big] =\tfrac{10}{3} a\big((c+1/a)^{0.3}-c^{0.3}\big)\tau
$$

### Hashrate Ratio Boost
We observe that the probability to mine a block given that its Sha256t is filtered to be in the interval $[c,c+1/a]$ is influenced via the term $a\big((c+1/a)^{0.3}-c^{0.3}\big)$. 

To express the effect of hashrate ratio $a$ compared to a hashrate ratio 1 on the filtered mining probability on $p_{\tau}$ we consider the quotient

$$
\gamma(a) := p_{\tau}(a)/p_{\tau}(1) = a\big((c+1/a)^{0.3}-c^{0.3}\big)/((c+1)^{0.3}-c^{0.3})
$$

for $a\ge 1$ (again mind the small range $[1,(1-c)^{-1}$] where the reasoning is not correct). Note that this does not depend on the target $\tau$ anymore. For every target $\tau$ probability to mine a block is multiplied by $\gamma(a)$ when the between GPU hashrate and CPU hashrate is $a$ compared to a quotient of $1$. We thereforecall $\gamma(a)$ the *hashrate ratio boost* for hashrate ratio $a$. The function $\gamma$ looks like this:
<p align="center">
  <img src="./img/miningratio_boost.png" alt="Acceptance Region"/>
</p>

<details>
  <summary> Julia code to plot this function</summary>

```julia
c = 0.005
beta = 0.7
f(a) = a*((c+1/a)^(1-beta)-c^(1-beta))
g(x)=f(x)/f(1)
using Plots
p = plot(g, xlim=[1,200], label = "\$\\gamma\$")
```
</details>

There is a limit on the hashrate ratio boost:

$$
\begin{align*} 
\lim_{a\to\infty} \gamma(a) &= \lim_{a\to\infty}a\big((c+1/a)^{0.3}-c^{0.3}\big)/((c+1)^{0.3}-c^{0.3})\\
&=\lim_{a\searrow 0}\frac{((c+a)^{0.3}-c^{0.3}\big)}{a}/((c+1)^{0.3}-c^{0.3})\\
&=\lim_{a\searrow 0}0.3(c+a)^{-0.7}/((c+1)^{0.3}-c^{0.3})\\
&=0.3c^{-0.7}/((c+1)^{0.3}-c^{0.3})\\
&\approx 15.35,
\end{align*}
$$

where we used [L'Hôpital's rule](https://en.wikipedia.org/wiki/L%27H%C3%B4pital%27s_rule) in the third step and finally plugged in the constant $c = 0.005$. This means that hashrate ratio boost cannot go above $\approx 15.35$ no matter how much Sha256t hashrate is thrown into the game. The higher the hashrate ratio of GPU/CPU hashrates, the more CPU, i.e. Verushash v2.1 hashrate becomes the bottleneck. 

This is intended and protects Warthog against ASICs applied to Sha256t. Furthermore, at the moment while there does not yet exist an optimized miner yet, it protects against exploitation of the algorithm by closed source miners that reach higher Sha256t hashrate. Such mining behavior will suffer heavily from being bottlenecked by CPU hashrate.

### Janusscore - a formula to determine mining efficiency
We define the *Janusscore* $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ for Verushash v2.1 hashrate $\mathfrak{h}_X$ and Sha256t hashrate $\mathfrak{h}_Y$ by

$$
S(\mathfrak{h}_X,\mathfrak{h}_Y)= \tfrac{10}{3}\big((c+\tfrac{\mathfrak{h}_X}{\mathfrak{h}_Y})^{0.3}-c^{0.3}\big)\mathfrak{h}_Y.
$$

With this definition we can express the expected number mined blocks with traget $\tau$ in time $t$ as
$$p_{\tau}(\tfrac{\mathfrak{h}_Y}{\mathfrak{h}_X})\mathfrak{h}_Xt = \tfrac{10}{3}\tfrac{\mathfrak{h}_Y}{\mathfrak{h}_X}\big((c+\tfrac{\mathfrak{h}_X}{\mathfrak{h}_Y})^{0.3}-c^{0.3}\big)\tau t = S(\mathfrak{h}_X,\mathfrak{h}_Y)\tau t.$$
This means two things:
* For every target $\tau$ the expected number of mined blocks in time $t$ is proportional to $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ and therefore $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ describes the mining efficiency.
* $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ takes the role of a hashrate. For example one can estimate the Janusscore $S$ by dividing the number of mined blocks by the time and the target (this can be used in pools to estimate the Janusscore based on number of shares, time and difficulty).


The Janusscore the unit "hashes per second" and can be interpreted as a hashrate equivalent to compare different setups.

Increasing one of the hashrates of $\mathfrak{h}_X$, $\mathfrak{h}_Y$ while leaving the other constant will always increase the Janusscore.

<details>
  <summary> Python code to compute the Janusscore</summary>

```python
# define Janusscore function
c = 0.005
S = lambda hx, hy: hy * 10 * ((c + hx/hy)**0.3 - c**0.3)/3

# example usage with 10 mh/s Verushash hashrate and 250 mh/s Sha256t hashrate
S(10000000, 250000000)
```
</details>

<details>
  <summary> Julia code to compute the Janusscore</summary>

```julia
# define Janusscore function
c = 0.005
S(hx, hy) = hy * 10 * ((c + hx/hy)^0.3 - c^0.3)/3

# example usage with 10 mh/s Verushash hashrate and 250 mh/s Sha256t hashrate
S(10000000, 250000000)
```
</details>

### Estimating Hashrate Ratio from mined blocks

The conditional density $p_{Y,a}$ of $Y$ given $(X,Y)\in A_{\tau}$ and $Y\in [c,c+\tfrac{1}{a}]$ is proportional to

$$
\begin{align*} 
p_{Y,a}(y)&= \frac{e^{-y}\int_{-\log(\tau)-0.7y}^{\infty}e^{-x}\textnormal{d}x}{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}\int_{-\log(\tau)-0.7y}^{\infty}e^{-x}\textnormal{d}x\textnormal{d}y}\\
&= \frac{\tau e^{-0.3y}}{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}\tau e^{-0.3y}\textnormal{d}y}\\
&= \frac{e^{-0.3y}}{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}e^{-0.3y}\textnormal{d}y}\\
&= 0.3\frac{e^{-0.3y}}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
\end{align*}
$$

for $y\in[c, c+\frac{1}{a}]$. Note that again this does not depend on the target $\tau$. The conditional expectation is

$$
\begin{align*} 
\mathbb{E}\big[Y| (X,Y)\in A_{\tau} \land Y\in [c, c+\tfrac{1}{a}]\big]
&= \frac{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}0.3e^{-y}e^{-0.3y}\textnormal{d}y}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
&= \tfrac{0.3}{1.3}\frac{(c+\frac{1}{a})^{1.3}-c^{1.3}}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
\end{align*}
$$

If we have $N$ blocks mined from a specific address we can consider their Sha256t hashes $y_1,\ldots,y_N$ to compute the empirical expectation (mean) $`\bar{y}=N^{-1}\sum\limits_{i=1}^{N} y_i`$.  

The [method of moments](https://en.wikipedia.org/wiki/Method_of_moments_(statistics)) can be used to get an estimate $\hat a$ of the hashrate ratio $a$ from observed Sha256t average $\bar{y}$. To do this we must find $a$ such that the empirical expectation observed from the blocks and the conditional expectation match:

$$
\begin{align*} 
\text{Find $a\in[1,\infty]$ such that}&\quad\bar{y} = \tfrac{0.3}{1.3}\frac{(c+\frac{1}{a})^{1.3}-c^{1.3}}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
\end{align*}
$$

Unfortunately this can only be solved numerically, we cannot express the solution analytically. Since the hashrate ratio $a$ is in the range $[1,\infty]$, the maximal conditional expectation is attained for $a=1$, if we observe an empirical mean $\bar y$ larger than this number

$$
\tfrac{0.3}{1.3}\frac{(c+\frac{1}{1})^{1.3}-c^{1.3}}{(c+\frac{1}{1})^{0.3}-c^{0.3}}\approx 0.269
$$

we just estimate $\hat a=1$, otherwise we solve the above equation numerically. In the following code we cap estimation at factor 100000:

<details>
  <summary> Python code to estimate hashrate ratio</summary>

```python
from math import exp
from scipy.optimize import fsolve

# example usage
def get_miningratio(sha256t_list):
    """Function to determine the hashrate ratio
    from a list of observed sha256t values

    :sha256t_list: list of numbers in [0,1] corresponding to sha256t hashes
    :returns: estimate of hashrate ratio

    """
    y_avg=sum(sha256t_list)/len(sha256t_list)
    c = 0.005
    p = lambda a: 0.3/1.3*((c+a)**1.3-c**1.3)/((c+a)**0.3-c**0.3)
    threshold = p(1/100000)
    if y_avg<threshold:
        return 100000
    elif y_avg >p(1):
        return 1
    f = lambda a: p(a)-y_avg
    return 1/fsolve(f, [threshold, 1])[0]


# example usage
sha256t_list=[0.025,0.014,0.032]
get_miningratio(sha256t_list)
```
</details>

[^1]: *CoinFuMasterShifu* (2023). **[Proof of Balanced Work: The Theory of Mining Hash Products](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf)**
