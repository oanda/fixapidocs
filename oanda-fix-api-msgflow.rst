.. |-->| unicode:: U+02192 .. right arrow
   :trim:


OANDA FIX API -- Message Workflow
=================================

.. contents::

In the sample message sequences below, not all fields required are 
spelled out in the example.  Please consult the supported messages
documentation to see all required and optional tags.

Multiple Connections Supported
------------------------------

Clients may make multiple connections, to both order and rates
servers.

Clients must take steps to avoid overloading the OANDA servers.
For example, if a client has multiple applications reading market
data, one connection should be used to read the market data and supply
the values to all applications.

Any abuse of the OANDA system will result in revoked access privileges.

Note also that `Rate-Limiting`_ will affect how many messages a client
can send across all connections owned by the user.

Rate-Limiting
~~~~~~~~~~~~~

Clients are limited to sending a maximum of 100 messages/second to each 
server providing rates or orders services.  Messages exceeding this
limit are rejected with a BusinessMessageReject <3>,
``BusinessRejectReason=0`` (other),
``Text=Incoming message soft rate limit reached.``.

In instances of egregious message flooding the server will reply with a 
BusinessMessageReject <3>, 
``BusinessRejectReason=0`` (other),
``Text=Incoming message hard rate limit reached; disconnecting.``,
and the connection will be dropped.

Connection Setup / Teardown
---------------------------

A connection is set up with a Logon <A> message.  On a successful
Logon, the OANDA server will reply with a News <B> message giving
important information on the server connected to, as well as useful
troubleshooting information.

A connection is terminated normally with a Logout <5> message.

A socket disconnect will also terminate the connection.

A session does not survive a disconnection - each new connection is its
own session and must start with a Logon <A> message.

::

    client                                      OANDA server
       |                                              |
       | -- Logon <A> ------------------------------> |
       |                                              |
       | <------------------------------ Logon <A> -- |
       |                                              |
       | <------------------------------- News <B> -- |
       |                                              |
       |                                              |
       | .......... (normal message flow) ........... |
       |                                              |
       |                                              |
       | -- Logout <A> -----------------------------> |
       |                                              |
       | <----------------------------- Logout <A> -- |
       |                                              |



..  wsd
    participant client as A
    participant "OANDA server" as B
    A -> B : Logon <A>
    B -> A : Logon <A>
    note over A,B : do things
    A -> B : Logout <5>
    B -> A : Logout <5>

In case of an invalid password, no reply is returned as a precaution
against DoS or other network attacks.

::

    client                                      OANDA server
       |                                              |
       | -- Logon <A> ------------------------------> |
       | (with invalid password)                      |
       |                                              |
       |                                 (disconnect) X

..  wsd
    participant client as A
    participant "OANDA server" as B
    A -> B : Logon <A>
    note right of A : with invalid password
    note over B : disconnect

In case of serious session-level problems, the OANDA server may
preemtively disconnect a client.

Heartbeat / Test Request
------------------------

The generation of Heartbeat <0> and Test Request <1> messages is usually
the responsibility of the FIX engine.  These "keepalive" messages and 
timing recordkeeping are typically not explicitly performed by the end 
user's business logic.

Clients are expected to follow standard heartbeat generation interval as
negotiated via HeartBtInt during logon.

::

    client                                      OANDA server
       |                                              |
      no messages sent for                            |
      HeartBtInt seconds                              |
       |                                              |
       | -- Heartbeat <0> --------------------------> |
       |                                              |

..  wsd
    A -> B : messages
    note over A : no messages sent for HeartBtInt seconds
    A -> B : Heartbeat <0>

If either party has not received a message from the other party within
the negotiated HeartBtInt (plus a small amount of time for propagation 
delay) then the engine should generate a Test Request.  A Test Request
requires a specific response (a Heartbeat with the Test Request's 
TestReqID returned).

::

    client                                      OANDA server
       |                                              |
       |                        no messages received for
       |                      HeartBtInt + delay seconds
       |                                              |
       | <----------------------- Test Request <1> -- |
       |                                              |
       | -- Heartbeat <0> --------------------------> |
       | (with TestReqID <112> returned)              |
       |                                              |

..  wsd
    A -> B : messages
    note over A : no messages received for HeartBtInt + delay seconds
    A -> B : Test Request <1>
    B -> A : Heartbeat <0> with TestReqID <112> returned

If no test request response is received after a further time
interval, the test-requesting-party must assume the connection is 
broken, even if any other message arrives in the meantime.

::

    client                                      OANDA server
       |                                              |
       |                        no messages received for
       |                      HeartBtInt + delay seconds
       |                                              |
       | <----------------------- Test Request <1> -- |
       |                                              |
       |               no test request response received
       |                                              |
       |                     connection is deemed broken

