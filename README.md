NIP-XX
======

Promoted Note Network
-----------------

`draft` `optional`

## Context
The "new" internet needs a new attention marketplace. Individual clients/apps/companies shouldn't have to build out an internal ad-tech in order to monetize projects. Instead of advertising being based on centralization, NOSTR should provide a decentralized ad network for the benefit of the entire network. 

## Terms
- `KEYPAIR` - A NOSTR key-pair

### ACTORS
- **BUYER** - A `KEYPAIR` with specific kind 0 metadata who wants to 'buy' attention from the marketplace. publishes `PROMOTED NOTE` events
- **SELLER** - A `KEYPAIR` with specific kind 0 metadata who wants to 'sell' attention from the marketplace. publishes `IMPRESSION`, `ACTION`
- **CLIENT** - A NOSTR client i.e. damus, ametheyst, etc.
- **MATCHER** - A service(DVM) that matches `BUYERS` and `SELLERS` and publishes `MATCH` events.
- **IMPRESSOR** - A NOSTR `CLIENT` that displays `MATCH` events to `SELLERS`
- **CONVERTER** - A NOSTR `CLIENT` that shows the full promoted note and ensures that the `SELLER` waited the corrrect duration before triggering the `PAYOUT` and publishing a `PAYOUT` event. 

### NEW EVENT TYPES
- `PROMOTED NOTE` - A note that pays you
- `MATCH` - When a `BUYER` is willing to pay equal to or greater than what a `SELLER` is asking
- `IMPRESSION` - `SELLER` views preview of `PROMOTED NOTE` and publishes an IMPRESSION event
- `ACTION` - Task for the SELLER to complete relative to the `PROMOTED NOTE`. (default click)
- `CONVERSION` - When a `SELLER` consumes `PROMOTED NOTE`
- `PAYOUT` - who got paid after `CONVERSION`

## INCENTIVES
Each particiant in the network has a different but complimentary set of incentives to keep the attention marketplace honest. 

- **BUYERS** are incentivised to place ads in the network because it is low stakes. If no one participates then the spend/lose no money. And they are incentivised to adjust their `PAYOUTS` if they find that people are participating, but they are not meeting their price threshold.
- **SELLERS** are incentivized to participate in the network because they are able to opt-in and opt-out at will and they are able to 'mine bitcon with their attention'. They also have complete control of their data since they are the KEYPAIR that publishes the `IMPRESSION` and the `ACTION` events.
- **MATCHERS** are incentivized to participate in the network because it a revenue stream for compute resources. Similar to home mining w/o asics. 
- **IMPRESSORS** are incentivized to participate in the network because it is a revenue stream and their users have opted in so there is no risk of revolt
- **CONVERTERS** are incentivized to participate in the network because they are ultimately who does the paying out. They are inentivised to check all relevant data to ensure that everything 'looks good' before initiating payout. Similar to bitcoin nodes validating blocks.

## CLIENTS
There are four(4) different types of `CLIENTS` for this process to work. Some client's will contain the functionality of all three but many will only contain one(1) or two(0)
- Seller Opt-in client: this client supports a user opting into the promoted note network. They will need to add the ability for a user to set their `attention_seller_*` fields
- Buyer Opt-in client: this client supports a user publishing a promoted note. They will also need to add the ability for a user to set their `attention_buyer_*` fields
- Preview client: this client supports notifying `SELLERS` there there are `PROMOTED NOTES` that match their `attention_seller_*` fields
- Converter client: this client supports showing `SELLERS` the full promoted note and is responsible for handling the `PAYOUTS`.

## FLOW
The basic flow of the `PROMOTED NOTE` network is as follows
1. `SELLER` signals they are willing to be shown `PROMOTED NOTE`s by updating their kind 0 event
2. `BUYER` publishes `PROMOTED NOTE` event
3. `MATCHER` requests all `SELLERS` and `PROMOTED NOTES` from relay(s), creates `MATCHES`, and publishes `MATCH` events to relay(s).
5. `CLIENT` pulls all `MATCH` events for a given `SELLER`
6. `CLIENT` confirms `SELLER` has all required data
7. `CLIENT` displays `PROMOTED NOTE`(s) preview to `SELLER`
8. `SELLER` publishes `IMPRESSION` event
9. IF `SELLER` completes `ACTION` of `PROMOTED NOTE` (default: click), `SELLER` publishes an `ACTION` event
10. `SELLER` is presented `PROMOTED NOTE`
11. After defined number of seconds viewing the `PROMOTED NOTE`, the `SELLER` publishes `CONVERSION` event
12. `CONVERTER` publishes `PAYOUT` event
13. `CONVERTER` pays the `SELLER`, `MATCHER`, and `IMPRESSOR` the amounts defined in the `PROMOTED NOTE`

