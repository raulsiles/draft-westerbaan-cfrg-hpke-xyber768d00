---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
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
area: AREA
workgroup: None
keyword:
 - kyber
 - post-quantum
 - x25519
 - hpke
venue:
  group: CFRG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: bwesterb/draft-westerbaan-cfrg-hpke-xyber768d00
  latest: "https://bwesterb.github.io/draft-westerbaan-tls-xyber768d00/draft-westerbaan-cfrg-hpke-xyber768d00.html"

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
  hybrid: I-D.ietf-tls-hybrid-design
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

This memo defines X25519Kyber768Draft00, a hybrid post-quantum KEM, for HPKE {{rfc9180}}.


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

X25519Kyber768Draft00 is the concatenation 

## DeriveKeyPair

.. finish


# Security Considerations


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

