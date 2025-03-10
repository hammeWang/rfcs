---
Rfc: 10
Title: 0010-darwinia-cross-chain-nft-bridge-protocol
Status: Draft
Desc: Cross-chain NFT Bridge Protocol
---


# Cross-chain NFT Bridge Protocol			

### v 0.2.2



## I. Overview

The XClaim-based framework provides an idea for cross-chaining, but there are still many problems with NFT, including the Vault collateral design on the Backing Blockchain. This problem can be effectively solved by introducing a chain-Relay contract on the Backing Blockchain. This paper will design a cross-chain NFT solution and standard based on this improved cross-chain transit bridge solution.

**Keywords**: Blockchain, NFT, cross chain, multi-chain



## II. Background

### A. Research history

The emergence of Bitcoin [1] allows everyone to operate their own assets without any trust, as long as they have a private key. The entire Bitcoin system consists of a series of blocks that record their pre-order block hashes, maintaining the same decentralized global “book”.

After the emergence of Bitcoin, the rapid development of the blockchain followed the public chain supporting the smart contract - Ethereum [2], the public chain of PoS - EOS [3]. The outbreak of these public chains has brought prosperity to the entire token trading market.

The mainstream token trading/exchange method is still a centralized exchange, and the user's token is managed by the centralized exchange. Trust and maintenance costs are high and there is a constant threat of hacking.

In order to overcome the drawbacks of centralized exchanges, decentralized exchanges (DEX) began to emerge. Most decentralized exchanges only support token trading/conversion in a chain, such as ERC20[4], ERC721 token[5] on Ethereum. This achieves decentralization and reduction to some extent. The cost of trust (from believing in the organization to believing code), but the use of the scenario is very limited, and is also limited by the tps and transaction costs of the public chain.

Of course, there are also some decentralized exchanges that implement ACCS, allowing tokens to be cross-chain exchanged. They use the hashed timelock contracts (HTLCs) [6]. The advantages of HTLCs are the same as their shortcomings. HTLCs can achieve atomic exchange of cross-chain tokens without trust, which both decentralizes and extends the functionality of DEX on a single chain. Its disadvantage is that the cost is too high, and there are many restrictions: (i) all participants must keep the whole process online (ii) invalidation of the dust transaction (iii) usually lock time is longer. Such token cross-chain switching is both expensive and inefficient. In actual use, there are very few examples of the use of HTLCs.

In order to achieve a trust-free, low-cost, and efficient token cross-chain operation, the XClaim team proposed a cross claim scheme based on CBA. And in XClaim's paper, how XClaim does the following four things is done: Issue, Transfer, Swap and Redeem.

 The role of ensuring economic security in the XClaim system is called $vault$. If anyone wants to change the native token $b$ on chain $B$ to $$$ to $i(b)$, then you need $vault$ over-collateralized $i$ on chain $I$. In the redemption operation, if $vault$ has malicious behavior, the $vault$ mortgaged $i$ is penalized for redemption of the operation initiator. For additional details, see XClaim's paper [7].

So far, a reliable and achievable solution has been obtained for the cross-chain of Fungible tokens with better fluidity.

### B. Problems with the XClaim framework

There is a basic assumption in the XClaim scheme that the total value of the native token $b$ of the chain $B$ that is chain-locked is equal to the total value of $i(b)$ issued on $I$, in XClaim. It is called *symmetric*, which is $ |b| = |i(b)|$. The assumption is that XClaim has a natural dilemma in the NFT cross-chain:

- The irreplaceability of NFT. Because of the identifiability and irreplaceability of NFT, it is impossible for $vault$ to mortgage NFT $nft_b$ on chain $B$ on chain $I$.
- The value of NFT is difficult to assess. In XClaim, determining whether the $vault$ collateral is full/overdated is achieved through Oracle $O$. There is also a potential assumption that token $b$ and token $i$ can be evaluated correctly. Based on the current prosperous centralization and decentralized exchanges, this potential assumption can be basically met in the case of providing good liquidity. However, the market of the NFT exchange is not yet mature, and even the centralized exchange cannot truly reflect the market's price judgment on the NFT. How NFT is pricing itself is a problem.
- NFT pricing is not continuous and predictable. Even if a NFT has a transaction record in the market, there is a price, because the frequency of NFT is sold far lower than FT, even if the market liquidity is very good, the next transaction price of the NFT is Not continuous or predictable.

