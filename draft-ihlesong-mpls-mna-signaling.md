---
title: "Signaling MNA Capabilities Using IGP"
abbrev: "SIG"
category: std

docname: draft-ihlesong-mpls-mna-signaling-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Multiprotocol Label Switching"
keyword:
 - signaling
 - mpls
 - mna
venue:
  group: "Multiprotocol Label Switching"
  type: "Working Group"
  mail: "mpls@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mpls/"
  github: "uni-tue-kn/draft-ihlesong-mpls-mna-signaling"
  latest: "https://uni-tue-kn.github.io/draft-ihle-song-mpls-mna-signaling/draft-ihle-song-mpls-mna-signaling.html"

author:
 -
    fullname: Fabian Ihle
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: fabian.ihle@uni-tuebingen.de
 -
    fullname: Xueyan Song
    organization: ZTE Corporation
    country: China
    email: song.xueyan2@zte.com.cn
 -
    fullname: Michael Menth
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: michael.menth@uni-tuebingen.de

normative:

informative:
  IhMe25:
    -: ihme25
    target: https://ieeexplore.ieee.org/document/10947349
    title: MPLS Network Actions; Technological Overview and P4-Based Implementation on a High-Speed Switching ASIC
    author:
      -
        ins: F. Ihle
        name: Fabian Ihle
        org: University of Tuebingen
      -
        ins: M. Menth
        name: Michael Menth
        org: University of Tuebingen
    seriesinfo:
      DOI: 10.1109/OJCOMS.2025.3557082
    date: 2025-04-02
    format:
      PDF: https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10947349



--- abstract

This document defines capabilities of nodes supporting MPLS Network Actions (MNA) and how to signal them using IS-IS and OSPF.
The capabilities include the Readable Label Depth (RLD), supported network action opcodes, and the maximum sizes of differently scoped Network Action Sub-stacks (NAS), called the NAS_MLD.
For IS-IS and OSPF signaling, sub-TLV encodings based on existing mechanisms to signal node- and link-specific capabilities are leveraged.

--- middle

# Introduction

With the MPLS Network Action (MNA) framework, network actions are encoded in the MPLS stack.
Those can be added to the MPLS tack using in-stack data (ISD), or follow after the MPLS stack using post-stack data (PSD).
{{?I-D.ietf-mpls-mna-hdr}} defines the encoding of such network actions and their data for ISD in a so-called Network Action Substack (NAS).
These network actions are processed by all nodes on a path (hop-by-hop, HBH), by only selected nodes, or on an ingress-to-egress (I2E) basis.
LSRs have different capabilites that depend on available hardware resources, e.g., the number of LSEs they can parse.
An ingress LER that pushes network actions to an MPLS stack MUST ensure that all nodes on the path can read and support the network actions.
For that purpose, the MNA capabilities of an LSR need to be signaled to the ingress LER.

This document defines the required parameters of LSRs regarding their MNA capability and proposes a signaling extension using an IGP such as IS-IS and OSPF.

## Terminology

{::boilerplate bcp14-tagged}

### Abbreviations
This document makes use of the terms defined in {{?I-D.ietf-mpls-mna-hdr}} and in {{?I-D.ietf-mpls-mna-fwk}}.

# Definition of MNA Capabilities

This section defines the parameters that an LSR uses to signal its MNA capabilities to the ingress LER.

## The Readable Label Depth (RLD)

The Readable Label Depth (RLD) is the number of LSEs an LSR can parse without performance impact {{?I-D.ietf-mpls-mna-fwk}}.
An LSR is required to search the MPLS stack for NAS that have to be processed by the LSR.
To that end, the network actions must be within the RLD of the node.
For HBH-scoped network actions, the ingress LER that pushes the network actions MUST ensure that the actions are readable at each LSR on the path, i.e., that it is placed within the RLD of each node.

### Example

An example for the RLD parameter is given in {{fig-rld_example}}.
With an RLD of 5, an LSR is capable of reading labels A, B, C, D, and E but not F.
An RLD of 8 is required in this example to read the entire MPLS stack.

