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

The More Instant Messaging Interoperability (MIMI) working group is defining a
suite of protocols that allow messaging providers to interoperate with one
another.  This document lays out an overall architecture enumerating the MIMI
protocols and how they work together to enable an overall messaging experience.

--- middle

# Introduction

Today, there are many providers of messaging functionality.  A provider
typically provides the client software (e.g., a mobile app) and the servers that
facilitate communications among clients.  The core function of MIMI is enabling
users to have messaging interactions across message providers.

This overall goal breaks down into several sub-goals:

* Message formats that enable the user-level features of a messaging system
* Tracking of state across multiple providers
* End-to-end security of user messages
* Transport of protocol messages among providers

In this document, we describe the high-level functions of these protocols, and
how they work toegether to enable an overall messaging application.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overall Scope

{{overview}} shows the critical entities in the overall MIMI system and their
interactions.  Each human _user_ is represented in the system by one or more
_clients_, where each client is a specific software or hardware system belonging
to a single user.  Each provider is represented by a _server_ (logically a
single server, but possibly realized by multiple physical devices).

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

# Room State

A room represnts a messaging interaction among a specific set of clients, with a
single _state_.  At any given time, all of the clients and servers participating
in the room have the same view of the room's state.  Changes to the room's state
can be proposed by either clients or servers, though as dicussed in {{policy}},
one important aspect of the room's state is an authorization policy that
determines which actors are allowed to make which changes.

The creation of a room is a local operation on the hub server, and thus outside
the scope of MIMI.  The hub server establishes the initial state of the room.

The state of the room includes a few types of information, most importantly:

* The end-to-end security state of the room
* The user-level membership of the room
* The authorization policy for the room

> NOTE: We use the phrase "membership" for both users and clients.  It might be
> clearer to invent some distinct terminology for these notions.

> NOTE: In the below, we discuss user-level membership as something independent
> from, and managed separately from, the end-to-end security state of the room.
> Another possibility is that the user-level membership could be derived from
> the client-level membership, in the sense that a user is a member if and only
> if they have one or more clients joined.

## End-to-End Security State

Messages sent within a room are protected by an end-to-end security protocol to
ensure that the servers handling messages cannot inspect or tamper with
messages.  This means that the required cryptographic keys need to be
provisioned to any client from which a user can interact with the room.  The
state of this end-to-end security protocol thus represents the precise set of
clients that can send and receive messages in the room, the most precise notion
of membership for a room.  A client that has the required keys for end-to-end
security is said to be _joined_ to the end-to-end security state of the room.

The end-to-end security state of a room has public and private aspects.  Servers
may store the public aspects of the end-to-end security state, such as
identities and credentials presented by the clients in the room.  The private
aspects of the group, such as the symmetric encryption keys, are known only to
the clients.

## Membership

The membership of a room is the set of users who are allowed to participate in
the room in some way.  The specific list of ways in which a user may
participate is defined by authorization policy, as discussed in {{policy}}.

This is a more coarse-grained notion of membership than the client-level notion.
Ideally, the two are aligned, with each member user having at least one client
joined to the end-to-end security state, and each joined client owned by a
member user.  A user may be a member of a group without any client belonging to
the user being part of the end-to-end security state of the room.  (In such a
case, a user will not be able to read or send messages, but may be able to take
other actions.  It is up to client implementations how this state is
represented) The reverse situation is forbidden; a client belonging to a user
may be joined to the end-to-end security state of the room only if its user is a
member.

## Membership Changes

The membership of a group can change over time, via _add_ and _remove_
operations at both the user level and the client level.  These operations are
independent at the protocol level: For example, a user may be added to a room
before any of its clients are available to join, or a user may begin using a new
device (adding the device without changing the user-level membership).

As discussed above, user-level and client-level membership must be kept in sync.
When a user is added, some set of their clients should be added as well; when a
user leaves or is evicted, any clients joined to the room should be removed.
The cryptographic constraints of end-to-end security protocols mean that servers
cannot perform this synchronization; it is up to clients to keep these two types
of state in sync.

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

