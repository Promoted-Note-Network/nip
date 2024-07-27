# btc-ad-server

NIP-XX
======

Nostr Promoted Content (Ad) Network
-----------------

`draft` `optional`

## Context
The "new" internet needs a new attention marketplace. Individual clients/apps/companies shouldn't have to build out an internal ad-tech in order to monetize projects. Instead of advertising being based on centralization, NOSTR should provide a decentralized ad network for the benefit of the entire network. 

## Terms

- `BUYER` - A NOSTR key-pair with specific kind 0 metadata 
- `SELLER` - A NOSTR key-pair with specific kind 0 metadata
- `Promoted Content` - 
- `Match` - When a Buyer is willing to pay equal to or greater than what a Seller is asking
- `Impression`
- `Conversion` - When a seller consumes (reads/listens/watches) a piece of promoted content
- `Payout` - who gets paid after conversion
- `Client` - 
- `MATCHER` - 
- `IMPRESSOR`
- `ACTION`

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
      "attention_seller_maximum_duration": 60,
      "attention_seller_minimum_price": 50,
      "attention_seller_buyer_whitelist": ["hex","hex"],
      "attention_seller_buyer_blacklist": ["hex","hex"],
    }
  }
  ```


### PROMOTED CONTENT
This event is published by a `BUYER` when `PROMOTED CONTENT` is added to the network.

#### TAGS

#### CONTENT
| Field                     | Required | Type   | Description |
|---------------------------|----------|--------|-------------|
| buyer_id                  | true     | hex    | The hex of the public key of owner of content |
| action                    | true     | string | [ click / follow / retweet / reply / quote / zap ] |
| type                      | true     | string | Type of promoted content [text / image / audio /video ] |
| satoshis                  | true     | int    | How many sats are being offered to view content |
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
    ["action", "< click | follow | retweet | reply | quote | zap >"]
    ["satoshis", "120"],
    ["seconds": "60"],
    ["buyer_id": "<hex>"],
    ["seller_whitelist", "<hex>"],
    ["seller_blacklist", "<hex>" ],
    ["preview_image_orientation", "<1:1 | 16:9 | 9:16 >"],
  ],
  "content": {
    "buyer_id": "<hex>"
    "type": "<text | image | audio | video>",
    "action": "<click | follow | retweet | reply | quote | zap >"
    "satoshis": 120,
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
This event is published by `MATCHER` when a client matches a `SELLER` of attention with a `BUYER` of attention. This event **MUST** be published in order to be paid if a `CONVERSION` happens from this match.

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
    ["match_maker_id": "<hex>"],
    ["content_id": "<event-id>"],
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<match_maker_id>"],
    ["e", "<content_id>"],
  ],
  "content": {
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "match_maker_id": "<hex>",
    "content_id": "<event-id>",
  }
}
```
  
### IMPRESSION
This event is published by the `SELLER` when a client displays a `MATCH` to a `SELLER`. This event **MUST** be published in order to be paid if a `CONVERSION` happens from this `MATCH`.

#### TAGS

#### CONTENT
| Field            | Required | Type   | Description |
|------------------|----------|--------|-------------|
| buyer_id         | true     | hex    | The hex of the public key of BUYER       |
| seller_id        | true     | hex    | The hex of the public key of SELLER      |
| matcher_maker_id | true     | hex    | The hex of the public key of MATCH MAKER |
| impressor_id     | true     | hex    | The hex of the public key of IMPRESSOR   |
| match_id         | true     | string | event_id of MATCH event                  |
| content_id       | true     | string | event_id of PROMOTED CONTENT event       |


```js
{
  "kind": "?"
  "tags": [
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["match_maker_id": "<hex>"],
    ["content_id": "<event_id>"],
    ["match_id": "<event_id>"],
    ["impressor_id": "<hex>"]
    ["e", "<content_id>"]
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<match_maker_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "match_maker_id": "<hex>",
    "content_id": "<event-id>",
    "impressor_id": "<hex>"
  }
}
```

### ACTION
This event is published when a `SELLER` completes the desired `ACTION`. This event **MUST** be published in order to be paid if a conversion happens from this `MATCH`.

#### TAGS

#### CONTENT
| Field            | Required | Type   | Description |
|------------------|----------|--------|-------------|
| buyer_id         | true     | hex    | The hex of the public key of `BUYER`       |
| seller_id        | true     | hex    | The hex of the public key of `SELLER`      |
| matcher_maker_id | true     | hex    | The hex of the public key of `MATCH MAKER` |
| impressor_id     | true     | hex    | The hex of the public key of `IMPRESSOR`   |
| match_id         | true     | string | event_id of `MATCH` event                  |
| content_id       | true     | string | event_id of `PROMOTED CONTENT` event       |
| action           | true     | string | [ click / follow / retweet / reply / quote / zap ] |


```js
{
  "kind": "?"
  "tags": [
    ["buyer_id": "<hex>"],
    ["seller_id": "<hex>"],
    ["match_maker_id": "<hex>"],
    ["content_id": "<event_id>"],
    ["match_id": "<event_id>"],
    ["impressor_id": "<hex>"],
    ["action", "< click | follow | retweet | reply | quote | zap >"],
    ["e", "<content_id>"],
    ["e", "<match_id>"]
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<match_maker_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "match_maker_id": "<hex>",
    "content_id": "<event_id>",
    "impressor_id": "<hex>",
    "match_id": "<event_id>",
    "action": "< click | follow | retweet | reply | quote | zap >",
  }
}
```

### CONVERSION
This event is published when a `SELLER` completes the desired `ACTION`.
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
    ["match_maker_id": "<hex>"],
    ["impressor_id": "<hex>"],
    ["e", "<content_id>"],
    ["e", "<match_id>"],
    ["e", "<impression_id>"],
    ["e", "<action_id>"],
    ["p", "<buyer_id>"],
    ["p", "<seller_id>"],
    ["p", "<match_maker_id>"],
    ["p", "<impressor_id>"],
  ],
  "content": {
    "content_id": "<event_id>",
    "match_id": "<event_id>",
    "impression_id": "<event_id>",
    "action_id": "<event_id>",
    "buyer_id": "<hex>",
    "seller_id": "<hex>",
    "match_maker_id": "<hex>",
    "impressor_id": "<hex>",
  }
}
```

