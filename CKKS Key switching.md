\-\-\-
share: "true"
\-\-\-


# CKKS Key switching 
## Motivation (First try)
**First idea,** define  $\text{swk}\_{s\_{in}\rightarrow s\_{out}}:=(a^\* s\_{out}+s\_{in}+e^\*, a^\*) \bmod Q$. This is an encryption of the plaintext $s\_{in}$ w.r.t secret key $s\_{out}$ modulo Q.

Let $ct$ be a [R-LWE ciphertext (RLWE)](R-LWE%20ciphertext%20(RLWE).md) $ct = (ct\_0,ct\_1)=(a\cdot s\_{in}+m+e,a) \bmod Q$ a ciphertext of the plaintext $m$ w.r.t secret key $s\_{in}$ 

The main idea is to subtract the term $a\cdot s\_{in}$  from $a\cdot s\_{in}+m+e$ by computing
$ct\_0 \\ \-a\cdot \text{swk}\_0$ :

$$\begin{array}{r}
a\cdot s\_{in}+m+e \\ 
\-a( a^\*s\_{out}+s\_{in}+e^\*)\\ 
\hline 
\-aa^\*s\_{out} + m+e\-ae^\* 
\end{array}$$

defining a new ciphertext $(ct\_0\-a\cdot swk\_0,\-aa^\*)\bmod Q$  which "encrypts" m w.r.t secret key $s\_{out}$.  note that as $a$ and $a^\*$ are uniformly random than so is their multiplication. 
#### But this will NOT work!
There are two problems:
\- The switching key is not valid as the noise $e^\*$ and the plaintext $s$ have both small norm and so $e\*$ will "hide" the plaintext $s$. 
\- The output $ct^\prime$ is NOT a ciphertext of $m$ w.r.t secret key $s$. The problem here is that the error term $a e^\*$ does not have a small norm and so $m +e^\prime$ may wrap around the modulo $Q$ i.e., this ciphertext is invalid and may not be decryptable.


**Second idea**, choose $P$ s.t. $|P|\approx |Q|$  and divide the noise $ae^\*$ by $P$ but not divide the plaintext $m$
define $\text{swk}\_{s\_{in}\rightarrow s\_{out}}:=(a^\* s\_{out}+P\cdot s\_{in}+e^\*, a^\*) \bmod PQ$ . This corrects the first issue as now the message is $P\cdot s\_{in}$ 
We raise $a \mod Q$ to $a \bmod QP$ (the coefficients of the latter have the same magnitude as the former \- raising the modulo does not change the polynomial), compute $a\cdot \text{swk}\_{s\_{in}\rightarrow s\_{out}}$ , divide by $P^{\-1}$ and drop the modulo back to $Q$: $$P^{\-1}\cdot a\cdot \text{swk}\_0=P^{\-1}aa^\*s\_{out}+as\_{in}+P^{\-1}ae^\* \bmod Q$$

note that the rightmost term has the magnitude of $e^\*$ as $|a|<Q$ and $|P|\approx|Q|$. This solves the second issue.


define $ct\_{s\_{in}\rightarrow s\_{out}}:=ct\_0 \- \lfloor P^{\-1}\cdot a\cdot \text{swk}\_0\rceil\bmod Q$  :

The bx part of $ct\_{s\_{in}\rightarrow s\_{out}}$  is:

$$\begin{array}{r} 
a\cdot s\_{in}+m+e \bmod Q\\
\- \lfloor a\cdot P^{\-1} ( a^\*s\_{out}+P\cdot s\_{in}+e^\*)\rceil\bmod Q\\
\hline 
\-P^{\-1}aa^\*s\_{out} + m+ \underbrace{e\-P^{\-1}ae^\* +e\_{\text{round}}}\_{\text{small error}}\bmod Q 
\end{array}$$ 

The "a" (random) part of $ct\_{s\_{in}\rightarrow s\_{out}}$  is $\-P^{\-1}aa^\*$ 
This ciphertext encrypts the original message $m$ w.r.t secret key $s\_{out}$

> Notes: 
> 1) division by $P$ of an element in ring $PQ$ result in an element with the same value divided by P (with possible rounding) in ring $Q$ e.g.,  $x=[x]\_{15}+15y$  s.t. $[x]\_{15}<15$ so $x/5=[x]\_{15}/5+3y$ s.t. $[x]\_{15}/5<3$ which means that $[x/5]\_{3}= [x]\_{15}/5$   . 
> 2) Computing the modulo reduction to $Q\_\ell$ requires converting the polynomial to coefficient representation, reduce the modulo of the coefficients and back to NTT
> 3) This idea caused a new problem \- on the one hand, the number of levels are limited per security level. On the other,  the encryption of the rotated secret key in the switching key requires to reserve $\log P$ of bits due to switch keys which are modulo $PQ$ i.e, it reduces the levels by factor of 2.

