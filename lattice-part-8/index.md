# Lattice Part 8: Identity-Based and Functional Encryption From Lattices


Reading: Peikert's survey is the right wide-angle backbone here because it keeps advanced lattice constructions on one trapdoor-and-sampling axis instead of letting IBE, ABE, and FE turn into unrelated acronyms.[^peikert] Boneh-Shoup is useful for a second reason: it separates public-key encryption, identity-based encryption, attribute-based encryption, and functional encryption at the interface level before we even choose lattices.[^bs]

The only extra primitive beyond Parts 0-3 is GPV-style trapdoor sampling: a hidden short basis is not merely evidence for a hard relation; it is a usable interface for sampling short preimages.[^gpv] Once that interface exists, the next question is forced: can decryption authority be derived, specialized, or delegated instead of being one monolithic secret key?

That is the organizing mechanism of this chapter. IBE, HIBE, ABE, and FE are different answers to the same question: given a master trapdoor, what short secret material can we derive for a public label, a hierarchy path, an attribute policy, or a function key, and what exactly should that derived key be allowed to recover?[^abb][^afv][^boyen]

<!--more-->

## Reader State After Part 0-3

This chapter can be read without assuming Parts 4-7, but it uses their vocabulary only as a thin wrapper. The minimum prerequisite is still Part 0-3:

- A public modular matrix defines exact relations such as $A\mathbf{x}=\mathbf{u}\pmod q$.
- The useful witnesses are short integer vectors or short bases.
- A coset $\mathbf{x}_0+\Lambda_q^\perp(A)$ is the set of all solutions to one inhomogeneous target.
- A good hidden basis lets the holder sample short representatives from those cosets.

The advanced encryption primitives in this article all ask the same question:

$$
\text{which short coset representative may this user, identity, policy, or function receive?}
$$

IBE, HIBE, ABE, and FE differ at the interface level, but they share this trapdoor-sampling backbone. The goal is to track that shared mechanism without collapsing the primitives into one acronym list.

### Notation For Delegated Decryption

Use $\theta$ for a public selector: an identity, identity path, attribute-policy object, or function label. The public relation associated with $\theta$ is a matrix

$$
F_\theta\in\mathbb{Z}_q^{n\times m}.
$$

A derived secret key is a short vector or short basis object $K_\theta$ that lets decryption cancel an LWE-style public relation. In the simplest vector form, the authority samples

$$
\mathbf{k}_\theta\quad\text{such that}\quad F_\theta\mathbf{k}_\theta=\mathbf{u}\pmod q.
$$

The IACR-style reading discipline is: first identify the selector $\theta$, then the relation $F_\theta$, then the short derived object, and only then the security surface.

## Why Trapdoors Naturally Lead To Delegation

### From One Secret Key To Derived Secret Material

In ordinary public-key encryption, the secret key is attached to one public key. In the trapdoor lattice line, the stronger object is not "a secret that decrypts one ciphertext family," but "a hidden sampling structure that can manufacture short witnesses for a whole public relation family."

Write that relation family as

$$
F_\theta \in \mathbb{Z}_q^{n \times m},
$$

where the public label $\theta$ might be an identity string, an identity prefix, an attribute structure, or a function description. A master trapdoor $T$ lets the authority derive a short object

$$
\mathbf{k}_\theta \leftarrow \textsf{TrapSample}(T,\theta)
\quad\text{such that}\quad
F_\theta \mathbf{k}_\theta = \mathbf{u} \pmod q
$$

for some public target $\mathbf{u}$, with $\lVert\mathbf{k}_\theta\rVert$ or the corresponding basis quality kept under control.

This is already much richer than an ordinary LWE public-key baseline. The hidden object is no longer just a secret inverse for one instance. It is an API for generating many short inverse objects, each tied to a public derivation target.

The generic dual-style decryption chain makes the point explicit. Suppose encryption under label $\theta$ produces

$$
c_0 = \langle \mathbf{u}, \mathbf{s}\rangle + x + \Delta m \pmod q,
\qquad
\mathbf{c} = F_\theta^\top \mathbf{s} + \mathbf{e} \pmod q,
$$

