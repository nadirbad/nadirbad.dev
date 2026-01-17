---
title: "Getting Started with the Vertical Slice Architecture Template for .NET 9"
seoTitle: "Vertical Slice Architecture Template for .NET 9: Quick Start Guide"
seoDescription: "Get started with the Vertical Slice Architecture template for .NET 9. Clone, configure, and run a production-ready healthcare API in minutes."
datePublished: Sat Jan 17 2026 19:52:35 GMT+0000 (Coordinated Universal Time)
cuid: cmkiq53fu000d02kz7au57h2s
slug: vertical-slice-architecture-template-quickstart
tags: software-design, software-architecture, software-engineering, dotnet, dotnetcore, c-sharp, aspnetcore, dotnet9

---

I built this template because I was tired of creating the same boilerplate every time I started a new project. After [writing about Vertical Slice Architecture](/vertical-slice-architecture-dotnet) and seeing the pattern gain traction, I wanted a starting point that showed how all the pieces fit together in a real domain—not another todo app.

This guide walks you through cloning the template, running it, understanding its structure, and adding your first feature.

## What's Included in the Template

The [Vertical Slice Architecture template](https://github.com/nadirbad/VerticalSliceArchitecture) models a medical clinic appointment scheduling system. Patients book appointments with doctors, and the system enforces real-world constraints: no double-booking, appointments scheduled in advance, and controlled state transitions.

**Features implemented:**

| Feature | Endpoint | What It Does |
|---------|----------|--------------|
| Book Appointment | `POST /api/appointments` | Creates appointment with conflict detection |
| Get Appointments | `GET /api/appointments` | Filtered, paginated list |
| Get by ID | `GET /api/appointments/{id}` | Single appointment with details |
| Complete | `POST /api/appointments/{id}/complete` | Marks appointment done |
| Cancel | `POST /api/appointments/{id}/cancel` | Cancels with required reason |

**Tech stack:**

- .NET 9 Minimal APIs
- MediatR for request/response pattern
- FluentValidation for declarative validation
- ErrorOr for result pattern error handling
- Entity Framework Core 9 (in-memory or SQL Server)
- xUnit + FluentAssertions for testing

The template has over 540 stars on GitHub. It's not production-ready as-is. Think of it as a learning tool that shows patterns you can adapt for your own projects.

## Prerequisites

### .NET 9 SDK

Download from [dot.net](https://dot.net/download). Verify your installation:

```bash
dotnet --version
# Should output 9.0.x
```

### IDE Options

Pick what works for you:

- **Visual Studio 2022** (17.8+) — Full IDE experience, best debugging
- **JetBrains Rider** — Cross-platform, strong refactoring tools
- **VS Code** — Lightweight, free

### Optional: Docker for SQL Server

The template runs with an in-memory database by default—no setup required. If you want persistent data with SQL Server, you'll need Docker:

```bash
docker pull mcr.microsoft.com/azure-sql-edge:latest
```

Azure SQL Edge works on all platforms including Apple Silicon.

## Installation and Setup

### Clone the Repository

```bash
git clone https://github.com/nadirbad/VerticalSliceArchitecture.git
cd VerticalSliceArchitecture
```

### Database Configuration

**Option 1: In-Memory (Default)**

Nothing to configure. The template uses an in-memory database by default. Sample data gets seeded automatically in development mode.

**Option 2: SQL Server**

Start a SQL Server container:

```bash
docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge
```

Update `src/Api/appsettings.json`:

```json
{
  "UseInMemoryDatabase": false,
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=VerticalSliceDb;User Id=sa;Password=yourStrong(!)Password;TrustServerCertificate=True"
  }
}
```

Apply migrations:

```bash
dotnet ef database update --project src/Application --startup-project src/Api
```

### Running the API

```bash
dotnet run --project src/Api/Api.csproj
```

Open [https://localhost:7098](https://localhost:7098) in your browser. Swagger UI loads at the root with sample patient and doctor IDs ready to use.

Try booking an appointment:

```bash
curl -X POST https://localhost:7098/api/appointments \
  -H "Content-Type: application/json" \
  -d '{
    "patientId": "11111111-1111-1111-1111-111111111111",
    "doctorId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
    "start": "2027-01-20T09:00:00Z",
    "end": "2027-01-20T09:30:00Z",
    "notes": "Annual checkup"
  }'
```

## Exploring the Codebase

### Solution Structure

```text
src/
├── Api/                          # ASP.NET Core entry point
│   └── Program.cs                # Minimal hosting, Swagger config
│
└── Application/                  # All features and domain logic
    ├── Scheduling/               # Feature slices
    │   ├── BookAppointment.cs
    │   ├── CancelAppointment.cs
    │   ├── CompleteAppointment.cs
    │   ├── GetAppointments.cs
    │   └── GetAppointmentById.cs
    │
    ├── Domain/                   # Entities and domain events
    │   ├── Appointment.cs
    │   ├── Patient.cs
    │   └── Doctor.cs
    │
    ├── Common/                   # Shared infrastructure
    │   └── Behaviours/           # MediatR pipeline behaviors
    │
    └── Infrastructure/
        └── Persistence/          # EF Core DbContext
```

Notice the `Scheduling/` folder. Each file is a complete feature—command, validator, handler, and endpoint in one place. No jumping between Controllers, Services, and Repositories folders.

### Feature Anatomy

Open `BookAppointment.cs`. A single file contains four components:

```csharp
public static class BookAppointment
{
    // 1. Command - What the caller sends
    public record Command(
        Guid PatientId,
        Guid DoctorId,
        DateTimeOffset Start,
        DateTimeOffset End,
        string? Notes) : IRequest<ErrorOr<Result>>;

    // 2. Result - What the caller gets back
    public record Result(Guid Id, DateTime StartUtc, DateTime EndUtc);

    // 3. Endpoint - HTTP binding
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

    // 4. Validator - Input validation
    internal sealed class Validator : AbstractValidator<Command>
    {
        public Validator()
        {
            RuleFor(v => v.PatientId).NotEmpty();
            RuleFor(v => v.DoctorId).NotEmpty();
            RuleFor(v => v.Start).LessThan(v => v.End);
            // ... more rules
        }
    }

    // 5. Handler - Business logic
    internal sealed class Handler : IRequestHandler<Command, ErrorOr<Result>>
    {
        public async Task<ErrorOr<Result>> Handle(
            Command request,
            CancellationToken cancellationToken)
        {
            // Check conflicts, create appointment, save
        }
    }
}
```

Everything for booking an appointment lives in one file. When requirements change, you modify one file. When you review a PR, you see the complete context.

### Testing Setup

Tests mirror the feature structure:

```bash
dotnet test

# Unit tests - validators and domain logic
dotnet test tests/Application.UnitTests

# Integration tests - API endpoints
dotnet test tests/Application.IntegrationTests
```

Unit tests use FluentValidation's `TestValidate` helper:

```csharp
public class BookAppointmentValidatorTests
{
    private readonly BookAppointment.Validator _validator = new();

    [Fact]
    public void Should_Have_Error_When_PatientId_Is_Empty()
    {
        var command = new BookAppointment.Command(
            Guid.Empty, Guid.NewGuid(),
            DateTimeOffset.UtcNow.AddHours(1),
            DateTimeOffset.UtcNow.AddHours(2),
            null);

        var result = _validator.TestValidate(command);
        result.ShouldHaveValidationErrorFor(x => x.PatientId);
    }
}
```

## Creating Your First Feature

Let's add a `RescheduleAppointment` feature that moves an existing appointment to a new time slot.

### Step 1: Create the Feature File

Create `src/Application/Scheduling/RescheduleAppointment.cs`:

```csharp
using ErrorOr;
using FluentValidation;
using MediatR;
using Microsoft.AspNetCore.Http;
using Microsoft.EntityFrameworkCore;
using VerticalSliceArchitecture.Application.Common;
using VerticalSliceArchitecture.Application.Domain;
using VerticalSliceArchitecture.Application.Infrastructure.Persistence;

namespace VerticalSliceArchitecture.Application.Scheduling;

public static class RescheduleAppointment
{
    public record Command(
        Guid AppointmentId,
        DateTimeOffset NewStart,
        DateTimeOffset NewEnd) : IRequest<ErrorOr<Result>>;

    public record Result(Guid Id, DateTime StartUtc, DateTime EndUtc);

    internal static class Endpoint
    {
        public static async Task<IResult> Handle(
            Guid appointmentId,
            Command command,
            ISender mediator)
        {
            if (appointmentId != command.AppointmentId)
            {
                return Results.BadRequest(new { error = "Route ID does not match command" });
            }

            var result = await mediator.Send(command);
            return result.Match(
                success => Results.Ok(success),
                errors => MinimalApiProblemHelper.Problem(errors));
        }
    }

    internal sealed class Validator : AbstractValidator<Command>
    {
        public Validator()
        {
            RuleFor(v => v.AppointmentId).NotEmpty();
            RuleFor(v => v.NewStart).LessThan(v => v.NewEnd)
                .WithMessage("Start time must be before end time");
            RuleFor(v => v.NewStart)
                .Must(start => start > DateTimeOffset.UtcNow.AddMinutes(15))
                .WithMessage("Must reschedule at least 15 minutes in advance");
        }
    }

    internal sealed class Handler(ApplicationDbContext context)
        : IRequestHandler<Command, ErrorOr<Result>>
    {
        public async Task<ErrorOr<Result>> Handle(
            Command request,
            CancellationToken cancellationToken)
        {
            var appointment = await context.Appointments
                .FirstOrDefaultAsync(a => a.Id == request.AppointmentId, cancellationToken);

            if (appointment is null)
            {
                return Error.NotFound("Appointment.NotFound",
                    $"Appointment {request.AppointmentId} not found");
            }

            if (appointment.Status != AppointmentStatus.Scheduled)
            {
                return Error.Validation("Appointment.CannotReschedule",
                    "Only scheduled appointments can be rescheduled");
            }

            // Check for conflicts at new time
            var hasConflict = await context.Appointments
                .AnyAsync(a =>
                    a.Id != request.AppointmentId &&
                    a.DoctorId == appointment.DoctorId &&
                    a.Status == AppointmentStatus.Scheduled &&
                    a.StartUtc < request.NewEnd.UtcDateTime &&
                    a.EndUtc > request.NewStart.UtcDateTime,
                    cancellationToken);

            if (hasConflict)
            {
                return Error.Conflict("Appointment.Conflict",
                    "Doctor has a conflicting appointment at the new time");
            }

            // Update times (you'd add a Reschedule method to the domain entity)
            // For now, this shows the pattern
            await context.SaveChangesAsync(cancellationToken);

            return new Result(appointment.Id, appointment.StartUtc, appointment.EndUtc);
        }
    }
}
```

### Step 2: Register the Endpoint

Open `src/Application/Scheduling/AppointmentEndpoints.cs` and add:

```csharp
group.MapPost("/{appointmentId:guid}/reschedule", RescheduleAppointment.Endpoint.Handle)
    .WithName("RescheduleAppointment")
    .WithSummary("Reschedule an existing appointment")
    .Produces<RescheduleAppointment.Result>()
    .ProducesProblem(StatusCodes.Status404NotFound)
    .ProducesProblem(StatusCodes.Status409Conflict);
```

### Step 3: Add a Test

Create `tests/Application.UnitTests/Scheduling/RescheduleAppointmentValidatorTests.cs`:

```csharp
using FluentValidation.TestHelper;
using VerticalSliceArchitecture.Application.Scheduling;

namespace VerticalSliceArchitecture.Application.UnitTests.Scheduling;

public class RescheduleAppointmentValidatorTests
{
    private readonly RescheduleAppointment.Validator _validator = new();

    [Fact]
    public void Should_Have_Error_When_Start_After_End()
    {
        var command = new RescheduleAppointment.Command(
            Guid.NewGuid(),
            DateTimeOffset.UtcNow.AddHours(2),
            DateTimeOffset.UtcNow.AddHours(1));

        var result = _validator.TestValidate(command);
        result.ShouldHaveValidationErrorFor(x => x.NewStart);
    }

    [Fact]
    public void Should_Not_Have_Error_When_Valid()
    {
        var start = DateTimeOffset.UtcNow.AddHours(1);
        var command = new RescheduleAppointment.Command(
            Guid.NewGuid(), start, start.AddHours(1));

        var result = _validator.TestValidate(command);
        result.ShouldNotHaveAnyValidationErrors();
    }
}
```

Run tests to verify:

```bash
dotnet test
```

## Customization Options

### Switching Databases

The `ConfigureServices.cs` file controls database selection:

```csharp
public static IServiceCollection AddInfrastructure(
    this IServiceCollection services,
    IConfiguration configuration)
{
    if (configuration.GetValue<bool>("UseInMemoryDatabase"))
    {
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseInMemoryDatabase("VerticalSliceDb"));
    }
    else
    {
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(
                configuration.GetConnectionString("DefaultConnection")));
    }
    // ...
}
```

For PostgreSQL, swap `UseSqlServer` for `UseNpgsql` and add the Npgsql.EntityFrameworkCore.PostgreSQL package.

### Adding New Domains

The template focuses on scheduling, but you can add parallel domains:

1. Create a new folder under `Application/` (e.g., `Billing/`)
2. Add feature files following the same pattern
3. Create a domain entity in `Domain/`
4. Register endpoints in a new `BillingEndpoints.cs`
5. Map endpoints in `Program.cs`: `app.MapBillingEndpoints()`

Keep domains isolated. Features within a domain can share its entities, but don't couple domains to each other.

### Modifying Pipeline Behaviors

MediatR pipeline behaviors run for every request. The template includes:

- **ValidationBehaviour** — Runs FluentValidation before the handler
- **PerformanceBehaviour** — Logs slow requests

Add your own in `Common/Behaviours/`:

```csharp
public class LoggingBehaviour<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        // Log before
        var response = await next();
        // Log after
        return response;
    }
}
```

Register in `ConfigureServices.cs`:

```csharp
options.AddOpenBehavior(typeof(LoggingBehaviour<,>));
```

## Next Steps

From here:

- **Understand the theory**: Read [Vertical Slice Architecture in .NET: The Ultimate Guide](/vertical-slice-architecture-dotnet) for the architectural reasoning behind this approach
- **Compare alternatives**: See [Vertical Slice vs. Clean Architecture: Which Should You Choose?](/vertical-slice-vs-clean-architecture) if you're deciding between patterns
- **Contribute**: The [template repo](https://github.com/nadirbad/VerticalSliceArchitecture) welcomes issues and PRs

The template is a starting point, not a framework. Take what works for your domain. Skip what doesn't.

---

*Found this useful? Star the [GitHub repo](https://github.com/nadirbad/VerticalSliceArchitecture) to help others discover it.*