<div align="center">
<h1> Guild SDK for TypeScript | WIP </h1>
<a href="https://www.npmjs.com/package/@guildxyz/sdk"><img src="https://img.shields.io/npm/v/prisma.svg?style=flat" /></a>
  <a href="https://github.com/agoraxyz/guild-sdk/blob/main/CONTRIBUTING.md"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" /></a>
  <a><img src="https://img.shields.io/badge/license-MIT-blue" /></a>
  <br/>
  <a href="https://guild.xyz">Application</a>
  <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://twitter.com/guildxyz">Twitter</a>
  <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://docs.guild.xyz/guild/">Docs</a>
  <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://roadmap.guild.xyz/">Community Roadmap</a>
  <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/agoraxyz">Github</a>
  <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://discord.gg/guildxyz">Discord</a>
</div>
  
  
  
## Summary
The Guild SDK library is a Typescript library for interacting with the Guild API. This document explains how to authenticate, manage your Guilds easily through the SDK. Developed and maintained by the @agoraxyz team.

#### Node.js

To install our SDK on Node.js, open your terminal and run:

```
npm i @guildxyz/sdk
```

#### Importing the package and specific types

```typescript
import { guild, role, user } from "@guildxyz/sdk";

await guild.get(1); // Get Guild by ID (detailed)
await guild.get("sismo-dao"); // Get Guild by url name (detailed)
await guild.getAll(); // Get All Guilds basic information
await guild.getUserAccess(1, "0xedd9C1954c77beDD8A2a524981e1ea08C7E484Be"); // Access checking for an address for a specific Guild
await guild.getUserMemberships(1, "0xedd9C1954c77beDD8A2a524981e1ea08C7E484Be"); // User current memberships for the given Guild
await guild.create(guildParams, wallet); // Create a guild with specific params - check the example below
await guild.update(1, guildParams, wallet); // Update a guild with the given params
await guild.delete(1, wallet); // Remove a guild by ID

await role.get(1); // Get Role by ID
await role.create(roleParams, wallet); // Create a role for an existing Guild
await role.update(1, wallet); // Update a role with the given params
await role.delete(1, wallet); // Remove a role by ID

await user.join(1, wallet); // Enables to join a user to the accessible roles in a Guild
await user.getMemberships("0xedd9C1954c77beDD8A2a524981e1ea08C7E484Be"); // Returns every Guild and Role of a given user
```

#### Browser

You can create an index.html file and include our SDK with:

```html
<script src="https://cdn.jsdelivr.net/npm/@guildxyz/sdk"></script>
```

#### Quick Start flow from Create Guild to Access Check and Join

```typescript
import { guild, CreateGuildResponse, GetUserAccessResponse } from "@guildxyz/sdk";
import { ethers } from "ethers";


// Creating a Guild
const myGuild: CreateGuildResponse = await guild.create(
  {
    name: "My New Guild",
    description: "Cool stuff",                                            // Optional
    imageUrl: "",                                                         // Optional
    theme: [{ mode: "DARK", color: "#000000" }],                          // Optional
    roles: [
      {
        name: "My First Role",
        logic: "AND",
        requirements: [
          {
            type: "ALLOWLIST",
            data: {
              addresses: [
                "0xedd9C1954c77beDD8A2a524981e1ea08C7E484Be",
                "0x1b64230Ad5092A4ABeecE1a50Dc7e9e0F0280304",
              ],
            },
          },
        ],
      },
      {
        name: "My Second Role",
        logic: "OR",
        requirements: [
          {
            type: "ERC20",
            chain: "ETHEREUM",
            address: "0xf76d80200226ac250665139b9e435617e4ba55f9",
            data: {
              amount: 1,
            },
          },
          {
            type: "ERC721",
            chain: "ETHEREUM",
            address: "0x734AA2dac868218D2A5F9757f16f6f881265441C",
            data: {
              amount: 1,
            },
          },
        ],
      },
    ],
  },
  ethers.Wallet.createRandom() // You have to insert your own wallet here
);



// Access checking for 0xedd9C1954c77beDD8A2a524981e1ea08C7E484Be
const userAccesses: GetUserAccessResponse[] = await guild.getUserAccess(myGuild.id, "0xedd9C1954c77beDD8A2a524981e1ea08C7E484Be");


// Joining to a Guild if the access check includes a Role, which accessible by the given address
if(userAccesses.some(uA => uA.access);){
    await user.join(myGuild.id, wallet);
    // If the Guild has no platform, the join successfully happens, if the given address is eligible
}
```

## Authentication Overview

One of the most common problems with digital signature-based authentication systems is the replay attack. We have developed a new authentication method against this vulnerability, which both ensures the integrity of the request independent of TLS encapsulation (HTTPS) and protects against replay based attacks. This ensures protection from the signature service (Wallet client) all the way to the API.

## Example for Authentication only

```typescript
import { prepareRequest } from "@guildxyz/sdk";
import { ethers } from "ethers";

const wallet = ethers.Wallet.createRandom(); // You have to insert your own wallet here

//Prepare request without payload
await prepareRequest(wallet);

// {"payload":{},"validation":{"address":"0xea66400591bf2485907749f71615128238f7ef0a","addressSignedMessage":"0xddc0d710043a232b430a3678d76367489b8f6c329e27e81795e75efb4744289034fdc4f7284e37b791609b0e1d76bf9a1837db2a3adf158e31a37ac6c91656511c","nonce":"0x26bb7d4c941aec37b239dbf6850e149faace8df740809c8f989c270f2a543c51","random":"wrETMso/e9YiMloSSeEusgMuoaVirTuIPfkzYGkDv7w=","timestamp":"1646265565126"}}

//Prepare request with payload
await auth.prepareRequest({ guildId: 1234 });
// {"payload":{"guildId":1234},"validation":{"address":"0xea66400591bf2485907749f71615128238f7ef0a","addressSignedMessage":"0x544855fc7c34b2411d74b45395ae59e87b6be10c15598a12446f3b0b0daf25f501ad8532a6420f9c8288724df2e03c14068786260a2eaaa9938e31318034fe1b1b","hash":"0xd24a3714283ef2c42428e247e76d4afe6bb6f4c73b10131978b877bc78238aa9","nonce":"0x3c3b72ba441b2740682d8974d96df2f61f3b9d49235d97ff6d5fd50373b2429c","random":"vrCxwqgt0ml9bF9z3Pxg9j9te1v0VU/9Yx9oFkfm84k=","timestamp":"1646267441728"}}
```
