---
v: 3

title: Key Management for Group Object Security for Constrained RESTful Environments (Group OSCORE) Using Authentication and Authorization for Constrained Environments (ACE)
abbrev: Key Management for Group OSCORE Using ACE
docname: draft-ietf-ace-key-groupcomm-oscore-latest

# stand_alone: true

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft
cat: std
submissiontype: IETF

coding: utf-8
pi:    # can use array (if all yes) or hash here

  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:
-
    ins: M. Tiloca
    name: Marco Tiloca
    org: RISE AB
    street: Isafjordsgatan 22
    city: Kista
    code: SE-164 29 Stockholm
    country: Sweden
    email: marco.tiloca@ri.se
-
    ins: F. Palombini
    name: Francesca Palombini
    org: Ericsson AB
    street: Torshamnsgatan 23
    city: Kista
    code: SE-16440 Stockholm
    country: Sweden
    email: francesca.palombini@ericsson.com

normative:
  RFC5246:
  RFC5705:
  RFC6347:
  RFC6979:
  RFC7252:
  RFC7748:
  RFC8017:
  RFC8032:
  RFC8126:
  RFC8446:
  RFC8447:
  RFC8610:
  RFC8613:
  RFC8949:
  RFC9052:
  RFC9053:
  RFC9147:
  RFC9200:
  RFC9202:
  RFC9203:
  RFC9237:
  RFC9277:
  RFC9290:
  RFC9430:
  RFC9594:
  I-D.ietf-core-oscore-groupcomm:
  NIST-800-56A:
    author:
      -
        ins: E. Barker
        name: Elaine Barker
      -
        ins: L. Chen
        name: Lily Chen
      -
        ins: A. Roginsky
        name: Allen Roginsky
      -
        ins: A. Vassilev
        name: Apostol Vassilev
      -
        ins: R. Davis
        name: Richard Davis
    title: Recommendation for Pair-Wise Key-Establishment Schemes Using Discrete Logarithm Cryptography - NIST Special Publication 800-56A, Revision 3
    date: 2018-04
    target: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-56Ar3.pdf
  COSE.Algorithms:
    author:
      org: IANA
    date: false
    title: COSE Algorithms
    target: https://www.iana.org/assignments/cose/cose.xhtml#algorithms
  COSE.Key.Types:
    author:
      org: IANA
    date: false
    title: COSE Key Types
    target: https://www.iana.org/assignments/cose/cose.xhtml#key-type
  COSE.Elliptic.Curves:
    author:
      org: IANA
    date: false
    title: COSE Elliptic Curves
    target: https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves
  COSE.Header.Parameters:
    author:
      org: IANA
    date: false
    title: COSE Header Parameters
    target: https://www.iana.org/assignments/cose/cose.xhtml#header-parameters

informative:
  I-D.ietf-core-groupcomm-bis:
  I-D.ietf-core-coap-pubsub:
  I-D.tiloca-core-oscore-discovery:
  I-D.ietf-ace-oscore-gm-admin:
  I-D.ietf-cose-cbor-encoded-cert:
  RFC5280:
  RFC5869:
  RFC6690:
  RFC6749:
  RFC7641:
  RFC8392:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document defines an application profile of the Authentication and Authorization for Constrained Environments (ACE) framework, to request and provision keying material in group communication scenarios that are based on the Constrained Application Protocol (CoAP) and are secured with Group Object Security for Constrained RESTful Environments (Group OSCORE). This application profile delegates the authentication and authorization of Clients, which join an OSCORE group through a Resource Server acting as Group Manager for that group. This application profile leverages protocol-specific transport profiles of ACE to achieve communication security, server authentication, and proof-of-possession for a key owned by the Client and bound to an OAuth 2.0 access token.

--- middle

# Introduction {#sec-introduction}

The secure communication protocol Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} provides application-layer protection for the Constrained Application Protocol (CoAP) {{RFC7252}}, using CBOR Object Signing and Encryption (COSE) {{RFC9052}}{{RFC9053}} and enabling end-to-end security of CoAP messages.

As defined in {{I-D.ietf-core-oscore-groupcomm}}, Group Object Security for Constrained RESTful Environments (Group OSCORE) enables end-to-end security for CoAP group communication {{I-D.ietf-core-groupcomm-bis}}, which can employ, for example, IP multicast as underlying data transport.

Group OSCORE relies on an entity called Group Manager, which is responsible for managing an OSCORE group and enables the group members to exchange CoAP messages secured with Group OSCORE. The Group Manager can be responsible for multiple groups, coordinates the joining process of new group members, and is entrusted with the distribution and renewal of group keying material.

This document is an application profile of {{RFC9594}}, which itself builds on the Authentication and Authorization for Constrained Environments (ACE) framework {{RFC9200}}. Message exchanges among the participants as well as message formats and processing follow what is specified in {{RFC9594}}, and enable the provisioning and renewing of keying material in group communication scenarios, where Group OSCORE is used to protect CoAP group communication.

## Terminology {#ssec-terminology}

{::boilerplate bcp14-tagged}

Readers are expected to be familiar with:

* The terms and concepts described in the ACE framework for authentication and authorization {{RFC9200}} and in the Authorization Information Format (AIF) {{RFC9237}} to express authorization information. The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}}. This includes Client (C), Resource Server (RS), and Authorization Server (AS).

* The terms and concepts related to the message formats and processing specified in {{RFC9594}}, for provisioning and renewing keying material in group communication scenarios. These include the abbreviations REQx and OPTx denoting the numbered mandatory-to-address and optional-to-address requirements, respectively.

* The terms and concepts related to Concise Data Definition Language (CDDL) {{RFC8610}}, Concise Binary Object Representation (CBOR) {{RFC8949}}, and COSE {{RFC9052}}{{RFC9053}}.

* The terms and concepts related to CoAP {{RFC7252}} and group communication for CoAP {{I-D.ietf-core-groupcomm-bis}}. Unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS, and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

* The terms and concepts for protection and processing of CoAP messages through OSCORE {{RFC8613}} and through Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} in group communication scenarios. These especially include:

    - Group Manager, as the entity responsible for a set of groups where communications are secured with Group OSCORE. In this document, the Group Manager acts as Resource Server.

    - Authentication credential, as the set of information associated with an entity, including that entity's public key and parameters associated with the public key. Examples of authentication credentials are CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC5280}}, and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}.

Additionally, this document makes use of the following terminology:

* Requester: member of an OSCORE group that sends request messages to other members of the group.

* Responder: member of an OSCORE group that receives request messages from other members of the group. A responder may reply back, by sending a response message to the requester which has sent the request message.

* Monitor: member of an OSCORE group that is configured as responder and never sends response messages protected with Group OSCORE. This corresponds to the term "silent server" used in {{I-D.ietf-core-oscore-groupcomm}}.

* Signature verifier: entity external to the OSCORE group, and intended to verify the signature of messages exchanged in the group that are protected with the group mode (see {{Sections 12.3, 7, and 7.5 of I-D.ietf-core-oscore-groupcomm}}).

  An authorized signature verifier does not join the OSCORE group as an actual member. However, it can interact with the Group Manager in order to retrieve what is needed to perform signature verifications, e.g., the authentication credentials of the current group members and of the Group Manager.

* Signature-only group: an OSCORE group that uses only the group mode (see {{Section 7 of I-D.ietf-core-oscore-groupcomm}}).

* Pairwise-only group: an OSCORE group that uses only the pairwise mode (see {{Section 8 of I-D.ietf-core-oscore-groupcomm}}).

Throughout this document, examples for CBOR data items are expressed in CBOR extended diagnostic notation as defined in {{Section 8 of RFC8949}} and {{Appendix G of RFC8610}} ("diagnostic notation"). Diagnostic notation comments are often used to provide a textual representation of the parameters' keys and values.

In the CBOR diagnostic notation used in this document, constructs of the form e'SOME_NAME' are replaced by the value assigned to SOME_NAME in the CDDL model shown in {{fig-cddl-model}} of {{sec-cddl-model}}. For example, {e'gp_enc_alg': 10, e'sign_alg': -8} stands for {9: 10, 10: -8}.

Note to RFC Editor: Please delete the paragraph immediately preceding this note. Also, in the CBOR diagnostic notation used in this document, please replace the constructs of the form e'SOME_NAME' with the value assigned to SOME_NAME in the CDDL model shown in {{fig-cddl-model}} of {{sec-cddl-model}}. Finally, please delete this note.

# Protocol Overview {#sec-protocol-overview}

Group communication for CoAP has been enabled in {{I-D.ietf-core-groupcomm-bis}} and can be secured by using Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}. A network node can join an OSCORE group by interacting with the responsible Group Manager. Once registered in the group, the new node can securely exchange messages with other group members.

This document describes how to use {{RFC9594}} and {{RFC9200}} to perform a number of authentication, authorization, and key distribution actions as overviewed in {{Section 2 of RFC9594}}, when the considered group is specifically an OSCORE group.

With reference to {{RFC9594}}:

* The node wishing to join the OSCORE group, i.e., the joining node, is the Client.

* The Group Manager is the Key Distribution Center (KDC), acting as a Resource Server.

* The Authorization Server associated with the Group Manager is the AS.

A node performs the steps described in {{Sections 3 and 4.3.1.1 of RFC9594}} in order to obtain an authorization for joining an OSCORE group and then to join that group. The format and processing of messages exchanged during such steps are further specified in {{sec-joining-node-to-AS}} and {{sec-joining-node-to-GM}} of this document.

All communications between the involved entities MUST be secured.

In particular, communications between the Client and the Group Manager leverage protocol-specific transport profiles of ACE to achieve communication security, proof-of-possession, and server authentication. It is expected that, in the commonly referred base-case of this document, the transport profile to use is pre-configured and well-known to nodes participating in constrained applications.

With respect to what is defined in {{RFC9594}}:

* The interface provided by the Group Manager extends the original interface defined in {{Section 4.1 of RFC9594}} for the KDC, as specified in {{sec-interface-GM}} of this document.

* In addition to those defined in {{Section 8 of RFC9594}}, additional parameters are defined in this document and summarized in {{ace-groupcomm-params}}.

* In addition to those defined in {{Section 9 of RFC9594}}, additional error identifiers are defined in this document and summarized in {{error-types}}.

Finally, {{profile-req}} lists the specifications on this application profile of ACE, based on the requirements defined in {{Section A of RFC9594}}.

# Format of Scope {#sec-format-scope}

Building on the definition in {{Section 3.3 of RFC6749}} considered in the ACE framework {{RFC9200}}, scope denotes: the permissions that the Client seeks to obtain from the AS for accessing resources at a Resource Server; and the permissions that the AS actually issues to the Client following its request. This process is detailed in {{Sections 5.8.1 and 5.8.2 of RFC9200}}.

Consistent with the above and building on {{Section 3.1 of RFC9594}}, this section defines the exact format and encoding of scope used in this profile.

To this end, this profile uses the Authorization Information Format (AIF) {{RFC9237}}. With reference to the generic AIF model

~~~~~~~~~~~
   AIF-Generic<Toid, Tperm> = [* [Toid, Tperm]]
~~~~~~~~~~~

the value of the CBOR byte string used as scope encodes the CBOR array \[* \[Toid, Tperm\]\], where each \[Toid, Tperm\] element corresponds to one scope entry.

Furthermore, this document defines the new AIF data model AIF-OSCORE-GROUPCOMM, which this profile MUST use to format and encode scope entries.

In particular, the following holds for each scope entry:

* The object identifier ("Toid") is specialized as a CBOR data item specifying the name of the groups pertaining to the scope entry.

* The permission set ("Tperm") is specialized as a CBOR unsigned integer with value R, specifying the permissions that the Client wishes to have in the groups indicated by "Toid".

More specifically, the following applies when, as defined in this document, a scope entry includes as set of permissions the set of roles to take in an OSCORE group.

* The object identifier ("Toid") is a CBOR text string, specifying the group name for the scope entry.

* The permission set ("Tperm") is a CBOR unsigned integer with value R, specifying the role(s) that the Client wishes to take in the group (REQ1). The value R is computed as follows.

   - Each role in the permission set is converted into the corresponding numeric identifier X from the "Value" column of the "Group OSCORE Roles" registry defined in {{ssec-iana-group-oscore-roles-registry}} of this document, for which the initial entries are specified in {{tab-role-values}}.

   - The set of N numbers is converted into the single value R, by taking two to the power of each numeric identifier X_1, X_2, ..., X_N, and then computing the inclusive OR of the binary representations of all the power values.

| Name      | Value | Description                                                            |
|-----------|-------|------------------------------------------------------------------------|
| Reserved  | 0     | This value is reserved                                                 |
|-----------|-------|------------------------------------------------------------------------|
| Requester | 1     | Send protected requests; receive protected responses                   |
|-----------|-------|------------------------------------------------------------------------|
| Responder | 2     | Send protected responses; receive protected requests                   |
|-----------|-------|------------------------------------------------------------------------|
| Monitor   | 3     | Receive protected requests; never send protected messages              |
|-----------|-------|------------------------------------------------------------------------|
| Verifier  | 4     | Verify signature of intercepted messages protected with the group mode |
{: #tab-role-values title="Numeric Identifier of Roles in an OSCORE Group" align="center"}

The following CDDL {{RFC8610}} notation defines a scope entry that uses the AIF-OSCORE-GROUPCOMM data model and expresses a set of Group OSCORE roles from those in {{tab-role-values}}.

~~~~~~~~~~~ CDDL
   ;# include rfc9237

   AIF-OSCORE-GROUPCOMM = AIF-Generic<oscore-gname, oscore-gperm>

   oscore-gname = tstr  ; Group name
   oscore-gperm = uint .bits group-oscore-roles

   group-oscore-roles = &(
      Requester: 1,
      Responder: 2,
      Monitor: 3,
      Verifier: 4
   )

   scope_entry = [oscore-gname, oscore-gperm]
~~~~~~~~~~~

Future specifications that define new Group OSCORE roles MUST register a corresponding numeric identifier in the "Group OSCORE Roles" registry.

Note that the value 0 is not available to use as numeric identifier to specify a Group OSCORE role. It follows that, when expressing Group OSCORE roles to take in a group as per this document, a scope entry has the least significant bit of "Tperm" always set to 0.

This is an explicit feature of the AIF-OSCORE-GROUPCOMM data model. That is, for each scope entry, the least significant bit of "Tperm" set to 0 explicitly identifies the scope entry as exactly expressing a set of Group OSCORE roles ("Tperm"), pertaining to a single group whose name is specified by the string literal in "Toid".

Instead, by relying on the same AIF-OSCORE-GROUPCOMM data model, {{I-D.ietf-ace-oscore-gm-admin}} defines the format of scope entries for Administrator Clients that wish to access an admin interface at the Group Manager. In such scope entries, the least significant bit of "Tperm" is always set to 1.

# Authentication Credentials # {#sec-public-keys-of-joining-nodes}

Source authentication of a message sent within the group and protected with Group OSCORE is ensured by means of a digital signature embedded in the message (in group mode), or by integrity-protecting the message with pairwise keying material derived from the asymmetric keys of the sender and recipient (in pairwise mode).

Therefore, group members must be able to retrieve each other's authentication credentials from a trusted repository, in order to verify source authenticity of incoming group messages.

As also discussed in {{I-D.ietf-core-oscore-groupcomm}}, the Group Manager acts as trusted repository of the authentication credentials of the group members, and provides those authentication credentials to group members if requested to.

Upon joining an OSCORE group, a joining node is thus expected to provide its authentication credential to the Group Manager (see {{ssec-join-req-sending}}). Later on as a group member, that node can provide the Group Manager with a different authentication credential that replaces the old one (see {{sec-update-pub-key}}). In either situation, the authentication credential can be provided within a chain or a bag (e.g., as the end-entity certificate in a chain of certificates), in which case the Group Manager stores the whole chain or bag. Consistently, the Group Manager specifies the whole chain or bag when providing that authentication credential, within the 'creds' parameter of a Join Response (see {{ssec-join-resp}}) or of an Authentication Credential Response (see {{sec-pub-keys}}).

In the following circumstances, a joining node is not required to provide its authentication credential to the Group Manager when joining an OSCORE group.

* The joining node is going to join the group exclusively as monitor, i.e., it is not going to send protected messages to the group.

  In this case, the joining node is not required to provide its own authentication credential to the Group Manager, which thus does not have to perform any check related to the format of the authentication credential, to a signature or ECDH algorithm, and to possible parameters associated with the algorithm and the public key.

  If the joining node still provides an authentication credential in the 'client_cred' parameter of the Join Request (see {{ssec-join-req-sending}}), the Group Manager silently ignores that parameter and the related parameter 'client_cred_verify'.

* The joining node is currently a group member, and it is re-joining the group not exclusively as monitor.

  In this case, the joining node MAY choose to omit the 'client_cred' parameter and the 'client_cred_verify' parameter in the Join Request, if it intends to use the same authentication credential that it is currently using in the group (see {{Section 4.3.1.1 of RFC9594}}).

# Authorization to Join a Group {#sec-joining-node-to-AS}

This section builds on {{Section 3 of RFC9594}} and is organized as follows.

First, {{ssec-auth-req}} and {{ssec-auth-resp}} describe how the joining node interacts with the AS, in order to be authorized to join an OSCORE group under a given Group Manager and to obtain an access token. Then, {{ssec-token-post}} describes how the joining node transfers the obtained access token to the Group Manager.

The following considers a joining node that intends to contact the Group Manager for the first time.

Note that what is defined in {{Section 3 of RFC9594}} applies, and only additions or modifications to that specification are defined in this document.

## Authorization Request {#ssec-auth-req}

The Authorization Request message is as defined in {{Section 3.1 of RFC9594}}, with the following additions.

* If the 'scope' parameter is present:

   - The value of the CBOR byte string encodes a CBOR array, whose format MUST follow the data model AIF-OSCORE-GROUPCOMM defined in {{sec-format-scope}} of this document. For each OSCORE group to join:

      - The group name is encoded as a CBOR text string.

      - The set of requested roles is expressed as a single CBOR unsigned integer. This is computed as defined in {{sec-format-scope}} of this document, from the numerical abbreviations of each requested role defined in the "Group OSCORE Roles" registry, for which this document defines the entries in {{tab-role-values}} (REQ1).

## Authorization Response {#ssec-auth-resp}

The Authorization Response message is as defined in {{Section 3.2 of RFC9594}}, with the following additions:

* The AS MUST include the 'expires_in' parameter. Other means for the AS to specify the lifetime of access tokens are out of the scope of this document.

* The AS MUST include the 'scope' parameter, when the value included in the access token differs from the one specified by the joining node in the Authorization Request. In such a case, the second element of each scope entry MUST be present, and specifies the set of roles that the joining node is actually authorized to take in the OSCORE group for that scope entry, encoded as specified in {{ssec-auth-req}} of this document.

Furthermore, the AS MAY use the extended format of scope defined in {{Section 7 of RFC9594}} for the 'scope' claim of the access token. In such a case, the AS MUST use the CBOR tag with tag number TAG_NUMBER, associated with the CoAP Content-Format CF_ID for the media type "application/aif+cbor" registered in {{ssec-iana-coap-content-format-registry}} of this document (REQ28).

Note to RFC Editor: In the previous paragraph, please replace "TAG_NUMBER" with the CBOR tag number computed as TN(ct) in {{Section 4.3 of RFC9277}}, where ct is the ID assigned to the CoAP Content-Format registered in {{ssec-iana-coap-content-format-registry}} of this document. Then, please replace "CF_ID" with the ID assigned to that CoAP Content-Format. Finally, please delete this paragraph.

This indicates that the binary encoded scope, as conveying the actual access control information, follows the scope semantics defined for this application profile in {{sec-format-scope}} of this document.

## Token Transferring {#ssec-token-post}

The exchange of Token Transfer Request and Token Transfer Response is defined in {{Section 3.3 of RFC9594}}. In addition to that, the following applies.

* The Token Transfer Request MAY additionally contain the following parameters, which, if included, MUST have the corresponding values defined below (OPT2):

   - 'ecdh_info' defined in {{ecdh-info}} of this document, with value the CBOR simple value `null` (0xf6) to request information about the ECDH algorithm, the ECDH algorithm parameters, the ECDH key parameters, and the exact format of authentication credentials used in the OSCORE groups that the Client has been authorized to join. This is relevant in case the joining node supports the pairwise mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}.

   - 'kdc_dh_creds' defined in {{gm-dh-info}} of this document, with value the CBOR simple value `null` (0xf6) to request the Diffie-Hellman authentication credentials of the Group Manager for the OSCORE groups that the Client has been authorized to join. That is, each of such authentication credentials includes a Diffie-Hellman public key of the Group Manager. This is relevant in case the joining node supports the pairwise mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} and the access token authorizes to join pairwise-only groups.

   Alternatively, the joining node may retrieve this information by other means.

* The 'kdcchallenge' parameter contains a dedicated nonce N_S generated by the Group Manager. For the N\_S value, it is RECOMMENDED to use an 8-byte long random nonce. The joining node can use this nonce in order to prove the possession of its own private key, upon joining the group (see {{ssec-join-req-sending}} of this document).

    The 'kdcchallenge' parameter MAY be omitted from the Token Transfer Response, if the 'scope' of the access token specifies only the role "monitor", or only the role "verifier", or only the two roles combined, for each and every of the specified groups.

