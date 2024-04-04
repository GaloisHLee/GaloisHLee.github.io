# SM4 Note



SM4 learning.


<!--more-->

# 1.Abstract

> bit: 0 or 1
>
> byte: 8bits
>
> word: 4bytes

SM4 is a block cipher that operates on blocks of 128 bits (16 bytes or 4 words). It uses a 128-bit key for encryption and decryption. Similar to DES and AES, the process involves 32 rounds of iteration, with each round requiring a round key.

During each round, a round key is applied to the input data. These round keys are derived from the original encryption key using a key expansion algorithm. The round function in SM4 combines substitution, permutation, and bitwise operations to provide diffusion and confusion properties, ensuring the security of the algorithm.

SM4 is a widely used symmetric encryption algorithm, particularly in China where it was developed.

# 2.Encryption

## 2.1 Simply

The SM4 algorithm is a block cipher with a block length of 4 words, where each word represents a 32-bit value. The input to the algorithm is a plaintext $(X_0, X_1, X_2, X_3)$,  where $ X_i$ represents a 32-bit word. After encryption, the output is a ciphertext $(Y_0, Y_1, Y_2, Y_3)$, where  $Y_i$  represents a 32-bit word.

The encryption process consists of two steps: 32 rounds of iteration and one final inverse transformation.



## 2.2 Iterations

Formally,we are requiring  to construct thirty-two rounds of iterations about this four-word plaintext. Each iteration needs a one-word **Wheel key**, totally 32 keys marked as $rk_0,rk_1,...,rk_31$.

The process of iteration is just using a wheel function $F$ over and over again ,simply calculating the very next word.

$F$ there, could accept  four one-word plaintext and a one-word round key as parameters, and finally produce a one-word result.



For instance:

- Iteration 1: $X_4 = F(X_0,X_1,X_2,X_3,rk_0)$
- Iteration 2: $X_5 = F(X_1,X_2,X_3,X_4,rk_1)$
  ……
- Iteration 32: $X_{35} = F(X_{31},X_{32},X_{33},X_{34},rk_{31})$



## 2.3 Inverse-order

$$
let~ (Y_0,Y_1,Y_2,Y_3) = (X_{35},X_{34},X_{33},X_{32})
$$



**That is that.**



# 3.Decryption

The decryption process of SM4 is exactly the same as the encryption process, which also includes 32 iterations and one inverse-order transformation. Only when the round iteration, the round key needs to be used in reverse order.

# 4.Structure of $F$

## 4.1 Function $F$

$$
F(X_{i},X_{i+1},X_{i+2},X_{I+3},rk_i)\overset{\text{def}}{=}X_i\oplus T(X_{i+1}\oplus X_{i+2} \oplus X_{i+3} \oplus rk_i)
$$

We have $T$ named Synthetic replacement.

## 4.2 Replacement $T$

$$
C \overset{\text{def}}{=}T(A)，
A\overset{\text{def}}{=}L(\tau(A))
$$

****

**Nonlinear transformation**  $\tau$

There is an input A consisting of four bits ,marked as $A = (a_0,a_1,a_2,a_3)$,we define that $\tau(A) = B$,$B =(b_0,b_1,b_2,b_3)$.
$$
B=(b_0,b_1,b_2,b_3)=\tau(A)
=(Sbox(a_0),Sbox(a_1),Sbox(a_2),Sbox(a_3))
$$
The specific substitution method is the same as byte substitution in AES. First, the 8-bit binary of a byte is written into 2-bit hexadecimal, and then the first digit of the hexadecimal is the row, the second digit is the column, and the corresponding number is found in the S-box table. This allows you to replace one byte with another.



**Linear Transformation** $L$
$$
C  \overset{\text{def}}{=}L(B)=B=B\oplus (B<<<2)\oplus (B<<<10) \oplus (B<<<24)
$$


# 5.Key Expansion

How  32 keys derives from Original four-word key?  That is not a Hume problem with cryptography, but a graceful algorithm.

We have original key :
$$
MK=(MK_0,MK_1,MK_2,MK_3)
$$

##  5.1Initialization

There we preset a set of systemic parameters $FK_i,i∈\{0,1,2,3\}$
$$
(K_0,K_1,K_2,K_3)=(MK_0 \oplus FK_0,MK_1 \oplus FK_1,MK_2 \oplus FK_2,MK_3 \oplus FK_3)
$$
There :
$$
FK_0=(A3B1BAC6),FK_1=(56AA3350),FK_2=(677D9197),FK_3=(B27022DC)
$$


## 5.2Iteration

**Method:**

After initialization, we finally get four new word $(K_0,K_1,K_2,K_3)$.



**Iteration 1:**
$$
K_4 = K_0 \oplus T'(K_1\oplus K_2 \oplus K_3 \oplus CK_0)
$$
then let $K_4=rk_0$



……



**Iteration 32:**
$$
K_{35} =K_{31} \oplus T'(K_{32}\oplus K_{33} \oplus K_{34} \oplus CK_{31})
$$
then let $K_{35}=rk_{31}$


$$
rk_i=K_{i+4} \oplus T'(K_{i+1}\oplus K_{i+2} \oplus K_{i+3} \oplus CK_{i})i∈\{0,1,...,31\}
$$

## 5.3 Replacement



There are totally 32 $CK_i,i∈[0，31]$,every round needs different $CK_i$.
$$
CK_i=(ck_{i,0},ck_{i,1},ck_{i,2},ck_{i,3})
$$

$$
ck_{i,j}=(4i+j)×7 \pmod{ 256}
$$

Here we calculate each $ck_{i,j}$ easily.










