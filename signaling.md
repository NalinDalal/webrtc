# Signaling

So while connecting agent has no idea about other peer.

Signaling messages(text)--->values exchanged ---->communicate

WebRTC use SDP for Signaling. simple to read and understand.

## Session Description Protocol(SDP)

defined in RFC 8866
key-value protocol.
SDP contains 0/more media Description.
Media Description usually maps to a single stream of media.

call with 3 video, 2 audio tracks=5 media Description

let's take a look at sdp format

```

a=my-sdp-value
a=second-value

```

only few SDP keys are only used, below are keys useful:
• v - Version, should be equal to `0`.
• o - Origin, contains a unique ID useful for renegotiations.
• s - Session Name, should be equal to `-`.
• t - Timing, should be equal to `0 0`.
• m - Media Description (`m=<media> <port> <proto> <fmt> ...`), described in detail below.
• a - Attribute, a free text field. This is the most common line in WebRTC.
• c - Connection Data, should be equal to `IN IP4 0.0.0.0`.

session can have unlimited media descriptors.

SDP (Session Description Protocol) is a menu/instruction sheet for a multimedia session.

Session Description = the whole call

A Session Description describes the entire communication session. can have multiple media describes.

```

Session Description
│
├── Media Description → Audio
├── Media Description → Video
└── Media Description → Screen sharing

```

> Session Description = the whole meeting
> Media Description = one part of the meeting (audio/video)
> Payload Type = a number identifying an RTP format
> rtpmap = tells you which codec that number represents
> Attributes = additional instructions about that media

example:

```

v=0
m=audio 4000 RTP/AVP 111
a=rtpmap:111 OPUS/48000/2
m=video 4000 RTP/AVP 96
a=rtpmap:96 VP8/90000
a=my-sdp-value

```

## SDP and WebRTC

WebRTC uses SDP to exchange information between two WebRTC Agents.

The main idea is:

```

Peer A                         Peer B
│                              │
│ -------- Offer ------------> │
│                              │
│ <-------- Answer ----------- │
│                              │

```

One peer creates an **Offer**.

The other peer checks the Offer and creates an **Answer**.

This is called the **Offer/Answer model**.

The Offer contains information such as:

- What media we want to send/receive
- Which codecs/formats we support
- ICE information
- DTLS information
- Media directions

The Answer tells the other peer:

- What it accepts
- What codecs it supports
- Whether it wants to send/receive media

This allows both peers to agree on how they will communicate.

For example:

```

Peer A Offer:

Audio:
OPUS
G722

Video:
VP8
H264

```

    ↓

```

Peer B checks what it supports

```

    ↓

```

Peer B Answer:

Audio:
OPUS

Video:
VP8

```

So unsupported codecs can be rejected through the Offer/Answer process.

## Transceivers

Transceiver is a WebRTC-specific concept.

It exposes a **Media Description** to the JavaScript API.

Think:

```

Media Description  <---->  Transceiver
SDP                  JavaScript API

```

Each Media Description becomes a Transceiver.

When we create a new Transceiver, a new Media Description is added to the local SDP.

Example:

```

create audio transceiver
↓
new audio Media Description

create video transceiver
↓
new video Media Description

```

So:

```

Transceiver
↓
Media Description
↓
SDP

```

## Transceiver Direction

Every Media Description in WebRTC has a `direction` attribute.

It tells the other peer whether we want to send and/or receive media.

There are 4 valid values:

• `sendrecv` - send and receive

• `sendonly` - only send

• `recvonly` - only receive

• `inactive` - neither send nor receive

Example:

```

a=sendrecv

```

means:

> I can send media to you and I can receive media from you.

```

a=sendonly

```

means:

> I will send media to you, but I don't want to receive media from you.

```

a=recvonly

```

means:

> I want to receive media from you, but I won't send media to you.

```

a=inactive

```

means:

> This media section is currently not sending or receiving anything.

## Common SDP Attributes used by WebRTC

### group:BUNDLE

BUNDLE means multiple types of traffic can use the same connection.

Without bundling:

