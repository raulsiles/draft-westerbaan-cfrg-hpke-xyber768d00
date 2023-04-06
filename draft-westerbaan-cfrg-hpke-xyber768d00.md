---
title: "X25519Kyber768Draft00 hybrid post-quantum KEM for HPKE"
abbrev: "hpke-xyber768d00"
category: info

docname: draft-westerbaan-cfrg-hpke-xyber768d00-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
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
  rfc7748:
  rfc9180:
  kyber: I-D.cfrg-schwabe-kyber

informative:
  combiners: I-D.ounsworth-cfrg-kem-combiners
  hybrid: I-D.ietf-tls-hybrid-design
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
for HPKE {{rfc9180}}.


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
    of DHKEM(X25519, HKDF-SHA256) and Kyber768Draft00:
    public key, private key, cipher texts and shared
    secrets are simple concatenations.

Kyber768Draft00 is Kyber768 as submitted to the third round
    of the NIST PQC process {{KyberV302}}, where it is
    also known as v3.02. It is also defined in {{kyber}}.

We use HKDF-SHA256 as HKDF.

## SerializePublicKey and DeserializePublicKey

X25519 and Kyber768Draft00 use fixed-length
    octet strings as public keys,
    and use the identity function for SerializePublicKey()
    and DeserializePublicKey().
For X25519Kyber768Draft00 we also use the identity function
    for SerializePublicKey() and DeserializePublicKey().

Note that X25519 public keys MUST be validated before they
    can be used as stipulated in {{Section 7.1.1 of rfc9180}}.

## SerializePrivateKey and DeserializePrivateKey

~~~
def SerializePrivateKey(skX):
  sk1 = skX[0:32]
  sk2 = skX[32:2432]
  return concat(
    X25519.SerializePrivateKey(sk1),
    sk2
  )

def DeserializePrivateKey(skXm):
  sk1 = skXm[0:32]
  sk2 = skXm[32:2432]
  return concat(
    X25519.DeserializePrivateKey(sk1),
    sk2
  )

~~~

X25519.SerializePrivateKey() and X25519.DeserializePrivateKey()
    are SerializePrivateKey() and respectively DeserializePrivateKey()
    as defined for X25519 in {{Section 7.1.2 of rfc9180}}.

## DeriveKeyPair

A single seed is stretched and passed to key derivation
of X25519 and Kyber768Draft00.

~~~
def DeriveKeyPair(ikm):
  dkp_prk = LabeledExtract("", "dkp_prk", ikm)
  seed = LabeledExpand(dkp_prk, "sk", 32 + 64)
  seed1 = seed[0:32]
  seed2 = seed[32:96]
  sk1, pk1 = X25519.DeriveKeyPair(seed1)
  sk2, pk2 = Kyber768Draft00.DeriveKeyPair(seed2)
  return (concat(sk1, sk2), concat(pk1, pk2))
~~~

X25519.DeriveKeyPair() is DeriveKeyPair() defined for X25519
in {{Section 7.1.3 of rfc9180}}. Kyber768Draft00.DeriveKeyPair() is the key
generation as defined in {{Section 11.1 of kyber}}.

ikm should be at least 32 octets in length.
(This is contrary to {{rfc9180}} which stipulates it should be
    at least Nsk=2432 octets in length.)

## Encap and Decap

~~~
def Encap(pkR):
  pkR1 = pkR[0:32]
  pkR2 = pkR[32:1216]
  (ss1, enc1) = DHKEM(X25519, HKDF-SHA256).Encap(pkR1)
  (ss2, enc2) = Kyber768Draft00.Encap(pkR2)
  return (
    concat(ss1, ss2),
    concat(enc1, enc2)
  )

def Decap(enc, skR):
  skR1 = skR[0:32]
  skR2 = skR[32:2432]
  enc1 = enc[0:32]
  enc2 = enc[32:1120]
  ss1 = DHKEM(X25519, HKDF-SHA256).Decap(enc1, skR1)
  ss2 = Kyber768Draft00.Decap(enc2, skR2)
  return concat(ss1, ss2)
~~~

# Security Considerations

## Concatenation and IND-CCA2 robustness

We aim for IND-CCA2 robustness: that means that if either constituent
KEM is not IND-CCA2 secure, but the other is, the combined hybrid
remains IND-CCA2 secure.

In general {{GHP18}} this requires a combiner that mixes in
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
