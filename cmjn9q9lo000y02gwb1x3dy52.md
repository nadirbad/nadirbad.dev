---
title: "Vertical Slice vs. Clean Architecture: Which Should You Choose?"
seoTitle: "Vertical Slice vs Clean Architecture: A Practical Comparison (2025)"
seoDescription: "Compare Vertical Slice vs Clean Architecture with real examples. Learn when each pattern fits, migration paths, and a hybrid approach for .NET teams."
datePublished: Fri Dec 26 2025 19:32:18 GMT+0000 (Coordinated Universal Time)
cuid: cmjn9q9lo000y02gwb1x3dy52
slug: vertical-slice-vs-clean-architecture
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1766777375944/21a2c2d1-ee1e-4840-ac4b-2ae9fd72c6da.png
tags: software-design, software-development, software-architecture, software-engineering, dotnet, clean-architecture, vertical-slice-architecture

---

Project structure shapes everything: onboarding speed, maintainability, how fast you ship features. For years, Clean Architecture (and its cousins Onion and Hexagonal) was the default. But Vertical Slice Architecture (VSA) has become a serious alternative, and I've watched teams argue about this more than almost anything else.

This comparison applies equally to Onion and Hexagonal architectures, which share Clean Architecture's layered philosophy. The feature-based vs layered architecture debate comes down to where you want your coupling.

So which one fits your project? Let's break it down.

