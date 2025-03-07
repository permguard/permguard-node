# Permguard Node.js SDK

[![GitHub License](https://img.shields.io/github/license/permguard/permguard-node)](https://github.com/permguard/permguard-node?tab=Apache-2.0-1-ov-file#readme)
[![X (formerly Twitter) Follow](https://img.shields.io/twitter/follow/permguard)](https://x.com/intent/follow?original_referer=https%3A%2F%2Fdeveloper.x.com%2F&ref_src=twsrc%5Etfw%7Ctwcamp%5Ebuttonembed%7Ctwterm%5Efollow%7Ctwgr%5ETwitterDev&screen_name=Permguard)

[![Documentation](https://img.shields.io/website?label=Docs&url=https%3A%2F%2Fwww.permguard.com%2F)](https://www.permguard.com/)
[![Build, test and publish the artifacts](https://github.com/permguard/permguard-node/actions/workflows/permguard-node-ci.yml/badge.svg)](https://github.com/permguard/permguard-node/actions/workflows/permguard-node-ci.yml)

<p align="left">
  <img src="https://raw.githubusercontent.com/permguard/permguard-assets/main/pink-txt//1line.svg" class="center" width="400px" height="auto"/>
</p>

The Permguard Node.js SDK provides a simple and flexible client to perform authorization checks against a Permguard Policy Decision Point (PDP) service using gRPC.
Please refer to the [Permguard Documentation](https://www.permguard.com/) for more information.

---

## Prerequisites

- **Node.js 20.x or later**
- **TypeScript 5.8 or later** (if using TypeScript)

---

## Installation

Run the following command to install the SDK:

```bash
npm install permguard-node
```

or with Yarn:

```bash
yarn add permguard-node
```

---

## Usage Example

Below is a sample TypeScript code demonstrating how to create a Permguard client, build an authorization request using a builder pattern, and process the authorization response:

```typescript
import {
  PrincipalBuilder,
  AZAtomicRequestBuilder,
  withEndpoint,
  AZClient,
} from "permguard-node";

// Create a new Permguard client
const azClient = new AZClient(withEndpoint("localhost", 9094));

// Create the Principal
const principal = new PrincipalBuilder("amy.smith@acmecorp.com").build();

// Create the entities
const entities = [
  {
    uid: {
      type: "MagicFarmacia::Platform::BranchInfo",
      id: "subscription",
    },
    attrs: {
      active: true,
    },
    parents: [],
  },
];

// Create a new authorization request
const req = new AZAtomicRequestBuilder(
  583438038653,
  "46706cb00ea248d6841cfe2c9f02205b",
  "platform-creator",
  "MagicFarmacia::Platform::Subscription",
  "MagicFarmacia::Platform::Action::create"
)
  .withRequestID("1234")
  .withPrincipal(principal)
  .withEntitiesItems("cedar", entities)
  .withSubjectRoleActorType()
  .withSubjectSource("keycloack")
  .withSubjectProperty("isSuperUser", true)
  .withResourceID("e3a786fd07e24bfa95ba4341d3695ae8")
  .withResourceProperty("isEnabled", true)
  .withActionProperty("isEnabled", true)
  .withContextProperty("time", "2025-01-23T16:17:46+00:00")
  .withContextProperty("isSubscriptionActive", true)
  .build();

// Check the authorization
const { decision, response } = await azClient.check(req);
if (decision) {
  console.log("✅ Authorization Permitted");
} else {
  console.log("❌ Authorization Denied");
  if (response) {
    if (response.Context?.ReasonAdmin) {
      console.log(`-> Reason Admin: ${response.Context.ReasonAdmin.Message}`);
    }
    if (response.Context?.ReasonUser) {
      console.log(`-> Reason User: ${response.Context.ReasonUser.Message}`);
    }
    for (const evaluation of response.Evaluations || []) {
      if (evaluation.Context?.ReasonAdmin) {
        console.log(
          `-> Reason Admin: ${evaluation.Context.ReasonAdmin.Message}`
        );
      }
      if (evaluation.Context?.ReasonUser) {
        console.log(`-> Reason User: ${evaluation.Context.ReasonUser.Message}`);
      }
    }
  }
}
```

---

Created by [Nitro Agility](https://www.nitroagility.com/).
