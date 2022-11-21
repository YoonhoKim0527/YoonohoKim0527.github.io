---
title:  "About PDAO(Postech DAO)'s Simperby"
excerpt: "I will talk about PDAO's simperby protocol preview with both Eng/Kor"
published: true

#categories:
#- PDAO
#tags:
#- [PDAO, Simperby, rust]

layout: single
toc: true
toc_sticky: true
 
date: 2022-11-21
last_modified_at: 2022--11-22
---

## Simperby is a blockchain engine that runs an organization which is

1. **Decentralized**

2. **Standalone**
which means it is able to operate independently of other hardware and software), Sovereign and Self-hosted (다른 사람의 제어할 수 없는 서비스를 사용하는 대신 개인 web server를 사용하는 것)
3. **With a set of permissioned members**

***

in that it provides

1. **Consensus**  
 a safe and live mechanism to finalize the state of the organization, which is tolerant to byzantine faults in an asynchronous network.
2. **Governance**  
a democratic mechanism to make decisions on the organization, which can be finalized by the consensus.
3. **Communication Channel**  
a mechanism to communicate with each other in a decentralized, verifiable, and fault-tolerant way.

*한국어 해석*  

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

*한국어 해석*  

1)Simperby는 blockchain의 instance를 build하는 하나의 engine으로써, 이로부터 만들어진 blockchain은 `Simperby Chain`이라고 부른다.   

2)하나의 단위 상태 변화는 `Transaction`이라고 부른다.  

3)이 `Transaction`들의 나열을 `agenda`라고 한다. 
이 때 Governance의 투표는 target block height와 agenda에서 진행된다.   

4)하나의 `block`은 block header와 하나의 agenda를 포함한다. Agenda를 포함하지 않는 block은 허용되지 않는다. 그리고 block은 추가적으로 `extra-agenda-transaction`을 포함할 수 있으며 이는 block 제안자가 추가할 수 있다. 또한 이는 3가지의 type으로 제한이 된다.   

5)Governance의 participants들은 `members`라고 부른다.   

6)Consensus의 participants들은 `validators`라고 부르며 이들은 `members`들의 부분집합이다.  

7)한 round의 `consensus leader`는 `block proposer`라고도 불리며 이 사람은 그 round에서 block을 제안하는 validator이다.  

8)`round`라고 하는 것은 consensus leader가 block을 제안하는 기간을 정해주며, 이 시기에 consensus vote들이 이루어진다. consensus vote는 pre-vote, pre-commit이 진행되며 이에 대한 자세한 정보는 tendermint 항목에서 다룰 것이다.   

9)Validator들은 consensus protocol을 따른다면 정직하거나 비잔틴이 아니어야 할 것이다.   

10)Block을 propose 하는 block proposer or consensus leader는 책임감을 모두 가지고 행동할 시에 `faithful`하다고 하며 그들이 힘을 남용하지 않을 경우에 `good`하다고 한다. 그러지 않을 경우에는 각각 `lazy`, `bad`라고 한다.  

***

## Summarized version of Simperby Protocol

1. We want the node operation to be **simple and lightweight** to realize a truly distributed, decentralized and self-hosted organization.  

2. To be so, the key assumption is that most of the nodes are **rarely online** and the protocol produces new blocks **on-demand**.  

3. To accomplish that, the **consensus round must be very long**.  

4. For every block, members can **vote on or propose an agenda**. Agendas and votes are **propagated to each other by gossip network**.  

5. At the same time, members may also **propagate their arbitrary chats** used for the human communication, which is **ordered by the block proposer**.  

6. If there is a **governance-approved (majority-voted) agenda** on the network, the block proposer should **include it in the block along with the chat log**, and propose to the consensus.  

7. In that the block proposer should be online most of the time (chat coordination and block proposing), this **responsibility should be laid on a few of the validators most of the time**, to ensure the rarely-online assumption.  

8. Also the role of **block proposer has some authorities** (chat coordination and agenda inclusion) that can't be cryptographically verified if misused (typically censorship).  

9. Thus there exists **'few validators' that take over the block proposal most of the time, but can be lazy or commit harmful misuses** of their authorities which aren't really byzantine faults or invalid blocks.  

10. Normally in ordinary blockchains, this isn't a problem because the rounds are short and the block proposer changes regularly. And as explained, we're not.  

11. Therefore, we need a special mechanism to **'veto' the block proposer not to waste long rounds in the consensus layer**. Thus we introduce a special variation of Tendermint called *Vetomint*.  

12. In Vetomint, validators may veto the current block proposer, but still, the round will progress without timeout expiration (changing the block proposer) if all the honest validators either vote or veto.  

*한국어 해석*  

1)우리는 node의 연산이 `simple and lightweight` 즉, 간단하고 가볍길 원한다. 이를 통하여 진정으로 분산되고 탈중앙화되어 있으며 self hosted가 가능한 조직을 만들 수 있다.  

2)그렇기 때문에 우리의 중요한 가정은 대부분의 node들이 **rarely online**일 것이라는 것이다. 즉, 대부분이 온라인이지 않고 protocol producer들은 요구가 있을 경우에만 새로운 block을 만들게 된다.  

3)이렇기에 한 consensus round는 당연히 매우 길어야 한다.  

4)매 블록마다 member들은 투표를 하거나 안건을 제안할 수 있다. 안건과 투표 결과는 서로에게 gossip network로 전달이 된다. gossip network는 전염병이 퍼지는 방식을 기반으로 한 컴퓨터의 P2P 통신 절차이다.  

5)동시에, 멤버들은 자신들의 무작위 채팅을 전파할 수 있으며, 이는 block proposer에 의하여 order되게 된다.  

6)만약 네트워크에서 governance에서 승인한 안건이 존재하면 block proposer는 이를 chat log와 함께 block에 포함시켜야 한다. 이 때 block proposer는 일을 수행하는 사람이지 가치판단을 하는 사람이 아니기 때문에 어떤 안건을 포함시킬지 스스로 결정하면 안된다.  

7)Block proposer는 chat coordination과 block proposing을 해야 하므로 거의 모든 시간에 online이어야 한다. 우리가 대부분의 node들이 온라인일 확률이 적다는 가정을 보증하기 위해서 이 책임감을 적은 validator들에게 전가한다.  

8)Block proposer는 chat coordination(채팅 조정)과 안건 포함 등의 권한이 있으며 이를 proposer가 악용한다고 하더라도 암호학적으로 확인이 불가능해진다. 형식적으로 검열이 가능은 하다는 것이다.  

9)결국엔 적은 validator들이 존재하여서 그들이 대부분의 시간동안 block proposal을 책임지고 진행한다. 하지만 그들 또한 lazy하거나 harmful misusesFMF commi할 수 있다. 이는 Byzantine fault나 invalid block이 아닌 권한에 대해서만 가능하다.  

10)일반적으로 blockchain에서는 이를 문제로 삼지 않는다. 그들의 round는 매우 짧고 block proposer가 주기적으로 바뀌기 때문이다. 하지만 simperby에서는 그렇지 않다.  

11)따라서 우리는 veto(거부권)이라는 특별한 매커니즘을 사용한다. 우리는 긴 round를 consensus layer에서 낭비하지 않는다. 이를 우리는 tendermint의 변형인 vetomint라고 하겠다.  

12)Vetomint에서 검증하는 사람(validator)들은 현재 Block proposer에 대하여 거부권을 행사할 수 있으며 round는 제한시간 만료가 없이 진행이 된다. 이는 block proposer를 바꿈으로써 진행이 된다. 위 과정은 정직한 validator들이 투표를 하거나 거부권을 행사할 때 이루어진다. 
