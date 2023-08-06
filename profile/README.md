# Summa

Summa is a zero-knowledge proof of solvency solution. Developed within [PSE](https://appliedzkp.org/) and [Ethereum Foundation](https://ethereum.foundation/)

<p align="center">
  <img src="https://hackmd.io/_uploads/SkdQSk5H3.png" width="100" height="100" alt="Image 1">
  &nbsp; &nbsp; &nbsp;
  <img src="https://hackmd.io/_uploads/HyuEr1cB2.png" width="180" height="80" alt="Image 2">
  &nbsp; &nbsp; &nbsp;
  <img src="https://hackmd.io/_uploads/HyZ0rycS2.png" width="70" height="100" alt="Image 3">
</p>

Summa lets centralized exchanges (CEXes) generate proof of solvency while keeping their sensitive data private. Proof of Solvency means that the assets under the control of the CEX are greater than its liabilities.

$Assets \geq  Liabilities$

Or, in simpler terms, the CEX has the assets to cover all the balances of its users so that they can withdraw their funds at any time.

## Background on zk and Proof of Solvency 

- [More on zkSNARKs](https://www.youtube.com/watch?v=lwbt-a8PLRw)
- [Having a safe CEX: proof of solvency and beyond - Vitalik Buterin](https://vitalik.ca/general/2022/11/19/proof_of_solvency.html)
- [SNARKed Merkle Sum Tree: A Practical Proof-of-Solvency Protocol based on Vitalik’s Proposal - Eth Research](https://ethresear.ch/t/snarked-merkle-sum-tree-a-practical-proof-of-solvency-protocol-based-on-vitaliks-proposal/14405)
- [ZK Podcast - Proof of Solvency with Kostas Chalkias](https://zeroknowledge.fm/257-2/)
- [Generalized Proof of Liabilities - Kostas Chalkias Yan Ji](https://eprint.iacr.org/2021/1350.pdf)
- [How can Cryptographic Proofs Provide a Guarantee of Financial Solvency? - Eli Ben Sasson](https://www.coincenter.org/how-can-cryptographic-proofs-provide-a-guarantee-of-financial-solvency/)

## Public Presentations 

- Ho Chi Min City - 0xParc/PSE Residency - 6/4/23 - [Slides](https://docs.google.com/presentation/d/1RidLMZTsAY0i0ENZYasL1LfsLngataGIGxaIJCqgGng/edit?usp=sharing), [Recording](https://youtu.be/F-Q31AwuTCU)
- Barcelona - 15/4/23 - [Slides](https://docs.google.com/presentation/d/1Qop5xDCThW5eIB_tY9Nd_5FwDn5DKSJj75nv3nwUaWU/edit?usp=sharing)
- Naples - Spaghetteth - 25/5/23 - [Slides](https://docs.google.com/presentation/d/1zJz412Ca0rkTuVg8jy3OBvZGOsxYL0EZXhYfBUK2vwo/edit?usp=sharing), [Recording](https://youtu.be/gdGqGC31_yU)
- Paris - ETHCC - 18/7/23 - [Slides](https://docs.google.com/presentation/d/1wpb9Up64Q61odrZ3ujTNGv3w_1hANJq5uGP5wdZaRfw/edit?usp=sharing), [Recording](https://www.youtube.com/live/cG-Y4-6kp68?feature=share)
- Barcelona - ZCON4 - 1/8/23 - [Slides](https://docs.google.com/presentation/d/1dmQhlFnrbrijYTRxCVDj6aKFvAVUZsdQ7NK0Mr9kcW0/edit?usp=sharing), [Recording](https://youtu.be/P7w6LWXkwns)

## Why Zero Knowledge?

Zero Knowledge Proof, particularly zkSNARKs, allows the prover to achieve **computational integrity guarantee**. Namely, given a proof π and a computation whose rules are known by everyone, π tells that the output is the result from running the computation on certain inputs according to the rules. On top of that, zkSNARKs allow the prover to keep certain inputs private. Summa leverages zkSNARKs to:

- Remove reliances on trusted third parties such as auditors
- Let the CEX prove to their users that they are solvent without revealing sensitive information such as:
  - Individual user information such as their balances and usernames
  - The total number of users
  - The total amount of liabilities

## Building Blocks 

- [Merkle Sum Tree](https://github.com/summa-dev/summa-solvency/tree/master/zk_prover/src/merkle_sum_tree) is the core data structure used by the CEX to store the data related to users' liabilities. The root of the MST contains a commitment to the state of the total CEX's liabilities per currency at a specific timestamp.
- [ZK Circuits](https://github.com/summa-dev/summa-solvency/tree/master/zk_prover/src/circuits) contain the two core circuits of Summa:
  - `Solvency`, allows a CEX to prove that its assets are greater than its liabilities
  - `MstInclusion`, allows a CEX to prove to a user that they have been accounted for correctly in the liabilities
- [Summa Smart Contract](https://github.com/summa-dev/summa-solvency/tree/master/contracts) allows a CEX to verify its proof of solvency on-chain and provide common knowledge of the state of its assets and liabilities.

## Protocol Flow

<div align="center">
<img src="https://github.com/summa-dev/.github/blob/main/profile/summa_uml.png" width="500" align="center" />
</div>
<br>

- `1. Build Snapshot`
  
  In order for the process to start, the CEX needs to build a `Snapshot` which is a data container for the CEX liabilities and the CEX wallets.

  The CEX needs to provide two csv files, one for the liabilities and one for the accounts controlled by the CEX.

  **Liabilites**
    
  The Exchange extracts all the users' entries from their database (`username -> balanceEth, balanceBTC`) in a csv file and pass it to the `Snapshot`. Under the hood, a Merkle Sum Tree will be build starting from these entries.

  Note: for this example we are only gonna use ETH and BTC as currency, but this can be extended to any existing currency on any existing blockchain.
  
    | username | balanceEth | balanceBTC |
    | --- | --- | --- | 
    | alice | 3223 | 34 |
    | bob | 100234 | 0.1 |
    | carl | 42069 | 25 |
  
   
  This action is performed privately by the CEX; the tree is never shared with the public.
  
  This action doesn’t require auditing or oversight. Any malicious operation that the CEX can perform here, such as:
    
    1. adding users with negative balances 
    2. excluding users
    3. understating users’ balances
  
  will be detected when the π of Inclusion is handed over to individual users.

  **Account Ownership**

    To prove control over certain accounts, the CEX will group its wallets and sign a specific message (it's up to the CEX to choose the message) with each of them:

    *ETH*
    
    | pubKey   | signature        |
    | -------- | ----------       |
    | 0x123    | 0xdaf23784       |
    | 0x456    | 0xfff99999       |
    | 0x789    | 0xabc00111       |
    
    *BTC*

    | pubKey                             | signature  |
    | ---------------------------------- | ---------- |
    | 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa | 0x9543aaaa |
    | 1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2 | 0x54432ffa |
    | 3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy | 0xdddaaaa9 |

  These data are provided to `Snapshot` through a csv file and packed into `AccountOwnershipProof`. At this point, no check check on the validity of the signatures is being performed. A malicious CEX pretending to control accounts that are not theirs will be detected in the next step. 

- `2. π of Account Ownership`

  The `Summa` Smart Contract (it can be deployed on any EVM-compatible chain) will receive a π of Account Ownership and check the validity of each signature to verify that the CEX controls such wallets. On successful verification, these accounts are written to the smart contract as `cexAddresses`

- `3. π of Solvency`

  π of Solvency means proving that $Assets \geq  Liabilities$. The CEX will generate a zk proof that the amount of assets controlled by the account they proved ownership of (as a result of step 2) is greater than the liabilities of the CEX. The expression must hold true for each asset. For example, the liabilities denominated in ETH must be covered at least 1:1 by assets denominated ETH. The CEX cannot cover the liabilities denominated in ETH with assets denominated in another currency.
  
  The public inputs of the circuit are `asset_sum_eth`, `asset_sum_btc` and `mst_root`. 
  
    The verification of the π of Solvency happens programmatically inside the smart contract. It involves verifying that: 

    - The cryptographic proof is valid
    - Fetching the balances of the `cexAddresses` for BTC and ETH. This happens using a mix of on-chain available data and oracles, in the case these balances do not exist on Ethereum
    - Verify that `asset_sum_eth` and `asset_sum_btc`, public inputs of the circuit, match the fetched balances of the CEX
  
  If the proof verifies, an event will be emitted by streaming `mst_root` to the public. This will be required in the next step for user-side proof verification.
  
  Note that no data about the liabilities of the CEX is leaked here.

- `4. π of Inclusion`

  At this point, the CEX has proven its solvency, but the liabilities could have been built maliciously. For example, a CEX might have arbitrarily excluded "whales" from the liabilities account to achieve a dishonest proof of solvency.
  
  π of Inclusion means proving that a user, identified by its username and balances denominated in different currencies, has accounted for correctly in the liabilities. In practice, it means generating a zk proof that an entry `username -> balanceEth, balanceBTC, ...` is included in a Merkle sum tree with a root equal to the one published on-chain in the previous step. The π doesn't reveal any information about the balances of any other users or even the aggregated liabilities of the CEX.

  If any user finds out that they haven't been included in the MST, or have been included with an understated balance, a warning related to the potential non-solvency of the CEX has to be raised.

  The proofs are generated by the CEX and shared with each user.

  The verification of the π of Inclusion happens locally on the user device. It involves verifying that: 

    - The cryptographic proof is valid
    - The `leafHash`, public input of the SNARK, matches the combination `H(username, balanceEth, balanceBTC)` of the user
    - The `rootHash` used as public input to the SNARK matches the one published on-chain in step 3.
         
    The rule is simple: if enough users request a Proof of Liability and they can all verify it, it becomes evident that the Exchange is not lying or understating its liabilities. If just one user cannot verify their π of Inclusion, the Exchange lies about its liabilities.

## Further Remarks

- The zkSNARK backend used by Summa is [Halo2 - PSE Fork](https://github.com/privacy-scaling-explorations/halo2). Such prover backend allows us to achieve great performance ([benchmarks](https://github.com/summa-dev/summa-solvency/tree/master/zk_prover#current-benchmarks)), be memory safe (it's implemented in Rust) and to rely on a universal trusted setup rather than a circuit-specific trusted setup. This last feature is given by the fact that Halo2 uses a PLONK prover in the backend, compared to other prover systems such as GROTH16, which relies on a circuit-specific trusted setup, increasing trust assumptions.
- Even though the smart contract is deployed on a EVM chain, the proof of solvency protocol supports any currency
- The Merkle Sum Tree supports the multi-currency feature, so liabilities denominated in any number of assets can be accumulated in a single merkle sum tree.
- Further privacy features (privacy of the total amount of assets owned by the CEX and on addresses of the wallets controlled by the CEX) will be enabled as part of the next release



