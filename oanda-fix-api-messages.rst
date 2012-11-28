.. include:: <isonum.txt>

.. |-->| unicode:: U+02192 .. right arrow
   :trim:

OANDA FIX API Messages
======================

Specifications for the OANDA fxTrade and fxTrade Practice
FIX Server version 2.4.x

supporting FIX Protocol versions 4.2, 4.3, and 4.4

Last updated November 2012

.. contents:: Table of Contents

Message directions are stated with respect to the client.
Messages travelling in the client |-->| OANDA direction are termed
"outbound"; messages in the OANDA |-->| client direction are
"inbound".

Supported Messages
------------------

Session Messages
~~~~~~~~~~~~~~~~

+------------------+-----------+---------------------------------+
| message          | direction | description                     |
+==================+===========+=================================+
| Heartbeat <0>    | out/in    | sent automatically by an engine |
|                  |           | during times of inactivity to   |
|                  |           | indicate that the connection is |
|                  |           | alive                           |
+------------------+-----------+---------------------------------+
| Test Request <1> | out/in    | sent automatically by an engine |
|                  |           | if an incoming message has not  |
|                  |           | arrived within the agreed heart |
|                  |           | beat interval                   |
+------------------+-----------+---------------------------------+
| Reject <3>       | in        | indicates a session error       |
+------------------+-----------+---------------------------------+


Administrative Messages
~~~~~~~~~~~~~~~~~~~~~~~

+------------------+-----------+---------------------------------+
| message          | direction | description                     |
+==================+===========+=================================+
| Logon <A>        | out/in    | initiates a FIX connection to   |
|                  |           | the OANDA Server                |
+------------------+-----------+---------------------------------+
| News <B>         | in        | gives information on the server |
|                  |           | connected to                    |
+------------------+-----------+---------------------------------+
| Logout <5>       | out/in    | terminates a FIX connection     |
+------------------+-----------+---------------------------------+
| Business Message | in        | indicates a client message that |
| Reject <j>       |           | does not meet business          |
|                  |           | requirements that cannot be     |
|                  |           | rejected by any other message   |
+------------------+-----------+---------------------------------+

Application Messages
~~~~~~~~~~~~~~~~~~~~

Rates Connection Messages
+++++++++++++++++++++++++

Market data messages are only supported on a rates connection.

+------------------+-----------+---------------------------------+
| message          | direction | description                     |
+==================+===========+=================================+
| Market Data      | out       | requests market data snapshot   |
| Request <V>      |           | or subscription, or cancels     |
|                  |           | existing subscription           |
+------------------+-----------+---------------------------------+
| Market Data      | in        | the current market data for the |
| Snapshot / Full  |           | requested symbols               |
| Refresh <W>      |           |                                 |
+------------------+-----------+---------------------------------+
| Market Data      | in        | market data updates for the     |
| Incremental      |           | requested symbols               |
| Refresh <X>      |           |                                 |
+------------------+-----------+---------------------------------+
| Market Data      | in        | indicates market data request   |
| Request Reject   |           | errors                          |
| <Y>              |           |                                 |
+------------------+-----------+---------------------------------+

Order Connection Messages
+++++++++++++++++++++++++

All order-related messaging must be sent on the order connection.

+------------------+-----------+---------------------------------+
| message          | direction | description                     |
+==================+===========+=================================+
| New Order Single | out       | submit a new order              |
| <D>              |           |                                 |
+------------------+-----------+---------------------------------+
| Order Cancel     | out       | cancel an existing order        |
| Request <F>      |           |                                 |
+------------------+-----------+---------------------------------+
| Order Cancel /   | out       | modify an existing order        |
| Replace Request  |           |                                 |
| <G>              |           |                                 |
+------------------+-----------+---------------------------------+
| Order Status     | out       | request information on an       |
| Request <H>      |           | existing order                  |
+------------------+-----------+---------------------------------+
| Execution Report | in        | current status of a order or    |
| <8>              |           | order submission                |
+------------------+-----------+---------------------------------+
| Order Cancel     | in        | indicates Order Cancel Request  |
| Reject <9>       |           | <F> and Order Cancel / Replace  |
|                  |           | Request <G> errors              |
+------------------+-----------+---------------------------------+

Message Descriptions
--------------------

Tags not listed in message descriptions are ignored if supplied in a
request.

Headers and Trailers
~~~~~~~~~~~~~~~~~~~~

Header Fields
+++++++++++++

