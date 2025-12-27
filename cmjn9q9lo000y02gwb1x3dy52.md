---
title: "Vertical Slice vs. Clean Architecture: Which Should You Choose?"
seoTitle: "Vertical Slice vs Clean Architecture: A Practical Comparison (2025)"
seoDescription: "Compare Vertical Slice vs Clean Architecture with real examples. Learn when each pattern fits, migration paths, and a hybrid approach for .NET teams."
datePublished: Fri Dec 26 2025 19:32:18 GMT+0000 (Coordinated Universal Time)
cuid: cmjn9q9lo000y02gwb1x3dy52
slug: vertical-slice-vs-clean-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1766777151842/6ae73a06-c194-45b6-b346-76dc0e40ea07.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1766777375944/21a2c2d1-ee1e-4840-ac4b-2ae9fd72c6da.png
tags: software-development, software-architecture, dotnet, clean-architecture, vertical-slice-architecture

---

Project structure shapes everything: onboarding speed, maintainability, how fast you ship features. For years, Clean Architecture (and its cousins Onion and Hexagonal) was the default. But Vertical Slice Architecture (VSA) has become a serious alternative, and I've watched teams argue about this choice more than almost anything else.

This comparison applies equally to Onion and Hexagonal architectures, which share Clean Architecture's layered philosophy. The feature-based vs layered architecture debate comes down to where you want your coupling.

So which one fits your project? Let's break it down.