## EVENTS
Below are a set of new EVENT kinds to facilitate implementing a decentralized ad netwrok.

### METADATA (kind 0)
In order to identify as a `BUYER` the following fields need to be added to the keypair's kind 0 event.

**Buyer**
  ```js
  {
    "tags": [
      ["e", "<event-id>"] // promoted content event
    ],
    "content": {
      ...,
      "attention_buyer_minimum_duration": 15, // optional
      "attention_buyer_maximum_duration": 60, // required
      "attention_buyer_minimum_price": 50, // optional
      "attention_buyer_maximum_price": 200, // required
      "attention_buyer_promoted_content": ["<event-id>", "<event-id>", "<event-id>", ...], // promoted content event
    }
  }
  ```
In order to identify as a `SELLER` the following fields need to be added to the keypair's kind 0 event.
**Seller**
  ```js
  {
    "tags": [
      ["buyer_whitelist", "<hex>"],
      ["buyer_blacklist", "<hex>" ],
    ],
    "content": {
      ...,
      "attention_seller_maximum_duration": 60, // required
      "attention_seller_minimum_price": 50, // required
      "attention_seller_buyer_whitelist": ["hex","hex"], // optional
      "attention_seller_buyer_blacklist": ["hex","hex"], // optional
    }
  }
  ```


### PROMOTED NOTE
This event is published by a `BUYER` when `PROMOTED NOTE` is added to the network.

#### TAGS

#### CONTENT
| Field                     | Required | Type   | Description |
|---------------------------|----------|--------|-------------|
| buyer_id                  | true     | hex    | The hex of the public key of owner of content |
| action                    | true     | string | [ click ] |
| type                      | true     | string | Type of promoted content [text / image / audio /video ] |
| payout_satoshis_seller    | true     | int    | How many sats are being offered to view content |
| payout_satoshis_matcher   | true     | int    | How many sats are being offered to whoever made the match that led to conversion |
| payout_satoshis_impressor | true     | int    | How many sats are being offered to whoever made the impression that led to conversion |
| seconds                   | true     | int    | Length of content in the case of audio or video. Duration of viewing in the case of text or image  |
| title                     | true     | string | Primary text that will be displayed when offering user promoted content |
| subtitle                  | false    | string | Secondary text that may be display when offering user promoted content  |
| description               | false    | string | Short description of promoted content that may be displayed  when offering user promoted content |
| preview_image_url         | true     | string | URL for image to use when offering user promoted content   |
| preview_image_orientation | true     | string | Aspect ratio of preview image [1:1 / 16:9 / 9:16 ]  |
| conversion_url            | true     | string | URL for where user should be navigated to if they accept offer  |
| seller_whitelist          | false    | hex[]  | Array of hex of public keys that will be allowed to view promoted content  |
| seller_blacklist          | false    | hex[]  | Array of hex of public keys that will not be allowed to view promoted content  |

```js
{
  "kind": "?"
  "tags": [
    ["type", "< text | image | audio | video >"]
    ["action", "< click >"],
    ["payout_satoshis_seller", "100"],
    ["payout_satoshis_matcher", "10"],
    ["payout_satoshis_impressor", "10"],
    ["seconds": "60"],
    ["buyer_id": "<hex>"],
    ["seller_whitelist", "<hex>"],
    ["seller_blacklist", "<hex>" ],
    ["preview_image_orientation", "<1:1 | 16:9 | 9:16 >"],
  ],
  "content": {
    "buyer_id": "<hex>"
    "type": "<text | image | audio | video>",
    "action": "< click >"
    "payout_satoshis_seller": 100,
    "payout_satoshis_matcher": 10,
    "payout_satoshis_impressor": 10,
    "seconds": 60
    "title": "",
    "subtitle": "",
    "description": "",
    "preview_image_url": "<url>",
    "preview_image_orientation": "<1:1 | 16:9 | 9:16 >",
    "conversion_url": "<url>",
    "seller_whitelist": ["hex", "hex"],
    "seller_blacklist": ["hex", "hex"],
  }
}
```

### MATCH
This event is published by `MATCHER` when a `CLIENT` matches a `SELLER` of attention with a `BUYER` of attention. 

This event **MUST** be published in order to be paid if a `CONVERSION` happens from this `MATCH`.

#### TAGS

#### CONTENT
| Field                     | Required | Type   | Description |
|---------------------------|----------|--------|-------------|
| buyer_id                  | true     | hex    | The hex of the public key of BUYER |
| seller_id                 | true     | hex    | The hex of the public key of SELLER |
| matcher_id                | true     | hex    | The hex of the public key of MATCH MAKER |
| content_id                | true     | hex    | The event_id of the PROMOTED CONTENT |

