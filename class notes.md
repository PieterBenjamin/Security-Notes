# Computer Security (CSE 484), Sp21
Pieter Benjamin (pieterb@uw.edu)

# Introduction to Security
- How do systems fail?
  - Reliability (accidental failures)
  - Usability (operating mistakes by users)
  - Design and goal oversights
  - Computing in the presence of an adversary
- What does it mean for a system to be secure?
  - Not a binary relation (*this* system provides *these* properties in *this* context)
  - Who are the stakeholders?
  - What are the assets to protect?
  - What are the threats to the assets?
  - Who are the adversaries and what are their resources?
  - What is the security policy?
- This class has two main themes
  1. Security mindset (think critically about designs and how they can fail)
  2. Current (2021) web security concepts
- The first theme is a much more valuable takeaway because technologies will inevitably change but critical thinking is always valuable
# Software Security
- The most important thing to learn is how to think about a system in terms of an adversary (threat modeling)
  - Assets (what are we protecting/how valuable)
  - Adversaries (who might attack, why, how strong)
  - Vulnerabilities
  - Threats (reasonably expectable attacks)
  - Risk (how much danger are the assets in)
  - Possible defenses
- Who might benefit from the system being exploited? Who would be harmed?
- What *is* security anyway?
- Common goals: CIA (Confidentiality, Integrity, Availability)
  - Another variant is Control, Authenticity, and Utility
- Confidentiality (privacy)
  - Concealment of information
- Integrity
- Availability
- No such thing as perfect security
- Attackers have limited resources and you want to raise the cost of attack past their tolerable threshold
- Threat Modelling (Example: Electronic Voting)
  - First step - how does the system work without an adversary?
    - First phase: workers show up and load "ballot definition files" onto the machine (controls what the voter sees)
    - Second phase: Voters show up and start voting (authenticate themselves and are given a "voter token")
    - Third phase: Voter inserts card to machine, gets verified, votes, and presses a "cast my ballot" vote (choice is then saved, token is nullified)
  - What do you think are the security goals of the electronic voting system described in class and shown above? What would be some of the assets that must be protected?
    - Goals: each voter is authenticated accurately and only gets access to a single voter token which gives them access to a correct ballot definition file
    - Assets: All the voter tokens themselves, the ballot definition files, the memory on the voting machines
  - Who are the adversaries who might try to attack this electronic voting system? What might be the attacker’s goals? What potential threats or vulnerabilities do you see?
    - Political opponents, foreign powers, trolls, incompetence (handing out used voter cards, damaging files, uploading wrong information at any step), spoofed voting centers
  - The source code was revealed for the system and some concrete vulnerabilities were discovered
    - No way to know what software was being run on the system (hacker could upload anything)
    - There is an access panel on the side of the machine which gave access to all ports (they published a picture of the key on the website AND PEOPLE CAN MAKE COPIES OF KEYS FROM PICTURES)
    - Ballot definition files are not authenticated, which means that votes cast for X can be redirected to Y
    - A voter could clone a voter token and then vote multiple times
- Approaches to Security
  - Prevention
    - Stop an attack from happening
  - Detection
    - Can we figure out during/after an attack that something is wrong?
  - Response and Resilience
    - What do we immediately respond with as an attempt to stop the hacker
- Securing a system requires a whole-system view (you are only as protected as your weakest link)
- Attackers have an asymmetric advantage - they only need to find a single flaws
- Defense in depth - basically redundancy
- Even after choosing a decent system, you can still whiff the implementation
- At large scale, different parties may have different goals/optimizations which might be misaligned  
## Buffer Overflows
## Miscellaneous and Principles

# Cryptography
## Symmetric Encryptions
## Hash Functions and MACs
## Asymmetric Key Cryptography

# Web Security
## Overview and Browser Security Model
## Certificates
## Web Application Security
## Web Privacy
## Authentication
## Alex Gantman
## Mobile Platform Security
## Usable Security
## Anonymity
## Side Channels
