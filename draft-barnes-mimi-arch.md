---
title: "An Architecture for More Instant Messaging Interoperability (MIMI)"
abbrev: "MIMI Architecture"
category: info

docname: draft-barnes-mimi-arch-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - messaging
 - end-to-end security
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "bifurcation/mimi-arch"

author:
 -
    fullname: Richard L. Barnes
    organization: Cisco
    email: rlb@ipv.sx

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overall Scope

Today, there are many providers of messaging functionality.  A provider
typically provides the client software (e.g., a mobile app) and the servers that
facilitate communications among clients.  The core function of MIMI is enabling
users to have messaging interactions across message providers.

{{overview}} shows the critical entities in the overall MIMI system and their
interactions.  Each human user is represented in the system by one or more
_clients_, where each client is a specific software or hardware system.  Each
provider is represented by a _server_ (logically a single server, but possibly
realized by multiple physical devices).

Messaging interactions are organized around _rooms_.  All messaging interactions
take place in the context of a room.  Rooms have associated membership lists (in
terms of both users and clients) and policies about things like how the room may
be joined and what capabilities each member has.

The protocol interactions that drive a room unfold among the servers whose users
are members of the room.  There is exactly one _hub_ server for the room, which
is in primary control of the room.  All other servers are known as _followers_.
Follower servers interact directly with the hub server.  Interactions between
clients occur indirectly, via the servers for the clients' providers.

~~~~~ aasvg
  Users            Provider X                Room 123
                 .--------------------.    .----------------.
 .-----.        | +----------+         |  |                  |
| Alice +---------+ Client A +--+      |  |                  |
 '-----'        | +----------+  |  +------------+            |
                |               +--+  Server 1  | (Follower) |
 .---.          | +----------+  |  +----------+-+            |
| Bob +-----------+ Client B +--+      |  |   |              |
 '---'          | +----------+         |  |   |              |
                 '--------------------'   |   |              |
                                          |   |              |
                   Provider Y             |   |              |
                 .--------------------.   |   |              |
                | +----------+         |  |   |              |
             +----+ Client C +--+      |  |   |              |
 .-------.   |  | +----------+  |  +----------+-+            |
| Charlie +--+  |               +--+  Server 2  | (Hub)      |
 '-------'   |  | +----------+  |  +----------+-+            |
             +----+ Client D +--+      |  |   |              |
                | +----------+         |  |   |              |
                 '--------------------'   |   |              |
                                          |   |              |
                   Provider Z             |   |              |
                 .--------------------.   |   |              |
 .-----.        | +----------+         |  |   |              |
| Diana +---------+ Client E +--+      |  |   |              |
 '-----'        | +----------+  |  +----------+-+            |
                |               +--+  Server 3  | (Follower) |
 .------.       | +----------+  |  +------------+            |
| Evelyn +--------+ Client F +--+      |  |                  |
 '------'       | +----------+         |  |                  |
                 '--------------------'    '----------------'