* If the 'sign_info' parameter is present in the Token Transfer Response, the following applies for each element 'sign_info_entry'.

  * 'id' MUST NOT refer to OSCORE groups that are pairwise-only groups.

  * 'sign_alg' takes value from the "Value" column of the "COSE Algorithms" registry {{COSE.Algorithms}} (REQ3).

  * 'sign_parameters' has the same format and value of the COSE capabilities array for the algorithm indicated in 'sign_alg', as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{COSE.Algorithms}} (REQ4).

  * 'sign_key_parameters' has the same format and value of the COSE capabilities array for the COSE key type of the keys used with the algorithm indicated in 'sign_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}} (REQ5).

  * 'cred_fmt' takes value from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}} (REQ6). Consistently with {{Section 2.4 of I-D.ietf-core-oscore-groupcomm}}, acceptable values denote a format of authentication credential that MUST explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm, including, e.g., the used elliptic curve.

     At the time of writing this specification, acceptable formats of authentication credentials are CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC5280}}, and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}. Further formats may be available in the future, and would be acceptable to use as long as they comply with the criteria defined above.

     \[ As to C509 certificates, the COSE Header Parameters 'c5b' and 'c5c' are under pending registration requested by draft-ietf-cose-cbor-encoded-cert. \]

  This format is consistent with every signature algorithm currently considered in {{RFC9053}}, i.e., with algorithms that have only the COSE key type as their COSE capability. {{Section B of RFC9594}} describes how the format of each 'sign_info_entry' can be generalized for possible future registered algorithms having a different set of COSE capabilities.

* If the 'ecdh_info' parameter is present in the Token Transfer Response, the following applies for each element 'ecdh_info_entry'.

  * 'id' MUST NOT refer to OSCORE groups that are signature-only groups.

  * 'ecdh_alg' takes value from the "Value" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

  * 'ecdh_parameters' has the same format and value of the COSE capabilities array for the algorithm indicated in 'ecdh_alg', as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

  * 'ecdh_key_parameters' has the same format and value of the COSE capabilities array for the COSE key type of the keys used with the algorithm indicated in 'ecdh_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}}.

  * 'cred_fmt' takes value from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}}. Consistently with {{Section 2.4 of I-D.ietf-core-oscore-groupcomm}}, acceptable values denote a format of authentication credential that MUST explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm, including, e.g., the used elliptic curve. The same considerations provided above on acceptable formats currently available for the 'cred_fmt' element of 'sign_info' apply.

  The Group Manager omits the 'ecdh_info' parameter in the Token Transfer Response even if 'ecdh_info' is included in the Token Transfer Request, in case all the OSCORE groups that the Client is authorized to join are signature-only groups.

* If the 'kdc_dh_creds' parameter is present in the Token Transfer Response, the following applies for each element 'kdc_dh_creds_entry'.

  * 'id' MUST refer exclusively to OSCORE groups that are pairwise-only groups.

  * 'cred_fmt' takes value from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}}. Consistently with {{Section 2.4 of I-D.ietf-core-oscore-groupcomm}}, acceptable values denote a format of authentication credential that MUST explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm, including, e.g., the used elliptic curve. The same considerations provided above on acceptable formats currently available for the 'cred_fmt' element of 'sign_info' apply.

  The Group Manager omits the 'kdc_dh_creds' parameter in the Token Transfer Response even if 'ecdh_info' is included in the Token Transfer Request, in case none of the OSCORE groups that the Client is authorized to join is a pairwise-only group.

Note that, other than through the above parameters as defined in {{Section 3.3 of RFC9594}}, the joining node may have obtained such information by alternative means. For example, information conveyed in the 'sign_info' and 'ecdh_info' parameters may have been pre-configured, or the joining node may early retrieve it, e.g., by using the approach described in {{I-D.tiloca-core-oscore-discovery}} to discover the OSCORE group and the link to the associated group-membership resource at the Group Manager (OPT3).

### 'ecdh_info' Parameter {#ecdh-info}

The 'ecdh_info' parameter is an OPTIONAL parameter of the request and response messages exchanged between the Client and the /authz-info endpoint at the RS (see {{Section 5.10.1. of RFC9200}}).

This parameter allows the Client and the RS to exchange information about an ECDH algorithm as well as about the authentication credentials and public keys to accordingly use for deriving Diffie-Hellman secrets. Its exact semantics and content are application specific.

In application profiles that build on {{RFC9594}}, this parameter is used to exchange information about the ECDH algorithm as well as about the authentication credentials and public keys to be used with it, in the groups indicated by the transferred access token as per its 'scope' claim (see {{Section 3.2 of RFC9594}}).

When used in the Token Transfer Request sent to the KDC (see {{Section 3.3 of RFC9594}}), the 'ecdh_info' parameter specifies the CBOR simple value `null` (0xf6). This is done to ask for information about the ECDH algorithm and about the authentication credentials used to compute static-static Diffie-Hellman shared secrets {{NIST-800-56A}}, in the groups that the Client has been authorized to join or interact with.

When used in the following Token Transfer Response from the KDC (see {{Section 3.3 of RFC9594}}), the 'ecdh_info' parameter is a CBOR array of one or more elements. The number of elements is at most the number of groups that the Client has been authorized to join or interact with. Each element contains information about ECDH parameters and about authentication credentials for one or more groups and is formatted as follows.

* The first element 'id' is a group name or a CBOR array of group names, which is associated with groups for which the next four elements apply. Each specified group name is a CBOR text string and is hereafter referred to as 'gname'.

* The second element 'ecdh_alg' is a CBOR integer or a text string that indicates the ECDH algorithm used in the groups identified by the 'gname' values.

  For application profiles of {{RFC9594}} that use the 'ecdh_info' parameter, it is REQUIRED to define specific values that 'ecdh_alg' can take, which are selected from the set of signing algorithms of the "COSE Algorithms" registry {{COSE.Algorithms}}.

* The third element 'ecdh_parameters' is a CBOR array that indicates the parameters of the ECDH algorithm used in the groups identified by the 'gname' values. Its content depends on the value of 'ecdh_alg'.

  For application profiles of {{RFC9594}} that use the 'ecdh_info' parameter, it is REQUIRED to define the possible values and structure for the elements of 'ecdh_parameters'.

* The fourth element 'ecdh_key_parameters' is a CBOR array that indicates the parameters of the key used with the ECDH algorithm in the groups identified by the 'gname' values. Its content depends on the value of 'ecdh_alg'.

  For application profiles of {{RFC9594}} that use the 'ecdh_info' parameter, it is REQUIRED to define the possible values and structure for the elements of 'ecdh_key_parameters'.

* The fifth element 'cred_fmt' either is a CBOR integer indicating the format of authentication credentials used in the groups identified by the 'gname' values or is the CBOR simple value `null` (0xf6), which indicates that the KDC does not act as a repository of authentication credentials for group members. Its acceptable integer values are taken from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}}, with some of those values also indicating the type of container to use for exchanging the authentication credentials with the KDC (e.g., a chain or bag of certificates).

  For application profiles of {{RFC9594}} that use the 'ecdh_info' parameter, it is REQUIRED to define specific values to use for 'cred_fmt', consistently with the acceptable formats of authentication credentials.

If 'ecdh_info' is included in the Token Transfer Request, the KDC SHOULD include the 'ecdh_info' parameter in the Token Transfer Response, as per the format defined above. Note that the field 'id' of each 'ecdh_info_entry' specifies the name, or array of group names to which that 'ecdh_info_entry' applies. As an exception, the KDC MAY omit the 'ecdh_info' parameter in the Token Transfer Response even if 'ecdh_info' is included in the Token Transfer Request, in case none of the groups that the Client is authorized to join uses an ECDH algorithm to derive Diffie-Hellman secrets.

The CDDL notation {{RFC8610}} of the 'ecdh_info' parameter is given below.

~~~~~~~~~~~ CDDL
ecdh_info = ecdh_info_req / ecdh_info_resp

ecdh_info_req = null                  ; in the Token Transfer
                                      ; Request to the KDC

ecdh_info_resp = [+ ecdh_info_entry]  ; in the Token Transfer
                                      ; Response from the KDC

ecdh_info_entry =
[
  id: gname / [+ gname],
  ecdh_alg: int / tstr,
  ecdh_parameters: [any],
  ecdh_key_parameters: [+ parameter: any],
  cred_fmt: int / null
]

gname = tstr
~~~~~~~~~~~

This format is consistent with every ECDH algorithm currently defined in {{RFC9053}}, i.e., with algorithms that have only the COSE key type as their COSE capability. {{sec-future-cose-algs}} of this document describes how the format of each 'ecdh_info_entry' can be generalized for possible future registered algorithms having a different set of COSE capabilities.

### 'kdc_dh_creds' Parameter {#gm-dh-info}

The 'kdc_dh_creds' parameter is an OPTIONAL parameter of the request and response messages exchanged between the Client and the /authz-info endpoint at the RS (see {{Section 5.10.1. of RFC9200}}).

This parameter allows the Client to request and retrieve the Diffie-Hellman authentication credentials of the RS, i.e., authentication credentials including a Diffie-Hellman public key of the RS.

In application profiles that build on {{RFC9594}}, this parameter is used to request and retrieve from the KDC its Diffie-Hellman authentication credentials to use, in the groups indicated by the transferred access token as per its 'scope' claim (see {{Section 3.2 of RFC9594}}).

When used in the Token Transfer Request sent to the KDC (see {{Section 3.3 of RFC9594}}), the 'kdc_dh_creds' parameter specifies the CBOR simple value `null` (0xf6). This is done to ask for the Diffie-Hellman authentication credentials that the KDC uses in the groups that the Client has been authorized to join or interact with.

When used in the following Token Transfer Response from the KDC (see {{Section 3.2 of RFC9594}}), the 'kdc_dh_creds' parameter is a CBOR array of one or more elements. The number of elements is at most the number of groups that the Client has been authorized to join or interact with. Each element contains information about the KDC's Diffie-Hellman authentication credentials for one or more groups and is formatted as follows.

* The first element 'id' is a group name or a CBOR array of group names, which is associated with groups for which the next two elements apply. Each specified group name is a CBOR text string and is hereafter referred to as 'gname'.

* The second element 'cred_fmt' is a CBOR integer indicating the format of the KDC's authentication credential used in the groups identified by the 'gname' values and specified by the following element 'cred'. Its acceptable integer values are taken from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}}, with some of those values also indicating the type of container to use for exchanging the authentication credentials with the KDC (e.g., a chain or bag of certificates).

  For application profiles of {{RFC9594}} that use the 'kdc_dh_creds' parameter, it is REQUIRED to define specific values to use for 'cred_fmt', consistently with the acceptable formats of the KDC's authentication credentials.

* The third element 'cred' is a CBOR byte string encoding the original binary representation of the Diffie-Hellman authentication credential that the KDC uses in the groups identified by the 'gname' values. The authentication credential complies with the format specified by the 'cred_fmt' element.

If 'kdc_dh_creds' is included in the Token Transfer Request, the KDC SHOULD include the 'kdc_dh_creds' parameter in the Token Transfer Response, as per the format defined above. Note that the field 'id' of each 'kdc_dh_creds_entry' specifies the name, or array of group names to which that 'kdc_dh_creds_entry' applies. As an exception, the KDC MAY omit the 'kdc_dh_creds' parameter in the Token Transfer Response even if 'kdc_dh_creds' is included in the Token Transfer Request, in case the KDC does not use a Diffie-Hellman authentication credential in any of the groups that the Client is authorized to join.

The CDDL notation {{RFC8610}} of the 'kdc_dh_creds' parameter is given below.

~~~~~~~~~~~ CDDL
kdc_dh_creds = kdc_dh_creds_req / kdc_dh_creds_resp

kdc_dh_creds_req = null                     ; in the Token Transfer
                                            ; Request to the KDC

kdc_dh_creds_resp = [+ kdc_dh_creds_entry]  ; in the Token Transfer
                                            ; Response from the KDC

kdc_dh_creds_entry =
[
  id: gname / [+ gname],
  cred_fmt: int,
  cred: bstr
]

gname = tstr
~~~~~~~~~~~

# Group Joining {#sec-joining-node-to-GM}

This section describes the interactions between the joining node and the Group Manager to join an OSCORE group. The message exchange between the joining node and the Group Manager consists of the messages defined in {{Section 4.3.1.1 of RFC9594}}. Note that what is defined in {{RFC9594}} applies, and only additions or modifications to that specification are defined in this document.

## Send the Join Request {#ssec-join-req-sending}

The joining node requests to join the OSCORE group by sending a Join Request message to the related group-membership resource at the Group Manager, as per {{Section 4.3.1.1 of RFC9594}}. Additionally to what is defined in {{Section 4.3.1 of RFC9594}}, the following applies.

* The 'scope' parameter MUST be included. Its value encodes one scope entry with the format defined in {{sec-format-scope}}, indicating the group name and the role(s) that the joining node wants to take in the group.

   The 'scope' parameter MUST NOT specify any of the following sets of roles: ("requester", "monitor") and ("responder", "monitor"). Future specifications that define a new role for members of OSCORE groups MUST define possible sets of roles (including the new role and existing roles) that are not acceptable to specify in the 'scope' parameter of a Join Request.

* The 'get_creds' parameter is present only if the joining node wants to retrieve the authentication credentials of the group members from the Group Manager during the joining process (see {{sec-public-keys-of-joining-nodes}}). Otherwise, this parameter MUST NOT be present.

   If this parameter is present and its value is not the CBOR simple value `null` (0xf6), each element of the inner CBOR array 'role_filter' is encoded as a CBOR unsigned integer, with the same value of a permission set ("Tperm") indicating that role or combination of roles in a scope entry, as defined in {{sec-format-scope}}.

* The 'cnonce' parameter contains a dedicated nonce N_C generated by the joining node. For the N\_C value, it is RECOMMENDED to use an 8-byte long random nonce.

* If the joining node intends to join the group exclusively as a monitor, then the 'client_cred' parameter and the 'client_cred_verify' parameter MUST be omitted.

* If the joining node is currently a group member and intends to use the same authentication credential that it is currently using in the group, then the 'client_cred' parameter and the 'client_cred_verify' parameter MAY be omitted.

* If the 'client_cred_verify' parameter is present, then the proof-of-possession (PoP) evidence included therein is computed as defined below (REQ14).

  * Specifically in the case where the joining node is not a current member of the group, the Group Manager might already have achieved proof-of-possession of the joining node's private key associated with the authentication credential AUTH_CRED_C that the joining node intends to use in the group.

    For example, that could have happened upon completing the establishment of the secure communication association that is used to protect the Join Request, if the joining node used AUTH_CRED_C to authenticate itself with the Group Manager.

    Under these circumstances, the joining node MAY specify an empty PoP evidence, i.e., it sets the value of the 'client_cred_verify' parameter to the empty CBOR byte string (0x40).

  * If the conditions above do not hold or the joining node prefers to compute a non-empty PoP evidence, then the joining node  proceeds as follows. In either case, the N_S used to build the PoP input is as defined in {{sssec-challenge-value}}.

    - If the group is not a pairwise-only group, the PoP evidence MUST be a signature. The joining node computes the signature by using the same private key and signature algorithm that it intends to use for signing messages in the OSCORE group.

    - If the group is a pairwise-only group, the PoP evidence MUST be a MAC computed as follows, by using the HKDF Algorithm HKDF SHA-256, which consists of composing the HKDF-Extract and HKDF-Expand steps {{RFC5869}}.

      MAC = HKDF(salt, IKM, info, L)

      The input parameters of HKDF are as follows:

      * salt takes as value the empty byte string.

      * IKM is computed as a cofactor Diffie-Hellman shared secret, see Section 5.7.1.2 of {{NIST-800-56A}}, using the ECDH algorithm used in the OSCORE group. The joining node uses its own Diffie-Hellman private key and the Diffie-Hellman public key of the Group Manager. For X25519 and X448, the procedure is described in {{Section 5 of RFC7748}}.

      * info takes as value the PoP input.

      * L is equal to 8, i.e., the size of the MAC, in bytes.

### Value of the N\_S Challenge {#sssec-challenge-value}

The value of the N\_S challenge is determined as follows.

1. If the joining node has provided the access token to the Group Manager by means of a Token Transfer Request to the /authz-info endpoint as in {{ssec-token-post}}, then N\_S takes the same value of the most recent 'kdcchallenge' parameter received by the joining node from the Group Manager. This can be either the one specified in the Token Transfer Response, or the one possibly specified in a 4.00 (Bad Request) error response to a following Join Request (see {{ssec-join-req-processing}}).

2. If the provisioning of the access token to the Group Manager has relied on the DTLS profile of ACE {{RFC9202}} and the access token was specified in the "psk_identity" field of the ClientKeyExchange message when using DTLS 1.2 {{RFC6347}}, then N\_S is an exporter value computed as defined in {{Section 4 of RFC5705}} (REQ15).

   Specifically, N\_S is exported from the DTLS session between the joining node and the Group Manager, using an empty context value (i.e., a context value of zero-length), 32 as length value in bytes, and the exporter label "EXPORTER-ACE-Pop-Input-coap-group-oscore-app" defined in {{ssec-iana-tls-esporter-label-registry}} of this document.

   The same as above holds if TLS 1.2 {{RFC5246}} was used instead of DTLS 1.2, as per {{RFC9430}}.

3. If the provisioning of the access token to the Group Manager has relied on the DTLS profile of ACE {{RFC9202}} and the access token was specified in the "identity" field of a PskIdentity within the PreSharedKeyExtension of the ClientHello message when using DTLS 1.3 {{RFC9147}}, then N\_S is an exporter value computed as defined in {{Section 7.5 of RFC8446}} (REQ15).

   Specifically, N\_S is exported from the DTLS session between the joining node and the Group Manager, using an empty 'context_value' (i.e., a 'context_value' of zero length), 32 as 'key_length' in bytes, and the exporter label "EXPORTER-ACE-Pop-Input-coap-group-oscore-app" defined in {{ssec-iana-tls-esporter-label-registry}} of this document.

   The same as above holds if TLS 1.3 {{RFC8446}} was used instead of DTLS 1.3, as per {{RFC9430}}.

It is up to applications to define how N_S is computed in further alternative settings.

{{ssec-security-considerations-reusage-nonces}} provides security considerations on the reusage of the N_S challenge.

## Receive the Join Request {#ssec-join-req-processing}

The Group Manager processes the Join Request as defined in {{Section 4.3.1 of RFC9594}}, with the following additions. Note that the Group Manager can determine whether the joining node is a current group member, e.g., based on the ongoing secure communication association that is used to protect the Join Request.

In case the joining node is going to join the group exclusively as monitor, then the Group Manager silently ignores the parameters 'client_cred' and 'client_cred_verify', if present.

In case the joining node is not going to join the group exclusively as monitor, it is a current member of the group, and the 'client_cred_verify' parameter is not present, then the following applies:

* If the 'client_cred' parameter is present, the Group Manager verifies that it is already storing the authentication credential specified therein, as associated with the joining node in the group. If the verification fails, the Group Manager MUST reply with a 4.00 (Bad Request) error response.

* If case the 'client_cred' parameter is not present, the Group Manager verifies that it is already storing an authentication credential, as associated with the joining node in the group. If the verification fails, the Group Manager MUST reply with a 4.00 (Bad Request) error response.

In case the joining node is not going to join the group exclusively as monitor and the 'client_cred_verify' parameter specifies the empty CBOR byte string (0x40), the Group Manager checks whether it has already achieved proof-of-possession of the joining node's private key associated with the authentication credential that is specified in the 'client_cred' parameter. If such verification fails, then the Group Manager MUST reply with a 4.00 (Bad Request) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 3 ("Invalid proof-of-possession evidence"). After receiving that response, the client MUST NOT specify an empty PoP evidence in the 'client_cred_verify' parameter of a follow-up Join Request for joining the same group.

In case the joining node is not going to join the group exclusively as monitor and the 'client_cred_verify' parameter specifies a value different from the empty CBOR byte string (0x40), then the Group Manager verifies the PoP evidence therein as follows:

* As PoP input, the Group Manager uses the value of the 'scope' parameter from the Join Request as a CBOR byte string, concatenated with N_S encoded as a CBOR byte string, concatenated with N_C encoded as a CBOR byte string. The value of N_S is determined as described in {{sssec-challenge-value}}, while N_C is the nonce provided in the 'cnonce' parameter of the Join Request.

* As public key of the joining node, the Group Manager uses either the one included in the authentication credential retrieved from the 'client_cred' parameter of the Join Request, or the one from the already stored authentication credential as acquired from previous interactions with the joining node (see above).

* If the group is not a pairwise-only group, the PoP evidence is a signature. The Group Manager verifies it by using the public key of the joining node, as well as the signature algorithm used in the OSCORE group and possible corresponding parameters.

* If the group is a pairwise-only group, the PoP evidence is a MAC. The Group Manager recomputes the MAC through the same process that is taken by the joining node when preparing the value of the 'client_cred_verify' parameter for the Join Request (see {{ssec-join-req-sending}}), with the difference that the Group Manager uses its own Diffie-Hellman private key and the Diffie-Hellman public key of the joining node. The verification succeeds if and only if the recomputed MAC is equal to the MAC conveyed as PoP evidence in the Join Request.

The Group Manager MUST reply with a 5.03 (Service Unavailable) error response in the following cases:

* There are currently no OSCORE Sender IDs available to assign in the OSCORE group and, at the same time, the joining node is not going to join the group exclusively as monitor. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 4 ("No available individual keying material").

* The OSCORE group that the joining node has been trying to join is currently inactive (see {{ssec-resource-active}}). The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 9 ("Group currently not active").

The Group Manager MUST reply with a 4.00 (Bad Request) error response in the following cases:

* The 'scope' parameter is not present in the Join Request, or it is present and specifies any of the following sets of roles: ("requester", "monitor") and ("responder", "monitor").

* The joining node is not going to join the group exclusively as monitor, and any of the following holds:

  - The joining node is not a current member of the group, and the 'client_cred' parameter and the 'client_cred_verify' parameter are not both present in the Join Request.

  - The 'client_cred' parameter is present in the Join Request and its value is not an eligible authentication credential (e.g., it is not of the format accepted in the group).

In order to prevent the acceptance of Ed25519 and Ed448 public keys that cannot be successfully converted to Montgomery coordinates, and thus cannot be used for the derivation of pairwise keys (see {{Section 2.5.1 of I-D.ietf-core-oscore-groupcomm}}), the Group Manager MAY reply with a 4.00 (Bad Request) error response in case all the following conditions hold:

