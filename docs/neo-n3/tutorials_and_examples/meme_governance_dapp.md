# Meme Governance dApp

This project should demonstrate how a decentralized application (dApp) could be built on the Neo blockchain, using Java and React/NextJS.

The dApp implements a governance protocol, where users can:

- Create a proposal to add a new meme.
- Create a proposal to remove an existing meme.
- Vote on an existing proposal (either to add or remove) within a specified timeframe.
- Execute a proposal that was accepted in the vote.
- Get the currently persisted memes.

> Note: Any user holding GAS can participate in the Meme Governance Protocol.

## Contracts

The GitHub repo with the Smart Contracts presented in this section can be found here:

> https://github.com/AxLabs/meme-governance-contracts

There are two contracts, the `MemeContract` and the `MemeGovernance`. The `MemeGovernance` is the owner of the `MemeContract` and as such is the
only entitled entity that can change the state of the `MemeContract`.

> The contracts are deployed on the Neo N3 Test Net with the following addresses:
> - **MemeContract:** _8cdad4b33692fb3e4d16d8ae0ec4e5f5324c702a_
> - **MemeGovernance:** _44588563c5a96a9d92c5b698e796c2eea7c99f0a_

The `MemeGovernance` has a built-in voting mechanism, so that every change on the `MemeContract` has to pass a vote. Users can vote in favor or
against a proposal. For a proposal to be accepted, the following criteria must be met:
- the voting time frame needs to be over (see [getVotingTime](#getVotingTime)).
- the proposal needs a minimum of votes in favor (see [getMinVotesInFavor](#getMinVotesInFavor)).
- the proposal needs to have more votes in favor than against.

When a proposal passes its vote, it can be executed (see [execute](#execute)) and thus persisted on the ´MemeContract´.

### Specification MemeGovernance

#### getMemeContract

```javascript
{
  "name": "getMemeContract",
  "safe": true,
  "parameters": [],
  "returntype": "Hash160"
}
```
Returns the address of the underlying `MemeContract`.

#### getVotingTime

```javascript
{
  "name": "getVotingTime",
  "safe": true,
  "parameters": [],
  "returntype": "Integer"
}
```
Returns the timeframe (number of blocks) to vote for a proposal after it was created.

#### getMinVotesInFavor

```javascript
{
  "name": "getMinVotesInFavor",
  "safe": true,
  "parameters": [],
  "returntype": "Integer"
}
```
Gets the minimum votes in favor for a proposal to be accepted.

#### proposeNewMeme

```javascript
{
  "name": "proposeNewMeme",
  "safe": false,
  "parameters": [
      {
          "name": "memeId", // A unique id/name for this new meme.
          "type": "String"
      },
      {
          "name": "description", // A description of the meme.
          "type": "String"
      },
      {
          "name": "url", // An image url that points directly to the meme file.
          "type": "String"
      },
      {
          "name": "imageHash", // The SHA-256 hash of the meme file found under the provided url.
          "type": "String"
      }
  ],
  "returntype": "Void"
}
```
Creates a proposal to add a new meme.

**Requirements:**
- no open (voting in progress) proposal with the same `memeId`.
- no closed **and** accepted proposal with the same `memeId`. (A closed proposal that was not accepted can be overwritten.)
- no meme with the same `memeId`.

#### proposeRemoval

```javascript
{
  "name": "proposeRemoval",
  "safe": false,
  "parameters": [
      {
          "name": "memeId", // The id/name of an existing meme to be removed.
          "type": "String"
      }
  ],
  "returntype": "Void"
}
```
Creates a proposal to remove an existing meme.

**Requirements:**
- an existing meme with the same `memeId`.

#### vote

```javascript
{
  "name": "vote",
  "safe": false,
  "parameters": [
      {
          "name": "memeId", // The id/name of the meme that this proposal is about.
          "type": "String"
      },
      {
          "name": "voter", // The voter's script hash.
          "type": "Hash160"
      },
      {
          "name": "inFavor", // True to vote in favor and false to vote against a proposal.
          "type": "Boolean"
      }
  ],
  "returntype": "Void"
}
```
Vote for a proposal (in favor or against).

**Requirements:**
- an open proposal for the given `memeId`.
- the voter is required to be a signer (with called by entry scope).

#### execute

```javascript
{
  "name": "execute",
  "safe": false,
  "parameters": [
      {
          "name": "memeId", // The meme id/name that the proposal was about.
          "type": "String"
      }
  ],
  "returntype": "Boolean"
}
```
Executes a finished proposal. If the proposal was about to create a meme, the meme with its properties is created on the `MemeContract`. If the
proposal was about removing a meme, the meme is removed from the `MemeContract`.

> **Note**: If the proposal was not accepted, it's removed.

**Requirements:**
- a closed proposal for the specified `memeId`.

#### getProposals

```javascript
{
  "name": "getProposals",
  "safe": true,
  "parameters": [
      {
          "name": "startingIndex", // The first index in the iterator on the contract.
          "type": "Integer"
      }
  ],
  "returntype": "Array"
}
```
Gets a list of proposals that have not been executed. The returned array holds the proposals in the structure explained in detail [below](#meme-and-proposal-structure).

> **Note:** The returned list size is limited. This method is intended to be used by RPCs, since those are not made for processing large data,
> the deployed contract has a limit of 100 entries in the returned array. If the contract holds more than 100 proposals, you can get the data by
> making multiple calls and increasing `startingIndex` by 100 for each RPC. E.g. use 0 to get the first 100 proposals.

### Specification MemeContract

#### getMeme

```javascript
{
  "name": "getMeme",
  "safe": true,
  "parameters": [
    {
      "name": "memeId", // The id/name of the meme.
      "type": "String"
    }
  ],
  "returntype": "Array"
}
```
Returns the meme properties of the specified meme id. The returned array is explained in detail [below](#meme-and-proposal-structure).

#### getMemes

```javascript
{
  "name": "getMemes",
  "safe": true,
  "parameters": [
    {
      "name": "startingIndex", // The first index in the iterator on the contract.
      "type": "Integer"
    }
  ],
  "returntype": "Array"
}
```
Gets a list of existing memes. The returned array holds the memes in the structure explained in detail [below](#meme-and-proposal-structure).

> **Note:** The starting index is handled the same way as in the function `getProposals` (see [above](#getProposals)).

### Additional notes

Both contracts are linked to each other upon deployment of the governance contract. The contracts are both deployed separately. The meme
contract is deployed first and then the governance contract is deployed with the meme contract's hash as data parameter. Upon deploying the
governance contract, the following steps are executed:

- The owner on the `MemeContract` is set to the address of the `MemeGovernance`.
- The memeContract is set on the `MemeGovernance`.

> **Note:** You can check whether the meme contract is initialized by calling the function `getOwner` on it and then `getMemeContract` on
> the returned address. The contracts are correctly linked, if the owner of the `MemeContract` is the address of the `MemeGovernance` contract.

### Meme and Proposal Structure

The properties of a meme are passed in an array of the following structure:

```javascript
{
    "type": "Array",
    "value": [
        {
            "name": "id", // The unique id/name of the meme.
            "type": "String"
        },
        {
            "name": "description", // The description of the meme.
            "type": "String"
        },
        {
            "name": "url", // The url of the meme.
            "type": "String"
        },
        {
            "name": "imageHash", // The sha-256 hash of the image of the above url.
            "type": "String"
        }
    ]
}
```

A proposal is returned as an array of the following structure:
```javascript
{
    "type": "Array",
    "value": [
        {
            "name": "meme", // Reference to the meme that this proposal refers to (structure as above).
            "type": "Array"
        },
        {
            "name": "create", // Whether the proposal is about to create or remove the above meme.
            "type": "Boolean"
        },
        {
            "name": "voteInProgress", // True, if this proposal can be voted for, false, if the voting is closed.
            "type": "Boolean"
        },
        {
            "name": "finalizationBlock", // The last block number that accepts any vote.
            "type": "Integer"
        },
        {
            "name": "votesInFavor", // The number of votes in favor of the proposal.
            "type": "Integer"
        },
        {
            "name": "votesAgainst", // The number of votes against the proposal.
            "type": "Integer"
        }
    ]
}
```

## dApp: Frontend

The GitHub repo with the frontend presented in this section can be found here:

> https://github.com/AxLabs/meme-governance-frontend

The frontend was built with [NextJS](https://nextjs.org/), and it's composed of:

* **Landing Page:** general information about the Meme Governance dApp
  * Served on the `/` path, e.g., `http://localhost:8080/`
* **dApp:** the actual dApp implementation, where all actions are exposed to users
  * Served on the `/dapp` path, e.g., `http://localhost:8080/dapp`
  * 🚀 **Full integration to [NeoLine](https://neoline.io/en/) browser wallet**

This project is meant to be used as a boilerplate to bootstrap projects in a fast and easy way.

### Getting Started

Run the following commands on your local environment:

```shell
git clone --depth=1 https://github.com/AxLabs/meme-governance-frontend.git my-project-name
cd my-project-name
npm install
```

If you're a developer, and want to run locally in **development mode with live reload**:

```shell
npm run dev
```

If you want to create an **optimized production build**, then execute:

```shell
npm run build-prod
```

To serve the optimized production build, run:

```shell
npm run start
```
