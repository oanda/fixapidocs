.. _stunnel: http://www.stunnel.org/

================================
OANDA FIX API General Guidelines
================================

.. contents::

Introduction
============

The OANDA FIX API documents describe the behavior of the current version
of the OANDA FIX Server, and indicate how to interact and place trades
through this service.

The Financial Information eXchange ("FIX") protocol is a series of
messaging specifications for the electronic communication of financial
data, including trade-related messages. It is a globally accepted
standard of messaging specifications developed through the
collaboration of banks, brokers, exchanges, institutional investors,
and information technology providers from around the world.

OANDA fxTrade supports versions 4.2, 4.3, and 4.4 of the FIX
protocol. You are urged to download and consult the official FIX 
specifications and the recommended best practices document
at http://www.fixprotocol.org/.

Connection Requirements
=======================

To use the OANDA FIX API:

* you must have an API license agreement with OANDA

* you need to use your own FIX engine to connect to the OANDA server

  * OANDA does not officially recommend or provide any particular
    FIX engine; any compliant engine should work


* all communications are done over SSL on the internet

  ::

     +-------------------+      SSL       +------------------+
     | client FIX engine |  ------------> | OANDA FIX Server |
     +-------------------+                +------------------+

  * strong encryption meeting the ``HIGH+SHA+AES`` cipher list is 
    required


* if your FIX engine is not natively SSL-capable, alternate SSL 
  tunneling software is required

  * stunnel_ has been used successfully by many customers but the
    responsibility for configuring and running it lies with the customer

    * a sample stunnel configuration file for the fxTrade *Practice* 
      system is shown below:

      ::

         ## fxTrade Practice (testing) ##
         client=yes
         verify=0

         [FXFIX]
         accept=30008
         connect=fxgame-fix.oanda.com:32009
         TIMEOUTclose=0
         ciphers=HIGH+SHA+AES

    * connections to the fxTrade production system use the hostname
      ``fxtrade-fix.oanda.com``
         
OANDA FIX API Connections
-------------------------

Customers are required to use two separate connections to the FIX API,
a RATES connection for market data, and an ORDER connection for all
order execution.

All connections use the customers login username for the system as the
``SenderCompID`` value.  All connections use ``OANDA`` as the 
``TargetCompID`` value.

A RATES connection is established by including ``TargetSubID=RATES``
in the header.  An ORDER connection is established by omitting any
``TargetSubID`` in the header.  (Unrecognized ``TargetSubID`` values
are currently treated as for the ORDER connection but this behavior is
not guaranteed.)

The OANDA system runs a continuous 24-hour trading session during the
trading week.  Trading is not available over the weekend (Friday 5pm
North America Eastern Time to market open Monday).  No daily or
weekly sequence number reset is explicitly required, although weekend
maintenance periods may take the servers offline and any reconnection
requires a sequence number reset.

System Requirements
-------------------

Accurate clock synchronization is important for both parties to ensure
communication is prompt and any latency can be detected.  The local
clock will need to be accurately synchronized via a properly-configured
NTP service using non-pool time servers or another high-accuracy 
mechanism.

    Note some Windows systems do not run a time service with enough
    accuracy to ensure clock synchronization.  See the 
    `troubleshooting guide <./oanda-fix-api-troubleshooting.rst>`_ 
    for further information.

Mapping of FIX Server Orders to OANDA Transaction-view Tickets
==============================================================

The OANDA FIX Server implements the FIX order model, which is the model
typically found in the equities and futures markets.  This order model
differs from the order model of the underlying OANDA trading system
exposed in the GUI, website transaction history, and OANDA OpenAPI.

Understanding the differences between the FIX order model and the OANDA
backend transaction ticket order model is critical to understanding how
one order representation is described in the other.

