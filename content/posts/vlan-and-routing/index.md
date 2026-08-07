---
title: "The Numbers, and What May Not Talk to What"
date: 2026-08-07
draft: false
tags: ["homelab", "networking", "vlan", "routing", "security", "sysadmin"]
summary: "Two cards: addresses that tell you where they live, and a management segment designed to have almost no surface at all."
---

![The gatekeeper: only explicitly allowed traffic may pass](Gatekeeper.png)

The numbers finally exist.

Two cards this week. Card 3 gave the three segments their VLAN IDs and subnets. Card 4 decided what may talk to what, and — more to the point — what may not. Both are design work; the lab is still powered off. Nobody has typed a single command into a switch. That comes later.

## The convention I tried to break, and couldn't

Start with the one I lost.

The scheme is simple: the VLAN number echoes into the address. VLAN 20 becomes 10.20.x.x. Read any address in the lab and you know its segment instantly, without looking anything up. That readability pays for itself the moment you're reasoning about a route or a firewall rule at speed.

I wanted the VLANs numbered in hundreds — 100, 200, 300 — because tens felt like everyone else's homelab and I fancied being a bit different.

The maths declined. An IP octet stops at 255. VLAN 300 would want to become 10.300.x.x, which is not a place. The echo that made the whole scheme readable survives two segments and then falls apart at the third.

So: tens. 10, 20, 30, with room for 250-odd segments before anything creaks. The useful part isn't the answer, it's what the attempt taught me — a "different" convention that fights the address maths isn't a flourish, it's a trap with better branding. Provoke the convention, find out whether it holds. This one held.

## The other numbers, briefly

The lab lives on 10.x, not 172.x, because the ISP's internal DNS resolver sits in 172-space and a clash there would be a genuinely tedious afternoon. Leave the neighbours' range alone.

The transit link between the edge router and the core got a /30 rather than a /31. Both work for a two-ended point-to-point link, and /31 is arguably the more correct choice — it wastes nothing. I picked /30 because it's the one every engineer reads without pausing, and recorded /31 in the document as the considered alternative. Readability won, but the alternative is on the record so it's clear it was a choice rather than an oversight.

Gateways always end .1. A reserved band at .240–.254 holds anything special or redundant. Load-bearing addresses live in predictable places.

And DHCP — automatic addressing — is designed but dormant. The .100–.239 range is drawn on the map today and hands out nothing, because there are no clients yet; VMs and workstations arrive in Phase 3. Servers and management stay static regardless. An anchor has to answer at a known address, and that goes double for a hardware controller, whose entire job is being reachable when its server isn't.

Drawing the empty range now costs nothing and keeps the address map whole. Retrofitting it later would mean reopening decisions that are currently closed.

## Then: what may talk to what

Card 4 is the filtering side, and it's where the design gets opinions.

All routing between segments happens on the core switch. The edge router does NAT, runs a basic stateful firewall, and holds static routes pointing back at the lab across the transit link — and nothing else. One routing authority in the middle, a deliberately boring device at the edge, because everything you expose is something you've agreed to defend.

## The management segment is a locked room

This is the centrepiece, and the bit I'd defend in an interview.

Management is Tier 0 — the segment that controls everything else. If it falls, arguing about the rest is academic. So it's default-deny in *both* directions. Nothing reaches into it. It initiates nothing outward. The exceptions are a small handful of explicitly reasoned flows, all originating from one privileged workstation.

The goal isn't to filter management traffic cleverly. It's for management to have almost no reachable surface to filter in the first place. A rule you never have to evaluate is stronger than a rule you evaluate correctly.

Which is why network devices are administered over serial console for now, with in-band SSH deliberately deferred. Serial isn't on the network at all — it's a cable and a physical presence. Inconvenient by design, and the inconvenience is the feature.

The same logic scopes ICMP. Ping is allowed only from one trusted origin, because a host that answers ping has confirmed it exists. "It's just ping" is still reconnaissance; it's simply polite about it.

## The deny that's actually holding the line

Here's the fact that reframed the whole edge design for me.

The edge router takes a real public IP straight from the ISP. No carrier NAT in front of it. The lab isn't hiding behind anything.

On an ordinary home connection, the ISP's NAT quietly absorbs unsolicited inbound traffic whether you've thought about it or not. Your firewall rules are a second line, and you may never learn whether they work. Here there's no such cushion. "Deny all inbound" stops being a tidy default and becomes the single thing standing between the lab and the open internet.

It's the same rule either way. The difference is entirely in what happens if it's wrong.

## The limitation, said out loud

The edge firewall today is the branch router's own stateful firewall. It's adequate for this stage. It is also a branch router doing firewall duty, which is not the same as a dedicated appliance, and pretending otherwise would be the kind of thing that reads well right up until someone asks a follow-up question.

So it's written into the document as a known limitation with a dedicated firewall named as the next step. Same reasoning as the unhardened workstation from last time: a document that only describes the intended state is a brochure.

The pattern across both cards is the same one. Challenge the convention, keep it if the reasoning survives, and write down what you don't allow — plus what you can't do yet.

Configuration next. Actual commands. Possibly even electricity.

---

*I'm Ioannis — an IT operations, systems, and networking engineer based near Stockholm. The Itchef Hybrid Project (IHPC) is a real hybrid infrastructure lab, built and documented in public. Repo: [github.com/TheITchef/IHPC](https://github.com/TheITchef/IHPC) · [LinkedIn](https://www.linkedin.com/in/ioannis-mintzivyris-a1b77873/)*