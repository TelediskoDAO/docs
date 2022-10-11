# Technical journey into the Neokingdom DAO
## Introduction

Neokingdom DAO is a relatively new but very ambitious project whose inception dates back to 2016.

The basic idea is to provide the legal and technical framework to create DAOcracies, namely companies running with the same founding principles of a DAO.

In a nutshell:

* Contributors invest their time and earn tokens.
* Tokens give access to dividends and voting rights on the DAO's *resolutions*.
* Tokens can also be sold internally or pledged to the DAO, at the nominal value of 1 EURO (yep, people need to pay the rent).
* Everything is regulated through *resolutions*, from the hiring of a new contributor, to the dissolution of the company.
* People can also just invest capital, get tokens and earn dividends, without having votings rights.

All this in a legally compliant way. Meaning that the founders can turn any type of company into a DAO (not only crypto projects), and that freelancers can be a part of the projects they work on by deciding how much to keep in share, and how much to cash out.

It's a new way of shaping the relationship between a company and its contributors. A way aimed at building a functional symbiosis between people and commercial entities.

## Coding the Law
Creating the groundwork to bring the first Neokingdom DAO to life (Teledisko DAO) required an initial effort that has been purely legal.

Since its inception, more than a year has been spent by the team to get the legal foundation that could guarantee the lawfulness of our initiative. It will take time for our DAO framework to merge perfectly with the Estonian legal framework. But given how closely we worked with the Estonian government, we are confident that any upcoming issue will be addressable without legal troubles.

