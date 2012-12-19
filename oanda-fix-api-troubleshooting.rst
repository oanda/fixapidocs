.. _stunnel: http://www.stunnel.org/



==================================
 OANDA FIX API -- Troubleshooting
==================================

.. contents::

General Comments
================

Clients need to be prepared to investigate network problems, setup
their host operating system, and configure any and all software
packages used to connect to the OANDA API.

Clients will need to take the intiative in investigating problems
with the API access.  Many times, the reason for the problem is reported
to the user, either as a text note or in various "reason" FIX fields.

Problems with Connecting
========================

1.  Verify that the internet connection between the client engine and
    the OANDA engine is working.

    telnet to the server ``fxgame-fix.oanda.com`` or
    ``fxtrade-fix.oanda.com``, port 32009, to verify that the
    server is reachable.

2.  Verify that your connection particulars are correct.

    a.  hostname is correct for the system you wish to access

    b.  port number is correct

    c.  SSL encryption is available, either supported natively by the
        FIX engine, or provided separately

        Clients using stunnel_ as an SSL transport need to configure
        their stunnel service correctly -- take the time to understand
        the tool you are using!  You need to configure your engine to
        connect to your stunnel instance, and your stunnel service to
        connect to the OANDA server:

        ::

           +-----------------------------+
           |  FIX engine                 |
           +-----------------------------+ 
           | SocketConnectHost=localhost |
           | SocketConnectPort=30008     |
           +-----------------------------+
                        |
                        | localhost:30008
                        |
                        V
           +------------------------------------+
           |   stunnel instance                 |
           +------------------------------------+
           | client=yes                         |
           | verify=0                           |
           | socket=l:TCP_NODELAY=1             |
           | socket=r:TCP_NODELAY=1             |
           |                                    |
           | [OANDAFIX]                         |
           | accept=30008                       |
           | connect=fxgame-fix.oanda.com:32009 |
           | TIMEOUTclose=0                     |
           | ciphers=HIGH+SHA+AES               |
           +------------------------------------+
                         |
                         | fxgame-fix.oanda.com:32009
                         |
                         V
                 +---------------+
                 | OANDA FIX API |
                 +---------------+

        Clients configuring their FIX engine to connect directly to the
        OANDA server with a plaintext connection will see a
        disconnection.

    d.  login username is correct - the ``SenderCompID`` used for the
        server matches the login username for the server

    e.  password is correct

    f.  password is provided in the correct FIX fields

        * for FIX.4.2 sessions, use the ``RawDataLen`` and ``RawData``
          tags

        * for FIX.4.3+ sessions, use the ``Password`` tag

        Logon <A> requests with an incorrect username and/or password
        will see a disconnection.

3.  If an error reply is given to the Logon message

    a. check that ``ResetSeqNumFlag=Y`` and ``MsgSeqNum=1`` for the
       Logon request

    b. verify that the ``HeartBtInt`` value proposed is acceptable


4.  If your account is locked out due to an excessive number of failed
    logon attempts, contact customer service for an unlock.


Problems with Message Exchanges
===============================

In general, if the OANDA server sends an error reply the reply will
contain some indication of what is wrong

1.  Reject <3>

    * the rejected message is indicated by ``RefSeqNum`` and ``RefMsgType``

    * the problem tag is indicated by ``RefTagID``

    * the reason is given in ``SessionRejectReason``

    * often, the ``Text`` will explain what is wrong

    Common reasons for rejections include

    a. SendingTime accuracy problem

       This indicates that the ``SendingTime`` reported by the client
       engine differs from the local time at reception, either it is
       later than the local clock or preceeds it by over 2 seconds.

       These problems arise due to either:

       * misconfigured or missynchronized local clock

         The client should be prepared to synchronize the machine clock
         with an accurate time source.  On unix systems, an NTP setup
         with time servers listed directly (no pools please!) will
         suffice.  Windows users may need to look at
         `alternate time services <http://support.microsoft.com/kb/939322>`.

         Consult your ISP to see if a high-accuracy NTP server is
         available.

       * network problems delayed reception of message

         The client needs to investigate the cause of latency between
         their network and the OANDA server.

         Some potential causes include:

         * insufficient TCP window sizes for long-latency connections

         * TCP_NODELAY not configured for the connection

         * ISP may be throttling traffic

         You may need the assistance of your ISP to resolve latency
         issues.

    b. structurally-invalid FIX messages

       Messages that fail structural validation are rejected by the
       OANDA engine.  Client engines should have structural validation
       turned on to ensure messages sent are valid.

       Some common causes of structurally-invalid messages include:

       * ``TargetSubID=RATES`` appearing in the body of the message

       * tag group problems

         - the group leader (which usually indicates the number of group
           items following) must preceede the groups

         - each group has a specific tag that must be the first tag in
           the group


