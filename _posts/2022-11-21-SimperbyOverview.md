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

```
*한국어 해석*  

즉, Governance에서 단체의 의사 결정을 진행하고 이는 finalized되기 바로 전까지 진행한다.  
그 후 Consensus에서 organization의 state를 finalize시킨다. 
이 때 Consensus는 asynchronous network상에서 byzantine fault(악의적으로 공격을 하는 것)에 대하여 저항성이 존재한다.  
마지막으로 communication channel에서는 탈중앙화되어 있고, 검증 가능하며, 장애 허용성이 있는 방식으로  
서로 communicate하는 mechanism을 제공한다. 
```

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

```
*한국어 해석*  

1)Simperby는 blockchain의 instance를 build하는 하나의 engine으로써,  
이로부터 만들어진 blockchain은 `Simperby Chain`이라고 부른다.   

2)하나의 단위 상태 변화는 `Transaction`이라고 부른다.  

3)이 `Transaction`들의 나열을 `agenda`라고 한다. 
이 때 Governance의 투표는 target block height와 agenda에서 진행된다.   

4)하나의 `block`은 block header와 하나의 agenda를 포함한다. Agenda를 포함하지 않는 block은 허용되지 않는다.  
그리고 block은 추가적으로 `extra-agenda-transaction`을 포함할 수 있으며 이는 block 제안자가 추가할 수 있다.  
또한 이는 3가지의 type으로 제한이 된다.   

5)Governance의 participants들은 `members`라고 부른다.   

6)Consensus의 participants들은 `validators`라고 부르며 이들은 `members`들의 부분집합이다.  

7)한 round의 `consensus leader`는 `block proposer`라고도 불리며 이 사람은 그 round에서 block을 제안하는 validator이다.  

8)`round`라고 하는 것은 consensus leader가 block을 제안하는 기간을 정해주며, 이 시기에 consensus vote들이 이루어진다.  
consensus vote는 pre-vote, pre-commit이 진행되며 이에 대한 자세한 정보는 tendermint 항목에서 다룰 것이다.   

9)Validator들은 consensus protocol을 따른다면 정직하거나 비잔틴이 아니어야 할 것이다.   

10)Block을 propose 하는 block proposer or consensus leader는 책임감을 모두 가지고 행동할 시에 `faithful`하다고  
하며 그들이 힘을 남용하지 않을 경우에 `good`하다고 한다. 그러지 않을 경우에는 각각 `lazy`, `bad`라고 한다.  
```

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