+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Tag | Field Name   | Required | Type / Value                                | Description                                                                                                                       |
+=====+==============+==========+=============================================+===================================================================================================================================+
| 8   | BeginString  | Y        | string, "FIX.4.2" or "FIX.4.3" or "FIX.4.4" | start of FIX message; must be first tag in message                                                                                |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 9   | BodyLength   | Y        | int                                         | message length in bytes, forward to checksum field; must be second field in a message                                             |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 35  | MsgType      | Y        | string                                      | the specific message type; must be third field in a message                                                                       |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 49  | SenderCompID | Y        | string                                      | identifies the sender of the message; for outbound messages, the login username, inbound messages will have the value "OANDA"     |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 56  | TargetCompID | Y        | string                                      | identifies the destination of the message; outbound messages use the value "OANDA", inbound messages will have the login username |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 34  | MsgSeqNum    | Y        | int                                         | message sequence number                                                                                                           |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 52  | SendingTime  | Y        | UTC timestamp                               | time of message transmission in UTC; it is highly recommended the customer supply millisecond-resolution timestamps               |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 50  | SenderSubID  | N        | string                                      | for rates connections, the OANDA server will state "RATES"                                                                        |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 57  | TargetSubID  | N        | string                                      | for rates connections, all outgoing messages must state "RATES"                                                                   |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+

Trailer Fields
++++++++++++++

+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Tag | Field Name   | Required | Type / Value                                | Description                                                                                                                       |
+=====+==============+==========+=============================================+===================================================================================================================================+
| 10  | CheckSum     | Y        | 3-digit string                              | three byte simple checksum and subsequent SOH character serve as the end-of-message marker                                        |
+-----+--------------+----------+---------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+

Administrative Messages
~~~~~~~~~~~~~~~~~~~~~~~

Logon <A>
+++++++++

The Logon <A> message authenticates a user and starts a connection.
It must be the first message sent by the client FIX engine.

Upon receipt of a Logon message the OANDA server will authenticate the
client using the provided password.  A Logon response is returned on
successful authentication.

If the server cannot authenticate the
logon request, no response is provided -- this is done as a precaution
against Denial of Service ("DoS") attacks.

+-----------+------------------+----------+--------------+-----------------------------------------------------------------+
| Tag       | Field Name       | Required | Type / Value | Description                                                     |
+===========+==================+==========+==============+=================================================================+
|           | standard headers | Y        |              | MsgType=A                                                       |
+-----------+------------------+----------+--------------+-----------------------------------------------------------------+
| 98        | EncryptMethod    | Y        | char "0"     | method of encryption                                            |
|           |                  |          |              |                                                                 |
|           |                  |          |              | * "0" - none                                                    |
+-----------+------------------+----------+--------------+-----------------------------------------------------------------+
| 108       | HeartBtInt       | Y        | int          | heart beat interval in seconds                                  |
|           |                  |          |              |                                                                 |
|           |                  |          |              | minimum value 30, recommended value 300                         |
+-----------+------------------+----------+--------------+-----------------------------------------------------------------+
|           | logon password   | Y        |              | see below                                                       |
+-----------+------------------+----------+--------------+-----------------------------------------------------------------+
| 141       | ResetSeqNumFlag  | Y        | boolean      |                                                                 |
|           |                  |          | "Y"/"N"      | indicates both sides should reset sequence numbers              |
|           |                  |          |              |                                                                 |
|           |                  |          |              | * "Y" - reset sequence numbers                                  |
+-----------+------------------+----------+--------------+-----------------------------------------------------------------+
|           | standard trailer | Y        |              |                                                                 |
+-----------+------------------+----------+--------------+-----------------------------------------------------------------+

FIX.4.2 connections specify the logon password via

+-----+---------------+--------------+--------------------------------------------------------+
| Tag | Field Name    | Type / Value | Description                                            |
+=====+===============+==============+========================================================+
| 95  | RawDataLength | int          | number of bytes in RawData field                       |
+-----+---------------+--------------+--------------------------------------------------------+
| 96  | RawData       | data         | unformatted raw data - provide the logon password here |
+-----+---------------+--------------+--------------------------------------------------------+

FIX.4.3+ connections specify the logon password via

+-----+---------------+--------------+--------------------------------------------------------+
| Tag | Field Name    | Type / Value | Description                                            |
+=====+===============+==============+========================================================+
| 554 | Password      | string       | the logon password                                     |
+-----+---------------+--------------+--------------------------------------------------------+

On a successful Logon, the server sends a News <B> message with
information on the server connected to.

News <B>
++++++++

The News <B> message provides important information about the
current server and connection.  This information is often imporant in
diagnosing problems.

Clients are advised to log this information for each connection in
their engine logs.

