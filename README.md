API Design vs. Use Cases
========================

Summary
-------

This document is the report prepared by WP4 confirming that all use cases
provided by WP2 have been satisfied in our current API design.


Introduction
------------

### What exactly is being compared?

 * [Version 1.0.0]
   (https://github.com/erasmus-without-paper/ewp-wp2-use-cases/tree/v1.0.0)
   of WP2's *Use Cases* document.
 * [Version 0.2.0]
   (https://github.com/erasmus-without-paper/ewp-specs-mobility-flowcharts/tree/v0.2.0)
   of WP4's *EWP Mobility Process Explained* document.
 * Draft versions of the API specifications (as committed on 2016-04-19).

New versions of this document may be released, as the structure and workflows
become cemented.

### How to read this comparison

We originally intended to simply refer to use case numbers directly in our API
specifications, but - as each of the use cases turned out to be quite broad (no
actors, etc.) - we decided that we will publish this document instead.

We originally intended for this to be a side-by-side comparison, but decided
that quotations and responses would be more readable.

 * Each quote is copied directly from WP2's document.
 * Each response attempts to explain how the functionality will be implemented
   in the EWP workflow.


Comparison
----------

### Use case #1 (satisfied)

> The Interinstitutional Agreement (IIA) can be divided into two different sets
> of data. 1. Agreement Data 2. Fact Sheet Data As Fact Sheets are updated
> regularly without needing to re-sign the IA, the process can happen
> independent from the set-up of the IIA.

Fact sheets will be **completely separated** from IIAs in our APIs.

 * All fact sheet data will be incorporated into [Institutions API]
   [institutions-api] and [Departments API][departments-api].

 * HEIs will be allowed to publish this data even if they don't sign any IIAs.
   In fact, they will be allowed to use these APIs even if they **don't
   intend** to implement any of the mobility-related APIs.

> The process is initiated by Institution1 sending an IIA proposal to
> Institution2.

Here's what we propose it should look like in EWP:

 * IRO staff member signs into a central web application dedicated for this
   particular purpose (so called *IIA Repository*).

 * Once the staff member is signed in, he can create a new agreement and assign
   new institutions to this agreement. The repository will automatically send
   notifications to proper IRO staff members of these institutions, and all
   members will be given the edit privilege.

Note, that IIAs are the *only* entities which will have a *central* master
repository in EWP (all other entities will be stored on HEIs' servers).

> Institution2 can accept the proposal, which should lead to a notification to
> Institution1, that the proposal has been agreed upon. In case Institution2
> does not want to establish an IIA with Institution1, they can simply decline
> the proposal.

> Institution2 has the possibility to propose changes to the initial proposal,
> which need to be accepted by Institution1. Should Institution accept the
> proposed changes, the IIA is final and Institution2 receives a notification. In
> case Institution1 does not agree to the changes, they could either initiate a
> new IIA proposal or simply inform the Institution2 that they do not accept the
> proposed changes. Institution2 can in that case send a new proposal for changes
> to the initial IIA proposal.

Privileged IRO staff members of institution X will be allowed to mark a given
revision as "accepted by institution X". Once all institutions mark the same
revision as accepted, the agreement is considered accepted by all parties.

Revisions which were accepted by all parties are then published via
[Interinstitutional Agreements API][iias-api]. All applications (which have
implemented proper CNR listeners) are notified about this new agreement and can
access its contents from now on. If an agreement is modified later on (and
accepted by everyone), all listeners are notified once more.

A staff member is given access to the IIA repository after his email address
appears in proper place in the fact sheets served by the institution
(Institutions API). When the institution removes this staff member from its
fact sheets, he is also instantly denied access to the IIA repository.

> A possibility to automatically decline all further proposals from
> Institution1 might be considered.

We don't think it's necessary to consider it at this point. However - since
this is implemented as a single, central application - this can be added later,
if the need arises.

> The Fact Sheets should be updateable at any time as long as an
> Interinstitutional Agreement has been established. For this reason, both
> Institution1 and Institution2 can at any time send fact sheet information to
> the other institution.

In EWP, fact sheets are served by the institutions themselves (hosted on their
servers, via Institutions API and Departments API). Fact sheets are not
connected to IIAs in any way. This means that all HEIs can update them at any
time.


### Use case #2 (satisfied)

> The Nominations are a rather simple use case and initiate a possible
> "mobility".

> The sending institution sends a nomination proposal and the receiving
> institution can either accept or reject the nomination proposal. Accepted
> nomination proposals become nominees. The receiving institution needs to
> confirm the nomination.

Sending institution needs to implement the [Outgoing Mobilities
APIs][mobilities-api] to support this feature. Once this is done, sending
institution creates a new Outgoing Mobility object and notifies the receiving
institution that such objects has been created. Every nominated student is
"given" exactly one Outgoing Mobility object.

The receiving institution has access to all the objects and can perform some
actions on them (by calling Outgoing Mobility Remote Update API served by the
sending institution). Accepting and rejecting nominations works exactly this
way.

Once a nomination is accepted (or rejected, or accepted again), a new log entry
appears in the Outgoing Mobility object's history.


### Use case #3 (satisfied)

> The **Learning Agreement** is established by **either** the Sending or the
> Receiving Institution sending a Learning Agreement proposal to the other
> institution. The other institution can either accept or reject the Learning
> Agreement proposal. In case the Learning Agreement proposal is accepted, it
> becomes the Learning Agreement and the other institution is notified about it.

In EWPs model, Learning Agreements are expansions of nomination. Both
nominations and LAs are described by the same entity - the Outgoing Mobility
object.

Internally, Outgoing Mobility object consists of **a history of events**. Once
a new nomination is created, its LA is (obviously) empty. Once a nomination is
accepted, both parties can begin to add and edit courses in the LA. LA can be
changed many times. Each such change is accompanied by a date and an author.

As described above, Outgoing Mobility APIs are implemented by the sending
institution. The receiving institution will be using Outgoing Mobility Remote
Update API for making such changes (adding "history events").

> The other institution can either accept or reject the Learning Agreement
> proposal.

Accepting and rejecting LAs are also "history events". Both parties will be
allowed to add a proper "accepted" event. Whenever there are two "accepted by
X" and "accepted by Y" events in a row, such revision of Learning Agreement
becomes "accepted by both parties".

> In case the Learning Agreement proposal is accepted, it becomes the Learning
> Agreement and the other institution is notified about it.

Both institutions are notified on **every** change (not just the accepting
change).

> **Learning Agreement changes** are proposed by either of the two institutions.
> The other institution can either accept the changes or refuse them. In case the
> Learning agreement changes proposal is accepted, it becomes the Learning
> Agreement changes and the other institution is notified about it.

In our model, LAs can be accepted and rejected many times. LA can be changed
whenever needed, by both sides. Whenever "accepted by X" and "accepted by Y"
events appear in unison, such new LA becomes "accepted by both parties".

Since both institutions have access to the complete history of the Outgoing
Mobility object, both institutions can always determine which revision was
accepted first, and what other versions of it has been accepted.

> **Nota Bene:** The consortium decided that the implementation of course
> selection and course catalogues is not part of the current design of EWP.
> Therefore, the Learning Agreement will be handled as a single entity document

True. Implementing **Courses APIs** is optional. LAs may contain references to
courses (their IDs), but these are optional too.

If Courses APIs are implemented however, they will make the process more
user-friendly.


### Use case #4 (satisfied)

> The Arrival and Departure Information are sent by the Receiving Institution to
> the Sending Institution. The data flow is very simple and does not require for
> the data to be confirmed.

Arrival and departure dates will also be stored in the Mobility object. Each
alteration of these dates will be recorded as a "history event", as above.


### Use case #5 (partially satisfied)

> The Transcript of Records (ToR) is being transferred from the Receiving
> Institution to the Sending Institution.

ToRs will be handled by a separate API (Transcript of Records API). They will
not be a part of the Outgoing Mobility objects.

Transcript of Records API is implemented by the receiving institution, and it
is called by the sending institution. So, the sending institution is always
able to fetch the current ToR.

We have also added an option for the receiving coordinator to notify the
sending institution that he believes the ToR is ready to be recognized. He does
so by adding a proper history entry to the Outgoing Mobility object.

> Optionally, the Sending Institution can send the Home ToR back to the
> Receiving Institution.

Currently, we do not provide such feature, but we are considering it. We're not
100% sure about the meaning of "optionally" in this context:

1. It is not required for us to design this feature (it's out of EWP's
   scope).

2. It is in EWP's scope. We should design this feature, but we cannot expect
   that institutions will provide it.

Please discuss [here]
(https://github.com/erasmus-without-paper/ewp-specs-mobility-flowcharts/issues/2).


### Use case #6 (more information needed)

> The grade conversion information will be transferred from the Receiving
> Institution to the Sending Institution.

This use case has not yet been designed for. This is something we need to
discuss further. Transcript of Records API, which will be based on the EMREX
ELMO file, allows us to include some grade conversion information, but without
more details we are unable to be certain if this use case has been satisfied.

Please discuss [here]
(https://github.com/erasmus-without-paper/ewp-wp2-use-cases/issues/2).


[registry-intro]: https://github.com/erasmus-without-paper/ewp-specs-architecture/blob/stable-v1/README.md#registry
[develhub]: http://developers.erasmuswithoutpaper.eu/
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management/blob/stable-v1/README.md#statuses
[architecture]: https://github.com/erasmus-without-paper/ewp-specs-architecture
[discovery-api]: https://github.com/erasmus-without-paper/ewp-specs-api-discovery
[notification-senders]: https://github.com/erasmus-without-paper/ewp-specs-mobility-flowcharts#notification-senders
[cnr]: https://github.com/erasmus-without-paper/ewp-specs-mobility-flowcharts#notification-senders
[institutions-api]: https://github.com/erasmus-without-paper/ewp-specs-api-institutions
[departments-api]: https://github.com/erasmus-without-paper/ewp-specs-api-departments
[iias-api]: https://github.com/erasmus-without-paper/ewp-specs-api-iias
[iia-cnr-api]: https://github.com/erasmus-without-paper/ewp-specs-api-iia-cnr
[iia-search-api]: https://github.com/erasmus-without-paper/ewp-specs-api-iia-search
[master-slave]: https://en.wikipedia.org/wiki/Master/slave_(technology)
[multi-master]: https://en.wikipedia.org/wiki/Multi-master_replication
[mobilities-api]: https://github.com/erasmus-without-paper/ewp-specs-api-mobilities
[mobility-update-api]: https://github.com/erasmus-without-paper/ewp-specs-api-mobility-update
[mobility-search-api]: https://github.com/erasmus-without-paper/ewp-specs-api-mobility-search
[mobility-cnr-api]: https://github.com/erasmus-without-paper/ewp-specs-api-mobility-cnr
[ewpmobility-file]: https://github.com/erasmus-without-paper/ewp-specs-fileext-ewpmobility