where $\mathbf{s}$ is ephemeral LWE randomness, $\mathbf{e}$ and $x$ are small errors, and $\Delta$ separates message scale from noise scale. A derived key $\mathbf{k}_\theta$ then gives

$$
\begin{aligned}
c_0 - \langle \mathbf{k}_\theta, \mathbf{c}\rangle
&=
\langle \mathbf{u}, \mathbf{s}\rangle + x + \Delta m \\
&\quad - \langle \mathbf{k}_\theta, F_\theta^\top \mathbf{s} + \mathbf{e}\rangle \pmod q \\
&=
\langle \mathbf{u}, \mathbf{s}\rangle + x + \Delta m \\
&\quad - \langle F_\theta \mathbf{k}_\theta, \mathbf{s}\rangle \\
&\quad - \langle \mathbf{k}_\theta, \mathbf{e}\rangle \pmod q \\
&=
\Delta m + x - \langle \mathbf{k}_\theta, \mathbf{e}\rangle \pmod q.
\end{aligned}
$$

If $\mathbf{k}_\theta$ is short and the noise terms are small, the last term stays within the decoding margin. This is the common trapdoor-decryption template behind the rest of the article.

> Local invariant: the advanced lattice-encryption line is really about derived short decryption material. The acronym changes; the short-object sampling spine does not.

### What Must Stay Hidden

Delegation only works if the derived object is useful enough to cancel the public relation, but not rich enough to reconstruct the master trapdoor. The GPV trapdoor-sampling line already shows why deterministic or overly basis-dependent outputs are dangerous: repeated samples can leak the hidden geometry through distribution bias.[^gpv]

The same concern returns here in a stronger form. A lattice authority may extract many user keys over time. So the construction has to maintain at least three boundaries:

1. a child or derived key should reveal only one slice of decryption authority;
2. many extracted keys should not algebraically splice back into the master trapdoor;
3. the extracted-key distribution should not fingerprint the hidden short basis.

This is why randomized short-object sampling remains central even after we leave signatures. In signatures the risk was trapdoor leakage across many responses. In IBE/HIBE/ABE/FE the risk is trapdoor leakage across many user keys plus many ciphertexts. Same geometric issue, larger authority surface.

## Lattice Identity-Based Encryption

### Identity Strings As Public Derivation Targets

Boneh-Shoup's taxonomy is still the cleanest conceptual starting point: IBE changes the interface from "user publishes a chosen public key" to "the system publishes master parameters, and an identity string becomes the public encryption target."[^bs] In the lattice line, that interface change becomes an explicit sampling problem.

The identity string does not become a secret. It selects a public matrix family member. A standard lattice pattern is to keep a base matrix $A$ with trapdoor $T_A$, add public matrices and a gadget block, and form an identity-specific matrix of the shape

$$
F_{\mathsf{id}} = [A \mid B + H(\mathsf{id})G],
$$

where $H(\mathsf{id})$ is a public encoding of the identity and $G$ is a gadget matrix that makes the derived block algebraically manageable. The master authority uses $T_A$ to sample a short key

$$
\mathbf{e}_{\mathsf{id}} \leftarrow \textsf{SamplePre}(T_A, F_{\mathsf{id}}, \mathbf{u})
\quad\text{with}\quad
F_{\mathsf{id}}\mathbf{e}_{\mathsf{id}} = \mathbf{u} \pmod q.
$$

Encryption to identity $\mathsf{id}$ then uses $F_{\mathsf{id}}$ publicly:

$$
c_0 = \langle \mathbf{u}, \mathbf{s}\rangle + x + \Delta m,
\qquad
\mathbf{c} = F_{\mathsf{id}}^\top \mathbf{s} + \mathbf{e}.
$$

Decryption is exactly the generic chain specialized to identity:

$$
c_0 - \langle \mathbf{e}_{\mathsf{id}}, \mathbf{c}\rangle=\Delta m + x - \langle \mathbf{e}_{\mathsf{id}}, \mathbf{e}\rangle \pmod q.
$$

