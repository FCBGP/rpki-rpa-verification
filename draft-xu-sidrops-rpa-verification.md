---
title: "BGP AS_PATH Verification Based on Route Path Authorizations (RPA) Objects"
abbrev: "RPA-based AS_PATH Verification"
category: std

docname: draft-xu-sidrops-rpa-verification-latest
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
  github: "FCBGP/rpki-rpa-verification"
  latest: "https://FCBGP.github.io/rpki-rpa-verification/draft-xu-sidrops-rpa-verification.html"

author:
  -
      fullname: Ke Xu
      org: Tsinghua University
      city: Beijing
      country: China
      email: xuke@tsinghua.edu.cn
  -
      fullname: Shenglin Jiang
      org: Zhongguancun Laboratory
      city: Beijing
      country: China
      email: jiangshl@zgclab.edu.cn
  -
      fullname: Yangfei Guo
      org: Zhongguancun Laboratory
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
  RPKI-RPA-Profile: I-D.guo-sidrops-fc-profile # I-D.guo-sidrops-rpa-profile, # TODO: modify here before submit.

informative:
  RFC7908:
  RFC9319:
  RFC9324:
  RPKI-ASPA-Profile: I-D.ietf-sidrops-aspa-profile         # ASPA profile
  # I-D.ietf-sidrops-aspa-verification:    # ASPA verification
  RPKI-SPL-Profile: I-D.ietf-sidrops-rpki-prefixlist      # SPL profile
  RPKI-SPL-Verification: I-D.ietf-sidrops-spl-verification     # SPL verification
  # I-D.geng-sidrops-asra-profile:         # ASRA profile
  # I-D.sriram-sidrops-asra-verification:    # ASRA verification

--- abstract

The Route Path Authorizations (RPA) is an RPKI object that attests to the complete routing paths description which an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation. This document specifies an RPA-based AS Path Verification methodology to mitigate, even solve, AS Path forgery and route leaks. This document also explains the various BGP security threats that RPA can help address and provides operational considerations associated with RPA deployment.

--- middle

# Introduction {#Introduction}

The Border Gateway Protocol (BGP) is vulnerable to route hijacks and route leaks {{RFC7908}}. Some existing BGP extensions can partially solve or alleviate these problems. Resource Public Key Infrastructure (RPKI) based route origin validation (RPKI-ROV) {{RFC6480}} {{RFC6811}} {{RFC9319}} {{RFC9582}} and Signed Prefix List-based Route Origin Verification (SPL-ROV) {{RPKI-SPL-Profile}} can be used to detect and filter accidental mis-originations. BGPsec is designed to provide security for the AS-path attribute in the BGP UPDATE message {{RFC8205}}. {{RFC9234}} and Autonomous System Provider Authorization (ASPA) {{RPKI-ASPA-Profile}} aim at detecting and mitigating accidental route leaks.

However, there are still some issues that need to be addressed. ASPA is a genius mechanism to verify BGP AS-path attribute content, which only stores customer-to-provider information in RPKI. Though the validity of the ASPA objects is verified, the relationship between two BGP neighbors cannot be attested. When two ASes announce mutually exclusive relationships, for example, AS A says AS B is its Provider and AS B says AS A is its Provider, no other ASes can verify their real relationships.

The Route Path Authorizations (RPA) {{RPKI-RPA-Profile}} is a Resource Public Key Infrastructure (RPKI) object that attests to the complete routing paths description an Autonomous System (AS) would obey in Border Gateway Protocol (BGP) route propagation.


This document specifies an RPA-based AS Path Verification methodology to prevent AS path forgery in the BGP AS-path attribute of advertised routes. RPA-based AS_PATH verification also detects and mitigates route leaks.


# Requirements Language

{::boilerplate bcp14-tagged}


# Definition of Commonly Used Terms {#DefinitionOfCommonlyUsedTerms}

The definitions and semantics of Route Path Authorizations (RPA) provided in {{RPKI-RPA-Profile}} are applied here.

- **Route is ineligible**: The term has the same meaning as in {{RFC4271}}, i.e., "route is ineligible to be installed in Loc-RIB and will be excluded from the next phase of route selection."
- **AS-path**: This term defines a sequence of ASes listed in the BGP UPDATE AS_PATH or AS4_PATH attribute. This document uses the terms AS-path, AS_PATH, and AS4_PATH interchangeably.
- **Weakly Valid**: This is one of the verification results of using RPA objects to verify AS_PATH, indicating that at least one AS in the path is validated as VALID by RPA, while all other ASes yield an UNKNOWN verification result.
- **VRPP**: Validated RPA Payload (see {{RoutePathAuthorizations}}).

# Route Path Authorizations (RPA) {#RoutePathAuthorizations}

Route Path Authorizations (RPA) objects encapsulate the routing paths description of an Autonomous System (AS). Similar to most RPKI-signed objects, the verification results for RPA are classified into four distinct states: Valid, Weakly Valid, Invalid, and Unknown.

It is RECOMMENDED that all routing paths be explicitly enumerated within a single RPA object. However, due to the inherent complexity of routing paths, providing a comprehensive list can be challenging. Consequently, it is RECOMMENDED to include routing paths with the origins/prefixes field designated as 'NONE' when the issuer is unable to specify which routes will be propagated from previousHops to nextHops. It may have a few of these routePathBlocks with the origins/prefixes field set as 'NONE'.

