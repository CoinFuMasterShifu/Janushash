# Details on Janushash 

## Definition

- Each hash (32 byte hex string) can be interpreted as a number in the interval $[0,1)$. 
    - For example the hash `FFFF...FFFF` would correspond to 1-2^256 which is almost 1 and 
    - the hash `0000...0000` would correspond to 0. 
    - **Note**: A hash cannot correspond to exactly 1 but almost 1. 
- For some byte sequence $b$ we denote
    - the verus hash interpreted as a number in $[0,1)$ by $x(b)$ and
    - the triple sha256 hash interpreted as a number in $[0,1)$ by $y(b)$.
- From now on we will omit the dependency on $b$ and will just write $x$ and $y$ instead of $x(b)$ and $y(b)$ respectively.
- We define the number $j\in [0,1)$ by
$$j := x y^{0.7}$$
- $j$ depends implicitly on the hashed byte sequence $b$.
- Similar to above where we converted a hash to a number we can do the reverse, i.e. interpret a number in $[0,1)$ as a hash.
- **Note**: To represent a small number as a hash, one might require more than 32 digits, there exist transcendental numbers which even require infinite digits to be represent exactly. However this is not of interest for us.
- The Janushash of $b$ is the number $j$ interpreted as a hash. 
- We will solely work with the number $j$ and will never explicitly consider the Janushash (hash equivalent of $j$). All theory works with numbers, so implementation only needs to convert hashes to numbers, but not numbers to hashes.
- For convenience from now on we will be sloppy and call $j$ the Janushash, despite $j$ is actually the number representation of the Janushash.


## Condition to solve a block

- We define the condition to solve a block by
$$j < t,$$ 
- Here $j$ is the janushash of the block header, i.e. one needs to compute triple sha256 and verushash of the header to compute $j$, and $t\in[0,1]$ is the target. 
- The target controls the difficulty, obviously if the target is decreased then the condition to solve a block is more difficult to satisfy and vice versa.
### Mining difficulty
- The number of block headers that a miner needs to try until it finds a janushash that solves a block is random, more precisely it is [geometrically distributed](https://en.wikipedia.org/wiki/Geometric_distribution) because it is [memoryless](https://en.wikipedia.org/wiki/Memorylessness) (not having found a block for some time does not change the probability to find a block in the future, this is a fundamental property in mining).
- Mining difficulty $d$ is defined as the average number of required of tries, i.e. the [expectation](https://en.wikipedia.org/wiki/Expected_value).
- **Main question**: How exactly does $t$ control the mining difficulty?
- Answer: To be precise this depends on the mining style (how much filtering is used in filtered mining, to be explained later) but in practice with a very good approximation 
    - we can just ignore the dependence in mining style (TODO: prove this but I know it's true :D) and
    - we can just use the normal retargeting mechanism of traditional PoW cryptocurrencies (I have shown this in my paper[^1], Lemma III.1).


## More insight on the log scale
Recall that the condition to solve a block is $xy^{0.7} < t$. 
It is more convenient to work on the logarithmic scale. So we apply the logarithm on both sides of the equation to reformulate the condition:
$$\log(x) + 0.7 \log(y) < \log(t)$$
Recall that $x$ and $y$ are less than 1. This means $-\log(x)$ and $-\log(y)$ are positive. We can therefore visualize the *acceptance region* (blue area, region where $xy^{0.7}< t$ is satisfied) in the first [quadrant](https://en.wikipedia.org/wiki/Quadrant_(plane_geometry)) of a [Cartesian coordinate system](https://en.wikipedia.org/wiki/Cartesian_coordinate_system) representing $-\log(x)$ and $-\log(y)$ along its axes:

<p align="center">
  <img src="./img/acceptance_region.svg" alt="Acceptance Region"/>
</p>

In the log scale the boundary of the acceptance region is formed by straight lines, and it extends infinitely along the two axes.



[^1]: *CoinFuMasterShifu* (2023). **[Proof of Balanced Work: The Theory of Mining Hash Products](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf)**
