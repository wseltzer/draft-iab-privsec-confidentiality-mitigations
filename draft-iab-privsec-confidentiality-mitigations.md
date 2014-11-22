---
title: Confidentiality in the Face of Pervasive Surveillance
abbrev: privsec-mitigations 
docname: draft-iab-privsec-confidentiality-mitigations-latest
date: 2014-11-25
category: info

ipr: trust200902
area: General
workgroup: IAB
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: T. Hardie
    name: Ted Hardie
    email: ted.ietf@gmail.com

normative:
  RFC2119:
  RFC7258:
  I-D.iab-privsec-confidentiality-threat:

informative:

  RFC4301:
  RFC6962:
  RFC5750:
  RFC6698:
  RFC4306:
  RFC5246:
  RFC2015:
  STRINT:
    target: https://www.w3.org/2014/strint/draft-iab-strint-report.html
    title:  Strint Workshop Report
    author:
      name: Stephen Farrell
      ins: S Farrell
    date: 2014-04-06




--- abstract

The IAB has published {{I-D.iab-privsec-confidentiality-threat}} in
response to several revelations of pervasive attack on Internet
communications.  In this document we survey the mitigations to those
threats which are currently available or which might plausibly be
deployed.  We discuss these primarily in the context of Internet
protocol design, focusing on robustness to pervasive monitoring and
avoidance of unwanted cross-mitigation impacts.

--- middle