**Third idea**. The problem with the 2nd idea is the half of the levels are taken by special primes and as security limits the number of levels then this reduce the number of effective levels for homomorphic computation. To solve this, the new idea is to reduce the size of $P$ by reducing the norm of $a$ (the uniform random element of a ciphertext $ct$). smaller $a$ means we need smaller $P$ that divides it and make its norm small when reducing the modulo to Q from modulo QP in the computation above.
1. Define switching keys for all $i$ by a set of ciphertexts. This is computed during scheme initialization. $$\text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}}:=(a^\*\_i\cdot  s\_{out}+ \hat{q\_i}\cdot P\cdot s\_{in}+e^\*\_i, a^\*\_i) \bmod  PQ$$ 
2. Decompose $a$ by $$ \big([a\cdot\hat{q\_0}^{\-1}]\_{q\_0},\ldots, [a\cdot\hat{q\_L}^{\-1}]\_{q\_L}\big)$$
   Note by the CRT: $$a=\sum\_{j=0}^L [a]\_{q\_j}\cdot[\hat{q}\_j^{\-1}]\_{q\_j}\cdot\hat{q}\_j=\sum\_{j=0}^L [a\hat{q}\_j^{\-1}]\_{q\_j}\hat{q}\_j \bmod Q = \langle ([a\hat{q}\_j^{\-1}]\_{q\_j})\_j , (\hat{q}\_j)\_j\rangle\bmod Q$$where $\hat{q\_j}:=\prod\_{i\ne j}{q\_i}$ and all components have small norm i.e., $\big|[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\big|< q\_i$ for $i=0,\ldots,L$
 1. Set $P$ to be a prime of size 64 bits i.e., $P>q\_i$ for all $i$
 2. mod\-up of the decomposition $i$ of $a$: $[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}$ to be modulo $P\cdot Q$ 
 3. multiply the i\-th switch key and the $i$\-th decomposition of $a$, we have
$$
\begin{align}
\big[[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\big]\_{PQ}\cdot \text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}} &= 
[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\cdot \text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}} \bmod PQ \\ 
&= \big([a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\cdot a^\*\_i\cdot s\_{out} + [a\cdot \hat{q\_i}^{\-1}]\_{q\_i}\cdot \hat{q\_i}\cdot
P\cdot s\_{in}+ e^\*\_i[a\cdot\hat{q\_i}^{\-1}]\_{q\_i} , [a\cdot\hat{q\_i}^{\-1}]\_{q\_i} a^\*\_i\big)\bmod PQ
\end{align}
$$

5. mod\_down to modulo $P$: 
$$
\bigg(P^{\-1}[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\cdot a^\*\_i\cdot s\_{out} + [a\cdot \hat{q\_i}^{\-1}]\_{q\_i}\cdot \hat{q\_i}\cdot
s\_{in}+ P^{\-1}\cdot e^\*\_i[a\cdot\hat{q\_i}^{\-1}]\_{q\_i} \ ,P^{\-1}\ [a\cdot\hat{q\_i}^{\-1}]\_{q\_i} a^\*\_i\bigg)\bmod Q
$$

6. sum all results: $e\_i^{**}:=P^{\-1}\cdot e^\*\_i[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}$, $a\_i^{**}:=P^{\-1}[a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\cdot a^\*\_i$ 
> $$t = \sum\_{i=0}^L \big([a\cdot\hat{q\_i}^{\-1}]\_{q\_i}\cdot \text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}}\big)= \big(\sum\_{i=0}^L a^{**}\_i\cdot s\_{out}+ a\cdot s\_{in}+ \sum\_{i=0}^L e\_i^{**},\sum\_{i=0}^L a^{**}\_i\big)\bmod Q$$

7. subtract from original ciphertext: $$ct\_0\- t=as\_{in}+m+e\-t=a^{***}s\_{out}+m+e+e^{***}$$ where $a^{***}:= \sum\_{i=0}^L a^{**}\_i$ and $e^{***}:=\sum\_{i=0}^L e\_i^{**}$

**Fourth idea**. The problem with the 3rd idea that you need more memory to store the extra switch keys and more computation is being done for each key switch. To balance this, the new idea is to use batches of $q\_i$'s, $Q\_j:=\prod\_{j\alpha}^{j(\alpha+1)} q\_i$  for decomposing $a$ to fewer smaller polynomials.
3. Decompose $a$ by $$ \big([a\cdot\hat{Q\_0}^{\-1}]\_{Q\_0},\ldots, [a\cdot\hat{Q\_k}^{\-1}]\_{Q\_k}\big)$$ 
 where $\hat{Q\_j}:=\prod\_{i\ne j}{Q\_i}$ and all components have small norm i.e., $\big|[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}\big|< Q\_i$ for $i=0,\ldots,k$
   Note: $$a=\sum\_{j=0}^k [a\hat{Q}\_j^{\-1}]\_{Q\_j}\hat{Q}\_j \bmod Q $$
   note: an alternative is to decompose by $([a]\_{Q\_0},\ldots,[a]\_{Q\_k})$ 
4. Set $P=\prod\_{i=0}^\alpha P\_i$ s.t $P\_i$ are primes and $|q\_i|<|P\_i|<\text{64 bits}$ for all $i$.
5. Define switching keys by a set of ciphertexts $$\text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}}:=(a^\*\_i\cdot  s\_{out}+ \hat{Q\_i}\cdot P\cdot s\_{in}+e^\*\_i, a^\*\_i) \bmod  PQ$$\*Note:\* An alternative, you can compute the decompose by only $[a]\_{Q\_i}$ and have $[\hat{Q\_i}^{\-1}]\_{Q\_i}$ multiply in the key which has the reduce the operations in the key switch i.e., $$\text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}}:=(a^\*\_i\cdot  s\_{out}+ \hat{Q\_i} \cdot [\hat{Q\_i}^{\-1}]\_{Q\_i}\cdot P\cdot s\_{in}+e^\*\_i, a^\*\_i) \bmod  PQ$$
6. mod\-up $[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}$ to be modulo $P\cdot Q$ and multiply, we have
$$
\begin{align}
\big[[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}\big]\_{PQ}\cdot \text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}} &= 
[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}\cdot \text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}} \bmod PQ \\ 
&= \big([a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}\cdot a^\*\_i\cdot s\_{out} + [a\cdot \hat{Q\_i}^{\-1}]\_{Q\_i}\cdot \hat{Q\_i}\cdot
P\cdot s\_{in}+ e^\*\_i[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i} , [a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i} a^\*\_i\big)\bmod PQ
\end{align}
$$

