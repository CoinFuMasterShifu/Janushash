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

For target $t\in[0,1]$ we define the following three conditions a header $h$ must satisfy to have a valid Proof of Balanced Work:

1. The Janushash must be below the threshold $J(h) < t$.
2. The Janushash must not be too small $J(h) \ge t/10$.
3. The Sha256t must not be too small $Y(h) \ge c$ for some constant $c=e^{-6.5}\approx 0.001503439$

An equivalent formulation is to require that $(X(h),Y(h))$ must be an element of the *acceptance region* $A_t\subset [0,1]^2$ defined as

$$
A_t := \big\lbrace (x,y) \in [0,1]^2\big \vert t/10 \le xy^{0.7} < t \land y>c\big\rbrace
$$


The target controls the difficulty, obviously if the target is decreased then the condition to solve a block is more difficult to satisfy.
### More insight on the log scale
If we apply the logarithmic transformation on the acceptance region $A_t$, the condition

$$ t/10\le xy^{0.7} <t\land y>c $$
can be reformulated as
$$ -\log(t/10)\ge -\log(x)+0.7(-\log(y)) >-\log(t)\land -\log(y)<-\log(c) $$

Recall that $x$ and $y$ are less than 1. This means $-\log(x)$ and $-\log(y)$ are positive. We can therefore visualize the acceptance region $A_t$ in the first [quadrant](https://en.wikipedia.org/wiki/Quadrant_(plane_geometry)) of a [Cartesian coordinate system](https://en.wikipedia.org/wiki/Cartesian_coordinate_system) representing $-\log(x)$ and $-\log(y)$ along its axes.

The following figure depicts the situation in log scale, the acceptance region $A_t$ is colored light blue:
<p align="center">
  <img src="./img/acceptance_region2.svg" alt="Acceptance Region"/>
</p>

## Probability Theory
### Distribution of hashes
A proper hash function should be random in the sense that each output bit cannot be predicted from the input and also cannot be predicted from other bits in its output. Therefore with the interpretation of a hash as a number in $[0,1)$ we can model the Verushash v2.1 $X(h)$ and the Sha256t $Y(h)$ of a header as samples of [uniform](https://en.wikipedia.org/wiki/Continuous_uniform_distribution) [random variable](https://en.wikipedia.org/wiki/Random_variable) on $[0,1]$. Since we use two different hash functions we can model the vector $(X(h),Y(h))$ as two [independent](https://en.wikipedia.org/wiki/Independence_(probability_theory)) realization of a uniform random variable on $[0,1]$. This means that the [joint distribution](https://en.wikipedia.org/wiki/Joint_probability_distribution) is the [product measure](https://en.wikipedia.org/wiki/Product_measure), i.e. the uniform distribution on $[0,1]^2$.

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
Recall that a block is rejected if the Sha256t hash of its header is too small, i.e. if $Y< c$. Furthermore, the smaller $Y$ the easier it is to satisfy the other two conditions $1/10t \le XY^{0.7}< t$ because larger and therefore easier-to-mine Verushash v2.1 hashes are accepted.

Now consider a specific mining setting. We denote the Verushash v2.1 hashrate by $\mathfrak{h}_X$ and the Sha256t hashrate by $\mathfrak{h}_Y$. For simplicity we will call $\mathfrak{h}_X$ the *CPU hashrate* and $\mathfrak{h}_Y$ the *GPU hashrate* because these are the devices that the respective algorithms are typically mined on at the moment. We will denote the $\frac{\mathfrak{h}_Y}{\mathfrak{h}_X}$ by $a$ and since GPU hashrate is usually greater than CPU hashrate $a$ will be greater than 1. We call this number the *mining ratio*.

To match CPU hashrate, hashes computed on GPU must be filtered, and from the discussion above a reasonable filter condition is to compute Verushash v2.1 on headers $h$ that satisfy

$$ c< Y(h) < c+1/a$$

This way we would select fraction 1/a of GPU hashes to check Verushash v2.1 on the corresponding headers. The fraction $1/a$ of GPU hashrate will exactly match the CPU hashrate such that filtered GPU results will come at the right rate to be processed by CPU. 

Note that this is only true for $a > (1-c)^{-1}$, for the small range between $1$ and $(1-c)^{-1}$ (which is only a tiny bit larger than 1) the above reasoning would need to treat the case where filtering cannot avoid that some GPU hashes are rejected for $Y$ being smaller than $c$ if we want to match CPU rate. In this case CPU and GPU hash rates are just too close to allow enough filtering. But we ignore this small range for now as usually GPU hash rate on Sha256t is orders of magnitude larger than CPU hashrate on Verushash v2.1.

For some number $d \in [c,1]$ it holds that

$$
\begin{align*} 
\mathbb{P}\big[(X,Y)\in A_t\land Y\in[c,d]\big]&=\int_{-\log(d)}^{-\log(c)}e^{-y}\int_{-\log(t)-0.7y}^{-\log(t/10)-0.7y}e^{-x}\textnormal{d}x\textnormal{d}y\\
&=\int_{-\log(d)}^{-\log(c)}e^{-(1-0.7)y}t(1-\tfrac{1}{10})\textnormal{d}y\\
&=
\tfrac{1}{1-0.7}\big(d^{1-0.7}-c^{1-0.7}\big)t(1-\tfrac{1}{10})\\
&=3t(d^{0.3}-c^{0.3}) \\
\mathbb{P}\big[Y\in[c,d]\big]&=\int_{\log(d)}^{-\log(c)}e^{-y}\textnormal{d}y=(d-c)\\
\mathbb{P}\big[(X,Y)\in A_t\vert Y\in[c,d]\big]&=\frac{\mathbb{P}\big[(X,Y)\in A_t\land Y\in[c,d]\big]}{\mathbb{P}\big[Y\in[c,d]\big]}\\
&=3t\frac{(d^{0.3}-c^{0.3})}{(d-c)}
\end{align*}
$$

We denote the conditional probability to mine a block for  $Y(h)$ filtered to be in the interval $[c, c+1/a]$ by $p_t(a)$. If we plug in $d=c+1/a$ above, we see observe that

$$
p_t(a) = \mathbb{P}\big[(X,Y)\in A_t\vert Y\in[c,c+1/a]\big] =3ta\big((c+1/a)^{0.3}-c^{0.3}\big)
$$

### Mining Ratio Boost
We observe that the probability to mine a block given that its Sha256t is filtered to be in the interval $[c,c+1/a]$ is influenced via the term $a\big((c+1/a)^{0.3}-c^{0.3}\big)$. 

To express the effect of mining ratio $a$ compared to a mining ratio 1 on the filtered mining probability on $p_t$ we consider the quotient

$$
\gamma(a) := p_t(a)/p_t(1) = a\big((c+1/a)^{0.3}-c^{0.3}\big)/((c+1)^{0.3}-c^{0.3})
$$

for $a\ge 1$ (again mind the small range $[1,(1-c)^{-1}$] where the reasoning is not correct). Note that this does not depend on the target $t$ anymore. For every target $t$ probability to mine a block is multiplied by $\gamma(a)$ when the between GPU hashrate and CPU hashrate is $a$ compared to a quotient of $1$. We thereforecall $\gamma(a)$ the *mining ratio boost* for mining ratio $a$. The function $\gamma$ looks like this:
<p align="center">
  <img src="./img/miningratio_boost.png" alt="Acceptance Region"/>
</p>

<details>
  <summary> Julia code to plot this function</summary>

```julia
c=exp(-6.5)
beta=0.7
f(a) = a*((c+1/a)^(1-beta)-c^(1-beta))
g(x)=f(x)/f(1)
using Plots
plot(f, xlim=[1,200])
```
</details>

There is a limit on the mining ratio boost:

$$
\begin{align*} 
\lim_{a\to\infty} \gamma(a) &= \lim_{a\to\infty}a\big((c+1/a)^{0.3}-c^{0.3}\big)/((c+1)^{0.3}-c^{0.3})\\
&=\lim_{a\searrow 0}\frac{((c+a)^{0.3}-c^{0.3}\big)}{a}((c+1)^{0.3}-c^{0.3})\\
&=\lim_{a\searrow 0}0.3(c+a)^{-0.7}((c+1)^{0.3}-c^{0.3})\\
&=0.3c^{-0.7}((c+1)^{0.3}-c^{0.3})\\
&\approx 24.36,
\end{align*}
$$

where we used [L'Hôpital's rule](https://en.wikipedia.org/wiki/L%27H%C3%B4pital%27s_rule) in the third step and finally plugged in the constant $c =e^{-6.5}$. This means that mining ratio boost cannot go above $\approx 24.36$ no matter how much Sha256t hashrate is thrown into the game. The higher the mining ratio of GPU/CPU hashrates, the more CPU, i.e. Verushash v2.1 hashrate becomes the bottleneck. 

This is intended and protects Warthog against ASICs applied to Sha256t. Furthermore, at the moment while there does not yet exist an optimized miner yet, it protects against exploitation of the algorithm by closed source miners that reach higher Sha256t hashrate. Such mining behavior will suffer heavily from being bottlenecked by CPU hashrate.

### Janusscore - a formula to determine mining efficiency
We can define the  *Janusscore*, which is the mining efficiency of an arbitrary combination of a Sha256t hashrate and a smaller Verushash v2.1 hashrate with respect to the baseline of 1 hash per second for both Sha256t and Verushash v2.1.

Regarding mining efficiency we observe the following:
- The expected yield is proportional to the probability to mine a block. Therefore a setup with mining ratio $a$ is expected to generate $\gamma(a)$ times the yield of a set up with mining ratio $1$. This determines the dependency on the mining ratio $a$.
- For each fixed mining ratio the expected yield is proportional to the number of computed Janushashes, which is equal to the number of Verushash v2.1 hashes. This determines the dependency on the CPU hashrate.

We can therefore define the following *Janusscore* $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ of a mining setup with Verushash v2.1 hashrate $\mathfrak{h}_X$ and Sha256t hashrate $\mathfrak{h}_Y$:

$$
S(\mathfrak{h}_X,\mathfrak{h}_Y)= \gamma(\tfrac{\mathfrak{h}_Y}{\mathfrak{h}_X})\mathfrak{h}_X= \mathfrak{h}_Y\frac{(c+\tfrac{\mathfrak{h}_X}{\mathfrak{h}_Y})^{0.3}-c^{0.3}}{(c+1)^{0.3}-c^{0.3}}
$$

This quantity has the unit "hashes per second" and indeed the Janusscore can be interpreted as a hashrate equivalent that can be used to compare different setups.

Increasing one of the hashrates of $\mathfrak{h}_X$, $\mathfrak{h}_Y$ while leaving the other constant will always increase the Janusscore.

### Estimating Mining Ratio from mined blocks

The conditional density $p_{Y,a}$ of $Y$ given $(X,Y)\in A_t$ and $Y\in [c,c+\tfrac{1}{a}]$ is proportional to

$$
\begin{align*} 
p_{Y,a}(y)&= \frac{e^{-y}\int_{-\log(t)-0.7y}^{-\log(t/10)-0.7y}e^{-x}\textnormal{d}x}{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}\int_{-\log(t)-0.7y}^{-\log(t/10)-0.7y}e^{-x}\textnormal{d}x\textnormal{d}y}\\
&= \frac{\frac{9}{10}te^{-0.3y}}{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}\frac{9}{10}te^{-0.3y}\textnormal{d}y}\\
&= \frac{e^{-0.3y}}{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}e^{-0.3y}\textnormal{d}y}\\
&= 0.3\frac{e^{-0.3y}}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
\end{align*}
$$