+-------------+------------------+----------+--------------+-------------------------------------------------------------+
| Tag         | Field Name       | Required | Type / Value | Description                                                 |
+=============+==================+==========+==============+=============================================================+
|             | standard headers | Y        |              | MsgType=B                                                   |
+-------------+------------------+----------+--------------+-------------------------------------------------------------+
| 148         | Headline         | Y        | string       | "OANDA FIX Server Information"                              |
+-------------+------------------+----------+--------------+-------------------------------------------------------------+
| 33          | LinesOfText      | Y        | int          | indicates the number of Text tags following                 |
+-------+-----+------------------+----------+--------------+-------------------------------------------------------------+
| |-->| | 58  | Text             | Y        | string       | "keyword: value" information                                |
+-------+-----+------------------+----------+--------------+-------------------------------------------------------------+
|             | standard trailer | Y        |              |                                                             |
+-------------+------------------+----------+--------------+-------------------------------------------------------------+

The Text tag provides information on topics such as:

* version - the current version of the server

* notice - important information about the server

* warning - explicitly flags issues such as compatibility concerns or
  behavioral changes

Logout <5>
++++++++++

The Logout message initiates or confirms the termination of a FIX
connection.  A connection disconnected without the exchange of Logout
message should be interpreted as an abnormal condition (e.g. network
failure).

+-----------+------------------+----------+--------------+---------------------------------------------------------------+
| Tag       | Field Name       | Required | Type / Value | Description                                                   |
+===========+==================+==========+==============+===============================================================+
|           | standard headers | Y        |              | MsgType=5                                                     |
+-----------+------------------+----------+--------------+---------------------------------------------------------------+
| 58        | Text             | N        | string       | optional explanatory text                                     |
+-----------+------------------+----------+--------------+---------------------------------------------------------------+
|           | standard trailer | Y        |              |                                                               |
+-----------+------------------+----------+--------------+---------------------------------------------------------------+

Business Message Reject <j>
+++++++++++++++++++++++++++

The Business Message Reject rejects application-level messages that
satisfy session rules but cannot be rejected by other means.

Clients are urged to consult the reasons and text provided to assist
in problem resolution.

+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
| Tag       | Field Name           | Required | Type / Value | Description                                                   |
+===========+======================+==========+==============+===============================================================+
|           | standard headers     | Y        |              | MsgType=j                                                     |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
| 45        | RefSeqNum            | N        | int          | MsgSeqNum of the rejected message                             |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
| 372       | RefMsgType           | Y        | string       | MsgType of the rejected message                               |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
| 379       | BusinessRejectRefID  | N        | string       | value of the business-level ID field being referenced         |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
| 380       | BusinessRejectReason | Y        | int          | code to identify reason for reject                            |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
| 58        | Text                 | N        | string       | explains the rejection                                        |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+
|           | standard trailer     | Y        |              |                                                               |
+-----------+----------------------+----------+--------------+---------------------------------------------------------------+

Please consult the official FIX specifications for
BusinessRejectReason valies.

Rates Connection Messages
~~~~~~~~~~~~~~~~~~~~~~~~~

Market Data Request <V>
+++++++++++++++++++++++

FIX clients access OANDA real-time prices using a Market Data Request
message.

Types of Market Data Requests:

* Snapshots (SubscriptionRequestType=0) are best for one-time
  requests.  If you require rates to be continually updated, we
  recommend the subscription request (SubscriptionRequestType=1)
  rather than rapid snapshot polling.

* Subscriptions (SubscriptionRequestType=1) request a snapshot of the
  current rates be returned and updates continually provided until the
  subscription is canceled.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=V                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 262         | MDReqID                 | Y                    | string       | *unique* identifier for new market data request, or the ID of |
|             |                         |                      |              | a previous request to disable if SubscriptionRequestType=2    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 263         | SubscriptionRequestType | Y                    | char         | type of request:                                              |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 0 = snapshot only                                           |
|             |                         |                      |              | * 1 = snapshot plus updates (subscription)                    |
|             |                         |                      |              | * 2 = disable previous subscription                           |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 264         | MarketDepth             | Y                    | int          | depth of market to return                                     |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 0 = all levels                                              |
|             |                         |                      |              | * N = return first N levels                                   |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 265         | MDUpdateType            | Y for subscriptions, |              |                                                               |
|             |                         | N otherwise          | int          | for subscriptions, indicates the desired update type:         |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 0 = full refresh                                            |
|             |                         |                      |              | * 1 = incremental refresh                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 267         | NoMDEntryTypes          | Y                    | int          | the number of MDEntryType entries following                   |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| |-->| | 269 | MDEntryType             | Y                    | char         | entries requested:                                            |
|       |     |                         |                      |              |                                                               |
|       |     |                         |                      |              | * 0 = bid                                                     |
|       |     |                         |                      |              | * 1 = offer                                                   |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 146         | NoRelatedSym            | Y                    | int          | the number of Symbol entries following                        |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| |-->| | 55  | Symbol                  | Y                    | string       | currency pair                                                 |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

