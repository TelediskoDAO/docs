# Technical journey into the Neokingdom DAO
## Introduction

Neokingdom DAO is relatively new but very ambitious experiment whose inceptions dates back to 2016.

The basic idea is to provide the legal and technical framework to create DAOcracies, namely companies running with the same founding principles of a DAO.

In a nutshell:

* Contributors invest their time and earn tokens
* Tokens can be used to get dividends and vote on the DAO's Resolutions
* Tokens can also be sold internally or pledged to the DAO, at the nominal value of 1 EURO (yep, people need to pay the rent)
* 

EVM based, legally compliant, DAOs

## Vision
In the canonical employment model, employees and companies have two different goals. Employees, even if they want, are not included in the decisional process of the company, and they don't have access to dividends (if the company is doing well all they can aim for is a salary raise).

Companies, on the other side, need to maximize their profits. From squeezing their employees as much as possible to compulsive hirings and layoffs, companies find their way to survive in the market at the expense of their employees.

In the case of VC–backed companies, investors own part of the company, having access to voting and dividends. They have decisional power, while employees that are contributing their time to the company don't have a say.

Companies and employees don't have their incentives aligned, creating all kinds of byproducts.

People should be able to invest their time, not sell it.

The inspiration to build Teledisko DAO came back in 2016 when Benjamin von Uphues (the founder of Teledisko GmbH) thought about how to change the structure of his company to bring its contributors away from capitalistic slavery.

It was a new way of shaping the relationship between a company and its contributors. A way aimed at building a symbiosis between people and commercial entities.

Turning the Teledisko GmbH into the Teledisko DAO was the first step toward the fulfillment of this vision.

In 2021 the legal work started (the DAO had to be a legally recognized entity).

In 2022, we started building what in July became the first release of the Teledisko DAO, on EVMOS.

## Coding the Law
Creating the groundwork to bring the first Neokingdom DAO to life (Teledisko DAO) required an initial effort that has been purely legal.

Since its inception, more than a year has been spent by the team to get the legal foundation that could guarantee the lawfulness of our initiative. It will take time for our DAO framework to merge perfectly with the Estonian legal framework. But given how closely we worked with the Estonian government, we are confident that any upcoming issue will be addressable without legal troubles.