* The OSCORE group uses the pairwise mode of Group OSCORE.

* The OSCORE group uses EdDSA public keys {{RFC8032}}.

* The authentication credential of the joining node from the 'client_cred' parameter includes a public key which:

   - Is for the elliptic curve Ed25519 and has its Y coordinate equal to -1 or 1 (mod p), with p = (2<sup>255</sup> - 19), see {{Section 4.1 of RFC7748}}; or

   - Is for the elliptic curve Ed448 and has its Y coordinate equal to -1 or 1 (mod p), with p =  (2<sup>448</sup> - 2<sup>224</sup> - 1), see {{Section 4.2 of RFC7748}}.

Unless it is already intended to use Content-Format "application/concise-problem-details+cbor", a 4.00 (Bad Request) error response from the Group Manager to the joining node MUST have Content-Format "application/ace-groupcomm+cbor". In such a case, the response payload is a CBOR map formatted as follows:

* If the group uses (also) the group mode of Group OSCORE, then the CBOR map MUST contain the 'sign_info' parameter, whose CBOR label is defined in {{Section 8 of RFC9594}}. This parameter has the same format of 'sign_info_res' defined in {{Section 3.3.1 of RFC9594}} and includes a single element 'sign_info_entry', which pertains to the OSCORE group that the joining node has tried to join with the Join Request.

* If the group uses (also) the pairwise mode of Group OSCORE, then the CBOR map MUST contain the 'ecdh_info' parameter, whose CBOR label is defined in {{ssec-iana-ace-groupcomm-parameters-registry}}. This parameter has the same format of 'ecdh_info_res' defined in {{ecdh-info}} and includes a single element 'ecdh_info_entry', which pertains to the OSCORE group that the joining node has tried to join with the Join Request.

* If the group is a pairwise-only group, the CBOR map MUST contain the 'kdc_dh_creds' parameter, whose CBOR label is defined in {{ssec-iana-ace-groupcomm-parameters-registry}}. This parameter has the same format of 'kdc_dh_creds_res' defined in {{gm-dh-info}} and includes a single element 'kdc_dh_creds_entry', which pertains to the OSCORE group that the joining node has tried to join with the Join Request.

* The CBOR map MAY include the 'kdcchallenge' parameter, whose CBOR label is defined in {{Section 8 of RFC9594}}. If present, this parameter is a CBOR byte string, which encodes a newly generated 'kdcchallenge' value that the Client can use when preparing a new Join Request (see {{ssec-join-req-sending}}). In such a case the Group Manager MUST store the newly generated value as the 'kdcchallenge' value associated with the joining node, thus replacing the currently stored value (if any).

### Follow-up to a 4.00 (Bad Request) Error Response

When receiving a 4.00 (Bad Request) error response, the joining node MAY send a new Join Request to the Group Manager. In such a case:

* The 'cnonce' parameter MUST include a new dedicated nonce N\_C generated by the joining node.

* In case the joining node is going to join the group exclusively as monitor, then the following applies:

  - If present, the 'client_cred' parameter MUST include an authentication credential in the format indicated by the Group Manager. Also, the authentication credential as well as the included public key MUST be compatible with the signature or ECDH algorithm, and with possible associated parameters.

  - If present, the 'client_cred_verify' parameter MUST include a PoP evidence computed as described in {{ssec-join-req-sending}}. The private key to use is the one associated with the authentication credential specified in the current 'client_cred' parameter, with the signature or ECDH algorithm, and with possible associated parameters indicated by the Group Manager. If the error response from the Group Manager includes the 'kdcchallenge' parameter, the joining node MUST use its content as new N\_S challenge to compute the PoP evidence.

## Send the Join Response {#ssec-join-resp}

If the processing of the Join Request described in {{ssec-join-req-processing}} is successful, the Group Manager updates the group membership by registering the joining node NODENAME as a new member of the OSCORE group GROUPNAME, as described in {{Section 4.3.1 of RFC9594}}.

If the joining node has not taken exclusively the role of monitor, the Group Manager performs also the following actions.

* The Group Manager selects an available OSCORE Sender ID in the OSCORE group, and exclusively assigns it to the joining node. The Group Manager MUST NOT assign an OSCORE Sender ID to the joining node if this joins the group exclusively with the role of monitor, according to what is specified in the access token (see {{ssec-auth-resp}}).

   Consistently with {{Section 12.2.1.2 of I-D.ietf-core-oscore-groupcomm}}, the Group Manager MUST assign an OSCORE Sender ID that has not been used in the OSCORE group since the latest time when the current Gid value was assigned to the group. The maximum length of a Sender ID in bytes is determined as defined in {{Section 2.2 of I-D.ietf-core-oscore-groupcomm}}.

   If the joining node is recognized as a current group member, e.g., through the ongoing secure communication association that is used to protect the Join Request, then the following also applies:

     - The Group Manager MUST assign a new OSCORE Sender ID different from the one currently used by the joining node in the OSCORE group.

     - The Group Manager MUST add the old, relinquished OSCORE Sender ID of the joining node to the set of stale Sender IDs associated with the current version of the group keying material for the group (see {{sssec-stale-sender-ids}}).

* The Group Manager stores the association between: i) the authentication credential of the joining node; and ii) the Group Identifier (Gid), i.e., the OSCORE ID Context associated with the OSCORE group, together with the OSCORE Sender ID assigned to the joining node in the group. The Group Manager MUST keep this association updated over time.

Then, the Group Manager replies to the joining node, providing the updated security parameters and keying material necessary to participate in the group communication. This success Join Response is formatted as defined in {{Section 4.3.1 of RFC9594}}, with the following additions:

* The 'gkty' parameter identifies a key of type "Group_OSCORE_Input_Material object", which is defined in {{ssec-iana-groupcomm-keys-registry}} of this document.

* The 'key' parameter includes what the joining node needs in order to set up the Group OSCORE Security Context as per {{Section 2 of I-D.ietf-core-oscore-groupcomm}}.

   This parameter has as value a Group_OSCORE_Input_Material object, which is defined in this document and extends the OSCORE_Input_Material object encoded in CBOR as defined in {{Section 3.2.1 of RFC9203}}. In particular, it contains the additional parameters 'group_senderId', 'cred_fmt', 'gp_enc_alg', 'sign_alg', 'sign_params', 'ecdh_alg', and 'ecdh_params', which are defined in {{ssec-iana-security-context-parameter-registry}} of this document.

   More specifically, the 'key' parameter is composed as follows.

   * The 'hkdf' parameter, if present, specifies the HKDF Algorithm that is used in the OSCORE group. The HKDF Algorithm is specified by the HMAC Algorithm value. This parameter MAY be omitted, if the HKDF Algorithm used in the group is HKDF SHA-256. Otherwise, this parameter MUST be present.

   * The 'salt' parameter, if present, has as value the OSCORE Master Salt that is used in the OSCORE group. This parameter MAY be omitted, if the Master Salt used in the group is the empty byte string. Otherwise, this parameter MUST be present.

   * The 'ms' parameter has as value the OSCORE Master Secret that is used in the OSCORE group. This parameter MUST be present.

   * The 'contextId' parameter has as value the Group Identifier (Gid), i.e., the OSCORE ID Context of the OSCORE group. This parameter MUST be present.

   * The 'group_senderId' parameter has as value the OSCORE Sender ID that the Group Manager has assigned to the joining node in the OSCORE group, as described above. This parameter MUST be present if and only if the node does not join the OSCORE group exclusively with the role of monitor, according to what is specified in the access token (see {{ssec-auth-resp}}).

   * The 'cred_fmt' parameter specifies the Authentication Credential Format used in the OSCORE group (see {{Section 2 of I-D.ietf-core-oscore-groupcomm}}). This parameter MUST be present and it takes value from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}} (REQ6), with some of those values also indicating the type of container to use for exchanging the authentication credentials with the Group Manager (e.g., a chain or bag of certificates). Consistently with {{Section 2.4 of I-D.ietf-core-oscore-groupcomm}}, acceptable values denote a format that MUST explicitly provide the public key as well as a comprehensive set of information related to the public key algorithm. This information includes, e.g., the used elliptic curve.

      At the time of writing this specification, acceptable formats of authentication credentials are CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC5280}}, and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}. Further formats may be available in the future, and would be acceptable to use as long as they comply with the criteria defined above.

      \[ As to C509 certificates, the COSE Header Parameters 'c5b' and 'c5c' are under pending registration requested by draft-ietf-cose-cbor-encoded-cert. \]

   The 'key' parameter MUST also include the following parameters, if and only if the OSCORE group is not a pairwise-only group.

   * The 'gp_enc_alg' parameter, specifying the Group Encryption Algorithm that is used in the OSCORE group to encrypt messages protected with the group mode. This parameter takes values from the "Value" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

   * The 'sign_alg' parameter, specifying the Signature Algorithm that is used in the OSCORE group to sign messages protected with the group mode. This parameter takes values from the "Value" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

   * The 'sign_params' parameter, specifying the parameters of the Signature Algorithm. This parameter is a CBOR array, which includes the following two elements:

      - 'sign_alg_capab': a CBOR array, with the same format and value of the COSE capabilities array for the Signature Algorithm indicated in 'sign_alg', as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

      - 'sign_key_type_capab': a CBOR array, with the same format and value of the COSE capabilities array for the COSE key type of the keys used with the Signature Algorithm indicated in 'sign_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}}.

   The 'key' parameter MUST also include the following parameters, if and only if the OSCORE group is not a signature-only group.

   * The 'alg' parameter, specifying the AEAD Algorithm used in the OSCORE group to encrypt messages protected with the pairwise mode.

   * The 'ecdh_alg' parameter, specifying the Pairwise Key Agreement Algorithm used in the OSCORE group to derive the pairwise keys for the pairwise mode. This parameter takes values from the "Value" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

   * The 'ecdh_params' parameter, specifying the parameters of the Pairwise Key Agreement Algorithm. This parameter is a CBOR array, which includes the following two elements:

      - 'ecdh_alg_capab': a CBOR array, with the same format and value of the COSE capabilities array for the algorithm indicated in 'ecdh_alg', as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

      - 'ecdh_key_type_capab': a CBOR array, with the same format and value of the COSE capabilities array for the COSE key type of the keys used with the algorithm indicated in 'ecdh_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}}.

   The format of 'key' defined above is consistent with every signature algorithm and ECDH algorithm currently considered in {{RFC9053}}, i.e., with algorithms that have only the COSE key type as their COSE capability. {{sec-future-cose-algs-key}} of this document describes how the format of the 'key' parameter can be generalized for possible future registered algorithms having a different set of COSE capabilities.

Furthermore, the following applies.

* The 'exi' parameter MUST be present.

* The 'ace_groupcomm_profile' parameter MUST be present and has value coap_group_oscore_app (PROFILE_TBD), which is registered in {{ssec-iana-groupcomm-profile-registry}} of this document.

* The 'creds' parameter, if present, specifies the authentication credentials requested by the joining node by means of the 'get_creds' parameter that was specified in the Join Request.

  If the joining node has asked for the authentication credentials of all the group members, i.e., the 'get_creds' parameter in the Join Request had value the CBOR Simple Value `null` (0xf6), then the Group Manager provides only the authentication credentials of the group members that are relevant to the joining node. That is, in such a case, the 'creds' parameter specifies only: i) the authentication credentials of the responders currently in the OSCORE group, in case the joining node is configured (also) as requester; and ii) the authentication credentials of the requesters currently in the OSCORE group, in case the joining node is configured (also) as responder or monitor.

* The 'peer_identifiers' parameter, if present, specifies the OSCORE Sender ID of each group member whose authentication credential is specified in the 'creds' parameter. That is, a group member's Sender ID is used as identifier for that group member (REQ25).

* The 'group_policies' parameter SHOULD be present, and SHOULD include the following elements:

  * "Key Update Check Interval" (see {{Section 4.3.1 of RFC9594}}), with default value 3600;

  * "Expiration Delta" (see {{Section 4.3.1 of RFC9594}}), with default value 0.

* The 'kdc_cred' parameter MUST be present, specifying the Group Manager's authentication credential in its original binary representation (REQ8). The Group Manager's authentication credential MUST be in the format used in the OSCORE group. Also, the authentication credential as well as the included public key MUST be compatible with the signature or ECDH algorithm, and with possible associated parameters used in the OSCORE group.

* The 'kdc_nonce' parameter MUST be present, specifying the dedicated nonce N_KDC generated by the Group Manager. For N_KDC, it is RECOMMENDED to use an 8-byte long random nonce.

* The 'kdc_cred_verify' parameter MUST be present, specifying the proof-of-possession (PoP) evidence computed by the Group Manager. The PoP evidence is computed as defined below (REQ21).

   - If the group is not a pairwise-only group, then the PoP evidence MUST be a signature. The Group Manager computes the signature by using the signature algorithm used in the OSCORE group, as well as its own private key associated with the authentication credential specified in the 'kdc_cred' parameter.

   - If the group is a pairwise-only group, then the PoP evidence MUST be a MAC computed as follows, by using the HKDF Algorithm HKDF SHA-256, which consists of composing the HKDF-Extract and HKDF-Expand steps {{RFC5869}}.

      MAC = HKDF(salt, IKM, info, L)

      The input parameters of HKDF are as follows.

        * salt takes as value the empty byte string.

        * IKM is computed as a cofactor Diffie-Hellman shared secret, see Section 5.7.1.2 of {{NIST-800-56A}}, using the ECDH algorithm used in the OSCORE group. The Group Manager uses its own Diffie-Hellman private key and the Diffie-Hellman public key of the joining node. For X25519 and X448, the procedure is described in {{Section 5 of RFC7748}}.

        * info takes as value the PoP input.

        * L is equal to 8, i.e., the size of the MAC, in bytes.

* The 'group_rekeying' parameter MAY be omitted, if the Group Manager uses the "Point-to-Point" group rekeying scheme registered in {{Section 11.13 of RFC9594}} as rekeying scheme in the OSCORE group (OPT9). Its detailed use for this profile is defined in {{sec-group-rekeying-process}} of this document. In any other case, the 'group_rekeying' parameter MUST be included.

As a last action, if the Group Manager reassigns Gid values during the group's lifetime (see {{Sections 12.2.1.1 and E of I-D.ietf-core-oscore-groupcomm}}), then the Group Manager MUST store the Gid specified in the 'contextId' parameter of the 'key' parameter, as the Birth Gid of the joining node in the joined group (see {{Section E of I-D.ietf-core-oscore-groupcomm}}). This applies also in case the joining node is in fact re-joining the group; in such a case, the newly determined Birth Gid overwrites the one currently stored.

## Receive the Join Response {#ssec-join-resp-processing}

Upon receiving the Join Response, the joining node retrieves the Group Manager's authentication credential from the 'kdc_cred' parameter. The joining node MUST verify the proof-of-possession (PoP) evidence specified in the 'kdc_cred_verify' parameter of the Join Response as defined below (REQ21).

* If the group is not a pairwise-only group, the PoP evidence is a signature. The joining node verifies it by using the public key of the Group Manager from the received authentication credential, as well as the signature algorithm used in the OSCORE group and possible corresponding parameters.

* If the group is a pairwise-only group, the PoP evidence is a MAC. The joining node recomputes the MAC through the same process that is taken by the Group Manager when computing the value of the 'kdc_cred_verify' parameter (see {{ssec-join-resp}}), with the difference that the joining node uses its own Diffie-Hellman private key and the Diffie-Hellman public key of the Group Manager from the received authentication credential. The verification succeeds if and only if the recomputed MAC is equal to the MAC conveyed as PoP evidence in the Join Response.

In case of failed verification of the PoP evidence, the joining node MUST stop processing the Join Response and MAY send a new Join Request to the Group Manager (see {{ssec-join-req-sending}}).

In case of successful verification of the PoP evidence, the joining node uses the information received in the Join Response to set up the Group OSCORE Security Context, as described in {{Section 2 of I-D.ietf-core-oscore-groupcomm}}. In particular, the following applies.

If the following parameters were not included in the 'key' parameter of the Join Response, then the joining node performs the following actions.

* Absent the 'gp_enc_alg' parameter, the parameter Group Encryption Algorithm in the Common Context of the Group OSCORE Security Context is not set.

* Absent the 'sign_alg' parameter, the parameter Signature Algorithm in the Common Context of the Group OSCORE Security Context is not set.

* Absent the 'alg' parameter, the parameter AEAD Algorithm in the Security Context of the Group OSCORE Security Context is not set.

* Absent the 'ecdh_alg' parameter, the parameter Pairwise Key Agreement Algorithm in the Common Context of the Group OSCORE Security Context is not set.

If the following parameters were not included in the 'key' parameter of the Join Response, then the joining node considers the default values specified below, consistently with {{Section 3.2 of RFC8613}}.

* Absent the 'hkdf' parameter, the joining node considers HKDF SHA-256 as the HKDF Algorithm to use in the OSCORE group.

* Absent the 'salt' parameter, the joining node considers the empty byte string as the Master Salt to use in the OSCORE group.

* Absent the 'group_rekeying' parameter, the joining node considers the "Point-to-Point" group rekeying scheme registered in {{Section 11.13 of RFC9594}} as the rekeying scheme used in the OSCORE group (OPT9). The detailed use of that rekeying scheme for this profile is defined in {{sec-group-rekeying-process}} of this document.

In addition, the joining node maintains an association between each authentication credential retrieved from the 'creds' parameter and the role(s) that the corresponding group member has in the OSCORE group.

From then on, the joining node can exchange group messages secured with Group OSCORE as described in {{I-D.ietf-core-oscore-groupcomm}}. When doing so:

* The joining node MUST NOT process an incoming request message, if protected by a group member whose authentication credential is not associated with the role "Requester".

* The joining node MUST NOT process an incoming response message, if protected by a group member whose authentication credential is not associated with the role "Responder".

* The joining node MUST NOT use the group mode of Group OSCORE to process messages in the group, if the Join Response did not include both the 'gp_enc_alg' parameter and the 'sign_alg' parameter.

* The joining node MUST NOT use the pairwise mode of Group OSCORE to process messages in the group, if the Join Response did not include both the 'alg' parameter and the 'ecdh_alg' parameter.

If the application requires backward security, the Group Manager MUST generate updated security parameters and group keying material, and provide it to the current group members, upon the new node's joining (see {{sec-group-rekeying-process}}). In such a case, the joining node is not able to access secure communication in the OSCORE group that occurred prior to its joining.

# Overview of the Group Rekeying Process {#ssec-overview-group-rekeying-process}

In a number of cases, the Group Manager has to generate new keying material and distribute it to the group (rekeying), as also discussed in {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}.

To this end the Group Manager MUST support the Group Rekeying Process described in {{sec-group-rekeying-process}} of this document, as an instance of the "Point-to-Point" rekeying scheme defined in {{Section 6.1 of RFC9594}} and registered in {{Section 11.13 of RFC9594}}. Future documents may define the use of alternative group rekeying schemes for this application profile, together with the corresponding rekeying message formats. The resulting group rekeying process MUST comply with the functional steps defined in {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}.

Upon generating the new group keying material and before starting its distribution, the Group Manager MUST increment the version number of the group keying material. When rekeying a group, the Group Manager MUST preserve the current value of the OSCORE Sender ID of each member in that group.

The data distributed to a group through a rekeying MUST include:

* The new version number of the group keying material for the group.

* A new Group Identifier (Gid) for the group as introduced in {{RFC9594}}, which is used as ID Context parameter of the Group OSCORE Common Security Context of that group (see {{Section 2 of I-D.ietf-core-oscore-groupcomm}}).

  Note that the Gid differs from the group name also introduced in {{RFC9594}}, which is a plain, stable, and invariant identifier, with no cryptographic relevance and meaning.

* A new value for the Master Secret parameter of the Group OSCORE Common Security Context of that group (see {{Section 2 of I-D.ietf-core-oscore-groupcomm}}).

* A set of stale Sender IDs, which allows each rekeyed node to purge authentication credentials and Recipient Contexts used in the group and associated with those Sender IDs. This in turn allows every group member to rely on stored authentication credentials, in order to confidently assert the group membership of other sender nodes, when receiving protected messages in the group (see {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}). More details on the maintenance of stale Sender IDs are provided in {{sssec-stale-sender-ids}}.

The data distributed through a group rekeying MAY also include a new value for the Master Salt parameter of the Group OSCORE Common Security Context of that group.

The Group Manager MUST rekey the group in the following cases.

* The application requires backward security - In this case, the group is rekeyed when a node joins the group as a new member. Therefore, a joining node cannot access communications in the group prior to its joining.

* One or more nodes leave the group - That is, the group is rekeyed when one or more current members spontaneously request to leave the group (see {{sec-leave-req}}), or when the Group Manager forcibly evicts them from the group, e.g., due to expired or revoked authorization (see {{sec-leaving}}). Therefore, a leaving node cannot access communications in the group after its leaving, thus ensuring forward security in the group.

  Due to the set of stale Sender IDs distributed through the rekeying, this ensures that a node storing the latest group keying material does not store the authentication credentials of former group members (see {{Sections 12.2 and 13.1 of I-D.ietf-core-oscore-groupcomm}}).

When the expiration time for the group keying material approaches or has passed, the Group Manager may want to extend the secure group operation, as considered appropriate. If the Group Manager does so, the Group Manager MUST rekey the group.

The Group Manager MAY rekey the group for other reasons, e.g., according to an application-specific rekeying period or scheduling.

## Stale OSCORE Sender IDs {#sssec-stale-sender-ids}

For each OSCORE group, the Group Manager MUST maintain N > 1 sets of "stale" OSCORE Sender IDs. It is up to the application to specify the value of N, possibly on a per-group basis.

Each set is uniquely associated with one version of the group keying material, and includes the OSCORE Sender IDs that have become "stale" in the OSCORE group under that version of the group keying material.

In the following cases, the Group Manager MUST add an element to the set X associated with the current version of the group keying material.