2.  Business Message Reject <j>

    * the rejected message is indicated by ``RefSeqNum`` and ``RefMsgType``

    * the ``BusinessRejectRefID`` field indicates the ID of the rejected
      message; the tag name is dependent on the ``RefMsgType``

    * consult the ``BusinessRejectReason`` and ``Text`` tags for an
      explanation

    Common reasons for rejections include

    a. rate limit reached

       This indicates the client has flooded the OANDA servers with an
       excessive number of requests.

       Customers repeatedly flooding the server risk having their
       access denied.

    b. OANDA server offline

       In very rare instances the OANDA server may be unavailable.

       OANDA has technical staff monitoring the systems continuously;
       any downtime is expected to be very short.

3.  Malformed messages submitted

    If the OANDA server receives a severely malformed message, the 
    message is dropped as per FIX Protocol Ltd spec.  You will not 
    receive any acknowledgement or reply for such a dropped message.


Problems with Market Data
=========================

The Market Data Request Reject <Y> message returned on any request
problems contains tags and text to describe the problem.  Consult
the MDReqRejReason <281> and Text <58>.

Common problems include:

1.  Duplicate MDReqID <262> values

    Each Market Data Request (except for unsubscribe requests) must
    use a unique MDReqID.

2.  Unknown MDReqID <262> value

    An unsubscribe request must name the MDReqID of the subscription
    to cancel

3.  Duplicate Symbols

    A symbol may not be the subject of multiple subscriptions.

4.  Unknown / Indicative Symbols

    The symbols available for trading differ depending on the division
    the customer is registered in.


Problems with Order Execution
=============================

Most problems with order execution can be solved by examining the 
Text <58> tag.  Consult the
`Text <58> Tag Sample Messages`_
to see what kind of information is provided.

Text <58> Tag Sample Messages
=============================

Most FIX messages have an optional ``Text <58>`` tag.  This tag is often
filled with an explanation of the current situation, an error message,
or some extra information pertinent to the message.

Considerable effort has been spent to make the reported messages useful.
This tag value should always be consulted first because the answer is
often right in the message!

The format of the messages is a list of sentences or phrases, each
ending with a dot.  Items are separated by a single space.

