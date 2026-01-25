---
title: "Vertical Slice Architecture Folder Structure: 4 Approaches Compared"
seoTitle: "VSA Folder Structure: 4 .NET Project Organization Approaches"
seoDescription: "Compare 4 different folder structure approaches for Vertical Slice Architecture in .NET. Includes code examples and recommendations for organizing features."
datePublished: Sun Jan 25 2026 17:43:49 GMT+0000 (Coordinated Universal Time)
cuid: cmku12bay000202ju618ohd9h
slug: vertical-slice-architecture-folder-structure-4-approaches-compared
tags: software-development, software-architecture, software-engineering, dotnet, project-structure, vertical-slice-architecture

---

You've decided on Vertical Slice Architecture. Now comes the question that trips up every team: *how do you actually organize the folders?*

I've tried all the common approaches. Some blew up at scale. Others worked fine until someone tried to extract a microservice. Here are four that actually hold up—and when to use each.

The choice depends on your team size, domain complexity, and how much ceremony you're willing to tolerate.

*This article is part of the [Vertical Slice Architecture Series](/vertical-slice-architecture-dotnet), which covers implementation patterns, architecture comparisons, and hands-on guides.*

## Why Folder Structure Matters in VSA

Vertical Slice Architecture promises that code which changes together lives together. Your folder structure is what makes that promise real.

Get it right, and your codebase becomes a "screaming architecture." One look at the folder tree tells you what the system does. Get it wrong, and you end up with "grep architecture," where you need search to find anything.

The difference is stark. In a well-organized VSA project, a new developer can open the solution, see folders like `Scheduling`, `Prescriptions`, and `Billing`, and grasp the business capabilities within seconds. They don't need to ask "where does the appointment booking logic live?" The structure answers that question.

Bad structure, on the other hand, creates friction. You waste time navigating. You accidentally duplicate code because you didn't realize it already existed somewhere else. You hesitate to delete features because you're not sure what depends on what.

Jimmy Bogard, who coined the term Vertical Slice Architecture: take all code related to a single user request—from the UI to the database—and move it to a single location on disk. That's the north star. The four approaches below are different ways to achieve it.