Once the first draft of the [Article of Association](https://github.com/TelediskoDAO/legal/blob/main/AoA.md) and the [Shareholders' Agreement (SHA)](https://github.com/TelediskoDAO/legal/blob/main/SHA.md) has been completed, we started developing the Smart Contracts and the dApp on top.

The main challenge for the development of this first implementation of the DAO has been the connection between law and code.

When we needed to start developing the Smart Contracts, we actually decided to spend the first weeks thoroughly studying the Article of Association and the Shareholder Agreement. The main point was to get to a very clear understanding of how the law could be automated with code, and this required quite a few iterations with the lawyers.

## General Architecture
The system architecture has 4 layers:

1. Legal: Article of Association and Shareholder Agreement. This is the layer connecting metaverse to universe, code to law. Everything that has been implemented, used this layer as a list of requirements.
2. Smart Contracts: this is the code implementing the law, running on EVMOS.
3. Content Delivery: we use The Graph to collect and structure our on-chain information, for easy and expedite access.
4. Clients:
   1. Mailer: a cloudfront worker sending out emails to the relevant DAO participant about legally sensitive events
   2. Dapp: this is the access point for the Contributors of the DAO and the general public, willing to look at its development

![Architecture](https://raw.githubusercontent.com/TelediskoDAO/docs/main/architecture.png)

## Shares and Permissions
Point `2.` of the SHA describes in aboundant detail the process of joining the DAO as Shareholder and as a Contributor. What ultimately discriminates a DAO Shareholder from any other person is described in point `2.1.3`

> `Upon accepting a person as a new shareholder of DAO, DAO shall insert the person into the shareholders' register and gift or sell such person 1 DAO share with the nominal value of EUR 1;`

The possession of this nominal share is what ultimately defines the rights of the Shareholder with respect to the DAO: voting rights, dividend rights, etc.

The logic is implemented by the [`ShareholderRegistry`](https://github.com/TelediskoDAO/contracts/tree/main/contracts/ShareholderRegistry), which is the single point of truth when it comes to the DAOs permission management.

Shares are managed by an ERC20 contract, extended to comply with the points of the Shareholder Agreements. For instance:

* every address is allowed to own max 1 share
* shares can only be transferred after a DAO vote (which implies that only an automatically executed resolution can trigger `transferFrom`)

The ownership of a share is the pre-condition to be have any of the statuses available to the DAO

* Investor
* Sharholed
* Contributor
* Managing Board

The contract also provides some utility methods that the other contracts can use to, for instance, undestand whether an account can vote.

The usage of tokens to deal with the shares was not explicitely required by the legal documentation. It was a technical decision that allowed us to more easily implement the logic by re-using existing compoents. It also allowed the implementation of the Smart Contract to match more closely the specific words of the SHA.

## Tokenomics
The tokenomics of this DAO is probably what distinguish it from most of the existing ones. As its founder said, 

> The system is socialist on the inside, capitalist on the outside.

The way tokens are managed inside the DAO makes sure no speculative or profit-maximising behaviour is incentivized. 

On the other side, once the tokens "leave the DAO", they become free to be traded as widly and heartlessly as it gets.

Chapters `4.` and `10.` of the SHA provide all the necessary information about what the tokens of teledisko represent and how they should be regulated.

The logic has been implemented with the [`TelediskoToken`](https://github.com/TelediskoDAO/contracts/tree/main/contracts/TelediskoToken). This is a beefed-up ERC20 Smart Contract.

"Beefed-up" in the sense that we needed to implement different levels of "freedom" for the tokens (remember when we were talking aobut "socialistic on the inside, capitalistic on the inside"?).

All tokens that are not owned by Contributors (information available in the `ShareholderRegistry`) are free to move.

For the others, we implemented a basic order book functionality within the token itself:

* Contributors are able to offer their tokens to other contributor.
* Other contributors are able to receive offered tokens.
* Tokens are locked until 7 days after the first offer.

All of this easily visualizable (and doable) from our Dapp

![Token Page](https://raw.githubusercontent.com/TelediskoDAO/docs/tech-article/tokens_page.png)

Currently, the acceptance of the offer and the transfer of the monetary amount to the token-holder is done more or less manually (a multisig wallet changes the state contract, Euros are transferred via bank-wire). But we are working to integrate EEUR in our ecosystem and have a fully automated escrow mechanism inside the Smart Contracts.

We used ERC20 even in this case, because (on top of the same reasons expressed for the ShareholderRegistry), the legal documents explicitly talk mention `tokens` as the legal tender of the DAO.

## Governance and Snapshotting

The governance of the DAO is regulated throughout the whole SHA and AoA, with legal obligations related to the proposal and execution of the resolutions.

Among the many points that we implemented in the DAO, one of them had a quite pervasive implication in the architecture of the Smart Contracts, namely `7.4` of the SHA:

> `The number of tokens being taken into account for a DAO vote and their allocation between Shareholders will be fixed on a day the DAO vote is announced. Subsequent transactions with tokens are not taken into account.`

In simple words: the voting power of each Contributor of the DAO depends on the time the resolution has been created. So if A had 42 tokens on 22nd September 2022, the Draft Resolution #9 is approved on 23nd September 2022 and A transfers 3 tokens afterward, A will still have a voting power of 42 for Resolution #9.

Same applies for delegation, demotion (when a Contributor is removed from the DAO), etc. Namely: if A delegated B on 22nd September 2022, the Draft Resolution #9 is approved on 23nd September 2022 and A redelegates C afterward, B will still be delegated for Resolution #9.

Basically most of the state of the DAO had to be *snapshottable*, with snapshots taken every time a draft resolution is approved by the managing board.

![Unnecessary but street-credits worth meme](https://raw.githubusercontent.com/TelediskoDAO/docs/tech-article/meme.png)

The base logic is implemented in [`Snapshottable`](https://github.com/TelediskoDAO/contracts/blob/main/contracts/extensions/Snapshottable.sol), that is inherited by all the contracts of the DAO.

Each contract has then, for each relevant state-reading function, a default version and an `at` version. For instance, TelediskoToken has  `balanceOf` and `balanceOfAt`, the former used to know the balance of an address for a given resolution id.

The logic was inspired by the [OpenZeppelin `ERC20Snapshot` contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Snapshot.sol), extended to support our state versionining requirements.

## Content Delivery: The Graph

The state of our DAO is tracked by two main entities:

* IPFS: for the more verbose type of content, such as the resolution texts
* Smart Contracts: for everything else

As some of you probably already know, Smart Contracts are not the best "content delivery" service that a dapp could possibly have.

For simple dapps, maybe it's sufficient to interact directly with the Smart Contract (for instance if we only need to display the current balance of ERC20 token holder), but for more complex use cases, there is a complication: the query interface. A Smart Contract is not a DB engine and has such, it offers very limited data access capabilities. The developer has to know ahead of time all the needed access patterns in order to provide functions able to serve that. Imagine having to join multiple data sets or aggregate/filter a long list of data points: would you really like to implement this logic in the Smart Contract?

To address this issue, we decided to leverage [The Graph](https://thegraph.com/en/), *an indexing protocol for querying networks like Ethereum and IPFS.*

What The Graph does is to listen to all events coming from a specific Smart Contract and collect its data in a structured format, ready to be queried via GraphQL. The indexing logic can be customized at will.

If the Smart Contract events refer to immutable data (such as anything hosted on IPFS), it's also possible to include that inside the data model.

It's pretty handy as it allows a certain degree of freedom in the creation of the dapp and any other read-only functionality related to the DAO.

For querying, it's possible to use the so-called **Graph Studio**, which is the decentralized network of indexers and curators that offers its data delivery service for a small fee (to be paid in GRT, Graph Token).

Given that EVMOS is currently not supported though, we had to spin-up our own indexer and IPFS node. We hope it's only a temporary solution.

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

![Rejected Resolutions](https://raw.githubusercontent.com/TelediskoDAO/docs/tech-article/rejected.png)

If you need help setting up your graph node with IPFS for your EVMOS project, feel free to get in touch with us!

## Upgradeability



## Incremental Decentralization

- temporary permissions to operator role
- resolutions could already be automated (with call data field)

## Next steps: EEUR