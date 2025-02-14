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
  -
      fullname: Shenglin Jiang
      org: ZGC Laboratory
      city: Beijing
      country: China
      email: jiangshl@zgclab.edu.cn
  -
      fullname: Yangfei Guo
      org: ZGC Laboratory
      city: Beijing
      country: China
      email: guoyangfei@zgclab.edu.cn
  -
      fullname: Xiaoliang Wang
      org: Tsinghua University
      city: Beijing
      country: China
      email: wangxiaoliang0623@foxmail.com
  
    

normative:
  RFC6480:
  RFC6811:
  RFC8205: # BGPsec
  RFC9582:
  RFC9234:
  # I-D.guo-sidrops-fc-profile:
  I-D.ietf-sidrops-aspa-profile         # ASPA profile
  # I-D.ietf-sidrops-aspa-verification    # ASPA verification
  I-D.ietf-sidrops-rpki-prefixlist      # SPL profile
  # I-D.ietf-sidrops-spl-verification     # SPL verification
  I-D.geng-sidrops-asra-profile         # ASRA profile
  # I-D.geng-sidrops-asra-verification    # ASRA verification

informative:
  RFC7908:
  RFC9319:

--- abstract

The Forwarding Commitment (FC) is an RPKI object that attests to the complete routing intents description which an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation. This document specifies an FC-based AS Path Verification methodology to mitigate, even solve, AS Path forgery and route leaks. This document also explains the various BGP security threats that FC can help address and provides operational considerations associated with FC deployment.

--- middle

# Introduction

The Border Gateway Protocol (BGP) is vulnerable to route hijacks and route leaks {{RFC7908}}. Some existing BGP extensions can partially solve or alleviate these problems. Resource Public Key Infrastructure (RPKI) based route origin validation (RPKI-ROV) {{RFC6480}} {{RFC6811}} {{RFC9319}} {{RFC9582}} and Signed Prefix List-based Route Origin Verification (SPL-ROV) {{I-D.ietf-sidrops-rpki-prefixlist}} can be used to detect and filter accidental mis-originations. BGPsec is designed to provide security for the AS_PATH attribute in the BGP UPDATE message {{RFC8205}}. {{RFC9234}} and Autonomous System Provider Authorization (ASPA) {{I-D.ietf-sidrops-aspa-profile}} aim at detecting and mitigating accidental route leaks.

However, there are still some issues that need to be addressed. ASPA is a genius mechanism to verify BGP AS_PATH attribute content, which only stores customer-to-provider information in RPKI. Autonomous System Relationship Authorization (ASRA) has listed several security problems with ASPA in {{Section 2 of I-D.ietf-sidrops-asra-profile}}. Though the validity of the ASPA/ASRA objects is verified, the relationship between two BGP neighbors cannot be attested. When two ASes announce mutually exclusive relationships, for example, AS A says AS B is its Provider and AS B says AS A is its Provider, no other ASes can verify their real relationships.

The Forwarding Commitment (FC) [I-D.guo-sidrops-fc-profile] is a Resource Public Key Infrastructure (RPKI) object that attests to the complete routing intents description an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation.


This document specifies an FC-based AS Path Verification methodology to prevent AS path forgery in the BGP AS_PATH attribute of advertised routes. FC-based AS_PATH verification also provides detection and mitigation of route leaks.


# Requirements Language

{::boilerplate bcp14-tagged}


# Definition of Commonly Used Terms

The definitions and semantics of Forwarding Commitment (FC) provided in [I-D.guo-sidrops-fc-profile] are applied here.

- **AS path**: This term defines a sequence of ASes that are listed in the BGP UPDATE AS_PATH attribute. The terms AS path and AS_PATH are interchangeably used in this document.
- **TotallyValid**: This is one of the verification results of using FC objects to verify AS_PATH. This means the FCs of each AS in AS_PATH are all with the 'roaASes' field.

# FC 

Not like most RPKI-signed objects, FC has a different design. FC references ROA in its content so that FC has two Valid states. The verification results of FC have four outcomes: TotallyValid, Valid, Invalid, and Unknown.

Let the sequence {AS(N), AS(N-1), ..., AS(2), AS(1)} represent the AS_PATH in terms of unique AS Numbers (ASNs), where AS(1) is the origin AS and AS(N) is the most recently added AS and neighbor of the receiving/verifying AS. N is the AS path length in unique ASes. Let AS(N+1) represent the receiving AS that is verifying the entire received AS path. For a given AS hop, say AS(i) to AS(i+1), AS(i) is the sender, and AS(i+1) is the receiver. For each such AS hop in the AS path, 

# Operational Considerations


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