Market Data Snapshot / Full Refresh <W>
+++++++++++++++++++++++++++++++++++++++

The Market Data Snapshot / Full Refresh message communicates the
current price of the requested Symbol at one point in time.  This
message is the response to a valid snapshot request, the first
response to a valid subscription request, and the message used to
communicate all updates for subscriptions where MDUpdateType=0 was 
requested.

A snapshot message gives market data for one symbol only.  If multiple
symbols were requested in a snapshot or subscription, multiple
<W> messages are returned to communicate the market data.

A snapshot message will give both bid and offer prices if both were
requested ("``267=2 269=0 269=1``").

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=W                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 262         | MDReqID                 | N                    | string       | ID of requesting Market Data Request                          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 55          | Symbol                  | Y                    | string       | currency pair                                                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 268         | NoMDEntries             | Y                    | int          | number of market data entries following                       |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| |-->| |     |                         |                      |              |                                                               |
|       | 269 | MDEntryType             | Y                    | char         | type of market data entry:                                    |
|       |     |                         |                      |              |                                                               |
|       |     |                         |                      |              | * 0 = bid                                                     |
|       |     |                         |                      |              | * 1 = offer                                                   |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 270 | MDEntryPx               | Y                    | price        | price of market data entry                                    |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 271 | MDEntrySize             | N                    | int          | price tier / number of units available                        |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 272 | MDEntryDate             | N                    | UTC date     | date of market data entry                                     |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 273 | MDEntryTime             | N                    | UTC time     | time of market data entry                                     |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 276 | QuoteCondition          | N                    | multiple     |                                                               |
|       |     |                         |                      | value        |                                                               |
|       |     |                         |                      | string       | conditions on market data entry                               |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 64  | FutSettDate             | N                    | LocalMktDate | for trades on the symbol, the specific settlement date for    |
|       |     | (FIX.4.2, FIX.4.3),     | (non-standard)       |              | customers with prime brokerage arrangements                   |
|       |     | SettlDate               |                      |              |                                                               |
|       |     | (FIX.4.4+)              |                      |              | value date accurate for MDEntryDate / MDEntryTime reported    |
|       |     |                         |                      |              |                                                               |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 58  | Text                    | N                    | string       | notes on market data entry                                    |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

SettlDate <64> is only provided to customers with prime brokerage 
arrangements and other specifically-configured customers.

Indicative prices are marked "Indicative" in the text notes in
addition to QuoteCondition=B.

Halted symbols are marked "Halted" in the text notes in addition to
QuoteCondition=B.

When a symbol is only tradeable in certain account home currencies,
the text notes will report

* "Tradeable when account currency is *currency*"
* "Tradeable when account currency is one of *[currencylist]*"

Market Data Incremental Refresh <X>
+++++++++++++++++++++++++++++++++++

The Market Data Incremental Refresh message communicates all price
updates for subscriptions where MDUpdateType=1 was requested.

An incremental refresh message can provide market data for more than
one symbol.

An incremental refresh message will give both bid and offer prices if
both were requested ("``267=2 269=0 269=1``").

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=X                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 262         | MDReqID                 | N                    | string       | ID of requesting Market Data Request                          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 268         | NoMDEntries             | Y                    | int          | number of market data entries following                       |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| |-->| |     |                         |                      |              |                                                               |
|       | 279 | MDUpdateAction          | Y                    | char         | type of update:                                               |
|       |     |                         |                      |              |                                                               |
|       |     |                         |                      |              | * 1 = change                                                  |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 269 | MDEntryType             | Y                    | char         | type of market data entry:                                    |
|       |     |                         |                      |              |                                                               |
|       |     |                         |                      |              | * 0 = bid                                                     |
|       |     |                         |                      |              | * 1 = offer                                                   |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 55  | Symbol                  | N                    | string       | currency pair                                                 |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 270 | MDEntryPx               | N                    | price        | price of market data entry                                    |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 271 | MDEntrySize             | N                    | int          | price tier / number of units available                        |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 272 | MDEntryDate             | N                    | UTC date     | date of market data entry                                     |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 273 | MDEntryTime             | N                    | UTC time     | time of market data entry                                     |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 276 | QuoteCondition          | N                    | multiple     |                                                               |
|       |     |                         |                      | value        |                                                               |
|       |     |                         |                      | string       | conditions on market data entry                               |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 64  | FutSettDate             | N                    | LocalMktDate | for trades on the symbol, the specific settlement date for    |
|       |     | (FIX.4.2, FIX.4.3),     | (non-standard)       |              | customers with prime brokerage arrangements                   |
|       |     | SettlDate               |                      |              |                                                               |
|       |     | (FIX.4.4+)              |                      |              | value date accurate for MDEntryDate / MDEntryTime reported    |
|       |     |                         |                      |              |                                                               |
|       +-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|       | 58  | Text                    | N                    | string       | notes on market data entry                                    |
+-------+-----+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

