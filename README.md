# PeerAssets RFCs

_This process attempts to emulate the success of the Rust programming language
and as such has almost mirrored the [RFC process the Rust developers
use](https://github.com/rust-lang/rfcs), which is tried and tested and appears
to work very well._

# Introduction

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put
through a bit of a design process and produce a consensus among the
community and the core team.

The "RFC" (request for comments) process is intended to provide a
consistent and controlled path for new features to enter the network
and core libraries, so that all stakeholders can be confident about
the direction in which the network is evolving.

## Table of Contents
[Table of Contents]: #table-of-contents
* [Before creating an RFC]
* [The role of the shepherd]
* [The RFC life-cycle]
* [Reviewing RFCs]
* [Implementing an RFC]
* [Help! This is all too informal]

## Before creating an RFC
[Before creating an RFC]: #before-creating-an-rfc

A hastily proposed RFC can hurt its chances of acceptance. Low quality
proposals, proposals for previously rejected features, may be quickly
rejected, which can be demotivating for the unprepared contributor.
Laying some groundwork ahead of the RFC can make the process smoother.

Although there is no single way to prepare for submitting an RFC, it
is generally a good idea to pursue feedback from other project
developers beforehand to ascertain that the RFC may be desirable.
Having a consistent impact on the project requires concerted effort
toward consensus-building.

The most common preparations for writing and submitting an RFC include
filing and discussing ideas on the [RFC issue tracker][issues], and occasionally
posting "pre-RFCs" on the [Peercointalk Forum][peercointalk] for early
review.

As a rule of thumb, receiving encouraging feedback from long-standing
project developers, and particularly members of the core team or existing
contributors, is a good indication that the RFC is worth pursuing.

## The role of the shepherd
[The role of the shepherd]: #the-role-of-the-shepherd

During triage, every RFC will either be closed or assigned a shepherd.
The role of the shepherd is to move the RFC through the process. This
starts with simply reading the RFC in detail and providing initial
feedback. The shepherd should also solicit feedback from people who
are likely to have strong opinions about the RFC. Finally, when this
feedback has been incorporated and the RFC seems to be in a steady
state, the shepherd will bring it to the meeting. In general, the idea
here is to "front-load" as much of the feedback as possible before the
point where we actually reach a decision.

## The RFC life-cycle
[The RFC life-cycle]: #the-rfc-life-cycle

Once an RFC becomes active then authors may implement it and submit
the feature as a pull request to the repo. Being "active" is not
a rubber stamp and in particular still does not mean the feature will
ultimately be merged. It does mean that in principle all the major
stakeholders have agreed to the feature and are amenable to merging
it.

Furthermore, the fact that a given RFC has been accepted and is
"active" implies nothing about what priority is assigned to its
implementation, nor does it imply anything about whether a
developer has been assigned the task of implementing the feature.
While it is not *necessary* that the author of the RFC also write the
implementation, it is by far the most effective way to see an RFC
through to completion. Authors should not expect that other project
developers will take on responsibility for implementing their accepted
feature.

Modifications to active RFCs can be done in follow up PRs. We strive
to write each RFC in a manner that it will reflect the final design of
the feature, however, the nature of the process means that we cannot expect
every merged RFC to actually reflect what the end result will be at
the time of the next major release. We therefore try to keep each RFC
document somewhat in sync with the network feature as planned,
tracking such changes via followup pull requests to the document.

An RFC that makes it through the entire process to implementation is
considered "implemented" and is moved to the "implemented" folder. An RFC
that fails after becoming active is "rejected" and moves to the
"rejected" folder.

## Reviewing RFCs
[Reviewing RFCs]: #reviewing-rfcs

While the RFC PR is up, the shepherd may schedule meetings with the
author and/or relevant stakeholders to discuss the issues in greater
detail. In some cases the topic may be discussed at the larger
[weekly meeting]. In either situation, a summary from the meeting will be
posted back to the RFC pull request.

The PeerAssets team and active contributors make final decisions about RFCs after
the benefits and drawbacks are well understood. These decisions can be
made at any time, but the core team will regularly issue decisions on
at least a weekly basis. When a decision is made, the RFC PR will either
be merged or closed, in either case with a comment describing the
rationale for the decision. The comment should largely be a summary of
discussion already on the comment thread.

## Implementing an RFC
[Implementing an RFC]: #implementing-an-rfc

Some accepted RFCs represent vital features that need to be
implemented right away. Other accepted RFCs can represent features
that can wait until some arbitrary developer feels like doing the
work. Every accepted RFC has an associated issue tracking its
implementation in the affected repositories. Therefore, the
associated issue can be assigned a priority via the triage process that
the team uses for all issues in the appropriate repositories.

The author of an RFC is not obligated to implement it. Of course, the
RFC author (like any other developer) is welcome to post an
implementation for review after the RFC has been accepted.

If you are interested in working on the implementation for an "active"
RFC, but cannot determine if someone else is already working on it,
feel free to ask (e.g. by leaving a comment on the associated issue).
Any issues assigned to a sprint will be clearly marked as such in the
library issue tracker.

### Help! This is all too informal
[Help! This is all too informal]: #help-this-is-all-too-informal

The process is intended to be as lightweight as reasonable for the
present circumstances. As usual, we are trying to let the process be
driven by consensus and community norms, not impose more structure than
necessary.

[issues]: https://github.com/peercoin/rfcs/issues
[peercointalk]: https://www.peercointalk.org
