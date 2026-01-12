---
title: "Clarification of Requirements for the Ordering of RRSets in CNAME Responses in the DNS"
abbrev: "CNAME Response Order"
category: std
updates: 1034

docname: draft-jabley-dnsop-cname-response-order-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Domain Name System Operations"
keyword:
 - dns
 - cname
venue:
  group: "Domain Name System Operations"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"
  github: "ableyjoe/draft-jabley-dnsop-cname-response-order"
  latest: "https://ableyjoe.github.io/draft-jabley-dnsop-cname-response-order/draft-jabley-dnsop-cname-response-order.html"

author:
 -
    fullname: "Joe Abley"
    organization: Cloudflare
    email: "jabley@cloudflare.com"
 -
    fullname: "Sebastiaan Neuteboom"
    organization: Cloudflare
    email: "sebastiaan@cloudflare.com"

normative:

informative:
 Neuteboom2026:
   title: "What Came First? The CNAME or the A Record?"
   author:
     -
       ins: S. Neuteboom
       name: Sebastiaan Neuteboom
       org: Cloudflare
   date: 2026
   target: http://blog.cloudflare.com/what-came-first-the-cname-or-the-a-record/


--- abstract

In the DNS, recursive resolvers receive queries and formulate
corresponding  responses by reference to data obtained from
authoritative servers. In the case where the query name (QNAME) and
class (QCLASS) in the query matches an owner name in the DNS where
a canonical name (CNAME) resource record is published, the response
includes both the CNAME record and the resource record set corresponding
to the request's query type (QTYPE) published at the CNAME target.
The ordering of these two different RRSets has not previously been
clearly specified, but operational experience based on real-world
client implementations confirms that the order is important. This
document updates the specification accordingly.


--- middle

# Introduction

