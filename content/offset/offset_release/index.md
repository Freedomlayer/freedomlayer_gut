+++
title = "Announcing Offset 0.1.0 🎉🎉🎉"
description = "Announcing Offset 0.1.0"
date = 2019-04-25
+++

We are glad to announce the first public release of Offset!  
Offset is a decentralized payment system, allowing to pay and process payments
efficiently and safely. Offset is an open source project, written in Rust, licensed under
[AGPL3](https://opensource.org/licenses/AGPL-3.0).

You don't need a bank account, credit card, phone number or even an email
address to use Offset. On the other hand, Offset currently doesn't have an
interface to the traditional banking system, and therefore you can not use it
yet to buy food at the grocery store.

## Warning

**This release of Offset is very initial and not ready to use in
production!**. There are most certainly bugs in this release, and the interface and protocols
will almost surely break in the next releases, so please don't move all your
savings into Offset yet.

## Quick links

- [Releases page](https://github.com/freedomlayer/offset/releases)
- [Repository](https://github.com/freedomlayer/offset)
- [Documentation](https://offset.readthedocs.io/en/latest/?badge=latest)
- [Quick Offset tutorial](https://offset.readthedocs.io/en/latest/tutorial/)


## Main features

- Efficient and quick payments. Offset does not rely on a [blockchain](https://en.wikipedia.org/wiki/Blockchain) or any kind
    of [proof of work](https://en.wikipedia.org/wiki/Proof-of-work_system).
    Every transaction affects only a few computers in the network. Payments
    usually take less than a second.

- Low flat payment fees. Payment fees are determined by the length of the route
    on which the payment is sent. Each participant along the route gets exactly
    one credit.

- Fair distribution of assets: The total amount of credits in the Offset system
    is **0**, and it is initially distributed evenly between all the
    participants (Read more to learn how it works)

- If you lost your key, you can talk to some friends and restore your
    account.

- Censorship resistant. Buy whatever you want, sell whatever you want. Offset is
    decentralized, and therefore no entity can block or censor payments.

- Full raw control: Offset is fully programmable and has open interface for
    writing applications.


## Command line Video tutorial

<script id="asciicast-242796" src="https://asciinema.org/a/242796.js" async></script>


## The core idea

Offset is a digital payment system based on [mutual credit](https://en.wikipedia.org/wiki/Mutual_credit).

The common method of running an economy is to divide some amount of money bills
between people and let them play. Usually there are some people in charge of
printing more money bills or destroying money bills to keep the economy going.

Offset works in a different way. 

Quoting From the [documentation](https://offset.readthedocs.io/en/latest/?badge=latest):

Consider Alice, who has two friends, Bob and [Charli](https://en.wikipedia.org/wiki/Charli_XCX).
Suppose that at the same time:

- Bob owes Alice 50 dollars
- Alice also owes Charli 50 dollars. 

```text
    (-50,+50)       (-50, +50)
Bob --------- Alice ---------- Charli

```

Suppose that Charli meets Bob one day, and Bob has a bicycle he wants to sell
for exactly 50 dollars. Suppose also that Charli is exactly looking to buy a
bicycle, and he decides to buy it from bob. In this case Charli can take the
bicycle, and they both ask Alice to forget about the debts on both sides.

In a sense, Charli made a payment to Bob by asking Alice to **offset** the debts.
No real money bills had to pass hands to make this happen.

```text
      (0, 0)          (0, 0)
Bob --------- Alice ---------- Charli

```

If Bob and Charli perform these kinds of transactions very often, it is
reasonable for Alice to charge for forwarding the transactions. For example,
Alice could charge 1 dollar for every transaction she forwards.

Offset is a system that works this way. People or organizations can set up
mutual credit with each other, and leverage those mutual credits to send
payments to anyone in the network. A participant that mediates a transaction
earns 1 credit.

In Offset we call two entities that have set up together mutual credit account
**friends**.

## Setting credit limit

Suppose that Alice and Bob has a mutual credit account. Initially the account
is balanced. If Bob starts buying lots of things along chains that go through
Alice, eventually Bob will have a very large debt to Alice. Alice might not
want that.

Why not? Alice is afraid that Bob will default on this debt one
day. Maybe Alice can trust Bob with certain amount of debt, but not with too
much debt. 

The solution Offset offers to this trust problem is that both sides configure
credit limits.

For example:

- Alice configures a credit limit for Bob: 100 credits.
- Bob configures a credit limit for Alice: 80 credits.

This means Bob can never owe Alice more than 100 credits, and Alice can never
owe Bob more than 80 credits. (EDIT: Fixed thanks to
[nirizr](https://github.com/nirizr/))


From the point of view of Alice:

```text
                    balance
 ----[-----------|-----^-------]--->
    -80          0            100
```

(In the figure above: Alice has control over the right cap, and Bob has control
over the left cap)


Conversely, By putting a limit to the maximum
amount of credits Bob can owe to Alice, Alice has in fact put a limit on the
maximum amount of credits it can own.

In other words: In the closed system of Alice and Bob, The richest Alice can be
is 100 credits. The richest Bob can be is 80 credits. 

As a corollary, the amount of trust (credit limit) a participant puts on other
participants limits the amount of maximum credits the participant can own. This
amount is also the maximum amount of credits a participant can lose by trusting
other participants.


## The value of one credit

The basic amount of value in Offset is **one credit**. One credit is exactly the
amount of value you earn when you help mediate a transaction. 

What is the value of one credit with respect to known currencies? We are not sure
yet. These are questions that we should probably let the market decide. We
might be able to have some estimates though.

1 credit should not be too expensive, because usually one might need to pay a
few credits [^1] as a fee for sending a transaction, and nobody wants to
use a payment system where the fees are too high.

On the other hand, if the value of 1 credit is too small, there is no real
incentive for people to run nodes. From this side, we might conjecture that the
amount of transactions mediated by one Offset node for a day should be able to
cover the costs of running that node.


## Total amount of credits

All participants begin initially with 0 credits, and **the total amount of
credits in the system always remains 0**.
The reason for this is that every excess in credits in one participant is the
dual of lack of credits in another participant.


## How Offset works?

The core of Offset is the credit pushing mechanism. It allows to send credits
along a route of mutual credit edges in a secure way. You can read more about it
in the [project documentation](https://offset.readthedocs.io/en/latest/theory).

Offset does not use a [blockchain](https://en.wikipedia.org/wiki/Blockchain) and
does not contain any form of proof of work. In addition, Offset does not attempt
to achieve a global consensus [^2]. Offset is very efficient. Every transaction
sent using Offset only affects a few other computers in the network (according to the
length of the route).


## Network topology

Offset is made of a few main services:

- Offset node (stnode)
- Offset relay (strelay)
- Offset index (stindex)

Example for part of the Offset network topology:

```text
                  +-------+
                  | Index |
                  +-+---+-+
                    |   |           
              +-----+   +------+
              |                |
         +----+--+         +---+---+          +-------+
         | Index +---------+ Index +----------+ Index |
         +-+-----+         +-------+          +-----+-+
           |                                        |
           |                                        +---+
           |     +-------+         +-------+            |    +-------+
           |     | Relay |         | Relay |            |    | Relay |
           |     +--+----+         +-+---+-+            |    +---+---+               
           |        |                |   |              |        |
           |+-------+                |   |              |        |
           || +----------------------+   +-----------+  |+-------+
           || |                                      |  ||
         /-++-+-\                                  /-+--++\
         | Node |                                  | Node |
         \-+--+-/                                  \---+--/
           |  |                                        |
        +--+  +---+                                    |
        |         |                                    |
     +--+--+   +--+--+                              +--+--+
     | App |   | App |                              | App |
     +-----+   +-----+                              +-----+

```

In the figure above: Two nodes can communicate using relays and find routes
using index servers. Some applications are connected to the nodes.
 

**stnode** is the binary that runs the Offset node service. It is the core part of
Offset that executes the credit pushing mechanism. This binary is what most
users will run on their end device.

**strelay** is a binary that runs the Offset relay service. It is used to relay
communication between nodes. The communication between nodes is end-to-end
encrypted, and therefore strelay can not read the communication. Anyone can run
a relay. We use relays because most users are behind one or more
[NAT](https://en.wikipedia.org/wiki/Network_address_translation)s and
firewalls, and may not be able to listen for TCP connections [^3].

Relay servers are a form of a network meeting point between nodes. It allows
nodes to communicate. Every node must configure at least one relay server to
be able to communicate with other nodes.

**stindex** is a binary that runs the Offset index service. It is used to index
the relationships between nodes and supply routes that can be used to send
funds. 

Every node sends periodic information to a (preconfigured) index server about
his relationship with his friend nodes. The index servers share this
information with each other and use it to index the whole network.

Whenever a node wants to send funds to another node, it first queries an Offset
index for a route. Then the node uses the provided route to send the funds to
the remote node.

Anyone can run an index, but an index is not useful if it is not part of the
index servers federation, because it will not be able to get information about
the full state of the Offset network. Therefore Index server owners should
configure their index servers to talk to each other.


## Offset apps

The way to communicate with Offset node is through Offset apps.
Offset comes with a simple command line app called stctrl. Apps can be
configured to have certain permissions. Offset Apps communicate with the Offset
node using TCP communication (Capnp serialized), therefore apps can be written
in any language by anyone.

An example for `stctrl` invocation:

```bash
$ stctrl -I app0/app0.ident -T node0/node0.ticket info friends
+----+-------+---------------+
| st | name  | balance       |
+====+=======+===============+
| E+ | node1 | C: LR=+, RR=+ |
|    |       | B  =100       |
|    |       | LMD=150       |
|    |       | RMD=200       |
|    |       | LPD=0         |
|    |       | RPD=0         |
+----+-------+---------------+
```

The arguments given to stctrl are: `-I`: the identity of the app (Used for
authentication against the node) and `-T`: the node ticket. The node ticket
contains the node connection info. The subcommand `info friends` shows the list
of currently configured friends.

In the example above the node has one friend (named node1), with balance of 100
credits. The local max debt (credit limit imposed by the remote node) is 150
credits, and the remote max debt (credit limit imposed by the local node) is
200 credits.


## Public relays and index servers

We set up a pair of relay servers and a pair of index servers to get you
started. We include here their tickets:

- index0:
    - [client_ticket](index0_client.ticket) (For node configuration)
    - [server_ticket](index0_server.ticket) (For index server federation)
- index1:
    - [client_ticket](index1_client.ticket) (For node configuration)
    - [server_ticket](index1_server.ticket) (For index server federation)
- relay0: [ticket](relay0.ticket)
- relay1: [ticket](relay1.ticket)


It doesn't matter which ones you pick. Learn how to configure your offset node
[here](https://offset.readthedocs.io/en/latest/tutorial/).


## Cryptography

Offset depends on [ring](https://github.com/briansmith/ring) for its cryptographic code.
Offset relies on the following cryptographic primitives:

- Diffie Hellman
- Symmetric encryption
- Cryptographic signatures
- Hash functions

All the communication is encrypted.

We chose not to rely on the [traditional TLS certificates authority
hierarchy](https://en.wikipedia.org/wiki/Certificate_authority) because:

- It forces users to buy domain names.
- Using it could give power to a small
    amount of organizations to shut down Offset services.


## License

The Offset project is licensed under the GNU Affero General Public License
version 3 (AGPL3), but Offset applications can be written under any license and
can also be written for commercial usage.

We plan to convert all the interface related crates to the
MIT license, to allow Rust developers to reuse Offset interface to write Rust applications.

Why AGPL3? Because we believe payment related code should always be open
source. This is important both from security and transparency perspectives. If
you have a different opinion about this subject, please [send us a
message](./about/_index.md).


## Future work

Offset is far from done. The following is a very general outline of what we plan
to do next:

- Gathering feedback from preview release

- Offset core (stnode, stindex, strelay):
    - Refactoring codebase
    - Documentation
    - Stabilizing Offset protocol and interfaces
    - Migrating to use an sqlite3 database for node
    - Adding tests
    - Security review

- Offset Applications:
    - Creating a GUI desktop application. (Possibly a browser addon?)

- A mobile GUI application for Offset.

## Add me

Let's start experimenting with Offset. I opened a node.

If you want to add my node as a friend, this is my friend file: [real_node.friend](real_node.friend).
Remember that I have to add you as a friend too, so email me your friend file.
You can find my email at the [about page](./about/_index.md).




## Contributors

This is a good time to thank people who helped making Offset possible:

- [Kamyuen Tse](https://github.com/kamyuentse)
- [A4Vision](https://github.com/A4Vision)
- [juchiast](https://github.com/juchiast)
- [pzmarzly](https://github.com/pzmarzly)
- [Cy6erlion](https://github.com/Cy6erlion)
- [Nemo157](https://github.com/Nemo157)
- [vitalyd](https://users.rust-lang.org/u/vitalyd)
- [Sam](https://morph.is/v0.8/)
- [amosonn](https://github.com/amosonn)
- BronzeAge
- DreadLord

We welcome contributions from anyone. Don't hesitate to reach out!

<hr/>

[^1]: If we believe the idea of [Six degrees of
separation](https://en.wikipedia.org/wiki/Six_degrees_of_separation), then the
average transaction might require a fee of 6 credits.  

[^2]:Instead, only a weak form of consensus is obtained along the route where the
funds are pushed.

[^3]: In the future we can implement smarter logic that attempts to form a real
p2p connection between nodes. At this point we chose the trivial and simple
solution to develop.

