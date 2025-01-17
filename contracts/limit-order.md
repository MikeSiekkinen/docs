# Limit Order

Limit Order allows submission, updates and execution of buy and sell orders at a limit price specified by the users. Once the limit order is submitted and the limit price is reached, market making agents can read the orders from the Limit Order contract and execute them when it provides an arbitrage opportunity.  
  
To create a market making bot for arbitrage opportunity, refer to this [Github link](https://github.com/Mirror-Protocol/mirror-contracts/tree/master/contracts/mirror_limit_order).

## InitMsg

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct InitMsg {}
```
{% endtab %}
{% endtabs %}

## HandleMsg

### `Receive`

Can be called during CW20 token transfer when the Limit Order contract is the recipient. Allows the token transfer to execute a Receive Hook as a subsequent action within the same transaction.

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum HandleMsg {
    Receive {
        amount: Uint128,
        sender: HumanAddr
        msg: Binary,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "receive": {
    "amount": "10000000",
    "sender": "terra1...",
    "msg": "eyAiZXhlY3V0ZV9tc2ciOiAiYmxhaCBibGFoIiB9"
  }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `amount` | Uint128 | Token amount received |
| `sender` | HumanAddr | Sender of token transaction |
| `msg` | Binary | Base64-encoded string of JSON of [Receive Hook](https://docs.mirror.finance/contracts/limit-order#receive-hooks) |

### `SubmitOrder`

Creates a new Limit Order with a types and amount of tokens to be traded, specified by the user. `offer_asset` is locked until the order is executed or canceled by the creator.   


When tokens other than native tokens are sent \(such as CW20\), it a [Receive Hook](https://docs.mirror.finance/contracts/limit-order#receive-hooks) must be sent to submit the order. 

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum HandleMsg {
    SubmitOrder {
        offer_asset: Asset,
        ask_asset: Asset,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "submit_order": {
    "offer_asset": {
        "info": {
            "token": {
                "contract_address": "terra1..."
            }
        },
        "amount": "1000000"
    },
    "ask_asset": {
        "info": {
            "native_token": {
                "denom": "uusd",
            }
        },
        "amount": "10000000"
    }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `offer_asset` | Asset | Asset to be submitted to Limit Order |
| `ask_asset` | Asset | Asset to receive when order is executed |

### `CancelOrder`

Order is canceled by the creator. `offer_asset` locked in this order is released and returned to the owner. 

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum HandleMsg {
    CancelOrder {
        order_id: u64,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "cancel_order": {
    "order_id": 10,
    },
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `order_id` | u64 | Order ID |

### `ExecuteOrder`

Executes order against an existing limit order. You cannot submit more than the amount defined by `ask_asset - filled_asset_amounnt` in a specific order. 

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum HandleMsg {
    ExecuteOrder {
        execute_asset: Asset,
        order_id: u64,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "execute_order": {
    "execute_asset": {
        "info": {
            "token": {
                "contract_address": "terra1..."
            }
        },
        "amount": "1000000"
    },
    "order_id": 8
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `execute_asset` | Asset | Asset to be executed from Limit Order |
| `order_id` | u64 | Order ID |

## Receive Hooks

{% hint style="danger" %}
If you send tokens to the Limit Order contract without issuing this hook, they will not be used to create or execute order and will **BE LOST FOREVER.**
{% endhint %}

### `SubmitOrder`

Issued when user sends CW20 tokens to Limit Order contract. 

Locks the sent amount to create a new order. 

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum Cw20HookMsg {
    SubmitOrder {
        ask_asset: Asset,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "submit_order": {
    "ask_asset": {
        "info": {
            "token": {
                "contract_address": "terra1..."
            }
          }
        },
        "amount": "10000000"
    }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `ask_asset` | Asset | Asset to be executed from Limit Order |

### `ExecuteOrder`

Issued when arbitrageur sends and fulfills the amount of `ask_asset` to a specific limit order.   
  
Reduces the balance of the `ask_asset` and `offer_asset` of the specified `order_id`. If all outstanding `ask_asset` has been filled, then the order is completed. 

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum Cw20HookMsg {
    ExecuteOrder {
        order_id: u64,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
    "execute_order": {
        "order_id": 10
    }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `order_id` | u64 | Order ID |

## QueryMsg

### `Order`

Gets information about specified order ID’s details.

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    Order: {
        order_id: u64,
    }
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
    "order": {
        "order_id": 10
    }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `order_id` | u64 | Order ID |

### `OrderResponse`

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct OrderResponse {
    pub order_id: u64,
    pub bidder_addr: HumanAddr,
    pub offer_asset: Asset,
    pub ask_asset: Asset,
    pub filled_offer_amount: Uint128,
    pub filled_ask_amount: Uint128,
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
    "OrderResponse": {
        "order_id": 10,
        "bidder_addr": "terra1...",
        "offer_asset": {
            "info": {
                "token": {
                    "contract_address": "terra1..."
                }
            },
            "amount": "10000000"
        },
        "ask_asset": {
            "info": {
                "native_token": {
                    "denom": "uusd",
                }
            },
            "amount": "10000000"
        },
        "filled_offer_amount": "10000000",
        "filled_ask_amount": "10000000"
    }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `order_id` | u64 | Order ID |
| `bidder_addr` | HumanAddr | Address of bidder |
| `offer_asset` | Asset | Amount of asset offered in order |
| `ask_asset` | Asset | Amount of asset asked in order |
| `filled_offer_amount` | Uint128 | Amount of offer asset already executed |
| `filled_ask_amount` | Uint128 | Amount of ask asset already executed |

### `Orders`

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    Orders: {
        bidder_addr: Option<HumanAddr>,
        start_after: Option<u64>,
        limit: Option<u32>,
        order_by: Option<OrderBy>,

    }
}
```
{% endtab %}

{% tab title="JSON" %}
```
{
  "orders": {
    "bidder_addr": "terra1...",
    "limit": 8,
    "order_by": "asc"
    "start_after": 8
    }
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `bidder_addr`\* | HumanAddr | Address of the order bidder |
| `start_after`\* | u64 | Begins search query at specific Order ID |
| `limit`\* | u32 | Limit of results to fetch |
| `order_by`\* | OrderBy | Can be ASC or DESC |

\*=optional  


### `OrdersResponse`

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct OrdersResponse {
    pub orders: Vec<OrderResponse>,
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "OrdersResponse": [
        {
        "order_id": 10
        "bidder_addr": "terra1...",
        "offer_asset": {
            "info": {
                "token": {
                    "contract_address": "terra1..."
                }
            },
            "amount": "10000000"
        },
        "ask_asset": {
            "info": {
                "native_token": {
                    "denom": "uusd",
                }
            },
            "amount": "10000000"
        },
        "filled_offer_amount": "10000000",
        "filled_ask_amount": "10000000",
        "order_id": 10
        },
        {
        "order_id": 10
        "bidder_addr": "terra1...",
        "offer_asset": {
            "info": {
                "token": {
                    "contract_address": "terra1..."
                }
            },
            "amount": "10000000"
        },
        "ask_asset": {
            "info": {
                "native_token": {
                    "denom": "uusd",
                }
            },
            "amount": "10000000"
        },
        "filled_offer_amount": "10000000",
        "filled_ask_amount": "10000000",
        }
    ]
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `orders` | Vec&lt;OrderResponse&gt; | Vector of user's order information |

| Key | Type | Description |
| :--- | :--- | :--- |
| `order_id` | u64 | Order ID |
| `bidder_addr` | HumanAddr | Address of bidder |
| `offer_asset` | Asset | Amount of asset offered in order |
| `ask_asset` | Asset | Amount of asset asked in order |
| `filled_offer_amount` | Uint128 | Amount of offer asset already executed |
| `filled_ask_amount` | Uint128 | Amount of ask asset already executed |

### `LastOrderID`

Gets the most recently submitted order ID.

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    LastOrderID: {}
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
    "last_order_id": {}
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
|  |  |  |

### `LastOrderIDResponse`

{% tabs %}
{% tab title="Rust" %}
```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct LastOrderIDResponse {
    pub last_order_id:u64
}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
    "LastOrderIDResponse": {
        "last_order_id": 10
}
```
{% endtab %}
{% endtabs %}

| Key | Type | Description |
| :--- | :--- | :--- |
| `last_order_id` | u64 | Index of the most recent order |