In FIX terminology, an *order* is any request to trade a symbol, whether
immediate or good for some time duration.  In the OANDA model, the term
*order* is used for entry orders, limit orders, or stop orders with some 
order lifetime; these orders result in a ``BuyEntry``, ``BuyLimit``, or 
``BuyStop`` (and the analogous ``Sell`` orders) transaction ticket; the
term *trade* is any position that results from an immediate buy or sell 
on the account (a ``BuyMarket`` or ``SellMarket`` transaction ticket)
or from the triggering of an OANDA *order*.

The OANDA transaction ticket stop-loss, take-profit, and trailing-stop
annotations are not available in the FIX order model.

Customers who place orders in the OANDA FIX API and view them via the
website transaction history will need to know which OANDA transaction 
tickets correspond to a FIX order.  Each Execution Report <8>
describing a order result will list OANDA transaction tickets in the
Text <58> field.

    The Text <58> field will contain a clause of the form

        ``OANDA transaction ID(s): list``

    where ``list`` is a comma-separated list of ticket number ranges.
    For example, tickets 21, 22, 23, 26, and 30 would be displayed as

        ``OANDA transaction ID(s): 21-23,26,30``

The way that FIX orders are mapped to transaction tickets is described
below:

Transaction Ticket Groups
-------------------------

Multiple OANDA transaction tickets can correspond to a FIX order.

Fills may result in multiple tickets if there is an existing open 
position in the opposite direction of the current trade; individual
transactions ticket would record the closing of the opposite-direction
position, with a potential extra ticket recording the excess quantity
in the current direction

    Example: with an existing position

    * buy 100 EUR/USD

    * buy 150 EUR/USD

    * buy 200 EUR/USD

    a sell 1000 EUR/USD will result in multiple transaction tickets
    recording the fill:

    * sell 100 EUR/USD - to close the long 100 position above

    * sell 150 EUR/USD - to close the long 150 position above

    * sell 200 EUR/USD - to close the long 200 position above

    * sell 550 EUR/USD - to record the short 550 position


Immediately-Executed Orders: Market Orders, Limit/Stop Orders Requested FOK/IOC
-------------------------------------------------------------------------------

A FIX market order entered with no TimeInForce <59> is submitted as an
OANDA ``BuyMarket`` or ``SellMarket`` request.  Filled orders are 
recorded; rejected orders do not result in any transaction ticket record.

A FIX order entered with TimeInForce <59> as
``3`` (immediate-or-cancel) or ``4`` (fill-or-kill) which results in any
fill will record a ``BuyMarket`` or ``SellMarket`` transaction ticket; 
fully-canceled (no fill) orders record a ``CancelledBuyMarket``
or ``CancelledSellMarket`` transaction ticket.  The price stipulation
for limit and stop orders is recorded in one of the the
``high_order_limit`` / ``low_order_limit`` fields; market orders do not
have any price stipulation.

+-------------------+--------------------------------------------------+
| OANDA transaction | information recorded                             |
| ticket record     |                                                  |
+===================+==================================================+
| price             | market price at time of execution                |
+-------------------+--------------------------------------------------+
| units             | number of units actually filled                  |
|                   |                                                  |
|                   | sum across the group represents the CumQty <14>  |
+-------------------+--------------------------------------------------+
| high_order_limit  | records the price stipulation (the Price <44>    |
+-------------------+ for the limit order, the StopPx <99> for the     |
| low_order_limit   | stop order); only one order_limit is valid       |
|                   | for any order                                    |
+-------------------+--------------------------------------------------+
| completion_code   | records type of ticket: FOK, IOC, or standard    |
+-------------------+--------------------------------------------------+
| transaction_link  | records the ticket number of any existing        |
|                   | position countered by this ticket                |
+-------------------+--------------------------------------------------+
| order_link        | not applicable                                   |
+-------------------+--------------------------------------------------+
| order_qty         | the first transaction ticket of a group will     |
|                   | record the OrderQty <38>                         |
+-------------------+--------------------------------------------------+
| min_qty           | the first transaction ticket of a group will     |
|                   | record the MinQty <110> if provided, otherwise 0 |
+-------------------+--------------------------------------------------+

