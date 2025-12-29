---
share: "true"
---

# CKKS Rescale \- mod\_down 
## Motivation
Divide both message and ciphertext's noise  by scale. Required after HE multiplication (of two ciphertexts or ciphertext and plaintext). Multiplication of $m\_1$ and $m\_2$ the error increases by $\Delta\cdot m\_1e\_2+\Delta\cdot m\_2e\_1$. by rescaling we reduce this error to $m\_1e\_2+m\_2e\_1$. This error is small enough and is much smaller than the plaintext $\Delta m\_1m\_2$. Before rescale the plaintext is $\Delta^2m\_1m\_2$.

Rescale is computed by dividing ciphertext polynomial's by the scheme scale $\Delta$ followed by rounding and dividing the modulo by a prime divider $q$ (with size close to $\Delta$):

$$\begin{align}
ct &= (ct\_0,ct\_1)\bmod Q\\
ct\_{rescaled}&= (\lceil ct\_0/q \rfloor , \lceil ct\_1/q\rfloor) \bmod Q/q
\end{align}$$

Note: this is how it was implemented in the [original CKKS scheme](@original%20ckks%20paper%20-%20Homomorphic%20Encryption%20for%20Arithmetic%20of%20Approximate%20Numbers.md) 

## Correctness
let $ct=(c\_0,c\_1)\bmod Q$ i.e., $c\_0\-c\_1\cdot s\approx m+Q I$ . Dividing both sides of the equation by $q$ results in $\lceil c\_0/q\rfloor \- \lceil c\_1/q\rfloor \cdot s\approx m/q+(Q/q)I$ which is equivalent to a ciphertext $c^\prime=\lceil(c\_0/q,c\_1/q)\rfloor \bmod Q/q$ that encrypts $m/q$.



## Rescaling ciphertext with more than 2 polynomials
Similar to the correctness proof above:
let $ct=(d\_0,d\_1,d\_2)\bmod Q$ be a non\-linear ciphertext that encrypts message $m$ i.e.,
$\langle ct, (1,\-s,\-s^2)\rangle = d\_0\-d\_1s\-d\_2s^2= m +QI$. 
Dividing the right equation by $q$ and round, we get: $\lceil d\_0/q \rfloor \- \lceil d\_1/q \rfloor\cdot s\-\lceil d\_2/q \rfloor\cdot s^2 = m/q+(Q/q)I$ 
i.e., $ct^\prime:=(\lceil d\_0/q \rfloor , \lceil d\_1/q \rfloor,\lceil d\_2/q \rfloor)\bmod Q/q$ is a non\-linear ciphertext that encrypts message $m/q$.

## In RNS CKKS scheme (CRT) [^2]
The modulo of level $\ell$ is a prime $q\_\ell$ with size close to the size of the scheme scale parameter.
Let $ct=([ct]\_{q\_j})\_{0\le j\le L}$. The rescale is computed: $ct^\prime=\text{RS}\_{\ell\to \ell\-1}(ct)$ where $$ct\_i^{(j)}=[q\_\ell]\_j^{\-1}([ct\_i]\_{q\_j}\-[ct\_i]\_{q\_\ell})$$ for $i=0,1$ and $0\le j\le\ell\-1$.
Subtraction of $[ct\_i]\_{q\_\ell}$ from each component \- rounds the coefficients to s nearest integer that is dividable by $q\_\ell$. Then, multiplying the j\-th CRT  component with $[q\_\ell]\_j^{\-1}$ (the inverse of $q\_\ell$ in $\mathbb{Z}\_{q\_j}$) will be CRT\-composed to the integer $\lfloor q^{\-1}\_\ell\cdot c\_i\rceil$ which is exactly the result of rescale.

The idea is that $c\_i\-[c\_i]\_{q\_\ell}$ is an integer that when divided by $q\_\ell$ the result is an integer. 
This is correct as $[c\_i]\_{q\_\ell}$ is the reminder of $c\_i$ divided by $q\_\ell$ so removing the reminder leave us with a number that is divisible by $q\_\ell$.

Here is the algorithm from [^2]
![Pasted image 20250930155650.png](./images/Pasted%20image%2020250930155650.png)
![Pasted image 20250605184024.png](./images/Pasted%20image%2020250605184024.png)


## Why do we need to divide the modulo?
See correction explanation above. Below is a counter example.
Just dividing without changing the modulo will not work as in the following example.

Assume all polynomials are of degree 0 i.e., have only constants:
$a=10, s=4, e=0, q=5\times 7$. The RLWE term is $(a\cdot s +e,a)\bmod q=(5,10)\bmod 35$ .
Dividing by $5$ we get the term $(1,2)\bmod 35$ . if we try to decrypt it, we get $1\-2\cdot 4\bmod 35=\-7\bmod 35=28\neq 0$ . But, if also divide the modulo we get $1\-2\cdot 4\bmod 7=\-7\bmod 7=0$.

Dividing of the modulo works because $(a+pqn\_1)s=(as+pqn\_2)$ and so $(a/p+qn\_1)(s+0\cdot pq) =(as/p)+qn\_2$ which is equivalent to $[(a/p)\bmod q] \cdot [s \bmod q] = (a/p)s\bmod q$ .

## Rescale in original CKKS
In the [original CKKS scheme](@original%20ckks%20paper%20-%20Homomorphic%20Encryption%20for%20Arithmetic%20of%20Approximate%20Numbers.md) , the modulo of the $\ell$ level $q\_\ell:=p^\ell q\_0$ where $p$ are integers and $0\le \ell \le L$ for some $L$. $$\text{RS}\_{\ell\to\ell^\prime}(ct):=\bigg\lfloor ct\frac{q\_{\ell\-1}}{q\_\ell} \bigg\rceil \bmod p\_{\ell\-1}$$
The rescaled ciphertext decrypts to the plaintext $m\frac{q\_{\ell\-1}}{q\_\ell}$ plus some small noise where $\text{Decrypt}(ct)\approx m$
Correctness: let $ct=(c\_0,c\_1)\bmod q\_\ell$ i.e., $c\_0\-c\_1\cdot s\approx m+q\_\ell I$ . Dividing both $c\_0$ and $c\_1$ by $q=q\_\ell/q\_{\ell\-1}$ leads to the rescaled ciphertext $c^\prime=\lceil(c\_0/q,c\_/q)\rfloor \bmod q\_\ell/q$  s.t., $c^\prime\_0\-c^\prime\_1\cdot s\approx m/q + q\ell/q I$ .


Related: [Rescale of integer](./Rescale%20of%20integer.md)

## Created 2024\-05\-01 11:15

[^1]: https://eprint.iacr.org/2019/688.pdf, page 4.
[^2]: [RNS ckks paper](https://eprint.iacr.org/2018/931.pdf), page 9