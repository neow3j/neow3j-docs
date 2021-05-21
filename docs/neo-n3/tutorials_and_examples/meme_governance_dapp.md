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

There are two contracts, the `MemeContract` and the `MemeGovernance`. The `MemeGovernance` is the owner of the `MemeContract` and as such is the only entitled entity that can change the state of the `MemeContract`.

> The contracts are deployed on the Neo N3 Test Net (RC2) with the following addresses:
> - **MemeContract:** _3b4572c517d08a8ba46632cc35f45f4ca8081a01_
> - **MemeGovernance:** _43000f84c46df29e25d58a089d5564dbe23c15bc_

The `MemeGovernance` has a built-in voting mechanism, so that every change on the `MemeContract` has to pass a vote.

### Specification MemeGovernance

**getMemeContract**

```json
{
  "name": "getMemeContract",
  "safe": true,
  "parameters": [],
  "returntype": "Hash160"
}
```
Returns the address of the underlying `MemeContract`.

**getVotingTime**

```json
{
  "name": "getVotingTime",
  "safe": true,
  "parameters": [],
  "returntype": "Integer"
}
```
Returns the timeframe (number of blocks) to vote for a proposal.

**getMinVotesInFavor**

```json
{
  "name": "getMinVotesInFavor",
  "safe": true,
  "parameters": [],
  "returntype": "Integer"
}
```
Gets the minimum votes in favor for a proposal to be accepted.

**proposeNewMeme**

```json
{
  "name": "proposeNewMeme",
  "safe": false,
  "parameters": [
      {
          "name": "memeId",
          "type": "ByteArray"
      },
      {
          "name": "description",
          "type": "String"
      },
      {
          "name": "url",
          "type": "String"
      },
      {
          "name": "imageHash",
          "type": "String"
      }
  ],
  "returntype": "Void"
}
```
Creates a proposal to add a new meme.

- `memeId`: The id of the new meme. It can be any byte array.
- `description`: A short description of the meme.
- `url`: The image url that points directly to the meme file.
- `imageHash`: The SHA-256 hash of the meme file found under the provided url.

> Requires:
> - no open (voting in progress) proposal with the same `memeId`.
> - no closed **and** accepted proposal with the same `memeId`. (A closed proposal that was not accepted can be overwritten.)
> - no meme with the same `memeId`.

**proposeRemoval**

```json
{
  "name": "proposeRemoval",
  "safe": false,
  "parameters": [
      {
          "name": "memeId",
          "type": "ByteArray"
      }
  ],
  "returntype": "Void"
}
```
Creates a proposal to remove an existing meme.

- `memeId`: The id of an existing meme to be removed.

> Requires:
> - an existing meme with the same `memeId`.

**vote**

```json
{
  "name": "vote",
  "safe": false,
  "parameters": [
      {
          "name": "memeId",
          "type": "ByteArray"
      },
      {
          "name": "voter",
          "type": "Hash160"
      },
      {
          "name": "inFavor",
          "type": "Boolean"
      }
  ],
  "returntype": "Void"
}
```

- `memeId`: The id of the meme that this proposal is about.
- `vote`: The voter's script hash.
- `inFavor`: True to vote in favor and false to vote against a proposal.

Vote for a proposal (in favor or against).

> Requires:
> - an open proposal for the given `memeId`.

**execute**

```json
{
  "name": "execute",
  "safe": false,
  "parameters": [
      {
          "name": "memeId",
          "type": "ByteArray"
      }
  ],
  "returntype": "Boolean"
}
```
- `memeId`: The meme id the proposal was about.

Executes a finished proposal. If the proposal was about to create a meme, the meme with its properties is created on the `MemeContract`. If the proposal was about removing a meme, the meme is removed from the `MemeContract`.

> **Note**: If the proposal was not accepted, it's just removed.

> **Requires:**
> - a closed proposal for the specified `memeId`.

**getProposals**

```json
{
  "name": "getProposals",
  "safe": true,
  "parameters": [
      {
          "name": "startingIndex",
          "type": "Integer"
      }
  ],
  "returntype": "Array"
}
```
Gets the proposals.

### Specification MemeContract

**getMeme**

```json
{
  "name": "getMeme",
  "safe": true,
  "parameters": [
    {
      "name": "memeId",
      "type": "ByteArray"
    }
  ],
  "returntype": "Array"
}
```
Returns the meme properties of the specified meme id. (see )

**getMemes**

```json
{
  "name": "getMemes",
  "safe": true,
  "parameters": [
    {
      "name": "startingIndex",
      "type": "Integer"
    }
  ],
  "returntype": "Array"
}
```
Returns a list of existing memes.

> **Note:** The returned list has a maximum length limit of N.

- `startingIndex`: the starting index of the list to be returned. To get the first N memes, use 0 as ´startingIndex´. If there exist more than N memes, use a multiple of N as ´startingIndex´.

### Additional notes

Both contracts are linked to each other upon an initialization invocation after deployment. The contracts are first deployed separately. To link them, the creator invokes the method `initialize` on the `MemeGovernance` contract with the address of the `MemeContract` as parameter. This method executes the following:

- Sets the owner on the `MemeContract` to the address of the `MemeGovernance`.
- Sets the memeContract to the provided address on the `MemeGovernance`.
- Sets the owner of the `MemeGovernance` to the zero address.

### Meme and Proposal Structure

The properties of a meme are passed in an array of the following structure:

```json
{
    "type": "Array",
    "value": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "description",
            "type": "String"
        },
        {
            "name": "url",
            "type": "String"
        },
        {
            "name": "imageHash",
            "type": "String"
        }
    ]
}
```

A proposal is returned as an array of the following structure:
```json
{
    "type": "Array",
    "value": [
        {
            "name": "meme",
            "type": "Array" // Meme Array as above
        },
        {
            "name": "create", // whether the proposal is about to create a new meme or remove an existing one.
            "type": "Boolean"
        },
        {
            "name": "voteInProgress",
            "type": "Boolean"
        },
        {
            "name": "finalizationBlock",
            "type": "Integer"
        },
        {
            "name": "votesInFavor",
            "type": "Integer"
        },
        {
            "name": "votesAgainst",
            "type": "Integer"
        }
    ]
}
```

## Frontend dApp

TBD.
