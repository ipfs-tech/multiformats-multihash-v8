---
title: "The Multihash Data Format"
abbrev: "Multihash"
docname: draft-caballero-multiformats-multihash-latest
date:
category: std
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ART
workgroup: Multiformats (proposed)
keyword:
 - namespace governance
 - identifiers
 - sparkling distributed ledger
pi: [toc, tocindent, sort refs, symrefs, strict, compact, inline]
venue:
  type: Working Group
  github: ipfs-tech/multiformats-multihash-v8
author:
  - fullname: Juan Benet
    organization: Protocol Labs
    email: juan@protocol.ai
    uri: http://juan.benet.ai/
  - fullname: Manu Sporny
    organization: Digital Bazaar
    phone: +1 540 961 4469
    email: msporny@digitalbazaar.com
    uri: http://manu.sporny.org/
  - fullname: Juan Caballero
    organization: Interplanetary File System Foundation
    email: bumblefudge@ipfs.tech
    uri: https://ipfs.tech/
normative:
  RFC6234:
  RFC6920:
  RFC7693:
  RFC9562:
  FIPS202:
    title: SHA-3 Standard, Permutation-Based Hash and Extendable-Output Functions
    target: http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf
    date: 2015-08-01
informative:
  RFC6256: SDNV
  RFC8126:
  DWARF:
    title: DWARF Debugging Information Format
    target: http://dwarfstd.org/doc/Dwarf3.pdf
    date: 2005-12-01
---

--- abstract

Cryptographic hash functions often generate multiple output sizes and encodings.
This variability makes it difficult for applications to examine a series of
bytes and determine which hash function produced them, and thus such context is
traditionally passed alongside the resulting bytes in defined protocols.
Multihash inlines this context information so that it can travel and be
translated more easily, decoupled from specific protocols.

--- middle

# Feedback