* When a current group member obtains a new Sender ID, its old Sender ID is added to X. This happens when the Group Manager assigns a new Sender ID upon request from the group member (see {{sec-new-key}}), or in case the group member re-joins the group (see {{ssec-join-req-sending}} and {{ssec-join-resp}}), thus also obtaining a new Sender ID.

* When a current group member leaves the group, its current Sender ID is added to X. This happens when a group member requests to leave the group (see {{sec-leave-req}}) or is forcibly evicted from the group (see {{sec-leaving}}).

The value of N can change during the lifetime of the group. If the new value N' is smaller than N, then the Group Manager MUST preserve the sets associated with the (up to) N' most recent versions of the group keying material.

When performing a group rekeying (see {{sec-group-rekeying-process}}) for switching from an old version V of the group keying material to a new version V' = (V + 1), the Group Manager MUST perform the following actions.

* Before creating the new group keying material with version V', if the number of sets of stale Sender IDs for the group is equal to N, then the Group Manager deletes the oldest set.

* The Group Manager rekeys the group. This includes also distributing the set of stale Sender IDs associated with the version V of the group keying material (see {{ssec-overview-group-rekeying-process}}).

* After completing the group rekeying, the Group Manager creates an empty set of stale Sender IDs, as associated with the version V' of the group keying material.

# Interface at the Group Manager {#sec-interface-GM}

The Group Manager provides the interface defined in {{Section 4.1 of RFC9594}}, with the following additions:

* The new FETCH handler is defined for the sub-resource /ace-group/GROUPNAME/kdc-cred (see {{sec-gm-pub-key-fetch}} of this document).

* Three new sub-resources are defined (see {{ssec-resource-active}}, {{ssec-resource-verif-data}}, and {{ssec-resource-stale-sids}} of this document).

{{ssec-admitted-methods}} provides a summary of the CoAP methods admitted to access different resources at the Group Manager, for nodes with different roles in the group or as non members (REQ11).

The GROUPNAME segment of the URI path MUST match with the group name specified in the scope entry of the scope in the access token (i.e., 'gname' in {{Section 3.1 of RFC9594}}) (REQ7).

The Resource Type (rt=) Link Target Attribute value "core.osc.gm" is registered in {{iana-rt}} (REQ10), and can be used to describe group-membership resources and its sub-resources at a Group Manager, e.g., by using a CoRE link-format document {{RFC6690}}.

Applications can use this common resource type to discover links to group-membership resources for joining OSCORE groups, e.g., by using the approach described in {{I-D.tiloca-core-oscore-discovery}}.

## /ace-group/GROUPNAME/kdc-cred {#sec-gm-pub-key-fetch}

In addition to what is defined in {{Section 4.5 of RFC9594}}, this resource also implements a FETCH handler.

### FETCH Handler {#kdc-cred-fetch}

The handler expects a FETCH request, whose payload is a CBOR map including a nonce N_C (see {{sec-gm-pub-key-signature-verifier}}).

In addition to what is defined in {{Section 4.1.2 of RFC9594}}, the Group Manager performs the following checks.

In case the requesting Client is a current group member, the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 8 ("Operation permitted only to signature verifiers").

In case GROUPNAME denotes a pairwise-only group, the Group Manager MUST reply with a 4.00 (Bad Request) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 7 ("Signatures not used in the group").

If all verifications succeed, the handler replies with a 2.05 (Content) response, specifying the authentication credential of the Group Manager together with a proof-of-possession (PoP) evidence. The payload of the response is formatted as defined in {{sec-gm-pub-key-signature-verifier}}.

## /ace-group/GROUPNAME/active {#ssec-resource-active}

This resource implements a GET handler.

### GET Handler {#active-get}

The handler expects a GET request.

In addition to what is defined in {{Section 4.1.2 of RFC9594}}, the handler verifies that the requesting Client is a current member of the group. If the verification fails, the KDC MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 0 ("Operation permitted only to group members").

If all verifications succeed, the handler replies with a 2.05 (Content) response, specifying the current status of the group, i.e., active or inactive. The payload of the response is formatted as defined in {{sec-status}}.

The method to set the current group status is out of the scope of this document, and is defined for the administrator interface of the Group Manager specified in {{I-D.ietf-ace-oscore-gm-admin}}.

## /ace-group/GROUPNAME/verif-data {#ssec-resource-verif-data}

This resource implements a GET handler.

### GET Handler {#verif-data-get}

The handler expects a GET request.

In addition to what is defined in {{Section 4.1.2 of RFC9594}}, the handler performs the following actions.

In case the requesting Client is a current group member, the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 8 ("Operation permitted only to signature verifiers").

In case GROUPNAME denotes a pairwise-only group, the Group Manager MUST reply with a 4.00 (Bad Request) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 7 ("Signatures not used in the group").

If all verifications succeed, the handler replies with a 2.05 (Content) response, specifying data that allow also an external signature verifier to verify signatures of messages protected with the group mode and sent to the group (see {{Sections 7.5 and 12.3 of I-D.ietf-core-oscore-groupcomm}}). The response MUST have Content-Format set to "application/ace-groupcomm+cbor". The payload of the response is a CBOR map, which is formatted as defined in {{sec-verif-data}}.

## /ace-group/GROUPNAME/stale-sids {#ssec-resource-stale-sids}

This resource implements a FETCH handler.

### FETCH Handler {#stale-sids-fetch}

The handler expects a FETCH request, whose payload specifies a version number of the group keying material, encoded as an unsigned CBOR integer (see {{sec-retrieve-stale-sids}}).

In addition to what is defined in {{Section 4.1.2 of RFC9594}}, the handler verifies that the requesting Client is a current member of the group. If the verification fails, the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 0 ("Operation permitted only to group members").

If all verifications succeed, the handler replies with a 2.05 (Content) response, specifying data that allow the requesting Client to delete the Recipient Contexts and authentication credentials associated with former members of the group (see {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}. The payload of the response is formatted as defined in {{sec-retrieve-stale-sids}}.

## Admitted Methods {#ssec-admitted-methods}

{{tab-methods}} summarizes the CoAP methods admitted to access different resources at the Group Manager, for (non-)members of a group with group name GROUPNAME, and considering different roles. The last two rows of the table apply to a node with node name NODENAME.

The table uses the following abbreviations.

* G = CoAP method GET
* F = CoAP method FETCH
* P = CoAP method POST
* D = CoAP method DELETE
* Type1 = Member as Requester and/or Responder
* Type2 = Member as Monitor
* Type3 = Non-member (authorized to be signature verifier)
* Type4 = Non-member (not authorized to be signature verifier)
* \* = Cannot join the group as signature verifier

| Resource                                 | Type1 | Type2 | Type3 | Type4 |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group                               | F     | F     | F     | F     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME                     | G P   | G P   | P *   | P     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/active              | G     | G     | -     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/verif-data          | -     | -     | G     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/creds               | G F   | G F   | G F   | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/kdc-cred            | G     | G     | F     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/stale-sids          | F     | F     | -     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/policies            | G     | G     | -     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/num                 | G     | G     | -     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/nodes/NODENAME      | G P D | G D   | -     | -     |
|------------------------------------------|-------|-------|-------|-------|
| /ace-group/GROUPNAME/nodes/NODENAME/cred | P     | -     | -     | -     |
{: #tab-methods title="Admitted CoAP Methods on the Group Manager Resources" align="center"}

### Signature Verifiers

Just like any candidate group member, a signature verifier provides the Group Manager with an access token, as described in {{ssec-token-post}}. However, unlike candidate group members, it does not join any OSCORE group, i.e., it does not perform the joining process defined in {{sec-joining-node-to-GM}}.

After successfully transferring an access token to the Group Manager, a signature verifier is allowed to perform only some operations as non-member of a group, and only for the OSCORE groups specified in the validated access token. These are the operations specified in {{sec-pub-keys}}, {{sec-gm-pub-key}}, {{sec-verif-data}}, and {{sec-retrieve-gnames}}.

Consistently, in case a node is not a member of the group with group name GROUPNAME and is authorized to be only signature verifier for that group, the Group Manager MUST reply with a 4.03 (Forbidden) error response if that node attempts to access any other endpoint than the following ones:

* /ace-group
* /ace-group/GROUPNAME/verif-data
* /ace-group/GROUPNAME/creds
* /ace-group/GROUPNAME/kdc-cred

## Operations Supported by Clients {#client-operations}

Building on what is defined in {{Section 4.1.1 of RFC9594}}, and with reference to the resources at the Group Manager newly defined earlier in {{sec-interface-GM}} of this document, it is expected that a Client minimally supports also the following set of operations and corresponding interactions with the Group Manager (REQ12).

* GET request to /ace-group/GROUPNAME/active, in order to check the current status of the OSCORE group.

* GET request to /ace-group/GROUPNAME/verif-data, in order for a signature verifier to retrieve data required to verify signatures of messages protected with the group mode of Group OSCORE and sent to a group (see {{Sections 12.3 and 7.5 of I-D.ietf-core-oscore-groupcomm}}). Note that this operation is relevant to support only to signature verifiers.

* FETCH request to /ace-group/GROUPNAME/stale-sids, in order to retrieve from the Group Manager the data required to delete some of the stored group members' authentication credentials and associated Recipient Contexts (see {{stale-sids-fetch}}). This data is provided as an aggregated set of stale Sender IDs, which are used as specified in {{missed-rekeying}}.

# Additional Interactions with the Group Manager # {#sec-additional-interactions}

This section defines the possible interactions with the Group Manager, in addition to the group joining specified in {{sec-joining-node-to-GM}}.

## Retrieve Updated Keying Material # {#sec-updated-key}

At some point, a group member considers the Group OSCORE Security Context invalid and to be renewed. This happens, for instance, after a number of unsuccessful security processing of incoming messages from other group members, or when the Security Context expires as specified by the 'exp' or 'exi' parameter of the Join Response.

When this happens, the group member retrieves updated security parameters and group keying material. This can occur in the two different ways described below.

### Get Group Keying Material ## {#ssec-updated-key-only}

If the group member wants to retrieve only the latest group keying material, it sends a Key Distribution Request to the Group Manager.

That is, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME at the Group Manager.

The Group Manager processes the Key Distribution Request according to {{Section 4.3.2 of RFC9594}}. The Key Distribution Response is formatted as defined in {{Section 4.3.2 of RFC9594}}, with the following additions.

* The 'key' parameter is formatted as defined in {{ssec-join-resp}} of this document, with the difference that it does not include the 'group_SenderId' parameter.

* The 'exp' parameter SHOULD be present.

* The 'exi' parameter MUST be present.

* The 'ace_groupcomm_profile' parameter MUST be present and has value coap_group_oscore_app.

Upon receiving the Key Distribution Response, the group member retrieves the updated security parameters and group keying material, and, if they differ from the current ones, uses them to set up the new Group OSCORE Security Context as described in {{Section 2 of I-D.ietf-core-oscore-groupcomm}}.

### Get Group Keying Material and OSCORE Sender ID ## {#ssec-updated-and-individual-key}

If the group member wants to retrieve the latest group keying material as well as the OSCORE Sender ID that it has in the OSCORE group, it sends a Key Distribution Request to the Group Manager.

That is, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the Group Manager.

The Group Manager processes the Key Distribution Request according to {{Section 4.8.1 of RFC9594}}. The Key Distribution Response is formatted as defined in {{Section 4.8.1 of RFC9594}}, with the following additions.

* The 'key' parameter is formatted as defined in {{ssec-join-resp}} of this document. If the requesting group member has exclusively the role of monitor, then the 'key' parameter does not include the 'group_SenderId' parameter.

  Note that, in any other case, the current Sender ID of the group member is not specified as a separate parameter, but instead by the 'group_SenderId' parameter within the 'key' parameter.

* The 'exp' parameter SHOULD be present.

* The 'exi' parameter MUST be present.

Upon receiving the Key Distribution Response, the group member retrieves the updated security parameters, group keying material, and Sender ID, and, if they differ from the current ones, uses them to set up the new Group OSCORE Security Context as described in {{Section 2 of I-D.ietf-core-oscore-groupcomm}}.

## Request to Change Individual Keying Material # {#sec-new-key}

As discussed in {{Section 2.6.2 of I-D.ietf-core-oscore-groupcomm}}, a group member may at some point exhaust its Sender Sequence Numbers in the OSCORE group.

When this happens, the group member MUST send a Key Renewal Request message to the Group Manager, as per {{Section 4.8.2.1 of RFC9594}}. That is, it sends a CoAP POST request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the Group Manager.

Upon receiving the Key Renewal Request, the Group Manager processes it as defined in {{Section 4.8.2 of RFC9594}}, with the following additions.

The Group Manager MUST return a 5.03 (Service Unavailable) response in case the OSCORE group identified by GROUPNAME is currently inactive (see {{ssec-resource-active}}). The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 9 ("Group currently not active").

Otherwise, the Group Manager performs one of the following actions.

1. If the requesting group member has exclusively the role of monitor, the Group Manager replies with a 4.00 (Bad Request) error response.  The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 1 ("Request inconsistent with the current roles").

2. Otherwise, the Group Manager takes one of the following actions.

   * The Group Manager rekeys the OSCORE group. That is, the Group Manager generates new group keying material for that group (see {{sec-group-rekeying-process}}), and replies to the group member with a group rekeying message as defined in {{sec-group-rekeying-process}}, providing the new group keying material. Then, the Group Manager rekeys the rest of the OSCORE group, as discussed in {{sec-group-rekeying-process}}.

     The Group Manager SHOULD perform a group rekeying only if already scheduled to  occur shortly, e.g., according to an application-specific rekeying period or scheduling, or as a reaction to a recent change in the group membership. In any other case, the Group Manager SHOULD NOT rekey the OSCORE group when receiving a Key Renewal Request (OPT12).

   * The Group Manager selects and assigns a new OSCORE Sender ID for that group member, according to the same criteria defined in {{ssec-join-resp}} for selecting and assigning an OSCORE Sender ID to include in a Join Response.

     Then, the Group Manager replies with a Key Renewal Response formatted as defined in {{Section 4.8.2 of RFC9594}}. The CBOR map in the response payload only includes the 'group_SenderId' parameter defined in {{ssec-iana-ace-groupcomm-parameters-registry}} of this document, specifying the new Sender ID of the group member encoded as a CBOR byte string.

     Consistently with {{Section 2.6.3.1 of I-D.ietf-core-oscore-groupcomm}}, the Group Manager MUST assign a new Sender ID that has not been used in the OSCORE group since the latest time when the current Gid value was assigned to the group.

     Furthermore, the Group Manager MUST add the old, relinquished Sender ID of the group member to the most recent set of stale Sender IDs for the group (see {{sssec-stale-sender-ids}}).

     The Group Manager MUST return a 5.03 (Service Unavailable) response in case there are currently no Sender IDs available to assign in the OSCORE group. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 4 ("No available individual keying material").

## Retrieve Authentication Credentials of Group Members # {#sec-pub-keys}

A group member or a signature verifier may need to retrieve the authentication credentials of (other) group members. To this end, the group member or signature verifier sends an Authentication Credential Request message to the Group Manager, as per {{Sections 4.4.1.1 and 4.4.2.1 of RFC9594}}. That is, it sends the request to the endpoint /ace-group/GROUPNAME/creds at the Group Manager.

If the Authentication Credential Request uses the method FETCH, then the Authentication Credential Request is formatted as defined in {{Section 4.4.1 of RFC9594}}. That is:

* Each element (if any) of the inner CBOR array 'role_filter' is formatted as in the inner CBOR array 'role_filter' of the 'get_creds' parameter of the Join Request when the parameter value is not the CBOR simple value `null` (0xf6) (see {{ssec-join-req-sending}}).

* Each element (if any) of the inner CBOR array 'id_filter' is a CBOR byte string, which encodes the OSCORE Sender ID of the group member for which the associated authentication credential is requested (REQ25).

Upon receiving the Authentication Credential Request, the Group Manager processes it as per Section 4.4.1 or Section 4.4.2 of {{RFC9594}}, depending on the request method being FETCH or GET, respectively. Additionally, if the Authentication Credential Request uses the method FETCH, the Group Manager silently ignores node identifiers included in the ’get_creds’ parameter of the request that are not associated with any current group member (REQ26).

The success Authentication Credential Response is formatted as defined in Section 4.4.1 or Section 4.4.2 of {{RFC9594}}, depending on the request method being FETCH or GET, respectively.

## Upload a New Authentication Credential # {#sec-update-pub-key}

A group member may need to provide the Group Manager with its new authentication credential to use in the group from then on, hence replacing the current one. This can be the case, for instance, if the signature or ECDH algorithm and possible associated parameters used in the OSCORE group have been changed, and the current authentication credential is not compatible with them.

To this end, the group member sends an Authentication Credential Update Request message to the Group Manager, as per {{Section 4.9.1.1 of RFC9594}}, with the following addition.

* The group member computes the proof-of-possession (PoP) evidence included in 'client_cred_verify' in the same way defined in {{ssec-join-req-sending}} when preparing a Join Request for the OSCORE group in question (REQ14), with the difference that the 'client_cred_verify' parameter MUST NOT specify an empty PoP evidence.

That is, the group member sends a CoAP POST request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME/cred at the Group Manager.

Upon receiving the Authentication Credential Update Request, the Group Manager processes it as per {{Section 4.9.1 of RFC9594}}, with the following additions.

* The N\_S challenge that is used to build the proof-of-possession input is computed as defined in {{sssec-challenge-value}} (REQ15).

* The Group Manager verifies the PoP challenge included in the 'client_cred_verify' parameter in the same way defined in {{ssec-join-req-processing}} when processing a Join Request for the OSCORE group in question (REQ14), with the difference that the verification MUST fail if the 'client_cred_verify' parameter specifies an empty PoP evidence.

* The Group Manager MUST return a 5.03 (Service Unavailable) response in case the OSCORE group identified by GROUPNAME is currently inactive (see {{ssec-resource-active}}). The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 9 ("Group currently not active").

* If the requesting group member has exclusively the role of monitor, the Group Manager replies with a 4.00 (Bad request) error response. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 1 ("Request inconsistent with the current roles").

* If the request is successfully processed, the Group Manager stores the association between: i) the new authentication credential of the group member; and ii) the Group Identifier (Gid), i.e., the OSCORE ID Context associated with the OSCORE group, together with the OSCORE Sender ID assigned to the group member in the group. The Group Manager MUST keep this association updated over time.

## Retrieve the Group Manager's Authentication Credential # {#sec-gm-pub-key}

A group member or a signature verifier may need to retrieve the authentication credential of the Group Manager. To this end, the requesting Client sends a KDC Authentication Credential Request message to the Group Manager.

{{sec-gm-pub-key-group-member}} defines how this operation is performed by a group member, building on {{Section 4.5.1.1 of RFC9594}}.

{{sec-gm-pub-key-signature-verifier}} defines how this operation is performed by a signature verifier, by relying on the additional FETCH handler defined in {{kdc-cred-fetch}} of this document.

### Retrieval for Group Members # {#sec-gm-pub-key-group-member}

A group member sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/kdc-cred at the Group Manager as per {{Section 4.5.1.1 of RFC9594}}, where GROUPNAME is the name of the OSCORE group.

In addition to what is defined in {{Section 4.5.1 of RFC9594}}, the Group Manager MUST respond with a 4.03 (Forbidden) error response, if the requesting Client is not a current group member. The response MUST have Content-Format set to "application/concise-problem-details+cbor" {{RFC9290}} and is formatted as defined in {{Section 4.1.2 of RFC9594}}. Within the Custom Problem Detail entry 'ace-groupcomm-error', the value of the 'error-id' field MUST be set to 0 ("Operation permitted only to group members").

The payload of the 2.05 (Content) KDC Authentication Credential Response is a CBOR map, which is formatted as defined in {{Section 4.5.1 of RFC9594}}. The Group Manager specifies the parameters 'kdc_cred', 'kdc_nonce', and 'kdc_cred_verify' as defined for the Join Response in {{ssec-join-resp}} of this document. This especially applies to the computing of the proof-of-possession (PoP) evidence included in 'kdc_cred_verify' (REQ21).

Upon receiving a 2.05 (Content) KDC Authentication Credential Response, the requesting Client retrieves the Group Manager's authentication credential from the 'kdc_cred' parameter, and proceeds as defined in {{Section 4.5.1.1 of RFC9594}}. The requesting Client verifies the PoP evidence included in 'kdc_cred_verify' by means of the same method used when processing the Join Response, as defined in {{ssec-join-resp}} of this document (REQ21).

### Retrieval for Signature Verifiers # {#sec-gm-pub-key-signature-verifier}

A Client signature verifier sends a CoAP FETCH request to the endpoint /ace-group/GROUPNAME/kdc-cred at the Group Manager defined in {{Section 4.5 of RFC9594}}, where GROUPNAME is the name of the OSCORE group.

The request MUST have Content-Format "application/ace-groupcomm+cbor". The payload of the request is formatted as a CBOR map, which MUST contain the following field with the value specified below:

* 'cnonce': encoded as a CBOR byte string, whose value is a dedicated nonce N_C generated by the Client. For the N_C value, it is RECOMMENDED to use an 8-byte long random nonce.

The payload of the 2.05 (Content) KDC Authentication Credential Response is a CBOR map, which is formatted as defined in {{Section 4.5.1 of RFC9594}}, with the following difference:

* The 'kdc_cred_verify' field  specifies the PoP evidence computed by the Group Manager over the following PoP input: the nonce N_C (encoded as a CBOR byte string) concatenated with the nonce N_KDC (encoded as a CBOR byte string), where:

  - N_C is the nonce generated by the Client signature verifier and specified in the 'cnonce' field of the received KDC Authentication Credential Request.

  - N_KDC is the nonce generated by the Group Manager and specified in the 'kdc_nonce' field of the KDC Authentication Credential Response.

