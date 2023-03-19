# summa

summa is a zero knowledge proof of solvency solution.

Proof of Solvency is achieved by proving that the assets under control of the exchange are greater than the liabilities. Or in simple terms, that the exchange have the assets to cover all the balances of its users so that they can withdraw their funds at any time.

## Background on zk and Proof of Solvency 

- [More on zkSNARKs](https://www.youtube.com/watch?v=lwbt-a8PLRw)
- [Having a safe CEX: proof of solvency and beyond - Vitalik Buterin](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html)
- [SNARKed Merkle Sum Tree: A Practical Proof-of-Solvency Protocol based on Vitalik’s Proposal - Eth Research](https://ethresear.ch/t/snarked-merkle-sum-tree-a-practical-proof-of-solvency-protocol-based-on-vitaliks-proposal/14405)
- [ZK Podcast - Proof of Solvency with Kostas Chalkias](https://zeroknowledge.fm/257-2/)
- [Generalized Proof of Liabilities - Kostas Chalkias Yan Ji](https://eprint.iacr.org/2021/1350.pdf)
- [How can Cryptographic Proofs Provide a Guarantee of Financial Solvency? - Eli Ben Sasson](https://www.coincenter.org/how-can-cryptographic-proofs-provide-a-guarantee-of-financial-solvency/)

## Zero Knowledge of What?

By using summa, centralized exchanges (CEXs) can prove to their users that they are solvent, namely that this expression holds true:

$Assets \geq  Liabilities$

without revealing critical information about their business, such as:

- Indivudual user information such as their balances and usernames
- The total number of users
- The total amount of liabilities
- The total amount of assets (WIP)
- The addresses of the wallets controlled by the CEX that hold the assets (WIP)

---

[**👷Join the Builders Discussion**](https://github.com/orgs/summa-dev/discussions)

---

## Building Blocks 

- [pyt-merkle-sum-tree](https://github.com/summa-dev/pyt-merkle-sum-tree) is a TypeScript library to create Merkle Sum Trees starting from `username -> balance` entries. The root of the tree contains an hash committment of CEX's state together with the sum of all the entries, representing the total liabilities of a CEX.
- [pyt-circuits](https://github.com/summa-dev/pyt-circuits) contains the circuits (written in circom) enforcing the rules that the Exchange must abide by to generate a Proof of Solvency for a specific user.
- [pyt-pos](https://github.com/summa-dev/pyt-pos) is a TypeScript Library to generate and verify Proof of Solvency. The library contains two main classes:

    - `Prover` contains the core APis to let CEXs provide credible Proof Of Solvency to its users.
    The proof doesn't reveal any information such as the total balances of each user, the number of users and the total amount of liabilities of the exchange.

    - `userVerifier` is a class that contains the core APIs to let a user verify the proof that has been provided them by the exchange.

## Protocol Flow

<div align="center">
<img src="https://github.com/summa-dev/.github/blob/main/profile/pyt-flow.png" width="500" align="center" />
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
    
    The Exchange extracts all the users' entries from their database (`username -> balance`) and adds them to the MST. 
    
    **This process is done privately by the exchange, the tree is never shared to the public**
    
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

    The APIs to generate the Merkle Sum Tree are available in [pyt-merkle-sum-tree](https://github.com/summa-dev/pyt-merkle-sum-tree)
    
- `3. Publish Tree Root Hash` (***not part of this specification***)
    
    The exchange has to publish the `rootHash` of the tree generated in step 2 to a Public Bulletin Board (Blockchain or Twitter for example). The `rootHash` represents the state of the Merkle Sum Tree. This action represents a commitment to that state. 
    
- `4. Generate Proofs`
    
    The exchange needs to generate a proof for each user following the rules encoded in [these circuits](https://github.com/summa-dev/pyt-circuits). Each proof is specific to a user. This proof demonstrates that the user has been included in the Merkle Tree that computes the total liabilities and that the total amount of liabilities is less than the total amount of assets (as made available from step 1).

    ```
    const proof = await Prover.generateProofForUser(userIndexInDB)
    ``` 

    The APIs to generate proofs are available in [pyt-pos](https://github.com/summa-dev/pyt-pos).
   
- `5. Share Proof` (***not part of this specification***)
    
    The exchange shares a proof to each of its users. It is important that the proof is the action of sharing the proof is initiated by the exchange rather than actively queried by the user. In the second scenario, the exchange gets to know which are the users that are interested in verifying the solvency of the exchange and which ones are not. The latter ones can be seen as “lazy” users that will likely not get involve any proof verification in the future. With this information the exchange can exclude this users from the liabilities computation in the future. By sharing the proofs to each user (for example via email), the exchange doesn't get to know which users have verified their proof. 
    
- `6. Verify proof`

    The user has to locally verify the proof that has been shared to them by the Exchange. Starting from the proof, a user can verify that they have been included (with their correct balance) in the computation of the Liabilities of the Exchange and that the total amount of Liabilities is less than its controlled Assets. 
    
    It involves verifying that: 

    - The cryptographic proof is valid
    - The `leafHash`, public output of the SNARK, matches the combination `H(usernameToBigInt, balance)` of the user
    - The `assetsSum` used as public input to the SNARK matches the one published in step 1
    - The `rootHash` used as public input to the SNARK matches the one published in step 3.


    The rule is simple: if enough users request a Proof of Liability and they can all verify it, it becomes evident that the Exchange is not lying or understating its liabilities. If just one user cannot verify the proof, the Exchange is lying about its actual liabilities. 








