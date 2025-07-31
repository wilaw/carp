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
  CMAF: International Organization for Standardization, "Information technology — 
  Multimedia application format (MPEG-A) — Part 19: Common media application format
  (CMAF) for segmented media", ISO/IEC 23000-19:2021, Fourth edition, October 2021.

informative:

...

--- abstract

This document updates [WARP] by defining a new optional feature for the streaming format.
It specifies the syntax and semantics for adding CMAF-packaged media [CMAF] to WARP.


--- middle

# Introduction
0123456789012345678901234567890123456789012345678901234567890123456789012345678901234567
CARP Streaming Format (CARP) is a media format designed to deliver CMAF [CMAF] and
LOC [LOC] compliant media content over MOQ Transport (MOQT) [MoQTransport]. CARP extends
WARP and retains all the scope, capabilities and features of WARP including the catalog
format, timeline, ABR switching and LOC support. CARP is targeted at real-time and
interactive levels of live latency, as well as VOD content.

This document describes version 1 of the CARP streaming format.

# WARP Extension
All of the specifications, requirements, and terminology defined in [WARP] apply to
implementations of this extension unless explicitly noted otherwise in this document.

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
