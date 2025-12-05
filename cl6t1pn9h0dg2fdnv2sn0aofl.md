---
title: "Vertical Slice Architecture in .NET 9: The Ultimate Guide (2025)"
seoTitle: "Vertical Slice Architecture in .NET 9: The Ultimate Guide (2025)"
seoDescription: "Master Vertical Slice Architecture in .NET 9. The comprehensive guide covering feature folders, CQRS, and MediatR-based on the 500+ star GitHub template"
datePublished: Fri Apr 22 2022 13:30:36 GMT+0000 (Coordinated Universal Time)
cuid: cl6t1pn9h0dg2fdnv2sn0aofl
slug: vertical-slice-architecture-dotnet
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1764973706150/9ce8e9e2-5930-417e-b21b-3a6cf2385cc4.png
tags: software-design, software-architecture, net, aspnet-core, clean-code, dotnet, dotnetcore, clean-architecture, dotnet8, dotnet9

---

Back in 2022, I shared a [GitHub template for Vertical Slice Architecture](https://github.com/nadirbad/VerticalSliceArchitecture). Since then, it has garnered over 500 stars and helped countless .NET developers break free from the rigidity of 'Clean Architecture.'
But the .NET ecosystem moves fast. With the release of .NET 9 and C# 13, implementing vertical slices is cleaner and more concise than ever. This guide is a complete 2025 refresh of my original approach, covering how to organize your monolithic solution for maximum velocity.

## Traditional Layered Architecture

The first approach is the traditional layered architecture. This is a very common way to organize code and has been a standard for decades. I’m sure you have seen and used this on many projects before. A traditional layered/onion architecture approach organizes code around technical concerns in layers. In this approach, different layers are defined based on their responsibilities in the system. The layers then depend on each other so that they can collaborate and achieve their responsibilities. The dependency flow is guaranteed by forcing each layer to only depend on the ones below them (e.g., the presentation layer can only call code in the business logic layer).

For example, in a web application, we might define these four layers:

* Presentation Layer (controllers)
    
* Business Logic Layer (services)
    
* Data Access Layer (repositories)
    
* Infrastructure Layer (database)

## The Problem with a traditional layered architecture

The most significant drawback of a layered architecture is that each layer is highly coupled to the layers it depends on. This coupling makes a layered architecture rigid and difficult to maintain.

The tight coupling also makes it more difficult for developers working on different parts of the application to make changes in parallel, because one developer's work might cause problems with another developer's work. Having tight coupling between the layers, when changes are made to a feature, all the layers must be changed.

Typically if I need to change a feature in a layered application, I end up touching different layers of the application and navigating through piles of projects, folders and files. For example for a simple change in a given feature, you could be editing more than 5 files in all the layers:

* `Domain/TodoItem.cs`
    
* `Repositories/TodoItemsRepository.cs`
    
* `Services/TodoItemsService.cs`
    
* `ViewModels/TodoItemsViewModel.cs`
    
* `Controllers/TodoItemsController.cs`

Layered architecture is great for some things, but it does have major drawbacks:

* Tight coupling between layers. You can't easily swap out a layer without rewriting code in other layers. This means that if you want to make a change to one feature, you might have to touch several different layers.
    
* Each layer is aware of the next layer down (and sometimes even a few more). This makes it very difficult to understand the "big picture" at any given time and can lead to unexpected side effects when we make changes in one part of our application.
    
* It's often unclear where some components belong; should they be placed in Business Logic or Data Access? Do they go in both? Or maybe in the Presentation as well? These are questions that we need answers to before writing any code or else we will end up with big messes of tightly coupled spaghetti code.

Instead of separating based on technical concerns, Vertical Slices are about focusing on features.

## Vertical Slice Architecture

Implementing a vertical slice architecture is a good step in designing a robust software system. It's important to be familiar with it because it provides the foundation for how to structure your codebase and establish the roles and responsibilities of each part of your application.

First, let's clarify what vertical slice architecture means. A vertical slice is an architectural pattern that organizes your code by feature instead of organizing by technical concerns. For example, you can have different features for creating an admin user account versus a normal user account. Or you could have different features for checking if something has been created versus an existing item. This method helps keep track of exactly what your application does for a particular use case within the application.

It's about the idea of grouping code according to the business functionality and putting all the relevant code close together. It improves maintenance, testability, and clean separation of concerns and is easier to adapt to changes.

The Vertical Slice architecture approach is a good starting point that can be evolved later when an application becomes more sophisticated:

> We can start simple (Transaction Script) and simply refactor to the patterns that emerge from code smells we see in the business logic.
> 
> * Jimmy Bogard.
>     

## Vertical Slice vs. Clean Architecture: Key Differences

While Clean Architecture emphasizes horizontal layers (UI, Application, Domain, Infrastructure), Vertical Slice Architecture emphasizes features.

| Feature | Clean Architecture (Layers) | Vertical Slice Architecture |
| :--- | :--- | :--- |
| Primary Unit | Layers (Projects) | Features (Slices) |
| Coupling | High logical coupling within layers | Low coupling between features |
| Code Sharing | Encouraged (Shared Services) | Discouraged (Duplicate > Wrong Abstraction) |
| Change Impact | Touches multiple projects/files | Contained within one slice |
| Best For | Enterprise standardization | Rapid development & Domain complexity |

## Example Vertical Slice Architecture Project solution in C# .NET 9

Most applications start simple but they tend to change and evolve. Because of this, I wanted to create a simpler example project template that showcases the [Vertical Slice Architecture](https://github.com/nadirbad/VerticalSliceArchitecture) approach.

The goal is to stop thinking about horizontal layers and start thinking about vertical slices and organize code by Features. When the code is organized by feature you get the benefits of not having to jump around projects, folders and files. Things related to given features are placed close together.

When moving toward the vertical slices we stop thinking about layers and abstractions. The reason is the vertical slice doesn't necessarily need shared layer abstractions like repositories, services and controllers. We are more focused on concrete Feature implementation and what is the best solution to implement. With this approach, every Feature (vertical slice) is in most cases self-contained and not coupled with other slices. The features relate and share the same domain model in most cases.

![vertical-slices.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660464625999/uAQq-c-7U.png align="left")

## The change from Clean Architecture to Vertical Slice Architecture

This project repository is created based on the Clean Architecture solution template by Jason Taylor, and it uses technology choices and application business logic from this template, like:

* ASP.NET API with .NET 9
    
* CQRS with MediatR
    
* FluentValidations
    
* EF Core 9
    
* xUnit, FluentAssertions, Moq
    
* Result pattern for handling exceptions and errors using
    

I used the Clean Architecture template because it uses the CQRS pattern with the MediatR library and vertical slices naturally fit into the commands and queries.

Another approach I took (taken from Derek Comartin) about organizing code, is to put all code related to a given feature in a single file in most cases. With this approach we are having self-explanatory file names `ExportTodos.cs` and all related code close together: Api controller action methods, MediatR requests, MediatR handlers, validations, and DTOs. This is what it looks like:

```csharp
public record CreateTodoListCommand(string? Title) : IRequest<ErrorOr<int>>;

internal sealed class CreateTodoListCommandValidator : AbstractValidator<CreateTodoListCommand>
{
    private readonly ApplicationDbContext _context;

    public CreateTodoListCommandValidator(ApplicationDbContext context)
    {
        _context = context;

        RuleFor(v => v.Title)
            .NotEmpty().WithMessage("Title is required.")
            .MaximumLength(200).WithMessage("Title must not exceed 200 characters.")
            .MustAsync(BeUniqueTitle).WithMessage("The specified title already exists.");
    }

    private Task<bool> BeUniqueTitle(string title, CancellationToken cancellationToken)
    {
        return _context.TodoLists
            .AllAsync(l => l.Title != title, cancellationToken);
    }
}

internal sealed class CreateTodoListCommandHandler(ApplicationDbContext context) : IRequestHandler<CreateTodoListCommand, ErrorOr<int>>
{
    private readonly ApplicationDbContext _context = context;

    public async Task<ErrorOr<int>> Handle(CreateTodoListCommand request, CancellationToken cancellationToken)
    {
        var todoList = new TodoList { Title = request.Title };
        _context.TodoLists.Add(todoList);
        await _context.SaveChangesAsync(cancellationToken);

        return todoList.Id;
    }
}
```

## Vertical Slice Architecture and Clean Architecture

For a detailed comparison between two prominent approaches of organizing code check this [blog post from NDepend](https://blog.ndepend.com/vertical-slice-architecture-in-asp-net-core/) where Clean Architecture is compared to Vertical Slice Architecture and [Vertical Slice Architecture template .NET](https://github.com/nadirbad/VerticalSliceArchitecture).

## Why Vertical Slices Work

Moving to a feature-first mindset fixes specific pain points for growing teams.

- **Faster Delivery.** You keep all code for a feature in one place. You build, test, and ship without getting tangled in shared layers. You get business value out the door quicker.

- **Easier Maintenance.** Co-locating code reduces cognitive load. You don't have to hunt through five different projects to trace how a single button works. It’s all right there.

- **Safer Changes.** Slices stay isolated. You can change one feature without the risk of breaking an unrelated part of the app. Your team gains the confidence to deploy more often.

- **Intuitive Codebase.** The folder structure matches the business capabilities. A new developer can look at the project and immediately understand what the application actually does.

## The Trade-offs

VSA isn't a magic fix. It relies heavily on discipline.

The flexibility is both the biggest strength and the biggest risk. As Jimmy Bogard notes, VSA "does assume that your team understands code smells and refactoring."

If you ignore this, you invite chaos. The risk of code duplication turns into reality fast. You have to actively manage shared logic. If your team doesn't know how to spot code smells, you will lose the benefits of the architecture entirely.

| Benefit | Corresponding Challenge / Trade-off |
| - | - |
| Feature Isolation & Speed | Potential for Code Duplication: Without careful management, common logic (like validation or mapping) might be repeated across different slices. |
| Implementation Flexibility | Risk of Inconsistency: Each slice can solve problems differently. Without team discipline and good code reviews, this can lead to inconsistent patterns across the application. |
| Reduced Abstractions | Requires Stronger Refactoring Skills: Since there are fewer mandatory layers, developers must be skilled at identifying code smells and refactoring complex logic inside a handler to keep it clean and maintainable. |

## Example Vertical Slice Architecture project solution source code

Check out my source code on [GitHub](https://github.com/nadirbad/VerticalSliceArchitecture) for more info. If you like this please give a star to the repository :).

## Inspired by

* [Clean Architecture solution template by Jason Taylor](https://github.com/jasontaylordev/CleanArchitecture)
    
* [Vertical slice architecture by Jimmy Bogard](https://jimmybogard.com/vertical-slice-architecture/)
    
* [Organize code by Feature using Vertical Slices by Derek Comartin](https://codeopinion.com/organizing-code-by-feature-using-vertical-slices/)