So the difference from ordinary public-key encryption is not merely administrative PKI simplification. Constructionally, IBE replaces one fixed decryption key with a trapdoor-powered derivation algorithm that can produce many identity-labeled short keys after setup. Agrawal, Boneh, and Boyen's lattice (H)IBE line is important precisely because it turns this into an efficient standard-model construction rather than an informal random-oracle heuristic.[^abb]

### Delegated Keys In The Hierarchical Setting

HIBE asks for more than identity extraction. It asks for identity-path delegation. If the identity is a tuple

$$
\mathsf{id}_{1:\ell} = (\mathsf{id}_1,\ldots,\mathsf{id}_\ell),
$$

then the public relation family becomes depth-aware, for example through a matrix of the schematic form

$$
F_{\mathsf{id}_{1:\ell}}=[A \mid B_1 + H(\mathsf{id}_1)G \mid \cdots \mid B_\ell + H(\mathsf{id}_\ell)G].
$$

Now the extracted secret is no longer just "a user key." It is also a local delegation trapdoor. Depending on the concrete scheme, the secret key for a prefix may be a short basis for $\Lambda_q^\perp(F_{\mathsf{id}_{1:\ell}})$ or an equivalent short preimage object strong enough to derive child keys. A schematic delegation step looks like

$$
SK_{\mathsf{id}_{1:\ell+1}}
\leftarrow
\textsf{BasisDel}\!\left(
SK_{\mathsf{id}_{1:\ell}},
B_{\ell+1} + H(\mathsf{id}_{\ell+1})G
\right).
$$

This is the main reason HIBE feels natural in lattices once trapdoor sampling is on the table. The scheme already has a sampling interface that can manipulate short trapdoor objects. HIBE asks that the output itself remain trapdoor-like for a narrower public family.

But the price is equally geometric. Every delegation step worsens basis quality or enlarges the norm budget. The Gaussian width and modulus must dominate that growth, otherwise the decryption error term

$$
\langle \mathbf{k}_{\mathsf{id}_{1:\ell}}, \mathbf{e}\rangle
$$

stops being negligible. So HIBE depth is not just an organizational tree depth. It is a trapdoor-quality budget.

## From Attributes To Predicates

### What ABE Adds

IBE is equality on one public string: the key for $\mathsf{id}$ opens ciphertexts addressed to the same $\mathsf{id}$. ABE enlarges the authorization language from equality to structured predicates on attribute sets.

That conceptual step is easy to say and easy to blur, so it is worth separating two common interfaces:

1. key-policy ABE (KP-ABE): ciphertext carries an attribute set $X$, key carries a policy $P$, and decryption succeeds iff $P(X)=1$;
2. ciphertext-policy ABE (CP-ABE): key carries attributes, ciphertext carries the policy.

Constructionally, this is not just "IBE with more strings." ABE needs a policy representation that resists collusion. In lattice terms, one typically spreads the ciphertext relation across multiple attribute-tagged blocks and equips the authorized key with a collection of short components plus recombination coefficients. If a subset $I$ of attributes satisfies the policy, there exist weights $\omega_i$ such that an authorized linear combination collapses the public randomness:

$$
\sum_{i\in I} \omega_i \big(c_{0,i} - \langle \mathbf{k}_i, \mathbf{c}_i\rangle\big)=\Delta m + \text{small noise}.
$$

The exact policy encoding may come from monotone span programs or related linear-secret-sharing structure, but the lattice mechanism underneath is still the same: short secret material is sampled so that the right authorized combination cancels the LWE relation while unauthorized coalitions cannot produce an equivalent short combination.[^boyen]

These are the construction-level differences that often get lost. In IBE, the key is indexed by one equality target. In ABE, the key or ciphertext carries a policy structure whose authorized recombination is itself part of the cryptographic object.

### Why Predicate Structure Matters

At the abstraction level, one can say that ABE is a restricted form of FE. But constructionally that shortcut hides the important split.

In ABE, successful authorization still recovers the whole plaintext. The cryptographic burden is to represent a structured predicate and prevent colluding users from stitching partial authorizations into a valid one.

In FE, authorization is not the last interface boundary. The output itself becomes restricted. A function key does not merely answer "may I decrypt?" It answers "which function of the encrypted object am I entitled to learn?"