```
*한국어 해석*  

1)우리는 node의 연산이 `simple and lightweight` 즉, 간단하고 가볍길 원한다. 이를 통하여 진정으로 분산되고 탈중앙화되어  
있으며 self hosted가 가능한 조직을 만들 수 있다.  

2)그렇기 때문에 우리의 중요한 가정은 대부분의 node들이 **rarely online**일 것이라는 것이다. 즉, 대부분이 온라인이지 않고  
protocol producer들은 요구가 있을 경우에만 새로운 block을 만들게 된다.  

3)이렇기에 한 consensus round는 당연히 매우 길어야 한다.  

4)매 블록마다 member들은 투표를 하거나 안건을 제안할 수 있다. 안건과 투표 결과는 서로에게 gossip network로 전달이 된다.  
gossip network는 전염병이 퍼지는 방식을 기반으로 한 컴퓨터의 P2P 통신 절차이다.  

5)동시에, 멤버들은 자신들의 무작위 채팅을 전파할 수 있으며, 이는 block proposer에 의하여 order되게 된다.  

6)만약 네트워크에서 governance에서 승인한 안건이 존재하면 block proposer는 이를 chat log와 함께 block에 포함시켜야 한다.  
이 때 block proposer는 일을 수행하는 사람이지 가치판단을 하는 사람이 아니기 때문에 어떤 안건을 포함시킬지 스스로 결정하면 안된다.  

7)Block proposer는 chat coordination과 block proposing을 해야 하므로 거의 모든 시간에 online이어야 한다.  
우리가 대부분의 node들이 온라인일 확률이 적다는 가정을 보증하기 위해서 이 책임감을 적은 validator들에게 전가한다.  

8)Block proposer는 chat coordination(채팅 조정)과 안건 포함 등의 권한이 있으며 이를 proposer가 악용한다고 하더라도  
암호학적으로 확인이 불가능해진다. 형식적으로 검열이 가능은 하다는 것이다.  

9)결국엔 적은 validator들이 존재하여서 그들이 대부분의 시간동안 block proposal을 책임지고 진행한다. 하지만 그들 또한  
lazy하거나 harmful misusesFMF commi할 수 있다. 이는 Byzantine fault나 invalid block이 아닌 권한에 대해서만 가능하다.  

10)일반적으로 blockchain에서는 이를 문제로 삼지 않는다. 그들의 round는 매우 짧고 block proposer가 주기적으로 바뀌기 때문이다.  
하지만 simperby에서는 그렇지 않다.  

11)따라서 우리는 veto(거부권)이라는 특별한 매커니즘을 사용한다. 우리는 긴 round를 consensus layer에서 낭비하지 않는다.  
이를 우리는 tendermint의 변형인 vetomint라고 하겠다.  

12)Vetomint에서 검증하는 사람(validator)들은 현재 Block proposer에 대하여 거부권을 행사할 수 있으며 round는 제한시간  
만료가 없이 진행이 된다. 이는 block proposer를 바꿈으로써 진행이 된다. 위 과정은 정직한 validator들이 투표를 하거나 거부권을  
행사할 때 이루어진다.
```

***

## Simple and Light Node Operation

Simperby strongly encourages each member to run their node,
to ensure that the network is decentralized and distributed (and so truly self-hosted). In other words, running a Simperby chain is not about using AWS servers by a few people but using their laptops to physically run the chain. To achieve that, it is important to **keep the node operation as simple and lightweight as possible**.  

```
Simperby에서는 각각의 멤버가 그들의 node를 돌릴 것을 강하게 추천한다. 이는 탈중앙화와 분산된 network를 보증하기 위해서이다.  
즉, simperby에서는 AWS 서버를 이용하지 않고 그들 각각의 laptop으로 chain을 돌리게 된다. 따라서 하나의 node operation을  
최대한 간단하고 가볍게 만드는 것이 중요하다. 
```

We will firstly move on to "Rarely-online nodes".  

### Rarely online nodes  

One of the most important conditions for *simple* and *lightweight* node operations is how often the node needs to be online. If we want to make the validators run nodes on their laptops, it is not realistic to assume that they will be online 24/7.  

Since Simperby's BFT consensus assumes a partially-synchronous network, rarely-online nodes can be trivially handled because it's no more than one typical case of an asynchrony, if  

1. The consensus round is (or has grown to be) long enough to cover all appearances of the nodes.  

2. At least one node stays online serving the gossip protocol reliably when there is a network broadcast.   

Condition 1 turns out to be a tough challenge in the later sections, so keep it in mind.  

```
간단하고 가벼운 node operation을 만들 때 가장 중요한 것중 하나는 node가 얼마나 자주 online이어야 하냐는 것이다.  
만약 우리가 validator들이 그들의 laptop으로 node를 돌리게 하고 싶다면 현실적으로 그들이 계속 online이라고 가정하기 힘들다.  
Simperby의 BFT consensus는 **partially-synchronous network**를 가정하기 때문에 rarely online node들은 당연히  
handle이 가능하다. 왜냐면 그들은 다음 두 가정을 따를 시에 그저 asynchrony의 특수 케이스중 하나이기 때문이다.  

1)충분하게 round가 길어서 rarely online인 모든 node들의 출현이 가능해질 때까지 cover할 수 있어야 한다.  

2)적어도 하나의 node는 online으로 존재하여서 network broadcast를 gossip protocol을 통하여 진행하여야 한다.  

1번 조건의 경우에 상당히 힘든 과제가 된다.  
```

### On-demand block production

