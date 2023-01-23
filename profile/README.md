# Pan y Tomate

## Problem Statement 

People don't trust Centralized Exchanges (CEXs)

## Solution 

Pan y Tomate is an open source tool that employs cryptographic to let CEXs achieve credibility maintaining secrecy over their Business Information.

**Credibility** - The validity of the proof fully ensures that the rules are met. Each user of the exchange can verify the validity of the proof without having to rely or trust auditors or other external parties.

**Secrecy** - The Business Information of the exchange, such as the total balances of each users, the number of users and total amount of liabilities are not revealed to the public.

### [👷Join the Builders Discussion](https://github.com/orgs/pan-y-tomate/discussions)

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

### [DEMO IT](https://github.com/pan-y-tomate/zk-pos-cli)

### How - Infrastructure

- [pyt-merkle-sum-tree](https://github.com/orgs/pan-y-tomate/repositories) is a TypeScript library where the entries of the exchange (`username -> balance`) are added to a Merkle Sum Tree data structure. The total sum of the leaves represents the total liabilities of a CEX.
- [pyt-circuits](https://github.com/pan-y-tomate/pyt-circuits) contains the circuits (written in circom) enforcing the rules that the Exchange must abide by to generate its Proof of Solvency

### How - Dev Tooling

- [pyt-pos](https://github.com/pan-y-tomate/pyt-pos) is a TypeScript Library to generate and verify pan-y-tomate Proof of Solvency

### User Flow

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/zk-pos-flow.png" width="500" align="center" />
</div>
<br>

The flow of is the following:

- `1. Proof of Assets`
    
    The exchange requires to prove ownerhsip of an address with a certain amount of assets. In order to do so, the exchange needs to sign a certain message, for example H(`poSolID, address`) where 
    
    - `poSolID` is a sequential nonce unique for each Proof Of Solvency process.
    - `address` is the address controlled by the exchange
    
    By signing this message, the Exchange is authenticating the following information: “I, Exchange, am attesting that I own `address` and this information will be used as part of the Proof Of Solvency `poSolID` ”.
    
    ********************************Attack Vector:******************************** At this point the exchange can bribe someone (ideally owning a large amount of assets) to sign the message on its behalf
    
- `2. Add DB to Merkle Sum Tree`
    
    The Exchange extracts all the users' entries from their database (`username -> balance`) and adds them to the MST according to these [rules](https://github.com/pan-y-tomate/pyt-merkle-sum-tree).  This process is done privately by the exchange, the tree is never shared to the public
    
    The exchange sorts its users by their username and balance (of a certain assets) and add them to the Merkle Sum Tree.
    
    | username | balance |
    | --- | --- |
    | alice | 3223 |
    | bob | 100234 |
    | carl | 42069 |
    
    The username is first parsed into its utf8 bytes representation and then converted to BigInt before getting added to the Sparse Merkle Tree.
    
    It is important here to use a username or a value that maps to a unique information about a specific user to avoid that a malicious exchange would reuse a entry for two different users.
    
    This action doesn’t require auditing or oversight. Any malicious operation that the exchange can perform here, such as 
    
    1. adding users with negative balances 
    2. excluding users
    3. understating users’ balances 
    
    will be detected either in the proof generation phase (1) or in the proof verification phase (2 and 3).
    
- `3. Publish Tree Root Hash`
    
    The exchange has to publish the `rootHash` of the tree generated in step 2 to a Public Bulletin Board (Blockchain or twitter for example). The `rootHash` represents the state of the Merkle Sum Tree. This action represents a commitment to that state. 
    
- `4. Generate Proofs`
    
    The exchange needs to generate a proof for each user following [these rules](https://github.com/pan-y-tomate/pyt-circuits). Each proof is specific to a user.
    
    The exchange generates the Proof of Solvency inside a zkSNARK. The SNARK takes the following inputs:
    
    - `rootHash` is the hash of the merkle sum tree published in step 3 (public input)
    - `leafUsername` is the username of the user whose proof is generated for in format. The username is first parsed into its utf8 bytes representation and then converted to BigInt to get to `leafUsername`
    - `pathIndices, siblingHashes and siblingSums` represents the Merkle proof of inclusion of an the user’s leaf inside the Merkle Sum Tree
    - `assetsSum`  are the total assets owned by the exchange as declared in step 1 (public input)

    <div align="center">
    <img src="https://github.com/pan-y-tomate/pyt-circuits/blob/main/imgs/pos.png" width="700" align="center" />
    </div>
    <br>
        
    The SNARK performs the following operations:
    
    - Perform the posiedon hash on the entry (`leafUsername, leafSum`) to get the `leafHash` (public output of the circuit)
    - Build the Merkle Sum Tree starting from the leaf and its Merkle Proof to get to the `computedRootHash` and the `computedRootSum`
    - Verify that `computedRootSum` is equal to the `rootHash` provided as input in order to prove that the operation has been performed against the committed tree with Root Hash equal to `rootHash`
    - Verify that `computedRootSum` is less or equal than `assetsSum` in order to prove solvency
- `5. Share Proof`
    
    The exchange shares a proof to each of its users. It is important that the proof is the action of sharing the proof is initiated by the exchange rather than actively queried by the user. In the second scenario, the exchange gets to know which are the users that are interested in verifying the solvency of the exchange and which ones are not. The latter ones can be seen as “lazy” users that will likely not get involve any proof verification in the future. With this information the exchange can exclude this users from the liabilities computation in the future. Sharing the proof (for example via email), the exchange won’t know which users have verified their proof. 
    
- `6. Verify zk Posol`

    The user has to locally verify the proof that has been shared to them by the Exchange. It involves verifying that: 

    - The cryptographic proof is valid
    - The `assetsSum` used as public input to the SNARK matches the one published in step 1
    - The `rootHash` used as public input to the SNARK matches the one published in step 3
    - The `leafHash` , public output of the SNARK matches the combination `H(usernameToBigInt, balance)` of the user

### Who

- [Enrico Bottazzi](https://github.com/enricobottazzi), Developer and Technical Writer @[Iden3](https://iden3.io/) @[PolygonID](https://polygon.technology/polygon-id)

### Phase 2 - Beyond Proof of Solvency 
Phase 2 of pan-y-tomate will bring the same model to a bigger set of applications that struggle with the same problem: lack of trust from their users.

<div align="center">
<img src="https://github.com/pan-y-tomate/.github/blob/main/profile/phase2-pyt.png" height="700" align="center" />
</div>
<br>

### Further Resources

- [More on zkSNARKs](https://www.youtube.com/watch?v=lwbt-a8PLRw)
- [Having a safe CEX: proof of solvency and beyond - Vitalik Buterin](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html)
- [SNARKed Merkle Sum Tree: A Practical Proof-of-Solvency Protocol based on Vitalik’s Proposal - Eth Research](https://ethresear.ch/t/snarked-merkle-sum-tree-a-practical-proof-of-solvency-protocol-based-on-vitaliks-proposal/14405)
- [ZK Podcast - Proof of Solvency with Kostas Chalkias](https://zeroknowledge.fm/257-2/)
- [Generalized Proof of Liabilities - Kostas Chalkias Yan Ji](https://eprint.iacr.org/2021/1350.pdf)







