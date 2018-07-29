---
iip: 1
title: IIP Purpose and Guidelines
status: Active
type: Meta
author: Martin Becze <mb@icon.org>, Hudson Jameson <hudson@icon.org>, and others
        https://github.com/icon/IIPs/blob/master/IIPS/iip-1.md
created: 2015-10-27, 2017-02-01
---

## What is an IIP?

IIP stands for ICON Improvement Proposal. An IIP is a design document providing information to the ICON community, or describing a new feature for ICON or its processes or environment. The IIP should provide a concise technical specification of the feature and a rationale for the feature. The IIP author is responsible for building consensus within the community and documenting dissenting opinions.

## IIP Rationale

We intend IIPs to be the primary mechanisms for proposing new features, for collecting community input on an issue, and for documenting the design decisions that have gone into ICON. Because the IIPs are maintained as text files in a versioned repository, their revision history is the historical record of the feature proposal.

For ICON implementers, IIPs are a convenient way to track the progress of their implementation. Ideally each implementation maintainer would list the IIPs that they have implemented. This will give end users a convenient way to know the current status of a given implementation or library.

## IIP Types

There are three types of IIP:

- A **Standard Track IIP** describes any change that affects most or all ICON implementations, such as a change to the the network protocol, a change in block or transaction validity rules, proposed application standards/conventions, or any change or addition that affects the interoperability of applications using ICON. Furthermore Standard IIPs can be broken down into the following categories. Standards Track IIPs consist of three parts, a design document, implementation, and finally if warranted an update to the [formal specification].
  - **Core** - improvements requiring a consensus fork (e.g. [IIP5], [IIP101]), as well as changes that are not necessarily consensus critical but may be relevant to [“core dev” discussions](https://github.com/icon/pm) (for example, [IIP90], and the miner/node strategy changes 2, 3, and 4 of [IIP86]).
  - **Networking** - includes improvements around [devp2p] ([IIP8]) and [Light ICON Subprotocol], as well as proposed improvements to network protocol specifications of [whisper] and [swarm].
  - **Interface** - includes improvements around client [API/RPC] specifications and standards, and also certain language-level standards like method names ([IIP59], [IIP6]) and [contract ABIs]. The label “interface” aligns with the [interfaces repo] and discussion should primarily occur in that repository before an IIP is submitted to the IIPs repository.
  - **IRC** - application-level standards and conventions, including contract standards such as token standards ([IRC20]), name registries ([IRC26], [IRC137]), URI schemes ([IRC67]), library/package formats ([IIP82]), and wallet formats ([IIP75], [IIP85]).
- An **Informational IIP** describes an ICON design issue, or provides general guidelines or information to the ICON community, but does not propose a new feature. Informational IIPs do not necessarily represent ICON community consensus or a recommendation, so users and implementers are free to ignore Informational IIPs or follow their advice.
- A **Meta IIP** describes a process surrounding ICON or proposes a change to (or an event in) a process. Process IIPs are like Standards Track IIPs but apply to areas other than the ICON protocol itself. They may propose an implementation, but not to ICON's codebase; they often require community consensus; unlike Informational IIPs, they are more than recommendations, and users are typically not free to ignore them. Examples include procedures, guidelines, changes to the decision-making process, and changes to the tools or environment used in ICON development. Any meta-IIP is also considered a Process IIP.

It is highly recommended that a single IIP contain a single key proposal or new idea. The more focused the IIP, the more successful it tends to be. A change to one client doesn't require an IIP; a change that affects multiple clients, or defines a standard for multiple apps to use, does.

An IIP must meet certain minimum criteria. It must be a clear and complete description of the proposed enhancement. The enhancement must represent a net improvement. The proposed implementation, if applicable, must be solid and must not complicate the protocol unduly.

## IIP Work Flow

Parties involved in the process are you, the champion or *IIP author*, the [*IIP editors*](#iip-editors), and the [*ICON Core Developers*](https://github.com/icon/pm).

:warning: Before you begin, vet your idea, this will save you time. Ask the ICON community first if an idea is original to avoid wasting time on something that will be be rejected based on prior research (searching the Internet does not always do the trick). It also helps to make sure the idea is applicable to the entire community and not just the author. Just because an idea sounds good to the author does not mean it will work for most people in most areas where ICON is used. Examples of appropriate public forums to gauge interest around your IIP include [the ICON subreddit], [the Issues section of this repository], and [one of the ICON Gitter chat rooms]. In particular, [the Issues section of this repository] is an excellent place to discuss your proposal with the community and start creating more formalized language around your IIP.

Your role as the champion is to write the IIP using the style and format described below, shepherd the discussions in the appropriate forums, and build community consensus around the idea. Following is the process that a successful IIP will move along:

```
[ WIP ] -> [ DRAFT ] -> [ LAST CALL ] -> [ ACCEPTED ] -> [ FINAL ]
```

Each status change is requested by the IIP author and reviewed by the IIP editors. Use a pull request to update the status. Please include a link to where people should continue discussing your IIP. The IIP editors will process these requests as per the conditions below.

* **Work in progress (WIP)** -- Once the champion has asked the ICON community whether an idea has any chance of support, they will write a draft IIP as a [pull request]. Consider including an implementation if this will aid people in studying the IIP.
  * :arrow_right: Draft -- If agreeable, IIP editor will assign the IIP a number (generally the issue or PR number related to the IIP) and merge your pull request. The IIP editor will not unreasonably deny an IIP.
  * :x: Draft -- Reasons for denying draft status include being too unfocused, too broad, duplication of effort, being technically unsound, not providing proper motivation or addressing backwards compatibility, or not in keeping with the [ICON philosophy](https://github.com/icon/wiki/wiki/White-Paper#philosophy).
* **Draft** -- Once the first draft has been merged, you may submit follow-up pull requests with further changes to your draft until such point as you believe the IIP to be mature and ready to proceed to the next status. An IIP in draft status must be implemented to be considered for promotion to the next status (ignore this requirement for core IIPs).
  * :arrow_right: Last Call -- If agreeable, the IIP editor will assign Last Call status and set a review end date, normally 14 days later.
  * :x: Last Call -- A request for Last Call status will be denied if material changes are still expected to be made to the draft. We hope that IIPs only enter Last Call once, so as to avoid unnecessary noise on the RSS feed. Last Call will be denied if the implementation is not complete and supported by the community.
* **Last Call** -- This IIP will listed prominently on the http://iips.icon.org/ website (subscribe via RSS at [last-call.xml](/last-call.xml)).
  * :x: -- A Last Call which results in material changes or substantial unaddressed complaints will cause the IIP to revert to Draft.
  * :arrow_right: Accepted (Core IIPs only) -- After the review end date, the ICON Core Developers will vote on whether to accept this change. If yes, the status will upgrade to Accepted.
  * :arrow_right: Final (Not core IIPs) -- A successful Last Call without material changes or unaddressed complaints will become Final.
* **Accepted (Core IIPs only)** -- This is being implemented by ICON Core Developers.
  * :arrow_right: Final -- Standards Track Core IIPs must be implemented in at least three viable ICON clients before it can be considered Final. When the implementation is complete and supported by the community, the status will be changed to “Final”.
* **Final** -- This IIP represents the current state-of-the-art. A Final IIP should only be updated to correct errata.

Other exceptional statuses include:

* Deferred -- This is for core IIPs that have been put off for a future hard fork.
* Rejected -- An IIP that is fundamentally broken and will not be implemented.
* Active -- This is similar to Final, but denotes an IIP which which may be updated without changing its IIP number.
* Superseded -- An IIP which was previously final but is no longer considered state-of-the-art. Another IIP will be in Final status and reference the Superseded IIP.

## What belongs in a successful IIP?

Each IIP should have the following parts:

- Preamble - RFC 822 style headers containing metadata about the IIP, including the IIP number, a short descriptive title (limited to a maximum of 44 characters), and the author details. See [below](https://github.com/icon/IIPs/blob/master/IIPS/iip-1.md#iip-header-preamble) for details.
- Simple Summary - “If you can’t explain it simply, you don’t understand it well enough.” Provide a simplified and layman-accessible explanation of the IIP.
- Abstract - a short (~200 word) description of the technical issue being addressed.
- Motivation (*optional) - The motivation is critical for IIPs that want to change the ICON protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the IIP solves. IIP submissions without sufficient motivation may be rejected outright.
- Specification - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current ICON platforms (cpp-icon, go-icon, parity, iconJ, iconjs-lib, [and others](https://github.com/icon/wiki/wiki/Clients).
- Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.
- Backwards Compatibility - All IIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The IIP must explain how the author proposes to deal with these incompatibilities. IIP submissions without a sufficient backwards compatibility treatise may be rejected outright.
- Test Cases - Test cases for an implementation are mandatory for IIPs that are affecting consensus changes. Other IIPs can choose to include links to test cases if applicable.
- Implementations - The implementations must be completed before any IIP is given status “Final”, but it need not be completed before the IIP is merged as draft. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of “rough consensus and running code” is still useful when it comes to resolving many discussions of API details.
- Copyright Waiver - All IIPs must be in the public domain. See the bottom of this IIP for an example copyright waiver.

## IIP Formats and Templates

IIPs should be written in [markdown] format.
Image files should be included in a subdirectory of the `assets` folder for that IIP as follow: `assets/iip-X` (for iip **X**). When linking to an image in the IIP, use relative links such as `../assets/iip-X/image.png`.

## IIP Header Preamble

Each IIP must begin with an RFC 822 style header preamble, preceded and followed by three hyphens (`---`). The headers must appear in the following order. Headers marked with "*" are optional and are described below. All other headers are required.

` iip:` <IIP number> (this is determined by the IIP editor)

` title:` <IIP title>

` author:` <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). Details are below.>

` * discussions-to:` <url>

` status:` <Draft | Last Call | Accepted | Final | Active | Deferred | Rejected | Superseded>

`* review-period-end: YYYY-MM-DD

` type: `<Standards Track (Core, Networking, Interface, IRC)  | Informational | Meta>

` * category:` <Core | Networking | Interface | IRC>

` created:` <date created on, in ISO 8601 (yyyy-mm-dd) format>

` * requires:` <IIP number(s)>

` * replaces:` <IIP number(s)>

` * superseded-by:` <IIP number(s)>

` * resolution:` <url>

#### Author header

The author header optionally lists the names, email addresses or usernames of the authors/owners of the IIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the author header value must be:

Random J. User &lt;address@dom.ain&gt;

or

Random J. User (@username)

if the email address or GitHub username is included, and

Random J. User

if the email address is not given.

Note: The resolution header is required for Standards Track IIPs only. It contains a URL that should point to an email message or other web resource where the pronouncement about the IIP is made.

While an IIP is a draft, a discussions-to header will indicate the mailing list or URL where the IIP is being discussed. As mentioned above, examples for places to discuss your IIP include [ICON topics on Gitter](https://gitter.im/icon/topics), an issue in this repo or in a fork of this repo, [ICON Magicians](https://icon-magicians.org/) (this is suitable for IIPs that may be contentious or have a strong governance aspect), and [Reddit r/icon](https://www.reddit.com/r/icon/). No discussions-to header is necessary if the IIP is being discussed privately with the author.

The type header specifies the type of IIP: Standards Track, Meta, or Informational. If the track is Standards please include the subcategory (core, networking, interface, or IRC).

The category header specifies the IIP's category. This is required for standards-track IIPs only.

The created header records the date that the IIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

IIPs may have a requires header, indicating the IIP numbers that this IIP depends on.

IIPs may also have a superseded-by header indicating that an IIP has been rendered obsolete by a later document; the value is the number of the IIP that replaces the current document. The newer IIP must have a Replaces header containing the number of the IIP that it rendered obsolete.

Headers that permit lists must separate elements with commas.

## Auxiliary Files

IIPs may include auxiliary files such as diagrams. Such files must be named IIP-XXXX-Y.ext, where “XXXX” is the IIP number, “Y” is a serial number (starting at 1), and “ext” is replaced by the actual file extension (e.g. “png”).

## Transferring IIP Ownership

It occasionally becomes necessary to transfer ownership of IIPs to a new champion. In general, we'd like to retain the original author as a co-author of the transferred IIP, but that's really up to the original author. A good reason to transfer ownership is because the original author no longer has the time or interest in updating it or following through with the IIP process, or has fallen off the face of the 'net (i.e. is unreachable or isn't responding to email). A bad reason to transfer ownership is because you don't agree with the direction of the IIP. We try to build consensus around an IIP, but if that's not possible, you can always submit a competing IIP.

If you are interested in assuming ownership of an IIP, send a message asking to take over, addressed to both the original author and the IIP editor. If the original author doesn't respond to email in a timely manner, the IIP editor will make a unilateral decision (it's not like such decisions can't be reversed :)).

## IIP Editors

The current IIP editors are

` * Nick Johnson (@arachnid)`

` * Casey Detrio (@cdetrio)`

` * Hudson Jameson (@Souptacular)`

` * Vitalik Buterin (@vbuterin)`

` * Nick Savers (@nicksavers)`

` * Martin Becze (@wanderer)`

## IIP Editor Responsibilities

For each new IIP that comes in, an editor does the following:

- Read the IIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the IIP for language (spelling, grammar, sentence structure, etc.), markup (Github flavored Markdown), code style

If the IIP isn't ready, the editor will send it back to the author for revision, with specific instructions.

Once the IIP is ready for the repository, the IIP editor will:

- Assign an IIP number (generally the PR number or, if preferred by the author, the Issue # if there was discussion in the Issues section of this repository about this IIP)

- Merge the corresponding pull request

- Send a message back to the IIP author with the next step.

Many IIPs are written and maintained by developers with write access to the ICON codebase. The IIP editors monitor IIP changes, and correct any structure, grammar, spelling, or markup mistakes we see.

The editors don't pass judgment on IIPs. We merely do the administrative & editorial part.

## History

This document was derived heavily from [Bitcoin's BIP-0001] written by Amir Taaki which in turn was derived from [Python's PEP-0001]. In many places text was simply copied and modified. Although the PEP-0001 text was written by Barry Warsaw, Jeremy Hylton, and David Goodger, they are not responsible for its use in the ICON Improvement Process, and should not be bothered with technical questions specific to ICON or the IIP. Please direct all comments to the IIP editors.

December 7, 2016: IIP 1 has been improved and will be placed as a PR.

February 1, 2016: IIP 1 has added editors, made draft improvements to process, and has merged with Master stream.

March 21, 2018: Minor edits to accommodate the new automatically-generated IIP directory on [iips.icon.org](http://iips.icon.org/).

May 29, 2018: A last call process was added.

See [the revision history for further details](https://github.com/icon/IIPs/commits/master/IIPS/iip-1.md), which is also available by clicking on the History button in the top right of the IIP.

### Bibliography

[IIP5]: https://github.com/icon/IIPs/blob/master/IIPS/iip-5.md
[IIP101]: https://github.com/icon/IIPs/issues/28
[IIP90]: https://github.com/icon/IIPs/issues/90
[IIP86]: https://github.com/icon/IIPs/issues/86#issue-145324865
[devp2p]: https://github.com/icon/wiki/wiki/%C3%90%CE%9EVp2p-Wire-Protocol
[IIP8]: https://github.com/icon/IIPs/blob/master/IIPS/iip-8.md
[Light ICON Subprotocol]: https://github.com/icon/wiki/wiki/Light-client-protocol
[whisper]: https://github.com/icon/go-icon/wiki/Whisper-Overview
[swarm]: https://github.com/icon/go-icon/pull/2959
[API/RPC]: https://github.com/icon/wiki/wiki/JSON-RPC
[IIP59]: https://github.com/icon/IIPs/issues/59
[IIP6]: https://github.com/icon/IIPs/blob/master/IIPS/iip-6.md
[contract ABIs]: https://github.com/icon/wiki/wiki/ICON-Contract-ABI
[interfaces repo]: https://github.com/icon/interfaces
[IRC20]: https://github.com/icon/IIPs/issues/20
[IRC26]: https://github.com/icon/IIPs/issues/26
[IRC137]: https://github.com/icon/IIPs/issues/137
[IRC67]: https://github.com/icon/IIPs/issues/67
[IIP82]: https://github.com/icon/IIPs/issues/82
[IIP75]: https://github.com/icon/IIPs/issues/75
[IIP85]: https://github.com/icon/IIPs/issues/85
[the ICON subreddit]: https://www.reddit.com/r/icon/
[one of the ICON Gitter chat rooms]: https://gitter.im/icon/
[pull request]: https://github.com/icon/IIPs/pulls
[formal specification]: https://github.com/icon/yellowpaper
[the Issues section of this repository]: https://github.com/icon/IIPs/issues
[markdown]: https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet
[Bitcoin's BIP-0001]: https://github.com/bitcoin/bips
[Python's PEP-0001]: https://www.python.org/dev/peps/

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