The hub server for a room defines the _policy envelope_ for the room, the set of
of acceptable policies for the room.  The hub also sets the initial policy for
the room when it is created.  Pursuant to that initial policy, the clients and
servers participating in the room may then make further changes to the policy.

At any given time, all of the clients and servers have the same view of the
room's policy.  A client or server that receives an event that is not compliant
with the room's policy may thus safely discard it, since all of the other
participating clients/servers should also reject the event.

# Protocols

As shown in {{fig-protocols}}, MIMI protocols define server-to-server interactions and
client-to-client interactions.  Each client interacts with the overall system by
means of its provider's server (whether hub or follower).  Client-to-client
interactions are done by means of these servers.

The messages sent within a room are forwarded among participating clients by
servers.  However, messages are protected by an end-to-end security protocol so
that their content is only accessible to the clients participating in the room.
In addition to forwarding messages, servers participate in control protocols
that coordinate the state of the room across the participating providers.  Both
message forwarding and control protocols leverage a common framework for sharing
_events_ among servers.

Note that some parts of the overall system are explicitly out of scope for MIMI.
Namely, client-server interactions internal to a provider (indicated by
"(Provider)" in {{fig-protocols}}) can be arranged however the provider likes.

A MIMI server thus participates in a few classes of protocols:

* A transport protocol
* Control protocols
* A message forwarding protocol

~~~~~ aasvg
       Provider             Provider             Provider
          |                    |                    |
 .-------' '--------.   .-----' '------.   .-------' '--------.
|                    | |                | |                    |
Client        Follower        Hub         Follower        Client
  |              |             |             |              |
  |              |             |             |              |
  |              |         Messaging         |              |
  |<=======================================================>|
  |              |             |             |              |
  |              |             |             |              |
  |  (Provider)  |          Control          |  (Provider)  |
  |<------------>|<------------------------->|<------------>|
  |              |             |             |              |
  |              |             |             |              |
  |              |  Transport  |  Transport  |              |
  |              |<----------->|<----------->|              |
  |              |             |             |              |
~~~~~
{: #fig-protocols title="MIMI Protocols" }

## Events and Transport

A room's activities are realized by servers exchanging _events_.  Events come in
two types:

* **State events**, which make changes to the room state
* **Message events**, which describe actual messaging activity in the room

Each event originates at one of the servers participating in the room (possibly
as a result of some interaction with a client).  The originating server sends
the event to the hub server for the room, who distributes it to the other follower
servers.

Each event is authenticated by its originating server so that all other
participating servers can verify its origin, even those to whom the event has
been distributed by the hub.  If an event was ultimately created by a client, it
is also authenticated by the client that created it.

The MIMI transport protocol defines this event framework, including its
authentication scheme, as well as the mechanics of how events are delivered from
one server to another.

## Control Protocols

The servers involved in a room use control protocols to perform actions related
to different types of information that comprise a room's state, particularly
those listed in {{room-state}}.  Because these types of information and the
operations they require are largely orthogonal, it makes sense to have a
separate control protocol for each type of information.

The **policy control protocol** distributes information about the policy
envelope of a room, and allows participants in a room to propose changes to the
policy within that envelope.

The **membership control protocol** manages the user-level membership of the
room, including the various ways that members might join or leave a room (or be
added/removed by other members).

The **end-to-end security control protocol** manages the end-to-end security
state of the room.  In addition to distributing messages that add or remove
clients from the end-to-end security state, this protocol also allows servers
to distribute cryptographic information that clients have pre-registered, which
allows clients to be asynchronously added to rooms.

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

Messages transit MIMI servers by means of a **message forwarding protocol**,
which carries an opaque, encrypted message payload together with enough metadata
to facilitate delivery to the clients participating in a room.

# Actors, Identifiers, and Authentication

> NOTE: This section is obsolete.  It should be rewritten to use concepts and
> terminology from draft-mahy-mimi-identity.

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

# Security Considerations

TODO

* Authorization policy attached to a room
* E2E security for messages provided by message delivery protocol
* E2E/E2M/M2E/M2M security for events provided by transport protocol
* HbH security provided by TLS


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
