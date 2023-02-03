# pan-y-tomate

## Problem Statement 

People don't trust Centralized Exchanges (CEXs)

## Solution 

pan-y-tomate is an open source tool that employs cryptography to let CEXs create credible Proofs of Solvency (POS) while maintaining secrecy over their Business Information.

**Credibility** - The validity of the proof fully ensures that the rules are met. Each user of the exchange can verify the validity of the proof without having to rely or trust auditors or other external parties.

**Secrecy** - The Business Information of the exchange, such as the total balances of each users, the number of users and total amount of liabilities are not revealed to the public.

### [👷Join the Builders Discussion](https://github.com/orgs/pan-y-tomate/discussions) - [🕹️ Jump to the APIs](https://github.com/pan-y-tomate/pyt-pos)

## Why?

Since the FTX collapse, many exchanges are exploring [ways](https://niccarter.info/proof-of-reserves/) to demonstrate their solvency to their customers. [Proof of Solvency](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html) is achieved by proving that the assets under control of the exchange are greater than the liabilities. Or in simple terms, that the exchange have the assets to cover all the balances of its users so that they can withdraw their funds at any time.

$Assets \geq  Liabilities$ 

Proving ownership of assets is relatively easy to achieve: by signing a specific message from their wallet, an exchange can prove ownership of an account and, therefore, of the assets controlled by that account.

The liabilities are the sum of the balances of the customers operating on the exchange. The exchange owes this sum to its customers. The exchange must be able to cover its liabilities with its assets. 

Proving liabilities is tricker. 

The simplest solution would be opening up the database and disclosing every user-balance entry. Individual users can verify that they have been included in the list. By summing all the balances together and comparing the sum to the total assets of the exchange is possible to verify the solvency of the exchange.

This Proof of Liability implementation is **credible** in the eye of customers but sacrifices the **secrecy** of the exchange's (and its customers!)  data.

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

pan-y-tomate makes use of [zkSNARKs](https://minaprotocol.com/blog/what-are-zk-snarks) to let Centralized Exchanges (CEX) generate a **credible** and **secret** Proof of Solvency. 

**Credbile** means that user can verify the Solvency of a CEX independently and without having to trust any auditor.
**Secret** means without leaking to the public any user's private data or other Business Intelligence Data.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/tradeoff-3.png" width="700" align="center" />
</div>
<br>

Ownership of assets is achieved by signing a specific message from their wallet. From that, it is possible to extract the exact amount of assets controlled by the CEX, at that specific time. 

The Liabilities of an Exchange are the sum of funds owned by the customers operating on the exchange. These balances, together with the identifiers of the users, are computed inside a [Merkle Sum Tree](https://github.com/pan-y-tomate/pyt-merkle-sum-tree). This Merkle Tree is private and never gets shared to the public.

The Exchange then commits to the State of the Merkle Tree by publishing its root hash on a Public Bulletin Board.

The Merkle Tree is then used to generate a Proof of Solvency. In particular, the exchange generates a Proof of Solvency for each user. This proof demonstrates that the user has been included in the Merkle Tree that computes the total liabilities and that the total amount of liabilities is less than the total amount of assets.

Starting from the proof, a user can verify that they have been included (with their correct balance) in the computation of the Liabilities of the Exchange and that the total amount of Liabilities is less than its controlled Assets. 

The rule is simple: if enough users request a Proof of Liability and they can all verify it, it becomes evident that the Exchange is not lying or understating its liabilities. If just one user cannot verify the proof, the Exchange is lying about its actual liabilities. 

The proof can be fastly verifiable and doesn't leak any data about the number of users that operate on the exchange, the funds owned by other users or even the total amount of liabilities of the exchange itself! 


[pyt-pos](https://github.com/pan-y-tomate/pyt-pos) is a TypeScript Library to generate and verify pan-y-tomate Proof of Solvency is a set of APIs that allows an exchange to plug in their database and generate a zk Proof to be shared with their users. 


## How - Infrastructure

- [pyt-merkle-sum-tree](https://github.com/pan-y-tomate/pyt-merkle-sum-tree) is a TypeScript library to create Merkle Sum Trees starting from `username -> balance` entries. The root of the tree contains the sum of all the entries, representing the total liabilities of a CEX.
- [pyt-circuits](https://github.com/pan-y-tomate/pyt-circuits) contains the circuits (written in circom) enforcing the rules that the Exchange must abide by to generate a Proof of Solvency for a specific user.

## How - APIs

- [pyt-pos](https://github.com/pan-y-tomate/pyt-pos) is a TypeScript Library to generate and verify pan-y-tomate Proof of Solvency. The library contains two main classes:

    - `Prover` contains the core APis to let CEXs provide credible Proof Of Solvency to its users.
    The proof doesn't reveal any information such as the total balances of each user, the number of users and the total amount of liabilities of the exchange.

    - `userVerifier` is a class that contains the core APIs to let a user verify the proof that has been provided them by the exchange


## User Flow

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/pyt-flow.png" width="500" align="center" />
</div>
<br>

The flow of is the following:

- `1. Proof of Assets` (***not part of this specification***)
    
    The exchange is required to prove ownerhsip of an address with a certain amount of assets. In order to do so, the exchange needs to sign a certain message, for example H(`poSolID, address`) where 
    
    - `poSolID` is a sequential nonce unique for each Proof Of Solvency process.
    - `address` is the address controlled by the exchange
    
    By signing this message, the Exchange is authenticating the following information: “I, Exchange, am attesting that I own `address` and this information will be used as part of the Proof Of Solvency `poSolID` ”.
    
    ********************************Attack Vector:******************************** At this point the exchange can bribe someone (ideally owning a large amount of assets) to sign the message on its behalf
    
- `2. Build Merkle Sum Tree`
    
    The Exchange extracts all the users' entries from their database (`username -> balance`) and adds them to the MST. This process is done privately by the exchange, the tree is never shared to the public.
    
    The exchange sorts its users by their username and balance (of a certain assets) and add them to the Merkle Sum Tree.
    
    | username | balance |
    | --- | --- |
    | alice | 3223 |
    | bob | 100234 |
    | carl | 42069 |
    
    The username is first parsed into its utf8 bytes representation and then converted to BigInt before getting added to the Sparse Merkle Tree.

    ```
    const tree = new IncrementalMerkleSumTree(DB.csv)
    ``` 
    
    It is important here to use a username or a value that maps to a unique information about a specific user to avoid that a malicious exchange would reuse the same entry for two different users.
    
    This action doesn’t require auditing or oversight. Any malicious operation that the exchange can perform here, such as 
    
    1. adding users with negative balances 
    2. excluding users
    3. understating users’ balances 
    
    will be detected either in the proof generation phase (case 1) or in the proof verification phase (cases 2 and 3).

    The APIs to generate the Merkle Sum Tree are available in [pyt-merkle-sum-tree](https://github.com/pan-y-tomate/pyt-merkle-sum-tree)
    
- `3. Publish Tree Root Hash` (***not part of this specification***)
    
    The exchange has to publish the `rootHash` of the tree generated in step 2 to a Public Bulletin Board (Blockchain or Twitter for example). The `rootHash` represents the state of the Merkle Sum Tree. This action represents a commitment to that state. 
    
- `4. Generate Proofs`
    
    The exchange needs to generate a proof for each user following the rules encoded in [these circuits](https://github.com/pan-y-tomate/pyt-circuits). Each proof is specific to a user.

    ```
    const proof = await Prover.generateProofForUser(userIndexInDB)
    ``` 

    The APIs to generate proofs are available in [pyt-pos](https://github.com/pan-y-tomate/pyt-pos).
   
- `5. Share Proof` (***not part of this specification***)
    
    The exchange shares a proof to each of its users. It is important that the proof is the action of sharing the proof is initiated by the exchange rather than actively queried by the user. In the second scenario, the exchange gets to know which are the users that are interested in verifying the solvency of the exchange and which ones are not. The latter ones can be seen as “lazy” users that will likely not get involve any proof verification in the future. With this information the exchange can exclude this users from the liabilities computation in the future. By sharing the proofs to each user (for example via email), the exchange doesn't get to know which users have verified their proof. 
    
- `6. Verify proof`

    The user has to locally verify the proof that has been shared to them by the Exchange. It involves verifying that: 

    - The cryptographic proof is valid
    - The `leafHash`, public output of the SNARK, matches the combination `H(usernameToBigInt, balance)` of the user
    - The `assetsSum` used as public input to the SNARK matches the one published in step 1
    - The `rootHash` used as public input to the SNARK matches the one published in step 3

## Phase 2 - Beyond Proof of Solvency 
Phase 2 of pan-y-tomate will bring the same model to a bigger set of applications that struggle with the same problem: lack of trust from their users.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/phase2-pyt.png" height="700" align="center" />
</div>
<br>

## Further Resources

- [More on zkSNARKs](https://www.youtube.com/watch?v=lwbt-a8PLRw)
- [Having a safe CEX: proof of solvency and beyond - Vitalik Buterin](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html)
- [SNARKed Merkle Sum Tree: A Practical Proof-of-Solvency Protocol based on Vitalik’s Proposal - Eth Research](https://ethresear.ch/t/snarked-merkle-sum-tree-a-practical-proof-of-solvency-protocol-based-on-vitaliks-proposal/14405)
- [ZK Podcast - Proof of Solvency with Kostas Chalkias](https://zeroknowledge.fm/257-2/)
- [Generalized Proof of Liabilities - Kostas Chalkias Yan Ji](https://eprint.iacr.org/2021/1350.pdf)