~~~~~
{: #overview title="MIMI Entities and Interactions" }

# Protocol Stack

As shown in {{stack}}, MIMI protocols define server-to-server interactions and
client-to-client interactions.  Each client interacts with the overall system by
means of its provider's server (whether hub or follower).  Client-to-client
interactions are done by means of these servers.

MIMI has several functional layers.  From the bottom of the stack to the top:

* **Transport** - Delivery of protocol information between servers.
* **Signaling** - Control of the structure of rooms, including user-level
  membership and policy configuration
* **Delivery Service** - Alignment between the end-to-end security state of the
  room and the application state (e.g., assuring that all members' devices have
  the keys used to provide end-to-end security)
* **End-to-End Security** - A cryptographic protocol that protects the
  confidentiality and authenticity of clients' messages against potential
  malicious actions by servers
* **Message Format** - The actual messages that realize the application-layer
  messaging interaction

Note that some parts of the overall system are explicitly out of scope for MIMI.
Namely, client-server interactions internal to a provider (indicated by
"(Provider)" in {{stack}}) can be arranged however the provider likes, as long
as

~~~~~ aasvg
  +---------------------------------------------------------+
  |                      Message Format                     |
  +---------------------------------------------------------+
  |                    End-to-End Security                  |
  +--------------+-------------+-------------+--------------+
  |              |  Del. Svc.  |  Del. Svc.  |              |
  |              +-------------+-------------+              |
  |  (Provider)  |  Signaling  |  Signaling  |  (Provider)  |
  |              +-------------+-------------+              |
  |              |  Transport  |  Transport  |              |
  +--------------+-------------+-------------+--------------+
  |              |             |             |              |
Client        Follower        Hub         Follower        Client
|                    | |                | |                    |
 '-------. .--------'   '-----. .------'   '-------. .--------'
          |                    |                    |
       Provider             Provider             Provider
~~~~~
{: #stack title="The MIMI Protocol Stack" }

# Actors, Identifiers, and Authentication

There are three types of identified actor in the MIMI system:

* Servers,
* Users, and
* Clients.

A server's identity is effectively the identity of the provider it represents.
Servers are represented by domain names {{!RFC1035}}.  User identities are
provider-specific, and thus scoped to a given server identity.  Likewise, a
client's identity is within the scope of its user.

> TODO: Define syntax for these identifiers. Something like:
>
> * Server: `example.com`
> * User: `@user@example.com`
> * Client: `%device@user@example.com`

To facilitate the application of policies based on these identifiers to protocol
actions, each actor presents one or more credentials that associate a signature
key pair to their identifiers.  Protocol messages are then signed by their
senders to authenticate the origin of the message.

# Rooms

Logically, a room comprises a messaging interaction among a specific set of
clients, governed by a single policy.  The membership and the policy, together
with other information about the room, are collectively known as the _metadata_
associated to the room.  All of the actors involved in the room (clients and
servers) have the same view of the room's metadata.  Changes to room metadata
can be proposed by either clients or servers, but are subject to policy
constraints as discussed in {{policy}}.

Each room has associated to it an end-to-end security context.  Public aspects
of this context are known to the servers involved in the room, but private
aspects are known only to the clients participating in the room.  For example,
the end-to-end security context might expose authenticated signature public keys
for room participants, but keep secret the keys that are used to encrypt
messages.

## Events

Physically, a room is realized by servers exchanging _events_ that describe
changes to the room's state, for example, messages sent within the room or
changes to membership and policy.  Events come in two types:

* **State events**, which make changes to the room metadata
* **Message events**, which describe actual messaging activity in the room

Each event originates at one of the servers participating in the room (possibly
as a result of some interaction with a client).  The originating server sends
the event to the hub server for the room, who distributes it to the other follower
servers.

The hub for a room arranges all of the state events into a single linear
_history_, so that the room's metadata has a single, traceable history.  Since
state events can also make updates to the end-to-end security state of the room,
this sequencing also means that the end-to-end security protocol does not need
to support concurrent changes by multiple actors.

> OPEN ISSUE: The Matrix history model, where all servers can infer the current
> state from the state events, has unfortunate privacy properties.  For example,
> a server that is added to the room late in the room's history can see the
> membership of the room at all earlier times.
>
> In the "hub" model we have here, this is unnecessary.  The hub can maintain a
> snapshot view of the state and simply distribute that view to new servers.
> That way, state events become more ephemeral, basically just authenticated
> diffs on the snapshot.

Each state event is signed by the client or server that created it. The
authenticated identity of the originator is used by the hub and follower servers
to verify that the changes described in the event are consistent with the room's
policy.

## Membership

The membership of a room is the set of users who are allowed to participate in
the room in some way.  (The specific list of ways in which a user may
participate is defined by policy; see {{policy}}.)  Membership is also tracked
at the level of individual clients, to ensure that all of the clients used
by the room's members have access to the end-to-end security context for the
room.  The servers for a room's members have a role in setting the room's
policy, so there is also a server-level notion of membership that derives from
the user-level membership.

The user-level membership of the group is managed by the signaling layer of the
protocol stack.  The delivery service layer then assures that the set of clients
with access to the room's end-to-end security context matches the user-level
membership.

The membership of a group can change over time, via _add_ and _remove_
operations.  At the signaling layer, operations are done at the level of user
identities.  The delivery service layer then assures that all of the affected
user's clients are added or removed from the end-to-end security context for the
room.  If a user starts using a new client or discards an old one, further
delivery-service-layer operations may be necessary to add or remove clients,
without affecting the user-level membership.

## Policy

Each room has an associated _policy_ that governs which protocol actions are
authorized for the room while the policy is in effect.  The policy defines
several aspects of the room's behavior, for example:

* Admission policy: Do new members need to be explicitly added by a current
  member of the room, or can some set of users join unilaterally?
* Capabilities per user: Is a given user allowed to ...
    * Send messages in the room?
    * Add or remove other users?
    * Grant or deny capabilities to other users?
* Capabilities per server: Is a given server participating in the room allowed
  to...
    * Add or remove users?
    * Grant or deny capabilities to users?

The hub server for a room defines the _policy envelope_ for the room (the set of
of acceptable policies for the room.  The hub also sets the initial policy for
the room when it is created.  Pursuant to that initial policy, the clients and
servers participating in the room may then make further changes to the policy. 

At any given time, all of the clients and servers have the same view of the
room's policy.  A client or server that receives an event that is not compliant
with the room's policy may thus safely discard it, since all of the other
participating clients/servers should also reject the event.

## Messages

Mesage events are end-to-end secure objects that carry application messages in
the standard MIMI content format.  The end-to-end encapsuation ensures that the
message content is only accessible to the clients participating in the room, not
the servers that help to distribute it.

The MIMI message format defines how clients achieve the various features of a
messaging application, for example:

* Text messaging
* File attachements
* Replies
* Reactions
* Initiation of real-time sessions

# Protocol Layers

As shown in {{stack}}, the overall MIMI system comprises a few layers of
protocol operations.  Each of these layers manages different aspects of the
room's operations.

The transport layer defines how events move between servers.  Servers are
generally assumed to be highly available and reachable over the Internet.

The signaling layer defines the structure of a room's events, for example, the
syntax for events that add or remove users or update a room's metadata.   

The delivery service layer defines objects and ancillary non-event interactions
that support the alignent of the group's end-to-end security context to its
signaling-layer state.  For example:

* How servers publish cryptographic information about their users' clients, so
  that those clients can be added to rooms
* How clients and servers encode requests for changes to the end-to-end security
  context for a room
* How accepted changes to the end-to-end security context for a room are
  delivered to participating clients and servers

The end-to-end security layer is a cryptographic protocol (defined outside of
MIMI) such as MLS {{?RFC9420}} that defines the cryptographic state machine that
ensures the end-to-end security properties of the system.  The end-to-end
security layer provides several important functions:

* Protect and unprotect application messages
* Add and remove clients from the end-to-end security context
* Confirm the agreement of clients on the room's policy, including the
  collection of servers authorized to propose changes to the end-to-end security
  context
* Inform servers of the public state of the end-to-end security context,
  including signature public keys for the clients and servers involved in the
  room

Finally the message format layer allows clients to encode messages that realize
messaging features, as discussed in {{messages}}.

A typical application-layer operation will execute across all of these layers.
The layers are largely nested within one another, in the sense that
transport-layer protocol messages will carry signaling messages, which will
embed delivery-service messages with end-to-end security messages inside of
them. Some layers, though, might define operations that are independent of the
others.  

For example, for a server to to add a user to a room:

* The server performs a delivery-service-layer interaction with the user's
  server, to fetch initialization information for the user's clients
* The server submits an event to the hub server including:
  * Signaling information to add the user to the room
  * Delivery service information encoding a proposal to add the user's clients
  * End-to-end security information with the cryptographic details for adding
    the user's clients

Likewise, when a user sends a message via a client, the client encodes the
message in the MIMI message format, encapuslates it using the end-to-end
security layer, and emits it as an event.

# Security Considerations

TODO

* Authentication and authorization
* HBH security ~= TLS
* E2E security ~= MLS


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
