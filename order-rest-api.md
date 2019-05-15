# Order Rest API for gx-ex (2019-05-15)
# General API Information
* The base endpoint is: **http://127.0.0.1**
* All endpoints return either a JSON object or array.
* Data is returned in **ascending** order. Oldest first, newest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side.
* HTTP `5XX` return codes are used for internal errors; the issue is on gx-ex's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.
* Any endpoint can return an ERROR; the error payload is as follows:
```javascript
{
  "code": 500,
  "msg": "Invalid symbol."
}
```

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded` or `application/json`.

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `X-BIFRAX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
* By default, API-keys can access all secure routes.

# SIGNED (Private) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* `totalParams` is must be sorted `alphabetically` for signatures.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body` include `application/json`.
  
 ## SIGNED Endpoint Examples for POST /v1/order
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | testApiKey
secretKey | testSecretKey


Parameter | Value
------------ | ------------
exchangeType| LOCAL
price | 0.1
quantity | 1
side | BUY
symbol | LTC/BTC
type | LIMIT




### Example 1: As a query string (Just example, Get mothd is not work for order)
* **queryString:** exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= aa406abc91c8a05f9e3e5d8aa31a9c6ef6ef015ec008f9c6334faa8fdc9c1b96
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X POST 'https://127.0.0.1/v1/order?exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT&signature=aa406abc91c8a05f9e3e5d8aa31a9c6ef6ef015ec008f9c6334faa8fdc9c1b96'
    ```

### Example 2: As a request body
* **requestBody:** 
  
  exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT
  
  or
  
  {"exchangeType":"LOCAL", "price":"0.1", "quantity":"1", "side":"BUY", "symbol":"LTC/BTC", "type":"LIMIT"}
  
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= aa406abc91c8a05f9e3e5d8aa31a9c6ef6ef015ec008f9c6334faa8fdc9c1b96
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X POST 'https://127.0.0.1/v1/order' -d '{"exchangeType":"LOCAL", "price":"0.1", "quantity":"1", "side":"BUY", "symbol":"LTC/BTC", "type":"LIMIT", "signature":"aa406abc91c8a05f9e3e5d8aa31a9c6ef6ef015ec008f9c6334faa8fdc9c1b96"}'

## Orders endpoints

### Infomation

* ExchangeType

`LOCAL`: local currency exchanges(piat currency)

`SPOT`: JGXcash exchange

`FUTURE`: future exchanges


### New order (TRADE)
```
POST /v1/order  (HMAC SHA256)
```
Send in a new order.

**Parameters:**


Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
exchangeType | STRING | NO | 
symbol | STRING | YES |
side | ENUM | YES | `BUY`, `SELL`
type | ENUM | YES | `MARKET`, `LIMIT`
quantity | DECIMAL | YES |
price | DECIMAL | NO |
signature | STRING | YES |

Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
LIMIT | `quantity`, `price`
MARKET | `quantity`


**Response RESULT:**
```javascript
{
    "orderId": "344287",
    "clientOrderId": "201903100228344287",
    "symbol": "BTC/KRW",
    "quantity": "0.003"
}
```
### Cancel order (TRADE)
```
POST /v1/order/cancel  (HMAC SHA256)
```
Cancel an active order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
origOrderId | STRING | YES |
origClientOrderId | STRING | YES |
exchangeType | STRING | NO | 


**Response:**
```javascript
{
    "orderId": "344832",
    "clientOrderId": "201903100614344832",
    "symbol": "BTC/KRW"
}
```

### Current open all orders 
```
GET /v1/orders  (HMAC SHA256)
```
Get all open orders on a symbol. 


**Parameters:**


Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
exchangeType | STRING | NO | 
symbol | STRING | YES |
signature | STRING | YES |

**Response:**
```javascript
{
    "orders": [
        {
            "orderId": "194671",
            "origOrderId": "0",
            "clientOrderId": "201903090636194671",
            "symbol": "BTC/KRW",
            "side": "BUY",
            "type": "LIMIT",
            "price": "3755000",
            "origQty": "0.003",
            "executedQty": "0",
            "remainedQty": "0.003",
            "txTime": 1552113370000
        },
        {
            "orderId": "194517",
            "origOrderId": "0",
            "clientOrderId": "201903090633194517",
            "symbol": "BTC/KRW",
            "side": "BUY",
            "type": "LIMIT",
            "price": "3755000",
            "origQty": "0.003",
            "executedQty": "0",
            "remainedQty": "0.003",
            "txTime": 1552113194000
        }
    ]
}
```

### all trades
```
GET /v1/trades (HMAC SHA256)
```
Get all orders; canceled, or filled.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
exchangeType | STRING | NO | 
startTime | LONG | NO | Actually system works by day.(exclude hours, minutes, seconds) , ex) 1552089600000
endTime | LONG | NO |Actually system works by day.(exclude hours, minutes, seconds), ex) 1552089600000
nextKey | STRING | NO | for Next Page


**Response:**
```javascript
{
    "trades": [
        {
            "orderId": "229590",
            "origOrderId": "0",
            "clientOrderId": "201903091212229590",
            "origClientOrderId": "00",
            "symbol": "BTC/KRW",
            "tradeType": "NEW_ORDER",
            "tradeStatus": "OK",
            "rejectReason": "0",
            "side": "BUY",
            "type": "MARKET",
            "price": "0",
            "origQty": "0.003",
            "executedPrice": "4339000",
            "executedQty": "0.003",
            "remainedQty": "0",
            "executedPriceSum": "13017",
            "tradeFee": "13",
            "txTime": 1552133565000
        },
        {
            "orderId": "205344",
            "origOrderId": "194962",
            "clientOrderId": "201903090759205344",
            "origClientOrderId": "201903090639194962",
            "symbol": "BTC/KRW",
            "tradeType": "CANCEL",
            "tradeStatus": "COMPLETE",
            "rejectReason": "0",
            "side": "BUY",
            "type": "LIMIT",
            "price": "0",
            "origQty": "0.003",
            "executedPrice": "0",
            "executedQty": "0",
            "remainedQty": "0",
            "executedPriceSum": "0",
            "tradeFee": "0",
            "txTime": 1552118360000
        },
    ]
}
```
### trade status
```
GET /v1/tradeStatus (HMAC SHA256)
```
Get all orders; canceled, or filled.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ClientOrderId | STRING | YES |
OrderId | STRING | YES |
exchangeType | STRING | YES | 
symbol | STRING | YES |

**Response:**
```javascript
{
    "tradeStatus": [
        {
            "orderId": "229590",
            "origOrderId": "0",
            "clientOrderId": "201903091212229590",
            "origClientOrderId": "00",
            "symbol": "BTC/KRW",
            "tradeType": "NEW_ORDER",
            "tradeStatus": "OK",
            "rejectReason": "0",
            "side": "BUY",
            "type": "MARKET",
            "price": "0",
            "origQty": "0.003",
            "executedPrice": "4339000",
            "executedQty": "0.003",
            "remainedQty": "0",
            "executedPriceSum": "13017",
            "tradeFee": "13",
            "txTime": 1552133565000
        }
    ]
}
```
