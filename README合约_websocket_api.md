# 永续合约
以下是永续合约V3 WebscoketAPI

-
# 概述
WebSocket是HTML5一种新的协议(Protocol)。它实现了客户端与服务器全双工通信， 使得数据可以快速地双向传播。通过一次简单的握手就可以建立客户端和服务器连接， 服务器根据业务规则可以主动推送信息给客户端。其优点如下：

- 客户端和服务器进行数据传输时，请求头信息比较小，大概2个字节。
- 客户端和服务器皆可以主动地发送数据给对方。
- 不需要多次创建TCP请求和销毁，节约宽带和服务器的资源。

### 强烈建议开发者使用WebSocket API获取市场行情和买卖深度等信息。
地址：wss://csocketapi.bitget.com/ws/v1

连接说明：

连接上ws后30s内订阅或订阅后30s内用户未发送ping指令，系统会自动断开连接

-
# 指令格式
请求格式：
> {"op": "`<value>`", "args": ["`<value1>`","`<value2>`"]}

其中 op 的取值为 1--subscribe 订阅； 2-- unsubscribe 取消订阅 ；3--login 登录

args: 取值为频道名，可以定义一个或者多个频道

成功响应格式:
> {"event": "`<value>`","channel":"`<value>`"}
> 
> {"table":"channel","data":"[{"`<value1>`","`<value2>`"}]"}

失败响应格式:
> {"event":"error","message":"<error_message>","errorCode":"<errorCode>"}

-
# 交易对格式
反向合约(示例):
> btcusd、ethusd、ltcusd、xrpusd、bchusd、eosusd

正向合约(示例):
> cmt_bnbusdt、cmt_btcusdt

模拟盘合约(示例):
> sbtcusd（反向合约模拟盘）、cmt_btcsusdt（正向合约模拟盘）

-
# 订阅
用户可以选择订阅一个或者多个频道，多个频道总长度不能超过4096个字节
> {"op": "subscribe", "args": ["`<SubscriptionTopic>`"]}

说明 ： op 的取值是 subscribe

args 数组内容为频道名称 ：<channelname>:<filter>

其中channelname 是以 business/channel组成

永续推送业务business为swap， channel为此业务下每个具体的名称，如果channel的名字不能以一个字母区分将会以 " _ " 进行连接

例：
> "swap/ticker:btcusd" or "swap/price_range:btcusd"

filter 是可筛选数据，具体参考每个频道说明

示例：

send:
> {"op": "subscribe", "args": ["swap/ticker:btcusd", "swap/candle60s:btcusd"]}

response:
> 
> {"event": subscribe,"channel":"swap/ticker:btcusd"}

> {"event": subscribe,"channel":"swap/candle60s:btcusd"}
> 
> {"table":"swap/ticker","data":[{"high_24h":"3369","instrument_id":"btcusd","last":"3299",
 "low_24h":"3112","timestamp":"2018-12-09T08:12:04.659Z","volume_24h":"8389225"}]}
 
> {"table":"swap/candle60s","data":[{"instrument_id":"btcusd","candle":["2018-12-09T08:12:00.000Z",
 "3299.8","3299.8","3299","3299","82","2.4854"]}]}
>

-
# 取消订阅
可以取消一个或者多个频道
> {"op": "unsubscribe", "args": [`<SubscriptionTopic>`]}

例如：

请求：
> {"op": "unsubscribe", "args": ["swap/ticker:btcusd", "swap/candle60s:btcusd"]}

返回：
>
> {"event":"unsubscribe","channel":"swap/ticker:btcusd"}
> {"event":"unsubscribe","channel":"swap/candle60s:btcusd"}
>

-
# 登录
签名方式说明参考API概述里的验证部分

登录订阅格式：
> {"op":"login","args":["`<access_key>`","`<timestamp>`","`<sign>`"]}

响应：
> {"event":"login","success":"true"}

例：
> {"op":"login","args":["985d5b66-57ce-40fb-b714-afc0b9787083","1538054050975","7L+zFQ+CEgGu5rzCj4+BdV2/uUHGqddA9pI6ztsRRPs="]}

**access_key**:为用户申请的ACCESS_KEY

**timestamp**:时间戳（req_time） 您发出请求的时间(13位时间戳，取整数)

**sign**:用户计算签名的基于哈希的协议，此处使用SHA1的SecretKey和请求参数进行MD5加密的结果作为取值

**sign示例如下:
    比如这个请求：https://api.bitget.com/api/v1/account/accounts?method=accounts&accesskey=ak4****d4&sign=7c8*****f1c&req_time=1529137093705
     步骤
        1 请求参数method=xxx作为字符串  如："method=accounts"
        2 使用SHA1加密后的SecretKey
        3 然后将这个两个参数进行MD5加密的结果作为签名值