..  wsd
    A -> B : messages
    note over A : no messages received for HeartBtInt + X seconds
    A -> B : Test Request <1>
    note over A : no test request response received for a further Y seconds
    note over A,B : connection is deemed broken

Market Data Connections
-----------------------

Market data is requested on a connection to a rates server, accomplished
by sending messages with ``TargetSubID=RATES`` in the header for every 
message on this connection.

Confirm in the News <B> message that you are connected to a market data
(rates) server.

::

    client                                      OANDA server
       |                                              |
       | -- Logon <A> ------------------------------> |
       | 57=RATES                                     |
       |                                              |
       | <------------------------------ Logon <A> -- |
       |                                     50=RATES |
       |                                              |
       | <------------------------------- News <B> -- |
       |                                              |

..  wsd
    participant client as A
    participant "OANDA server" as B
    A -> B : Logon <A> with TargetSubID=RATES
    B -> A : Logon <A> with SenderSubID=RATES

Snapshots
~~~~~~~~~

To request a snapshot, send a Market Data Request message with 
``SubscriptionRequestType=0`` and one or more symbols.  We recommend
always requesting both BID and OFFER MDEntryType types on every
request, and we recommend requesting all desired symbols in one 
request.

A snapshot giving the current pricing for each symbol is immediately
returned.

::

    client                                      OANDA server
       |                                              |
       | ........... on RATES connection ............ |
       |                                              |
       | -- Market Data Request <V> ----------------> |
       | 263=0 267=2 269=0 269=1                      |
       | 146=2 55=EUR/USD 55=USD/CAD                  |
       |                                              |
       | <-- Market Data Snapshot Full Refresh <W> -- |
       |                                      EUR/USD |
       |                                              |
       | <-- Market Data Snapshot Full Refresh <W> -- |
       |                                      USD/CAD |
       |                                              |

..  wsd
    participant client as A
    participant "OANDA server" as B
    note over A,B: on RATES connection
    A -> B : Market Data Request <V> with 263=0 267=2 269=0 269=1 146=2 55=EUR/USD 55=USD/CAD
    B -> A : Market Data Snapshot / Full Refresh <W> for EUR/USD
    B -> A : Market Data Snapshot / Full Refresh <W> for USD/CAD

Customers needing continuous rates updates are urged to request 
subscriptions instead of polling for snapshots.

Subscriptions
~~~~~~~~~~~~~

To request a subscription, send a Market Data Request message with
``SubscriptionRequestType=1`` and one or more symbols.  We recommend
always requesting both BID and OFFER ``MDEntryType`` types on every
request, and we recommend requesting all desired symbols in one 
request.

A snapshot giving the current pricing for each symbol is immediately
returned, followed by price updates as they occur.  In the example 
below, ``MDUpdateType=1`` (incremental refresh) is requested and
as a result, all updates are communicated with Market Data 
Incremental Refresh <X> messages; note that each <X> message may
contain market data for more than one symbol.

    Market data updates are sent only when price changes occur.
    Lulls in the arrival of market data are not a cause for concern.

To cancel a subscription, send a Market Data Request message with
``SubscriptionRequestType=2`` and indicate the MDReqID of the
subscription you wish to cancel.  Note that subscriptions are also
automatically canceled on a disconnect; you will need to resubscribe 
after a reconnection (Logon <A>) to continue receiving market data.

::

    client                                      OANDA server
       |                                              |
       | ........... on RATES connection ............ |
       |                                              |
       | -- Market Data Request <V> ----------------> |
       | 263=1 267=2 269=0 269=1                      |
       | 146=2 55=EUR/USD 55=USD/CAD                  |
       |                                              |
       | <-- Market Data Snapshot Full Refresh <W> -- |
       |                                      EUR/USD |
       |                                              |
       | <-- Market Data Snapshot Full Refresh <W> -- |
       |                                      USD/CAD |
       |                                              |
     --- on every price update for symbols requested ----
       | <---- Market Data Incremental Refresh <W> -- |
     ----------------------------------------------------
       |                                              |
       | -- Market Data Request <V> ----------------> |
       | 263=2                                        |
       | with MDReqID of subscription to cancel       |
       |                                              |
       |                            subscription canceled
       |                                              |

..  wsd
    participant client as A
    participant "OANDA server" as B
    note over A,B: on RATES connection
    A -> B : Market Data Request <V> with 263=1 267=2 269=0 269=1 146=2 55=EUR/USD 55=USD/CAD
    B -> A : Market Data Snapshot / Full Refresh <W> for EUR/USD
    B -> A : Market Data Snapshot / Full Refresh <W> for USD/CAD
    loop on market data updates
    B -> A : Market Data Incremental Refresh <X>
    end
    A -> B : Market Data Request <V> with 263=2
    note over B: subscription canceled

If ``MDUpdateType=0`` (full refresh) was requested, all updates are
communicated with Market Data Snapshot / Full Refresh <W> messages.