That boundary is exactly why Boyen's 2013 framing is useful here. "Attribute-based functional encryption" is not redundant terminology. The attribute side describes how authorization structure is encoded; the functional side describes whether the output rule is still full-message recovery or has already been compressed to a prescribed image.[^boyen]

So ABE should be read as the midpoint between IBE and FE:

- IBE: equality predicate on one public identity, full-message recovery;
- HIBE: equality predicate plus delegated subtree authority, full-message recovery;
- ABE: structured Boolean authorization, still full-message recovery;
- FE: function-labeled decryption, intentionally partial recovery.

That is why the taxonomy must be separated before discussing constructions. Otherwise "fine-grained access control" compresses away the actual change in the decryption interface.

## Functional Encryption From Lattices

### Inner Products As The First Clean Example

Functional encryption is the point where the derived key is indexed by a function $f$, and decryption returns $f(m)$ rather than $m$. This is the branch where "delegated decryption" becomes "delegated leakage with a proof obligation."

The early lattice line is best entered through inner-product predicates and inner-product style function families rather than through vague claims about arbitrary computation. Agrawal, Freeman, and Vaikuntanathan show that LWE-based trapdoor machinery can issue keys tied to linear forms, so that decryption learns only the designated predicate or functional projection attached to that key.[^afv]

Schematically, think of a plaintext vector $\mathbf{x}=(x_1,\ldots,x_t)$ and a function label $\mathbf{y}=(y_1,\ldots,y_t)$. Public parameters expose matrix blocks that can encode the coordinates of $\mathbf{x}$ into ciphertext form. The authority then samples a short key $SK_{\mathbf{y}}$ for the linear combination specified by $\mathbf{y}$. Decryption does not reconstruct all coordinates. It combines ciphertext blocks using the function key so that the public randomness cancels and the surviving payload is exactly the authorized linear form, or a predicate of it:

$$
\textsf{Dec}(SK_{\mathbf{y}}, CT_{\mathbf{x}})
\approx
\Delta \langle \mathbf{x}, \mathbf{y}\rangle + \text{small noise},
$$

or, in predicate-style variants, a zero-test / threshold test on the same quantity.

The point is not that one equation by itself yields general FE. The point is narrower and more important for this series: trapdoor sampling can tie a secret key to a function description rather than to a recipient label. That is the exact place where the lattice line leaves "who can decrypt?" and enters "what can a decryptor learn?"

### What Changes In The Security Surface

Once decryption is supposed to reveal $f(m)$, indistinguishability must be stated against that intended leakage. A standard admissibility condition for challenge messages $m_0,m_1$ therefore looks like

$$
f_j(m_0)=f_j(m_1)
\qquad
\text{for every queried function key } SK_{f_j}.
$$

Otherwise perfect hiding is impossible, because the adversary is supposed to learn those outputs.

This is a genuinely different security surface from IBE or ABE. These are the security-surface differences between taxonomy branches, not merely different names for the same delegated-key interface.

In IBE/HIBE/ABE:

- authorized decryption reveals the full plaintext;
- unauthorized users should learn essentially nothing;
- collusion resistance means partial capabilities should not compose into full authorization.

In FE:

- each key intentionally leaks a prescribed statistic or predicate of the message;
- multiple function keys leak the joint image $(f_1(m),\ldots,f_t(m))$;
- the function family, message domain, and norm bounds now matter because too many linear queries may reconstruct the entire plaintext.

So FE is not just "stronger ABE." It is a different contract between ciphertext privacy and derived-key capability.

## The Common Lattice Mechanism

### Trapdoor Sampling As The Shared Engine

Peikert's survey is especially useful here because it refuses to present these constructions as separate species.[^peikert] Under the hood, the shared mechanism is short-object derivation from a master trapdoor. In exactly that sense, trapdoor sampling ideas connect directly to delegated decryption and key-derivation capabilities.

One can compress the common template as follows:

1. `Setup`: publish a relation family $\{F_\theta\}$ and keep a master trapdoor $T$ for the base object behind that family.
2. `Derive`: use $T$, or a delegated descendant of $T$, to sample a short key object $K_\theta$ tied to label / policy / function $\theta$.
3. `Encrypt`: publish noisy linear data whose hidden structure is parameterized by the public side of the interface.
4. `Decrypt/Eval`: combine $K_\theta$ with the ciphertext so that the public linear part cancels and only the intended payload remains.

