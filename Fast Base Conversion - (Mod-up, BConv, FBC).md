---
share: "true"
---


# Fast Basis Conversion \- (Mod\-up, BConv, FBC)
A method for converting RNS representation from one base to another base which are co\-prime. This is useful for [Mod-up](Mod-up.md) and [Mod\-down](CKKS%20Rescale%20-%20mod_down.md) algorithms. 
let $x\in\mathbb{Z}\_q$ which is represented in CRT by $[x]\_{q\_i}$ . we would like to compute $[x]\_{p\_i}$. note that $x\in\mathbb{Z}\_q$ implies it is smaller than $q$. also note that the integer reconstruct by [CRT](./Chinese%20Reminder%20Theorem%20(CRT).md) can be larger than $q$ and so it is required to compute modulo $q$ after the CRT reconstruction of an integer. This is important! $$x= \bigg[\sum\_{i=0}^\ell [x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}\cdot \hat{q\_i}\bigg]\_q$$ Converting the equation to be over the integers, we get: 
$$(1)\ \ \ \ \ \ \ x=\sum\_i [x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}\cdot \hat{q\_i} \-v\cdot q$$ for some integer $v$. We can approximate $v$ using the fact that $x/q<1$ for $|x|<q$ and we get $\lfloor x/q\rfloor=0$.
Divide (1) by $q$ (as real number and not as inverse in $\mathbb{Z}\_q$), floor and add $v$ to both sides of the equation, we get $$v = \Bigg\lfloor\bigg( \sum\_i [x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}\cdot \hat{q\_i}\bigg)/q \Bigg\rfloor=\bigg\lfloor \sum\_i \frac{[x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}}{q\_i}\bigg \rfloor$$
Now, we ready to compute $[x]\_{p\_i}$,
$$
\begin{align}
[x]\_{p\_j}&=\bigg[\sum\_{i=0}^{\ell} [x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}\cdot [\hat{q\_i}]\_{p\_j}\-[v]\_{p\_j}\cdot[q]\_{p\_j}\bigg]\_{p\_j}\\
&=
\sum\_i\bigg[\big[ [x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i} \big]\_{q\_i}\cdot[\hat{q\_i}]\_{p\_j}\bigg]\_{p\_j} \- \Bigg[\bigg\lfloor\sum\_i \frac{\big[[x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}\big]\_{q\_i}}{q\_i}\bigg\rfloor\Bigg]\_{p\_j}\cdot [q]\_{p\_j}

\end{align}
$$

This computation is due Shai et.al[^1].
Note: $v$ can be computed once and reuse when computing $[x]\_{p\_i}$ for each $p\_i$.

## RNS\-HEAAN Fast base conversion formula
HEAAN\-RNS [^3] scheme requires all primes $q\_i,p\_i$ to be close to some prime $q$. More recent implementation of CKKS do not hold this requirements and the values of $q\_i$ and $p\_i$ are independent and can have large differences.

The above requirement of HEAAN\-RNS enable a simpler computation (without computing $v$) than the above and allow inaccuracy in the computation of FBC.
$$
[x]\_{p\_j}=\sum\_{i=0}^{\ell}\bigg( \big[[x]\_{q\_i}\cdot [\hat{q\_i}^{\-1}]\_{q\_i}\big]\_{q\_i}\cdot [\hat{q\_i}]\_{p\_j} \bmod {p\_j}\bigg) \bmod p\_j
$$
This computation is also mentioned in [^2] and I've implemented it here[^5]
The FBC algorithm was originally introduced in [^4].
## Created 2024\-07\-07 15:54

[^1]: [@An Improved RNS Variant of the BFV Homomorphic Encryption Scheme](./@An%20Improved%20RNS%20Variant%20of%20the%20BFV%20Homomorphic%20Encryption%20Scheme.md), section 2.2.
[^2]: [Better Bootstrapping for Approximate Homomorphic Encryption](https://eprint.iacr.org/2019/688.pdf) 
[^3]: [A Full RNS Variant of Approximate Homomorphic Encryption](https://eprint.iacr.org/2018/931.pdf) , page 6
[^4]: [A Full RNS Variant of FV like Somewhat Homomorphic Encryption Schemes](https://eprint.iacr.org/2016/510.pdf), page 5
[^5]: CKKS\-RNS/tools/exp\_fast\_base\_conversion.py in FHE\_RESEARCH repository