```

Audio → Connection 1
Video → Connection 2

```

With BUNDLE:

```

Audio ─┐
Video ─┼──→ One Connection
Other ─┘

```

Bundling is generally preferred because it reduces the number of connections required.

Example:

```

a=group:BUNDLE 0 1

```

`0` and `1` refer to the Media Descriptions using their `mid` values.

### fingerprint

Example:

```

a=fingerprint:sha-256 0F:74:31:25:CB:A2:...

```

This is a hash/fingerprint of the certificate used by DTLS.

It helps verify that the certificate received during the DTLS connection matches the certificate we expected.

In simple terms:

> "I expect this peer to use this certificate."

### setup

Controls the DTLS role.

Possible values:

```

a=setup:active

```

Run as DTLS Client.

```

a=setup:passive

```

Run as DTLS Server.

```

a=setup:actpass

```

Let the other WebRTC Agent choose.

### mid

`mid` means **Media ID**.

It identifies a particular Media Description inside the session.

Example:

```

a=mid:0

```

and

```

a=mid:1

```

could identify:

```

mid:0 → Audio
mid:1 → Video

```

It is also used by BUNDLE to identify which media sections are being bundled.

### ice-ufrag

ICE username fragment.

Used by ICE to authenticate ICE traffic.

Example:

```

a=ice-ufrag:CsxzEWmoKpJyscFj

```

Think of it as part of the credentials used by ICE.

### ice-pwd

ICE password.

Example:

```

a=ice-pwd:mktpbhgREmjEwUFSIJyPINPUhgDqJlSd

```

Used together with `ice-ufrag` to authenticate ICE traffic.

So:

```

ice-ufrag + ice-pwd
↓
ICE authentication

```

### rtpmap

Maps an RTP Payload Type number to a codec.

Example:

```

a=rtpmap:111 opus/48000/2

```

means:

```

Payload Type 111
↓
OPUS
↓
48000 Hz
↓
2 channels

```

Another example:

```

a=rtpmap:96 VP8/90000

```

means:

```

Payload Type 96
↓
VP8
↓
90000 Hz clock rate

```

Payload Types are not necessarily fixed for every call.

The Offerer can assign Payload Type numbers to codecs for that particular session.

### fmtp

`fmtp` defines additional parameters for a specific Payload Type.

Example:

```

a=fmtp:111 minptime=10;useinbandfec=1

```

Here `111` refers to the OPUS Payload Type.

`fmtp` can communicate additional codec settings, such as video profiles or encoder parameters.

Think:

```

rtpmap → Which codec is this?

fmtp   → What extra settings does this codec use?

```

### candidate

An ICE Candidate represents one possible network address through which the peer may be reachable.

Example:

```

a=candidate:foundation 1 udp 2130706431 192.168.1.1 53165 typ host

```

A peer can have multiple candidates.

For example:

```

Candidate 1 → Local/private IP
Candidate 2 → Public IP
Candidate 3 → Relay address

```

ICE will use these candidates to try to establish a connection.

### ssrc

SSRC = Synchronization Source.

It identifies a particular RTP media stream.

Example:

```

a=ssrc:350842737 ...

```

The number:

```

350842737

```

is the SSRC identifier for that RTP stream.

Think:

```

SSRC
↓
identifies one RTP media source/stream

```

### msid

`msid` identifies the MediaStream and MediaStreamTrack associated with a media source.

Example:

```

a=msid:yvKPspsHcYcwGFTw DfQnKjQQuwceLFdV

```

Think:

```

MediaStream
+
MediaStreamTrack
↓
msid

```

### mslabel

`mslabel` identifies the MediaStream/container.

A MediaStream can contain multiple tracks.

Example:

```

MediaStream
│
├── Audio Track
└── Video Track

```

`mslabel` identifies the MediaStream.

### label

`label` identifies an individual media stream/track.

So:

```

mslabel → identifies the MediaStream
label   → identifies an individual track

```

## Complete WebRTC SDP Example

A WebRTC client might generate:

```

v=0
o=- 3546004397921447048 1596742744 IN IP4 0.0.0.0
s=-
t=0 0

a=fingerprint:sha-256 0F:74:31:25:CB:A2:13:EC:28:6F:6D:2C:61:FF:5D:C2:BC:B9:DB:3D:98:14:8D:1
a=group:BUNDLE 0 1

m=audio 9 UDP/TLS/RTP/SAVPF 111
c=IN IP4 0.0.0.0

a=setup:active
a=mid:0

a=ice-ufrag:CsxzEWmoKpJyscFj
a=ice-pwd:mktpbhgREmjEwUFSIJyPINPUhgDqJlSd

a=rtcp-mux
a=rtcp-rsize

a=rtpmap:111 opus/48000/2
a=fmtp:111 minptime=10;useinbandfec=1

a=ssrc:350842737 cname:yvKPspsHcYcwGFTw
a=sendrecv

a=candidate:foundation 1 udp 2130706431 192.168.1.1 53165 typ host
a=end-of-candidates

m=video 9 UDP/TLS/RTP/SAVPF 96
c=IN IP4 0.0.0.0

a=setup:active
a=mid:1

a=ice-ufrag:CsxzEWmoKpJyscFj
a=ice-pwd:mktpbhgREmjEwUFSIJyPINPUhgDqJlSd

a=rtcp-mux
a=rtcp-rsize

a=rtpmap:96 VP8/90000

a=ssrc:2180035812 cname:XHbOTNRFnLtesHwJ
a=sendrecv

```

## How to Read This SDP

First:

```

a=group:BUNDLE 0 1

```

means Media IDs `0` and `1` are bundled onto the same connection.

Then:

```

m=audio ...

```

means we have an **audio Media Description**.

Inside it:

```

a=mid:0

```

identifies the audio section.

```

a=rtpmap:111 opus/48000/2

```

means Payload Type `111` is OPUS.

```

a=sendrecv

```

means we can both send and receive audio.

Then:

```

m=video ...

```

means we have a **video Media Description**.

Inside it:

```

a=mid:1

```

identifies the video section.

```

a=rtpmap:96 VP8/90000

```

means Payload Type `96` is VP8.

```

a=sendrecv

```

means we can both send and receive video.

The ICE information:

```

a=ice-ufrag:...
a=ice-pwd:...
a=candidate:...

```

gives the information needed to attempt to establish the network connection.

The DTLS fingerprint:

```

a=fingerprint:sha-256 ...

```

helps verify the peer's DTLS certificate.

## What We Know From This SDP

From the example above:

• We have **2 Media Descriptions**.

• One Media Description is for **audio**.

• One Media Description is for **video**.

• Both use `sendrecv`, so both sides can send and receive those media types.

• Audio uses the **OPUS** codec.

• Video uses the **VP8** codec.

• RTP Payload Type `111` maps to OPUS.

• RTP Payload Type `96` maps to VP8.

• `mid:0` identifies the audio Media Description.

• `mid:1` identifies the video Media Description.

• ICE credentials and ICE Candidates are present, so the peers have information needed to attempt connectivity.

• The DTLS fingerprint is present, allowing the DTLS connection to be authenticated.

• `group:BUNDLE 0 1` indicates that the audio and video Media Descriptions can share one connection.

## Big Picture

The whole WebRTC process can be thought of as:

```

             WebRTC
                │
                ↓
           Signaling
                │
                ↓
               SDP
                │
      ┌─────────┴─────────┐
      ↓                   ↓
    Offer                Answer
      │                   │
      └─────────┬─────────┘
                ↓
         Both peers agree
                │
    ┌───────────┼───────────┐
    ↓           ↓           ↓

    ICE      DTLS        RTP

```

connectivity security media

```

SDP itself does **not** transport the audio/video.

SDP mainly tells the peers:

> "Here is what I support, here is what I want to send/receive, here are my codecs, here is my ICE information, and here is how you should establish the secure media connection."

Then the other WebRTC Agent responds with an **Answer**, and the two peers use the negotiated information to establish the actual connection and exchange media.

```