~~~~
{::include ./drawings/rld_example.txt}
~~~~
{: #fig-rld_example title="Example MPLS stack of 8 MPLS LSEs illustrating the concept of RLD."}

## Maximum NAS Sizes
This section gives a motivation for signaling maximum NAS sizes and then introduces the NAS Maximum Label Depth (NAS_MLD).

### Motivation
A NAS in the MNA header encoding is at least 2 LSEs and at most 17 LSEs large {{?I-D.ietf-mpls-mna-hdr}}.
At an LSR, one or more NAS, e.g., a select-scoped and a hop-by-hop-scoped NAS, are possible.
With two maximum-sized NAS, an LSR is required to reserve 34 LSEs in hardware to be able to process network actions.
This consumes hardware resources that may be needed to encode other LSEs, e.g., forwarding labels for SR-MPLS paths, or are not available in less capable devices.

Many use cases in the MNA framework {{?I-D.ietf-mpls-mna-usecases}} do not require a maximum-sized NAS of 17 LSEs to encode network actions and their ancillary data.
Therefore, a NAS can be up to 17 LSEs but nodes can also support smaller maximum NAS.
By signaling the maximum supported NAS size to the ingress LER, an LSR receiving packets with a larger NAS than supported is avoided.
This way, the allocated resources for NAS can be reduced if smaller maximum NAS are supported.
More resources are available for other purposes, and hardware with a low RLD can be made MNA-capable {{IhMe25}}.

### NAS Maximum Label Depth (NAS_MLD)
The maximum supported number of LSEs in a NAS that an LSR can process is referred to as NAS Maximum Label Depth (NAS_MLD) in this document.
For each scope in MNA, a separate parameter for the NAS_MLD exists, called NAS_MLD_Select, NAS_MLD_HBH, and NAS_MLD_I2E.

An LSR SHOULD signal the maximum-supported size of a NAS for each scope, i.e., the parameters NAS_MLD_Select, NAS_MLD_HBH, and NAS_MLD_I2E.
Those parameters include the Format A, B, C, and D LSEs from {{?I-D.ietf-mpls-mna-hdr}} in a NAS.

Based on the signaled parameters, the ingress LER MUST ensure the following when pushing the MPLS stack and NAS on a packet:

- The ingress LER MUST NOT push a select-scoped NAS that is larger than the signaled NAS_MLD_Select value of the node that will process the select-scoped NAS.
- The ingress LER MUST NOT push an HBH-scoped NAS that is larger than the minimum of all signaled NAS_MLD_HBH values of all nodes on the path.
- The ingress LER MUST NOT push an I2E-scoped NAS that is larger than the signaled NAS_MLD_I2E value of the egress node.

### Example

{{fig-rld_example}} illustrates the different NAS_MLD sizes in an MPLS stack that are signaled to the LSR.
In this example, a select-scoped NAS has a maximum size of 4 LSEs, a hop-by-hop-scoped NAS of 7 LSEs, and an I2E-scoped NAS of 4 LSEs.

~~~~
{::include ./drawings/nas_sizes_example.txt}
~~~~
{: #fig-nas_sizes_example title="Example MPLS stack illustrating the different NAS sizes."}


## Supported Network Action Opcodes
An LSR MUST signal the network action opcodes it supports.
If a network action opcode is not signaled, it is assumed that this opcode is not supported by the node.

# Signaling MNA Capabilites
This section defines a method for IGP routers to advertise the maximum supported numbers of LSEs in I2E-scoped NAS, select-scoped NAS, and HBH-scoped NAS, i.e., the per-scope NAS_MLD, the RLD, and supported opcodes.

## Using IS-IS
This section defines the signaling of the RLD and the NAS_MLD that can be supported for specific NAS using IS-IS node and link advertisement.
{{?rfc7981}} defines the IS-IS Router Capability TLV that supports optional sub-TLVs to signal capabilities.
Further, {{?rfc8491}} introduces a sub-TLV for node- and link-specific advertisement based on {{?rfc7981}}.
They are used to signal MNA capabilities with IS-IS.

### NAS_MLD Advertisement
To signal the per-scope NAS_MLD, this document introduces new sub-TLVs based on {{?rfc8491}}.
The NAS_MLD Sub-TLV is defined node- or link-specific as below:

~~~~
{::include ./drawings/is-is_signaling.txt}
~~~~
{: #fig-is-is_signaling title="NAS_MLD Sub-TLV for IS-IS signaling."}

- Type:
   - 15 (link-specifc) {{?rfc8491}}
   - 23 (node-specific) {{?rfc8491}}
- Length: variable (multiple of 2 octets); represents the total length of the Value field
- Value: field consists of one or more pairs of a 1-octet MSD-Type and 1-octet MSD-Value
   - NAS_MLD-Type: value defined in the "IGP MSD-Types" registry created by the IANA Considerations section of this document (I2E, HBH, or Select).
   - NAS_MLD-Value: number in the range of 2-17.

This sub-TLV is optional.
The scope of the advertisement is specific to the deployment.

### RLD Advertisment
For the RLD advertisement, a sub-TLV based on {{?rfc8491}} is requested in {{?I-D.draft-ietf-mpls-mna-fwk}}.

### Supported Network Action Opcodes
tbd

## Using OSPF
This section defines the signaling of the RLD and the NAS_MLD that can be supported for specific NAS using OSPF node and link advertisement.
{{?rfc7770}} defines the OSPF RI Opaque LSA which is used in {{?rfc8476}} to carry the node-specific provisioned SID depth of the router originating the Router Information (RI) LSA in a sub-TLV.
Further, {{?rfc7684}} defines link-specific advertisements using the optional sub-TLV of the OSPFv2 Extended Link TLV for OSPFv2, and {{?rfc8362}} defines link-specific advertisements using the optional sub-TLV of the E-Router-LSA TLV.

### NAS_MLD Advertisement
To signal the per-scope NAS_MLD, this document introduces new sub-TLVs based on {{?rfc7684}}, {{?rfc8476}}, and {{?rfc8362}}.
The NAS_MLD Sub-TLV is defined node- or link-specific as below:

~~~~
{::include ./drawings/ospf_signaling.txt}
~~~~
{: #fig-ospf_signaling title="NAS_MLD Sub-TLV for OSPF signaling."}

- Type:
    - 6 (link-specific, OSPFv2 {{?RFC7684}})
    - 9 (link-specific, OSPFv3 {{?RFC8362}})
    - 12 (node-specific, OSPFv2 and OSPFv3 {{?rfc8476}})
- Length: variable (in octets); represents the total length of the Value field
- Value: field consists of one or more pairs of a 2-octet MSD-Type and 2-octet MSD-Value
   - NAS_MLD-Type: value defined in the "IGP MSD-Types" registry created by the IANA Considerations section of this document (I2E, HBH, or Select).
   - NAS_MLD-Value: number in the range of 2-17.

This sub-TLV is optional.
The scope of the advertisement is specific to the deployment.

### RLD Advertisment
For the RLD advertisement, a sub-TLV is requested in {{?I-D.draft-ietf-mpls-mna-fwk}}.

### Supported Network Action Opcodes
tbd

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}}, {{?rfc8476}}, and {{?rfc8491}} apply to this document.

# IANA Considerations

This document requests the allocation of following codepoints in the "IGP MSD-Types" registry.

| Value | Name                     | Data Plane | Reference     |
| ----- | ------------------------ | ---------- |
| TBA1  | MLD of select-scoped NAS | MPLS       | This document |
| TBA2  | MLD of I2E-scoped NAS    | MPLS       | This document |
| TBA3  | MLD of HBH-scoped NAS    | MPLS       | This document |
{: #table_iana title="IGP Signaling Sub-TLV allocation."}

--- back