The DNS specification includes some ambiguity around the processing
of queries that require the processing of CNAME resource records,
as discussed in {{ambiguity}}. Different assumptions about the way
that corresponding DNS responses are constructed are known to have
caused widespread negative impact, as described in {{impact}}. This
document updates the DNS specification to resolve the ambiguity.
The clarification to the specification can be found in {{updates}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document assumes familiarity with the terminology of the Domain
Name System as described in {{?RFC9499}}.

# Ambiguity in the DNS Specification {#ambiguity}

{{!RFC1034}} introduces the CNAME RRType to be one that "identifies
the canonical name of an alias". The implementation of the CNAME
resource record is further described in section 3.6 as follows:

> CNAME RRs cause special action in DNS software.  When a name server
> fails to find a desired RR in the resource set associated with the
> domain name, it checks to see if the resource set consists of a CNAME
> record with a matching class.  If so, the name server includes the CNAME
> record in the response and restarts the query at the domain name
> specified in the data field of the CNAME record.  The one exception to
> this rule is that queries which match the CNAME type are not restarted.
>
> For example, suppose a name server was processing a query with for USC-
> ISIC.ARPA, asking for type A information, and had the following resource
> records:
>
>     USC-ISIC.ARPA   IN      CNAME   C.ISI.EDU
>
>     C.ISI.EDU       IN      A       10.0.0.52
>
> Both of these RRs would be returned in the response to the type A query,
> while a type CNAME or * query should return just the CNAME.

The required processing of requests that require CNAME processing
is further described in section 4.3.2, in particular in the step
labelled 3 (a) of the recursive algorithm, as follows:

>          a. If the whole of QNAME is matched, we have found the
>             node.
>
>             If the data at the node is a CNAME, and QTYPE doesn't
>             match CNAME, copy the CNAME RR into the answer section
>             of the response, change QNAME to the canonical name in
>             the CNAME RR, and go back to step 1.
>
>             Otherwise, copy all RRs which match QTYPE into the
>             answer section and go to step 6.

Some implementations behave as though the direction "copy all RRs
which match QTYPE into the answer section" implies a required or
expected ordering, in the sense that resource records added first
should be at the beginning of the section and subsequent RRs should
be appended (see {{impact}} for an example). However, the various
sections of a DNS message are not defined in the specification as
being ordered lists. For example, the "answer section of the response"
is described in section 3.7 as follows:

> DNS queries and responses are carried in a standard message format.  The
> message format has a header containing a number of fixed fields which
> are always present, and four sections which carry query parameters and
> RRs.
>
> [...]
>
> Answer          Carries RRs which directly answer the query.

The confusion on CNAME ordering stems from section 4.3.1, which describes recursive responses as follows:

> If recursive service is requested and available, the recursive
> response to a query will be one of the following:
>
>     - The answer to the query, possibly preface by one or more
>     CNAME RRs that specify aliases encountered on the way to an
>     answer.

While the word "preface" suggests that CNAME RRs should precede other
records, this language is descriptive rather than normative,
lacking the explicit requirement keywords defined in {{!RFC2119}}.  This
ambiguity has led to varying interpretations among implementers and
contributed to interoperability issues as described in Section 4.

This document aims to resolve this ambiguity with a clear specification
for the ordering of RRSets in the answer section of a DNS message
for CNAME responses (see {{updates}}).

# Consequences of Ambiguity {#impact}

A recursive server performing CNAME preocessing without interpreting
the direction in {{!RFC1034}} that "copy" means "append" might
reasonably generate response messages with arbitrary ordering of
RRSets within the answer section.

A client of a recursive server that interprets the direction in
{{!RFC1034}} the other way, that "copy" does indeed mean "append",
might potentially react badly to receiving a response where the
answer section has been constructed differently. A real-world example
of such a consequence is described in {{cloudflare}}.

Since it is known that at least some widely-deployed clients of
resolver servers expect the "copy" direction in {{!RFC1034}} to be
interpreted as "append", and react badly when they receive responses
that do not meet this expectation, this document updates the DNS
standard to choose and require the interpretation of {{!RFC1034}}
that promotes the greatest operational stability as described in
{{updates}}.

# Updates to RFC 1034 {#updates}

A DNS implementation following the algorithm described in {{!RFC1034}}
MUST interpret any direction to "copy RRs into the answer section"
to mean that the RRs concerned should be appended to any existing
records that have already been accumulated in an earlier step of
the algorithm. The ordering of individual RRs within the set of RRs
being appended to the answer section in any step is not significant.

In the case where one or more CNAME RRs are processed in order to
construct a DNS response, the answer section MUST include a CNAME
RRSet whose owner name matches the QNAME in the query first, and
subsequent RRSets whose owner name matches the preceding CNAME
RDATA, in order.

For example, consider a query for www.example.com with QTYPE A where
the following CNAME chain exists:

>     www.example.com.    CNAME cdn.example.com.
>     cdn.example.com.    CNAME origin.example.com.
>     origin.example.com. A     192.0.2.1

The answer section MUST be ordered the same.

The CNAME records MUST NOT appear after the records they alias to:

>     origin.example.com. A     192.0.2.1
>     www.example.com.    CNAME cdn.example.com.
>     cdn.example.com.    CNAME origin.example.com.

The CNAME records MUST NOT appear out of order in their chain:

>     cdn.example.com.    CNAME origin.example.com.
>     www.example.com.    CNAME cdn.example.com.
>     origin.example.com. A     192.0.2.1

Client implementations MUST accept responses where CNAME RRSets
appear in any order, but server implementations MUST NOT rely
on this capability when generating responses.

# Security Considerations

The ambiguity described in {{ambiguity}} is known to have led to
widespread instability in at least one case, as described in
{{impact}}. The clarification provided by this document does not
present additional security concerns for the DNS.

# IANA Considerations

This document has no IANA actions.


--- back

# Events of 8 January 2026 {#cloudflare}

Cloudflare operates a well-known public DNS resolver known as
1.1.1.1, after one of the IPv4 addresses associated with the service.
On 8 January a software change in the 1.1.1.1 service had the
unintential side-effect of changing the order in which RRSets were
encoded in the answer section of DNS responses, in the case where
constructing the responses involved CNAME processing. The previous
ordering was as described in {{updates}}. The change in behaviour
was not detected by a corresponding failure in a regression test,
since the ordering in the answer section was not considered to be
significant.

Following the software release, Cloudflare became aware of significant
numbers of deployed DNS client implementations that were suffering
from failure. In particular, the getanswer_r() function invoked by
the getaddrinfo() function in glibc was found to fail to function,
and some deployed ethernet switches were observed to reboot when
trying to resolve the names of configured NTP servers.

The impact associated with this event was particularly widespread
because of the widespread use of the 1.1.1.1 resolver. However, the
two examples of client implementations are also widely deployed in
systems that may well be upgraded only infrequently (or never
upgraded at all).

See {{Neuteboom2026}} for additional information.

# Editorial Notes (remove before publication)

## Open Questions

1. This document only concerns itself with the ordering of RRSets in the
answer section for reasons of CNAME processing. There is an argument
that the clarification should be more general, in other words that all
sections in a DNS message should be treated as ordered lists, and that
adding RRSets to a section for any reason while a message is under
construction should always be an append operation. Which is better?

## Change History

### draft-jabley-dnsop-cname-response-order-00

Initial draft circulated for comment.

# Acknowledgments
{:numbered="false"}

Your name here, etc.