*This article is part of my* [*Complete Guide to Vertical Slice Architecture in .NET*](/vertical-slice-architecture-dotnet)*, which includes a production-ready template with 530+ GitHub stars-*[*featured in NDepend's architecture comparison*](https://blog.ndepend.com/vertical-slice-architecture-in-asp-net-core/)*.

## Contents

* [Clean Architecture: The Layered Approach](#clean-architecture-the-layered-approach)
    
* [Vertical Slice Architecture: The Feature-First Approach](#vertical-slice-architecture-the-feature-first-approach)
    
* [How They Compare](#how-they-compare)
    
* [When to Pick Each](#when-to-pick-each)
    
* [The Hybrid: Best of Both?](#the-hybrid-best-of-both)
    
* [Migrating From Clean Architecture to VSA](#migrating-from-clean-architecture-to-vsa)
    
* [Choosing Based on Your Team](#choosing-based-on-your-team)
    
* [Quick Reference](#quick-reference)

![Clean Architecture vs Vertical Slice Architecture comparison infographic showing layered approach versus feature-first approach with decision guide](https://cdn.hashnode.com/res/hashnode/image/upload/v1766775679779/81d5b107-b07f-4189-a5ee-f710c9b06659.png align="center")

## Clean Architecture: The Layered Approach

Clean Architecture (CA), popularized by Robert C. Martin, organizes code through concentric layers. The key rule: all dependencies point inward toward the core.

* **Domain (Core):** Business rules and entities live here. This layer depends on nothing else, so database or UI changes can't touch it.
    
* **Application (Use Cases):** Application-specific business logic.

* **Infrastructure/UI (Outer Rings):** The messy stuff: databases, web frameworks, external APIs.
    

The goal? Keep business logic independent from frameworks and tools. In theory, you could swap your database with minimal pain.

## Vertical Slice Architecture: The Feature-First Approach

Vertical Slice Architecture (VSA), coined by Jimmy Bogard, flips the script. Instead of organizing by technical layer, you organize by feature.

A "slice" cuts through the entire stack - from API endpoint to database for one specific use case. The guiding principle: **minimize coupling between slices, maximize cohesion within a slice.**

When you add a feature in VSA, you're not scattering changes across layers. You're working in one folder that owns everything for that request.

## How They Compare

### Code Organization

Clean Architecture organizes by **type**. You get a folder for controllers, another for services, a project for repositories. Classic technical layering.

VSA organizes by **capability**. Got an "Order Cancellation" feature? There's an `OrderCancellation` folder with the endpoint, request model, handler, and data access all together. Some call this "Screaming Architecture" because your folder structure tells you what the system does.

### The Cohesion Problem

Clean Architecture keeps technical layers loosely coupled. But here's the trade-off: code that changes together (a feature's UI and its database schema) ends up scattered across multiple projects.

VSA takes the opposite bet, following what's sometimes called the **code locality principle**: *artifacts that change together should be grouped together.* The API, logic, and database for a feature are naturally coupled—they change together. So keep them together. Less jumping between files to understand what's happening.

### Real Example: Adding a Field

Say you need to add a field to "Get Weather Forecast."

**In Clean Architecture:** You're touching 5-7 files across 3-4 projects. The ViewModel in Web, the DTO in Application, the Entity in Domain, the Repository in Infrastructure. Hope you remembered them all.

**In VSA:** Open `GetForecasts.cs`, add the field to the request, handler, and response. Done. New features add new code instead of modifying shared service classes.

*Change impact: CA scatters changes across layers; VSA contains them in one folder.*

### Testing

Clean Architecture tends to be **mock-heavy**. Every layer hides behind interfaces, so your unit tests often end up testing mocks of the layer below. You're testing the wiring, not the behavior.

VSA leans toward **integration testing**. A slice is a self-contained request/response, so the most useful test exercises the full slice from API to database. Save unit tests for the shared domain model, not the handlers.

## When to Pick Each

### VSA Works Well When

* **You need to ship fast.** Features live in isolation, so you can move quickly without stepping on other code.
    
* **Requirements keep changing.** Throwing away a self-contained slice is easier than untangling logic spread across layers.
    
* **Your team is experienced.** VSA has fewer guardrails. You need developers who'll recognize when to extract shared domain logic instead of copy-pasting.
    
* **You're building APIs.** Request-response systems map naturally to slices.
    

### Clean Architecture Works Well When

* **You have complex business rules.** The protected domain core gives you a place for rich logic that won't get polluted by infrastructure concerns.
    
* **You actually might swap infrastructure.** If there's a real chance you'll change databases or frameworks, CA's abstraction boundaries pay off.
    
* **Your team is newer.** The rigid "Controller calls Service calls Repository" structure acts as training wheels. It's harder to make a mess.
    

### The Hybrid: Best of Both?

Here's what I've seen work in practice: organize your top-level structure by features (VSA), but inside complex slices, apply CA principles.

1. **Feature folders at the macro level.** Each slice owns its endpoint, handler, and simple data access.
    
2. **Domain separation at the micro level.** When a slice gets complex, extract domain logic into a separate class.
    
3. **Shared domain model.** Multiple slices can reference common entities and value objects that stay persistence-ignorant.
    

You don't have to pick one religion. Use the structure that fits the complexity at hand.

*For detailed folder organization patterns, see* [*VSA Folder Structure Best Practices*](/vertical-slice-architecture-folder-structure) *(coming soon).*

## Migrating From Clean Architecture to VSA

If your CA project has turned into "lasagna architecture" with too many layers, you can simplify through **defactoring**:

1. **Inline the abstractions.** Move repository and service logic back into the handler for a single feature.
    
2. **Group by feature.** Put the endpoint, command, handler, and validator in one folder.
    
3. **Delete the interfaces.** If an interface only exists to satisfy the layering, remove it.
    

This feels wrong at first. You've been taught that more abstraction is better. But if an abstraction doesn't have multiple implementations and doesn't enable testing, it's just noise.

### Cross-Cutting Concerns

"But what about logging? Validation? Transactions?"

Use a mediator pipeline (MediatR, Wolverine). Each slice stays simple while behaviors handle the cross-cutting stuff in one place. You get consistency without polluting every handler.

*For implementation details, see* [Vertical Slice Architecture template for .NET 9](https://github.com/nadirbad/VerticalSliceArchitecture)*

## Choosing Based on Your Team

**Junior-heavy teams:** Lean toward Clean Architecture. The prescriptive rules ("Controller calls Service") keep code organized while people learn. The structure does some of the thinking for them.

**Experienced teams:** VSA gives you speed. Senior developers have the judgment to refactor procedural code into a rich domain model when it makes sense, and to leave simple CRUD operations simple.

## Quick Reference

| **If your project is...** | **Consider...** |
| --- | --- |
| Mostly CRUD | VSA or simple N-Tier |
| Complex business rules | Clean Architecture |
| Rapidly changing features | VSA |
| Long-term (5+ years) | Hybrid approach |
| Headed toward microservices | VSA (slices become services) |

## The Bottom Line

There's no universal winner. Clean Architecture protects complexity but adds ceremony. VSA reduces ceremony but requires discipline.

Start simple. If you're building a straightforward CRUD API, VSA will get you moving faster. If you're modeling a complex domain with lots of business rules, Clean Architecture's protected core is worth the overhead.

And don't be afraid to mix them. The best architectures I've seen treat this as a spectrum, not a binary choice.

---

## Next Steps

**Ready to try VSA?** Check out my [Vertical Slice Architecture template for .NET 9](https://github.com/nadirbad/VerticalSliceArchitecture), a production-ready starting point with healthcare domain examples and 500+ GitHub stars.

**Want the full picture?** Read the [Complete Guide to Vertical Slice Architecture](/vertical-slice-architecture-dotnet) for implementation details, folder structure, MediatR patterns, and testing strategies.