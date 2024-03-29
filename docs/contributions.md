---
layout: default
title: Contributing to Open SIMH
permalink: /contributions/
---

# Contributing to [Open SIMH](/)

---

All contributions must support the core project principles:

- Preserve the ability to run old/historically significant software. This
  means functionally accurate, sometimes bug-compatible, but not
  cycle-accurate, simulation.
- Make it reasonably easy to add new simulators for other hardware while
  leveraging common functions between the simulators.
- Exploit the software nature of simulation and make SIMH convenient for
  debugging a simulated system, by adding non-historical features to the environment.
- Make it convenient for users to explore old system environments, with as
  close to historical interfaces, by mapping them to new features that modern
  host operating systems provide.
- Be inclusive of people and new technology. It's serious work, but it should be fun.

**Non-coders are welcome:** Contributions may be in the form of code,
documentation, testing, debugging, historical software, expertise in historic
systems or software, advice, help with licensing, and other forms.

### Tangible Contributions

All tangible contributions must be licensed under an MIT or BSD style license,

The standard license used and preferred by Open SIMH is available [here](../simh_license),

Contributions must be by pull requests, approved by a Steering Group member,
or for subprojects, a delegee authorized by the Steering Group.

In addition to licensing, proposed contributions are evaluated based on
the following criteria - which may evolve, and aren't absolute.

- Compatibility with the project mission
- Completeness
  - At a minimum it must build and run on at least one - preferably all - of the major platforms.
  - It should be reasonably complete - the TODO list should be reasonable.
  - External dependencies must be generally available, and appropriately licensed. Note that
    while LGPL licensed code may be used, GPL is inconsitent with SimH and is absolutely prohibited.
  - Embedded firmware/microcode and the like from the emulated system must be compatibly
    licensed, acquired by the end user, or by some other means avoid encumbering the SimH project.
- Documentation - people other than the author need to be able to understand and use it.
- We don't currently require formal Certificates of Origin for contributions, but we expect
  contributors to ensure that they have the right and authority to allow the project to use
  them under the standard [Open SIMH License](../simh_license), or an equivalent MIT/BSD style license.
- Supportability
  - Ideally, the contributor plans to support it for the indefinite future.
  - It should be in a state where someone else can take over with reasonable effort.
  - Changes that break compatibility with previous versions are strongly discouraged.

The applicablity of these criteria to proposed contributions
requires judgement. The Steering Group encourages contributions, but
is the ultimate authority on what will be accepted.

Contributions by the Steering Group members are held to the same
standards. Contributions by Steering Group members may (always? **TBD**)
require sign-off by another member to ensure consistency and fairness.

The Steering group welcomes contributions from everyone, and
intends to provide timely feedback on and/or assistance with proposals.

### Behaviors

We hope this section isn't necessary. But in any case: we expect professional
behavior from all contributors to the project. This includes these guidlines:

- Treat everyone respectfully - "Don't be a jerk"
- If unsure, ask before doing something.
- Keep the project core principles in mind.
- We work for concensus - but if things don't go our way, we "disagee but
  commit" to the final decision.