SettlDate <64> is only provided to customers with prime brokerage 
arrangements and other specifically-configured customers.

Please also consult the notes on market data snapshot messages.

Market Data Request Reject <Y>
++++++++++++++++++++++++++++++

The Market Data Request Reject message is returned to the client when
a market data request cannot be honored due to business or technical
reasons.  The tags and text field will describe the reason for the
rejection.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=Y                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 262         | MDReqID                 | N                    | string       | ID of requesting Market Data Request                          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 281         | MDReqRejReason          | N                    | char         | rejection reason                                              |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 58          | Text                    | N                    | string       | optional explanatory text                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

Order Connection Messages
~~~~~~~~~~~~~~~~~~~~~~~~~

New Order Single <D>
++++++++++++++++++++

The New Order Single message requests an order be placed.

The Execution Report reply will provide information on the result of
the order request, or give the reason for its rejection.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=D                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 11          | ClOrdID                 | Y                    | string       | unique order identifier assigned by the client                |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 1           | Account                 | Y                    | numeric      | the OANDA fxTrade practice or live account number the order   |
|             |                         |                      |              | is booked to                                                  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 21          | HandlInst               | Y (FIX.4.2, FIX.4.3),|              |                                                               |
|             |                         | N (FIX.4.4)          | "1"          | instructions for order handling:                              |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 1 = automated, private, no intervention                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 110         | MinQty                  | N                    | qty          | minimum quantity required for execution; only valid for       |
|             |                         |                      |              | TimeInForce = 3 or 4                                          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 55          | Symbol                  | Y                    | string       | currency pair                                                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 54          | Side                    | Y                    | char         |                                                               |
|             |                         |                      |              | * 1 = buy                                                     |
|             |                         |                      |              | * 2 = sell                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 60          | TransactTime            | Y                    | UTC timestamp| time of new order request; the timestamp is used to determine |
|             |                         |                      |              | if the request is potentially stale                           |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 38          | OrderQty                | Y                    | qty          | number of units ordered - value *must* be integer             |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 40          | OrdType                 | Y                    | char         | order type:                                                   |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 1 = market                                                  |
|             |                         |                      |              | * 2 = limit                                                   |
|             |                         |                      |              | * 3 = stop                                                    |
|             |                         |                      |              | * J = market-if-touched (FIX.4.3+ only)                       |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 44          | Price                   | Y for limit and      |              |                                                               |
|             |                         | market-if-touched    |              |                                                               |
|             |                         | orders only          | price        | limit or market-if-touched trigger price                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 99          | StopPx                  | Y for stop orders    |              |                                                               |
|             |                         | only                 | price        | stop price                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 59          | TimeInForce             | N                    | char         | see below                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 432         | ExpireDate              | one of ExpireDate /  |              |                                                               |
|             |                         | ExpireTime is        |              |                                                               |
|             |                         | required when        |              |                                                               |
|             |                         | TimeInForce=6        |              |                                                               |
|             |                         |                      | LocalMktDate | for GTD orders, requests expiry at 5pm Eastern (EST or EDT)   |
|             |                         |                      |              | on local market date indicated                                |
+-------------+-------------------------+                      +--------------+---------------------------------------------------------------+
| 126         | ExpireTime              |                      |              |                                                               |
|             |                         |                      |              |                                                               |
|             |                         |                      |              |                                                               |
|             |                         |                      | UTC timestamp| for GTD orders, requests expiry at exact timestamp indicated  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

The order type determines what TimeInForce values are supported:

+ market order

  * no value specified - requests standard market order
  * FOK - fill-or-kill market order
  * IOC - immediate-or-cancel market order

+ limit order, stop order

  * no value specified - requests DAY expiry
  * DAY - expires at 5pm Eastern time subject to submission before 4:55pm
  * GTD - expires at indicated ExpireDate or ExpireTime
  * FOK - fill-or-kill
  * IOC - immediate-or-cancel

+ market-if-touched order

  * no value specified - requests DAY expiry
  * DAY - expires at 5pm Eastern time subject to submission before 4:55pm
  * GTD - expires at indicated ExpireDate or ExpireTime

