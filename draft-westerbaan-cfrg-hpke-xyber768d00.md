---
title: "X25519Kyber768Draft00 hybrid post-quantum KEM for HPKE"
abbrev: "hpke-xyber768d00"
category: info

docname: draft-westerbaan-cfrg-hpke-xyber768d00-latest
submissiontype: IRTF
number:
date:
consensus: true
stand_alone: true
v: 3
area: "IRTF"
workgroup: "Crypto Forum"
keyword:
 - kyber
 - post-quantum
 - x25519
 - hpke
venue:
  group: "Crypto Forum"
  type: "Research Group"
  mail: "cfrg@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/search/?email_list=cfrg"
  github: "bwesterb/draft-westerbaan-cfrg-hpke-xyber768d00"
  latest: "https://bwesterb.github.io/draft-westerbaan-cfrg-hpke-xyber768d00/draft-westerbaan-cfrg-hpke-xyber768d00.html"

author:
 -
    fullname: Bas Westerbaan
    organization: Cloudflare
    email: bas@cloudflare.com


normative:
  RFC7748:
  RFC9180:
  KYBER: I-D.cfrg-schwabe-kyber

informative:
  COMBINERS: I-D.ounsworth-cfrg-kem-combiners
  HYBRID: I-D.ietf-tls-hybrid-design
  TLS-XYBER: I-D.tls-westerbaan-xyber768d00
  GHP18:
    target: https://doi.org/10.1007/978-3-319-76578-5_7
    title: KEM Combiners
    date: 2018
    author:
      -
        ins: F. Giacon
        name: Federico Giacon
      -
        ins: F. Heuer
        name: Felix Heuer
      -
        ins: B. Poettering
        name: Bertram Poettering
  KyberV302:
    target: https://pq-crystals.org/kyber/data/kyber-specification-round3-20210804.pdf
    title: CRYSTALS-Kyber, Algorithm Specification And Supporting Documentation (version 3.02)
    author:
      -
        ins: R. Avanzi
      -
        ins: J. Bos
      -
        ins: L. Ducas
      -
        ins: E. Kiltz
      -
        ins: T. Lepoint
      -
        ins: V. Lyubashevsky
      -
        ins: J. Schanck
      -
        ins: P. Schwabe
      -
        ins: G. Seiler
      -
        ins: D. Stehle # TODO unicode in references
    date: 2021
    format:
      PDF: https://pq-crystals.org/kyber/data/kyber-specification-round3-20210804.pdf

--- abstract

This memo defines X25519Kyber768Draft00, a hybrid post-quantum KEM,
for HPKE (RFC9180). This KEM does not support the authenticated modes
of HPKE.

--- middle

# Introduction

## Motivation

The final draft for Kyber is expected in 2024.
There is a desire to deploy post-quantum cryptography earlier than that.
To promote interoperability of early implementations,
    this document specifies a preliminary hybrid post-quantum key agreement.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Construction

In short, X25519Kyber768Draft00 is the parallel combination
of DHKEM(X25519, HKDF-SHA256) {{RFC9180}} {{RFC7748}}
and Kyber768Draft00 {{KYBER}}:
wire encodings of public key, private key, cipher texts
and shared secrets are simple concatenations.

A KEM private key is a tuple of an DHKEM(X25519, HKDF-SHA256)
private key and Kyber768Draft00 private key, where each is
an octet string of length 32 and 2400 bytes, respectively.
Similarly, a KEM public key is a tuple of an DHKEM(X25519, HKDF-SHA256)
public key and Kyber768Draft00 public key.

Kyber768Draft00 is Kyber768 as submitted to the third round
of the NIST PQC process {{KyberV302}}, where it is
also known as v3.02.

Note that this hybrid KEM is different from the one
defined in {{TLS-XYBER}} based on {{HYBRID}} for TLS,
as raw X25519 shared secrets can be used,
thanks to the message transcript.

We use HKDF-SHA256 as the HPKE HKDF. We denote the DHKEM(X25519, HKDF-SHA256)
KEM as DHKEM throughout the document.

## SerializePublicKey and DeserializePublicKey

SerializePublicKey and DeserializePublicKey encode and decode
X25519Kyber768Draft00 public keys to and from their wire format representation.