Order Connections
-----------------

An order connection is made by omitting any ``TargetSubID`` in the 
header.
Unrecognized values (any value other than ``RATES``) will also be routed 
to the order server. Any TargetSubID value will be returned in the 
``SenderSubID`` by the OANDA server, whether useful or not.

All orders are submitted by sending a New Order Single <D> message.
Orders with some order lifetime may be canceled via 
Order Cancel Request <F> and modified via
Order Cancel / Replace Request <G>.

Since sessions are only alive for the duration of a connection, any
order fill or expiry occurring when the client is disconnected will
not be reported.  Clients are urged to keep an accurate blotter of
all outstanding orders and query Order Status Request <H> to retrieve
order status when reconnecting.

Note the OANDA server does not record the status of any 
``OrdStatus=8`` (rejected) order.  An Order Status Request <H> 
requesting information on such an order will yield an unknown 
order result.

Market Orders
~~~~~~~~~~~~~

A regular market order is submitted with ``OrdType=1`` and omitting 
any ``TimeInForce`` value.  The order is either filled or rejected.

A market order (``OrdType=1``) submitted FOK/IOC will be filled or
canceled.  A canceled order is recorded in order history, and order
status is available.

Market orders and FOK/IOC orders enjoy the fastest turnaround time.

::

    client                                      OANDA server
       |                                              |
       | -- New Order Single <D> -------------------> |
       | 40=1                                         |
       |                                              |
       | <--------------------Execution Report <8> -- |
       |                                              |

Limit/Stop Orders submitted FOK/IOC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

FOK/IOC can be requested on market, limit, and stop orders.

Fill-or-Kill (``TimeInForce=4``) orders will either be filled in
their entirety, or canceled.  Immediate-or-Cancel (``TimeInForce=3``)
orders will be filled as much as is possible, and the remainder 
canceled.

A Limit order (``OrdType=2``) requests execution at the requested
``Price`` or "better" (lower on buy, higher on sell).  A Stop
order (``OrdType=3``) is an order that becomes a market order when the
market price is at or "worse" (higher on buy, lower on sell) than the
requested ``StopPx``.

FOK/IOC orders filled in full will result in ``OrdStatus=2`` (filled).
IOC orders partially filled and FOK/IOC orders not filled at all
result in ``OrdStatus=4`` as per the FPL specification.  To determine
the quantity filled, consult the Execution Report <8> ``ExecType``,
``CumQty``, and ``AvgPx`` values.

::

    client                                      OANDA server
       |                                              |
       | -- New Order Single <D> -------------------> |
       | 40=2 59=3                                    |
       |                                              |
       | <--------------------Execution Report <8> -- |
       |                                              |


Limit/Stop/Market-if-Touched Orders with Order Lifetime
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A Limit order (``OrdType=2``) requests execution at the requested
``Price`` or "better" (lower on buy, higher on sell).  A Stop
order (``OrdType=3``) is an order that becomes a market order when the
market price is at or "worse" (higher on buy, lower on sell) than the
requested ``StopPx``.
A Market-if-Touched order (``OrdType=J``) is an order that becomes a
market order when the market price touches or crosses the requested
``Price``.

Orders specified with a ``TimeInForce`` of ``0`` (day) or
``6`` (good-till-date), with a default of DAY, create an open order
on the OANDA system.  This order remains open until canceled or filled,
and can be modified (price, units, or order expiry time only).

The Execution Report <8> confirming order acceptance, cancellation,
or modification, as well as any Order Cancel Reject <9> for any failed
Order Cancel Request <F> or Order Cancel / Replace Request <G>,
is returned on the submitting order connection.  The Execution
Report <8> announcing order fill or expiry is broadcast to all order
connections currently open for the customer (all connections for the
order owner's ``SenderCompID``).

::

    client                                      OANDA server
       |                                              |
       | -- New Order Single <D> -------------------> |
       | 40=2 59=6                                    |
       |                                              |
       | <--------------------Execution Report <8> -- |
       |        (confirming order open on the server) |
       |                                              |
       | ............. time passes by ............... |
       |                                              |
       | -- Order Cancel / Replace Request <G> -----> |
       |                                              |
       |                          successful order modify
       |                                              |
       | <--------------------Execution Report <8> -- |
       |        (confirming order open on the server) |
       |                                              |
       | ............. time passes by ............... |
       |                                              |
       |                          order filled or expired
       |                                              |
       | <--------------------Execution Report <8> -- |
       |     (to all connections for the order owner) |
       |                                              |

Order Status Request <H>
~~~~~~~~~~~~~~~~~~~~~~~~

An Order Status Request <H> requests the most up-to-date information
on the indicated order.

::

    client                                      OANDA server
       |                                              |
       | -- Order Status Request <H> ---------------> |
       |                                              |
       | <--------------------Execution Report <8> -- |
       |                                              |