Introduction        {#intro}
============

To ensure that the Internet can be trusted by users, it is necessary
for the Internet technical community to address the vulnerabilities
exploited in the attacks document in {{RFC7258}} and the threats
described in {{I-D.iab-privsec-confidentiality-threat}}.  The goal of
this document is to describe more precisely the mitigations available
for those threats and to lay out the interactions among them should
they be deployed in combination.


Available Mitigations	{#responses}
=====================

Given the threat model laid out in
{{I-D.iab-privsec-confidentiality-threat}}., how should the Internet
technical community respond to pervasive attack?The cost and risk
considerations discussed in it provide a guide to responses.  Namely,
responses to passive attack should close off avenues for attack that
are safe, scalable, and cheap, forcing the attacker to mount attacks
that expose it to higher cost and risk.


In this section, we discuss a collection of high-level approaches to
mitigating pervasive attacks.  These approaches are not meant to be
exhaustive, but rather to provide general guidance to protocol
designers in creating protocols that are resistant to pervasive
attack.

~~~~~~~~~~
   +--------------------------+----------------------------------------+
   | Attack Class             | High-level mitigations                 |
   +--------------------------+----------------------------------------+
   | Passive observation      | Encryption for confidentiality         |
   |                          |                                        |
   | Passive inference        | Path differentiation                   |
   |                          |                                        |
   | Active                   | Authentication, monitoring             |
   |                          |                                        |
   | Static key exfiltration  | Encryption with per-session state      |
   |                          | (PFS)                                  |
   |                          |                                        |
   | Dynamic key exfiltration | Transparency, validation of end        |
   |                          | systems                                |
   |                          |                                        |
   | Content exfiltration     | Object encryption, distributed systems |
   +--------------------------+----------------------------------------+
~~~~~~~~~~
{: #figops title="Table of Mitigations"}

The traditional mitigation to passive attack is to render content
unintelligible to the attacker by applying encryption, for example, by
using TLS or IPsec {{RFC5246}}{{RFC4301}}.  Even without authentication,
encryption will prevent a passive attacker from being able to read the
encrypted content.  Exploiting unauthenticated encryption requires an
active attack (man in the middle); with authentication, a key
exfiltration attack is required.

The additional capabilities of a pervasive passive attacker, however,
require some changes in how protocol designers evaluate what
information is encrypted.  In addition to directly collecting
unencrypted data, a pervasive passive attacker can also make
inferences about the content of encrypted messages based on what is
observable.  For example, if a user typically visits a particular set
of web sites, then a pervasive passive attacker observing all of the
user's behavior can track the user based on the hosts the user
communicates with, even if the user changes IP addresses, and even if
all of the connections are encrypted.

Thus, in designing protocols to be resistant to pervasive passive
attacks, protocol designers should consider what information is left
unencrypted in the protocol, and how that information might be
correlated with other traffic.  Information that cannot be encrypted
should be anonymized, i.  e.  , it should be dissociated from other
information.  For example, the Tor overlay routing network anonymizes
IP addresses by using multi-hop onion routing.

As with traditional, limited active attacks, the basic mitigation to
pervasive active attack is to enable the endpoints of a communication
to authenticate each other.  However, attackers that can mount
pervasive active attacks can often subvert the authorities on which
authentication systems rely.  Thus, in order to make authentication
systems more resilient to pervasive attack, it is beneficial to
monitor these authorities to detect misbehavior that could enable
active attack.  For example, DANE and Certificate Transparency both
provide mechanisms for detecting when a CA has issued a certificate
for a domain name without the authorization of the holder of that
domain name {{RFC6962}}{{RFC6698}}.

While encryption and authentication protect the security of individual
sessions, these sessions may still leak information, such as IP
addresses or server names, that a pervasive attacker can use to
correlate sessions and derive additional information about the target.
Thus, pervasive attack highlights the need for anonymization
technologies, which make correlation more difficult.  Typical
approaches to anonymization against traffic analysis include:

 o Aggregation: Routing sessions for many endpoints through a common
 mid-point (e.g, an HTTP proxy).  Since the midpoint appears as
 the end of the communication, individual endpoints cannot be
 distinguished.

 o Onion routing: Routing a session through several mid-points, rather
than directly end-to-end, with encryption that guarantees that each
node can only see the previous and next hops.  This ensures that
the source and destination of a communication are never revealed
simultaneously.

 o Multi-path: Routing different sessions via different paths (even if
 they originate from the same endpoint).  This reduces the probability
 that the same attacker will be able to collect many sessions.

An encrypted, authenticated session is safe from content-monitoring
attacks in which neither end collaborates with the attacker, but can
still be subverted by the endpoints.  The most common ciphersuites
used for HTTPS today, for example, are based on using RSA encryption
in such a way that if an attacker has the private key, the attacker
can derive the session keys from passive observation of a session.
These ciphersuites are thus vulnerable to a static key exfiltration
attack - if the attacker obtains the server's private key once, then
they can decrypt all past and future sessions for that server.


Static key exfiltration attacks are prevented by including ephemeral,
per-session secret information in the keys used for a session.  Most
IETF security protocols include modes of operation that have this
property.  These modes are known in the literature under the heading
"perfect forward secrecy" (PFS) because even if an adversary has all
of the secrets for one session, the next session will use new,
different secrets and the attacker will not be able to decrypt it.
The Internet Key Exchange (IKE) protocol used by IPsec supports PFS by
default {{RFC4306}}, and TLS supports PFS via the use of specific
ciphersuites {{RFC5246}}.

Dynamic key exfiltration cannot be prevent by protocol means.  By
definition, any secrets that are used in the protocol will be
transmitted to the attacker and used to decrypt what the protocol
encrypts.  Likewise, no technical means will stop a willing
collaborator from sharing keys with an attacker.  However, this attack
model also covers "unwitting collaborators", whose technical resources
are collaborating with the attacker without their owners' knowledge.
This could happen, for example, if flaws are built into products or if
malware is injected later on.

 Standards can also define protocols that provide greater or lesser
 opportunity for dynamic key exfiltration.  Collaborators engaging in
 key exfiltration through a standard protocol will need to use covert
 channels in the protocol to leak information that can be used by the
 attacker to recover the key.  Such use of covert channels has been
 demonstrated for SSL, TLS, and SSH.  Any protocol bits
 that can be freely set by the collaborator can be used as a covert
 channel, including, for example, TCP options or unencrypted traffic
 sent before a STARTTLS message in SMTP or XMPP.  Protocol designers
 should consider what covert channels their protocols expose, and how
 those channels can be exploited to exfiltrate key information.

Content exfiltration has some similarity to the dynamic exfiltration
case, in that nothing can prevent a collaborator from revealing what
they know, and the mitigations against becoming an unwitting
collaborator apply.  In this case, however, applications can limit
what the collaborator is able to reveal.  For example, the S/MIME and
PGP systems for secure email both deny intermediate servers access to
certain parts of the message {{RFC5750}}{{RFC2015}}.  Even if a server
were to provide an attacker with full access, the attacker would still
not be able to read the protected parts of the message.


Mechanisms like S/MIME and PGP are often referred to as
"end-to-end"security mechanisms, as opposed to "hop-by-hop" or
"end-to-middle" mechanisms like the use of SMTP over TLS.  These two
different mechanisms address different types of attackers: Hop-by-hop
mechanisms protect from attackers on the wire (passive or active),
while end-to-end mechansims protect against attackers within
intermediate nodes.  Thus, neither of these mechanisms provides
complete protection by itself.  For example:

 o Two users messaging via Facebook over HTTPS are protected against
passive and active attackers in the network between the users and
Facebook.  However, if Facebook is a collaborator in an exfiltration
attack, their communications can still be monitored.  They would need
to encrypt their messages end-to-end in order to protect themselves
against this risk.


o Two users exchanging PGP-protected email have protected the content
of their exchange from network attackers and intermediate servers, but
the header information (e. g.  , To and From addresses) is
unnecessarily exposed to passive and active attackers that can see
communications among the mail agents handling the email messages.
These mail agents need to use hop-by-hop encryption and traffic
analysis mitigation to address this risk.

Mechanisms such as S/MIME and PGP are also known as "object-based"
security mechanisms (as opposed to "communications security"
mechanisms), since they operate at the level of objects, rather than
communications sessions.  Such secure object can be safely handled by
intermediaries in order to realize, for example, store and forward
messaging.  In the examples above, the encrypted instant messages or
email messages would be the secure objects.

The mitigations to the content exfiltration case regard participants
in the protocol as potential passive attackers themselves, and apply
the mitigations discussed above with regard to passive attack.
Information that is not necessary for these participants to fulfill
their role in the protocol can be encrypted, and other information can
be anonymized.

In summary, many of the basic tools for mitigating pervasive attack
already exist.  As Edward Snowden put it, "properly implemented strong
crypto systems are one of the few things you can rely on".
The task for the Internet community is to ensure that applications are
able to use the strong crypto systems we have defined - for example,
TLS with PFS ciphersuites - and that these properly implemented.
(And, one might add, turned on!)Some of this work will require
architectural changes to applications, e. g., in order to limit the
information that is exposed to servers.  In many other cases, however,
the need is simply to make the best use we can of the cryptographic
tools we have.

Interplay among Responses
=========================

There is work in progress in IEEE 802 to organize the “randomization”
of MAC Addresses, ensuring that the address varies as the device
connects to different networks, or connects at different times. In
theory, the randomization will mitigate the tracking by MAC
address. However, the randomization will be defeated if the adversary
can link the randomized MAC address to other identifiers such as for
example the interface identifier in IPv6 addresses, the unique
identifiers used in DHCP or DHCPv6, or unique identifiers used in
various link-local discovery protocols.  The need to consider the
interplay among responses is a general one, and this section exams
some common interactions.


IANA Considerations
===================

This memo makes no request of IANA.


Security Considerations
=======================

This memorandum describes a series of mitigations to the
attacks described in {{RFC7258}}.  No such list could possibly
be comprehensive, nor is the attack therein described the 
only possible attack.



--- back

