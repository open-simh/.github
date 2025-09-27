---
layout: default
title: DEC VMS licensing status
permalink: /dec_vms_licensing/
---

# Licensing for DEC VMS

---
# Disclaimer
The information on this page is correct to the best of our knowledge, but does not constitute legal advice.

# Summary

 - To the best of our knowledge, there is currently no legal way to obtain or renew a license for the VMS operating system or its layered products on OpenSimH.
 - The former and popular "hobbyist"  program has been discontinued, and all licenses issued under it have expired.
- Possession of a license "key" (PAK )does not grant the right to use licensed software.
- A license is required whether or not the software uses technical means to verify compliance.
- The OpenSIMH steering group is well aware of the frustration this causes, and has attempted to obtain a new hobbyist license from HP. So far, unsuccessfully.

# Historical background
**All** DEC software, including diagnostics, operating systems, layered products, firmware, and (most) custom software was provided under license.

DEC software licensing was complex and evolved over time. In general, its software licenses are tied to the hardware with which they were sold or on which they were in first installed.

Some software licenses limited the number of users and/or the computing capacity of the machine on which it could be used.  Others authorized product features, such as networking or volume shadowing. Still others licensed a group of products, restricted use to particular environments (such as instruction or field testing), and/or other other market requirements.

Transferring a license to another machine required permission from DEC, which was granted (or not) on a case-by-case basis. (OpenSIMH is considered a new machine. Some of the commercial emulator vendors including some based on OpenSIMH, have the right to transfer licenses from physical machines to their products. OpenSIMH does not.)

Some products used technical means to enable product features and/or to assist customers' compliance with license terms.  Regardless of whether or not technical means were used, the licensee (customer) was (and is) responsible for having a valid license and complying with its terms.

Starting with VAX/VMS version 5, VMS, Ultrix, and Digital Unix included a tool known as the License Management Facility (LMF), specifically intended to help customers to comply with license terms. LMF represented licenses with a document called a Product Authorization Key (PAK), which was distributed in both paper and electronic forms.  PAKs are cryptographically signed by the issuer and entered into the LMF database by the customer, serving as a "license key".  Possession of a PAK does **not** not grant a license to the software.  However, both the operating systems and layered products used LMF to determine whether use was authorized.

Prior versions of VAX/VMS do not include LMF, but a license is still required to use a product.

In the same way that a stolen key does not authorize driving a car, **a PAK is NOT a license; it is an assertion that a valid license was legally obtained.**

Most licenses issued by DEC do not expire. However, updates were provided under software maintenance agreements, which extended the license to include a right to new versions for the duration of the agreement.

Some DEC-provided software required royalty payments to third parties, complicating licensing.  This is one reason that hobbyist licensing can be infeasible for these products.

# Hobbyist Licensing
DEC introduced and Compaq/HP continued a popular [license program for hobbyist use](/dec_vms_license/), which provided licenses for VAX/VMS and many layered products. Many OpenSIMH users took advantage of this program, which provided licenses that expired annually.

The business case for this was DEC's interest in providing Alpha and IPF customers with a pool of knowledgeable people who could work with the VAX/VMS operating system and its layered products

The hobbyist license program was discontinued when HP decided that the business case was no longer valid.

At this point, all licenses issued under this program have expired and are no longer valid.

# Current status
There is currently no legal means of obtaining a valid license for VAX/VMS or its layered products for use with OpenSIMH.

Although the subject of unofficial PAK generators and other means of avoiding license compliance are discussed periodically, it is important to note that such mechanisms facilitate the misappropriation of intellectual property. In the event that enforcement action is taken by a current rights holder(s), the use of such mechanisms could be interpreted to show intent.

The OpenSIMH steering group is well aware of the frustration this causes, and has attempted to obtain a new hobbyist license from HP. So far, unsuccessfully.