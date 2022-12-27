# Pan y Tomate

Pan y Tomate employs cryptography to create bulletproof credibility.

---

## zk Proof Of Solvency

zk Proof Of Solvency (zkPOS) makes use of [zk SNARKs](https://minaprotocol.com/blog/what-are-zk-snarks) to let Centralized Exchanges generate **credible** and **private** Proof of Liabilities.

### Why?

Since the FTX collapse, many exchanges are exploring [ways](https://niccarter.info/proof-of-reserves/) to demonstrate their solvency to their customers. In particular, [Proof of Solvency](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html) is achieved by proving that the assets under control are greater than the liabilities.

$Assets \geq  Liabilities$ 

Proving ownership of assets is relatively easy to achieve: by signing a specific message from their wallet, an exchange can prove ownership of an account and, therefore, of the assets controlled by that account.

The liabilities are the sum of the balances of the customers operating on the exchange. The exchange owes this sum to its customers. The exchange must be able to cover its liabilities with its assets. 

Proving liabilities is tricker. 

The simplest solution would be opening up the database and disclosing every user-balance entry. The user can check if they have been included in the list so that the exchange is not understating their liabilities. This solution also prevents the exchange to add fake accounts with negative balances to understate their liabilities.

By summing all the balances together and comparing the sum to the total assets of the exchange is possible to verify the solvency of the exchange.

This Proof of Liability implementation is **credible** in the eye of customers but sacrifices the **privacy** of the exchange's (and its customers') data.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-1.png" width="500" align="center" />
</div>
<br>

Other implementations, such as [Binance's](https://www.binance.com/en/proof-of-reserves), achieve Proof of Liability by splitting users' balances across the leaves of a [Merkle Tree](https://en.wikipedia.org/wiki/Merkle_tree). The exchange would publish the Merkle Tree itself together with the total sum of liabilities. Each user can verify being included in the Merkle Tree. Since the individual balances are not public (only a hash is visible to the outside) users cannot independently sum the balances to calculate the total liabilities. This process is performed by an auditor that customers need to trust. 

This Proof of Liability implementation is partially **private** but sacrifices the **credibility** of the proof.

This solution is only partially private because critical information are leaked in the process such as the total number of users of Binance or the amount of liabilities that Binance, which are the sum of the balances of its users. 

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-2.png" width="500" align="center" />
</div>
<br>

## What

zkPOS makes use of zk SNARKs to let Centralized Exchanges (CEX) generate a **credible** and **private** Proof of Solvency. 

**Credbile** means that user can verify the Solvency of a CEX independently and without having to trust any auditor.
**Private** means without leaking to the public any user's private data or other Business Intelligence Data.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-3.png" width="500" align="center" />
</div>
<br>

Ownership of assets is achieved by signing a specific message from their wallet. From that, it is possible to extract the exact amount of assets controlled by the CEX, at that specific time. 

The Liabilities of an Exchange are the sum of funds owned by the customers operating on the exchange. This balances are computed inside a Merkle Tree. This Merkle Tree is private and gets never shared to the customers.

zkPOL is a set of APIs that allows an exchange to plug in their database and generate a zk Proof to be shared with their users. 

Starting from the proof, a user can verify that they have been included (with their correct balance) in the computation of the Liabilities of the Exchange and that the total amount of Liabilities is less than its controlled Assets. 

The rule is simple: if enough users request a Proof of Liability and they can all verify it, it becomes evident that the Exchange is not lying or understating its liabilities. If just one user cannot verify the proof, the Exchange is lying 
about its actual liabilities. 

The proof can be fastly verifiable and doesn't leak any data about the number of users that operate on the exchange, the funds owned by other users or even the total amount of liabilities of the exchange itself! 

### How

- Sparse Merkle Sum Tree is a plug-in component to an existing database of users. The entries of the database (`username -> balance`) are added to a Sparse Merkle Sum Tree data structure. The total sum of the leaves represents the total liabilities of a CEX.
- zkPOS Proving System contains the circuits enforcing the rules that the Exchange must abide by to generate its proof of solvency and a set of Javascript APIs to generate (and verify) Proof of Liabilities for each user

## Who

[Enrico Bottazzi](https://github.com/enricobottazzi), Developer and Technical Writer @[Iden3](https://iden3.io/) @[PolygonID](https://polygon.technology/polygon-id)

## When

- [ ] Jan 2023, complete the SMST library
- [ ] Feb 2023, complete the Proving System library
- [ ] March 2023, first POC with an Exchange

## Main Challenges

- Create a friendly user experience and explain it to users
- Make the APIs pluggable with existing db systems
- Make it work with users' balances denominated in different currencies








