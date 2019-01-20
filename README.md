## obfs4 - The obfourscator
#### Yawning Angel (yawning at torproject dot org)

### What?

This is a look-like nothing obfuscation protocol that incorporates ideas and
concepts from Philipp Winter's ScrambleSuit protocol.  The obfs naming was
chosen primarily because it was shorter, in terms of protocol ancestery obfs4
is much closer to ScrambleSuit than obfs2/obfs3.

The notable differences between ScrambleSuit and obfs4:

 * The handshake always does a full key exchange (no such thing as a Session
   Ticket Handshake).
 * The handshake uses the Tor Project's ntor handshake with public keys
   obfuscated via the Elligator 2 mapping.
 * The link layer encryption uses NaCl secret boxes (Poly1305/XSalsa20).

As an added bonus, obfs4proxy also supports acting as an obfs2/3 client and
bridge to ease the transition to the new protocol.

### Why not extend ScrambleSuit?

It's my protocol and I'll obfuscate if I want to.

Since a lot of the changes are to the handshaking process, it didn't make sense
to extend ScrambleSuit as writing a server implementation that supported both
handshake variants without being obscenely slow is non-trivial.

### Dependencies

Build time library dependencies are handled by the Go module automatically.

If you are on Go versions earlier than 1.11, you might need to run `go get -d
./...` to download all the dependencies. Note however, that modules always use
the same dependency versions, while `go get -d` always downloads master.

 * Go 1.11.0 or later. Patches to support up to 2 prior major releases will
   be accepted if they are not overly intrusive and well written.
 * See `go.mod` for build time dependencies.

### Installation

To build:

	`go build -o obfs4proxy/obfs4proxy ./obfs4proxy`

To install, copy `./obfs4proxy/obfsproxy` to a permanent location
(Eg: `/usr/local/bin`)

Client side torrc configuration:
```
ClientTransportPlugin obfs4 exec /usr/local/bin/obfs4proxy
```

Bridge side torrc configuration:
```
# Act as a bridge relay.
BridgeRelay 1

# Enable the Extended ORPort
ExtORPort auto

# Use obfs4proxy to provide the obfs4 protocol.
ServerTransportPlugin obfs4 exec /usr/local/bin/obfs4proxy

# (Optional) Listen on the specified address/port for obfs4 connections as
# opposed to picking a port automatically.
#ServerTransportListenAddr obfs4 0.0.0.0:443
```

### Tips and tricks

 * On modern Linux systems it is possible to have obfs4proxy bind to reserved
   ports (<=1024) even when not running as root by granting the
   `CAP_NET_BIND_SERVICE` capability with setcap:

   `# setcap 'cap_net_bind_service=+ep' /usr/local/bin/obfs4proxy`

 * obfs4proxy can also act as an obfs2 and obfs3 client or server.  Adjust the
   `ClientTransportPlugin` and `ServerTransportPlugin` lines in the torrc as
   appropriate.

 * obfs4proxy can also act as a ScrambleSuit client.  Adjust the
   `ClientTransportPlugin` line in the torrc as appropriate.

 * The autogenerated obfs4 bridge parameters are placed in
   `DataDir/pt_state/obfs4_state.json`.  To ease deployment, the client side
   bridge line is written to `DataDir/pt_state/obfs4_bridgeline.txt`.

### Thanks

 * David Fifield for goptlib.
 * Adam Langley for his Elligator implementation.
 * Philipp Winter for the ScrambleSuit protocol which provided much of the
   design.