This specification is a joint work product of [The IPFS
Foundation](https://ipfs.tech/) and the [W3C Credentials Community
Group](https://w3c-ccg.github.io/). Feedback related to this specification
should logged in the [issue tracker](https://github.com/ipfs-tech/multiformats-multihash-v8/issues)
and/or be sent to [Multiformats Mailing List at the
IETF](mailto:multiformats@ietfa.amsl.com).

# Introduction

Multihash responds to evolving design patterns in systems which depend on
cryptographically-secure hash functions, contributing to cryptographic agility
and allowing for easier translation (e.g. across multiple wire formats) within a
given system, and for ambient verifiability throughout a system, not just in the
context of protocols. To facilitate self-describing hashes rather than
context-bound ones, multihash inlines an identifier representing the hash
function used (and its configuration or auxiliary inputs) as a prefix before the
hash function output. This allows for cryptographic agility and provides a
valuable building block to content-addressing systems and URI-safety mechanisms
alike.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The Multihash Fields

A multihash follows the TLV (type-length-value) pattern and consists of several
fields composed of a combination of unsigned variable length integers and byte
information.

## Multihash Core Data Types

The following section details the core data types used by the Multihash
data format.

### Unsigned Variable Integer

A data type that enables one to express an unsigned integer of variable length.
The format uses the Unsigned Little Endian Base 128 (ULEB128) encoding that was
canonically defined in Appendix C of the DWARF Debugging Information Format
standard, initially released in 1993, and further specified in 2011 by IRTF
{{?RFC6256}} as Self-Delimiting Numeric Values or SDNVs.

As suggested by the name, this variable length encoding is only capable of
representing unsigned integers.  Further, while there is no theoretical maximum
integer value that can be represented by the format, implementations MUST NOT
encode more than nine (9) bytes giving a practical limit of integers in a range
between 0 and 2^63 - 1. When encoding an unsigned variable integer, the unsigned
integer is serialized seven bits at a time, starting with the least significant
bits. The most significant bit in each output byte indicates if there is a
continuation byte. It is not possible to express a signed integer with this data
type.

|Value|Encoding (bits)|hexadecimal notation|
|---|---|---|
|1|00000001|0x01|
|127|01111111|0x7F|
|128|10000000 00000001|0x8001|
|255|11111111 00000001|0xFF01|
|300|10101100 00000010|0xAC02|
|16384|10000000 10000000 00000001|0x808001|

Implementations MUST restrict the size of the varint to a max of nine bytes (63
bits). In order to avoid memory attacks on the encoding, the aforementioned
practical maximum length of nine bytes is used. There is no theoretical limit,
and future specs can grow this number if it is truly necessary to have code or
length values larger than 2^31.

## Multihash Fields

A multihash follows the TLV (type-length-value) pattern.

### Hash Function Identifier

The hash function identifier is an unsigned variable integer identifying the
hash function. Possible values for this field are provided in The Multihash
Identifier Registry (see IANA considerations below).

### Digest Length

The digest length is an unsigned variable integer counting the length of the
digest in bytes.

### Digest Value

The digest value is the hash function digest with a length of exactly what is
specified in the digest length, which is specified in bytes.

## A Multihash Example

For example, the following is an expression of a SHA2-256 hash in hexadecimal
notation (spaces added for readability purposes):

~~~~ bash
0x12 20 41dd7b6443542e75701aa98a0c235951a28a0d851b11564d20022ab11d2589a8
~~~~

The first byte (0x12) specifies the SHA2-256 hash function. The second byte
(0x20) specifies the length of the hash, which is 32 bytes. The rest of the data
specifies the value of the output of the hash function.

# Prior Art And Translation

In IETF's corpus of normative protocols, there are three partial overlaps of
problem space worth familiarizing oneself with to minimize collisions and
confusions:

* "Named Information Hash", specified in {{!RFC6920}}, defines an hierarchical
  URI scheme for content-identifiers, partitioned by enumerated hash functions.
  The [NIH registry][] at IANA contains all of these.
* UUIDv5, aka "Namespaced UUIDs", defined in {{!RFC9562}}
  [section 5.5](https://datatracker.ietf.org/doc/html/rfc9562#uuidv5), does the
  inverse, defining a universal namespace for one hash function, partitioned by
  the application of that function to multiple URI schemes (i.e. DNS names,
  valid URLs, etc.)
* The IANA [NIH registry][] has a similar shape and governance mode to the IANA
  [hashAlgorithm registry][] that TLS 1.2 implementations use to compactly
  signal supported hash+signature combinations. Since the former has different
  entries for some hash functions based on output length and the latter does
  not, the two registries are not alignable. However, given their different
  contexts, collisions between the two would not be a practical concern for
  users of either.

## Named Information Hash

The "Named Information Hash" URI scheme allows for minimally self-describing
hash strings to serve as content-identifiers for arbitrary binary inputs. This
lightweight identifier scheme is defined in {{!RFC6920}} and the supported
hash-context prefixes live in an IANA registry named
["https://www.iana.org/assignments/named-information/named-information.xhtml#hash-alg"](https://www.iana.org/assignments/named-information/named-information.xhtml#hash-alg).
Its syntactic similarity to HTTP headers and [support for MIME
content-types](https://datatracker.ietf.org/doc/html/rfc6920#section-3.1) makes
it potentially useful for web use-cases, but use-cases are not constrained by
URI scheme, only hinted at by the specification in sections 3 through 7.

### Translation from multihash to named-information hash

Some hash functions and output lengths specified in the Multihash registry below
correspond to the few entries in the smaller Named Information Hash registry,
leading to simple round-trip translations for multihashes produced by these
dual-registered hash functions.

Formatting a multihash with _any other_ multihash prefix as a Named Information
Hash (only useful, of course, for consumers supporting both formats) is
facilitated by a generic cross-registry tag for self-describing multihashes,
first proposed to the [NIH registry][] by [Appendix
B](https://www.ietf.org/archive/id/draft-multiformats-multihash-03.html#appendix-D.2)
in the 2021 internet-draft (v3) of this same document.

The translation is achieved thusly:

1. Strip the prefix bytes from the hash value and use the prefix bytes to identity the hash function used from the registry below.
2. If the multihash prefix corresponds to any tags in the [NIH registry][]:
   1. translate multicodec tag to NIH tag, i.e., if `0x12` (`sha2-256`) in
      `multicodec` registry, then `0x01` (`sha256`) in `named-information`
      registry
   2. transcode the hash value from "unsigned varint" to standard MSB binary
   3. (for binary form:) reattach new prefix to transcoded hash value
   4. (for ASCII form:) convert prefix to URL format, i.e., `ni:///sha-256;` for
      `0x01`, and reattach to base64-encoded transcoded hash value
3. If multihash prefix does NOT map cleanly to a registered value in [NIH registry][]:
   1. (for binary form:) prefix existing binary multihash with `0x42` to
      designate that what follows is a multicodec prefix followed by an ULEB128
      hash value.
   2. (for ASCII form:) convert the `0x42` prefix to URL format, i.e.,
      `ni:///mh;` and then append a base64url, no-padding encoding of the entire
      binary multihash with prefix (and _without_ adding the additional
      base-64-url-no-padding prefix, `u`, if using a [multibase][] library for
      this base-encoding).

## Using Multihashes as Namespaced UUIDs

Since the "Named Information Hash" URI scheme conforms to URL syntax (with or
without an authority), each valid Named Information Hash URI can be assumed to
be unique within the namespace of all valid URLs. As such, any `ni://` URL (with
or without an authority) can be hashed and used as a
[UUIDv5](https://datatracker.ietf.org/doc/html/rfc9562#uuidv5) in the URL
namespace, i.e. `6ba7b811-9dad-11d1-80b4-00c04fd430c8` (See [section
6.6](https://datatracker.ietf.org/doc/html/rfc9562#namespaces)).

Since this approach relies on SHA-1, and discards all but the most significant
128 bits of the hash output, its security may not be adequate for all
applications, as noted in the specification. Alternative ways of using a bounded
namespace could include a novel namespace registration for UUIDv5, or a UUIDv8
approach, to content-address arbitrary information with namespaced UUID
variants.

[NIH registry]: https://www.iana.org/assignments/named-information/named-information.xhtml#hash-alg
[hashAlgorithm registry]: https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#tls-parameters-18
[multibase]: https://github.com/multiformats/multibase

--- back

# Security Considerations

TODO Security

# IANA Considerations

TODO - format [current Contributing.md document language](https://github.com/multiformats/multiformats/blob/master/contributing.md#multiformats-registries) to align better with {{?RFC8126}}

## Initial Values for the Multihash Identifier Registry

The Multihash Identifier Registry contains hash functions supported by Multihash
each with its canonical name, its value in hexadecimal notation, and its status.
The following initial entries should be added to the registry to be created and
maintained at (the suggested URI):

http://www.iana.org/assignments/multihash-identifiers

|Name|Identifier|Status|Specification|
|---|---|---|---|
|identity|0x00|active|n/a|
|sha1|0x11|active|{{?RFC6234}}|
|sha2-256|0x12|active| [FIPS202] |
|sha2-512|0x13|active| [FIPS202] |
|sha3-512|0x14|active| [FIPS202] |
|sha3-384|0x15|active| [FIPS202] |
|sha3-256|0x16|active| [FIPS202] |
|sha3-224|0x17|active| [FIPS202] |
|sha3-384|0x20|active| [FIPS202] |
|sha2-256-trunc264-padded|0x1012|active|{{!RFC6234}}|
|sha2-224|0x1013|active|{{!RFC6234}}|
|sha2-512-224|0x1014|active|{{!RFC6234}}|
|sha2-512-256|0x1015|active|{{!RFC6234}}|
|blake2b-256|0xb220|active|{{!RFC7693}}|

# Acknowledgments
{:numbered="false"}

Thanks to Carsten Borman, Aaron Goldman, and others for their substantial
contributions to this document on the multiformats mailing list.
