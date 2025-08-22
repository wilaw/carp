---
title: "CARP - a CMAF compliant implementation of WARP"
category: info

docname: draft-law-moq-carp-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Media Over QUIC"
keyword:
 - moq
 - moqt
 - WARP
 - CMAF
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "wilaw/carp"
  latest: "https://wilaw.github.io/carp/draft-law-moq-carp.html"

author:
  -
    ins: W. Law
    name: "Will Law"
    organization: Akamai
    email: wilaw@akamai.com

normative:
  MoQTransport: I-D.draft-ietf-moq-transport-10
  WARP:  I-D.draft-ietf-moq-warp
  CMAF:
    author:
      - name: International Organization for Standardization
        org: ISO
    title: "Information technology — Multimedia application format (MPEG-A) — Part 19:
            Common media application format (CMAF) for segmented media"
    date: 2021-10
  BASE64: RFC4648


informative:

...

--- abstract

This document updates [WARP] by defining a new optional feature for the streaming format.
It specifies the syntax and semantics for adding CMAF-packaged media [CMAF] to WARP.


--- middle

# Introduction

CARP Streaming Format (CARP) is a media format designed to deliver CMAF [CMAF] and
LOC [LOC] compliant media content over MOQ Transport (MOQT) [MoQTransport]. CARP extends
WARP and retains all the scope, capabilities and features of WARP including the catalog
format, timeline, ABR switching and LOC support. CARP is targeted at real-time and
interactive levels of live latency, as well as VOD content.

This document describes version 1 of the CARP streaming format.

# WARP Extension
All of the specifications, requirements, and terminology defined in [WARP] apply to
implementations of this extension unless explicitly noted otherwise in this document.

# CMAF Packaging

## Initialization headers
A CMAF header is a sequence of CMAF constrained ISO BMFF boxes that do not reference any
media samples, but are associated with a CMAF track and are necessary for initializing
the decoding of the subsequent CMAF fragments.

The header for a given MOQT Track MUST be packaged by encoding the header using [BASE64]
and then inserting that payload as the value of the Initialization data "initData" field
in the catalog entry for that Track.

## Switching sets and tracks
This specification defines a direct mapping between CMAF Tracks ( [CMAF] Sect 3.2.1) and
MOQT tracks ([MoQTransport] Sect 2.3).

CMAF switching sets are a set of one or more CMAF tracks (3.2.1), where each track is an
alternative encoding of the same source content and are constrained to enable seamless
track switching (3.3.9).

Each CMAF track in a switching set MUST be transmitted as a separate MOQT Track. The
catalog entry for each of these tracks in the switching set MUST carry a Alternate group
(altGroup) key with a common value.

The MOQT Group numbers within these switching set tracks MUST be media time-aligned.
Mandating the track being media time-aligned requires that the presentation time of the
first media sample contained within the first MOQT Object of each MOQT Group is identical.

## Object Packaging

Each MOQT Group MUST begin with a stream access point (SAP type 1 or 2).
Each MOQT Object MUST hold one or more CMAF Chunks. The payload of each Object is subject
to the following requirements:

* MUST consist of a Segment Type Box (styp) followed by any number of media fragments.
  Each media fragment consists of a Movie Fragment Box (moof) followed by a Media Data
  Box (mdat). The Media Fragment Box (moof) MUST contain a Movie Fragment Header Box
  (mfhd) and Track Box (trak) with a Track ID (track_ID) matching a Track Box in the
  initialization fragment.
* MUST contain a single [ISOBMFF] track.
* MUST contain media content encoded in decode order.


## Catalog description
This specification extends the allowed packaging values defined in WARP Section 5.2.10
to include a new entry, as defined in Table 1 below:

| Name            |   Value   |      Reference        |
|:================|:==========|:======================|
| CMAF            | cmaf      | This RFC              |

Every Track entry in a CARP catalog MUST declare a "packaging" type value of "cmaf".

## Timeline description
This specification extends the METADATA element of the timeline track, defined in WARP
Section 7, in the following four aspects: 

* Specification of a general scheme for metadata signalled through the METADATA element
  of the timeline track [Ed.Note: This aspect should also be applied to WARP, thus
  should be moved to WARP later on.]
* Definition of a namespace for CARP-specific metadata signalled through the METADATA
  element of the timeline track
* Addition of metadata signalling of SAP type
* Addition of metadata signalling of earliest presentation time


### General metadata scheme
When not empty, the string in the METADATA field MUST contain one or more comma
separated key=value pairs, formatted as Strings as specified in Section 3.3.3 of
[RFC9651]. For example, the METADATA field can be "key1=1,key2=3268". For another
example, the METADATA field can be "key1=1,key2=""hello-world"",key3=3268".

A key name MAY be prefixed with a namespace. When a namespace is present, the
separator between the namespace prefix and the key name is '.'.

### Namespace for CARP-specific metadata signalled through the METADATA element
For CARP-specific metadata signalled through the METADATA element, the namespace is
"timeline:metadata:carp".

### Metadata signalling of SAP type
When the key name of a key=value pair is "SAP_TYPE", the value indicates the SAP type
the Object begins with. The namesapce-prefixed key is "timeline:metadata:carp.SAP_TYPE".

The value 0 indicates that the Object does not start with an ISOBMFF stream access point.
The value equal to 1, 2, or 3 indicates that the Object begins with a stream access point
of SAP type 1, 2, or 3, respectively. When the Object is the first Object in the Group,
the value MUST be equal to 1 or 2.

### Metadata signalling of earliest presentation time
When the key name of a key=value pair is "EARLIEST_PTS", the value indicates the earliest
media presentation timestamp rounded to the nearest millisecond of all media samples in
the Object. The namesapce-prefixed key is "timeline:metadata:carp.SAP_TYPE".

WWhen the SAP type the Object begins with is 2 or 3, the EARLIEST_PTS key SHOULD be
present.


# Catalog Examples

The following section provides non-normative JSON examples of various catalogs
compliant with this draft.


## Simulcast video tracks - 3 alternate video qualities along with audio

This example shows catalog for a media producer capable of sending 3
time-aligned video tracks for high definition, low definition and medium
definition video qualities, along with an audio track.

~~~json
{
  "version": 1,
  "generatedAt": 1746104606044,
  "tracks":[
    {
      "name": "hd",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "initData": "AAAAIGZ0eXBpc281AAA...AAAAAAAAAAAAA",
      "role": "video",
      "codec":"avc1.640028",
      "width":1920,
      "height":1080,
      "bitrate":5000000,
      "framerate":30,
      "altGroup":1
    },
    {
      "name": "md",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "initData": "AAAAHGZ0eXBpc281AAA...AAAAAAAAAAAAAA",
      "role": "video",
      "codec":"avc1.64001e",
      "width":720,
      "height":640,
      "bitrate":3000000,
      "framerate":30,
      "altGroup":1
    },
    {
      "name": "sd",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "initData": "AAAAHGZ0eXBpc281AAA...AAAAAAAAAAAAAA",
      "role": "video",
      "codec":"avc1.64000d",
      "width":192,
      "height":144,
      "bitrate":500000,
      "framerate":30,
      "altGroup":1
    },
    {
      "name": "audio",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "initData": "AAAAHGZ0eXBpc281AAA...AAAAAAAAAAAAAA",
      "role": "audio",
      "codec":"mp4a.40.5",
      "samplerate":48000,
      "channelConfig":"2",
      "bitrate":67071
    }
   ]
}
~~~


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
