---
share: "true"
---

# CKKS Rotations
#### Notation
- Let $ct(X)$ be a [R-LWE ciphertext (RLWE)](./R-LWE%20ciphertext%20(RLWE).md) encrypts $m(X)$ w.r.t secret $s(X)$ i.e., $ct(X)=(b(X),a(X)) = \big[a(X) s(X)+m(X)+e(X),a(X)\big]\bmod Q$ 
- $\Phi_{M}(X)= \prod_{i=0}^{\varphi(M)}(X-\zeta_i)$ where $\{\zeta_i\}$ are the $2n$-th primitive roots of unity over the complex number field.
- Note: the degree of $c_i(X)$ is $n$ because the value of Euler totient function $\varphi(2n)=n$ (all the even numbers < $M$)

## Background - decoding
- motivation: encode vector of complex numbers into polynomial in $\mathbb{Z}$.
- plaintext $m(X)$ is decoded to slots by $[m(z_1),m(z_2),\ldots,m(z_n)]$ where $z_i$ are the $2n$-th primitive roots of unity. For every $z_i$ the roots contain it's conjugate $z_j$. So only half of the roots are used and they decode into $n/2$ slots are used. let call them $\hat{z}_{1}$
- $,\ldots,\hat{z}_{n/2}$. 


#### What is $ct(X^k)$ ?
- $ct(X^k)=\big(b(X^k),a(X^k)\big)=\big[a(X^k)s(X^k)+m(X^k)+e(X^k),a(X^k)\big]\bmod Q$
- the norm $|e(X^k)|=|e(X)|$ is small
- $ct(X^k)$ is a valid [R-LWE ciphertext (RLWE)](./R-LWE%20ciphertext%20(RLWE).md) which  encrypts $m(X^k)$ w.r.t secret key $s(X^k)$.

#### Is there $k$ s.t. the plaintext $m(X^k)$ is a rotation by one slot?
-   $m(X)$ encode the vector $\{m(\zeta^i)\}_{i\in \mathbb{Z}_{2n}^\times}$ where $\zeta:=e^{\frac{2\pi i}{2n}}$. In other words, the vector is computed by evaluating $m(X)$ in all $2n$-th primitive roots of unity. This is called the [Canonical mapping (CKKS encoding)](./Canonical%20mapping%20(CKKS%20encoding).md).
- The 2n-th primitive roots of unity are the following $\zeta^t$ where $\gcd(t,2n)=1$ in other words for all $t\in \mathbb{Z}_{2n}^\times$. 
- Note: The primitive roots do not form a group (the unity is not included)
- A theorem in group theory state that if $2n$ a power of 2 then $\mathbb{Z}_{2n}^\times\cong C_2\times C_{n/2}$ , see  [Multiplicative group of integers modulo n](./Multiplicative%20group%20of%20integers%20modulo%20n.md).
- Let $k$ be a generator of the cyclic group $C_{n/2}$ which is a subgroup of $\mathbb{Z}_{2n}^\times$ i.e., $\mathbb{Z}_{2n}^\times=\{1,k,k^2,\ldots,k^{n/2-1},-1,-k,-k^2,\ldots,-k^{n/2-1}\}$
-  We consider the powers of $\zeta$ that are associated with $\{1,k,k^2,\ldots,k^{n/2-1}\}$. The rest $n/2$ powers of $\zeta$ are their conjugates (because they are multiply by an involution) 
-  Evaluating $m(X^k)$ with the ordered zeta's is the same as evaluating $m(X)$ by  the ordered zeta's rotated by one i.e., the slots are rotated by one.  replacing $X$ with $\zeta^k$ we get: 
$$ \begin{align*}
&m\big((\zeta)^k\big)=m\big(\zeta^k\big)\\
&m\big((\zeta^k)^k\big)=m\big(\zeta^{k^2}\big)\\
&m\big((\zeta^{k^2})^k\big)=m\big(\zeta^{k^3}\big)\\
& \vdots\\
& m\big((\zeta^{k^{N/2-1}})^k\big)=m\big(\zeta^{k^{N/2}}\big)=m\big(\zeta^1\big)\end{align*}$$ 
-  Therefore, $m(X^k)$ is a rotation by one slot of the encoded vector in $m(X)$
-   Conclusion: $ct^\prime$ encrypts the rotation by one slot of the original plaintext $m(X)$ w.r.t secret key $s(X^k)$.
- note that for rotating the slots by $r$ step, we compute $ct(X^{k^r})$
- note that the basis for the rotation is the existence of a generator $k$ that exist due to the theorem $\mathbb{Z}_{2n}^\times\cong C_2\times C_{n/2}$ for $2n$ a power of 2.
- note that $M$, the dimension of the polynomial ring, is required to be a power of 2 to enable rotation based on this computation.

#### How to convert $ct^\prime$  to encrypt $m(X^k)$ w.r.t $s(X)$?
- In order to continue and use $ct^\prime$ in more computations it requires to have the same secret key of other ciphertext it "interact" with. This means we  should have $ct^\prime(X^k)=\big(b^\prime(X),a^\prime(X)\big)=\big[a^\prime(X)s(X)+m(X^k)+e^\prime(X),a^\prime(X)\big] \bmod Q$ 
- We need to compute [key switching](./CKKS%20Key%20switching.md) $s(X^k)\rightarrow s(X)$ for $ct^\prime$ and get a new ciphertext $ct^{\prime\prime}$.

#TODO: explain why it the same to compute x^k in both sides? is it because x^k is automorphism?
#### Why CKKS use k=5 and not k=3?
#TODO: find out why and summarize here.

## Usage
- Rotation is used for efficiently computing a [Plaintext matrix & encrypted vector multiplication (Diagonal and Rotate)](./Plaintext%20matrix%20&%20encrypted%20vector%20multiplication%20(Diagonal%20and%20Rotate).md)  e.g., for computing FFT matrix multiplication during bootstrapping which is used for transforming between [Double-CRT](./Double-CRT.md) and coefficient representations.
- [sum of all slots](./sum%20of%20all%20slots.md)

Related: [Automorphism in NTT representation](./Automorphism%20in%20NTT%20representation.md)
## References
- [@Bootstrapping for Approximate Homomorphic Encryption](./@Bootstrapping%20for%20Approximate%20Homomorphic%20Encryption.md) section 4.2, page 9.


## Created 2022-03-02 08:47