Order Cancel Request <F>
++++++++++++++++++++++++

The Order Cancel Request requests cancellation of an order.

The cancel request will only be accepted (and execution report
returned) if the order hasn't already been executed and can be
cancelled successfully; otherwise an order cancel reject is returned.

The OANDA server supports full cancellations only; any OrderQty
provided is ignored.  To reduce the quantity of an order use the Order
Cancel / Replace Request instead.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=F                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 37          | OrderID                 | N                    | string       | unique id for order assigned by OANDA;                        |
|             |                         |                      |              | used to identify order if provided                            |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 41          | OrigClOrdID             | Y                    | string       | identifies order to cancel in this request                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 11          | ClOrdID                 | Y                    | string       | *new* unique order identifier assigned by the client, if      |
|             |                         |                      |              | request is successful                                         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 1           | Account                 | Y                    | numeric      | the OANDA fxTrade practice or live account number the order   |
|             |                         |                      |              | is booked to                                                  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 55          | Symbol                  | Y                    | string       | *existing* currency pair                                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 54          | Side                    | Y                    | char         | *existing* side                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 60          | TransactTime            | Y                    | UTC timestamp| time of order cancel request; the timestamp is used to        |
|             |                         |                      |              | determine if the request is potentially stale                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

The OrigClOrdID, Symbol, and Side must *always* match the existing
order.  If an OrderID is provided it is used to uniquely identify
the order; this is required if the OrigClOrdID does not uniquely
identify the order (i.e., duplicate ClOrdID values were used).

If the request is successful the order is identified by the new
ClOrdID, otherwise the OrigClOrdID remains the order ClOrdID.

Order Cancel / Replace Request <G>
++++++++++++++++++++++++++++++++++

The Order Cancel / Replace Request (sometimes called a modify request)
is used to change the parameters of an existing order.

This request can be used to change the price, stop price, order expiry
time, and order quantity for existing limit, stop, and
market-if-touched orders with expiry time remaining.

Market orders and limit/stop orders requested FOK or IOC execute 
immediately on reception and cannot be canceled or replaced.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=G                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 37          | OrderID                 | N                    | string       | unique id for order assigned by OANDA;                        |
|             |                         |                      |              | used to identify order if provided                            |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 41          | OrigClOrdID             | Y                    | string       | identifies order to cancel in this request                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 11          | ClOrdID                 | Y                    | string       | *new* unique order identifier assigned by the client, if      |
|             |                         |                      |              | request is successful                                         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 1           | Account                 | Y                    | numeric      | the OANDA fxTrade practice or live account number the order   |
|             |                         |                      |              | is booked to                                                  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 21          | HandlInst               | Y (FIX.4.2, FIX.4.3),|              |                                                               |
|             |                         | N (FIX.4.4)          | "1"          | instructions for order handling:                              |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 1 = automated, private, no intervention                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 55          | Symbol                  | Y                    | string       | currency pair                                                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 54          | Side                    | Y                    | char         |                                                               |
|             |                         |                      |              | * 1 = buy                                                     |
|             |                         |                      |              | * 2 = sell                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 60          | TransactTime            | Y                    | UTC timestamp| time of order cancel/replace request; the timestamp is used   |
|             |                         |                      |              | to determine if the request is potentially stale              |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 38          | OrderQty                | Y                    | qty          | number of units ordered - value *must* be integer             |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 40          | OrdType                 | Y                    | char         | order type:                                                   |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 2 = limit                                                   |
|             |                         |                      |              | * 3 = stop                                                    |
|             |                         |                      |              | * J = market-if-touched (FIX.4.3+ only)                       |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 44          | Price                   | Y for limit and      |              |                                                               |
|             |                         | market-if-touched    |              |                                                               |
|             |                         | orders only          | price        | limit or market-if-touched trigger price                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 99          | StopPx                  | Y for stop orders    |              |                                                               |
|             |                         | only                 | price        | stop price                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 59          | TimeInForce             | N                    | char         | valid values:                                                 |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * no value specified = DAY                                    |
|             |                         |                      |              | * 0 = DAY                                                     |
|             |                         |                      |              | * 6 = GTD                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 432         | ExpireDate              | one of ExpireDate /  |              |                                                               |
|             |                         | ExpireTime is        |              |                                                               |
|             |                         | required when        |              |                                                               |
|             |                         | TimeInForce=6        |              |                                                               |
|             |                         |                      | LocalMktDate | for GTD orders, requests expiry at 5pm Eastern (EST or EDT)   |
|             |                         |                      |              | on local market date indicated                                |
+-------------+-------------------------+                      +--------------+---------------------------------------------------------------+
| 126         | ExpireTime              |                      |              |                                                               |
|             |                         |                      |              |                                                               |
|             |                         |                      |              |                                                               |
|             |                         |                      | UTC timestamp| for GTD orders, requests expiry at exact timestamp indicated  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