DHKEM and Kyber768Draft00 use fixed-length
octet strings as public keys,
and use the identity function for SerializePublicKey()
and DeserializePublicKey().

For X25519Kyber768Draft00 we also use the identity function
for SerializePublicKey() and DeserializePublicKey().

Note that DHKEM public keys MUST be validated before they
can be used as stipulated in {{Section 7.1.1 of RFC9180}}.

## SerializePrivateKey and DeserializePrivateKey

SerializePrivateKey and DeserializePrivateKey encode and decode
X25519Kyber768Draft00 private keys to and from their wire format representation.
Their implementation is described below.

~~~
def SerializePrivateKey(skX):
  (skA, skB) = skX
  return concat(
    DHKEM.SerializePrivateKey(skA),
    skB
  )

def DeserializePrivateKey(skXm):
  (skA, skB) = skX
  return concat(
    DHKEM.DeserializePrivateKey(skA),
    skB
  )
~~~

DHKEM.SerializePrivateKey() and DHKEM.DeserializePrivateKey()
are SerializePrivateKey() and respectively DeserializePrivateKey()
as defined for DHKEM in {{Section 7.1.2 of RFC9180}}.

## DeriveKeyPair

DeriveKeyPair deterministically derives a X25519Kyber768Draft00 private
and public key pair from a fixed-length seed. In particular, a single seed
is stretched and passed to the relevant key derivation functions for 
DHKEM and Kyber768Draft00.

~~~
def DeriveKeyPair(ikm):
  dkp_prk = LabeledExtract("", "dkp_prk", ikm)
  seed = LabeledExpand(dkp_prk, "sk", 32 + 64)
  seed1 = seed[0:32]
  seed2 = seed[32:96]
  sk1, pk1 = DHKEM.DeriveKeyPair(seed1)
  sk2, pk2 = Kyber768Draft00.DeriveKeyPair(seed2)
  return (concat(sk1, sk2), concat(pk1, pk2))
~~~

DHKEM.DeriveKeyPair() is DeriveKeyPair() defined for DHKEM
in {{Section 7.1.3 of RFC9180}}. Kyber768Draft00.DeriveKeyPair() is the key
generation as defined in {{Section 11.1 of kyber}}.

ikm should be at least 32 octets in length.
(This is contrary to {{RFC9180}} which stipulates it should be
at least Nsk=2432 octets in length.)

## Encap and Decap

Encap and Decap are the primary KEM functions. Their implementation
is described below.

~~~
def Encap(pkR):
  (pkA, pkB) = pkR
  (ss1, enc1) = DHKEM.Encap(pkA)
  (ss2, enc2) = Kyber768Draft00.Encap(pkB)
  return (
    concat(ss1, ss2),
    concat(enc1, enc2)
  )

def Decap(enc, skR):
  (skA, skB) = skR
  enc1 = enc[0:32]
  enc2 = enc[32:1120]
  ss1 = DHKEM.Decap(enc1, skA)
  ss2 = Kyber768Draft00.Decap(enc2, skB)
  return concat(ss1, ss2)
~~~

# Security Considerations

We aim for IND-CCA2 robustness: that means that if either constituent
KEM is not IND-CCA2 secure, but the other is, the combined hybrid
remains IND-CCA2 secure.

In general {{GHP18}} {{COMBINERS}} this requires a combiner that mixes in
the cipher texts, such as, assuming fixed-length cipher texts and shared secrets:

    HKDF(concat(ss1, ss2, enc1, enc2)).

In the present case, DHKEM(X25519, -) and Kyber768Draft00 already mix in
the respective cipher texts into their shared secrets. Thus we can
forego mixing in the cipher texts a second time.

Furthermore, in HPKE, the shared secret is never used directly, but
passed through HKDF (via KeySchedule), and thus we can
forego the call to HKDF as well.

# IANA Considerations

This document requests/registers a new entry to the "HPKE KEM Identifiers"
 registry.

 Value:
 : 0x30 (please)

 KEM:
 : X25519Kyber768Draft00

 Nsecret:
 : 64

 Nenc:
 : 1120

 Npk:
 : 1216

 Nsk:
 : 2432

 Auth:
 : no

 Reference:
 : This document

--- back

# Change log

> **RFC Editor's Note:** Please remove this section prior to publication of a
> final version of this document.