### C. Solutions and ideas

There are two ideas for the NFT cross-chain solution to solve the above problems. One is based on the NFT extension based on the XClaim framework and retaining the bridge pledge mechanism. The Haberberg mechanism is introduced to solve the NFT pricing problem. For detailed solutions, see [RFC- 0011: Using Harberger Tax to find price for XClaim Vault Collaterals](./0011-using-harberger-tax-to-find-price-for-xclaim-vault-collaterals.md). But this solution still can't solve it well. Due to the large change in the price of the NFT, the problem of insufficient collateral is caused.

Another idea is to introduce the chainRelay scheme in the Backing Blockchain to protect the assets of the endorsement, so that the pledge mechanism is no longer needed, which is referred to as [RFC-0012: XClaim using two chainRelay model] (./0012-xclaim- Using-two-chainrelay-model.md), the detailed introduction will not be described in detail in this article, this article will focus on this improved cross-chain transfer bridge solution, design a cross-chain NFT standard, and in the multi-chain cross-span In this case, a cross-chain protocol with lower cost and scalability is proposed.



Among them, in [RFC-0012: Darwinia Bridge Core: Interoperation in ChainRelay Enabled Blockchains] (./0012-darwinia-bridge-core-interoperation-in-chainrelay-enabled-blockchains.md) VA, we introduced Darwinia Bridge Core Model to optimize the number of chainRelays in a blockchain network topology. This article will be based on the Darwinia Bridge Hub and will be refined for NFT-specific issues.