for $y\in[c, c+\frac{1}{a}]$. Note that again this does not depend on the target $t$. The conditional expectation is

$$
\begin{align*} 
\mathbb{E}\big[Y| (X,Y)\in A_t \land Y\in [c, c+\tfrac{1}{a}]\big]
&= \frac{\int_{-\log(c+\frac{1}{a})}^{-\log(c)}0.3e^{-y}e^{-0.3y}\textnormal{d}y}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
&= \tfrac{0.3}{1.3}\frac{(c+\frac{1}{a})^{1.3}-c^{1.3}}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
\end{align*}
$$

If we have $N$ blocks mined from a specific address we can consider their Sha256t hashes $y_1,\ldots,y_N$ to compute the empirical expectation (mean) $\bar{y}=N^{-1}\sum_{i=1}^N y_i$.  

The [method of moments](https://en.wikipedia.org/wiki/Method_of_moments_(statistics)) can be used to get an estimate $\hat a$ of the mining ratio $a$ from observed Sha256t average $\bar{y}$. To do this we must find $a$ such that the empirical expectation observed from the blocks and the conditional expectation match:

$$
\begin{align*} 
\text{Find $a\in[1,\infty]$ such that}&\quad\bar{y} = \tfrac{0.3}{1.3}\frac{(c+\frac{1}{a})^{1.3}-c^{1.3}}{(c+\frac{1}{a})^{0.3}-c^{0.3}}\\
\end{align*}
$$

Unfortunately this can only be solved numerically, we cannot express the solution analytically. Since the mining ratio $a$ is in the range $[1,\infty]$, the maximal conditional expectation is attained for $a=1$, if we observe an empirical mean $\bar y$ larger than this number

$$
\tfrac{0.3}{1.3}\frac{(c+\frac{1}{1})^{1.3}-c^{1.3}}{(c+\frac{1}{1})^{0.3}-c^{0.3}}\approx 0.269
$$

we just estimate $\hat a=1$, otherwise we solve the above equation numerically. In the following code we cap estimation at factor 100000:

<details>
  <summary> Python code to estimate mining ratio</summary>

```python
from math import exp
from scipy.optimize import fsolve

# example usage
def get_miningratio(sha256t_list):
    """Function to determine the mining ratio
    from a list of observed sha256t values

    :sha256t_list: list of sha256t hashes
    :returns: estimate of mining ratio

    """
    y_avg=sum(sha256t_list)/len(sha256t_list)
    c = exp(-6.5)
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



## UNFINISHED STUFF
# Pools

Conventional Proof of Work pools provide a service to allow miners to collaboratively find a proof of work that is too difficult for them to solve alone. The main benefit is the reduction of variance in mining income and since most people don't like variance they are willing to pay a small fee to reduce income variance (for the same reason insurances exist despite they cost more than they provide on average).

## Nested Tasks and their associated value
Mining pools provide simpler Proof of Work tasks to miners. Miners solve the simpler tasks to prove their contribution to the pool. It is essential that some solutions to the simpler tasks are also solutions to the difficult task that solves a block otherwise the pool tasks sent to miners would not be useful for solving a block. Pools must be able to send tasks at granular difficulty level to match the hashrate of the miner such that the miner does not solve its task too often nor too rarely. This means that their tasks are *nested* in the sense that a more difficult task is also a solution to an easier pool task. We can think of this as a nested chain of tasks where mining the block itself is the most difficult task that is contained in all other tasks (in the sense of mathematical set theory). Furthermore each task must have a value assigned such that the pool can count the contribution of each miner correctly. The value of a task should match the difficulty to allow the pool to correctly estimate each miner's contribution. 

## Acceptance regions for pool tasks

In traditional Proof of Work pools just send out headers with a simpler target. This is a very easy setting because we can easily understand and control the difficulty. If we require one zero less, this halves the difficulty and the value of a solved pool task (i.e. of a share), if we require one additional zero, this doubles the difficulty and the value. Furthermore this kind of tasks are nested because providing a hash starting with a specified number of zeros also is a solution to a task which requires less leading zeros.

However in Janushash things are much more complicated :D
- Firstly we now have two dimensions instead of just one (one for verushash and one for triple sha256)
- Secondly different mining styles (balance of GPU/CPU capabilities)  might behave differently in how difficult pool tasks are solvable. This would be a disaster, if the pool cannot understand how difficult its tasks are it cannot assign a value to estimate contribution value of a share.

Luckily the god of mathematics shines his light on us and makes this problem solvable. In fact we can find a nested set of two-dimensional acceptance regions such that *any* mining style behaves equally, i.e. we can find interesting two-dimensional shapes that have equal probability to be hit by a janushash, no matter how much GPU vs CPU performance is. This allows us to assign a value to them to keep track of miner contribution on pool side without knowing its mining style. Think about it: if such regions did not exist, fair pool mining would not be possible with Janushash. So we are very lucky that they exist!

<p align="center">
  <img src="./img/acceptance_region_pool.svg" alt="Acceptance Region"/>
</p>


$$
\int_0^1
$$

## DEPRECATED SINCE LAST ALGO UPDATE
### Mining difficulty
- The number of block headers that a miner needs to try until it finds a janushash that solves a block is random, more precisely it is [geometrically distributed](https://en.wikipedia.org/wiki/Geometric_distribution) because it is [memoryless](https://en.wikipedia.org/wiki/Memorylessness) (not having found a block for some time does not change the probability to find a block in the future, this is a fundamental property in mining).
- Mining difficulty $d$ is defined as the average number of required of tries, i.e. the [expectation](https://en.wikipedia.org/wiki/Expected_value).
- **Main question**: How exactly does $t$ control the mining difficulty?
- Answer: To be precise this depends on the mining style (how much filtering is used in filtered mining, to be explained later) but in practice with a very good approximation 
    - we can just ignore the dependence in mining style (TODO: prove this but I know it's true :D) and
    - we can just use the normal retargeting mechanism of traditional PoW cryptocurrencies (I have shown this in my paper[^1], Lemma III.1).


### How to find the acceptance region for pool tasks?
NOTE: This section lacks proper explanation, it is just a raw skeleton for now, I will provide detailed explanation soon. 

We define $T(x):=\int_0^x e^{-t}e^{-f(t)-f(x)}\text{d}t$. With this definition we get

$$
\begin{align*} 
F(x) &= \underbrace{\int_0^x e^{-t}e^{-(f(t)-f(x))}\text{d}t}_{T(x)} + e^{-x}=e^{f(x)}\int_0^x e^{-t-f(t)}\text{d}t +e^{-x}\\ 
F'(x) &= f'(x)T(x)+e^{f(x)}e^{-x-f(x)}-e^{-x} =f'(x)T(x)\\ 
\end{align*}
$$

$g(x) = -\frac{1}{\beta}x$ with $\beta=0.7$

Now let's calculate $G$ and $G'$.

$$
\begin{align*} 
G(x) &= \int_0^x e^{-t}e^{-(g(t)-g(x))}\text{d}t + e^{-x}=e^{-\frac{1}{\beta}x}\int_0^x e^{(\frac{1}{\beta}-1)t}\text{d}t +e^{-x}\\ 
&=e^{-\frac{1}{\beta}x}\big[\tfrac{1}{(\frac{1}{\beta}-1)} e^{(\frac{1}{\beta}-1)t}\big]_{t=0}^x-e^{-x} = \tfrac{\beta}{(1-\beta)}\big(e^{-x}-e^{-\frac{1}{\beta}x}\big) + e^{-x}\\
&=\tfrac{1}{(1-\beta)}\big(e^{-x}-\beta e^{-\frac{1}{\beta}x}\big)\\
G'(x) &= \tfrac{1}{(1-\beta)}\big(-e^{-x}+e^{-\frac{1}{\beta}x}\big)
\end{align*}
$$

$$
\begin{align*} 
H(x) &= G(-\beta f(x))\\
H'(x) &= -\beta f'(x)G'(-\beta f(x))\\
\end{align*}
$$

The condition we want for pool mining is $\alpha F(x) = H(x)$ for all $x>=0$ where $\alpha \in (0,1)$ is the fraction of share value of a solved block. It follows that

$$
\begin{align*} 
H(x) &= \alpha F(x) = \alpha (T(x)+e^{-x})\\
H'(x) &= \alpha F'(x) = \alpha f'(x)T(x)\\
\end{align*}
$$

If we solve the first equation above for $T(x)$ and plug this into the second equation we get

$$
-\beta f'(x) G'(-\beta f(x)) = H'(x) = f'(x)(H(x)-\alpha e^{-x}),
$$

This is either satisfied for $f'(x) = 0$ or for

$$
\begin{align*} 
0 &= H(x)-\alpha e^{-x}+\beta G'(-\beta f(x))\\
 &= G(-\beta f(x))+\beta G'(-\beta f(x))-\alpha e^{-x}\\
 &= \tfrac{1}{(1-\beta)}\big(e^{\beta f(x)}-\beta e^{f(x)}\big)
+\beta \tfrac{1}{(1-\beta)}\big(-e^{\beta f(x)}+e^{f(x)}\big)-\alpha e^{-x}\\
 &= e^{\beta f(x)} -\alpha e^{-x}\\
\end{align*}
$$

from which we find $f(x)=-\frac{1}{\beta}x +\frac{\log(\alpha)}{\beta}$. But as we saw before this condition is just required for $f'(x)\not=0$. 
This means that the $f$ is composed of constant sections ($x$ where $f'(x)=0$ and sections where $f(x)=-\frac{1}{\beta}x +\frac{\log(\alpha)}{\beta}$. This gives us an idea how the correct function looks like. The correct function $f$ is

$$
f_{\alpha}(x) = \min\big(-c_{\alpha}, -\frac{1}{\beta}x +\frac{\log(\alpha)}{\beta}\big)\textnormal{ for }x\ge 0
$$

where the positive constant $c_{\alpha}$ is chosen such that 
$\alpha= \tfrac{1}{(1-\beta)}\big(e^{-\beta c_\alpha}-\beta e^{-c_\alpha}\big)$. The reader can easily verify that indeed this function satisfies the equation

$$H(x) = \alpha F(x) \textnormal{ for all }x\ge 0.$$

Values for $c_\alpha$ must be precomputed for fixed $\beta=0.7$ in order evaluate $f_\alpha$ in pools and miners.

### Precomputing values for $c_\alpha$
<details>
  <summary> Julia code to precompute values</summary>

```julia
using Roots
beta = 0.7
for i = 1:100
    p(x) = log(1/(1-beta))+log(exp(-beta*x))+log(1 - beta*exp(-(1-beta)*x))-i*log(1/2)
    root = find_zero(p, (0,400), Bisection())
    println("c_alpha for alpha = 2^(-$i) ≈ $root")
end
```

</details>

<details>
  <summary> Table of precomputed values </summary>

$\alpha$ | $c_{\alpha}$
---------|-------------
$2^{-1}$ | `2.0240078767956975`
$2^{-2}$ | `3.26467020367989`
$2^{-3}$ | `4.394288840992969`
$2^{-4}$ | `5.472731744244513`
$2^{-5}$ | `6.52219327976034`
$2^{-6}$ | `7.553545024545997`
$2^{-7}$ | `8.572922537289955`
$2^{-8}$ | `9.584096317310902`
$2^{-9}$ | `10.589515587487824`
$2^{-10}$ | `11.590832562534855`
$2^{-11}$ | `12.5891916955424`
$2^{-12}$ | `13.58540082663572`
$2^{-13}$ | `14.580038009112693`
$2^{-14}$ | `15.573520991157688`
$2^{-15}$ | `16.566153869289867`
$2^{-16}$ | `17.558159191338017`
$2^{-17}$ | `18.549700466269613`
$2^{-18}$ | `19.540898175512883`
$2^{-19}$ | `20.53184128648575`
$2^{-20}$ | `21.52259560017043`
$2^{-21}$ | `22.513209840865642`
$2^{-22}$ | `23.503720119481823`
$2^{-23}$ | `24.494153216166737`
$2^{-24}$ | `25.484529000862032`
$2^{-25}$ | `26.474862221646834`
$2^{-26}$ | `27.46516382790318`
$2^{-27}$ | `28.455441950358125`
$2^{-28}$ | `29.445702627562884`
$2^{-29}$ | `30.435950344733506`
$2^{-30}$ | `31.426188433594447`
$2^{-31}$ | `32.41641936917795`
$2^{-32}$ | `33.40664499018884`
$2^{-33}$ | `34.39686666264867`
$2^{-34}$ | `35.38708540143535`
$2^{-35}$ | `36.37730196056039`
$2^{-36}$ | `37.36751690023017`
$2^{-37}$ | `38.35773063666429`
$2^{-38}$ | `39.34794347910603`
$2^{-39}$ | `40.338155657318524`
$2^{-40}$ | `41.32836734201312`
$2^{-41}$ | `42.31857866002671`
$2^{-42}$ | `43.30878970559813`
$2^{-43}$ | `44.29900054874621`
$2^{-44}$ | `45.28921124149458`
$2^{-45}$ | `46.279421822496495`
$2^{-46}$ | `47.26963232047122`
$2^{-47}$ | `48.25984275675699`
$2^{-48}$ | `49.25005314720808`
$2^{-49}$ | `50.24026350360414`
$2^{-50}$ | `51.23047383469741`
$2^{-51}$ | `52.2206841469908`
$2^{-52}$ | `53.210894445315915`
$2^{-53}$ | `54.20110473326267`
$2^{-54}$ | `55.19131501349831`
$2^{-55}$ | `56.181525288004615`
$2^{-56}$ | `57.17173555825405`
$2^{-57}$ | `58.16194582534064`
$2^{-58}$ | `59.15215609007725`
$2^{-59}$ | `60.14236635306781`
$2^{-60}$ | `61.132576614761085`
$2^{-61}$ | `62.12278687549047`
$2^{-62}$ | `63.11299713550369`
$2^{-63}$ | `64.1032073949848`
$2^{-64}$ | `65.09341765407056`
$2^{-65}$ | `66.08362791286257`
$2^{-66}$ | `67.07383817143632`
$2^{-67}$ | `68.0640484298479`
$2^{-68}$ | `69.05425868813902`
$2^{-69}$ | `70.0444689463406`
$2^{-70}$ | `71.03467920447567`
$2^{-71}$ | `72.02488946256132`
$2^{-72}$ | `73.01509972061025`
$2^{-73}$ | `74.0053099786319`
$2^{-74}$ | `74.99552023663328`
$2^{-75}$ | `75.9857304946196`
$2^{-76}$ | `76.97594075259474`
$2^{-77}$ | `77.96615101056155`
$2^{-78}$ | `78.95636126852219`
$2^{-79}$ | `79.94657152647824`
$2^{-80}$ | `80.93678178443088`
$2^{-81}$ | `81.92699204238097`
$2^{-82}$ | `82.9172023003292`
$2^{-83}$ | `83.90741255827602`
$2^{-84}$ | `84.89762281622181`
$2^{-85}$ | `85.8878330741668`
$2^{-86}$ | `86.87804333211126`
$2^{-87}$ | `87.86825359005528`
$2^{-88}$ | `88.85846384799896`
$2^{-89}$ | `89.84867410594242`
$2^{-90}$ | `90.83888436388571`
$2^{-91}$ | `91.82909462182886`
$2^{-92}$ | `92.81930487977193`
$2^{-93}$ | `93.8095151377149`
$2^{-94}$ | `94.79972539565784`
$2^{-95}$ | `95.78993565360074`
$2^{-96}$ | `96.7801459115436`
$2^{-97}$ | `97.77035616948643`
$2^{-98}$ | `98.76056642742924`
$2^{-99}$ | `99.75077668537207`
$2^{-100}$ | `100.7409869433149`
</details>




[^1]: *CoinFuMasterShifu* (2023). **[Proof of Balanced Work: The Theory of Mining Hash Products](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf)**