*This article is part of my* [*Complete Guide to Vertical Slice Architecture in .NET*](/vertical-slice-architecture-dotnet)*, which includes a production-ready template with 530+ GitHub stars-*[*featured in NDepend's architecture comparison*](https://blog.ndepend.com/vertical-slice-architecture-in-asp-net-core/).

> **Real-world comparison:** This article compares two production-ready templates—[Vertical Slice Architecture template for .NET 9](https://github.com/nadirbad/VerticalSliceArchitecture) (530+ stars, healthcare domain) and [Jason Taylor's Clean Architecture template](https://github.com/jasontaylordev/CleanArchitecture) (17k+ stars, ToDo). All code examples are from these actual codebases.

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

Let's trace what happens when you need to add a field to a feature in both architectures. I'll use real examples from production templates.

**In Clean Architecture** ([Jason Taylor's template](https://github.com/jasontaylordev/CleanArchitecture)):

To add a "Tags" field to the TodoItem feature, you touch **8-10 files across 4 projects**:

1. [`Domain/Entities/TodoItem.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Domain/Entities/TodoItem.cs) — Add property to entity
2. [`Application/TodoItems/Commands/CreateTodoItem/CreateTodoItem.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Application/TodoItems/Commands/CreateTodoItem/CreateTodoItem.cs) — Add to command
3. [`Application/TodoItems/Commands/CreateTodoItem/CreateTodoItemCommandValidator.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Application/TodoItems/Commands/CreateTodoItem/CreateTodoItemCommandValidator.cs) — Add validation rule
4. [`Application/TodoItems/Commands/UpdateTodoItemDetail/UpdateTodoItemDetail.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Application/TodoItems/Commands/UpdateTodoItemDetail/UpdateTodoItemDetail.cs) — Add to update command
5. [`Application/TodoItems/Queries/GetTodoItemsWithPagination/TodoItemBriefDto.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Application/TodoItems/Queries/GetTodoItemsWithPagination/TodoItemBriefDto.cs) — Add to list DTO
6. [`Application/TodoLists/Queries/GetTodos/TodoItemDto.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Application/TodoLists/Queries/GetTodos/TodoItemDto.cs) — Add to detail DTO
7. [`Infrastructure/Data/Configurations/TodoItemConfiguration.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Infrastructure/Data/Configurations/TodoItemConfiguration.cs) — Add EF Core mapping
8. [`Web/Endpoints/TodoItems.cs`](https://github.com/jasontaylordev/CleanArchitecture/blob/main/src/Web/Endpoints/TodoItems.cs) — Update endpoint mappings (if needed)
9. Plus corresponding test files in `tests/Application.FunctionalTests/TodoItems/`

You also need to rebuild 4 separate projects (Domain, Application, Infrastructure, Web) for the change to compile.

**In VSA** ([my template](https://github.com/nadirbad/VerticalSliceArchitecture)):

To add a "CancellationCategory" enum field to the CancelAppointment feature, you touch **2-3 files**:

1. [`Application/Domain/Appointment.cs`](https://github.com/nadirbad/VerticalSliceArchitecture/blob/main/src/Application/Domain/Appointment.cs) — Add property and update domain method
2. [`Application/Scheduling/CancelAppointment.cs`](https://github.com/nadirbad/VerticalSliceArchitecture/blob/main/src/Application/Scheduling/CancelAppointment.cs) — Update command, validator, handler
3. [`Infrastructure/Persistence/Configurations/AppointmentConfiguration.cs`](https://github.com/nadirbad/VerticalSliceArchitecture/blob/main/src/Application/Infrastructure/Persistence/Configurations/AppointmentConfiguration.cs) — Add EF mapping (if needed)

```csharp
// In Appointment.cs - Add property
public CancellationCategory? Category { get; private set; }

// Update Cancel method signature
public void Cancel(string reason, CancellationCategory category, DateTime? cancelledAtUtc = null)
{
    // ... existing validation ...
    Category = category;
}

// In CancelAppointment.cs - Update command
public record Command(Guid AppointmentId, string Reason, CancellationCategory Category);

// Add validation
RuleFor(v => v.Category).IsInEnum();

// Use in handler
appointment.Cancel(request.Reason, request.Category);

// In AppointmentConfiguration.cs (optional - only if custom mapping needed)
builder.Property(a => a.Category).HasConversion<int>();
```

The entire request/response flow lives in one cohesive area. Domain model, slice logic, and persistence config all sit in the Application project.

*Change impact: CA scatters changes across layers; VSA contains them in one folder.*

### Testing

Clean Architecture is **mock-heavy**. Every layer hides behind interfaces, so your unit tests often end up testing mocks of the layer below. You're testing the wiring, not the behavior.

VSA leans toward **integration testing**. A slice is a self-contained request/response, so the most useful test exercises the full slice from API to database. Save unit tests for the shared domain model, not the handlers.

Why the difference? In Clean Architecture, the layering makes it natural to test each layer in isolation. In VSA, the handler is so thin (mostly orchestration) that integration tests give you more bang for your buck. You save unit tests for the parts with complex logic domain and validators.

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

*For detailed folder organization patterns, see* [*VSA Folder Structure: 4 Approaches Compared*](/vertical-slice-architecture-folder-structure).

## Migrating From Clean Architecture to VSA

If your CA project has turned into "lasagna architecture" with too many layers, you can simplify through **defactoring**:

1. **Inline the abstractions.** Move repository and service logic back into the handler for a single feature.
    
2. **Group by feature.** Put the endpoint, command, handler, and validator in one folder.
    
3. **Delete the interfaces.** If an interface only exists to satisfy the layering, remove it.
    

This feels wrong at first. You've been taught that more abstraction is better. But if an abstraction doesn't have multiple implementations and doesn't enable testing, it's just noise.

### Cross-Cutting Concerns

"But what about logging? Validation? Transactions?"

Use a mediator pipeline (MediatR, Wolverine). Each slice stays simple while behaviors handle the cross-cutting stuff in one place. You get consistency without polluting every handler.

*For implementation details, see* [Vertical Slice Architecture template for .NET 9](https://github.com/nadirbad/VerticalSliceArchitecture).

## Choosing Based on Your Team

**Junior-heavy teams:** Lean toward Clean Architecture. The prescriptive rules ("Controller calls Service") keep code organized while people learn. The structure does some of the thinking for them.

**Experienced teams:** VSA gives you speed. Senior developers have the judgment to refactor procedural code into a rich domain model and abstractions when it makes sense, and to leave simple CRUD operations simple.

## Project Complexity Checklist

| If your project is... | Consider... | Example Template |
| :--- | :--- | :--- |
| Primarily CRUD / Data-driven | VSA or simple N-Tier | [VSA template for .NET 9](https://github.com/nadirbad/VerticalSliceArchitecture) |
| Rich, complex business rules | Clean Architecture | [Jason Taylor's Clean Architecture](https://github.com/jasontaylordev/CleanArchitecture) |
| Rapidly changing features | VSA | [VSA template for .NET 9](https://github.com/nadirbad/VerticalSliceArchitecture) |
| Long-term (5+ years) maintanance | Hybrid approach | Both patterns can evolve |
| Needs independent scalability, headed toward microservices | VSA (slices become services) | Vertical slices map to service boundaries |

## The Bottom Line

There's no universal winner. Clean Architecture protects complexity but adds ceremony. VSA reduces ceremony but requires discipline.

Start simple. If you're building a straightforward APIs, VSA will get you moving faster. If you're modeling a complex domain with lots of business rules, Clean Architecture's protected core is worth the overhead.

And don't be afraid to mix them. The best architectures I've seen treat this as a spectrum, not a binary choice.

---

## Next Steps

**New to the template?** Start with the [Quick Start Guide](/vertical-slice-architecture-template-quickstart) to get the healthcare API running in minutes.

**Deciding on folder organization?** See [VSA Folder Structure: 4 Approaches Compared](/vertical-slice-architecture-folder-structure) for practical patterns.

**Ready to try VSA?** Clone the [Vertical Slice Architecture template](https://github.com/nadirbad/VerticalSliceArchitecture) (540+ stars) — a production-ready starting point with healthcare domain examples.

**Want the full picture?** Read the [Complete Guide to Vertical Slice Architecture](/vertical-slice-architecture-dotnet) for implementation details, MediatR patterns, and testing strategies.