![chain-relay-framework](https://tva1.sinaimg.cn/large/006y8mN6ly1g7fe8rjjzvj30kb0bfgmc.jpg)

## III. NFT in Darwinia Bridge Core

The difficulty of NFT cross-chain operation is that different public chains have their own NFT standards, and even the NFT token ids on different public chains are not equal in length. When NFT crosses different public chains, it will inevitably experience token id. Conversion. How to avoid the identifiability of NFT in the process of cross-chain is a proposition worth studying.

When designing NFT flow logic in Bridge Core, we want to solve the following three problems:

- Preserving the NFT's cross-chain flow path/history without losing the identifiability of the NFT;
- Calculate and verify decoupling with higher processing speed;
- Implement additional functions, such as NFT to complete decomposition, merging, etc. while cross-chaining;

To this end, we chose to use the extended UTXO model as a storage/state flow unit, which we call UNFO (Unspent Non-Fungible token Output).

The NFT in the intermediate state within Bridge Core is marked as $nft_{BC}^{X,n}$ above, indicating that there is an NFT to be issued/locked in the corresponding chain $X$. 

These intermediate states of NFT in Bridge Core are marked as UNFO (Unspent Non-Fungible Output). The idea stems from UTXO, when an UNFO is destroyed, it means that a new UNFO will be generated at the same time.

### A. UNFO structure

UNFO structure:

```rust
Struct UNFO {
  Pub local_id, // chain_id + smart_cotnract_id + token_id
  Pub global_id,
  Pub phase, // current phase
  Pub lock_script, // eg ownership or state management
}
```

- **local_id**: indicates that the UNFO corresponds to *token_id* in a *smart_contract_id* on an external blockchain *chain_id*
- **global_id**: indicates that UNFO is globally unique in the scope of Bridge Core and all boquelous blockchains
- **phase**: Indicates the stage in which the UNFO is in the process of cross-chaining. such as:
  - 1: The NFT on the UNFO corresponding blockchain *chain_id* is locked/destroyed; the cross-chain process is in the intermediate state;
  - 2: The NFT on the UNFO corresponding blockchain *chain_id* is pending/issued; the cross-chain process is nearing completion / completed
- **lock_script**: for more complex logic, fine-grained control scripts, maintaining UNFO's scalability



### B. UNFO conversion

When the destruction of one UNFO means the creation of another UNFO, if we trace the history of UNFO's destruction creation, we can trace back the entire cross-chain history of an NFT, which helps to achieve the NFT's recognizability to a certain extent;

Each UNFO can only be destroyed once, which makes it unnecessary to verify before the calculation, thus improving the processing speed;

Just like Bitcoin's UTXO, there can be more than one Input and Output. This feature allows the NFT to perform some extended functions, such as NFT splitting and merging, in the process of cross-chaining.

NFT has always been more useful than FT. For example, in games, the NFT as a prop is required to be detachable, synthesizable, and scalable. For this reason, many NFT standards, such as ERC721 and ERC1155, have been extended. ERC721X and so on. The more standards, the more difficult it is to be widely used.

If some of the common requirements can be implemented at the same time across the chain, the number and redundancy of standards can be effectively reduced, and to a certain extent, it is more conducive to achieve a unified standard.



When an UNFO is produced, it must be satisfied:

- Provide a lock record for the corresponding NFT of *backing blockchain*;
- Another UNFO is destroyed
  - Condition: The GID of the destroyed and produced UNFO must be the same

<img src="https://tva1.sinaimg.cn/large/006y8mN6ly1g7fe8skd28j30hj06z0sz.jpg" alt="0010-UNFO-transform" style="zoom:50%;" />





### C. Bridge Core Internal Structure



<img src="https://tva1.sinaimg.cn/large/006y8mN6ly1g7fe8qjwd9j30fe0gh3zg.jpg" alt="0010-framework-of-bridge-core" style="zoom:50%;" />



### D. Component definition

- *Issuing Smart Contract*, $iSC_N$: indicates the asset issuance contract on chain *N*;
- *Backing Smart Contract*, $bSC_N$ : indicates an asset lockout contract on chain $N$;
- *Verifying Smart Contract*, $vSC_N$ : Indicates the asset issuance contract/module responsible for verifying transactions on chain *N* on Bridge Core;
- *NFT identifier*, $nft_B^{x,n}$, representing the NFT identified as $n$ in the contract $x$ on the chain $B$
  - *NFT identifier in Bridge Core*, $nft_{BC}^{B, n}$ , which means on Bridge Core, and mirrors $nft_B^{x,n}$ on chain $B$
- *NFT identifier*, $nft_I^{x',n'}$, representing the NFT newly added on chain $I$ after the chain is identified as $n'$ in contract $x'$
  - *NFT identifier in Bridge Core*, $nft_{BC}^{I, n'}$ , which means on Bridge Core, and $nft_I^{x',n'}$ on chain $I$ are mirror images of each other
- *Locking Transaction* , $T_{B}^{lock}$, the transaction that locks the NFT in $bSC_B$ on chain *B*
- *Redeem Transaction* , $T_I^{redeem}$, the transaction that locks the NFT in $bSC_I$ on chain *I*
- *Extrinsic Issue*, $EX_{issue}$ , the issue of the issue on Bridge Core 
- *Extrinsic redeem*, $EX_{redeem}$ , redeem trading on Bridge Core 
- *Global NFT identifier*, $GID$

participants:

- *validator*, maintains the participants of the Bridge Core;

### E. Preliminary implementation plan

The scenario is the same as described in Chapter II. Still need to implement three kinds of protocols: *Issue, Transfer, Redeem*. Also to simplify the model, the details of the fee will not be discussed here.

#### Protocol: Issue

(i) ***Lock ***. *requester* locks the NFT asset $nft_B^{n,x}$ on chain $B$ in $bSC_B$ and declares the destination public chain *I* and its own address on chain $I$; One step will generate the transaction $T_B^{lock}$

(ii) *** Issued on Bridge Core***. *requester* Submits the locked transaction $T_B^{lock}$ to Bridge Core. After the corresponding chain relay is verified, $vSC_B$ is triggered, in $vSC_B$:

- Generate new $GID$ and $nft_{BC}^{B,n}$ , record the relationship between $GID$ and $nft_{BC}^{B,n}$
- and trigger $vSC_I$ 

In $vSC_I$:

- Destroy $nft_{BC}^{B,n}$, issue $nft_{BC}^{I,?}$, $issue\_{ex}(\ GID,\ address\_on\_I) \rightarrow EX_ {issue}$

(iii) *** issuance ***. *requester* Submit $EX_{issue}$ to chain $I$ , after the chain relay on chain $I$ is verified, a new NFT will be added to $iSC_I$: $nft_I^{x', n '}$, and record the relationship between $GID$ and $nft_I^{x', n'}$, and pass ownership to *requester* address on chain *I*

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g7sznhszi8j30pz0elabd.jpg" alt="chain-relay-framework-1" style="zoom:50%;" />

#### Protocol: Transfer

(i) *** Transfer ***. *sender* Put $nft_I^{x',n'}$ on $I$ in $iSC_I$ and transfer ownership to *receiver*, refer to ERC721.

(ii) *** Witness ***. When $nft_I^{x',n'}$ has transferred ownership in $iSC_I$, both $iSC_I$ and $bSC_I$ should be aware. At this point, when *sender* wants to redeem $nft_I^{x',n'}$, it needs to lock it in $bSC_I$ first, and $bSC_I$ will not allow the operation to succeed.

#### Protocol: Redeem

(i) ***Lock ***. *redeemer* locks the NFT asset $nft_I^{x', n'}$ on chain $I$ after $bSC_I$ (if there is a corresponding GID, declare $GID$ when locked) and declare the destination public The chain chain $B$ and its own address on the chain $B$; $bSC_I$ will atomically confirm the correctness of $GUID$ in $iSC_I$. This step will generate the transaction $T_I^{redeem}$. $lock\_I(nft\_id\_on\_I,\GID,\ address\_on\_B) \rightarrow T_I^{redeem}$

(ii) *** Unlock *** on Bridge Core. *redeemer* Submit $T_I^{redeem}$ to $vSC_I$ and verify it in the chain relay, it will be in $vSC_I$:

- Record the correspondence between $GUID$ and $nft_I^{x', n'}$,
- Determine the destination public chain and trigger the corresponding $vSC_B$

In $vSC_B$, 

- Retrieve $nft_{BC}^{I,n'}$ by $GID$ to generate $nft_{BC}^{B, n}$ , $ redeem\_ex(\ GID,\ nft\_id\_on \_B,\ address\_on\_I) \rightarrow EX_{issue}$

The above process is triggered in an Extrinsic, which will generate an Extrinsic id, recorded as $EX_{redeem}$

(iii) ***Unlock ***. *redeemer* submits $EX_{redeem}$ to chain $B$ , after $iSC_B$ verification, the corresponding relationship between $GUID$ and $nft_B^{x,n}$ is recorded in $iSC_B$ The method in $bSC_B$ is triggered atomically, and $nft_B^{x,n}$ is returned to the specified address.

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g7szni9t0lj30r70elgn2.jpg" alt="chain-relay-framework-2" style="zoom:50%;" />

### F. Algorithms 

##### Protocol: Issue

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g7sznio88rj30uc0j0aca.jpg" alt="image-20191010113808729" style="zoom:50%;" />

Explanation:

###### *requester* Related Operations:

- **lockB**($nft_B^{x,n}$, cond): Occurs in chain $B$. Lock $nft_B^{x,n}$ in $bSC_B$ and declare the address of *requester* on chain $I$, which corresponds to the transaction $T_B^{lock}$

- **verifyBOp**(lockB, $T_B^{lock}$, $\Delta_{lock}$) $\rightarrow T$ : Occurs in Bridge Core. *requester* submits $T_B^{lock}$ to $vSC_B$ in Bridge Core for verification if $T_B^{lock}$ actually happened on chain $B$ and satisfies the minimum required delay $\Delta_ {lock}$, which returns the result T(True), otherwise returns F(False).  

  If the result is T, then newGid($nft_B^{x,n}$) is automatically triggered in $vSC_B$, a new GID is generated, and $nft_B^{x,n}$ is mirrored in Bridge Core $nft_ {BC}^{B,n}$ and establish the correspondence between GID and $nft_{BC}^{B,n}$

- **verifyBCOp**(trigger, $EX_{issue}$, $\Delta_{trigger}$) $\rightarrow T$ : Occurs in chain $I$. *requester* Submit $EX_{issue}$ to $iSC_I$ in chain $I$, returning T if $iSC_I$ verifies the authenticity of $EX_{issue}$, otherwise returns F. After the verification is passed, by issuing the issue method, $nft_I^{x',n'}$ is issued to *requester* at the address of chain $I$.

###### *validator* Related Operations:

- **issueTransform**($vSC_I,\ pk_I^{requester},\ GID$ ): *validator* will automatically trigger the method in $vSC_I$ to destroy $nft_{BC}^{B,n}$ and Generate $nft_{BC}^{I,?}$ to indicate that the nft image will be added on the chain $I$ (this is why $?$ is used because the nft on the chain $I$ has not been added yet) , so it is not possible to get its token id) and establish a correspondence between GID and $nft_{BC}^{I,?}$. This operation will generate $EX_{issue}$.



##### Protocol: Transfer

![image-20190927191635665](https://tva1.sinaimg.cn/large/006y8mN6ly1g7fe8pswk9j3120050aac.jpg)

Explanation:

Calling the transfer method in $iSC_I$ on chain $I$ *sender* sends $nft_I^{x',n'}$ to *receiver*



##### Protocol: Redeem

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g7sznhfdu6j314w0q0n0c.jpg" alt="image-20191010114326817" style="zoom:50%;" />

Explanation:

###### *redeemer* Related Operations:

- **burn**( $nft_I^{x',n'}$, $GID$, $pk_B^{redeemer}$ ) : Occurs on chain $I$. *redeemer* Triggers $bSC_I$ to destroy $nft_I^{x',n'}$, but keeps the destruction record, $bSC_I$ can atomically check $GID$ and $nft_I^{x',n'} $ Correspondence. This will generate the transaction $T_I^{redeem}$
- **verifyIOp**(burn, $T_I^{redeem}$, $\Delta_{redeem}$) : Occurs in Bridge Core. The user submits $T_I^{redeem}$ to $vSC_I$. If $T_I^{redeem}$ actually happened on chain $I$ and met the minimum required delay of $\Delta_{redeem}$, then Find the corresponding $nft_{BC}^{I,?}$ according to $GID$ and automatically complete it to $nft_{BC}^{I,n'}$
- **verifyBCOp**(trigger, $EX_{redeem}$, $\Delta_{trigger}$) $\rightarrow T$ : Occurs in chain $B$. *redeemer* Submit $EX_{redeem}$ to $iSC_B$ in chain $B$. If $iSC_B$ verifies the authenticity of $EX_{redeem}$, it returns T, otherwise it returns F. After the verification is passed, by calling the issue method, $nft_B^{x,n}$ is released to *redeemer* at the address of chain $B$.

###### *validator* Related Operations:

- **burnTransform**($vSC_B,\GID,\nft_{BC}^{x,n},\ pk_B^{redeemer}$ ): *validator* automatically triggers the method in $vSC_B$, which will be $nft_{ BC}^{I,n'}$ destroys both $nft_{BC}^{B,n}$, indicating the image of nft to be released on chain $B$. This operation will generate $EX_{redeem}$.



## 

## IV. Cross-chain NFT Standards

To easily mark an item or an asset, we mark it with a unique identifier, which has different identifiers. Let us take the example of the items in the physical space. In an ideal situation, all the items should be in the same time and space, so that everyone can observe and facilitate the distinction and identification. But the reality is that different items may exist in different time and space, and the observer may not be able to see every item. In the same situation, in the virtual asset world, because there are different books or blockchain networks (referred to as domains), different items can be easily distinguished and located in the same domain because they have different identifiers, but the domain is inside. The observer does not recognize and parse the item ID from the external domain.

At present, many of the existing standards of the pass standard are mainly designed for the identification of intra-domain assets, and the reuse of assets in different domains is not taken into account. As a result, when the non-homogeneous assets are multiplexed, the individual Token ID cannot be To identify a unique asset, you also need to bring a lot of domain information, which is very complicated to implement.

Cross-chain technology can greatly help the exchange to achieve interoperability in a wider blockchain network, but at the same time, it also brings some cognition and usage thresholds to developers and users, including the identifiability of the certificate. The problem.

Because current pass standards, such as ERC20 or ERC721, only record ownership information on a particular chain, it is not considered that the pass may be distributed across two blockchain networks. When the pass is distributed across two blockchain networks, we need a set of identification and resolution systems to help the user and the pass application resolve and query the current pass status. When we give a NFT Token ID, we can't determine which blockchain network it is currently in, and who its owner is, because when the NFT cross-chain transfer, the pass is on one of the blockchain networks. It is active, while others are in an unavailable state, such as a locked state. In the absence of a pass-through resolution system, the out-of-chain operation cannot determine which chain the NFT is active on.

In a cross-chain environment, Token's identification and resolution problems require new solutions and standards to solve. Therefore, we introduce a analytic system based on the cross-chain certification of the pass to solve the positioning and analysis requirements of the cross-chain of the pass. Through the census system and the unique identifier in the domain, we can have the relationship between the certificate and the certificate of different domains. Map them up and identify the same and different between them.



### A. Globally unique identifier

In order to standardize the different standard pass identifiers, to provide identification and analysis methods, coordinate and interface with existing standards, and meet the standard requirements of community infrastructure construction. (To harmonise existing practice in identifier assignment and resolution, to support resources in implementing community standards and to promote the creation of identifier services.) The cross-chain system will assign a global ID (global_id) to each cross-chain pass.

### B. Local pass analysis method

The pass-through parsing module is a module embedded in the NFT cross-chain protocol for recording and parsing the global status of the current pass in the relay chain on the *Issuing chain* or its connected relay chain, and normalizing it. In an analytical format, it provides a pass-through parsing query and certification service for a cross-chain network.

In UNFO, the chainId and token id of the native NFT are placed in the type and value before entering the Bridge Core. The lock indicates who the owner of the NFT is. When the NFT is circulating within the Bridge Core, The lock_script may point to an ownership contract. When the NFT is locked in the backing contract, the lock_script may point to the redeem contract of the backing contract.

<img src="./images/nft_resolution.png" alt="NFT Resolution" style="zoom:200%;" />

### C. Non-fungible Token Standards on Polkadot/Darwinia

[WIP]



## Reference

[1] https://bitcoin.org/bitcoin.pdf

[2] https://github.com/ethereum/wiki/wiki/White-Paper

[3] https://github.com/EOSIO/Documentation/blob/master/TechnicalWhitePaper.md

[4] https://eips.ethereum.org/EIPS/eip-20

[5] https://eips.ethereum.org/EIPS/eip-721

[6] https://en.bitcoin.it/wiki/Hashed_Timelock_Contracts

[7] https://eprint.iacr.org/2018/643.pdf

[8] https://opensea.io/

[9] https://vitalik.ca/general/2018/04/20/radical_markets.html

[10] https://github.com/ethereum/wiki/wiki/Light-client-protocol

[11] https://elixir-europe.org/platforms/interoperability

[12] https://github.com/AlphaWallet/TokenScript

[13] https://github.com/darwinia-network/rfcs/blob/v0.1.0/en_US/0005-interstella-asset-encoding.md

[14] https://onlinelibrary.wiley.com/doi/pdf/10.1087/20120404

[15] https://wiki.polkadot.network/en/latest/polkadot/learn/spree/

[16] https://en.wikipedia.org/wiki/Unique_identifier

[17] https://en.wikipedia.org/wiki/Identifiers.org

[18] https://schema.org/

[19] https://medium.com/drep-family/cross-chains-a-bridge-connecting-reputation-value-in-silo-b65729cb9cd9

[20] https://github.com/paritytech/parity-bridge

[21] https://vitalik.ca/general/2018/04/20/radical_markets.html

[22] https://talk.darwinia.network/topics/99