+------------------+-----------------+---------------+--------------------------------------+
| Situation        | Text format     | Example Text  | Explanation                          |
+==================+=================+===============+======================================+
| required tag     | *tag* required  | OrderQty <38> | tag and value must be provided in    |
| missing          |                 | required      | the message                          |
+------------------+-----------------+---------------+--------------------------------------+
| tag value        | *tag* = *value* | Side <54> = 4 | choose a supported value             |
| supplied is not  | not supported   | not supported |                                      |
| supported        |                 |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| tag value not    | *tag* value     | OrderQty <38> | order qty must be a positive integer |
| valid            | invalid         | value invalid |                                      |
|                  |                 +---------------+--------------------------------------+
|                  |                 | Account <1>   | account number must be a positive    |
|                  |                 | value invalid | integer                              |
+------------------+-----------------+---------------+--------------------------------------+
| tag value in     | *tag* format    | OrderID <37>  | order id must be a numeric value     |
| incorrect        | error           | format error  |                                      |
| format           |                 |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| tag value not    | *tag* not       |               |                                      |
| valid for the    | valid when      |               |                                      |
| specific request | *condition*     |               |                                      |
|                  |                 | Price <44>    | user tried to specify a limit price  |
|                  |                 | not valid     | for a market order                   |
|                  |                 | when          |                                      |
|                  |                 | OrdType <40>  |                                      |
|                  |                 | = 1           |                                      |
|                  |                 +---------------+--------------------------------------+
|                  |                 | StopPx <99>   | user tried to specify a stop price   |
|                  |                 | not valid     | for a market order                   |
|                  |                 | when          |                                      |
|                  |                 | OrdType <40>  |                                      |
|                  |                 | = 1           |                                      |
|                  |                 +---------------+--------------------------------------+
|                  |                 | StopPx <99>   | user tried to specify a stop price   |
|                  |                 | not valid     | for a limit order                    |
|                  |                 | when          |                                      |
|                  |                 | OrdType <40>  |                                      |
|                  |                 | = 2           |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| tag required for | *tag*           |               |                                      |
| specific request | required when   |               |                                      |
| is missing       | *condition*     |               |                                      |
|                  |                 | Price <44>    | limit price for limit order not      |
|                  |                 | required      | specified                            |
|                  |                 | when          |                                      |
|                  |                 | OrdType <40>  |                                      |
|                  |                 | = 2           |                                      |
|                  |                 +---------------+--------------------------------------+
|                  |                 | StopPx <99>   | stop price for stop order not        |
|                  |                 | required      | specified                            |
|                  |                 | when          |                                      |
|                  |                 | OrdType <40>  |                                      |
|                  |                 | = 3           |                                      |
|                  |                 +---------------+--------------------------------------+
|                  |                 | One of        |                                      |
|                  |                 | ExpireDate    |                                      |
|                  |                 | <432>,        |                                      |
|                  |                 | ExpireTime    |                                      |
|                  |                 | <126>         |                                      |
|                  |                 | required when |                                      |
|                  |                 | TimeInForce   |                                      |
|                  |                 | <59> = 6      |                                      |
|                  |                 |               | order lifetime not specified         |
+------------------+-----------------+---------------+--------------------------------------+
| tag value        | *tag* = *value* | TimeInForce   | user asked for IOC execution on a    |
| supplied not     | not supported   | <59> = 3 not  | market-if-touched order              |
| supported for    | when            | supported     |                                      |
| specific request | *condition*     | when OrdType  |                                      |
|                  |                 | <40> = J      |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| day order placed |                 | Order         |                                      |
| too close to day |                 | received      |                                      |
| expiry time      |                 | after 16:55   |                                      |
|                  |                 | ET; order     |                                      |
|                  |                 | will expire   |                                      |
|                  |                 | next day      |                                      |
|                  |                 | 17:00 ET      |                                      |
|                  |                 | (21:00 UTC)   |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| attempted to     | Account <1> =   | Account <1>   | account 15 does not exist or is not  |
| trade on a       | *value*         | = 15 access   | tradeable by the user                |
| nonexistent      | access          | denied        |                                      |
| account or on an | denied          |               |                                      |
| account without  |                 |               |                                      |
| trading          |                 |               |                                      |
| permission       |                 |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| tag value        | *tag* = *value* | Symbol <55>   |                                      |
| supplied is not  | not valid       | = gold/dollar |                                      |
| a valid value    |                 | not valid     |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| trade fails due  | Account <1> =   |               |                                      |
| to insufficient  | *value*         |               |                                      |
| funds            | insufficient    |               |                                      |
|                  | funds           |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| trade fails due  | Maximum number  |               | there is a 1000 open order limit and |
| to trade ticket  | of open orders  |               | a 1000 trade-ticket limit per        |
| limit or open    | or trades       |               | account                              |
| orders limit     | exceeded        |               |                                      |
| exceeded         |                 |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| trading halted   | Symbol <55> =   |               |                                      |
| on symbol        | *value* trading |               |                                      |
|                  | halted          |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| trade size       | OrderQty <38> = |               |                                      |
| exceeds maximum  | *value* exceeds |               |                                      |
| trade size or    | available       |               |                                      |
| quantity         | quantity for    |               |                                      |
| available for    | symbol          |               |                                      |
| execution        |                 |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| request on       | Multiple orders |               | multiple orders matched; use the     |
| existing order   | matched:        |               | OrderID to specify an exact order    |
| results in       | \[*OrderID*\]   |               |                                      |
| multiple         | (*OrdType*),    |               |                                      |
| candidate        | \[*OrderID*\]   |               |                                      |
| matched orders   | (*OrdType*),    |               |                                      |
|                  | ...             |               |                                      |
|                  |                 |               |                                      |
|                  +-----------------+---------------+--------------------------------------+
|                  | Multiple orders |               | multiple orders matched; use the     |
|                  | match           |               | OrderID to specify an exact order    |
|                  | (ClOrdID,       |               |                                      |
|                  | OrderID,        |               |                                      |
|                  | OrdType):       |               |                                      |
|                  | *ClOrdID*,      |               |                                      |
|                  | *OrderID*,      |               |                                      |
|                  | *OrdType*;      |               |                                      |
|                  | *ClOrdID*,      |               |                                      |
|                  | *OrderID*,      |               |                                      |
|                  | *OrdType*;      |               |                                      |
|                  | ...             |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| exact order      | *tag* value     | Symbol <55>   | OrderID found but supplied symbol    |
| OrderID provided | incorrect       | value         | does not match the order's symbol    |
| but some values  |                 | incorrect     |                                      |
| supplied do not  |                 +---------------+--------------------------------------+
| match order      |                 | Side <54>     | OrderID found but supplied side      |
| particulars      |                 | value         | does not match the order's side      |
|                  |                 | incorrect     |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| user tried to    | *tag* changes   | TimeInForce   |                                      |
| change fixed     | not permitted   | <59>          |                                      |
| order            |                 | changes not   |                                      |
| parameters       |                 | permitted     |                                      |
|                  |                 +---------------+                                      |
|                  |                 | Symbol <55>   |                                      |
|                  |                 | changes not   |                                      |
|                  |                 | permitted     |                                      |
|                  |                 +---------------+                                      |
|                  |                 | Side <54>     |                                      |
|                  |                 | changes not   |                                      |
|                  |                 | permitted     |                                      |
|                  |                 +---------------+                                      |
|                  |                 | OrdType <40>  |                                      |
|                  |                 | changes not   |                                      |
|                  |                 | permitted     |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| order duration   | *tag* = *value* | ExpireDate    |                                      |
| out of range     | out of range;   | <432> =       |                                      |
|                  | Order lifetime  | 21000115 out  |                                      |
|                  | minimum 5       | of range;     |                                      |
|                  | minutes,        | Order         |                                      |
|                  | maximum 30      | lifetime      |                                      |
|                  | calendar days   | minimum 5     |                                      |
|                  |                 | minutes,      |                                      |
|                  |                 | maximum 30    |                                      |
|                  |                 | calendar days |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| FOK/IOC order    | *tag* = *value* | Price <44> =  |                                      |
| was canceled due | not met (market | 1.01 not met  |                                      |
| to price         | *type* =        | (market offer |                                      |
| stipulation not  | *price*)        | = 1.00)       |                                      |
| met              |                 |               |                                      |
|                  |                 +---------------+                                      |
|                  |                 | StopPx <99> = |                                      |
|                  |                 | 1.01 not met  |                                      |
|                  |                 | (market bid   |                                      |
|                  |                 | = 1.00)       |                                      |
|                  |                 |               |                                      |
+------------------+-----------------+---------------+--------------------------------------+
| OANDA            | OANDA           | OANDA         | *list* is a comma-separated list of  |
| transaction IDs  | transaction     | transaction   | ticket number ranges                 |
| associated with  | ID(s): *list*   | ID(s):        |                                      |
| a FIX order      |                 | 21-23,26,30   | the sample text indicates that       |
|                  |                 |               | tickets 21, 22, 23, 26, and 30 are   |
|                  |                 |               | associated with this FIX order       |
|                  |                 |               |                                      |
|                  |                 |               | the string ``none`` is used if no    |
|                  |                 |               | OANDA tickets are associated with    |
|                  |                 |               | this FIX order                       |
+------------------+-----------------+---------------+--------------------------------------+