Rarely-online nodes mean rarely-progressed blockchain.
(note that in any BFT algorithm, at least 2/3 of nodes must have participated in the consensus process to produce a block).  

This will inevitably compromise the latency or throughput of the blockchain, but it doesn't matter because
**Simperby is NOT a general contract platform for serving public users,**
**but a governance platform for a set of permissioned members**.  

Simperby blockchain includes only governance-approved state transitions (there are very [few exceptions](#consensus-leader)).
Every governance agenda must be approved by a majority vote of the members so it will take time(days or even weeks).
Considering that, naturally, Simperby's performance is bound by the governance process, not the consensus which is slowed by lightweight node operation.

```
rarely online이라는 것은 즉, 자주 진행되지 않는 블록체인이라는 뜻이다. BFT algorithm에서는 2/3 이상의 node들이 consensus    
과정에서 협력을 해야 block을 만들 수 있다. 이렇게 처리하는 것은 블록체인의 대기 시간이나 처리량에 안좋은 영향을 미칠 수 있지만    
simperby는 공개적으로 사용자에게 서비스를 제공하는 플랫폼이 아니라 권한이 있는 구성원들을 위한 governance 플랫폼이기 때문에    
큰 문제가 되지 않는다.  

Simperby blockchain은 governance에서 승인한 상태 변화만을 포함한다. 이 때 consensus leader에 의한 것들은 예외로 처리한다.    
모든 거버넌스의 안건들은 멤버들의 투표를 통하여 승인을 받아야 한다. 이것이 몇 일 혹은 몇 주가 걸릴 수도 있다. 따라서 자연스럽게    
simperby의 performance는 가벼운 node operation에 의하여 slow down된 consensus에 의하여 결정되지 않고 governance의    
진행에 Bound 되어 있게 된다.  
```

### MultiChain DAO

One of the notable use cases of Simperby is that it can establish a **multi-chain DAO**. (See [this](./multichain_dao.md) for more details) . 
It's super-easy to expand the organization to other existing chains (so-called 'colony chains') using
the light client of the Simperby consensus.  
Since the light client is uploaded as a contract to other chains,
any block progress of the Simperby consensus will require an additional
transaction to update and store the Merkle root of it.  
This might not be cheap, because **the organization will have to pay for the gas cost for every colony chain, for every Simperby block**.  
That is another reason that on-demand block production is considered reasonable.  

```
Simperby는  multi-chain DAO이다. 이 때 multichain이란 다른 chain끼리 상호작용을 하는 것을 말한다. 그리고 Multichain  
DAO의 경우에는 여러 체인에 걸쳐서 DAO가 작용한다는 것이다. 보통의 DAO의 경우는 단일 체인 위의 contract로 올라간다.  
Organization을 다른 존재하고 있던 chain으로 확장하는 것은 매우 쉽다. 이를 colony chain이라고 부르며, 이는 Simperby  
consensus의 light client에 의하여 작용한다. Light client는 다른 chain의 contract로 올라가기에 Simperby Consensus의  
아무 블록 진행은 추가적인 transaction update가 필요하고 이에 따라서 Merkle root를 저장하게 한다. 이는 싸지만은 않은데,  
그 이유는 조직이 모든 colony chain에 대하여 비용을 지불해야하고, 이는 모든 Simperby block에 대해서 이루어져야 하기 때문이다.    
```

***

## Governance

Simperby's governance is implemented by P2P voting.
- Any member can propose an agenda, and it will be propagated over the P2P network.
- Any member also can cast a vote on the agenda if they prefer it. The vote will be propagated as well.
- Any agenda that has over 50% votes is **eligible** to be included in a block.
- A block can be **valid only if it includes an eligible agenda**. In other words, empty blocks are not allowed.
- To decide which eligible agenda to finalize is the role of consensus, not the governance.
- The consensus is a BFT algorithm based on a leader-and-round style.

```
먼저 Simperby의 Governance는 P2P voting에 의하여 구현된다. 이 때 P2P(Peer to Peer) network란 비교적 소수의 서버에  
집중하기보단 망구성에 참여하는 기계들의 계산과 대역폭 성능에 의존하여 구성되는 통신망이다. P2P 통신망은 일반적으로 노드들을 규모가  
큰 애드혹으로 서로 연결하는 경우에 이용된다. 이에 대한 posting을 tag하겠다.
[about P2P network](https://post.naver.com/viewer/postView.nhn?volumeNo=14678102&memberNo=19185109).  

이 때 특성은 다음과 같다.  
- 모든 member는 안건을 제시할 수 있다. 그리고 이것은 P2P network에 의하여 전파된다.  
- 아무나 member이기만 하면 마음에 들 경우에 그 안건에 대하여 vote를 할 수 있다. 그 투표도 마찬가지로 전파가 될 것이다.  
- block에 들어갈 자격이 있는 vote는 50%이상의 vote를 받은 안건이다.  
- block은 자격이 있는 안건을 포함할 경우에만 valid하다. 즉, 빈 block은 불가능하다는 것이다.  
- governance에서는 어떤 자격이 있는 안건을 finalize하는 역할은 하지 않는다. 이는 consensus의 역할이다.  
- Consensus는 leader-and-round style의 BFT 알고리즘에 기반한다.  
```

***

### What can be an agenda?

- Transaction is only the unit of a **state transition**, not an individual item signed and broadcasted as in the case of a conventional blockchain.
- An 'agenda' is defined as `(Proposer, Height, [Transaction(s)])`.
- Only a **single agenda** can be included in a block. This is for preventing complicated dependency problems.
  - If multiple agendas are allowed, each agenda must be independent; Otherwise, voters can't be sure how each agenda is ordered, or even whether it is included in the finalized block. This will cause a severe restriction of the possible agenda items.
- Because `Height` is a part of an agenda,
  every agenda and its vote will be outdated and thus discarded if the block height progresses though it may be re-proposed and re-voted.
  
```
- 먼저 transaction은 상태 변화의 단위일 뿐이고, conventional - - blockchain에서 이는 사인되고 broadcast되는 독립적인   
item이 아니다.  
- 안건은 `(제안자, 블록 높이, [transaction(s)])`으로 정의된다.  
- 오직 하나의 안건만이 block에 포함되어야 하며, 이로써 복잡한 dependency problem을 해결한다.  
- 만약 여러 개의 안건들이 허용이 된다면 이들 각각이 독립적이어야 한다. 반면에, 투표자들은 각 안건이 어떻게 order되어 있는지 확신할 수  
없고, 그것들이 finalized block에 포함되는지도 알 수 없다. 이로써 가능한 agenda item에 심각한 규제를 발생시키게 된다.  
- 또한 agenda에 높이가 적혀 있기 때문에 체인의 높이가 변경될 시에 그 agenda는 무효화된다.  
```

*** 

## Chatting

Again, Simperby is a blockchain engine that runs a **standalone, sovereign, and self-hosted** organization.
Thus, the communication channel for the organization should be so as well.
The only way to implement such channel (instead of Discord) would be to leverage the existing P2P network.  

- Any member can broadcast its message, signed by their public key.  

- Each message contains a list of hashes of every previous messages that the signer has perceived, **forming a chain**.  

- Like PoW, the longest chat chain is considered a candidate for the canonical one.  

- The consensus leader (block proposer) for the round plays a special role.  

- the leader can broadcast a chat as well, but such chat is considered a **semifinalized point**.  

- A *faithful* leader would **frequently** insert its `ack` chat **on the (seemingly) longest chain**.  

- A *good* leader must not commit a fork (semifinalizing two conflicting chains) . 

- On top of the recent leader-semifinalized chat, other members continue chatting using the longest-chain rule until the next finalization point.  

- Once the governance reaches an eligible agenda, the leader must semifinalize the chat chain for the last time, and include it in the block. If the block is finalized by the consensus then the chat log for the block is finalized as well.

- A *good* leader must include the last-semifinalized chat chain which is the one that the other members would consider the canonical one.

```
먼저, Simperby는 다시 말하지만 **Standalone, Sovereign, and self-hosted** 되어 있는 조직이다. 그래서, 조직의 대화 채널 또한   
그래야 한다. 이런 채널을 구현하는 Discord를 제외한 유일한 방법은 이미 존재하는 P2P network를 레버리지하는 것이다. 여기서 레버리지의   
원래 뜻은 남의 자본을 가지고 자기 자본의 이익률을 높이는 것이다.  

- 아무 멤버는 메세지를 broadcast할 수 있으며 그것들은 그들의 public key로 사인되어 있다.  
- 모든 메시지에는 signer가 받은 전 메시지의 hash list들이 포함되어 있다. 그들이 결국에 chain을 형성하게 된다.  
- PoW와 같이 longest chat chain이 canonical chain의 후보로 고려된다.  
- 그 round에 대한 컨센서스 리더가 특별한 역할을 한다.  
- 리더도 Chat을 broadcast할 수 있지만 그들은 `semifinalized point`로 고려된다.  
- 충실한 리더는 `act` chat을 가장 길어보이는 chain에 자주 삽입할 것이다.  
- 좋은 리더는 fork를 commit하면 안된다. 충돌하는 두 chain을 semifinalize하면 안된다는 의미이다.  
- 가장 최근의 leader-semifinalized된 chat이후에 다른 멤버들은 다음 finalized point까지 가장 긴 체인 법칙을 따라서 채팅을 이어  
나간다.  
- 한번 거버넌스가 자격이 있는 안건에 도달하게 되면 리더는 무조건 chat chain을 semifinalize시켜야 하고, 이를 block안에 포함시켜야  
한다. 만약 block이 consensus에   의하여 finalized되면 그에 따라 채팅 log도 마찬가지로 finalized되게 된다.  
- 좋은 리더는 last-semifainalized chat chain을 포함시켜야 하고, 그것은 다른 멤버들이 canonical한 것으로 고려하던 것이어야 한다.   
```

*** 

### Why doesn't the leader just serve a chat server?

The reason that the leader only 'semifinalizes' the chat chain, instead of running a full chat server, is that it is more censorship-resistant.

- Compared to a full chat server, the longest-chain protocol has a
  **weak** (because of the lack of network synchrony) **consensus** (longest chain) of the canonical total chat ordering, among the members.
- Thus if the leader attempts censorship (i.e., repeatedly semifinalizes a chat chain that is not the longest one observed by other members), it's easy to *notice* that, though it's not theoretically verifiable due to the assumption of network asynchrony. (it might be a coincidence originated from the possible network delay around the leader)
- Nonetheless, if the leader attempts censorship more, it becomes more suspicious.

Note that if there is a malicious member who tries to spam the chat, the leader will not finalize the spam chain, and the other members would recognize such censorship as something `not bad`.

```
리더가 그저 chain을 `semifinalize`하기만 하고 채팅 서버를 운영하지 않는 이유는 그것이 더 검열에 저항성이 있기 때문이다. 

- full chat server에 비교하여서 가장 긴 chain에 기반한 프로토콜은 total chat ordering에 대하여 **weak**(네트워크 synchrony의 부족), **consensus**(가장 긴 체인) 하다. 

- 따라서 만약 리더가 검열을 시도(가장 길지 않은 체인을 semifinalize 시킨다던지)한다면 이는 알아채기 쉽긴 하다. 하지만 이는 네트워크 asynchrony라는 가정이 있기에 이론적으로 검증이 가능하지는 않다. (왜냐하면 리더의 네트워크의 지연에 의하여 우연하게 이루어졌을 수 있기 때문이다.)  

- 그래도 만약 리더가 계속 검열을 시도한다면 의심스러워지긴 할 것이다.  

스팸 채팅을 시도하는 나쁜 멤버가 있다면, 리더는 그 spam chain을 finalize하지 않을 것이고, 다른 멤버들은 그러한 검열에 대하여 나쁘지 않다고 기억할 것이다.  
```

***
