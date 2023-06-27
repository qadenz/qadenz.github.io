---
title: "FAQ"
linkTitle: "FAQ"
weight: 9
description: >
    Frequently Asked Questions
---

## Licensing
Questions concerning Qadenz's license terms

#### What license does Qadenz use?

Qadenz uses the [PolyForm Internal Use License 1.0](https://polyformproject.org/licenses/internal-use/1.0.0/). Teams are welcome to use Qadenz to power their internally managed test automation projects and modify as needed, but are prohibited from re-distributing the library or marketing/selling Qadenz (and derivative works of Qadenz) as a product for their customers.

#### Is "internal use" the same as "non-commercial"?

Not at all. Qadenz is intended for use **in** a commercial setting, just not **as** the commercial product. Teams/Companies may use Qadenz for the testing and validation of their commercial software products, but are restricted from distributing Qadenz, or products built upon or derived from Qadenz.

#### I've customized Qadenz for my own use, can I release my changes?

From a technical perspective, Qadenz is built to be extensible, so that custom configuration can be made without having to alter the core code-base. Teams are encouraged to import Qadenz as a dependency, then if modifications are required, to extend and override where needed. Making changes to the core code-base adds maintenance overhead and may introduce difficulty in upgrading to future versions.

From a licensing perspective, the limitations of the Internal Use license do prohibit re-distribution of Qadenz and distribution of works derivative of Qadenz. Thus, releasing changes not allowed.

That said, if a team finds that Qadenz is lacking functionality in a certain area, or that a change to existing functionality could positively impact their own and other teams that use Qadenz, the best course of action is to [create an issue](https://github.com/qadenz/qadenz/issues/new) so that the changes can be considered for inclusion in a future release.

#### Can I fork the Qadenz project?

Compliance with the license, when it comes to creating forks of Qadenz, would come down to intent. If an individual or team is involved in any contributions to Qadenz, or is simply tinkering with the code for personal education purposes, then forking would be no problem (though perhaps forking to a private repo might be a good idea). If an individual or team has forked Qadenz with the intent to re-release as either an individual library or bundled with other libraries, then this would not be allowed by the terms of the license.

#### I am a vendor, may I offer Qadenz as a product to my clients?

Vendors, in this context, refers to individual consultants, consulting companies, staff augmentation providers, managed services providers, and any other entity offering services and products to their clients.

A vendor marketing or selling Qadenz to their clients is not an internal business use, and is prohibited by the license.

A vendor recommending to a client to use Qadenz on their testing project(s) is an internal business use for the client, and perfectly acceptable.

A vendor providing team members to a client to contribute to a client-owned project powered by Qadenz is also internal business use for the client, and acceptable - as long as the vendor terms do not lock the client into a relationship with the vendor in order for the client to maintain access or ownership of any Qadenz-backed projects.

A vendor hired by a client to design, develop, test, and deliver a software solution may use Qadenz for testing projects, as this is an internal business use for the vendor. If the deliverable package to the client includes any projects powered by Qadenz, Qadenz must not be featured as a product sold to the client, and the vendor terms must not lock the client into a relationship with the vendor in order for the client to maintain access or ownership of any Qadenz-backed projects.

#### Can I build applications or libraries on top of Qadenz?

Qadenz may not be included as a component, dependency, or library of any software application intended for sale or any other distribution, as this is not an internal business use. The only exception to this is if the application or library is only to be used for internal business within the originating organization.
