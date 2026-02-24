---
title: "Signaling MNA Capabilities Using LSP Ping"
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
  latest: "https://uni-tue-kn.github.io/draft-ihlesong-mpls-mna-signaling/draft-ihlesong-mpls-mna-signaling.html"

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

This document defines a mechanism for discovering MPLS Network Actions (MNA) capabilities along a Label Switched Path (LSP) using the LSP Ping echo request/reply mechanism defined in RFC 8029. The capabilities include the Readable Label Depth (RLD), the maximum sizes of differently scoped Network Action Sub-stacks (NAS_MLD), and supported network action opcodes. This mechanism allows the ingress Label Edge Router (LER) to discover MNA capabilities of each transit and egress node on the path, enabling correct construction of MPLS label stacks containing MNA network actions.

--- middle

# Introduction
The MPLS Network Actions (MNA) framework {{?I-D.ietf-mpls-mna-hdr}} provides a general mechanism for encoding network actions and their data in the MPLS label stack.
Network actions are encoded in Network Action Sub-stacks (NAS) that are placed within (ISD) or follow after (PSD) the MPLS label stack.
The MNA header encoding is defined in {{?I-D.ietf-mpls-mna-hdr}}.
To correctly construct MPLS label stacks containing network actions, the ingress LER needs to know the MNA capabilities of each node along the path.
These capabilities include:

1. The Readable Label Depth (RLD): the number of Label Stack Entries (LSEs) a node can parse without performance impact.
2. The NAS Maximum Label Depth (NAS_MLD): the maximum supported NAS size for each scope (select, HBH, I2E).
3. The supported network action opcodes.

This document defines new TLVs for the MPLS echo request/reply messages {{rfc8029}} to query and report MNA capabilities. The mechanism supports both "ping" mode (querying only the egress node) and "traceroute" mode (querying all nodes along the path).

## Terminology

{::boilerplate bcp14-tagged}

### Abbreviations
This document makes use of the terms defined in {{?I-D.ietf-mpls-mna-hdr}} and in {{?I-D.ietf-mpls-mna-fwk}}.

