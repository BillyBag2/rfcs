# Passive Trace Route

- Start Date: 2024-02-21
- RFC PR: [Meshtastic/rfcs#0000](https://github.com/Meshtastic/rfcs/pull/0000)
- Affected Components: Firmware, all clients

## Summary

[One paragraph explanation of the feature or change and its impact across the
Meshtastic ecosystem.]

Make trace route data available to the client both during the route discovery
and when the trace route reply is returning, even if not the initiator or target.

## Motivation

[Why are we doing this? What problem does it solve or what use cases does it
support? Discuss the benefits for the different components of the Meshtastic
ecosystem, including end-users, developers, and core maintainers.]

The trace route data has is a wealth of information that is not currently made
available to the client. Requesting a trace route consumes a lot of mesh
bandwidth traffic and rarely results in the initiator receiving the requested
information. However the trace route request propagates widely through the mesh
carrying useful mesh topology information. This information is only available to
the originator and only if a successful round trip occurs.

Making this information available to the client  would allow the client
to store and maintain a record of the mesh topology without the need to actively
introduce more packets to the mesh or initiate their own trace route.

Note also that trace route will only trace the out going route. The mesh will
have asymmetry where the ability to transmit to a node does not mean you can
receive from it when it transmits. Hearing a route discovery packet give
you information about nodes you can receive, as well as transmit to. 

## Ecosystem Impact

[Explain how this change will affect the various components of the Meshtastic
ecosystem, including firmware, phone, desktop, and web applications. Address
any coordination required among these components.]

A client receiving passive route discovery and trace route reply packets could
choose to build a map of
the mesh providing an incite into how the mesh is functioning.
The client would know which nodes
are able to communicate with directly, and routes to or from more distant nodes.
This would provide the exact same features of the current trace route
implementation but many nodes would receive the information without adding extra
packets to the network. Also nodes would gain an insight into viable receive
routes as well as transmit ones.

## Protocol Buffer Changes

[Detail the changes proposed to Meshtastic's protocol buffer definitions,
if any. Explain how these changes will be managed and propagated across the
different applications and firmware.]

The packet protocol on the mesh does not need changing. How nodes behave to
those packets does not need to change.

How the radio will communicate with the client to to be discussed.

## Technical Details

[Provide a more in-depth technical explanation of the proposed changes,
focusing on the high-level architecture and how different components of the
ecosystem will interact with these changes. This section should explain your
proposed solution in enough detail that someone familiar with the Meshtastic
ecosystem can understand the design and implementation of the feature.]

### Current implementation

Route discovery packets are sent out from a node to an intended destination.
Nodes repeating the route discovery information add themselves to the route and
then pass on the route discovery packet. When the intended destination receives
the route discovery information the intended destination sends back a reply 
containing the route.

### (RFC) Client receives all route discovery payloads and all trace route replies

Clients could be given all route discovery and trace route replies that are
received. A client receiving such information can passively build up a routing
map of the mesh without adding any extra data. They can still participate in the
normal trace route as currently implemented.

A lot of the route discovery packets will 'go the wrong way'. However the route
discovery packets still contain valuable mesh topology information. Also
participants in the route discovery can passively receive trace discovery
packets that have been retransmitted by a new participant in the trace. This
provides knowledge about routes to and from new participants, even if, having
already participated, this packet would not normally be retransmitted.

### Compatibility Considerations

[Discuss how the proposed changes will affect backward compatibility across the
ecosystem. Include strategies for handling compatibility issues. Discuss
whether this change requires any version bumps.]

No change to haw packets are handled in the mesh.

How the received data is delivered to the client need to be discussed.

### Security Considerations

[Address any security implications of the proposed changes, especially those
that might arise from modifications in protocol buffers and cross-component
communication.]

Trace route is considered public information and is currently available though
MQTT, for example. Anyone can request a trace route in the current
implementation.

### Performance Considerations

[Evaluate how the change will impact the performance of various components in
the ecosystem, including the efficiency and responsiveness of Meshtastic
devices and applications.]

No extra packets on mesh. However more traffic between radio and client.

## Drawbacks

[Discuss the potential downsides or limitations of the proposal. Why might this
change be controversial or challenging to implement?]

There will need to be new forms of communication between radio and client.
For a client to construct a node map of the mesh it will have to store and
process a lot of information.

## Rationale and Alternatives

- No extra data on the mesh.
- More mesh topology information for clients.
- Trace route sends packets on the mesh that are currently not useful to others, even if they participate in the trace.
- Trace routes currently only provides value if the trace completes and the
  reply gets back to the originator.
- Trace route only provides a transmit route. Not a receive route.

## Prior Art

[Discuss any similar implementations in other projects or technologies and what
can be learned from them. Include both successful and unsuccessful examples.]

- This could be prototyped using MQTT where all packets are accessible.

## Unresolved Questions

- How the received trace discovery and trace route reply would be
  communicated to the client needs to be discussed.