```js
{
  "kind": "?"
  "tags": [
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["matcher_id": "<hex>"],
    ["content_id": "<event-id>"],
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<match_maker_id>"],
    ["e", "<content_id>"],
  ],
  "content": {
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "matcher_id": "<hex>",
    "content_id": "<event-id>",
  }
}
```
  
### IMPRESSION
This event is published by the `SELLER` when a client displays a `MATCH` to a `SELLER`. 

This event **MUST** be published in order to be paid if a `CONVERSION` happens from this `MATCH`.

#### TAGS

#### CONTENT
| Field            | Required | Type   | Description |
|------------------|----------|--------|-------------|
| buyer_id         | true     | hex    | The hex of the public key of BUYER       |
| seller_id        | true     | hex    | The hex of the public key of SELLER      |
| matcher_id       | true     | hex    | The hex of the public key of MATCH MAKER |
| impressor_id     | true     | hex    | The hex of the public key of IMPRESSOR   |
| match_id         | true     | string | event_id of MATCH event                  |
| content_id       | true     | string | event_id of PROMOTED CONTENT event       |


```js
{
  "kind": "?"
  "tags": [
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["matcher_id": "<hex>"],
    ["content_id": "<event_id>"],
    ["match_id": "<event_id>"],
    ["impressor_id": "<hex>"]
    ["e", "<content_id>"]
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<matcher_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "matcher_id": "<hex>",
    "content_id": "<event-id>",
    "impressor_id": "<hex>"
  }
}
```

### ACTION
This event is published by `SELLER` after completing the desired `ACTION`. 

This event **MUST** be published in order to be paid if a `CONVERSION` happens from this `ACTION`.

#### TAGS

#### CONTENT
| Field            | Required | Type   | Description |
|------------------|----------|--------|-------------|
| buyer_id         | true     | hex    | The hex of the public key of `BUYER`       |
| seller_id        | true     | hex    | The hex of the public key of `SELLER`      |
| matcher_id       | true     | hex    | The hex of the public key of `MATCH MAKER` |
| impressor_id     | true     | hex    | The hex of the public key of `IMPRESSOR`   |
| match_id         | true     | string | event_id of `MATCH` event                  |
| content_id       | true     | string | event_id of `PROMOTED CONTENT` event       |
| action           | true     | string | [ click ] |


```js
{
  "kind": "?"
  "tags": [
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["matcher_id": "<hex>"],
    ["content_id": "<event_id>"],
    ["match_id": "<event_id>"],
    ["impressor_id": "<hex>"],
    ["action", "< click >"],
    ["e", "<content_id>"],
    ["e", "<match_id>"]
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<matcher_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "matcher_id": "<hex>",
    "content_id": "<event_id>",
    "impressor_id": "<hex>",
    "match_id": "<event_id>",
    "action": "< click >",
  }
}
```

### CONVERSION
This event is published when a `SELLER` 'consumes' the `PROMTED NOTE` for the desired number of seconds
```js
{
  "kind": "?"
  "tags": [
    ["content_id": "<event_id>"],
    ["match_id": "<event_id>"],
    ["impression_id": "<event_id>"],
    ["action_id": "<event_id>"],
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["matcher_id": "<hex>"],
    ["impressor_id": "<hex>"],
    ["e", "<content_id>"],
    ["e", "<match_id>"],
    ["e", "<impression_id>"],
    ["e", "<action_id>"],
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<matcher_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "content_id": "<event_id>",
    "match_id": "<event_id>",
    "impression_id": "<event_id>",
    "action_id": "<event_id>",
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "matcher_id": "<hex>",
    "impressor_id": "<hex>",
  }
}
```

### PAYOUT
This event is published when a `CONVERTER` paysout the `SELLER`, `MATCHER` and `IMPRESSOR`
```js
{
  "kind": "?"
  "tags": [
    ["payout_satoshis_seller", "100"],
    ["payout_satoshis_matcher", "10"],
    ["payout_satoshis_impressor", "10"],
    ["content_id": "<event_id>"],
    ["match_id": "<event_id>"],
    ["impression_id": "<event_id>"],
    ["action_id": "<event_id>"],
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["matcher_id": "<hex>"],
    ["impressor_id": "<hex>"],
    ["e", "<content_id>"],
    ["e", "<match_id>"],
    ["e", "<impression_id>"],
    ["e", "<action_id>"],
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<matcher_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "payout_satoshis_seller": 100,
    "payout_satoshis_matcher": 10,
    "payout_satoshis_impressor": 10,
    "content_id": "<event_id>",
    "match_id": "<event_id>",
    "impression_id": "<event_id>",
    "action_id": "<event_id>",
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "matcher_id": "<hex>",
    "impressor_id": "<hex>",
  }
}
```