The OrigClOrdID, Symbol, and Side must *always* match the existing
order.  If an OrderID is provided it is used to uniquely identify
the order; this is required if the OrigClOrdID does not uniquely
identify the order (i.e., duplicate ClOrdID values were used).

If the request is successful the order is identified by the new
ClOrdID, otherwise the OrigClOrdID remains the order ClOrdID.

Order Status Request <H>
++++++++++++++++++++++++

Information on previous orders is available via the Order Status
Request.  An Execution Report supplies the information.

Status information is guaranteed to be available for at least one
month after order completion (fill, expiry, cancellation).

No information is recorded for OrdStatus=REJECTED orders.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=H                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 37          | OrderID                 | N                    | string       | unique id for order assigned by OANDA;                        |
|             |                         |                      |              | used to identify order if provided                            |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 11          | ClOrdID                 | Y                    | string       | unique order identifier assigned by the client                |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 790         | OrdStatusReqID          | N                    | string       | if provided, is echoed back in Execution Report               |
|             |                         | (non-standard prior  |              |                                                               |
|             |                         | to FIX.4.4)          |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 55          | Symbol                  | Y                    | string       | currency pair                                                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 54          | Side                    | Y                    | char         |                                                               |
|             |                         |                      |              | * 1 = buy                                                     |
|             |                         |                      |              | * 2 = sell                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

The ClOrdID, Symbol, and Side must *always* match the existing
order.  If an OrderID is provided it is used to uniquely identify
the order; this is required if the ClOrdID does not uniquely
identify the order (i.e., duplicate ClOrdID values were used).

The OANDA server supports OrdStatusReqID in FIX.4.2 and FIX.4.3
connections; the client engine must be prepared to accept the tag in
the Execution Report if one was provided in an Order Status Request.

Execution Report <8>
++++++++++++++++++++

The Execution Report communicates the result of an order submission,
change, or cancellation.  Specifically, this message:

* confirms acceptance or rejection of a new order
* confirms acceptance of changes to existing order

Rejected order cancel and cancel/replace requests are communicated via
Order Cancel Reject.

Note that the format of an Execution Report message varies by FIX
version. In FIX.4.2, each execution report contains three fields that
are used to communicate both the current state of the order and the
purpose of the message: OrdStatus <39>, ExecType <150> and
ExecTransType <20>. In FIX.4.3+, each execution report contains
two fields that are used to communicate both the current state of the
order (OrdStatus <39>) and the purpose of the message (ExecType
<150>).

In execution reports, the TimeInForce <59> tag that is reported
for a long-duration order will be reported as DAY in response to a
request where TimeInForce=DAY was specified. In general, order expiry
times will be reported using TimeInForce=GTD with the exact UTC expiry
time reported via ExpireTime <126>.

The Text <58> field provides supplemental information about the
order execution. It consists of phrases or sentences separated with a
period and space. Execution reports also list the OANDA transaction
IDs (transaction tickets) that correspond to the FIX order.