7. mod\_down to modulo Q
8. add all results: $e\_i^{**}:=P^{\-1}\cdot e^\*\_i[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}$, $a\_i^{**}:=P^{\-1}[a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}\cdot a^\*\_i$ 
$$t = \sum\_{i=0}^L \big([a\cdot\hat{Q\_i}^{\-1}]\_{Q\_i}\cdot \text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}}\big)= \sum\_{i=0}^L a^{**}\_i\cdot s\_{out}+ a\cdot s\_{in}+ \sum\_{i=0}^L e\_i^{**}$$
9. subtract from original ciphertext: $$ct\_0\- t=as\_{in}+m+e\-t=a^{***}s\_{out}+m+e+e^{***}$$ where $a^{***}:= \sum\_{i=0}^L a^{**}\_i$ and $e^{***}:=\sum\_{i=0}^L e\_i^{**}$

## Equivalent computation with inner product[^2]
Given a ciphertext $(b, a)$ and $swk^j=\big([\text{swk}^{(i)}\_{s\_{in}\rightarrow s\_{out}}]\_j \big)\_{i=0}^L$ for $j=0,1$ 
1. Let $d$ be the decomposition of $a$ after modulo\-up to $PQ$
2. $t\_1 = \langle d, swk^0 \rangle$ 
3. $t\_2=\langle d, swk^1\rangle$
4. $t = \lfloor P^{\-1}\cdot t\_1 \rceil$ (mod down)
5. $a^{***}=\lfloor P^{\-1}\cdot t\_2 \rceil$ (mod down)


Notes
\- this method require that we compute $L/\alpha$ key switch algorithms compared to $L$ computations for the method using single $P$.
\- $a$ is required to be in CRT representation before calling mod\-up in every method described above. 
\- Using multiple "small" switch keys requires more memory: $(L+1) \Delta n\frac{L}{2}$ vs. $2L\Delta n$ of single key i.e., it is quadratic in $L$ the effective multiplication depth of the scheme, $n$ ring dimension and $\Delta$ the scale. But it enables more levels for homomorphic computation for secured setups.
\- $a$ is the random element of a given ciphertext and so it's decomposition is needed to be compute right before key switch and can't be precomputed during initialization of the scheme.


Related: 
\- [History of key switch methods](./History%20of%20key%20switch%20methods.md)
\- [Gadget Decomposition and External Product](./Gadget%20Decomposition%20and%20External%20Product.md)
\- Generate switch keys homomorphically: [Hierarchical rotation key](./Hierarchical%20rotation%20key.md),[Generate rotation keys homomorphically by scheme conversion (RLWE<->LWE)](Generate%20rotation%20keys%20homomorphically%20by%20scheme%20conversion%20(RLWE%3C-%3ELWE).md)


## Created 2022\-03\-01 09:30


[^2]: https://eprint.iacr.org/2020/1203.pdf#page=12.72, page 12