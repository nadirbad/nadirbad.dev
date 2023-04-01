---
title: "Vertical Slice Architecture in .NET"
seoTitle: "Vertical Slice Architecture Example in C# .NET - Nadir Badnjevic"
seoDescription: "The Vertical Slice Architecture style is about organizing code by features and vertical slices instead of by technical concerns. Example in .NET 7 API C#"
datePublished: Fri Apr 22 2022 13:30:36 GMT+0000 (Coordinated Universal Time)
cuid: cl6t1pn9h0dg2fdnv2sn0aofl
slug: vertical-slice-architecture-dotnet
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/f80a756f1cc8c81153b070a916e7d345.jpeg
tags: software-architecture, aspnet-core, clean-code, dotnet, clean-architecture

---

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

## Example API solution in C# .NET 7

Most applications start simple but they tend to change and evolve. Because of this, I wanted to create a simpler [project template](https://github.com/nadirbad/VerticalSliceArchitecture) that showcases the Vertical Slice Architecture approach.

The goal is to stop thinking about horizontal layers and start thinking about vertical slices and organize code by Features. When the code is organized by feature you get the benefits of not having to jump around projects, folders and files. Things related to given features are placed close together.

When moving toward the vertical slices we stop thinking about layers and abstractions. The reason is the vertical slice doesn't necessarily need shared layer abstractions like repositories, services and controllers. We are more focused on concrete Feature implementation and what is the best solution to implement. With this approach, every Feature (vertical slice) is in most cases self-contained and not coupled with other slices. The features relate and share the same domain model in most cases.

![vertical-slices.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660464625999/uAQq-c-7U.png align="left")

## The change

This project repository is created based on the Clean Architecture solution template by Jason Taylor, and it uses technology choices and application business logic from this template, like:

* ASP.NET API with .NET 7
    
* CQRS with MediatR
    
* FluentValidations
    
* EF Core 6
    
* NUnit, FluentAssertions, Moq
    

I used the Clean Architecture template because it uses the CQRS pattern with the MediatR library and vertical slices naturally fit into the commands and queries.

Another approach I took (taken from Derek Comartin) about organizing code, is to put all code related to a given feature in a single file in most cases. With this approach we are having self-explanatory file names ExportTodos.cs and all related code close together: Api controller action methods, MediatR requests, MediatR handlers, validations, and DTOs. This is what it looks like:

![example-feature.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660464646530/8qGxIe2go.png align="left")

## Benefits of Vertical Slice Architecture

By building the system around vertical slices, you can avoid making compromises between cohesion and coupling. This is achieved by keeping a low coupling between vertical slices and high cohesion within the slice.

Systems are developed and structured around Features and are use-case driven.

Also, most abstractions aren't needed, and we usually don't need shared abstractions like services and repositories.

When developing and working on features where there are fewer side effects and less paralysis surrounding the change.

When following the principle of keeping things related to each other close, the structure and navigation within the codebase become more clear and save time.

## The source code

Check out the source code on [GitHub](https://github.com/nadirbad/VerticalSliceArchitecture) for more info. If you like this please give a star to the repository :).

## Inspired by

* [Clean Architecture solution template by Jason Taylor](https://github.com/jasontaylordev/CleanArchitecture)
    
* [Vertical slice architecture by Jimmy Bogard](https://jimmybogard.com/vertical-slice-architecture/)
    
* [Organize code by Feature using Vertical Slices by Derek Comartin](https://codeopinion.com/organizing-code-by-feature-using-vertical-slices/)