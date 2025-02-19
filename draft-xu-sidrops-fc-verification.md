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
  RFC4271:
  RFC6480:
  RFC6811:
  RFC8205: # BGPsec
  RFC9582:
  RFC9234:
  # I-D.guo-sidrops-fc-profile:
  I-D.ietf-sidrops-aspa-profile:         # ASPA profile
  # I-D.ietf-sidrops-aspa-verification:    # ASPA verification
  I-D.ietf-sidrops-rpki-prefixlist:      # SPL profile
  I-D.ietf-sidrops-spl-verification:     # SPL verification
  I-D.geng-sidrops-asra-profile:         # ASRA profile
  # I-D.sriram-sidrops-asra-verification:    # ASRA verification

informative:
  RFC7908:
  RFC9319:
  RFC9324:

--- abstract

The Forwarding Commitment (FC) is an RPKI object that attests to the complete routing intents description which an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation. This document specifies an FC-based AS Path Verification methodology to mitigate, even solve, AS Path forgery and route leaks. This document also explains the various BGP security threats that FC can help address and provides operational considerations associated with FC deployment.

--- middle

# Introduction {#Introduction}

The Border Gateway Protocol (BGP) is vulnerable to route hijacks and route leaks {{RFC7908}}. Some existing BGP extensions can partially solve or alleviate these problems. Resource Public Key Infrastructure (RPKI) based route origin validation (RPKI-ROV) {{RFC6480}} {{RFC6811}} {{RFC9319}} {{RFC9582}} and Signed Prefix List-based Route Origin Verification (SPL-ROV) {{I-D.ietf-sidrops-rpki-prefixlist}} can be used to detect and filter accidental mis-originations. BGPsec is designed to provide security for the AS-path attribute in the BGP UPDATE message {{RFC8205}}. {{RFC9234}} and Autonomous System Provider Authorization (ASPA) {{I-D.ietf-sidrops-aspa-profile}} aim at detecting and mitigating accidental route leaks.

However, there are still some issues that need to be addressed. ASPA is a genius mechanism to verify BGP AS-path attribute content, which only stores customer-to-provider information in RPKI. Autonomous System Relationship Authorization (ASRA) has listed several security problems with ASPA in {{Section 2 of I-D.geng-sidrops-asra-profile}}. Though the validity of the ASPA/ASRA objects is verified, the relationship between two BGP neighbors cannot be attested. When two ASes announce mutually exclusive relationships, for example, AS A says AS B is its Provider and AS B says AS A is its Provider, no other ASes can verify their real relationships.

The Forwarding Commitment (FC) [I-D.guo-sidrops-fc-profile] is a Resource Public Key Infrastructure (RPKI) object that attests to the complete routing intents description an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation.


This document specifies an FC-based AS Path Verification methodology to prevent AS path forgery in the BGP AS-path attribute of advertised routes. FC-based AS_PATH verification also detects and mitigates route leaks.


# Requirements Language

{::boilerplate bcp14-tagged}


# Definition of Commonly Used Terms {#DefinitionOfCommonlyUsedTerms}

The definitions and semantics of Forwarding Commitment (FC) provided in [I-D.guo-sidrops-fc-profile] are applied here.

- **Route is ineligible**: The term has the same meaning as in {{RFC4271}}, i.e., "route is ineligible to be installed in Loc-RIB and will be excluded from the next phase of route selection."
- **AS-path**: This term defines a sequence of ASes listed in the BGP UPDATE AS_PATH or AS4_PATH attribute. In this document, the terms AS-path, AS_PATH, and AS4_PATH are interchangeably used.
- **TotallyValid**: This is one of the verification results of using FC objects to verify AS_PATH. This means the FCs of each AS in AS_PATH are all with the 'originASes' field.
- **VFP**: Validated FC Payload (see {{ForwardingCommitment}}).

# Forwarding Commitment (FC) {#ForwardingCommitment}