如果登录失败会自动断开链接

-
# 连接限制
**连接限制**：1次/s

**订阅限制**：每小时240次

连接上ws后如果一直没有数据返回，30s 后自动断开链接， 建议用户进行以下操作:

1. 每次接收到消息后，用户设置一个定时器 ，定时20秒。

2. 如果定时器被触发（20 秒内没有收到新消息），发送字符串 'ping'。

3. 期待一个文字字符串'pong'作为回应。如果在 20秒内未收到，请发出错误或重新连接。

出现网络问题会自动断开连接

-
# 频道说明
**无需登录的频道包括**：行情频道，K线频道，交易数据频道，资金费率频道，限价范围频道，深度数据频道，标记价格频道

**需登录的频道包括**：用户账户频道，用户交易频道，用户持仓频道

**无需登录的频道名称如下**：

swap/ticker // 行情数据频道

swap/candle60s // 1分钟 k线数据频道

swap/candle300s // 5分钟 k线数据频道

swap/candle900s // 15分钟 k线数据频道

swap/candle1800s // 30分钟 k线数据频道

swap/candle3600s // 1小时 k线数据频道

swap/candle14400s // 4小时 k线数据频道

swap/candle43200s // 12小时 k线数据频道

swap/candle86400s // 1day k线数据频道

swap/candle604800s // 1week k线数据频道

swap/trade // 交易信息频道

swap/funding_rate//资金费率频道

swap/price_range//限价范围频道

swap/depth //深度数据频道，首次200档，后续增量

swap/depth5 //深度数据频道，每次返回前5档

swap/mark_price// 标记价格频道

**需登录的频道如下**：

swap/account //用户账户信息频道

swap/position //用户持仓信息频道

swap/order //用户交易数据频道

-
# 用户持仓频道
获取用户持仓信息，需用用户登录

### send示例
> {"op": "subscribe", "args": ["swap/position:btcusd"]}

其中swap/ position为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
liquidation_price | String | 预估强平价 |
position | String | 持仓数量
avail_position | String | 可平数量 |
avg_cost | String | 开仓平均价 |
instrument_id | String | 合约名称 |
leverage | String | 杠杆 |
realized_pnl | String | 已实现盈亏 |
unrealized_pnl | String | 未实现盈亏 |
side | String | 方向（long/short） | 
timestamp | String | 创建时间 |
margin | String | 保证金 |
margin_mode | String | fixed:逐仓crossed:全仓 |

### # 返回示例
> {<br>
    "table": "swap/position",<br>
    "data": [{<br>
        "holding": [{<br>
            "avail_position": "1",<br>
            "avg_cost": "0.2985",<br>
            "leverage": "5",<br>
            "liquidation_price": "0.0136",<br>
            "margin": "6.8129",<br>
            "position": "1",<br>
            "realized_pnl": "-0.0239",<br>
            "unrealized_pnl": "0.0001",<br>
            "side": "long",<br>
            "timestamp": "1559544244016"<br>
        }, {<br>
            "avail_position": "1",<br>
            "avg_cost": "0.2935",<br>
            "leverage": "5",<br>
            "liquidation_price": "0.0136",<br>
            "margin": "6.8129",<br>
            "position": "1",<br>
            "realized_pnl": "-0.0239",<br>
            "unrealized_pnl": "0.0001",<br>
            "side": "short",<br>
            "timestamp": "1559544244016"<br>
        }],<br>
        "instrument_id": "xrpusd",<br>
        "margin_mode": "crossed"<br>
    }]<br>
}<br>

-
# 用户账户频道
获取账户信息，需用用户登录

### send示例
> {"op": "subscribe", "args": ["swap/account:btcusd"]}

其中swap/account为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
equity | String | 账户权益 |
instrument_id | String | 合约名称 |
margin | String | 已用保证金 |
margin_frozen | String | 开仓冻结保证金 |
realized_pnl | String | 已实现盈亏 |
timestamp | String | 创建时间 |
total_avail_balance | String | 账户余额 |
unrealized_pnl | String | 未实现盈亏 |
fixed_balance | String | 逐仓账户余额 |
margin_mode | String | 账户类型：逐仓fixed 全仓crossed |

