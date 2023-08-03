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
are members of the room.  At any given time, there is exactly one _hub_ server
for the room, which is in primary control of the room.  All other servers are
known as _followers_.  Follower servers interact directly with the hub server.
Interactions between clients occur indirectly, via the servers for the clients'
providers.

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

As shown in {{stack}}, MIMI protocols define server-to-server interactions and
client-to-client interactions.  Each client interacts with the overall system by
means of its provider's server (whether hub or follower).

MIMI has several functional layers.  From the bottom of the stack to the top:

* **Transport** - Delivery of protocol information between servers.
* **Signaling** - Control of the structure of rooms, including user-level
  membership and policy configuration
* **Delivery Service** - Alignment between the end-to-end security state of the
  room and the application state (e.g., assuring that all members' devices have
  the keys used to provide end-to-end security)
* **End-to-End Security** - A cryptographic protocol that assures the end-to-end
  confidentiality and authenticity of application messages
* **Message Format** - The actual messages that realize the application-layer
  messaging interaction

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

# Room Structure

* Logically:
    * Membership
        * User-level
        * Client-level
        * Server-level <= user-level
    * Policy:
        * Admission policy
        * Capabilities per user
        * Capabilities per server
    * Messages

* Physically: All events
    * Persistent events
    * Ephemeral events

# APIs

* Message format
    * All the cool message features
* E2E Security ~= MLS
    * Protect/Unprotect messages
    * Add/Remove clients
    * Add/Remove servers as external joiners
    * Update agreed policy
* Delivery Service
    * Publish/Fetch KeyPackage
    * Enqueue Proposal
    * Deliver Commit
* Signaling
    * Add/Remove users
    * Update agreed policy
* Transport
    * Submit/Deliver events

A typical application-layer operation will execute across all of these layers.
For example, to add a user:

* Fetch KeyPackages for all the user's clients via the DS
* Submit an event...
* Signaling the addition of the user...
* And broadcasting a Commit adding the user's devices


# Security Considerations

* Authentication and authorization
* HBH security ~= TLS
* E2E security ~= MLS


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