> **Want my recommendation?** Skip to [My Recommended Structure](#my-recommended-structure). But understanding the trade-offs will help you adapt it to your context.

## Approach 1: Feature-Based Folders

This approach creates a separate folder for each feature, with multiple files inside representing different concerns.

### Structure Overview

```text
src/
└── Features/
    └── Scheduling/
        └── BookAppointment/
            ├── BookAppointmentCommand.cs
            ├── BookAppointmentHandler.cs
            ├── BookAppointmentValidator.cs
            ├── BookAppointmentEndpoint.cs
            └── BookAppointmentResult.cs
```

Each feature gets its own folder. Inside, you'll find separate files for the command/query, handler, validator, endpoint, and any DTOs. This mirrors the "one class per file" convention that many .NET teams follow.

### Trade-offs

The big win: clear separation makes IDE navigation straightforward. Git diffs show exactly which concern changed (handler vs. validator). Code generation tools that create one file per class work out of the box.

The downside: many small files mean lots of folder drilling to understand a complete feature. For simple CRUD operations, the overhead feels excessive. You'll end up with dozens of folders even for a modest application. And renaming a feature? You're updating multiple files and folders.

### When It Works

Large teams where multiple developers work on different aspects of the same feature simultaneously. The file separation creates natural ownership boundaries. It's also a good fit when your features have real complexity—five separate files make sense when each file contains substantial logic. For a handler that's 10 lines, it's overkill.

## Approach 2: Single File with Nested Classes

This approach collapses everything for a feature into one file using nested classes.

### Structure Overview

```text
src/
└── Application/
    └── Scheduling/
        ├── BookAppointment.cs
        ├── CancelAppointment.cs
        ├── CompleteAppointment.cs
        ├── GetAppointments.cs
        └── GetAppointmentById.cs
```

One file, one feature. The file contains the command/query record, validator, handler, and endpoint as nested classes inside a static outer class. Here's what it looks like in practice, from my [Vertical Slice Architecture template](https://github.com/nadirbad/VerticalSliceArchitecture):

```csharp
public static class BookAppointment
{
    public record Command(
        Guid PatientId,
        Guid DoctorId,
        DateTimeOffset Start,
        DateTimeOffset End,
        string? Notes) : IRequest<ErrorOr<Result>>;

    public record Result(Guid Id, DateTime StartUtc, DateTime EndUtc);

    internal static class Endpoint
    {
        public static async Task<IResult> Handle(
            Command command,
            ISender mediator)
        {
            var result = await mediator.Send(command);
            return result.Match(
                success => Results.Created($"/api/appointments/{success.Id}", success),
                errors => MinimalApiProblemHelper.Problem(errors));
        }
    }

    internal sealed class Validator : AbstractValidator<Command>
    {
        public Validator()
        {
            RuleFor(v => v.PatientId).NotEmpty();
            RuleFor(v => v.DoctorId).NotEmpty();
            RuleFor(v => v.Start).LessThan(v => v.End)
                .WithMessage("Start time must be before end time");
        }
    }

    internal sealed class Handler(ApplicationDbContext context)
        : IRequestHandler<Command, ErrorOr<Result>>
    {
        public async Task<ErrorOr<Result>> Handle(
            Command request, CancellationToken ct)
        {
            // Check for conflicts, create appointment, save...
            var appointment = Appointment.Schedule(
                request.PatientId, request.DoctorId,
                request.Start.UtcDateTime, request.End.UtcDateTime,
                request.Notes);

            context.Appointments.Add(appointment);
            await context.SaveChangesAsync(ct);

            return new Result(appointment.Id,
                appointment.StartUtc, appointment.EndUtc);
        }
    }
}
```

### Pros and Cons

**Pros:**
- Everything in one place—no jumping between files to understand a feature
- Deleting a feature means deleting one file
- Reduces ceremony for simple features
- The file name (`BookAppointment.cs`) is self-documenting
- Easier to move or refactor entire features

**Cons:**
- Files can grow large for complex features (300+ lines isn't unusual)
- Some developers dislike nested classes and find them harder to read
- IDE navigation within the file requires folding/unfolding
- Code generation tools may need customization

### When to Use This Approach

This is my preferred approach for small to medium teams building CRUD-heavy applications. When feature cohesion matters more than file size, this structure keeps you moving fast. Putting all related code close together eliminates the "scatter" problem.

The cognitive load argument is strong: when you open `BookAppointment.cs`, you see everything. No context switching. No wondering where the validator lives.

**Want to try this approach?** Check out the [Vertical Slice Architecture template](https://github.com/nadirbad/VerticalSliceArchitecture) or follow the [Quick Start Guide](/vertical-slice-architecture-template-quickstart) to set it up.

## Approach 3: Hybrid Clean + VSA

This approach keeps some horizontal layers (like a shared Domain) while organizing the Application layer by feature.

### Structure Overview

```text
src/
├── Api/
│   └── Program.cs
│
├── Domain/                    # Shared domain entities
│   ├── Appointment.cs
│   ├── Patient.cs
│   └── Doctor.cs
│
├── Application/               # Feature slices
│   ├── Scheduling/
│   │   ├── BookAppointment.cs
│   │   └── CancelAppointment.cs
│   └── Prescriptions/
│       └── CreatePrescription.cs
│
└── Infrastructure/            # Shared infrastructure
    └── Persistence/
        └── ApplicationDbContext.cs
```

The Domain and Infrastructure remain as shared horizontal layers. The Application layer uses vertical slices. This is essentially Clean Architecture with the Application layer reorganized by feature.

### Why Teams Choose This

It's familiar. Teams coming from Clean Architecture don't have to unlearn everything. The domain model stays protected in its own layer. You get a gradual migration path—change the Application layer now, deal with the rest later.

### The Catch

You still jump between layers (Domain → Application → Infrastructure). There's a real risk of drifting back toward pure Clean Architecture over time. That "protected domain" layer? It often becomes a shared dumping ground. And you won't get full isolation—features still reach across layers.

### When It Makes Sense

Teams migrating from Clean Architecture who want feature organization without abandoning familiar patterns. Also appropriate when you have a genuinely complex domain model that benefits from isolation—if your `Appointment` entity has complex business rules enforced through invariants, keeping it in a dedicated Domain layer is reasonable.

The risk: teams often adopt this as a stepping stone to "real" VSA but never complete the journey. If the hybrid becomes permanent, make that decision intentionally.

## Approach 4: Domain-Organized Slices

This approach organizes top-level folders by bounded context or business domain, with each domain containing its own features, entities, and infrastructure.

### Structure Overview

```text
src/
├── Api/
│   └── Program.cs
│
├── Scheduling/                # Bounded context
│   ├── Features/
│   │   ├── BookAppointment.cs
│   │   ├── CancelAppointment.cs
│   │   └── GetAppointments.cs
│   ├── Domain/
│   │   └── Appointment.cs
│   └── Infrastructure/
│       └── SchedulingDbContext.cs
│
├── Prescriptions/             # Bounded context
│   ├── Features/
│   │   └── CreatePrescription.cs
│   ├── Domain/
│   │   └── Prescription.cs
│   └── Infrastructure/
│       └── PrescriptionsDbContext.cs
│
└── Shared/                    # Cross-cutting kernel
    └── Common/
        └── AuditableEntity.cs
```

Each bounded context is self-contained. It has its own features, domain entities, and infrastructure. The only shared code lives in a `Shared` or `Kernel` folder for truly cross-cutting concerns.

### Pros and Cons

**Pros:**
- Natural path to a modular monolith
- Clear bounded context boundaries from day one
- Each domain can evolve independently (different patterns, different complexity)
- Easy to extract into microservices later if needed

**Cons:**
- May duplicate common infrastructure (separate DbContexts, separate validators)
- Requires upfront domain knowledge to draw the right boundaries
- More complex project structure for small applications
- Risk of creating artificial boundaries that don't match the business

### When to Use This Approach

This is the right choice when you're building a system that you expect to split into modules or microservices. If multiple teams will own different business domains, this structure sets clear ownership boundaries from the start.

It's also appropriate when bounded contexts are well-defined—because you've already done the collaborative discovery work. Techniques like [Event Storming](https://www.eventstorming.com/) or [Domain Storytelling](https://domainstorytelling.org/) help teams build shared understanding of the business domain before writing code. If you've been through workshops like these and can articulate the key use cases, processes, and where one domain ends and another begins ("Scheduling is separate from Prescriptions because..."), domain-organized slices are the obvious choice.

If you haven't done this discovery work yet, start with a simpler approach and refactor when the domain boundaries become clear through experience.

## My Recommended Structure

I combine **Approach 4 (domain-organized slices)** at the top level with **Approach 2 (single file)** inside each feature. Here's what it looks like in my [Vertical Slice Architecture template](https://github.com/nadirbad/VerticalSliceArchitecture):

```text
src/
├── Api/                          # ASP.NET Core entry point
│   └── Program.cs
│
└── Application/                  # All features and domain logic
    ├── Scheduling/               # Feature slice folder
    │   ├── BookAppointment.cs    # Command + Validator + Handler + Endpoint
    │   ├── CancelAppointment.cs
    │   ├── CompleteAppointment.cs
    │   ├── GetAppointments.cs
    │   ├── GetAppointmentById.cs
    │   └── AppointmentDto.cs     # Shared DTO for the feature area
    │
    ├── Domain/                   # Domain entities and events
    │   ├── Appointment.cs
    │   ├── Patient.cs
    │   └── Doctor.cs
    │
    ├── Common/                   # Shared infrastructure
    │   ├── Behaviours/
    │   └── Models/
    │
    └── Infrastructure/           # Data access
        └── Persistence/
            └── ApplicationDbContext.cs
```

**Why this combination?**

The domain-organized top level (`Scheduling/`, `Domain/`, `Common/`) gives me clear navigation and screaming architecture. I can see the business capabilities at a glance. The single-file features inside each area minimize ceremony and keep related code together.

The `Common/` and `Infrastructure/` folders hold genuinely shared code—MediatR behaviors, validation filters, the DbContext. These are things every feature needs, so centralizing them makes sense. But feature-specific logic stays in feature files.

This structure works for most projects I encounter: complex enough to need organization, but not so large that separate bounded context projects are justified. Start here, evolve when needed.

## Common Mistakes to Avoid

**Creating a "Shared" folder that becomes a dumping ground.** Every team does this. You create `Shared/` or `Common/` for genuinely cross-cutting code, and within months it contains business logic that belongs in specific features. Be ruthless: if code relates to a specific feature, it lives with that feature. Only true infrastructure belongs in shared folders.

**Mixing approaches inconsistently.** Pick one approach and stick with it across the codebase. I've seen projects where half the features use single files and half use separate folders. The cognitive overhead of switching mental models as you navigate the code is significant. Consistency matters more than picking the "perfect" approach.

**Over-engineering folder structure before you have features.** Don't create elaborate hierarchies for a system that has three endpoints. Start flat, and add structure when you feel the pain. Premature organization is just as problematic as premature optimization.

**Ignoring the "screaming architecture" test.** Periodically ask: can a new developer look at this folder structure and understand what the system does? If they see `Controllers/`, `Services/`, `Repositories/` instead of `Scheduling/`, `Prescriptions/`, `Billing/`, your structure is screaming about technology instead of business capabilities.

## Picking the Right Approach

| Approach | Best For | Avoid When |
|----------|----------|------------|
| Feature-Based Folders | Large teams, complex features | Simple CRUD, small teams |
| Single File (Nested Classes) | Small-medium teams, CRUD apps | Very complex features (300+ lines) |
| Hybrid Clean + VSA | Clean Architecture migrations | Greenfield projects |
| Domain-Organized Slices | Modular monoliths, multi-team | Unclear domain boundaries |

There's no universally correct answer. The best structure is the one your team can maintain consistently.

Start with the single-file approach (Approach 2) if you're unsure. It delivers most of the VSA benefits with minimal overhead. You can always refactor toward feature folders or domain organization as your application grows.

For a deeper dive into Vertical Slice Architecture itself—including CQRS patterns, MediatR setup, and testing strategies—see [Vertical Slice Architecture in .NET 10: The Ultimate Guide](/vertical-slice-architecture-dotnet) or explore [how VSA compares to Clean Architecture](/vertical-slice-vs-clean-architecture).

---

**Stop debating folder structure in code reviews.** Clone the [Vertical Slice Architecture template](https://github.com/nadirbad/VerticalSliceArchitecture) and start building. It uses the single-file approach with domain organization - the combination I recommend for most .NET projects.