### # 返回示例
> {<br>
    "table": "swap/account",<br>
    "data": [{<br>
        "equity": "21.9334",<br>
        "fixed_balance": "0.0000",<br>
        "instrument_id": "xrpusd",<br>
        "margin": "6.8129",<br>
        "margin_frozen": "6.8143",<br>
        "margin_mode": "crossed",<br>
        "realized_pnl": "0.0000",<br>
        "timestamp": "1559544244016",<br>
        "total_avail_balance": "22.0043",<br>
        "unrealized_pnl": "-0.0710"<br>
    }]<br>
}<br>

-
# 用户交易频道
获取用户交易数据，需用用户登录

### send示例
> {"op": "subscribe", "args": ["swap/order:btcusd"]}

其中swap/order为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
instrument_id | String | 合约名称，如btcusd |
size | String | 委托数量 |
timestamp | String | 创建时间 |
filled_qty | String | 成交数量 |
fee | String | 手续费 |
order_id | String | 订单id |
client_oid | String | 用户设置的订单id |
price | String | 委托价格 |
price_avg | String | 成交均价 |
type | String | 1:开多 2:开空 3:平多 4:平空 |
contract_val | String | 合约面值 |
order_type | String | 0：普通委托 1：只做Maker（Post only） 2：全部成交或立即取消（FOK） 3：立即成交并取消剩余（IOC） |
last_fill_time | String | 最新成交时间 |
state | String | 订单状态("1":新单,"2":部分成交,"3":全部成交 ,"5":撤销, "6":触发风险撤销） |

### # 返回示例
> {<br>
    "table": "swap/order",<br>
    "data": [{<br>
        "last_fill_time": "1559544244016",<br>
        "filled_qty": "0",<br>
        "fee": "0.000000",<br>
        "client_oid": "",<br>
        "last_fill_qty": "0",<br>
        "price_avg": "0.0000",<br>
        "type": "2",<br>
        "instrument_id": "xrpusd",<br>
        "last_fill_px": "0",<br>
        "size": "1",<br>
        "price": "0.2935",<br>
        "error_code": "0",<br>
        "contract_val": "10",<br>
        "state": "0",<br>
        "order_id": "6c-a-625f6f638-0",<br>
        "order_type": "0",<br>
        "status": "0",<br>
        "timestamp": "1559544244016"<br>
    }]<br>
}<br>

-
# 公共-Ticker频道
获取平台全部永续合约的最新成交价、买一价、卖一价和24交易量

### send示例
> {"op": "subscribe", "args": ["swap/ticker:btcusd"]}

其中swap/ticker为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
instrument_id | String | 合约名称，如btcusd |
best_bid | String | 买一价 |
best_ask | String | 卖一价 |
last | String | 最新成交价 |
high_24h | String | 24小时最高价 |
low_24h | String | 24小时最低价 |
volume_24h | String | 24小时成交量 |
timestamp | String | 系统时间戳 |

### # 返回示例
> {<br>
    "table": "swap/ticker",<br>
    "data": [{<br>
        "best_ask": "5603.5",<br>
        "best_bid": "5600.1",<br>
        "high_24h": "5773.7",<br>
        "instrument_id": "btcusd",<br>
        "last": "5603.3",<br>
        "low_24h": "5566",<br>
        "timestamp": "1559544244016",<br>
        "volume_24h": "1538076"<br>
    }]<br>
}<br>

-
# 公共-K线频道
获取合约的K线数据

频道列表：

swap/candle60s // 1分钟k线数据频道

swap/candle300s // 5分钟k线数据频道

swap/candle900s // 15分钟k线数据频道

swap/candle1800s // 30分钟k线数据频道

swap/candle3600s // 1小时k线数据频道

swap/candle14400s // 4小时k线数据频道

swap/candle43200s // 12小时k线数据频道

swap/candle86400s // 1dayk线数据频道

swap/candle604800s // 1week k线数据频道

### send示例
> {"op": "subscribe", "args": ["swap/candle60s:btcusd"]}

其中swap/candle60s为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
timestamp | String | 开始时间 |
open | String | 开盘价格 |
high | String | 最高价格 |
low | String | 最低价格 |
close | String | 收盘价格 |
volume | String | 交易量(张) |
instrument_id | String | 合约btcusd |

### # 返回示例
> {<br>
    "table": "swap/candle60s",<br>
    "data": [{<br>
        "instrument_id": "btcusd",<br>
        "candle": ["1559544244016", "5613", "5613", "5611.9", "5611.9", "1218", "21.7009"]<br>
    }]<br>
}<br>

-
# 公共-交易频道
获取最近的成交数据。

### send示例
> {"op": "subscribe", "args": ["swap/trade:btcusd"]}

其中swap/trade为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
price | String | 成交价格 |
size | String | 成交数量 |
side | String | 成交方向（buy or sell） |
timestamp | String | 成交时间 |
instrument_id | String | btcusd |

### # 返回示例
> {<br>
     "table": "swap/trade",<br>
    "data": [{<br>
        "instrument_id": "btcusd",<br>
        "price": "5611.9",<br>
        "side": "buy",<br>
        "size": "2",<br>
        "timestamp": "1559544244016",<br>
    }]<br>
}<br>

-
# 公共-资金费率频道
获取合约资金费率

### send示例
> {"op": "subscribe", "args": ["swap/funding_rate:btcusd"]}

其中swap/funding_rate为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
instrument_id | String | 合约名称，如btcusd |
funding_time | String | 下一次结算时间 |
funding_rate | String | 当前资金费率 |

### # 返回示例
> {<br>
    "table": "swap/funding_rate",<br>
    "data": [{<br>
        "funding_rate": "-0.00067",<br>
        "funding_time": "1559544244016",<br>
        "instrument_id": "btcusd",<br>
    }]<br>
}<br>

-
# 公共-限价频道
获取合约当前开仓的最高买价和最低卖价。

### send示例
> {"op": "subscribe", "args": ["swap/price_range:btcusd"]}

其中swap/ price_range为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
timestamp | String | 系统时间戳 |
lowest | String | 最低卖价 |
instrument_id | String | 合约名称，如btcusd |
highest | String | 最高买价 |

### # 返回示例
> {<br>
    "table": "swap/price_range",<br>
    "data": [{<br>
        "highest": "5665.9",<br>
        "instrument_id": "btcusd",<br>
        "lowest": "5553.6",<br>
        "timestamp": "1559544244016"<br>
    }]<br>
}<br>

-
# 公共-深度5频道
每次返回前五档的深度数据

### send示例
> {"op": "subscribe", "args": ["swap/depth5:btcusd"]}

其中swap/depth5为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
asks | String | 卖方深度 |
bids | String | 买方深度 |
timestamp | String | 时间戳 |
instrument_id | String | 合约ID |

["411.8","6"] [String,String] 411.8为深度价格，10为此价格数量

### # 返回示例
> {<br>
    "table": "swap/depth5",<br>
    "data": [{<br>
        "asks": [<br>
            ["5621.7", "58"],<br>
            ["5621.8", "125"],<br>
            ["5622", "84"],<br>
            ["5622.5", "6"],<br>
            ["5623", "1"]<br>
        ],<br>
        "bids": [<br>
            ["5621.3", "287"],<br>
            ["5621.2", "41"],<br>
            ["5621.1", "2"],<br>
            ["5621", "26"],<br>
            ["5620.9", "640"]<br>
        ],<br>
        "instrument_id": "btcusd",<br>
        "timestamp": "1559544244016"<br>
    }]<br>
}<br>

-
# 公共-深度频道
首次返回200档，后续为增量

### send示例
> {"op": "subscribe", "args": ["swap/depth:btcusd"]}

其中swap/depth为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
asks | String | 卖方深度 |
bids | String | 买方深度 |
action | String | 全量增量标识 |
timestamp | String | 时间戳 |
instrument_id | String | 合约ID |

["411.8","6"] [String,String] 411.8为深度价格，10为此价格数量

### # 返回示例
> 首次200档<br><br>
{<br>
    "table": "swap/depth",<br>
    "action": "(partial/update)",<br>
    "data": [{<br>
        "asks": [<br>
            ["5621.7", "58"],<br>
            ["5621.8", "125"],<br>
            ["5622", "84"],<br>
            ["5622.5", "6"],<br>
            ["5623", "1"]<br>
        ],<br>
        "bids": [<br>
            ["5621.3", "287"],<br>
            ["5621.2", "41"],<br>
            ["5621.1", "2"],<br>
            ["5621", "26"],<br>
            ["5620.9", "640"]<br>
        ],<br>
        "instrument_id": "btcusd",<br>
        "timestamp": "1559544244016"<br>
    }]<br>
}<br>

-
# 公共-标记价格频道
获取标记价格

### send示例
> {"op": "subscribe", "args": ["swap/mark_price:btcusd"]}

其中swap/ mark_price为频道名，btcusd为instrument_id

### # 返回参数
参数名 | 参数类型 | 描述 |
----------- | ----------- | ----------- |
instrument_id | String | 合约名称，如btcusd |
mark_price | String | 标记价格 |
timestamp | String | 系统时间戳 |

### # 返回示例
> {<br>
    "table": "swap/mark_price",<br>
    "data": [{<br>
        "instrument_id": "btcusd",<br>
        "mark_price": "5620.9",<br>
        "timestamp": "1559544244016"<br>
    }]<br>
}<br>

-