| Abbreviation | Name                     | Description                                                                              | Reference                  |
| ------------ | ------------------------ | ---------------------------------------------------------------------------------------- | -------------------------- |
| NAS          | Network Action Sub-stack | A stack of related LSEs in the MPLS stack containing network actions and ancillary data. | {{?rfc9789}}               |
| RLD          | Readable Label Depth     | The number of LSEs a node can parse.                                                     | {{?I-D.ietf-mpls-mna-hdr}} |
| NAS_MLD      | NAS Maximum Label Depth  | The maximum number of LSEs in a NAS that a node can process, defined per scope.          | This document              |
{: #table_abbrev title="Abbreviations."}


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

{{fig-nas_sizes_example}} illustrates the different NAS_MLD sizes in an MPLS stack that are signaled to the LSR.
In this example, a select-scoped NAS has a maximum size of 4 LSEs, a hop-by-hop-scoped NAS of 7 LSEs, and an I2E-scoped NAS of 4 LSEs.

~~~~
{::include ./drawings/nas_sizes_example.txt}
~~~~
{: #fig-nas_sizes_example title="Example MPLS stack illustrating the different NAS sizes."}


## Supported Network Action Opcodes
An LSR MUST signal the network action opcodes it supports.
If a network action opcode is not signaled, it is assumed that this opcode is not supported by the node.

# LSP Ping MNA Operation Overview
The MNA capability discovery mechanism operates as follows:

1.  The ingress LER sends MPLS echo request messages containing the MNA Capabilities Query TLV. In traceroute mode, echo requests are sent with incrementing TTL values to reach each node on the path. In ping mode, a single echo request is sent to the egress LER.
2. Each node that receives the echo request and supports MNA capability discovery responds with an MPLS echo reply containing the MNA Capabilities Response TLV. The response includes sub-TLVs corresponding to the queried capabilities.
3. The ingress LER aggregates the received responses to determine path-wide MNA constraints. Specifically:

  - The path-wide RLD is the minimum RLD reported by any node on the path.
  - The path-wide NAS_MLD_HBH is the minimum NAS_MLD_HBH reported by any node.
  - The NAS_MLD_Select for a specific node is the value reported by that node.
  - The NAS_MLD_I2E is the value reported by the egress node.
  - The path-wide supported opcodes for HBH-scoped NAS is the intersection of opcodes supported by all nodes.

The ingress LER SHOULD perform MNA capability discovery before pushing MNA-enabled label stacks onto a path. The ingress LER SHOULD re-query capabilities when the path changes, e.g., due to IGP reconvergence or Fast Reroute activation.

## MNA Capabilities Query TLV
The MNA Capabilities Query TLV is carried in the MPLS Echo Request message.
Its format is as follows:

~~~~
{::include ./drawings/query-tlv.txt}
~~~~
{: #fig-query-tlv title="MNA Capabilities Query TLV."}

The fields are defined as follows:

- Type: Indicates the MNA Capabilities Query TLV. The value is TBA1.
- Length: The length of the Value field in octets. For this TLV, Length is 4.
- Query Flags: An 8-bit field indicating which capabilities are being queried:

| Bit | Name          | Description                                         |
| --- | ------------- | --------------------------------------------------- |
| 0   | QUERY_RLD     | Query the Readable Label Depth                      |
| 1   | QUERY_NAS_MLD | Query NAS Maximum Label Depth (scopes)              |
| 2   | QUERY_OPCODES | Query supported network action opcodes              |
| 3-7 | Reserved      | MUST be set to zero on transmit, ignored on receipt |
{: #query-flags title="Query Flags."}

- Reserved: MUST be set to zero on transmit and MUST be ignored on receipt.

A node that receives an MNA Capabilities Query TLV with all Query Flags set to zero SHOULD respond with all available MNA capabilities.

## MNA Capabilities Response TLV
The MNA Capabilities Response TLV is carried in the MPLS Echo Reply message. Its format is as follows:

~~~~
{::include ./drawings/response-tlv.txt}
~~~~
{: #fig-response-tlv title="MNA Capabilities Response TLV."}

The fields are defined as follows:

- Type: Indicates the MNA Capabilities Response TLV. The value is TBA2.
- Length: The length of the Value field in octets.

The Value field consists of one or more sub-TLVs as defined in the following sections. The responding node MUST include sub-TLVs corresponding to the flags set in the Query TLV. If no flags were set in the query, the responding node SHOULD include all sub-TLVs for which it has information.

### RLD Sub-TLV
The RLD Sub-TLV reports the Readable Label Depth of the responding node.

~~~~
{::include ./drawings/rld-tlv.txt}
~~~~
{: #fig-rld-tlv title="RLD Sub-TLV."}

- Sub-type: 1 (RLD).
- Length: 4 octets.
- RLD Value: An 8-bit unsigned integer indicating the number of LSEs the node can parse without performance impact. A value of 0 indicates that the node did not provide an RLD value.
- Reserved: MUST be set to zero on transmit and MUST be ignored on receipt.

### NAS_MLD Sub-TLV
The NAS_MLD Sub-TLV reports the maximum supported NAS sizes for each scope. All three scope values are encoded in a single sub-TLV.

~~~~
{::include ./drawings/mld-tlv.txt}
~~~~
{: #fig-mld-tlv title="NAS_MLD Sub-TLV."}

- Sub-type: 2 (NAS_MLD).
- Length: 4 octets.
- NAS_MLD_Select: An 8-bit unsigned integer indicating the maximum number of LSEs in a select-scoped NAS that the node can process. Valid range: 2-17. A value of 0 indicates that select-scoped NAS are not supported.
- NAS_MLD_HBH: An 8-bit unsigned integer indicating the maximum number of LSEs in an HBH-scoped NAS that the node can process. Valid range: 2-17. A value of 0 indicates that HBH-scoped NAS are not supported.
- NAS_MLD_I2E: An 8-bit unsigned integer indicating the maximum number of LSEs in an I2E-scoped NAS that the node can process. Valid range: 2-17. A value of 0 indicates that I2E-scoped NAS are not supported.
- Reserved: MUST be set to zero on transmit and MUST be ignored on receipt.

### Supported Opcodes Sub-TLV
The Supported Opcodes Sub-TLV reports the network action opcodes supported by the responding node using a bitmap encoding. The MNA opcode space is 7 bits, supporting 128 opcodes. Each bit in the bitmap corresponds to one opcode value.

~~~~
{::include ./drawings/opcode-tlv.txt}
~~~~
{: #fig-opcode-tlv title="Opcode Sub-TLV."}

- Sub-type: 3 (Supported Opcodes).
- Length: 16 octets.
- Supported Opcodes bitmap: A 128-bit bitmap where bit N (counting from bit 0 as the most significant bit of the first octet) corresponds to opcode value N. If bit N is set to 1, the node supports opcode N. If bit N is set to 0, the node does not support opcode N.

This bitmap encoding is compact and fixed-size regardless of the number of supported opcodes. It allows efficient computation of the intersection of supported opcodes across all nodes on a path using bitwise AND operations.

# Processing Rules
This section defines the processing rules for querying and responding nodes.

## Ingress LER (Querier)
The ingress LER constructs MPLS echo request messages containing the MNA Capabilities Query TLV.
In traceroute mode, the ingress LER sends echo requests with TTL values from 1 to the path length. This allows discovery of MNA capabilities at each hop.
Traceroute mode SHOULD be used when HBH-scoped network actions are planned, as the ingress LER needs the capabilities of every node to correctly place NAS copies within each node's RLD.
In ping mode, a single echo request with TTL set to 255 is sent.
This is sufficient when only I2E-scoped network actions are planned, as only the egress node's capabilities are needed.
After collecting responses, the ingress LER computes path-wide constraints as described in Section 3.
The ingress LER MUST ensure the following when constructing MPLS stacks with MNA:

1.  A select-scoped NAS pushed for a specific node MUST NOT exceed that node's NAS_MLD_Select.
2. An HBH-scoped NAS MUST NOT exceed the minimum NAS_MLD_HBH across all nodes on the path.
3. An I2E-scoped NAS MUST NOT exceed the egress node's NAS_MLD_I2E.
4. All NAS intended for a node MUST be within that node's RLD.

## Responding Node
A node that supports MNA and receives an MPLS echo request containing the MNA Capabilities Query TLV MUST respond with an MPLS echo reply containing the MNA Capabilities Response TLV.
The responding node MUST include sub-TLVs corresponding to the flags set in the query.
If the QUERY_RLD flag is set, the RLD Sub-TLV MUST be included.
If the QUERY_NAS_MLD flag is set, the NAS_MLD Sub-TLV MUST be included.
If the QUERY_OPCODES flag is set, the Supported Opcodes Sub-TLV MUST be included.
If no Query Flags are set (all zero), the responding node SHOULD include all available sub-TLVs.
The reported capabilities are those of the node as a whole.
If capabilities vary per interface, the node SHOULD report the capabilities applicable to the interface on which the echo request was received.

## MNA-incapable Nodes
A node that does not support MNA will not recognize the MNA Capabilities Query TLV.
According to RFC 8029, the handling depends on the TLV type value range.
The TLV type for the MNA Capabilities Query TLV SHOULD be assigned from the range that requires an error message if the TLV is not recognized.
This allows the ingress LER to detect nodes that do not support MNA.

If a node does not support MNA, but recognizes the MNA Capabilities Query TLV, it MUST reply with the following Return Code for MPLS echo reply messages:

| Value | Meaning           | Reference     |
| ----- | ----------------- | ------------- |
| TBA3  | MNA not supported | This document |
{: #table_return_code title="New return code."}


# Example
Consider an SR-MPLS path with three LSRs: R1, R2 (transit), and R3 (egress). The ingress LER R0 wants to push an HBH-scoped NAS and a select-scoped NAS for R2 along this path.
R0 sends MPLS echo requests in traceroute mode with all Query Flags set. The responses are:

| Node | RLD | MLD_Select | MLD_HBH | MLD_I2E         |
| ---- | --- | ---------- | ------- | --------------- |
| R1   | 20  | 9          | 9       | 0 (not egress ) |
| R2   | 51  | 9          | 3       | 0 (not egress ) |
| R3   | 35  | 9          | 9       | 9               |
{: #table_example_ping title="Example MNA Capabilities Responses."}

From these responses, R0 determines:

- Path-wide RLD: min(20, 51, 35) = 20 LSEs
- Path-wide NAS_MLD_HBH: min(9, 3, 9) = 3 LSEs
- NAS_MLD_Select for R2: 9 LSEs
- NAS_MLD_I2E: 9 LSEs (from R3)

R0 can now construct a label stack ensuring that all NAS are within each node's RLD and do not exceed the per-scope NAS_MLD constraints.

# Security Considerations
The security considerations described in {{?rfc8029}} apply to this document.
The MNA capability discovery mechanism reveals information about node capabilities, which could potentially be exploited by an attacker to craft targeted attacks against nodes with limited MNA support.
Nodes that support MNA capability discovery SHOULD support configuration options to enable or disable the MNA Capabilities Query/Response functionality.
By default, MNA capability discovery SHOULD be enabled only within an MNA-capable MPLS domain.
The security considerations from {{?I-D.ietf-mpls-mna-hdr}} and {{?rfc9789}} also apply.

# IANA Considerations
This section requests new TLVs and sub-TLVs.

## TLV Assignments
IANA is requested to assign two new TLVs from the "TLV" registry in the "Multiprotocol Label Switching (MPLS) Label Switched Paths (LSPs) Ping Parameters" registry group.
The TLV values SHOULD be assigned from the range that requires an error message if the TLV is not recognized.

| Value | TLV Name                  | Reference     | Sub-TLV Registry    |
| ----- | ------------------------- | ------------- | ------------------- |
| TBA1  | MNA Capabilities Query    | This document | No                  |
| TBA2  | MNA Capabilities Response | This document | See {{table_iana2}} |
{: #table_iana title="New TLVs."}

## New Sub-TLV Registry
IANA is requested to create a new sub-TLV registry for TLV TBA2 with the following initial entries:

| Sub-Type | Sub-TLV Name      | Reference     |
| -------- | ----------------- | ------------- |
| 0        | Reserved          | This document |
| 1        | RLD               | This document |
| 2        | NAS_MLD           | This document |
| 3        | Supported Opcodes | This document |
{: #table_iana2 title="Sub-TLV Registry for TLV TBA2."}


--- back
