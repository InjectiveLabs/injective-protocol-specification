# Sending Testnet Transactions

Our testnet is a Tendermint based sidechain but abstracts away the necessity for users to send Tendermint style transactions. To perform actions on our sidechain (e.g. creating orders, placing trades, canceling orders, etc.) users can simply send HTTP requests to any relayer which runs our `relayer-api` REST server which is compliant with the [0x v2 Standard Relayer API specification](https://github.com/0xProject/standard-relayer-api/blob/master/http/v2.md). 

The full OpenAPI specification can be found [here](https://injective-tendermint-external-and-internal-api-2.api-docs.io/undefined/api). 

### GetAccount
```bash
$ curl "http://testnet-validator-1.injective.dev/api/private/getAccount?address=cosmos1nsm4jltjmjwfah3xxzfcv4r2za3qtt4mhqp2h9"
```

### GetOnlineAccounts
```bash
$ curl "http://testnet-validator-1.injective.dev/private/getOnlineAccounts?threshold=60"
```
### GetEthTransactions
```bash
$ curl -X "POST" "http://testnet-validator-1.injective.dev/getEthTransactions" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "proposer": "cosmos15ujfj08ny209krcxyls423y5lfxmpxxv6klrpw"
}'
```

### PostOrder

```bash
$ curl -X "POST" "http://testnet-validator-1.injective.dev/postOrder" \
     -H 'Authorization: Bearer' \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "expirationTimeSeconds": "1571309285",
  "feeRecipientAddress": "0x0000000000000000000000000000000000000000",
  "takerAssetAmount": "9998000000000000",
  "takerFee": "0",
  "exchangeAddress": "0x48bacb9266a570d521063ef5dd96e61686dbe788",
  "makerFee": "0",
  "takerAddress": "0xb125995f5a4766c451cd8c34c4f5cac89b724571",
  "salt": "1915",
  "signature": "0x1c69937ac0397f9075cb528a84b809ecddadfc0f971392d1512d9ea2147d270fb36be17105a8d97fe9833a135fc6ca498315832f604ad772ddc859af3ea6383ce403",
  "makerAddress": "0x6ecbe1db9ef729cbe972c83fb886247691fb6beb",
  "senderAddress": "0x0000000000000000000000000000000000000000",
  "makerAssetData": "0xf47261b0000000000000000000000000e1703da878afcebff5b7624a826902af475b9c03",
  "makerAssetAmount": "10000000000000000000",
  "takerAssetData": "0xf47261b00000000000000000000000000b1ba0af832d7c05fd64161e0db78e85978e8082"
}'
```
### TakeOrder
```bash
$ curl -X "POST" "http://testnet-validator-1.injective.dev/takeOrder" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "makeOrders": [
    {
      "expirationTimeSeconds": "1571309285",
      "feeRecipientAddress": "0x0000000000000000000000000000000000000000",
      "takerAssetAmount": "9998000000000000",
      "takerFee": "0",
      "exchangeAddress": "0x48bacb9266a570d521063ef5dd96e61686dbe788",
      "makerFee": "0",
      "takerAddress": "0xb125995f5a4766c451cd8c34c4f5cac89b724571",
      "salt": "1915",
      "signature": "0x1c69937ac0397f9075cb528a84b809ecddadfc0f971392d1512d9ea2147d270fb36be17105a8d97fe9833a135fc6ca498315832f604ad772ddc859af3ea6383ce403",
      "makerAddress": "0x6ecbe1db9ef729cbe972c83fb886247691fb6beb",
      "senderAddress": "0x0000000000000000000000000000000000000000",
      "makerAssetData": "0xf47261b0000000000000000000000000e1703da878afcebff5b7624a826902af475b9c03",
      "makerAssetAmount": "10000000000000000000",
      "takerAssetData": "0xf47261b00000000000000000000000000b1ba0af832d7c05fd64161e0db78e85978e8082"
    }
  ],
  "takeOrder": {
    "expirationTimeSeconds": "1571571090",
    "feeRecipientAddress": "0x0000000000000000000000000000000000000000",
    "takerAssetAmount": "9998000000000000000",
    "takerFee": "0",
    "exchangeAddress": "0x48bacb9266a570d521063ef5dd96e61686dbe788",
    "makerFee": "0",
    "takerAddress": "0xb125995f5a4766c451cd8c34c4f5cac89b724571",
    "salt": "4009",
    "signature": "0x1b154cd1b82d4f178d52dbc6bddea660b88c0cba07313fce971bb10ceb3442fc567e85a9c4a5558bd37c40a8498b7d837eb8962757e0a9b398e028aaba08922d7103",
    "makerAddress": "0x5409ed021d9299bf6814279a6a1411a7e866a631",
    "senderAddress": "0x0000000000000000000000000000000000000000",
    "makerAssetData": "0xf47261b00000000000000000000000000b1ba0af832d7c05fd64161e0db78e85978e8082",
    "makerAssetAmount": "10000000000000000",
    "takerAssetData": "0xf47261b0000000000000000000000000e1703da878afcebff5b7624a826902af475b9c03"
  }
}'
```

### GetOrder
```bash
$ curl "http://testnet-validator-1.injective.dev/getOrder?orderHash=0x40c215ad7c180f5c30145b64c4c987b012af66996cb9288e3a6ae621db8fedf2"
```
### GetActiveOrder
```bash
$ curl "http://testnet-validator-1.injective.dev/getActiveOrder?orderHash=0x40c215ad7c180f5c30145b64c4c987b012af66996cb9288e3a6ae621db8fedf2"
```
### GetArchiveOrder
```bash
$ curl "http://testnet-validator-1.injective.dev/getArchiveOrder?orderHash=0x40c215ad7c180f5c30145b64c4c987b012af66996cb9288e3a6ae621db8fedf2"
```
### ListOrders
```bash
$ curl -X "POST" "http://testnet-validator-1.injective.dev/listOrders" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{}'
```
### GetTradePair
```bash
$ curl "http://testnet-validator-1.injective.dev/getTradePair?name=MOCK%2FWETH" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{}'
```
### ListTradePairs
```bash
$ curl -X "POST" "http://testnet-validator-1.injective.dev/listTradePairs" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{}'
```

