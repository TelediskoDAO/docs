# TelediskoDAO: a technical overview
## Intro and Vision
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

Once the first draft of the [Article of Association](https://github.com/TelediskoDAO/legal/blob/main/AoA.md)and the [Shareholders' Agreement (SHA)](https://github.com/TelediskoDAO/legal/blob/main/SHA.md) has been completed, we started developing the Smart Contracts and the dApp on top.

The main challenge for the development of this first implementation of the DAO has been the connection between law and code.

When we needed to start developing the Smart Contracts, we actually decided to spend the first weeks thoroughly studying the Article of Association and the Shareholder Agreement. The main point was to get to a very clear understanding of how the law could be automated with code, and this required quite a few iterations with the lawyers.

## General Architecture
The system architecture has 4 layers:

* Legal: Article of Association and Shareholder Agreement. This is the layer connecting metaverse to universe, code to law. Everything that has been implemented, used this layer as a list of requirements.
* Smart Contracts: this is the code implementing the law, running on EVMOS.
* Content Delivery: we use The Graph to collect and structure our on-chain information, for easy and expedite access.
* Mailer: a cloudfront worker sending out emails to the relevant DAO participant about legally sensitive events
* Dapp: this is the access point for the Contributors of the DAO and the general public, willing to look at its development
![Architecture](https://raw.githubusercontent.com/TelediskoDAO/docs/main/architecture.png)

## Shares and Permissions
Point `2.` of the SHA describes in aboundant detail the process of joining the DAO as Shareholder and as a Contributor. What ultimately discriminates a DAO Shareholder from any other person is described in point `2.1.3`

```
Upon accepting a person as a new shareholder of DAO, DAO shall insert the person into the shareholders' register and gift or sell such person 1 DAO share with the nominal value of EUR 1;
```

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

## Voting
## Content Delivery: The Graph
## Upgradeability
## Incremental Decentralization
## Next steps: EEUR