The Group Manager specifies the 'kdc_cred' field and 'kdc_nonce' field as defined for the Join Response in {{ssec-join-resp}} of this document. The computed PoP evidence included in the 'kdc_cred_verify' field is always a signature computed over the PoP input defined above (REQ21).

Upon receiving a 2.05 (Content) KDC Authentication Credential Response, the requesting Client retrieves the Group Manager's authentication credential from the 'kdc_cred' parameter. Then, it proceeds as defined in {{Section 4.5.1.1 of RFC9594}}, with the difference that it verifies the PoP evidence included in 'kdc_cred_verify' field by verifying a signature and using the PoP input defined above (REQ21)

Note that a signature verifier would not receive a successful response from the Group Manager, in case GROUPNAME denotes a pairwise-only group (see {{kdc-cred-fetch}}).

{{fig-gm-pub-key-signature-verifier-req-resp}} gives an overview of the exchange described above,  while {{fig-gm-pub-key-signature-verifier-resp-ex}} shows an example of Signature Verification Data Request-Response.

~~~~~~~~~~~ aasvg
Signature                                                       Group
Verifier                                                       Manager
  |                                                               |
  |              KDC Authentication Credential Request            |
  |-------------------------------------------------------------->|
  |               FETCH /ace-group/GROUPNAME/kdc-cred             |
  |                                                               |
  |<--- KDC Authentication Credential Response: 2.05 (Content) ---|
  |                                                               |
