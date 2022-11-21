---
title:  "About PDAO(Postech DAO)'s Simperby"
excerpt: "I will talk about PDAO's simperby protocol preview with both Eng/Kor"

#categories:
#- PDAO
tags:
  - [PDAO, Simperby, rust]

toc: true
toc_sticky: true
 
date: 2022-11-22
last_modified_at: 2022--11-22
---

##Simperby is a blockchain engine that runs an organization which is

####1. Decentralized 

####2. Standalone
which means it is able to operate independently of other hardware and software), Sovereign and Self-hosted (다른 사람의 제어할 수 없는 서비스를 사용하는 대신 개인 web server를 사용하는 것)
####3. With a set of permissioned members

in that it provides

####1. Consensus
 a safe and live mechanism to finalize the state of the organization, which is tolerant to byzantine faults in an asynchronous network.
####2. Governance
a democratic mechanism to make decisions on the organization, which can be finalized by the consensus.
####3. Communication Channel
a mechanism to communicate with each other in a decentralized, verifiable, and fault-tolerant way.

즉, Governance에서 단체의 의사 결정을 진행하고 이는 finalized되기 바로 전까지 진행한다. 그 후 Consensus에서 organization의 state를 finalize시킨다. 이 때 Consensus는 asynchronous network상에서 byzantine fault(악의적으로 공격을 하는 것)에 대하여 저항성이 존재한다. 마지막으로 communication channel에서는 탈중앙화되어 있고, 검증 가능하며, 장애 허용성이 있는 방식으로 서로 communicate하는 mechanism을 제공한다. 

***

## Basics and Terminologies

We assume that the reader of this document is familiar with the basic concepts of blockchain 

link of concepts of blockchain : "url", //TODO

- Simperby is an engine that can build an instance of a blockchain.
  Any blockchain built with Simperby is sometimes called a *Simperby chain*.
- The unit of state transition is called a *transaction*.
- A sequence of transactions is called an *agenda*.
  **Any governance voting is performed on an agenda and the target block height**.
- A *block* consists of the header and **a single agenda. A block without agenda is not permitted**.
- Additionally, a block may include some *extra-agenda transactions* which the proposer can include ex officio, but it is strictly restricted in only three types.
- The participants of the governance are called *members*.
- The participants of the consensus are called *validators*, which is a subset of the *members*.
- A *consensus leader* or a *block proposer* is a validator that proposes a block for the round.
- A *round* is a period of time in which a consensus leader proposes a block, and consensus votes(pre-vote, pre-commit) are performed. Refer to [Tendermint](https://tendermint.com/) for more details.
- A validator is *honest* or *not byzantine* if they follow the consensus protocol.
- A block proposer is *faithful* if they fulfill their responsibility and *good* if they don't overuse their power. They are *lazy* and *bad* otherwise.
  The notion of *responsibility* and *power* will be explained [later](#consensus-leader).
