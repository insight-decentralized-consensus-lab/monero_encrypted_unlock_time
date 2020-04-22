![](https://raw.githubusercontent.com/insight-decentralized-consensus-lab/monero_encrypted_unlock_time/master/images/dual_logos.png) 

# Monero encrypted unlock\_time field

## Motivation:

Monero&#39;s privacy is highly dependent upon transaction indistinguishability to prevent transaction linkability. Anonymity is reduced when any characteristic of the transaction reveals something about the user or software that created it. One such example is statistical analysis of plaintext unlock times, which allows the Monero anonymity pool to be partitioned based on the software that generated the transactions.

In practice, unlock\_time for can be leveraged for blockchain analysis. We have observed 5+ different signatures in the wild:

- **unlock\_time = 0**
  - This is the default behavior of the core wallet (and any that properly mimic it)
  - An output that is &quot;spendable&quot; at the genesis block is never locked
  - Example: [e4098567e981e9596f8b2a449c7df24cc77268ff08280b5901b624c2de234202](https://xmrchain.net/tx/e4098567e981e9596f8b2a449c7df24cc77268ff08280b5901b624c2de234202/1)
- **unlock\_time = {1-6,10,12,13,15}**
  - An unknown wallet uses low-integer lock times
  - Setting an output to be spendable at height=2 does not make sense, so the developer&#39;s intent is unclear. Perhaps they thought block times were relative rather than absolute, or the field was being used for something else unrelated to unlock\_time.
  - It should be noted that there is an extremely unusual distribution of low values: 
![](https://raw.githubusercontent.com/insight-decentralized-consensus-lab/monero_encrypted_unlock_time/master/images/low_locktimes.png)
  - Example: [bf800d30889423fafdf7cde841f1a61d3372667a0efc7c6e8784f220c0dcc3a8](https://docs.google.com/document/d/1VOEF3Ntb8yk3DXzwu7-xHQ6QGMz3KTKoGe0gHYB4E4g/edit)
- **unlock\_time ~ 1,000,000+**
  - Lock times less than 500,000,000 are interpreted as block heights
  - Example: [93df46c18742ff6fd0ba86076bd360b0a32cda4f670b9944c9f176d5c9783959](https://xmrchain.net/tx/93df46c18742ff6fd0ba86076bd360b0a32cda4f670b9944c9f176d5c9783959)
- **unlock\_time ~1,400,000,000**
  - Large lock times represent unix epoch timestamps in seconds
  - Example: [012932593e59f21d10b7badc5f0556c1aaaefd60d0ebf05f1637361a66b17273](https://xmrchain.net/search?value=012932593e59f21d10b7badc5f0556c1aaaefd60d0ebf05f1637361a66b17273)
- **unlock\_time ~1,400,000,000,000**
  - Extremely large lock times might be intended as unix epoch timestamps in milliseconds?
  - Example: [2c2762d8817ea4d1cb667752698f2ff7597a051d433043776945669043d908b5](https://xmrchain.net/search?value=2c2762d8817ea4d1cb667752698f2ff7597a051d433043776945669043d908b5)
  - ^ Since unix time in milliseconds is [not supported in Monero](https://github.com/monero-project/monero/blob/master/src/cryptonote_core/blockchain.cpp#L3478), these outputs theoretically unlock in the year 46990

Peculiar signatures that enable transaction linkability affect not only the sender and the receiver, but also any user whose ring signatures selected these outputs as mix-ins. We must stress that in a decoy-based anonymity scheme, a user cannot negatively impact their own privacy without affecting others.

We could mitigate this issue by removing the unlock\_time field entirely, however there may be current and future use cases that would benefit from this functionality. A more sophisticated approach is implementation of encrypted lock times, which will not leak user/software fingerprints since the ciphertext will be uniformly distributed.

This research will deliver a thorough analysis of encryption solutions and their tradeoffs. The addition of an encrypted lock time requires changes to the range proof construction. Batching the lock time with other calculations will increase the verification time (which scales quadratically with number of commitments), however a separate proof results in a larger transaction. Consequently, we must ascertain the space/time complexity tradeoffs between different approaches before selecting one to implement. To be thorough, we will also examine how such modifications (which impact transaction size and verification time) could impact network security with respect to denial of service and spam attacks.

It should be noted that Monero is not the only protocol exhibiting information leaks due to erratic use of unlock\_time field, and other privacy coins have even seen the unlock\_time field used for steganography ([see thread](https://twitter.com/f2pool_official/status/1246154346481381378)). To our knowledge, it is possible for Monero to become the first private cryptocurrency to solve this issue and implement encrypted lock times.

## Key deliverables &amp; features:

- Detailed **system design** decisions (e.g. unlock\_time per output or per transaction?)
- Prototype code to quickly **test different approaches** , including simulating transaction construction, signing and verification
- Report of **quantified space/time/privacy tradeoffs** with each mitigation strategy
- Implementation **code for Monero** source tree, for at least one of the chosen approaches
- Comprehensive **research analysis writeup** , cross-referenced with code and documentation

## Overview:

R &amp; D Institution: Insight

Funding Institution: Monero CCS

Duration: 12 weeks (June - September 2020)
 Contributors:

- Developer in Residence: TheCharlatan
  - Decentralized Consensus Fellow at Insight
  - Hardware Wallet Developer at [Shiftcrypto](https://shiftcrypto.ch/)
  - Discovered multiple weaknesses in hardware wallets
  - Comp Sci and Physics Undergrad Student at the University of Zurich
  - [GitHub](https://github.com/TheCharlatan), [Twitter](https://twitter.com/the_charlatan_), [Blog](https://thecharlatan.github.io/)
- Principal Investigator: Isthmus (Mitchell Krawiec-Thayer)
  - Head of Research, Developers in Residence at [Insight](http://www.insightconsensus.com/)
  - Data Science for Monero Research Lab
  - Discovered information leaks in unlock\_time field
  - [GitHub](https://github.com/mitchellpkt/), [Twitter](https://twitter.com/Mitchellpkt0), [LinkedIn](https://www.linkedin.com/in/mitchellpkt/), [Medium](https://medium.com/@mitchellpkt)
- Other Insight contributors
  - Code &amp; documentation reviewers will be assigned as milestones near completion.
  - Additional thanks to office and administrative staff for creating a productive workspace.

## Timeline:

![](https://raw.githubusercontent.com/insight-decentralized-consensus-lab/monero_encrypted_unlock_time/master/images/timeline_v1.png)

## Project Roadmap:

### Phase 1: Technical Description

Phase 1 includes collecting and aggregating basic information around the problem. To aid this, a basic unlock\_time blinding implementation should be created. Currently there is no complete and correct mathematical formulation for unlock\_time blinding. This should be created as well. Some still open ended questions, like the communication of the locktime value to a transaction recipient and whether an entire transaction, or just a single output should be encumbered with an unlock\_time should also be answered here. The range proof required for the blinded unlock\_time has already been identified as a major technical hurdle. This is why further time has to be allocated here towards research into the range proof techniques and their size and verification time trade-offs.

**Phase 1 deliverables:**

- **Technical summary of the unlock\_time problem, including:**
  - **Mathematical formulation of unlock\_time blinding**
  - **Description of embedding into the current Monero transaction structure under current Monero consensus limitations**
  - **Description of current unlock\_time value distribution and a cursory explanation**
- **Basic code implementation to showcase the mathematical solution**
  - **Emulates normal Monero transaction structure**

### Phase 2: Monero Implementation

Phase 2 focuses on implementing what has been described in the technical writeup in the Monero source code. At first, nuances in aggregating the range proofs should be ignored. Once implemented and integrated in the unit tests, performance tests should be run, including performance tests on the simple non-aggregated range proofs. On completion, a test implementation on aggregated range proofs can be started.

**Phase 2 deliverables:**

- **Implementation of technical and mathematical description in the Monero codebase**
  - **Unit and performance tests**
  - **Serialization of new fields and integration with the existing api&#39;s**
- **Measures of size and verification time increase for implementation**

### Phase 3: Bulletproof Research

The range proofs are a significant chunk of the size and verification time increase due to the adoption of unlock\_time blinding. Building on the knowledge acquired in phase 1, an investigation should be started into the concrete technical steps needed to implement some form of batching for these range proofs. The performance tests as coded in phase 2, should be used here to show the tradeoffs in verification time and space. If verification is impacted significantly, this could have an impact on the Monero transaction fee structure. An advisory should be given here on its adoption into the Monero protocol.

**Phase 3 deliverables:**

- **Summary of range proof trade offs**
- **If significant an analysis on the fee structure effect**

### Phase 4: Review of Implementation, Wrap-Up

With the results gathered from phase 1-3, a final report of the work should be presented to the Monero research lab. Depending on the state of the work and feedback, a pull request of the implementation can be opened to the Monero repository, allowing developers to test the implementation and review the code. If feedback is positive, and integration into the hardfork schedule should be targeted. If more research into one of the various technical hurdles is required due to unacceptable trade-offs in transaction verification time or size, a technical brief of these problems should be compiled. Lastly, a final report of the work should be done, referencing the implementation and additional documentation.

**Phase 4 deliverables:**

- **Final report summarising the contents of phases 1-4**
- **Pull Request to Monero repository**
