---
title: "EventStorming: How to Stop Building the Wrong System"
seoTitle: "Event Storming: How to Stop Building the Wrong System"
seoDescription: "Learn how Event Storming workshops help teams discover real business processes before writing code.A practical guide to running your first Big Picture"
datePublished: 2026-03-07T09:30:00.000Z
cuid: cmmsudjjn01pd2ehdghx94pzl
slug: event-storming-how-to-stop-building-the-wrong-system
tags: software-architecture, domain-driven-design, workshop, eventstorming, domain-modeling

---

Most teams make their most critical architectural decisions during the first few weeks of a project. That's also when collective knowledge about the domain is at its absolute lowest.

It's the Project Paradox, and it's how legacy systems are born. You rush past domain exploration, celebrate quick technical progress, and end up building something that doesn't solve the actual business problem.

[Event Storming](https://www.eventstorming.com/) breaks this cycle. It's a collaborative workshop technique created by Alberto Brandolini that puts sticky notes on a wall and forces everyone in the room to confront reality together, before a single line of code gets written.

I've used it on many projects ranging from enterprise platforms to multi-tenant SaaS products. Every time, the same thing happens: people who thought they agreed on how the business process works discover they don't. And that discovery, made on Day 3 instead of Month 6, saves the project.

## **Domain Events: Why Start There?**

An event in Event Storming is a business fact. Something that already happened. Past tense. Always.

***Contract Signed. Invoice Sent. Appointment Scheduled. Patient Registered*.**

This matters because it keeps the conversation grounded. When people write in past tense, they describe reality. When they write in present or future tense, they speculate. And speculation during discovery leads to building features nobody needs.

There's another reason past-tense verbs work so well: they cut through language confusion. Different departments often use the same noun - *"Order"*, *"Client"*, *"Contract"* - to mean completely different things. But when you say *"Order Placed"* versus *"Order Shipped"*, the ambiguity disappears. Events reveal hidden boundaries that nouns hide.

Think about the start of your own morning: You heard the alarm -> You switched it off -> You got dressed -> You had breakfast. Each of those is an event. Stringing them together creates a timeline. And that timeline reveals how a process actually works.

Event Storming is, at its core, putting these events on a wall where everyone can see them. You visualize what happens in a business process and then do something with it.

## From Events to Domain Modeling

Events are just what you can see on the surface. Underneath them is behavior: the repeating patterns a series of events reveals. Deeper still is system structure, the root causes that drive those patterns.

On one project, a healthcare platform, we started by listing events like *"Appointment Requested", "Appointment Confirmed",* and *"Appointment Cancelled"*. Simple enough. But when we looked at the pattern, we noticed that *"Appointment Cancelled"* appeared far more often than anyone expected. That led us to the root cause: the scheduling rules were so rigid that patients couldn't find available slots, so they'd book anything and cancel later. The events told us what was happening. The patterns told us why.

When you run an Event Storming workshop, you start at the surface and work your way down. You're either changing an existing system for the better or designing a new one that produces the events you actually want.

## The Toolkit: What You Actually Need

For an in-person Big Picture workshop, the requirements are simple. A long wall, a paper roll to cover it, plenty of orange sticky notes, markers for everyone, food and coffee, and no chairs. Standing keeps energy high. Hunger kills it.

For remote sessions, a collaborative whiteboard like Miro works. But know that remote workshops require extra structure. Split them into 1.5-hour sessions across multiple days, use breakout rooms, and find a co-facilitator. Running a remote Event Storming session alone is a bad idea.

Four sticky note types drive a Big Picture session:

### Event

![An example of an event](https://cdn.hashnode.com/uploads/covers/62a344e51ba253b13869b821/a04d67c4-6d1a-4678-ad8e-424d70429de2.png align="center")

Orange notes for Domain Events. The business facts. The backbone of the entire exercise.

### Hot Spot

![An example of a hot spot](https://cdn.hashnode.com/uploads/covers/62a344e51ba253b13869b821/9514d71a-3640-4c95-82f4-993b4c9f3dde.png align="center")

Red notes for Hot Spots. Questions nobody can answer, risks nobody has addressed, or topics that spark heated disagreement. I pay more attention to these than anything else on the wall - they're the problems that will surface in production if you don't address them now.

### External System

![An example of an external systems](https://cdn.hashnode.com/uploads/covers/62a344e51ba253b13869b821/8529a1d9-3f95-4985-a4a1-e600fbe4d3a9.png align="center")

Pink notes for External Systems. Payment gateways, third-party APIs, Excel sheets etc. Systems or tools you don't control but that shape your process.

### Comment

![An example of a comment](https://cdn.hashnode.com/uploads/covers/62a344e51ba253b13869b821/1813acf9-08ed-429f-9fd3-760b2ddc4ca4.png align="center")

Yellow notes for Comments. Extra context, clarifications, or observations that don't fit neatly into the other categories.

Keep a visible legend on the wall throughout the session. First-timers forget which color means what, and stopping to explain it five times kills momentum.

## Three Levels of Detail

Event Storming isn't a single technique. It scales across three levels, each suited to a different stage of understanding.

### Big Picture

Gives you the general overview. You identify key business events, order them in time, and find major pain points. This is where beginners should start, and it's where most of the strategic value lives.

### Process Level

Goes deeper. You add detail to the map by adding:

*   Actors - who triggers the event?
    
*   Commands - what action is requested?
    
*   Policies - what automated rules apply?
    
*   Read models - what data does someone need to make a decision?
    

### Design Level

The most detailed. Here, participants group related business rules into aggregates, define pre-conditions and post-conditions, and sometimes write pseudo-code directly on sticky notes. At this level, you're approaching code-ready models, the kind that map to actual project folders and feature slices in your solution.

The Big Picture session is where 80% of the strategic insight comes from. It's also where the cross-functional conversations happen between people who rarely talk to each other. Don't skip it by jumping straight into Design Level. That's a common mistake.

## Running a Big Picture Workshop: Four Steps

Here's how a typical Big Picture session runs.

![Flow diagram showing the four steps of a Big Picture Event Storming session: Storm, Timeline, Pivotal Events, and Grouping and Naming](https://cdn.hashnode.com/uploads/covers/62a344e51ba253b13869b821/e6b3a7be-21a4-4fa5-a2a4-e8aa3eb22c6c.png align="center")

### Step 1: The Storm

Everyone gets orange sticky notes and markers. Write down every business event you can think of. Post them on the wall. No wrong answers. The goal is volume. One rule: focus on business events, not technical ones. *"Contract Signed"* belongs. *"Button Clicked"* or *"Request Sent to API"* doesn't. Those details are noise at this level.

When you see a note that's clearly technical or too vague, don't call the person out. Just rotate it 45 degrees. That's the signal it needs revisiting. It keeps the energy up without embarrassing anyone.

### Step 2: The Timeline

Take all those chaotic orange notes and arrange them chronologically, left to right. The result is a long horizontal line, often called the "snake." This is where gaps become visible. When two events sit next to each other but the team can't explain what connects them, you've found a blind spot.

Do this silently at first. When people talk while sorting, one or two confident voices take over and the timeline reflects their mental model, not the team's. Silent sorting forces everyone to engage physically with the material. Disagreements surface naturally when two people try to place the same event in different spots - that's the conversation you want.

### Step 3: Pivotal Events

Identify the turning points in the process. They share two characteristics. First, they mark moments where actions become hard to undo. Once an invoice is sent, reversing it costs real money and effort. Second, they often signal a switch of actors, where responsibility shifts from one person or department to another. Mark these with a vertical separator on the wall.

### Step 4: Grouping and Naming

Pivotal events create natural boundaries. The linear snake breaks into meaningful segments. Let the domain experts name these groups. This turns a flat timeline into a high-level map of the business domain. In DDD terms, these named groups map directly to Bounded Contexts.

Before the session ends, do a quick Vote. Give everyone three to five sticky arrows and ask them to mark the area of the board they think is the most critical problem to solve. You don't need consensus. You need to see where attention is clustering. That becomes the input for your next workshop.

## Why This Matters for Your Architecture

The groups you discover in Step 4 aren't just organizational labels. They're the starting point for your system's module boundaries.

I've seen teams spend weeks debating module boundaries in meeting rooms, drawing boxes on whiteboards that looked clean but had no connection to how the business actually worked. Event Storming shortcuts that. The groups you discover aren't based on someone's mental model of what the system *should* look like. They come from how the business actually operates.

And that connects to something I care a lot about in architecture: removability. If your module boundaries match the natural seams of the business, each module becomes replaceable. You can drop a bad idea without the rest of the system collapsing. Not perfect architecture on day one, but architecture that can evolve. That's what you're after. (I wrote about how this plays out in practice in [Vertical Slice Architecture Folder Structure: 4 Approaches Compared](https://nadirbad.dev/vertical-slice-architecture-folder-structure-4-approaches-compared).

## Pre-Workshop Checklist

Before you run your first session, cover these basics:

*   Get a sponsor with authority to send the invitation. Not just someone who thinks it's a good idea — someone who will act on the findings afterward. Without that commitment, the wall of sticky notes goes nowhere.
    
*   Define scope to one or two concrete use cases / processes. Don't try to map the entire domain.
    
*   Assemble the right people. Roughly half domain experts, half technical. Keep it under 10 participants.
    
*   Run a 15-30 minute briefing beforehand so everyone arrives with shared context.
    
*   Prepare the physical space. Long wall. Paper roll. Markers. Plenty of orange stickies. Food. No chairs.
    

## Don't Map the Mess

If you're working on a legacy system, don't model the as-is state. I made this mistake early on. We spent an entire session mapping how a healthcare scheduling platform currently worked, and all it did was depress the room. Everyone already knew the system was broken. What they needed was a shared picture of where they were going. Event Storm the target vision. Design the future, don't document the mess.

## What You Walk Away With

The single most valuable outcome of an Event Storming session isn't the wall of sticky notes. It's the shared understanding that forms in the minds of everyone who participated.

You could photograph the board and throw the stickies in the bin. The session was still a success if the team is aligned. That alignment — between business and technical, between what the system does and what the business needs — is what makes the difference between architecture that holds up and architecture you're rewriting in two years.

By the end of the workshop, you have a clear map: the subdomains, the boundaries, and where the natural module seams are. That's your starting point for architecture decisions that hold up as the system grows.

## Quick-Start Checklist

If you want to run your first Big Picture session this week, here's the short version:

*   Get a sponsor to send the invite and commit to acting on the findings
    
*   Set the scope (1-2 use cases max)
    
*   Assemble 6-10 people, half domain experts, half technical
    
*   Run a 15-minute briefing so everyone shows up with shared context
    
*   Storm: everyone writes orange sticky notes with business events (rotate bad ones 45°)
    
*   Timeline: arrange events left to right, silently first - let disagreements surface naturally
    
*   Pivotal Events: mark the turning points with vertical separators
    
*   Group and Name: let domain experts label the segments
    
*   Vote: mark the most critical problem area before leaving the room
    
*   Capture Hot Spots as action items
    

Event Storming sits between Big Up-Front Design and pure emergent design. Enough structure to align everyone, not so much that you're buried in details before writing a line of code. If your team is about to start a new project, or if your current system has grown into something nobody fully understands, put sticky notes on a wall. The conversations they trigger will save you months of rework.

But here's the question I hear most often after teams run their first workshop: "We have a wall full of sticky notes. Now what?"

And it's exactly what I'll cover in the next article in this series.

If you're planning your first Event Storming session and want a second pair of eyes on your approach, [book a free discovery call](https://www.linkedin.com/in/nadir-badnjevic/) and I'll help you set it up right.