Price-Conditional Long-Duration Orders
--------------------------------------

Limit, Stop, and Market-if-Touched orders entered with TimeInForce of
DAY, GTD, or nothing (defaulting to DAY) result in a number of 
transaction tickets representing the outstanding order, and a number
of tickets representing the fill if one occurs.

Of the tickets representing the outstanding order:

* a new limit order creates a ``BuyLimit`` or ``SellLimit`` transaction
  ticket

* a new stop order creates a ``BuyStop`` or ``SellStop`` transaction
  ticket

* a new market-if-touched order creates a ``BuyEntry`` or ``SellEntry``
  transaction ticket

  * note this order is called a "limit order" on the GUI interface

* any Order Cancel / Replace Request <G> results in a ``ChangeOrder``
  transaction ticket recording the new order parameters

* any Order Cancel Request <F> results in a ``CloseOrder`` transaction
  ticket recording the cancelation of the order

* any order trigger results in a ``CloseOrder`` transaction ticket
  recording the triggering of the order

* any order expiry results in a ``CloseOrder`` transaction ticket
  recording the expiry of the order

+-------------------+--------------------------------------------------+
| OANDA transaction | information recorded                             |
| ticket record     |                                                  |
+===================+==================================================+
| order creation ticket                                                |
+-------------------+--------------------------------------------------+
| units             | records the initial requested OrderQty <38>      |
+-------------------+--------------------------------------------------+
| time              | time of order entry                              |
+-------------------+--------------------------------------------------+
| price             | records the price stipulation (Price <44> or     |
|                   | StopPx <99>)                                     |
+-------------------+--------------------------------------------------+
| duration          | records the expiry time of the order             |
+-------------------+--------------------------------------------------+
| ``ChangeOrder`` ticket                                               |
+-------------------+--------------------------------------------------+
| units             | records updated OrderQty <38>                    |
+-------------------+--------------------------------------------------+
| time              | time of order modification                       |
+-------------------+--------------------------------------------------+
| price             | records updated price stipulation                |
+-------------------+--------------------------------------------------+
| duration          | records updated expiry time                      |
+-------------------+--------------------------------------------------+
| transaction_link  | records the order this modification ticket       |
|                   | pertains to                                      |
+-------------------+--------------------------------------------------+
| ``CloseOrder`` ticket                                                |
+-------------------+--------------------------------------------------+
| completion_code   | records type of close: cancel, expiry, or fill   |
+-------------------+--------------------------------------------------+
| transaction_link  | records the order this closing ticket            |
|                   | pertains to                                      |
+-------------------+--------------------------------------------------+

If a fill occurred, a group of transaction tickets will record the units
resulting from the fill.  Similar to immediate-execution orders 
described above, any existing opposite-position tickets will have a
ticket to close that position, and the net position is recorded in the
last transaction ticket.

+-------------------+--------------------------------------------------+
| OANDA transaction | information recorded                             |
| ticket record     |                                                  |
+===================+==================================================+
| price             | price of fill; the value represents the          |
|                   | LastPx <31>                                      |
+-------------------+--------------------------------------------------+
| units             | number of units actually filled                  |
|                   |                                                  |
|                   | sum across the group represents the CumQty <14>  |
+-------------------+--------------------------------------------------+
| high_order_limit  | not applicable                                   |
+-------------------+                                                  |
| low_order_limit   |                                                  |
+-------------------+--------------------------------------------------+
| completion_code   |                                                  |
+-------------------+--------------------------------------------------+
| transaction_link  | records the ticket number of any existing        |
|                   | position countered by this ticket                |
+-------------------+--------------------------------------------------+
| order_link        | records the ticket number of the order entry     |
|                   | that this fill amount is a result of             |
+-------------------+--------------------------------------------------+

The diaspora record cannot be relied on to group position tickets 
resulting from one action together.  Please consult the execution 
reported list of ticket numbers instead.