~~~~~~~~~~~
{: #fig-gm-pub-key-signature-verifier-req-resp title="Message Flow of KDC Authentication Credential Request-Response, with a Signature Verifier as Requesting Client" artwork-align="center"}

~~~~~~~~~~~
   Request:

   Header: FETCH (Code=0.05)
   Uri-Host: "kdc.example.com"
   Uri-Path: "ace-group"
   Uri-Path: "g1"
   Uri-Path: "kdc-cred"
   Content-Format: 261 (application/ace-groupcomm+cbor)
   Payload (in CBOR diagnostic notation):
   {
     / cnonce / 6: h'6c5a8891bbcf4199'
   }


   Response:

   Header: Content (Code=2.05)
   Content-Format: 261 (application/ace-groupcomm+cbor)
   Payload (in CBOR diagnostic notation):
   {
     / kdc_cred /        17: h'a2026008a101a5010202419920012158
                               2065eda5a12577c2bae829437fe33870
                               1a10aaa375e1bb5b5de108de439c0855
                               1d2258201e52ed75701163f7f9e40ddf
                               9f341b3dc9ba860af7e0ca7ca7e9eecd
                               0084d19c',
     / kdc_nonce /       18: h'aff56da30b7db12a',
     / kdc_cred_verify / 19: h'f3e4be39445b1a3e83e1510d1aca2f2e
                               3fc54702aa56e1b2cb20284294c9106a
                               8a7c081c7645042b18aba9d1fad1bd9c
                               63f91bac658d69351210a031d8fc7c5f'
   }
~~~~~~~~~~~
{: #fig-gm-pub-key-signature-verifier-resp-ex title="Example of KDC Authentication Credential Request-Response, with a Signature Verifier as Requesting Client"}

## Retrieve Signature Verification Data # {#sec-verif-data}

A signature verifier may need to retrieve data required to verify signatures of messages protected with the group mode and sent to a group (see {{Sections 7.5 and 12.3 of I-D.ietf-core-oscore-groupcomm}}). To this end, the signature verifier sends a Signature Verification Data Request message to the Group Manager.

That is, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/verif-data at the Group Manager defined in {{ssec-resource-verif-data}} of this document, where GROUPNAME is the name of the OSCORE group.

The payload of the 2.05 (Content) Signature Verification Data Response is a CBOR map, which has the format used for the Join Response message in {{ssec-join-resp}}, with the following differences.

* Of the parameters present in the Join Response message, only the parameters 'gkty', 'key', 'num', 'exp', 'exi', and 'ace_groupcomm_profile' are present in the Signature Verification Data Response.

  The 'key' parameter includes only the following data.

  - The parameters 'hkdf', 'contextId', 'cred_fmt', 'gp_enc_alg', 'sign_alg', and 'sign_params'. These parameters MUST be present.

  - The parameters 'alg' and 'ecdh_alg'. These parameters MUST NOT be present if the group is a signature-only group. Otherwise, they MUST be present.

* The 'sign_enc_key' parameter is also included, with CBOR label registered in {{ssec-iana-ace-groupcomm-parameters-registry}}. This parameter specifies the Signature Encryption Key of the OSCORE Group, encoded as a CBOR byte string. The Group Manager derives the Signature Encryption Key from the group keying material, as per {{Section 2.1.9 of I-D.ietf-core-oscore-groupcomm}}. This parameter MUST be present.

In order to verify signatures in the group (see {{Section 7.5 of I-D.ietf-core-oscore-groupcomm}}), the signature verifier relies on: the data retrieved from the 2.05 (Content) Signature Verification Data Response; the public keys of the group members signing the messages to verify, retrieved from those members' authentication credentials that can be obtained as defined in {{sec-pub-keys}}; and the public key of the Group Manager, retrieved from the Group Manager's authentication credential that can be obtained as defined in {{sec-gm-pub-key-signature-verifier}}.

{{fig-verif-data-req-resp}} gives an overview of the exchange described above,  while {{fig-verif-data-req-resp-ex}} shows an example of Signature Verification Data Request-Response.

~~~~~~~~~~~ aasvg
Signature                                                     Group
Verifier                                                     Manager
  |                                                             |
  |              Signature Verification Data Request            |
  |------------------------------------------------------------>|
  |              GET /ace-group/GROUPNAME/verif-data            |
  |                                                             |
  |<--- Signature Verification Data Response: 2.05 (Content) ---|
  |                                                             |
~~~~~~~~~~~
{: #fig-verif-data-req-resp title="Message Flow of Signature Verification Data Request-Response" artwork-align="center"}

~~~~~~~~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Host: "kdc.example.com"
   Uri-Path: "ace-group"
   Uri-Path: "g1"
   Uri-Path: "verif-data"


   Response:

   Header: Content (Code=2.05)
   Content-Format: 261 (application/ace-groupcomm+cbor)
   Payload (in CBOR diagnostic notation):
   {
                       / gkty / 7: e'group_oscore_input_material_obj',
                       / key /  8: {
                         / hkdf /       3: 5, / HMAC with SHA-256 /
                         / contextId /  6: h'37fc',
                              e'cred_fmt': 33, / x5chain /
                            e'gp_enc_alg': 10, / AES-CCM-16-64-128 /
                              e'sign_alg': -8, / EdDSA /
                           e'sign_params': [[1], [1, 6]]
                                           / [[OKP], [OKP, Ed25519]] /
                       },
     / num /                    9: 12,
     / ace_groupcomm_profile / 10: e'coap_group_oscore_app',
     / exp /                   11: 1609459200,
     / exi /                   12: 2592000,
                  e'sign_enc_key': h'bc661fae6742abc3dd01beda1142567c'
   }
~~~~~~~~~~~
{: #fig-verif-data-req-resp-ex title="Example of Signature Verification Data Request-Response"}

## Retrieve the Group Policies # {#sec-policies}

A group member may request the current policies used in the OSCORE group. To this end, the group member sends a Policies Request, as per {{Section 4.6.1.1 of RFC9594}}. That is, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/policies at the Group Manager, where GROUPNAME is the name of the OSCORE group.

Upon receiving the Policies Request, the Group Manager processes it as per {{Section 4.6.1 of RFC9594}}. The success Policies Response is formatted as defined in {{Section 4.6.1 of RFC9594}}.

## Retrieve the Keying Material Version # {#sec-version}

A group member may request the current version of the keying material used in the OSCORE group. To this end, the group member sends a Version Request, as per {{Section 4.7.1.1 of RFC9594}}. That is, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/num at the Group Manager, where GROUPNAME is the name of the OSCORE group.

Upon receiving the Version Request, the Group Manager processes it as per {{Section 4.7.1 of RFC9594}}. The success Version Response is formatted as defined in {{Section 4.7.1 of RFC9594}}.

## Retrieve the Group Status # {#sec-status}

A group member may request the current status of the OSCORE group, i.e., active or inactive. To this end, the group member sends a Group Status Request to the Group Manager.

That is, the group member sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/active at the Group Manager defined in {{ssec-resource-active}} of this document, where GROUPNAME is the name of the OSCORE group.

The payload of the 2.05 (Content) Group Status Response includes the CBOR Simple Value `true` (0xf5) if the group is currently active, or the CBOR Simple Value `false` (0xf4) otherwise. The group is considered active if it is set to allow new members to join, and if communication within the group is fine to occur.

Upon learning from a 2.05 (Content) Group Status Response that the group is currently inactive, the group member SHOULD stop taking part in communications within the group, until the group becomes active again.

Upon learning from a 2.05 (Content) Group Status Response that the group has become active again, the group member can resume taking part in communications within the group.

{{fig-key-status-req-resp}} gives an overview of the exchange described above, while {{fig-key-status-req-resp-ex}} shows an example of Group Status Request-Response.

~~~~~~~~~~~ aasvg
Group                                                          Group
Member                                                        Manager
  |                                                              |
  |--- Group Status Request: GET /ace-group/GROUPNAME/active --->|
  |                                                              |
  |<----------- Group Status Response: 2.05 (Content) -----------|
  |                                                              |
~~~~~~~~~~~
{: #fig-key-status-req-resp title="Message Flow of Group Status Request-Response" artwork-align="center"}

~~~~~~~~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Host: "kdc.example.com"
   Uri-Path: "ace-group"
   Uri-Path: "g1"
   Uri-Path: "active"


   Response:

   Header: Content (Code=2.05)
   Content-Format: 60 (application/cbor)
   Payload (in CBOR diagnostic notation):
     true
~~~~~~~~~~~
{: #fig-key-status-req-resp-ex title="Example of Group Status Request-Response"}

## Retrieve Group Names {#sec-retrieve-gnames}

A node may want to retrieve from the Group Manager the group name and the URI of the group-membership resource of a group. This is relevant in the following cases.

* Before joining a group, a joining node may know only the current Group Identifier (Gid) of that group, but not the group name and the URI of the group-membership resource.

* As current group member in several groups, the node has missed a previous group rekeying in one of them (see {{sec-group-rekeying-process}}). Hence, it retains stale keying material and fails to decrypt received messages exchanged in that group.

  Such messages do not provide a direct hint to the correct group name, that the node would need in order to retrieve the latest keying material and authentication credentials from the Group Manager (see {{ssec-updated-key-only}}, {{ssec-updated-and-individual-key}}, and {{sec-pub-keys}}). However, such messages may specify the current Gid of the group, as value of the 'kid_context' field of the OSCORE CoAP option (see {{Section 6.1 of RFC8613}} and {{Section 3.2 of I-D.ietf-core-oscore-groupcomm}}).

* As signature verifier, the node also refers to a group name for retrieving the required authentication credentials from the Group Manager (see {{sec-pub-keys}}). As discussed above, intercepted messages do not provide a direct hint to the correct group name, while they may specify the current Gid of the group, as value of the 'kid_context' field of the OSCORE CoAP option. In such a case, upon intercepting a message in the group, the node requires to correctly map the Gid currently used in the group with the invariant group name.

   Furthermore, since it is not a group member, the node does not take part to a possible group rekeying. Thus, following a group rekeying and the consequent change of Gid in a group, the node would retain the old Gid value and cannot correctly associate intercepted messages to the right group, especially if acting as signature verifier in several groups. This in turn prevents the efficient verification of signatures, and especially the retrieval of required, new authentication credentials from the Group Manager.

In either case, the node only knows the current Gid of the group, as learned from received or intercepted messages exchanged in the group. As detailed below, the node can contact the Group Manager, and request the group name and URI of the group-membership resource corresponding to that Gid. Then, it can use that information to join the group, or get the latest keying material as a current group member, or retrieve authentication credentials used in the group as a signature verifier. To this end, the node sends a Group Name and URI Retrieval Request, as per {{Section 4.2.1.1 of RFC9594}}.

That is, the node sends a CoAP FETCH request to the endpoint /ace-group at the Group Manager formatted as defined in {{Section 4.2.1 of RFC9594}}. Each element of the CBOR array 'gid' is a CBOR byte string (REQ13), which encodes the Gid of the group for which the group name and the URI of the group-membership resource are requested.

Upon receiving the Group Name and URI Retrieval Request, the Group Manager processes it as per {{Section 4.2.1 of RFC9594}}. The success Group Name and URI Retrieval Response is formatted as defined in {{Section 4.2.1 of RFC9594}}. Each element of the CBOR array 'gid' is a CBOR byte string (REQ13), which encodes the Gid of the group for which the group name and the URI of the group-membership resource are provided.

For each of its groups, the Group Manager maintains an association between the group name and the URI of the group-membership resource on one hand, and only the current Gid for that group on the other hand. That is, the Group Manager does not maintain an association between the former pair and any other Gid for that group than the current, most recent one.

{{fig-group-names-req-resp}} gives an overview of the exchanges described above, while {{fig-group-names-req-resp-ex}} shows an example of Group Name and URI Retrieval Request-Response.

~~~~~~~~~~~ aasvg
                                                                Group
Node                                                           Manager
 |                                                                |
 |--- Group Name and URI Retrieval Request: FETCH /ace-group ---->|
 |                                                                |
 |<--- Group Name and URI Retrieval Response: 2.05 (Content) -----|
 |                                                                |
~~~~~~~~~~~
{: #fig-group-names-req-resp title="Message Flow of Group Name and URI Retrieval Request-Response" artwork-align="center"}

~~~~~~~~~~~
   Request:

   Header: FETCH (Code=0.05)
   Uri-Host: "kdc.example.com"
   Uri-Path: "ace-group"
   Content-Format: 261 (application/ace-groupcomm+cbor)
   Payload (in CBOR diagnostic notation):
   {
     / gid / 0: [h'37fc', h'84bd']
   }


   Response:

   Header: Content (Code=2.05)
   Content-Format: 261 (application/ace-groupcomm+cbor)
   Payload (in CBOR diagnostic notation):
   {
     / gid /   0: [h'37fc', h'84bd'],
     / gname / 1: ["g1", "g2"],
     / guri /  2: ["/ace-group/g1", "/ace-group/g2"]
   }
~~~~~~~~~~~
{: #fig-group-names-req-resp-ex title="Example of Group Name and URI Retrieval Request-Response"}

## Leave the Group # {#sec-leave-req}

A group member may request to leave the OSCORE group. To this end, the group member sends a Group Leaving Request, as per {{Section 4.8.3.1 of RFC9594}}. That is, it sends a CoAP DELETE request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the Group Manager.

Upon receiving the Group Leaving Request, the Group Manager processes it as per {{Section 4.8.3 of RFC9594}}. Then, the Group Manager performs the follow-up actions defined in {{sec-leaving}} of this document.

# Removal of a Group Member # {#sec-leaving}

Other than after a spontaneous request to the Group Manager as described in {{sec-leave-req}}, a node may be forcibly removed from the OSCORE group, e.g., due to expired or revoked authorization.

In either case, if the Group Manager reassigns Gid values during the group's lifetime (see {{Sections 12.2.1.1 and E of I-D.ietf-core-oscore-groupcomm}}), the Group Manager "forgets" the Birth Gid currently associated with the leaving node in the OSCORE group. This was stored following the Join Response sent to that node, after its latest (re-)joining of the OSCORE group (see {{ssec-join-resp}}).

If any of the two conditions below holds, the Group Manager MUST inform the leaving node of its eviction as follows. If both conditions hold, the Group Manager MUST inform the leaving node by using only the method corresponding to one of either conditions.

* If, upon joining the group (see {{ssec-join-req-sending}}), the leaving node specified a URI in the 'control_uri' parameter defined in {{Section 4.3.1 of RFC9594}}, then the Group Manager sends a DELETE request targeting the URI specified in the 'control_uri' parameter (OPT7).

* If the leaving node has been observing the associated resource at /ace-group/GROUPNAME/nodes/NODENAME, then the Group Manager sends an unsolicited 4.04 (Not Found) error response to the leaving node, as specified in {{Section 4.3.2 of RFC9594}}.

Furthermore, the Group Manager might intend to evict all the current group members from the group at once. In such a case, if the Join Responses sent by the Group Manager to nodes joining the group (see {{ssec-join-resp}}) specify a URI in the 'control_group_uri' parameter defined in {{Section 4.3.1 of RFC9594}}, then the Group Manager MUST additionally send a DELETE request targeting the URI specified in the 'control_group_uri' parameter (OPT10).

If the leaving node has not exclusively the role of monitor, then the Group Manager performs the following actions.

* The Group Manager frees the OSCORE Sender ID value of the leaving node. This value MUST NOT become available for possible upcoming joining nodes in the same group, until the group has been rekeyed and assigned a new Group Identifier (Gid).

* The Group Manager MUST add the relinquished Sender ID of the leaving node to the most recent set of stale Sender IDs for the group (see {{sssec-stale-sender-ids}}).

* The Group Manager cancels the association between, on one hand, the authentication credential of the leaving node and, on the other hand, the Gid associated with the OSCORE group together with the freed Sender ID value.

* The Group Manager deletes the authentication credential of the leaving node, if that authentication credential has no remaining association with any pair (Gid, Sender ID).

Then, the Group Manager MUST generate updated security parameters and group keying material, and provide it to the remaining group members (see {{sec-group-rekeying-process}}). As a consequence, the leaving node is not able to acquire the new security parameters and group keying material distributed after its leaving.

The same considerations from {{Section 5 of RFC9594}} apply here as well, considering the Group Manager acting as KDC.

# Group Rekeying Process {#sec-group-rekeying-process}

In order to rekey an OSCORE group, the Group Manager distributes the following information for that group:

* A new Group Identifier (Gid), i.e., a new OSCORE ID Context.

* A new OSCORE Master Secret.

* Optionally, a new OSCORE Master Salt.

Before starting such distribution, the Group Manager MUST increment the version number of the group keying material used in the group.

As per {{Sections 12.2.1.1 and E of I-D.ietf-core-oscore-groupcomm}}, the Group Manager MAY reassign a Gid to the same group over that group's lifetime, e.g., once the whole space of Gid values has been used for the group in question. If the Group Manager supports reassignment of Gid values and performs it in a group, then the Group Manager additionally takes the following actions.

* Before rekeying the group, the Group Manager MUST check if the new Gid to be distributed coincides with the Birth Gid of any of the current group members (see {{ssec-join-resp}}).

* If any of such "elder members" is found in the group, then the Group Manager MUST evict them from the group. That is, the Group Manager MUST terminate their membership and MUST rekey the group in such a way that the new keying material is not provided to those evicted elder members. This also includes adding their relinquished Sender IDs to the most recent set of stale Sender IDs for the group (see {{sssec-stale-sender-ids}}), before rekeying the group.

  Until a further following group rekeying, the Group Manager MUST store the list of those latest-evicted elder members. If any of those nodes re-joins the group before a further following group rekeying occurs, the Group Manager MUST NOT rekey the group upon their re-joining. When one of those nodes re-joins the group, the Group Manager can rely, e.g., on the ongoing secure communication association to recognize the node as included in the stored list.

Throughout the rekeying execution, the Group Manager MUST preserve the same unchanged OSCORE Sender IDs for all group members that are intended to remain in the group. This avoids affecting the retrieval of authentication credentials from the Group Manager and the verification of group messages.

The Group Manager MUST support the "Point-to-Point" group rekeying scheme registered in {{Section 11.12 of RFC9594}}, as per the detailed use defined in {{sending-rekeying-msg}} of this document. Future specifications may define how this application profile can use alternative group rekeying schemes, which MUST comply with the functional steps defined in {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}. The Group Manager MUST indicate the use of such an alternative group rekeying scheme to joining nodes, by means of the 'group_rekeying' parameter included in Join Response messages (see {{ssec-join-resp}}).

It is RECOMMENDED that the Group Manager gets confirmation of successful distribution from the group members, and admits a maximum number of individual retransmissions to non-confirming group members. Once completed the group rekeying process, the Group Manager creates a new empty set of stale Sender IDs associated with the version of the newly distributed group keying material (see {{sssec-stale-sender-ids}}).

In case the rekeying terminates and some group members have not received the new keying material, they will not be able to correctly process following secured messages exchanged in the group. These group members will eventually contact the Group Manager, in order to retrieve the current keying material and its version.

Some of these group members may be in multiple groups, each associated with a different Group Manager. When failing to correctly process messages secured with the new keying material, these group members may not have sufficient information to determine which exact Group Manager they should contact, in order to retrieve the current keying material they are missing.

If the Gid is formatted as described in {{Section C of I-D.ietf-core-oscore-groupcomm}}, then the Group Prefix can be used as a hint to determine the right Group Manager, as long as no collisions among Group Prefixes are experienced. Otherwise, a group member needs to contact the Group Manager of each group, e.g., by first requesting only the version of the current group keying material (see {{sec-version}}) and then possibly requesting the current keying material (see {{ssec-updated-key-only}}).

Furthermore, some of these group members can be in multiple groups, all of which are associated with the same Group Manager. In this case, these group members may also not have sufficient information to determine which exact group they should refer to, when contacting the right Group Manager. Hence, they need to contact a Group Manager multiple times, i.e., separately for each group they belong to and associated with that Group Manager.

{{receiving-rekeying-msg}} defines the actions performed by a group member upon receiving the new group keying material. {{missed-rekeying}} discusses how a group member can realize that it has missed one or more rekeying instances, and the actions it is accordingly required to take.

## Sending Rekeying Messages {#sending-rekeying-msg}

When using the "Point-to-Point" group rekeying scheme, the group rekeying messages MUST have Content-Format set to "application/ace-groupcomm+cbor" and have the same format used for the Join Response message in {{ssec-join-resp}}, with the following differences. Note that this extends the minimal content of a rekeying message as defined in {{Section 6 of RFC9594}} (OPT14).

* Of the parameters present in the Join Response message, only the parameters 'gkty', 'key', 'num', 'exp', 'exi', and 'ace_groupcomm_profile' are present.

  The 'key' parameter includes only the following data.

  - The 'ms' parameter, specifying the new OSCORE Master Secret value. This parameter MUST be present.

  - The 'contextId' parameter, specifying the new Gid to use as OSCORE ID Context value. This parameter MUST be present.

  - The 'salt' value, specifying the new OSCORE Master Salt value. This parameter MAY be present.

* The 'stale_node_ids' parameter MUST also be included, with CBOR label registered in {{ssec-iana-ace-groupcomm-parameters-registry}}. This parameter is encoded as a CBOR array, where each element is encoded as a CBOR byte string. The order of elements in the CBOR array is irrelevant. The parameter is populated as follows.

  - The Group Manager creates an empty CBOR array ARRAY.

  - The Group Manager considers the most recent set of stale Sender IDs for the group (see {{sssec-stale-sender-ids}}), i.e., the set X associated with the current version of the group keying material about to be relinquished.

  - For each Sender ID in X, the Group Manager encodes it as a CBOR byte string and adds the result to ARRAY.

  - The 'stale_node_ids' parameter takes ARRAY as value.

* The parameters 'creds', 'peer_roles', and 'peer_identifiers' SHOULD be present, if the group rekeying is performed due to one or multiple Clients that have requested to join the group.

  Following the same semantics used in the Join Response message (see {{ssec-join-resp}}), the three parameters specify the authentication credential, roles in the group and node identifier of each of the Clients that have requested to join the group. The Group Manager MUST NOT include a non-empty subset of these three parameters.

The Group Manager separately sends a group rekeying message formatted as defined above to each group member to be rekeyed.

Each rekeying message MUST be secured with the pairwise secure communication association between the Group Manager and the group member used during the joining process. Each rekeying message can target the 'control_uri' URI path defined in {{Section 4.3.1 of RFC9594}} (OPT7), if provided by the intended recipient upon joining the group (see {{ssec-join-req-sending}}).

This distribution approach requires group members to act (also) as servers, in order to correctly handle unsolicited group rekeying messages from the Group Manager. If a group member and the Group Manager use OSCORE {{RFC8613}} to secure their pairwise communications, then the group member MUST create a Replay Window in its own Recipient Context upon establishing the OSCORE Security Context with the Group Manager, e.g., by means of the OSCORE profile of ACE {{RFC9203}}.

Group members and the Group Manager SHOULD additionally support alternative distribution approaches that do not require group members to act (also) as servers. A number of such approaches are defined in {{Section 6 of RFC9594}}. In particular, a group member may use CoAP Observe {{RFC7641}} and subscribe for updates to the group-membership resource of the group, at the endpoint /ace-group/GROUPNAME of the Group Manager (see {{Section 6.1 of RFC9594}}). Alternatively, a full-fledged Pub-Sub model can be considered {{I-D.ietf-core-coap-pubsub}}, where the Group Manager publishes to a rekeying topic hosted at a Broker, while the group members subscribe to such topic (see {{Section 6.2 of RFC9594}}).

## Receiving Rekeying Messages {#receiving-rekeying-msg}

After having received the new group keying material, a group member proceeds as follows. Unless otherwise specified, the following is independent of the specifically used group rekeying scheme.

The group member considers the stale Sender IDs received from the Group Manager. If the "Point-to-Point" group rekeying scheme as detailed in {{sending-rekeying-msg}} is used, the stale Sender IDs are specified by the 'stale_node_ids' parameter.

After that, as per {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}, the group member MUST remove every authentication credential associated with a stale Sender ID from its list of group members' authentication credentials used in the group, and MUST delete each of its Recipient Contexts used in the group whose corresponding Recipient ID is a stale Sender ID.

Then, the following cases can occur, based on the version number V' of the new group keying material distributed through the rekeying process. If the "Point-to-Point" group rekeying scheme as detailed in {{sending-rekeying-msg}} is used, this information is specified by the 'num' parameter.

* The group member has not missed any group rekeying. That is, the old keying material stored by the group member has version number V, while the received new keying material has version number V' = (V + 1). In such a case, the group member simply installs the new keying material and derives the corresponding new Security Context.

* The group member has missed one or more group rekeying instances. That is, the old keying material stored by the group member has version number V, while the received new keying material has version number V' > (V + 1). In such a case, the group member MUST proceed as defined in {{missed-rekeying}}.

* The group member has received keying material not newer than the stored one. That is, the old keying material stored by the group member has version number V, while the received keying material has version number V' < (V + 1). In such a case, the group member MUST ignore the received rekeying messages and MUST NOT install the received keying material.

## Missed Rekeying Instances {#missed-rekeying}

A group member can realize to have missed one or more rekeying instances in one of the ways discussed below. In the following, V denotes the version number of the old keying material stored by the group member, while V' denotes the version number of the latest, possibly just distributed, keying material.

a. The group member has participated to a rekeying process that has distributed new keying material with version number V' > (V + 1), as discussed in {{receiving-rekeying-msg}}.

b. The group member has obtained the latest keying material from the Group Manager, as a response to a Key Distribution Request (see {{ssec-updated-key-only}}) or to a Join Request when re-joining the group (see {{ssec-join-req-sending}}). That is, V is different than V' specified by the 'num' parameter in the response.

c. The group member has obtained the authentication credentials of other group members, through an Authentication Credential Request-Response exchange with the Group Manager (see {{sec-pub-keys}}). That is, V is different than V' specified by the 'num' parameter in the response.

d. The group member has performed a Version Request-Response exchange with the Group Manager (see {{sec-version}}). That is, V is different than V' specified by the 'num' parameter in the response.

In either case, the group member MUST delete the stored keying material with version number V.

If case (a) or case (b) applies, the group member MUST perform the following actions.

1. The group member MUST NOT install the latest keying material yet, in case that was already obtained.

2. The group member sends a Stale Sender IDs Request to the Group Manager (see {{sec-retrieve-stale-sids}}), specifying the version number V as payload of the request.

   If the Stale Sender IDs Response from the Group Manager has no payload, the group member MUST remove all the authentication credentials from its list of group members' authentication credentials used in the group, and MUST delete all its Recipient Contexts used in the group.

   Otherwise, the group member considers the stale Sender IDs specified in the Stale Sender IDs Response from the Group Manager. Then, the group member MUST remove every authentication credential associated with a stale Sender ID from its list of group members' authentication credentials used in the group, and MUST delete each of its Recipient Contexts used in the group whose corresponding Recipient ID is a stale Sender ID.

3. The group member installs the latest keying material with version number V' and derives the corresponding new Security Context.

If case (c) or case (d) applies, the group member SHOULD perform the following actions.

1. The group member sends a Stale Sender IDs Request to the Group Manager (see {{sec-retrieve-stale-sids}}), specifying the version number V as payload of the request.

   If the Stale Sender IDs Response from the Group Manager has no payload, the group member MUST remove all the authentication credentials from its list of group members' authentication credentials used in the group, and MUST delete all its Recipient Contexts used in the group.

   Otherwise, the group member considers the stale Sender IDs specified in the Stale Sender IDs Response from the Group Manager. Then, the group member MUST remove every authentication credential associated with a stale Sender ID from its list of group members' authentication credentials used in the group, and MUST delete each of its Recipient Contexts used in the group whose corresponding Recipient ID is a stale Sender ID.

2. The group member retrieves the latest keying material with version number V' from the Group Manager. This can happen by sending a Key Distribution Request to the Group Manager (see {{ssec-updated-key-only}}) and {{ssec-updated-and-individual-key}}).

3. The group member installs the latest keying material with version number V' and derives the corresponding new Security Context.

If case (c) or case (d) applies, the group member can alternatively perform the following actions.

1. The group member re-joins the group (see {{ssec-join-req-sending}}). When doing so, the group member MUST re-join with the same roles it currently has in the group, and MUST request from the Group Manager the authentication credentials of all the current group members. That is, the 'get_creds' parameter of the Join Request MUST be present and MUST be set to the CBOR Simple Value `null` (0xf6).

2. When receiving the Join Response (see {{ssec-join-resp-processing}} and {{ssec-join-resp-processing}}), the group member retrieves the set Z of authentication credentials specified in the 'creds' parameter.

   Then, the group member MUST remove every authentication credential which is not in Z from its list of group members' authentication credentials used in the group, and MUST delete each of its Recipient Contexts used in the group that does not include any of the authentication credentials in Z.

3. The group member installs the latest keying material with version number V' and derives the corresponding new Security Context.

### Retrieve Stale Sender IDs {#sec-retrieve-stale-sids}

When realizing to have missed one or more group rekeying instances (see {{missed-rekeying}}), a node needs to retrieve from the Group Manager the data required to delete some of its stored group members' authentication credentials and Recipient Contexts (see {{stale-sids-fetch}}). These data is provided as an aggregated set of stale Sender IDs, which are used as specified in {{missed-rekeying}}.

That is, the node sends a CoAP FETCH request to the endpoint /ace-group/GROUPNAME/stale-sids at the Group Manager defined in {{ssec-resource-stale-sids}} of this document, where GROUPNAME is the name of the OSCORE group.

The payload of the Stale Sender IDs Request MUST include a CBOR unsigned integer. This encodes the version number V of the most recent group keying material stored and installed by the requesting Client, which is older than the latest, possibly just distributed, keying material with version number V'.

The handler MUST reply with a 4.00 (Bad Request) error response, if the request is not formatted correctly. Also, the handler MUST respond with a 4.00 (Bad Request) error response, if the specified version number V is greater or equal than the version number V' associated with the latest keying material in the group, i.e., in case V >= V'.

Otherwise, the handler responds with a 2.05 (Content) Stale Sender IDs Response. The payload of the response is formatted as defined below, where SKEW = (V' - V + 1).

* The Group Manager considers ITEMS as the current number of sets of stale Sender IDs for the group (see {{sssec-stale-sender-ids}}).

* If SKEW > ITEMS, the Stale Sender IDs Response MUST NOT have a payload.

* Otherwise, the payload of the Stale Sender IDs Response MUST include a CBOR array, where each element is encoded as a CBOR byte string. The order of elements in the CBOR array is irrelevant. The Group Manager populates the CBOR array as follows.

   - The Group Manager creates an empty CBOR array ARRAY and an empty set X.

   - The Group Manager considers the SKEW most recent sets of stale Sender IDs for the group. Note that the most recent set is the one associated with the latest version of the group keying material.

   - The Group Manager copies all the Sender IDs from the selected sets into X. When doing so, the Group Manager MUST discard duplicates. That is, the same Sender ID MUST NOT be present more than once in the final content of X.

   - For each Sender ID in X, the Group Manager encodes it as a CBOR byte string and adds the result to ARRAY.

   - Finally, ARRAY is specified as payload of the Stale Sender IDs Response. Note that ARRAY might result in the empty CBOR array.

{{fig-stale-ids-req-resp}} gives an overview of the exchange described above,  while {{fig-stale-ids-req-resp-ex}} shows an example of Stale Sender IDs Request-Response.

~~~~~~~~~~~ aasvg
                                                              Group
Node                                                         Manager
  |                                                             |
  |                   Stale Sender IDs Request                  |
  |------------------------------------------------------------>|
  |             FETCH /ace-group/GROUPNAME/stale-sids           |
  |                                                             |
  |<---------- Stale Sender IDs Response: 2.05 (Content) -------|
  |                                                             |
~~~~~~~~~~~
{: #fig-stale-ids-req-resp title="Message Flow of Stale Sender IDs Request-Response" artwork-align="center"}

~~~~~~~~~~~
   Request:

   Header: FETCH (Code=0.05)
   Uri-Host: "kdc.example.com"
   Uri-Path: "ace-group"
   Uri-Path: "g1"
   Uri-Path: "stale-sids"
   Content-Format: 60 (application/cbor)
   Payload (in CBOR diagnostic notation):
     42


   Response:

   Header: Content (Code=2.05)
   Content-Format: 60 (application/cbor)
   Payload (in CBOR diagnostic notation):
     [h'01', h'fc', h'12ab', h'de44', h'ff']
~~~~~~~~~~~
{: #fig-stale-ids-req-resp-ex title="Example of Stale Sender IDs Request-Response"}

# ACE Groupcomm Parameters {#ace-groupcomm-params}

In addition to what is defined in {{Section 8 of RFC9594}}, this application profile defines additional parameters used during the second part of the message exchange with the Group Manager, i.e., after the exchange of Token Transfer Request and Response (see {{ssec-token-post}}). The table below summarizes them and specifies the CBOR key to use instead of the full descriptive name.

Note that the media type "application/ace-groupcomm+cbor" MUST be used when these parameters are transported in the respective message fields.

| Name           | CBOR Key | CBOR Type | Reference |
+----------------|----------|-----------|-----------|
| group_senderId | 21       | bstr      | {{&SELF}} |
+----------------|----------|-----------|-----------|
| ecdh_info      | 31       | array     | {{&SELF}} |
+----------------|----------|-----------|-----------|
| kdc_dh_creds   | 32       | array     | {{&SELF}} |
+----------------|----------|-----------|-----------|
| sign_enc_key   | 33       | bstr      | {{&SELF}} |
+----------------|----------|-----------|-----------|
| stale_node_ids | 34       | array     | {{&SELF}} |
{: #tab-ACE-Groupcomm-Parameters title="ACE Groupcomm Parameters" align="center"}

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

The Group Manager is expected to support all the parameters above. Instead, a Client is required to support the new parameters defined in this application profile as specified below (REQ29).

* 'group_senderId' MUST be supported by a Client that intends to join an OSCORE group with the role of Requester and/or Responder.

* 'ecdh_info' MUST be supported by a Client that intends to join a group which uses the pairwise mode of Group OSCORE.

* 'kdc_dh_creds' MUST be supported by a Client that intends to join a group which uses the pairwise mode of Group OSCORE and that does not plan to or cannot rely on an early retrieval of the Group Manager's Diffie-Hellman authentication credential.

* 'sign_enc_key' MUST be supported by a Client that intends to join a group which uses the group mode of Group OSCORE or to be signature verifier for that group.

* 'stale_node_ids' MUST be supported.

When the conditional parameters defined in {{Section 8 of RFC9594}} are used with this application profile, a Client must, should, or may support them as specified below (REQ30).

* 'client_cred' and 'client_cred_verify'. A Client that has an own authentication credential to use in a group MUST support these parameters.

* 'kdcchallenge'. A Client that has an own authentication credential to use in a group and that provides the access token to the Group Manager through a Token Transfer Request (see {{ssec-token-post}}) MUST support this parameter.

* 'creds_repo'. This parameter is not relevant for this application profile, since the Group Manager always acts as repository of the group members' authentication credentials.

* 'group_policies'. A Client that is interested in the specific policies used in a group, but that does not know them or cannot become aware of them before joining that group, SHOULD support this parameter.

* 'peer_roles'. A Client MUST support this parameter, since in this application profile it is relevant for Clients to know the roles of the group member associated with each authentication credential.

* 'kdc_nonce', 'kdc_cred', and 'kdc_cred_verify'. A Client MUST support these parameters, since the Group Manager's authentication credential is required to process messages protected with Group OSCORE (see Section 4.3 of {{I-D.ietf-core-oscore-groupcomm}}).

* 'mgt_key_material'. A Client that supports an advanced rekeying scheme possibly used in the group, such as based on one-to-many rekeying messages sent by the Group Manager (e.g., over IP multicast), MUST support this parameter.

* 'control_group_uri'. A Client that supports the hosting of local resources each associated with a group (hence acting as CoAP server) and the reception of one-to-many requests sent to those resources by the Group Manager (e.g., over IP
multicast) MUST support this parameter.

# ACE Groupcomm Error Identifiers {#error-types}

In addition to what is defined in {{Section 9 of RFC9594}}, this document defines new values that the Group Manager can use as error identifiers. These are used in error responses with Content-Format "application/concise-problem-details+cbor" {{RFC9290}}, as values of the 'error-id' field within the Custom Problem Detail entry 'ace- groupcomm-error' (see {{Section 4.1.2 of RFC9594}}).

| Value |                   Description                   |
|-------|-------------------------------------------------|
|   7   | Signatures not used in the group                |
|-------|-------------------------------------------------|
|   8   | Operation permitted only to signature verifiers |
|-------|-------------------------------------------------|
|   9   | Group currently not active                      |
{: #tab-ACE-Groupcomm-Error-Identifiers title="ACE Groupcomm Error Identifiers" align="center"}

If the Client supports the problem-details format {{RFC9290}} and the Custom Problem Detail entry 'ace-groupcomm-error' defined in {{Section 4.1.2 of RFC9594}}, and is able to understand the error specified in the 'error-id' field therein, then the Client may use that information to determine what actions to take next. If the Concise Problem Details data item specified in the error response includes the 'detail' entry and the Client supports it, such an entry may provide additional context.

* In case of error 7, the Client should stop sending the request in question to the Group Manager. In this application profile, this error is relevant only for a signature verifier, in case it tries to access resources related to a pairwise-only group.

* In case of error 8, the Client should stop sending the request in question to the Group Manager.

* In case of error 9, the Client should wait for a certain (pre-configured) amount of time, before trying to re-send its request to the Group Manager.

# Default Values for Group Configuration Parameters

This section defines the default values that the Group Manager assumes for the configuration parameters of an OSCORE group, unless differently specified when creating and configuring the group (for example, by means of the admin interface specified in {{I-D.ietf-ace-oscore-gm-admin}}).

A possible reason for the Group Manager to consider default values different from those recommended in this section is to ensure that each of those are consistent with what the Group Manager supports, e.g., in terms of signature algorithm and format of authentication credentials used in the OSCORE group.

This ensures that the Group Manager is able to perform the operations defined in this document, e.g., to achieve proof-of-possession of a joining node's private key (see {{ssec-join-req-processing}}), as well as to provide a joining node with its own authentication credential and the associated proof-of-possession challenge (see {{ssec-join-resp}}).

The following builds on the "COSE Header Parameters" registry {{COSE.Header.Parameters}}, the "COSE Algorithms" registry {{COSE.Algorithms}}, the "COSE Key Types" registry {{COSE.Key.Types}}, and the "COSE Elliptic Curves" registry {{COSE.Elliptic.Curves}}.

## Common

This section always applies, as related to common configuration parameters.

* For the HKDF Algorithm 'hkdf', the Group Manager SHOULD use HKDF SHA-256, defined as default in {{Section 3.2 of RFC8613}}. In the 'hkdf' parameter, this HKDF Algorithm is specified by the HMAC Algorithm HMAC 256/256 (COSE algorithm encoding: 5).

* For the Authentication Credential Format 'cred_fmt', the Group Manager SHOULD use CBOR Web Token Claims Set (CCS) {{RFC8392}}, i.e., the COSE Header Parameter 'kccs' (COSE header parameter encoding: 14).

* For 'max_stale_sets', the Group Manager SHOULD consider N = 3 as the maximum number of stored sets of stale Sender IDs for the group (see {{sssec-stale-sender-ids}}).

## Group Mode

This section applies if the group uses (also) the group mode of Group OSCORE.

* For the Group Encryption Algorithm 'gp_enc_alg' used to encrypt messages protected with the group mode, the Group Manager SHOULD use AES-CCM-16-64-128 (COSE algorithm encoding: 10).

* For the Signature Algorithm 'sign_alg' used to sign messages protected with the group mode, the Group Manager SHOULD use EdDSA {{RFC8032}}.

* For the parameters 'sign_params' of the Signature Algorithm, the Group Manager SHOULD use the following:

    - The array \[\[OKP\], \[OKP, Ed25519\]\], in case EdDSA is assumed or specified for 'sign_alg'. This indicates to use the COSE key type OKP and the elliptic curve Ed25519 {{RFC8032}}.

    - The array \[\[EC2\], \[EC2, P-256\]\], in case ES256 {{RFC6979}} is specified for 'sign_alg'. This indicates to use the COSE key type EC2 and the elliptic curve P-256.

    - The array \[\[EC2\], \[EC2, P-384\]\], in case ES384 {{RFC6979}} is specified for 'sign_alg'. This indicates to use the COSE key type EC2 and the elliptic curve P-384.

    - The array \[\[EC2\], \[EC2, P-521\]\], in case ES512 {{RFC6979}} is specified for 'sign_alg'. This indicates to use the COSE key type EC2 and the elliptic curve P-521.

    - The array \[\[RSA\], \[RSA\]\], in case PS256, PS384, or PS512 {{RFC8017}} is specified for 'sign_alg'. This indicates to use the COSE key type RSA.

## Pairwise Mode

This section applies if the group uses (also) the pairwise mode of Group OSCORE.

* For the AEAD Algorithm 'alg' used to encrypt messages protected with the pairwise mode, the Group Manager SHOULD use the same default value defined in {{Section 3.2 of RFC8613}}, i.e., AES-CCM-16-64-128 (COSE algorithm encoding: 10).

* For the Pairwise Key Agreement Algorithm 'ecdh_alg' used to compute static-static Diffie-Hellman shared secrets, the Group Manager SHOULD use the following:

  - The ECDH algorithm ECDH-SS + HKDF-256 (COSE algorithm encoding: -27), in case the HKDF Algorithm assumed or specified for 'hkdf' is HKDF SHA-256 (specified by the HMAC Algorithm HMAC 256/256).

  - The ECDH algorithm ECDH-SS + HKDF-512 (COSE algorithm encoding: -28), in case the HKDF Algorithm specified for 'hkdf' is HKDF SHA-512 (specified by the HMAC Algorithm HMAC HMAC 512/512).

* For the parameter 'ecdh_params' of the Pairwise Key Agreement Algorithm, the Group Manager SHOULD use the following:

    - The array \[\[OKP\], \[OKP, X25519\]\], in case EdDSA is assumed or specified for 'sign_alg', or in case the group is a pairwise-only group. This indicates to use the COSE key type OKP and the elliptic curve X25519 {{RFC8032}}.

    - The array \[\[EC2\], \[EC2, P-256\]\], in case ES256 {{RFC6979}} is specified for 'sign_alg'. This indicates to use the COSE key type EC2 and the elliptic curve P-256.

    - The array \[\[EC2\], \[EC2, P-384\]\], in case ES384 {{RFC6979}} is specified for 'sign_alg'. This indicates to use the COSE key type EC2 and the elliptic curve P-384.

    - The array \[\[EC2\], \[EC2, P-521\]\], in case ES512 {{RFC6979}} is specified for 'sign_alg'. This indicates to use the COSE key type EC2 and the elliptic curve P-521.

# Security Considerations {#sec-security-considerations}

Security considerations for this profile are inherited from {{RFC9594}}, the ACE framework for Authentication and Authorization {{RFC9200}}, and the specific transport profile of ACE signaled by the AS, such as {{RFC9202}} and {{RFC9203}}.

The following security considerations also apply for this profile.

## Management of OSCORE Groups {#ssec-security-considerations-management}

This profile leverages the following management aspects related to OSCORE groups and discussed in the sections of {{I-D.ietf-core-oscore-groupcomm}} referred below.

* Management of group keying material (see {{Section 12.2 of I-D.ietf-core-oscore-groupcomm}}). The Group Manager is responsible for the renewal and re-distribution of the keying material in the groups of its competence (rekeying).

   The Group Manager performs a rekeying when one or more members leave the group, thus preserving forward security and ensuring that the security properties of Group OSCORE are fulfilled. According to the specific application requirements, the Group Manager can also rekey the group upon a new node's joining, in case backward security has also to be preserved.

* Provisioning and retrieval of authentication credentials (see {{Section 12 of I-D.ietf-core-oscore-groupcomm}}). The Group Manager acts as repository of authentication credentials of group members, and provides them upon request.

* Freshness of messages (see {{Section 5.2 of I-D.ietf-core-oscore-groupcomm}}). This concerns how a recipient node can assert freshness of messages received within the group.

Before sending the Join Response, the Group Manager MUST verify that the joining node actually owns the associated private key. To this end, the Group Manager relies on the proof-of-possession challenge-response defined in {{sec-joining-node-to-GM}}.

A node may have joined multiple OSCORE groups under different non-synchronized Group Managers. Therefore, it can happen that those OSCORE groups have the same Group Identifier (Gid). It follows that, upon receiving a Group OSCORE message addressed to one of those groups, the node would have multiple Security Contexts matching with the Gid in the incoming message. It is up to the application to decide how to handle such collisions of Group Identifiers, e.g., by trying to process the incoming message using one Security Context at the time until the right one is found.

## Size of Nonces as Proof-of-Possesion Challenge {#ssec-security-considerations-challenges}

With reference to the Join Request message in {{ssec-join-req-sending}}, the proof-of-possession (PoP) evidence included in 'client\_cred\_verify' is computed over an input including also N\_C \| N\_S, where \| denotes concatenation.

For the N\_C challenge, it is RECOMMENDED to use an 8-byte long random nonce. Furthermore, N\_C is always conveyed in the 'cnonce' parameter of the Join Request, which is always sent over the secure communication association between the joining node and the Group Manager.

As defined in {{sssec-challenge-value}}, the way the N\_S value is computed depends on the particular way the joining node provides the Group Manager with the access token, as well as on following interactions between the two.

* If the access token has not been provided to the Group Manager by means of a Token Transfer Request to the /authz-info endpoint as in {{ssec-token-post}}, then N\_S is computed as a 32-byte long challenge. For an example, see points (2) and (3) in {{sssec-challenge-value}}.

* If the access token has been provided to the Group Manager by means of a Token Transfer Request to the /authz-info endpoint as in {{ssec-token-post}}, then N\_S takes the most recent value provided to the Client by the Group Manager in the 'kdcchallenge' parameter, as specified in point (1) of {{sssec-challenge-value}}. This value is provided either in the Token Transfer Response (see {{ssec-token-post}}), or in a 4.00 (Bad Request) error response to a following Join Request (see {{ssec-join-req-processing}}). In either case, it is RECOMMENDED to use an 8-byte long random challenge as value for N\_S.

If we consider both N\_C and N\_S to take 8-byte long values, the following considerations hold.

* Let us consider both N\_C and N\_S as taking random values, and the Group Manager to never change the value of the N\_S provided to a Client during the lifetime of an access token. Then, as per the birthday paradox, the average collision for N\_S will happen after 2<sup>32</sup> new transferred access tokens, while the average collision for N\_C will happen after 2<sup>32</sup> new Join Requests. This amounts to considerably more token provisionings than the expected new joinings to OSCORE groups under a same Group Manager, as well as to considerably more requests to join OSCORE groups from a same Client using a same access token under a same Group Manager.

* {{Section 7 of RFC9203}} as well {{Section B.2 of RFC8613}} recommend the use of 8-byte random values as well. Unlike in those cases, the values of N\_C and N\_S considered in this document are not used for as sensitive operations as the derivation of a Security Context, and thus do not have possible implications in the security of AEAD ciphers.

## Reusage of Nonces for Proof-of-Possession Input {#ssec-security-considerations-reusage-nonces}

As long as the Group Manager preserves the same N\_S value currently associated with an access token, i.e., the latest value provided to a Client in a 'kdcchallenge' parameter, the Client is able to successfully reuse the same proof-of-possession (PoP) input for multiple Join Requests to that Group Manager.

In particular, the Client can reuse the same N\_C value for every Join Request to the Group Manager, and combine it with the same unchanged N\_S value. This results in reusing the same PoP input for producing the PoP evidence to include in the 'client_cred_verify' parameter of the Join Requests.

Unless the Group Manager maintains a list of N\_C values already used by that Client since the latest update to the N\_S value associated with the access token, the Group Manager can be forced to falsely believe that the Client possesses its own private key at that point in time, upon verifying the PoP evidence in the 'client_cred_verify' parameter.

# IANA Considerations {#sec-iana}

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

## OAuth Parameters {#iana-kinfo}

IANA is asked to register the following entries in the "OAuth Parameters" registry, following the procedure specified in {{Section 11.2 of RFC6749}}.

* Name: ecdh_info
* Parameter Usage Location: client-rs request, rs-client response
* Change Controller: IETF
* Reference: {{&SELF}}

<br>

* Name: kdc_dh_creds
* Parameter Usage Location: client-rs request, rs-client response
* Change Controller: IETF
* Reference: {{&SELF}}

## OAuth Parameters CBOR Mappings {#iana-kinfo-map}

IANA is asked to register the following entries in the "OAuth Parameters CBOR Mappings" registry, following the procedure specified in {{Section 8.10 of RFC9200}}.

* Name: ecdh_info
* CBOR Key: TBD (range -256 to 255)
* Value Type: Null or array
* Reference: {{&SELF}}
* Original Specification: {{&SELF}}

<br>

* Name: kdc_dh_creds
* CBOR Key: TBD (range -256 to 255)
* Value Type: Null or array
* Reference: {{&SELF}}
* Original Specification: {{&SELF}}

## ACE Groupcomm Parameters {#ssec-iana-ace-groupcomm-parameters-registry}

IANA is asked to register the following entries to the "ACE Groupcomm Parameters" registry defined in {{Section 11.7 of RFC9594}}.

* Name: group_senderId
* CBOR Key: 21 (suggested)
* CBOR Type: bstr
* Reference: {{&SELF}}

<br>

* Name: ecdh_info
* CBOR Key: 31 (suggested)
* CBOR Type: array
* Reference: {{&SELF}}

<br>

* Name: kdc_dh_creds
* CBOR Key: 32 (suggested)
* CBOR Type: array
* Reference: {{&SELF}}

<br>

* Name: sign_enc_key
* CBOR Key: 33 (suggested)
* CBOR Type: bstr
* Reference: {{&SELF}}

<br>

* Name: stale_node_ids
* CBOR Key: 34 (suggested)
* CBOR Type: array
* Reference: {{&SELF}}

## ACE Groupcomm Key Types {#ssec-iana-groupcomm-keys-registry}

IANA is asked to register the following entry in the "ACE Groupcomm Key Types" registry defined in {{Section 11.8 of RFC9594}}.

*  Name: Group_OSCORE_Input_Material object
*  Key Type Value: GROUPCOMM_KEY_TBD
*  Profile: "coap_group_oscore_app", defined in {{ssec-iana-groupcomm-profile-registry}} of this document.
*  Description: A Group_OSCORE_Input_Material object encoded as described in {{ssec-join-resp}} of this document.
*  Reference: {{&SELF}}

## ACE Groupcomm Profiles {#ssec-iana-groupcomm-profile-registry}

IANA is asked to register the following entry in the "ACE Groupcomm Profiles" registry defined in {{Section 11.9 of RFC9594}}.

*  Name: coap_group_oscore_app
*  Description: Application profile to provision keying material for participating in group communication protected with Group OSCORE as per {{I-D.ietf-core-oscore-groupcomm}}.
*  CBOR Value: PROFILE_TBD
*  Reference: {{&SELF}}

## OSCORE Security Context Parameters {#ssec-iana-security-context-parameter-registry}

IANA is asked to register the following entries in the "OSCORE Security Context Parameters" registry defined in {{Section 9.4 of RFC9203}}.

*  Name: group_SenderId
*  CBOR Label: 7 (suggested)
*  CBOR Type: byte string
*  Registry: -
*  Description: OSCORE Sender ID assigned to a member of an OSCORE group
*  Reference: {{&SELF}}

<br>

*  Name: cred_fmt
*  CBOR Label: 8 (suggested)
*  CBOR Type: integer
*  Registry: {{COSE.Header.Parameters}} Labels (integer)
*  Description: Format of authentication credentials to be used in the OSCORE group
*  Reference: {{&SELF}}

<br>

*  Name: gp_enc_alg
*  CBOR Label: 9 (suggested)
*  CBOR Type: text string / integer
*  Registry: {{COSE.Algorithms}} Values
*  Description: OSCORE Group Encryption Algorithm Value
*  Reference: {{&SELF}}

<br>

*  Name: sign_alg
*  CBOR Label: 10 (suggested)
*  CBOR Type: text string / integer
*  Registry: {{COSE.Algorithms}} Values
*  Description: OSCORE Signature Algorithm Value
*  Reference: {{&SELF}}

<br>

*  Name: sign_params
*  CBOR Label: 11 (suggested)
*  CBOR Type: array
*  Registry: {{COSE.Algorithms}} Capabilities, {{COSE.Key.Types}} Capabilities, {{COSE.Elliptic.Curves}} Values
*  Description: OSCORE Signature Algorithm Parameters
*  Reference: {{&SELF}}

<br>

*  Name: ecdh_alg
*  CBOR Label: 12 (suggested)
*  CBOR Type: text string / integer
*  Registry: {{COSE.Algorithms}} Values
*  Description: OSCORE Pairwise Key Agreement Algorithm Value
*  Reference: {{&SELF}}

<br>

*  Name: ecdh_params
*  CBOR Label: 13 (suggested)
*  CBOR Type: array
*  Registry: {{COSE.Algorithms}} Capabilities, {{COSE.Key.Types}} Capabilities, {{COSE.Elliptic.Curves}} Values
*  Description: OSCORE Pairwise Key Agreement Algorithm Parameters
*  Reference: {{&SELF}}

## TLS Exporter Labels {#ssec-iana-tls-esporter-label-registry}

IANA is asked to register the following entry in the "TLS Exporter Labels" registry defined in {{Section 6 of RFC5705}} and updated in {{Section 12 of RFC8447}}.

* Value: EXPORTER-ACE-Pop-Input-coap-group-oscore-app
* DTLS-OK: Y
* Recommended: N
* Reference: {{&SELF}}

## AIF Media-Type Sub-Parameters {#ssec-iana-AIF-registry}

For the media-types application/aif+cbor and application/aif+json defined in {{Section 5.1 of RFC9237}}, IANA is requested to register the following entries for the two media-type parameters Toid and Tperm, in the respective sub-registry defined in {{Section 5.2 of RFC9237}} within the "MIME Media Type Sub-Parameter" registry group.

* Parameter: Toid
* Name: oscore-gname
* Description/Specification: OSCORE group name
* Reference: {{&SELF}}

<br>

* Parameter: Tperm
* Name: oscore-gperm
* Description/Specification: permissions pertaining OSCORE groups
* Reference: {{&SELF}}

## CoAP Content-Formats {#ssec-iana-coap-content-format-registry}

IANA is asked to register the following entries in the "CoAP Content-Formats" registry within the "Constrained RESTful Environments (CoRE) Parameters" registry group.

* Content Type: application/aif+cbor;Toid="oscore-gname",Tperm="oscore-gperm"

* Content Coding: -

* ID: 292 (suggested)

* Reference: {{&SELF}}

<br>

* Content Type: application/aif+json;Toid="oscore-gname",Tperm="oscore-gperm"

* Content Coding: -

* ID: 293 (suggested)

* Reference: {{&SELF}}

## CoRE Resource Type # {#iana-rt}

IANA is asked to register the following entry in the "Resource Type (rt=) Link Target Attribute Values" registry within the "Constrained Restful Environments (CoRE) Parameters" registry group.

* Value: "core.osc.gm"

* Description: Group-membership resource of an OSCORE Group Manager.

* Reference: {{&SELF}}

Client applications can use this resource type to discover a group-membership resource at an OSCORE Group Manager, where to send a request for joining the corresponding OSCORE group.

## ACE Groupcomm Errors {#iana-ace-groupcomm-errors}

IANA is asked to register the following entries in the "ACE Groupcomm Errors" registry defined in {{Section 11.12 of RFC9594}}.

* Value: 7 (suggested)

* Description: Signatures not used in the group.

* Reference: {{&SELF}}

<br>

* Value: 8 (suggested)

* Description: Operation permitted only to signature verifiers.

* Reference: {{&SELF}}

<br>

* Value: 9 (suggested)

* Description: Group currently not active.

* Reference: {{&SELF}}

## Group OSCORE Roles {#ssec-iana-group-oscore-roles-registry}

This document establishes the IANA "Group OSCORE Roles" registry. The registry has been created to use the "Expert Review" registration procedure {{RFC8126}}. Expert review guidelines are provided in {{ssec-iana-expert-review}}.

This registry includes the possible roles that nodes can take in an OSCORE group, each in combination with a numeric identifier. These numeric identifiers are used to express authorization information about joining OSCORE groups, as specified in {{sec-format-scope}} of {{&SELF}}.

The columns of this registry are:

* Name: A value that can be used in documents for easier comprehension, to identify a possible role that nodes can take in an OSCORE group.

* Value: The numeric identifier for this role. Integer values greater than 65535 are marked as "Private Use", all other values use the registration policy "Expert Review" {{RFC8126}}.

* Description: This field contains a brief description of the role.

* Reference: This contains a pointer to the public specification for the role.

This registry will be initially populated by the values in {{tab-role-values}}. The Reference column for all of these entries will be {{&SELF}}.

## Expert Review Instructions {#ssec-iana-expert-review}

The IANA registry established in this document is defined as "Expert Review".  This section gives some general guidelines for what the experts should be looking for, but they are being designated as experts for a reason so they should be given substantial latitude.

Expert reviewers should take into consideration the following points:

* Clarity and correctness of registrations. Experts are expected to check the clarity of purpose and use of the requested entries. Experts should inspect the entry for the considered role, to verify the correctness of its description against the role as intended in the specification that defined it. Experts should consider requesting an opinion on the correctness of registered parameters from the Authentication and Authorization for Constrained Environments (ACE) Working Group and the Constrained RESTful Environments (CoRE) Working Group.

  Entries that do not meet these objectives of clarity and completeness should not be registered.

* Duplicated registration and point squatting should be discouraged. Reviewers are encouraged to get sufficient information for registration requests to ensure that the usage is not going to duplicate one that is already registered and that the point is likely to be used in deployments.

* Experts should take into account the expected usage of roles when approving point assignments. Given a 'Value' V as code point, the length of the encoding of (2<sup>V+1</sup> - 1) should be weighed against the usage of the entry, considering the resources and capabilities of devices it will be used on. Additionally, given a 'Value' V as code point, the length of the encoding of (2<sup>V+1</sup> - 1) should be weighed against how many code points resulting in that encoding length are left, and the resources and capabilities of devices it will be used on.

* Specifications are recommended. When specifications are not provided, the description provided needs to have sufficient information to verify the points above.

--- back

# Profile Requirements # {#profile-req}

This section lists how this application profile of ACE addresses the requirements defined in {{Section A of RFC9594}}.

## Mandatory-to-Address Requirements

* REQ1: Specify the format and encoding of scope. This includes defining the set of possible roles and their identifiers, as well as the corresponding encoding to use in the scope entries according to the used scope format: see {{sec-format-scope}} and {{ssec-auth-req}}.

* REQ2: If scope uses AIF, register its specific instance of "Toid" and "Tperm" as media type parameters and a corresponding Content-Format, as per the guidelines in {{RFC9237}}: see {{ssec-iana-AIF-registry}} and {{ssec-iana-coap-content-format-registry}}.

* REQ3: If used, specify the acceptable values for the 'sign_alg' parameter: values from the "Value" column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

* REQ4: If used, specify the acceptable values and structure for the 'sign_parameters' parameter: values and structure from the COSE algorithm capabilities as specified in the "COSE Algorithms" registry {{COSE.Algorithms}}.

* REQ5: If used, specify the acceptable values and structure for the 'sign_key_parameters' parameter: values and structure from the COSE key type capabilities as specified in the "COSE Key Types" registry {{COSE.Key.Types}}.

* REQ6: Specify the acceptable formats for authentication credentials and, if applicable, the acceptable values for the 'cred_fmt' parameter: acceptable formats explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm (see {{ssec-token-post}} and {{ssec-join-resp}}). Consistent acceptable values for 'cred_fmt' are taken from the "Label" column of the "COSE Header Parameters" registry {{COSE.Header.Parameters}}, with some of those values also indicating the type of container to use for exchanging the authentication credentials with the Group Manager (e.g., a chain or bag of certificates).

* REQ7: If the value of the GROUPNAME URI path and the group name in the access token scope ('gname') are not required to coincide, specify the mechanism to map the GROUPNAME value in the URI to the group name: not applicable, since a perfect matching is required.

* REQ8: Define whether the KDC has an authentication credential as required for the correct group operation and if this has to be provided through the 'kdc_cred' parameter: yes, as required by the Group OSCORE protocol {{I-D.ietf-core-oscore-groupcomm}}, see {{ssec-join-resp}} of this document.

* REQ9: Specify if any part of the KDC interface as defined in {{RFC9594}} is not supported by the KDC: not applicable.

* REQ10: Register a Resource Type for the group-membership resources, which is used to discover the correct URL for sending a Join Request to the KDC: the Resource Type (rt=) Link Target Attribute value "core.osc.gm" is registered in {{iana-rt}}.

* REQ11: Define what specific actions (e.g., CoAP methods) are allowed on each resource accessible through the KDC interface, depending on: whether the Client is a current group member; the roles that a Client is authorized to take as per the obtained access token; and the roles that the Client has as a current group member: see {{ssec-admitted-methods}}.

* REQ12: Categorize possible newly defined operations for Clients into primary operations expected to be minimally supported and secondary operations, and provide accompanying considerations: see {{client-operations}}.

* REQ13: Specify the encoding of group identifiers: CBOR byte string (see {{sec-retrieve-gnames}}).

* REQ14: Specify the approaches used to compute and verify the PoP evidence to include in the 'client_cred_verify' parameter and which of those approaches is used in which case: see {{ssec-join-req-sending}} and {{ssec-join-req-processing}}.

* REQ15: Specify how the nonce N_S is generated, if the access token is not provided to the KDC through the Token Transfer Request sent to the /authz-info endpoint (e.g., the access token is instead transferred during the establishment of a secure communication association): see {{sssec-challenge-value}}.

* REQ16: Define the initial value of the version number for the group keying material: the initial value MUST be set to 0 when creating the OSCORE group, e.g., as in {{I-D.ietf-ace-oscore-gm-admin}}.

* REQ17: Specify the format of the group keying material that is conveyed in the 'key' parameter: see {{ssec-join-resp}}.

* REQ18: Specify the acceptable values of the 'gkty' parameter. For each of them, register a corresponding entry in the "ACE Groupcomm Key Types" IANA registry if such an entry does not exist already: Group_OSCORE_Input_Material object (see {{ssec-join-resp}}).

* REQ19: Specify and register the application profile identifier: coap_group_oscore_app (see {{ssec-iana-groupcomm-profile-registry}}).

* REQ20: If used, specify the format and default values of the entries of the CBOR map to include in the 'group_policies' parameter: see {{ssec-join-resp}}.

* REQ21: Specify the approaches used to compute and verify the PoP evidence to include in the 'kdc_cred_verify' parameter and which of those approaches is used in which case. If external signature verifiers are supported, specify how those provide a nonce to the KDC to be used for computing the PoP evidence: see {{ssec-join-resp}}, {{ssec-join-resp-processing}} and {{sec-gm-pub-key-signature-verifier}}.

* REQ22: Specify the communication protocol that members of the group use to communicate with each other (e.g., CoAP for group communication): CoAP {{RFC7252}}, also for group communication {{I-D.ietf-core-groupcomm-bis}}.

* REQ23: Specify the security protocol that members of the group use to protect the group communication (e.g., Group OSCORE). This must provide encryption, integrity, and replay protection: Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}.

* REQ24: Specify how the communication is secured between the Client and the KDC. Optionally, specify a transport profile of ACE {{RFC9200}} to use between the Client and the KDC: by means of any transport profile of ACE {{RFC9200}} between Client and Group Manager that complies with the requirements in {{Section C of RFC9200}}.

* REQ25: Specify the format of the node identifiers of group members: the Sender ID used in the OSCORE group (see {{ssec-join-resp}} and {{sec-pub-keys}}).

* REQ26: Specify policies at the KDC to handle node identifiers that are included in the 'get_creds' parameter but are not associated with any current group member: see {{sec-pub-keys}}.

* REQ27: Specify the format of (newly generated) individual keying material for group members or of the information to derive such keying material, as well as the corresponding CBOR map key that has to be registered in the "ACE Groupcomm Parameters" registry: see {{sec-new-key}}.

* REQ28: Specify which CBOR tag is used for identifying the semantics of binary scopes, or register a new CBOR tag if a suitable one does not exist already: see {{ssec-auth-resp}}.

* REQ29: Categorize newly defined parameters according to the same criteria of {{Section 8 of RFC9594}}: see {{ace-groupcomm-params}}.

* REQ30: Define whether Clients must, should, or may support the conditional parameters defined in {{Section 8 of RFC9594}} and under which circumstances: see {{ace-groupcomm-params}}.

## Optional-to-Address Requirements

* OPT1: Optionally, if the textual format of scope is used, specify CBOR values to use for abbreviating the role identifiers in the group: not applicable.

* OPT2: Optionally, specify the additional parameters used in the exchange of Token Transfer Request and Response:

   - 'ecdh_info', to negotiate the ECDH algorithm, ECDH algorithm parameters, ECDH key parameters, and exact format of authentication credentials used in the group, in case the joining node supports the pairwise mode of Group OSCORE (see {{ssec-token-post}}).

   - 'kdc_dh_creds', to ask for and retrieve the Group Manager's Diffie-Hellman authentication credentials, in case the joining node supports the pairwise mode of Group OSCORE and the access token authorizes to join pairwise-only groups (see {{ssec-token-post}}).

* OPT3: Optionally, specify the negotiation of parameter values for signature algorithm and signature keys, if the 'sign_info' parameter is not used: possible early discovery by using the approach based on the CoRE Resource Directory described in {{I-D.tiloca-core-oscore-discovery}}.

* OPT4: Optionally, specify possible or required payload formats for specific error cases: send a 4.00 (Bad Request) error response to a Join Request (see {{ssec-join-req-processing}}).

* OPT5: Optionally, specify additional identifiers of error types as values of the 'error-id' field within the Custom Problem Detail entry 'ace-groupcomm-error': see {{iana-ace-groupcomm-errors}}.

* OPT6: Optionally, specify the encoding of the 'creds_repo' parameter if the default one is not used: no.

* OPT7: Optionally, specify the functionalities implemented at the resource hosted by the Client at the URI indicated in the 'control_uri' parameter, including the encoding of exchanged messages and other details: see {{sec-leaving}} for the eviction of a group member; see {{sec-group-rekeying-process}} for the group rekeying process.

* OPT8: Optionally, specify the behavior of the POST handler of group-membership resources, for the case when it fails to retrieve an authentication credential for the specific Client: send a 4.00 (Bad Request) error response to a Join Request (see {{ssec-join-req-processing}}).

* OPT9: Optionally, define a default group rekeying scheme to refer to in case the 'rekeying_scheme' parameter is not included in the Join Response: the "Point-to-Point" rekeying scheme registered in {{Section 11.13 of RFC9594}}, whose detailed use for this profile is defined in {{sec-group-rekeying-process}} of this document.

* OPT10: Optionally, specify the functionalities implemented at the resource hosted by the Client at the URI indicated in the 'control_group_uri' parameter, including the encoding of exchanged messages and other details: see {{sec-leaving}} for the eviction of multiple group members.

* OPT11: Optionally, specify policies that instruct Clients to retain messages and for how long, if those are unsuccessfully decrypted: no.

* OPT12: Optionally, specify for the KDC to perform a group rekeying when receiving a Key Renewal Request, together with or instead of renewing individual keying material: the Group Manager SHOULD NOT perform a group rekeying, unless already scheduled to occur shortly (see {{sec-new-key}}).

* OPT13: Optionally, specify how the identifier of a group member's authentication credential is included in requests sent to other group members: no.

* OPT14: Optionally, specify additional information to include in rekeying messages for the "Point-to-Point" group rekeying scheme (see {{Section 6 of RFC9594}}): see {{sending-rekeying-msg}}.

# Extensibility for Future COSE Algorithms # {#sec-future-cose-algs}

As defined in {{Section 8.1 of RFC9053}}, future algorithms can be registered in the "COSE Algorithms" registry {{COSE.Algorithms}} as specifying none or multiple COSE capabilities.

To enable the seamless use of such future registered algorithms, this section defines a general, agile format for:

* Each 'ecdh_info_entry' of the 'ecdh_info' parameter in the Token Transfer Response; see {{ecdh-info}}).

  {{Section B of RFC9594}} describes the analogous general format for each 'sign_info_entry' of the 'sign_info' parameter in the Token Transfer Response (see {{ssec-token-post}} of this document).

* The 'sign_params' and 'ecdh_params' parameters within the 'key' parameter (see {{ssec-join-resp}}), as part of the response payloads used in {{ssec-join-resp}}, {{ssec-updated-key-only}}, {{ssec-updated-and-individual-key}}, and {{sec-group-rekeying-process}}.

If any of the currently registered COSE algorithms is considered, using this general format yields the same structure defined in this document for the items above, thus ensuring backward compatibility.

## Format of 'ecdh_info_entry' ## {#sec-future-cose-algs-ecdh-info-entry}

The format of each 'ecdh_info_entry' (see {{ssec-token-post}} and {{ecdh-info}}) is generalized as follows.

* 'ecdh_parameters' includes N >= 0 elements, each of which is a COSE capability of the ECDH algorithm indicated in 'ecdh_alg'.

  In particular, 'ecdh_parameters' has the same format and value of the COSE capabilities array for the ECDH algorithm indicated in 'ecdh_alg', as specified for that algorithm in the 'Capabilities' column of the "COSE Algorithms" registry {{COSE.Algorithms}}.

* 'ecdh_key_parameters' is replaced by N elements 'ecdh_capab', each of which is a CBOR array.

  The i-th 'ecdh_capab' array (i = 0, ..., N-1) is the array of COSE capabilities for the algorithm capability specified in 'ecdh_parameters'\[i\].

  In particular, each 'ecdh_capab' array has the same format and value of the COSE capabilities array for the algorithm capability specified in 'ecdh_parameters'\[i\].

  Such a COSE capabilities array is currently defined for the algorithm capability COSE key type, in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}}.

The CDDL notation {{RFC8610}} of the 'ecdh_info_entry' parameter is given below.

~~~~~~~~~~~ CDDL
ecdh_info_entry =
[
    id: gname / [+ gname],
    ecdh_alg: int / tstr,
    ecdh_parameters : [* ecdh_capab: any],
  * ecdh_capab: [* capab: any],
    cred_fmt: int / null
]

gname = tstr
~~~~~~~~~~~
{: #fig-ecdh-info-entry-general title="'ecdh_info_entry' with General Format"}

## Format of 'key' ## {#sec-future-cose-algs-key}

The format of 'key' (see {{ssec-join-resp}}) is generalized as follows.

* The 'sign_params' array includes N+1 elements, whose exact structure and value depend on the value of the signature algorithm specified in 'sign_alg'.

   - The first element, i.e., 'sign_params'\[0\], is the array of the N COSE capabilities for the signature algorithm, as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{COSE.Algorithms}} (see {{Section 8.1 of RFC9053}}).

   - Each following element 'sign_params'\[i\], i.e., with index i > 0, is the array of COSE capabilities for the algorithm capability specified in 'sign_params'\[0\]\[i-1\].

   For example, if 'sign_params'\[0\]\[0\] specifies the key type as capability of the algorithm, then 'sign_params'\[1\] is the array of COSE capabilities for the COSE key type associated with the signature algorithm, as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}} (see {{Section 8.2 of RFC9053}}).

* The 'ecdh_params' array includes M+1 elements, whose exact structure and value depend on the value of the ECDH algorithm specified in 'ecdh_alg'.

   - The first element, i.e., 'ecdh_params'\[0\], is the array of the M COSE capabilities for the ECDH algorithm, as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{COSE.Algorithms}} (see {{Section 8.1 of RFC9053}}).

   - Each following element 'ecdh_params'\[i\], i.e., with index i > 0, is the array of COSE capabilities for the algorithm capability specified in 'ecdh_params'\[0\]\[i-1\].

   For example, if 'ecdh_params'\[0\]\[0\] specifies the key type as capability of the algorithm, then 'ecdh_params'\[1\] is the array of COSE capabilities for the COSE key type associated with the ECDH algorithm, as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{COSE.Key.Types}} (see {{Section 8.2 of RFC9053}}).

# CDDL Model # {#sec-cddl-model}
{:removeinrfc}

~~~~~~~~~~~~~~~~~~~~ CDDL
; ACE Groupcomm Parameters
sign_enc_key = 21

; ACE Groupcomm Key Types
group_oscore_input_material_obj = 1

; ACE Groupcomm Profiles
coap_group_oscore_app = 1

; OSCORE Security Context Parameters
cred_fmt = 8
gp_enc_alg = 9
sign_alg = 10
sign_params = 11
~~~~~~~~~~~~~~~~~~~~
{: #fig-cddl-model title="CDDL Model" artwork-align="left"}

# Document Updates # {#sec-document-updates}
{:removeinrfc}

## Version -16 to -17 ## {#sec-16-17}

* CBOR diagnostic notation uses placeholders from a CDDL model.

* Fixes in the CDDL definitions.

* Fixes in the examples in CBOR diagnostic notation.

* Updated author list.

* Updated references and section numbers of referred documents.

* Use actual tables.

* Add high-level recap of the concept of scope.

* Fixed name of the error with error code 4.

* Renamed parameters to align with RFC 9594

  - "Group Encryption Key" becomes "Signature Encryption Key"

  - 'group_enc_key' becomes 'sign_enc_key'

  - "Signature Encryption Algorithm" becomes "Group Encryption Algorithm"

  - 'sign_enc_alg' becomes 'gp_enc_alg'

* Added CBOR integer abbreviations for ACE Groupcomm Parameters.

* Considerations on authentication credentials consistent with RFC 9594.

* Revised alternative computing of N_S challenge when DTLS is used.

* Generalized definition of ecdh_info.

* Generalized definition of kdc_dh_creds.

* Clarified maximum size of the OSCORE Sender ID.

* Clarified parameters left "not set" in the Security Context.

* Clarified meaning of 'cred_fmt'.

* Consistent mandatory use of 'cnonce'.

* Relation between 'cred_fmt' and Authentication Credential Format.

* Implicit PoP evidence of the Client's authentication credential.

* Process of 'client_cred' and 'client_cred_verify' consistent with RFC 9594.

* GET to ace-group/GROUPNAME/kdc-cred only for group members.

* Added FETCH handler for /ace-group/GROUPNAME/kdc-cred.

* PUT becomes POST for ace-group/GROUPNAME/nodes/NODENAME.

* Fixed error response code from /ace-group/GROUPNAME/nodes/NODENAME.

* Consistent use of the 'exi' ACE Groupcomm Parameter.

* Use concise problem details (RFC9290) for error responses.

* Revised default values on group configuration parameters.

* Revised future-ready generalization of 'ecdh_info_entry'.

* CCS is used as default format of authentication credential.

* Updated name of TLS exporter label.

* Revised IANA considerations.

* Aligned requirement formulation with that in RFC 9594.

* Use of AASVG in message diagrams.

* Clarifications and editorial fixes.

## Version -15 to -16 ## {#sec-15-16}

* Early mentioning of invalid combinations of roles.

* Revised presentation of handling of stale Sender IDs.

* Fixed CDDL notation.

* Fixed diagnostic notation in examples.

* Possible reason to deviate from default parameter values.

* Clarifications and editorial fixes.

## Version -14 to -15 ## {#sec-14-15}

* Alignment with renaming in draft-ietf-ace-key-groupcomm.

* Updated signaling of semantics for binary encoded scopes.

* Considered the upload of access tokens in the DTLS 1.3 Handshake.

* Fixes in IANA registrations.

* Editorial fixes.

## Version -13 to -14 ## {#sec-13-14}

* Major reordering of the document sections.

* The HKDF Algorithm is specified by the HMAC Algorithm.

* Group communication does not necessarily use IP multicast.

* Generalized AIF data model, also for draft-ace-oscore-gm-admin.

* Clarifications and editorial improvements.

## Version -12 to -13 ## {#sec-12-13}

* Renamed parameters about authentication credentials.

* It is optional for the Group Manager to reassign Gids by tracking "Birth Gids".

* Distinction between authentication credentials and public keys.

* Updated IANA considerations related to AIF.

* Updated textual description of registered ACE Scope Semantics value.

## Version -11 to -12 ## {#sec-11-12}

* Clarified semantics of 'ecdh_info' and 'kdc_dh_creds'.

* Definition of /ace-group/GROUPNAME/kdc-pub-key moved to draft-ietf-ace-key-groupcomm.

* /ace-group accessible also to non-members that are not Verifiers.

* Clarified what resources are accessible to Verifiers.

* Revised error handling for the newly defined resources.

* Revised use of CoAP error codes.

* Use of "Token Transfer Request" and "Token Transfer Response".

* New parameter 'rekeying_scheme'.

* Categorization of new parameters and inherited conditional parameters.

* Clarifications on what to do in case of enhanced error responses.

* Changed UCCS to CCS.

* Authentication credentials of just joined Clients can be in rekeying messages.

* Revised names of new IANA registries.

* Clarified meaning of registered CoRE resource type.

* Alignment to new requirements from draft-ietf-ace-key-groupcomm.

* Fixes and editorial improvements.

## Version -10 to -11 ## {#sec-10-11}

* Removed redundancy of key type capabilities, from 'sign_info', 'ecdh_info' and 'key'.

* New resource to retrieve the Group Manager's authentication credential.

* New resource to retrieve material for Signature Verifiers.

* New parameter 'gp_enc_alg' related to the group mode.

* 'cred_fmt' takes value from the COSE Header Parameters registry.

* Improved alignment of the Join Response payload with the Group OSCORE Security Context parameters.

* Recycling Group IDs by tracking "Birth GIDs".

* Error handling in case of non available Sender IDs upon joining.

* Error handling in case EdDSA public keys with invalid Y coordinate when the pairwise mode of Group OSCORE is supported.

* Generalized proof-of-possession (PoP) for the joining node's private key; defined Diffie-Hellman based PoP for OSCORE groups using only the pairwise mode.

* Proof-of-possession of the Group Manager's private key in the Join Response.

* Always use 'peer_identifiers' to convey Sender IDs as node identifiers.

* Stale Sender IDs provided when rekeying the group.

* New resource for late retrieval of stale Sender IDs.

* Added examples of message exchanges.

* Revised default values of group configuration parameters.

* Fixes to IANA registrations.

* General format of parameters related to COSE capabilities, supporting future registered COSE algorithms (new Appendix).

## Version -09 to -10 ## {#sec-09-10}

* Updated non-recycling policy of Sender IDs.

* Removed policies about Sender Sequence Number synchronization.

* 'control_path' renamed to 'control_uri'.

* Format of 'get_pub_keys' aligned with draft-ietf-ace-key-groupcomm.

* Additional way to inform of group eviction.

* Registered semantics identifier for extended scope format.

* Extended error handling, with error type specified in some error responses.

* Renumbered requirements.

## Version -08 to -09 ## {#sec-08-09}

* The url-path "ace-group" is used.

* Added overview of admitted methods on the Group Manager resources.

* Added exchange of parameters relevant for the pairwise mode of Group OSCORE.

* The signed value for 'client_cred_verify' includes also the scope.

* Renamed the key material object as Group_OSCORE_Input_Material object.

* Replaced 'clientId' with 'group_SenderId'.

* Added message exchange for Group Names request-response.

* No reassignment of Sender ID and Gid in the same OSCORE group.

* Updates on group rekeying contextual with request of new Sender ID.

* Signature verifiers can also retrieve Group Names and URIs.

* Removed group policy about supporting Group OSCORE in pairwise mode.

* Registration of the resource type rt="core.osc.gm".

* Update list of requirements.

* Clarifications and editorial revision.

## Version -07 to -08 ## {#sec-07-08}

* AIF data model to express scope entries.

* A set of roles is checked as valid when processing the Join Request.

* Updated format of 'get_pub_keys' in the Join Request.

* Payload format and default values of group policies in the Join Response.

* Updated payload format of the FETCH request to retrieve authentication credentials.

* Default values for group configuration parameters.

* IANA registrations to support the AIF data model.

## Version -06 to -07 ## {#sec-06-07}

* Alignments with draft-ietf-core-oscore-groupcomm.

* New format of 'sign_info', using the COSE capabilities.

* New format of Join Response parameters, using the COSE capabilities.

* Considerations on group rekeying.

* Editorial revision.

## Version -05 to -06 ## {#sec-05-06}

* Added role of external signature verifier.

* Parameter 'rsnonce' renamed to 'kdcchallenge'.

* Parameter 'kdcchallenge' may be omitted in some cases.

* Clarified difference between group name and OSCORE Gid.

* Removed the role combination \["requester", "monitor"\].

* Admit implicit scope and audience in the Authorization Request.

* New format for the 'sign_info' parameter.

* Scope not mandatory to include in the Join Request.

* Group policy about supporting Group OSCORE in pairwise mode.

* Possible individual rekeying of a single requesting node combined with a group rekeying.

* Security considerations on reusage of signature challenges.

* Addressing optional requirement OPT12 from draft-ietf-ace-key-groupcomm

* Editorial improvements.

## Version -04 to -05 ## {#sec-04-05}

* Nonce N\_S also in error responses to the Join Requests.

* Supporting single access token for multiple groups/topics.

* Supporting legal requesters/responders using the 'peer_roles' parameter.

* Registered and used dedicated label for TLS Exporter.

* Added method for uploading a new authentication credential to the Group Manager.

* Added resource and method for retrieving the current group status.

* Fixed inconsistency in retrieving group keying material only.

* Clarified retrieval of keying material for monitor-only members.

* Clarification on incrementing version number when rekeying the group.

* Clarification on what is re-distributed with the group rekeying.

* Security considerations on the size of the nonces used for the signature challenge.

* Added CBOR values to abbreviate role identifiers in the group.

## Version -03 to -04 ## {#sec-03-04}

* New abstract.

* Moved general content to draft-ietf-ace-key-groupcomm

* Terminology: node name; node resource.

* Creation and pointing at node resource.

* Updated Group Manager API (REST methods and offered services).

* Size of challenges 'cnonce' and 'rsnonce'.

* Value of 'rsnonce' for reused or non-traditionally-posted tokens.

* Removed reference to RFC 7390.

* New requirements from draft-ietf-ace-key-groupcomm

* Editorial improvements.

## Version -02 to -03 ## {#sec-02-03}

* New sections, aligned with the interface of ace-key-groupcomm .

* Exchange of information on the signature algorithm and related parameters, during the Token POST (Section 4.1).

* Nonce 'rsnonce' from the Group Manager to the Client (Section 4.1).

* Client PoP signature in the Key Distribution Request upon joining (Section 4.2).

* Local actions on the Group Manager, upon a new node's joining (Section 4.2).

* Local actions on the Group Manager, upon a node's leaving (Section 12).

* IANA registration in ACE Groupcomm Parameters registry.

* More fulfilled profile requirements (Appendix A).

## Version -01 to -02 ## {#sec-01-02}

* Editorial fixes.

* Changed: "listener" to "responder"; "pure listener" to "monitor".

* Changed profile name to "coap_group_oscore_app", to reflect it is an application profile.

* Added the 'type' parameter for all requests to a Join Resource.

* Added parameters to indicate the encoding of authentication credentials.

* Challenge-response for proof-of-possession of signature keys (Section 4).

* Renamed 'key_info' parameter to 'sign_info'; updated its format; extended to include also parameters of the signature key (Section 4.1).

* Code 4.00 (Bad request), in responses to joining nodes providing an invalid authentication credential (Section 4.3).

* Clarifications on provisioning and checking of authentication credentials (Sections 4 and 6).

* Extended discussion on group rekeying and possible different approaches (Section 7).

* Extended security considerations: proof-of-possession of signature keys; collision of OSCORE Group Identifiers (Section 8).

* Registered three entries in the IANA registry "Sequence Number Synchronization Method" (Section 9).

* Registered one public key encoding in the "ACE Public Key Encoding" IANA registry (Section 9).

## Version -00 to -01 ## {#sec-00-01}

* Changed name of 'req_aud' to 'audience' in the Authorization Request (Section 3.1).

* Added negotiation of signature algorithm/parameters between Client and Group Manager (Section 4).

* Updated format of the Key Distribution Response as a whole (Section 4.3).

* Added parameter 'cs_params' in the 'key' parameter of the Key Distribution Response (Section 4.3).

* New IANA registrations in the "ACE Authorization Server Request Creation Hints" registry, "ACE Groupcomm Key" registry, "OSCORE Security Context Parameters" registry and "ACE Groupcomm Profile" registry (Section 9).

# Acknowledgments {#sec-acknowledgments}
{: numbered="no"}

Jiye Park contributed as a co-author of initial versions of this document.

The authors sincerely thank {{{Christian Amsüss}}}, {{{Santiago Aragón}}}, {{{Stefan Beck}}}, {{{Carsten Bormann}}}, {{{Martin Gunnarsson}}}, {{{Rikard Höglund}}}, {{{Watson Ladd}}}, {{{Daniel Migault}}}, {{{Jim Schaad}}}, {{{Ludwig Seitz}}}, {{{Göran Selander}}}, and {{{Peter van der Stok}}} for their comments and feedback.

The work on this document has been partly supported by the Sweden's Innovation Agency VINNOVA and the Celtic-Next projects CRITISEC and CYPRESS; by the H2020 project SIFIS-Home (Grant agreement 952652); and by the EIT-Digital High Impact Initiative ACTIVE.

--- fluff
