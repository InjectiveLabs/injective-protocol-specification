---
description: >-
  This page provides small examples to test your API integration. The full
  reference is available on https://api.injective.dev only.
---

# Relayer API Examples

## Public Testnet Endpoints

* Europe `https://testnet-api.injective.dev`
* US `https://testnet-api-us.injective.dev`
* Asia \(Hong Kong\) `https://testnet-api-ap.injective.dev`

{% api-method method="post" host="https://testnet-api.injective.dev" path="/api/sra/v3/order" %}
{% api-method-summary %}
Post Order \(Spot Markets\)
{% endapi-method-summary %}

{% api-method-description %}
This is a SRA v3 compatible endpoint from 0x spec. Allows user to submit a signed make order to the relayer to be added into spot market's orderbook.  
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="chainId" type="integer" required=true %}
Specify Chain ID of the transaction.
{% endapi-method-parameter %}

{% api-method-parameter name="exchangeAddress" type="string" required=true %}
Exchange v3 contract address.
{% endapi-method-parameter %}

{% api-method-parameter name="expirationTimeSeconds" type="string" required=true %}
Timestamp in seconds at which order expires.
{% endapi-method-parameter %}

{% api-method-parameter name="feeRecipientAddress" type="string" required=true %}
Address that will receive fees when order is filled.
{% endapi-method-parameter %}

{% api-method-parameter name="makerAddress" type="string" required=true %}
Address that created the order.
{% endapi-method-parameter %}

{% api-method-parameter name="makerAssetAmount" type="string" required=true %}
Amount of makerAsset being offered by maker. Must be greater than 0.
{% endapi-method-parameter %}

{% api-method-parameter name="makerAssetData" type="string" required=true %}
ABIv2 encoded data that can be decoded by a specified proxy contract when transferring makerAsset.
{% endapi-method-parameter %}

{% api-method-parameter name="makerFee" type="string" required=true %}
Amount of FeeAsset paid to feeRecipient by maker when order is filled. If set to 0, no transfer of FeeAsset from maker to feeRecipient will be attempted.
{% endapi-method-parameter %}

{% api-method-parameter name="makerFeeAssetData" type="string" required=true %}
ABIv2 encoded data that can be decoded by a specified proxy contract when transferring makerFee.
{% endapi-method-parameter %}

{% api-method-parameter name="salt" type="string" required=true %}
Arbitrary number to facilitate uniqueness of the order's hash.
{% endapi-method-parameter %}

{% api-method-parameter name="senderAddress" type="string" required=true %}
Address that is allowed to call Exchange contract methods that affect this order. If set to 0, any address is allowed to call these methods.
{% endapi-method-parameter %}

{% api-method-parameter name="signature" type="string" required=true %}
Order EIP712 signature.
{% endapi-method-parameter %}

{% api-method-parameter name="takerAddress" type="string" required=true %}
Address that is allowed to fill the order. If set to 0, any address is allowed to fill the order.
{% endapi-method-parameter %}

{% api-method-parameter name="takerAssetAmount" type="string" required=true %}
Amount of takerAsset being bid on by maker. Must be greater than 0.
{% endapi-method-parameter %}

{% api-method-parameter name="takerAssetData" type="string" required=true %}
ABIv2 encoded data that can be decoded by a specified proxy contract when transferring takerAsset.
{% endapi-method-parameter %}

{% api-method-parameter name="takerFee" type="string" required=true %}
Amount of FeeAsset paid to feeRecipient by taker when order is filled. If set to 0, no transfer of FeeAsset from taker to feeRecipient will be attempted.
{% endapi-method-parameter %}

{% api-method-parameter name="takerFeeAssetData" type="string" required=true %}
ABIv2 encoded data that can be decoded by a specified proxy contract when transferring takerFee.
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=201 %}
{% api-method-response-example-description %}
Order successfully posted.
{% endapi-method-response-example-description %}