An Execution Report for a market-if-touched order appearing on a
FIX.4.2 connection will be reported as OrdType <40> = 2 with a
Text <58> annotation "OrdType=J".

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=8                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 37          | OrderID                 | N                    | string       | unique id for order assigned by OANDA                         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 11          | ClOrdID                 | Y                    | string       | unique order identifier assigned by the client                |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 41          | OrigClOrdID             | N                    | string       | previous order identifier assigned by the client              |
|             |                         |                      |              | for order cancel and cancel/replace request responses         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 790         | OrdStatusReqID          | N                    | string       | provided if present in requesting Order Status Request <H>    |
|             |                         | (non-standard prior  |              |                                                               |
|             |                         | to FIX.4.4)          |              | value from request echoed back                                |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 17          | ExecID                  | Y                    | string       | execution ID assigned by OANDA server                         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 20          | ExecTransType           |                      |              |                                                               |
|             | (FIX.4.2 only)          | Y                    | string       | identifies transaction type                                   |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 150         | ExecType                | Y                    | char         | describes specific execution report                           |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 39          | OrdStatus               | Y                    | char         | identifies the current order status                           |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 103         | OrdRejReason            | N                    | int          | code to identify order rejection reason                       |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 1           | Account                 | Y                    | numeric      | the OANDA fxTrade practice or live account number the order   |
|             |                         |                      |              | is booked to                                                  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 55          | Symbol                  | Y                    | string       | currency pair                                                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 461         | CFICode (FIX.4.4 only)  | N                    | string       | ISO 10962 compliant CFI code; value: MRCXXX                   |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 54          | Side                    | Y                    | char         |                                                               |
|             |                         |                      |              | * 1 = buy                                                     |
|             |                         |                      |              | * 2 = sell                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 38          | OrderQty                | Y                    | qty          | number of units ordered - integer value                       |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 40          | OrdType                 | Y                    | char         | order type:                                                   |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 1 = market                                                  |
|             |                         |                      |              | * 2 = limit                                                   |
|             |                         |                      |              | * 3 = stop                                                    |
|             |                         |                      |              | * J = market-if-touched (FIX.4.3+ only)                       |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 44          | Price                   | Y for limit and      |              |                                                               |
|             |                         | market-if-touched    |              |                                                               |
|             |                         | orders only          | price        | limit or market-if-touched trigger price                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 99          | StopPx                  | Y for stop orders    |              |                                                               |
|             |                         | only                 | price        | stop price                                                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 59          | TimeInForce             | Y                    | char         |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 432         | ExpireDate              | one of ExpireDate /  |              |                                                               |
|             |                         | ExpireTime is        |              |                                                               |
|             |                         | required when        |              |                                                               |
|             |                         | TimeInForce=6        |              |                                                               |
|             |                         |                      | LocalMktDate | for GTD orders, states expiry at 5pm Eastern (EST or EDT)     |
|             |                         |                      |              | on local market date indicated                                |
+-------------+-------------------------+                      +--------------+---------------------------------------------------------------+
| 126         | ExpireTime              |                      |              |                                                               |
|             |                         |                      |              |                                                               |
|             |                         |                      |              |                                                               |
|             |                         |                      | UTC timestamp| for GTD orders, states expiry at exact timestamp indicated    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 32          | LastShares (FIX.4.2) /  |                      |              |                                                               |
|             | LastQty (FIX.4.3+)      | N                    | qty          | units transacted on the most recent fill                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 31          | LastPx                  | N                    | price        | price of the most recent fill                                 |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 151         | LeavesQty               | Y                    | qty          | amount open for futher execution                              |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | if the OrdStatus is CANCELED, EXPIRED, or REJECTED, LeavesQty |
|             |                         |                      |              | is reported as 0 (i.e., no further qty for execution)         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 14          | CumQty                  | Y                    | qty          | total number of units filled on this order                    |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 6           | AvgPx                   | Y                    | price        | average price of all fills on this order                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 60          | TransactTime            | Y                    | UTC timestamp| time the transaction represented by this execution report     |
|             |                         |                      |              | occurred                                                      |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 21          | HandlInst               | Y (FIX.4.2, FIX.4.3),|              |                                                               |
|             |                         | N (FIX.4.4)          | "1"          | instructions for order handling:                              |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 1 = automated, private, no intervention                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 110         | MinQty                  | N                    | qty          | minimum quantity required for execution; only valid for       |
|             |                         |                      |              | TimeInForce = 3 or 4                                          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 58          | Text                    | N                    | string       | optional explanatory text                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+

Order Cancel Reject <9>
+++++++++++++++++++++++

The Order Cancel Reject message is issued in response to cancel and
cancel/replace requests which cannot be honored.

The ClOrdID for the order retains its original value as the cancel or cancel/replace did not occur.

+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| Tag         | Field Name              | Required             | Type / Value | Description                                                   |
+=============+=========================+======================+==============+===============================================================+
|             | standard headers        | Y                    |              | MsgType=9                                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 37          | OrderID                 | N                    | string       | unique id for order assigned by OANDA                         |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 11          | ClOrdID                 | Y                    | string       | unique order identifier assigned by the client                |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 41          | OrigClOrdID             | Y                    | string       | identifies order that could not be canceled/replaced          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 39          | OrdStatus               | Y                    | char         | identifies the current order status                           |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 1           | Account                 | Y                    | numeric      | the OANDA fxTrade practice or live account number the order   |
|             |                         |                      |              | is booked to                                                  |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 434         | CxlRejResponseTo        | Y                    | char         | indicates the request this reject is in response to:          |
|             |                         |                      |              |                                                               |
|             |                         |                      |              | * 1 = order cancel request                                    |
|             |                         |                      |              | * 2 = order cancel / replace request                          |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 102         | CxlRejReason            | N                    | int          | identifies reason for rejection                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
| 58          | Text                    | N                    | string       | optional explanatory text                                     |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
|             | standard trailer        | Y                    |              |                                                               |
+-------------+-------------------------+----------------------+--------------+---------------------------------------------------------------+