The payload is what separates the primitives:

| Primitive | Public selector | Derived key authorizes | Output |
| --- | --- | --- | --- |
| IBE | identity $\mathsf{id}$ | equality on $\mathsf{id}$ | full $m$ |
| HIBE | identity path $\mathsf{id}_{1:\ell}$ | descendant subtree | full $m$ |
| ABE | attributes / policy | policy satisfaction | full $m$ |
| FE | function label $f$ or $\mathbf{y}$ | designated functional leakage | $f(m)$ or a predicate of it |

What changes from row to row is not whether lattices are used. What changes is the type of short secret object being sampled and the semantic meaning of the surviving payload.

This is why trapdoor delegation is the right spine. The master trapdoor is not the same object as every extracted key, but each branch derives its authority by manufacturing a new short object whose geometry is still controlled by that hidden sampling structure.

### Why These Systems Are Not Ordinary PKI

Calling lattice IBE/ABE/FE "PKI variants" is too weak and usually wrong.

Certificates in ordinary PKI bind names to preexisting public keys. They do not change the algebra of decryption. Here the label, policy, or function is part of the algebra itself:

- it changes which public matrix family member is encrypted against;
- it changes what short object the authority must sample;
- and in FE it changes what the decryption output is even allowed to mean.

So the right mental model is not administrative key management. It is richer encryption interfaces built on the same trapdoor-sampling engine introduced above.

That perspective also explains why the primary papers line up the way they do. Agrawal-Boneh-Boyen show how to turn trapdoors into identity and hierarchy derivation.[^abb] Boyen shows how attribute structure becomes a first-class part of the lattice key/ciphertext algebra.[^boyen] Agrawal-Freeman-Vaikuntanathan push the same line into function-indexed keys.[^afv] The taxonomy broadens, but the engine stays recognizable.

## Conclusion

The clean way to remember this chapter is not "lattices can also do IBE, HIBE, ABE, and FE." That is true and not very useful.

The useful claim is narrower:

$$
\text{short basis / trapdoor}
\Longrightarrow
\text{preimage sampling}
\Longrightarrow
\text{derived short decryption material}
\Longrightarrow
\text{delegated identities, policies, and functions}.
$$

IBE is equality-targeted derived decryption. HIBE is delegated identity-path derivation. ABE is policy-structured full-message recovery. FE is function-structured partial recovery. If those four are kept separate at the interface and construction levels, the lattice line stops looking like an acronym catalog and starts looking like one continuous trapdoor-delegation mechanism.

## References

Suggested reading order:

1. Boneh-Shoup first, for the interface taxonomy: PKE, IBE, ABE, and FE should not be conflated.
2. Peikert next, for the lattice trapdoor and sampling backbone.
3. GPV before the advanced papers, because the short-preimage sampling interface is the mechanism reused throughout.
4. Agrawal-Boneh-Boyen, Agrawal-Freeman-Vaikuntanathan, and Boyen last, with the selector $\theta$, relation $F_\theta$, and derived key object tracked separately in each construction.

[^peikert]: Chris Peikert, *A Decade of Lattice Cryptography*, 2016.
[^bs]: Dan Boneh and Victor Shoup, *A Graduate Course in Applied Cryptography*, chapters on public-key encryption, IBE, ABE, and FE.
[^gpv]: Craig Gentry, Chris Peikert, and Vinod Vaikuntanathan, *Trapdoors for Hard Lattices and New Cryptographic Constructions*, 2008.
[^abb]: Shweta Agrawal, Dan Boneh, and Xavier Boyen, *Efficient Lattice (H)IBE in the Standard Model*, 2010.
[^afv]: Shweta Agrawal, David Mandell Freeman, and Vinod Vaikuntanathan, *Functional Encryption for Inner Product Predicates from Learning with Errors*, 2011.
[^boyen]: Xavier Boyen, *Attribute-Based Functional Encryption on Lattices*, 2013.