```
{
  "rLimitLimit": 60,
  "rLimitRemaining": 56,
  "rLimitReset": 1372700873
}
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=417 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{
  "code": 101,
  "reason": "Validation failed",
  "validationErrors": [
    {
      "code": 1001,
      "field": "maker",
      "reason": "Incorrect format"
    }
  ]
}
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=500 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{
    "name": "internal",
    "id": "EMlNh2zD",
    "message": "post order failed: rpc error: code = Internal desc = unknown request: invalid signature",
    "temporary": false,
    "timeout": false,
    "fault": false
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request:

```javascript
curl -X POST "https://testnet-api.injective.dev/api/sra/v3/order" \
    -H "accept: application/json" \
    -H "Content-Type: application/json" -d'
{
  "chainId": 1337,
  "exchangeAddress": "0x12459c951127e0c374ff9105dda097662a027093",
  "expirationTimeSeconds": "1532560590",
  "feeRecipientAddress": "0xb046140686d052fff581f63f8136cce132e857da",
  "makerAddress": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
  "makerAssetAmount": "10000000000000000",
  "makerAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498",
  "makerFee": "100000000000000",
  "makerFeeAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498",
  "salt": "1532559225",
  "senderAddress": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
  "signature": "0x012761a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
  "takerAddress": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
  "takerAssetAmount": "20000000000000000",
  "takerAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498",
  "takerFee": "200000000000000",
  "takerFeeAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498"
}'
```

{% api-method method="get" host="https://testnet-api.injective.dev" path="/api/chronos/v1/history" %}
{% api-method-summary %}
History \(for TradingView\)
{% endapi-method-summary %}

{% api-method-description %}
Request for history bars for TradingView. Corresponds to UDF methods from this TradingView spec - https://www.tradingview.com/rest-api-spec/\#operation/getHistory.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-query-parameters %}
{% api-method-parameter name="sybol" type="string" required=true %}
Symbol name or ticker.
{% endapi-method-parameter %}

{% api-method-parameter name="resolution" type="string" required=true %}
Symbol resolution. Possible resolutions are daily \(D or 1D, 2D ... \), weekly \(1W, 2W ...\), monthly \(1M, 2M...\) and an intra-day resolution – minutes\(1, 2 ...\).
{% endapi-method-parameter %}

{% api-method-parameter name="from" type="integer" required=false %}
Unix timestamp \(UTC\) of the leftmost required bar, including from.
{% endapi-method-parameter %}

{% api-method-parameter name="to" type="integer" required=true %}
Unix timestamp \(UTC\) of the rightmost required bar, including to. It can be in the future. In this case, the rightmost required bar is the latest available bar.
{% endapi-method-parameter %}

{% api-method-parameter name="countback" type="integer" required=false %}
Number of bars \(higher priority than from\) starting with to. If countback is set, from should be ignored.
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{
  "c": [
    3662.25,
    3663.13,
    3664.01
  ],
  "h": [
    3667.24,
    3664.47,
    3664.3
  ],
  "l": [
    3661.55,
    3661.9,
    3662.43
  ],
  "nb": 1484871000,
  "o": [
    3667,
    3662.25,
    3664.29
  ],
  "t": [
    1547942400,
    1547942460,
    1547942520
  ],
  "v": [
    34.7336,
    2.4413,
    11.7075
  ]
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request

```javascript
curl -X GET "https://testnet-api.injective.dev/api/chronos/v1/history?symbol=INJ%2FWETH&resolution=1D&from=1596011845&to=1596019845" -H "accept: application/json"
```

{% api-method method="post" host="https://testnet-api.injective.dev" path="/api/sra/v3/ws" %}
{% api-method-summary %}
0x SRAv3 WebSocket Subscription
{% endapi-method-summary %}

{% api-method-description %}
Implements a WebSocket endpoint conforming the 0x WebSocket API Specification v3. https://github.com/0xProject/standard-relayer-api/blob/master/ws/v3.md  
Clients must subscribe by sending a message of the following contents, the socket will send the updates for existing orders, so the orderbook can be updated accordingly.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="type" type="string" required=true %}
Type of request, e.g. \`subscribe\`.
{% endapi-method-parameter %}

{% api-method-parameter name="channel" type="string" required=true %}
Should be \`orders\`.
{% endapi-method-parameter %}

{% api-method-parameter name="requestId" type="string" required=true %}
A string UUID that will be sent back by the server in response messages so the client can appropriately respond when multiple subscriptions are made.
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{
    "type": "update",
    "channel": "orders",
    "requestId": "123e4567-e89b-12d3-a456-426655440000",
    "payload":  [
        {
          "order": {
              "exchangeAddress": "0x12459c951127e0c374ff9105dda097662a027093",
              "domainHash" "0x78772b297e1b0b31089589a6608930cceba855af9d3ccf7b92cf47fa881e21f7",
              "makerAddress": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
              "takerAddress": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
              "feeRecipientAddress": "0xb046140686d052fff581f63f8136cce132e857da",
              "senderAddress": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
              "makerAssetAmount": "10000000000000000",
              "takerAssetAmount": "20000000000000000",
              "makerFee": "100000000000000",
              "takerFee": "200000000000000",
              "expirationTimeSeconds": "1532560590",
              "salt": "1532559225",
              "makerAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498",
              "takerAssetData": "0x02571792000000000000000000000000371b13d97f4bf77d724e78c16b7dc74099f40e840000000000000000000000000000000000000000000000000000000000000063",
              "exchangeAddress": "0x12459c951127e0c374ff9105dda097662a027093",
              "makerFeeAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498",
              "takerFeeAssetData": "0xf47261b0000000000000000000000000e41d2489571d322189246dafa5ebde1f4699f498",
              "signature": "0x012761a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33"
            },
            "metaData": {
              "remainingTakerAssetAmount": "500000000"
            }
        },
        ...
    ]
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Message

```javascript
{
    "type": "subscribe",
    "channel": "orders",
    "requestId": "123e4567-e89b-12d3-a456-426655440000"
}
```

## Learn from Dexterm

Dexterm is our barebones trading terminal for CLI \(Command Line Interface\). It's written in Go programming language and is already [open-sourced](http://github.com/InjectiveLabs/dexterm). You can learn how to make Relayer API calls from the code in order to make trades. Also, if you're a Gopher, there is a handful of [auto-generated API bindings](https://github.com/InjectiveLabs/dexterm/tree/master/gen) with client-side validation and such.