Once the first drafts of the [Articles of Association (AOA)](https://github.com/TelediskoDAO/legal/blob/v1.0.0/AoA.md) and the [Shareholders' Agreement (SHA)](https://github.com/TelediskoDAO/legal/blob/v1.0.0/SHA.md) have been completed, we started developing the Smart Contracts and the dApp on top.

The main challenge for the development of this first implementation of the DAO was *translating* legal documents to running code (solidity smart contracts).

Before starting the development of the smart contracts, we decided to spend the first weeks thoroughly studying the Articles of Association and the Shareholders' Agreement. The main point was to get to a very clear understanding of which articles could be automated with code, how much be done on-chain, and what had to be implemented off-chain. This process required quite a few iterations with the lawyers.

## General Architecture
The system architecture has 4 layers:

1. Legal: Articles of Association and Shareholders' Agreement. This is the layer connecting metaverse to universe, code to law. Everything that has been implemented, used this layer as a list of requirements.
2. Smart Contracts: this is the code implementing the law, implemented in solidity and running on EVMOS.
3. Content Delivery: we use The Graph to collect and structure our on-chain information, for easy and expedite access.
4. Clients:
   1. Mailer: a cloudfront worker sending out emails to the relevant DAO participant about legally sensitive events.
   2. Dapp: this is the access point for the Contributors of the DAO. Contributors use the dapp to vote, submit new resolutions, sell tokens, and track their time. The dapp can be accessed by the general public as well to check out resolutions and votes.

![Architecture](https://raw.githubusercontent.com/TelediskoDAO/docs/264faac43cc1b3c61d49a21e5be63fc1385c2629/architecture.png)

## Shares and Permissions
Article `2.` of the SHA describes in aboundant detail the process of joining the DAO as Shareholder and as a Contributor. What ultimately discriminates a DAO Shareholder from any other person is described in point `2.1.3`

> `Upon accepting a person as a new shareholder of DAO, DAO shall insert the person into the shareholders' register and gift or sell such person 1 DAO share with the nominal value of EUR 1;`

The possession of this nominal share is what ultimately defines the rights of the Shareholder with respect to the DAO: voting rights, dividend rights, etc.

The logic is implemented by the [`ShareholderRegistry`](https://github.com/TelediskoDAO/contracts/tree/v1.0.0/contracts/ShareholderRegistry), which is the single point of truth when it comes to the DAOs permission management.

Shares are managed by an `ERC20` contract, extended to comply with the points of the SHA. For instance:

* Every address is allowed to own 1 share maximum.
* Shares can only be transferred after a DAO vote (which implies that only an automatically executed resolution can trigger `transferFrom`).

The ownership of a share is the pre-condition to have any of the statuses available to the DAO

* Investor
* Shareholed
* Contributor
* Managing Board

The contract also provides some utility methods that the other contracts can use to, for instance, understand whether an account can vote.

The usage of tokens to deal with the shares was not explicitely required by the legal documentation. It was a technical decision that allowed us to more easily implement the logic by re-using existing components. It also allowed the implementation of the Smart Contract to match more closely the specific words of the SHA.

## Tokenomics
The tokenomics of this DAO is probably what distinguishes it from most of the existing ones. As its founder said: 

> The system is socialist on the inside, capitalist on the outside.

The way tokens are managed inside the DAO makes sure no speculative or profit-maximizing behaviour is incentivized. 

On the other side, once the tokens "leave the DAO", they become free to be traded as wildly and heartlessly as it gets.

Articles `4.` and `10.` of the SHA provide all the necessary information about what the tokens of teledisko represent and how they should be regulated.

The logic has been implemented within the [`TelediskoToken`](https://github.com/TelediskoDAO/contracts/tree/v1.0.0/contracts/TelediskoToken). This is a beefed-up ERC20 Smart Contract.

"Beefed-up" in the sense that we needed to implement different levels of "freedom" for the tokens (remember when we were talking about "socialistic on the inside, capitalistic on the inside"?).

All tokens that are not owned by Contributors (information available in the `ShareholderRegistry`) are free to move.

For the others, we implemented a basic order book functionality within the token itself:

* Contributors are able to offer their tokens to other contributor.
* Other contributors are able to receive offered tokens.
* Tokens are locked until 7 days after the first offer.

All of this easily visualizable (and doable) from our Dapp

![Token Page](https://raw.githubusercontent.com/TelediskoDAO/docs/264faac43cc1b3c61d49a21e5be63fc1385c2629/tokens_page.png)

Currently, the acceptance of the offer and the transfer of the monetary amount to the token-holder is done more or less manually (a multisig wallet changes the state of the contract, Euros are transferred via bank-wire). But we are working to integrate EEUR in our ecosystem and have a fully automated escrow mechanism inside the Smart Contracts.

We used ERC20 even in this case, because (on top of the same reasons expressed for the ShareholderRegistry) the legal documents explicitly mentions `tokens` as the legal tender of the DAO.

## Governance and Snapshotting

The governance of the DAO is regulated throughout the whole SHA and AoA, with legal obligations related to the proposal and execution of the resolutions.

Among the many points that we implemented in the DAO, one of them had a quite pervasive implication in the architecture of the Smart Contracts, namely Article `7.4` of the SHA:

> `The number of tokens being taken into account for a DAO vote and their allocation between Shareholders will be fixed on a day the DAO vote is announced. Subsequent transactions with tokens are not taken into account.`

In simple words: the voting power of each Contributor of the DAO depends on the time the resolution has been created. So if Alice has 42 tokens on Sept 22nd, a resolution is approved on Sept 23rd, and Alice receives 10 tokens right after that resolution approval, then Alice's voting power for that resolution is still 42.

Same applies for delegation, demotion (when a Contributor is removed from the DAO), etc. Namely: if Alice delegates Bob on Sept 22nd, a resolution is approved on Sept 23rd, and Alice redelegates Carol afterward, then Bob is still Alice's delegatee for that resolution.

Basically most of the state of the DAO is *snapshottable*, with snapshots taken every time a draft resolution is approved by the managing board.

![Unnecessary but street-credits worth meme](https://raw.githubusercontent.com/TelediskoDAO/docs/264faac43cc1b3c61d49a21e5be63fc1385c2629/meme.png)

The base logic is implemented in [`Snapshottable`](https://github.com/TelediskoDAO/contracts/blob/v1.0.0/contracts/extensions/Snapshottable.sol), that is inherited by all the contracts of the DAO.

Each contract has then, for each relevant state-reading function, a default version and an `at` version. For instance, TelediskoToken has `balanceOf` and `balanceOfAt`, the former used to know the balance of an address for a given resolution id.

The logic was inspired by the [OpenZeppelin `ERC20Snapshot` contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Snapshot.sol), extended to support our state versionining requirements.

## Content Delivery: The Graph

The state of our DAO is tracked by two main components:

* IPFS: for the more verbose type of content, such as the resolution texts.
* Smart Contracts: for everything else.

As some of you probably already know, smart contracts are not the best "content delivery" service that a dapp could possibly have.

For simple dapps, maybe it's sufficient to interact directly with the Smart Contract (for instance if we only need to display the current balance of an ERC20 token holder), but for more complex use cases, there is a complication: the query interface. A Smart Contract is not a DB engine and as such, it offers very limited data access capabilities. The developer has to know ahead of time all the needed access patterns in order to provide functions able to serve that. Imagine having to join multiple data sets or aggregate/filter a long list of data points: would you really like to implement this logic in the Smart Contract?

To address this issue, we decided to leverage [The Graph](https://thegraph.com/en/), *an indexing protocol for querying networks like Ethereum and IPFS.*

What The Graph does is to listen to all events coming from a specific Smart Contract and collect its data in a structured format, ready to be queried via GraphQL. The indexing logic can be customized at will.

If the Smart Contract events refer to immutable data (such as anything hosted on IPFS), it's also possible to  index that inside the data model.

It's pretty handy as it allows a certain degree of freedom in the creation of the dapp and any other read-only functionality related to the DAO.

For querying, it's possible to use the so-called **Graph Studio**, which is the decentralized network of indexers and curators that offers its data delivery service for a small fee (to be paid in GRT, Graph Token).  Given that EVMOS is currently not supported though, we had to spin-up our own Graph indexer and IPFS node. We hope it's only a temporary solution.

Our data model and indexing logic is implemented in [its own repository](https://github.com/TelediskoDAO/subgraph).

One example:
```
export function handleResolutionRejected(event: ResolutionRejected): void {
  const resolutionIdStringified = event.params.resolutionId.toString();
  const resolutionEntity = Resolution.load(resolutionIdStringified);

  if (resolutionEntity) {
    resolutionEntity.rejectTimestamp = event.block.timestamp;
    resolutionEntity.rejectBy = event.transaction.from;

    resolutionEntity.save();
    return;
  }

  log.error("Trying to reject non-existing resolution {}", [
    resolutionIdStringified,
  ]);
}
```

This code snippet is what makes sure that each rejected resolution is marked as such inside our data-model, allowing us to show it in our dapp:

![Rejected Resolutions](https://raw.githubusercontent.com/TelediskoDAO/docs/264faac43cc1b3c61d49a21e5be63fc1385c2629/rejected.png)

If you need help setting up your graph node with IPFS for your EVMOS project, feel free to get in touch with us!

## Upgradeability

Teledisko DAO is the first experiment of what will hopefully be a series of transitions. We are positive that the work we have done is meticulous, both on the legal and technical level.

It is realistic to assume, though, that given the novelty of our approach to work, some knots might emerge on the way. In order to be able to untie them, we need the flexibility to perform maintenance operations on our contracts, both to fix potential security issues and also in case the law beneath should change.

For this reason, we deployed all the smart contracts behind proxies. The proxy update pattern allows us to have a configuration like this (diagram sourced from [the official OpenZeppelin documentation](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies))

```
User ---- tx ---> Proxy ----------> Implementation_v0
                     |
                      ------------> Implementation_v1
                     |
                      ------------> Implementation_v2
```

Each of our smart contracts is actually a proxy that points to the current implementation. Should a bug fix be needed, we can just deploy a new implementation and point the proxy to it.

This configuration comes pretty much out of the box [with hardhat](https://docs.openzeppelin.com/upgrades-plugins/1.x/api-hardhat-upgrades). There are some catches of course (you can't for instance change the storage layout), but we recommend to refer to the official documentation to know the details.

## Incremental Decentralization

The ultimate vision of a Neokingdom DAOs is to be fully autonomous and decentralized. In the ideal world, no one inside the DAO is a single point of failure, everything that happens is triggered by all contributors and executed by the smart contracts.

This though requires a very high level of automation. Postponing the launch of the first DAO until that moment would have implied a very late landing in the market and the risk of spending too much money before we even knew whether what we are doing makes sense. Remember: this is a first timer, there are many new ideas and some legal alchemy that need to be validated.

We therefore decided to "go live" with the minimum necessary level of decentralization (hence automation), where us developers have still the right to operate some of the functions of the DAO (e.g.: the minting of tokens).

Thanks to the upgradability of the smart contracts and the way we engineered the resolutions (the core of the DAO machinery), we will be able to slowly let go of these responsabilities.

More concretely, this bit of code in [`ResolutionManager.sol`](https://github.com/TelediskoDAO/contracts/tree/v1.0.0/contracts/ResolutionManager/ResolutioManager.sol) is the game changer:

```
  address[] memory to = resolution.executionTo;
  bytes[] memory data = resolution.executionData;

  resolution.executionTimestamp = block.timestamp;

  for (uint256 i; i < to.length; i++) {
    (bool success, ) = to[i].call(data[i]);
    require(success, "Resolution: execution failed");
  }
```

Each resolution can optionally contain some execution data and the contract that should execute it.
Meaning that, in principle, each resolution can come also with the code that fulfills it.

The possibility is there, but we are still not using it: the more we get confident about the goodness of our process, the more we can automate.

## Next steps

There is a magnificent future ahead of Neokingdom DAO, and our vision is still at the very early stages. Now we need to make sure the DAO structure for Teledisko is solid and tested, and expand the framework we are building to other Neokingdoms.

Technically speaking, though, we are aiming to decentralize the most important features of the DAO by the end of the year. The next step in this direction is the management of contributors' tokens: currently, if a contributor wants to sell a token within the DAO, the offer is automated, but the match-making and consequent money transfer is manual.

For this reason, we need first to be able to automatically deal with Euros or Euro-equivalent cryptocurrencies within the smart contracts. 

With [EVMOS Proposal #56](https://app.evmos.org/governance/56), we successfully registered EEUR inside the EVMOS ecosystem, hence laying the foundation to integrate a Euro cryptocurrency in the DAO's economy and to ultimately automate the financial processes of the company.

We hope you liked the article and if you questions, feel free to jump [into our discord](https://discord.com/invite/CvcTJWD4aS). We are currently quite busy, but we would love to have you around :)