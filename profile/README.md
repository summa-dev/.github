# Pan y Tomate

## Problem Statement 

People don't trust Centralized Exchanges (CEXs)

## Solution 

Pan y Tomate is an open source tool that employs cryptographic to let CEXs achieve credibility maintaining secrecy over their Business Information.

**Credibility** - The validity of the proof fully ensures that the rules are met. Each user of the exchange can verify the validity of the proof without having to rely or trust auditors or other external parties.

**Secrecy** - The Business Information of the exchange, such as the total balances of each users, the number of users and total amount of liabilities are not revealed to the public.

---

### Why?

Since the FTX collapse, many exchanges are exploring [ways](https://niccarter.info/proof-of-reserves/) to demonstrate their solvency to their customers. In particular, [Proof of Solvency](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html) is achieved by proving that the assets under control are greater than the liabilities. Or in simple terms, that the exchange have the assets to cover all the balances of its users.

$Assets \geq  Liabilities$ 

Proving ownership of assets is relatively easy to achieve: by signing a specific message from their wallet, an exchange can prove ownership of an account and, therefore, of the assets controlled by that account.

The liabilities are the sum of the balances of the customers operating on the exchange. The exchange owes this sum to its customers. The exchange must be able to cover its liabilities with its assets. 

Proving liabilities is tricker. 

The simplest solution would be opening up the database and disclosing every user-balance entry. The user can check if they have been included in the list so that the exchange is not understating their liabilities. This solution also prevents the exchange to add fake accounts with negative balances to understate their liabilities.

By summing all the balances together and comparing the sum to the total assets of the exchange is possible to verify the solvency of the exchange.

This Proof of Liability implementation is **credible** in the eye of customers but sacrifices the **secrecy** of the exchange's business data.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-1.png" width="700" align="center" />
</div>
<br>

Other implementations, such as [Binance's](https://www.binance.com/en/proof-of-reserves), achieve Proof of Liability by splitting users' balances across the leaves of a [Merkle Tree](https://en.wikipedia.org/wiki/Merkle_tree). The exchange would publish the Merkle Tree itself together with the total sum of liabilities. Each user can verify being included in the Merkle Tree. Since the individual users' balances are not public (only a hash is visible to the outside) users cannot independently sum the balances to calculate the total liabilities. This process is performed by an auditor that customers need to trust. 

This Proof of Liability implementation is partially **secret** but sacrifices the **credibility** of the proof as it requires auditors' overisight.

This solution is only partially private because critical information are leaked in the process such as the total number of users of Binance or the amount of liabilities that Binance, which are the sum of the balances of its users. 

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-2.png" width="700" align="center" />
</div>
<br>

## What

zkPOS makes use of [zkSNARKs](https://minaprotocol.com/blog/what-are-zk-snarks) to let Centralized Exchanges (CEX) generate a **credible** and **secret** Proof of Solvency. 

**Credbile** means that user can verify the Solvency of a CEX independently and without having to trust any auditor.
**Secret** means without leaking to the public any user's private data or other Business Intelligence Data.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-3.png" width="700" align="center" />
</div>
<br>

Ownership of assets is achieved by signing a specific message from their wallet. From that, it is possible to extract the exact amount of assets controlled by the CEX, at that specific time. 

The Liabilities of an Exchange are the sum of funds owned by the customers operating on the exchange. This balances are computed inside a Merkle Tree. This Merkle Tree is private and gets never shared to the customers.

zkPOS is a set of APIs that allows an exchange to plug in their database and generate a zk Proof to be shared with their users. 

Starting from the proof, a user can verify that they have been included (with their correct balance) in the computation of the Liabilities of the Exchange and that the total amount of Liabilities is less than its controlled Assets. 

The rule is simple: if enough users request a Proof of Liability and they can all verify it, it becomes evident that the Exchange is not lying or understating its liabilities. If just one user cannot verify the proof, the Exchange is lying 
about its actual liabilities. 

The proof can be fastly verifiable and doesn't leak any data about the number of users that operate on the exchange, the funds owned by other users or even the total amount of liabilities of the exchange itself! 

### Demo it

Test it out with our [CLI](https://github.com/pan-y-tomate/zk-pos-cli)

### How - Infrastructure

- [pyt-merkle-sum-tree](https://github.com/orgs/pan-y-tomate/repositories) is a TypeScript library where the entries of the exchange (`username -> balance`) are added to a Merkle Sum Tree data structure. The total sum of the leaves represents the total liabilities of a CEX.
- [pyt-circuits](https://github.com/pan-y-tomate/pyt-circuits) contains the circuits (written in circom) enforcing the rules that the Exchange must abide by to generate its Proof of Solvency

### How - Dev Tooling

- [pyt-pos](https://github.com/pan-y-tomate/pyt-pos) is a TypeScript Library to generate and verify pan-y-tomate Proof of Solvency

## User Flow

The flow of the zkPOS system is the following:

1. The Exchange generates and publishes its Proof of Assets by proving ownership of a particular address (for example by signing a specific message)
2. The Exchange extracts all the users' entries from their database (`username -> balance`) and adds them to the SMST
3. The Exchange publishes the Root of the SMST computed inside the SMST to a Public Bulletin Board 
4. Alice asks the Exchange for a Proof of Solvency
5. The Exchange generates a (zk) Proof of Solvency for Alice and sends it to her. This is the proof that Alice (and her correct balance) were included in the SMST and the total sum of the liabilities is less or equal to the Assets controlled by the Exchange.
6. The Exchange hands the Proof of Solvency (and the Public Signals used as inputs) to Alice
7. Alice verifies the Proof of Liability 
8. Alice executes further verification on the Public Signals used as input. The leaf must match Alice's username and balance. The assets must match the one published in the Proof of Assets (step 1). The root must match the one published on the Public Bulletin Board.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/zk POS.png" width="500" align="center" />
</div>
<br>

### Who

- [Enrico Bottazzi](https://github.com/enricobottazzi), Developer and Technical Writer @[Iden3](https://iden3.io/) @[PolygonID](https://polygon.technology/polygon-id)

### What's next - Phase 1

- [ ] privacy, research ways to let the exchange hide their proof of assets while verifying it inside a zk proof
- [ ] efficiency, explore further zk backend such as HALO2 to increase the proving time 
- [ ] UX, create a friendly user experience and make it understandable to users
- [ ] Add support to on-chain verification
- [ ] Explore on-chain mechanisms to incentivize users to verify their proof of solvency
- [ ] Perform security audit on the zk circuit 
- [ ] Support balances in different currencies

### What's next - Phase 2

Phase 2 of pan-y-tomate will bring the same model to a bigger set of applications that struggle with the same problem: lack of trust from their users.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/phase2-pyt.png" align="center" />
</div>
<br>

### Further Resources

- [More on zkSNARKs](https://www.youtube.com/watch?v=lwbt-a8PLRw)
- [Having a safe CEX: proof of solvency and beyond - Vitalik Buterin](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html)
- [SNARKed Merkle Sum Tree: A Practical Proof-of-Solvency Protocol based on Vitalik’s Proposal - Eth Research](https://ethresear.ch/t/snarked-merkle-sum-tree-a-practical-proof-of-solvency-protocol-based-on-vitaliks-proposal/14405)
- [ZK Podcast - Proof of Solvency with Kostas Chalkias](https://zeroknowledge.fm/257-2/)
- [Generalized Proof of Liabilities - Kostas Chalkias Yan Ji](https://eprint.iacr.org/2021/1350.pdf)