Forwarding Commitment (FC) objects encapsulate the routing intent description of an Autonomous System (AS). Unlike most RPKI-signed objects, FC objects possess a distinct design regarding verification results. Since FC objects reference Route Origin Authorizations (ROAs) within their content, the verification outcomes for FC are categorized into four distinct states: TotallyValid, Valid, Invalid, and Unknown.

It is RECOMMENDED that all routing intents be explicitly enumerated within a single FC object. However, due to the inherent complexity of routing intents, providing a comprehensive list can be challenging. Consequently, it is RECOMMENDED to include routing intents with the originASes field designated as 'NONE' when the issuer is unable to specify which routes will be propagated from previousASes to nexthopASes. It may have a few of these routingIntents with the originASes field set as 'NONE'.

In general, there exists a singular valid FC object corresponding to a specific asID. However, in instances where multiple valid FC objects containing the same asID are present, the union of the resulting routingIntent members constitutes the comprehensive set of members. This complete set, which may arise from either a single or multiple FCs, is locally maintained by a Relying Party (RP) or a compliant router. Such an object is referred to as the Validated FC Payload (VFP) for the asID.

Except for the empty originASes, there would also be empty previousASes and nexthopASes in a routing intent. It is NOT RECOMMENDED to describe routing intent without nexthopASes as this does not help verify BGP AS_PATH.

It is REQUIRED at least one routing intent description in an FC object. Otherwise, the empty FC object means no routes can be transited or transformed from this asID.

# BGP AS_PATH Verification Algorithm Using FC

FCs describe the local routing intents of an AS. It can be used to verify the AS-path attribute in the BGP UPDATE message.

Before the AS_PATH verification procedure, it can first perform prefix origin verification with ROA-ROV defined in {{Section 2 of RFC6811}} or SPL-ROV defined in {{Section 4 of I-D.ietf-sidrops-spl-verification}}.

An eBGP router that conforms to this specification MUST implement FC-based AS_PATH verification procedures specified below.

For each received BGP route:

1. Query all the FCs that are issued by the ASes that are in the AS-path attribute;
2. If all ASes on the AS-path have their FCs with the BGP AS-path conforming to all ASes routing intents and the route is also specified in the originASes field, the verification result is TotallyValid;
3. Else, if the originASes field is missing but all ASes on the AS-path have their FCs with the BGP AS-path conforming to all ASes routing intents, the verification result is Valid;
4. Else, if none of the AS on the AS path has its FC, the verification result is Unknown;
5. Else, if some of the ASes on the AS path have their FCs but others ASes do not have their FCs, the verification result is Invalid.

## Mitigation Policy {#MitigationPolicy}

The specific configuration of a mitigation policy based on AS_PATH verification using FC is at the discretion of the network operator. However, the following mitigation policy is highly recommended.

**Invalid**: If the AS_PATH is determined to be Invalid, then the route SHOULD be considered ineligible for route selection and MUST be kept in the Adj-RIB-In for potential future re-evaluation (see {{RFC9324}}).

**TotallyValid, Valid, or Unknown**: When a route is evaluated as Unknown (using FC-based AS_PATH verification), it SHOULD be treated at the same preference level as a route evaluated as Valid. But TotallyValid has the highest priority in BGP route selection while Valid has a second priority.



# Operational Considerations {#OperationalConsiderations}

Multiple valid Forwarding Commitment objects which contain the same asID could exist. In such a case, the union of these objects forms the complete routing intent set of this AS. For a given asID, it is RECOMMENDED that a CA maintains a single Forwarding Commitment. If an AS holder publishes a Forwarding Commitment, then relying parties SHOULD assume that this object is complete for that issuer AS.

If one AS receives a BGP UPDATE message with the issuer AS in the AS-path attribute which cannot match any routing intents of this issuer AS, it implies that there is an AS-path forgery in this message.

# Security Considerations {#SecurityConsiderations}

TODO Security


# IANA Considerations {#IANAConsiderations}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