In general, there exists a singular valid RPA object corresponding to a specific asID. However, in instances where multiple valid RPA objects containing the same asID are present, the union of the resulting routePathBlocks members constitutes the comprehensive set of members. This complete set, which may arise from either a single or multiple RPAs, is locally maintained by a Relying Party (RP) or a compliant router. Such an object is referred to as the Validated RPA Payload (VRPP) for the asID.

Except for the empty origins, there would also be empty previousHops and nextHops in a routing path. It is NOT RECOMMENDED to describe routing path without nextHops as this does not help verify BGP AS_PATH.

It is REQUIRED at least one routing path description in an RPA object. Otherwise, the empty RPA object means no routes can be transited or transformed from this asID.

# AS_PATH Verification Using RPA

RPAs describe the local routing paths of an AS. They can be used to verify the AS_PATH attribute in BGP UPDATE messages.

Upon receiving a BGP UPDATE message, the AS_PATH verification procedure is initiated. This process involves querying the corresponding RPA for each AS along the path individually. If the prefixes field of an RPA object is non-empty, prefix matching is performed. Furthermore, if the origins field is present, additional validations are carried out using ROA-based Route Origin Validation (ROA-ROV) as defined in {{Section 2 of RFC6811}} and SPL-ROV as defined in {{Section 4 of RPKI-SPL-Verification}}.

An eBGP router that conforms to this specification MUST implement RPA-based AS_PATH verification procedures defined below. These procedures operates in a two-stage process:

1. Per-AS Verification: At the first stage, each AS in the AS_PATH is evaluated individually based on its corresponding RPA object, if available. This stage validates whether each AS's declared routing path is consistent with the received AS_PATH attributes.
2. Path-Level Verification: At the second stage, the system derives an overall path verification result by aggregating the outcomes of the per-AS verifications. The final status reflects the consistency and completeness of the entire path with respect to the available RPAs.

## Per-AS Verification Algorithm

The verification algorithm is applied to each individual AS in the AS_PATH of the received BGP UPDATE message. For each AS, its corresponding RPA object is examined to verify attributes such as prefix scope, authorized neighbors, and origin declaration. The verification result for each AS is one of: Valid, Invalid, or Unknown, depending on the presence and content of the RPA and its alignment with the BGP announcement.

1. Query the RPA associated with AS.
2. If RPA is not available, then set AS verification result is Unknown.
3. Perform authorized neighbors matching against the AS_PATH. If RPA.previousHops or RPA.nextHops do not match the AS_PATH context, set AS verification result is Invalid.
4. If RPA.prefixes is non-empty, perform prefix matching with the UPDATE message.
5. If RPA.origins is non-empty, perform ROA-ROV and SPL-ROV validation.
6. If both prefix and origin checks succeed, set AS verification result is Valid.
7. If either check fails, set AS verification result is Invalid.
8. Else, set AS verification result is Unknown.


## Path-Level Verification Algorithm

This process determines whether the sequence of ASes in the AS_PATH attribute conforms to the collectively declared routing paths published in RPAs. By aggregating the per-AS verification results, the algorithm computes a comprehensive path verification result for each received BGP route.

1. Let valid_count is set equal to number of ASes with Valid. Let invalid_count is set equal to number of ASes with Invalid. Let unknown_count is set equal to number of ASes with Unknown.
2. If valid_count == 0 AND invalid_count == 0, then the verification result is Unknown.
3. Else, if invalid_count == 0 AND unknown_count == 0, then the verification result is Valid.
4. Else, if valid_count >= 1 AND invalid_count == 0, then the verification result is Weakly Valid.
5. Else, if invalid_count >= 1, then the verification result is Invalid.
6. Else, the verification result is Unkown.


## Mitigation Policy {#MitigationPolicy}

The specific configuration of a mitigation policy based on AS_PATH verification using RPA is at the discretion of the network operator. However, the following mitigation policy is highly recommended.

**Invalid**: If the AS_PATH is determined to be Invalid, then the route SHOULD be considered ineligible for route selection and MUST be kept in the Adj-RIB-In for potential future re-evaluation (see {{RFC9324}}).

**Valid, Weakly Valid, or Unknown**: When a route is evaluated as Unknown (using RPA-based AS_PATH verification), it SHOULD be treated at the same preference level as a route evaluated as Valid. But Valid has the highest priority in BGP route selection while Weakly Valid has a second priority.


# Operational Considerations {#OperationalConsiderations}

Multiple valid RPA objects that contain the same asID could exist. In such a case, the union of these objects forms the complete routing path set of this AS. For a given asID, it is RECOMMENDED that a CA maintains a single RPA. If an AS holder publishes an RPA, then relying parties SHOULD assume that this object is complete for that issuer AS.

If one AS receives a BGP UPDATE message with the issuer AS in the AS-path attribute which cannot match any routing path of this issuer AS, it implies that there is an AS-path forgery in this message.

# Security Considerations {#SecurityConsiderations}

TODO Security


# IANA Considerations {#IANAConsiderations}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
