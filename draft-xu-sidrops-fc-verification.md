---
title: "BGP AS_PATH Verification Based on Forwarding Commitment (FC) Objects"
abbrev: "FC-based AS_PATH Verification"
category: std

docname: draft-xu-sidrops-fc-verification-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "SIDR Operations"
# keyword:
# - next generation
# - unicorn
# - sparkling distributed ledger
venue:
  group: "SIDR Operations"
  type: ""
  mail: "sidrops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sidrops/"
  github: "FCBGP/fc-verification"
  latest: "https://FCBGP.github.io/fc-verification/draft-xu-sidrops-fc-verification.html"

author:
  -
      fullname: Ke Xu
      org: Tsinghua University
      city: Beijing
      country: China
      email: xuke@tsinghua.edu.cn


normative:
  I-D.guo-sidrops-fc-profile:

informative:


--- abstract

The Forwarding Commitment (FC) is an RPKI object that attests to the complete routing intents description which an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation. This document specifies an FC-based AS Path Verification methodology to mitigate AS Path forgery and route leaks. This document also explains the various BGP security threats that FC can help address and provides operational considerations associated with FC deployment.




--- middle

# Introduction

The Forwarding Commitment (FC) {{I-D.guo-sidrops-fc-profile}} is a Resource Public Key Infrastructure (RPKI) object that attests to the complete routing intents description which an Autonomous System (AS) would obey in  Border Gateway Protocol (BGP) route propagation. This document specifies an FC-based AS Path Verification methodology to prevent AS-path forgery and route leaks.

Typically, an AS-path forgery occurs when an offending AS illegitimately 


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
