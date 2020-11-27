---
title: "Dotnet interactive as a query playground"
date: 2020-11-27
---

The tool I miss most when leaving ruby-on-rails is the [rails console](https://guides.rubyonrails.org/command_line.html) because it allows the me (or any developer) to run queries against the database in the same language as the rest of the codebase.
This is extremely useful for testing out queries that I'm writing in some service ( like `User.where('age > 15)`) or for ad-hoc data updates (like `User.last.update(email: 'test@test.org')`).

Recently, with the release of [dotnet interactive](https://github.com/dotnet/interactive), I've found a way to build myself a fairly close approximation.
An example notebook that loads a DLL with an appropriate DbContext and executes a few queries can be found [here](https://github.com/SaahilClaypool/BookyAPI/blob/221c8fc847ca2ba28b4347f6a094b52fc9a59b6d/Scratch.ipynb)(github).

![Interactive Example](/BookyInteractive.png)

<!--more-->

## Motivational use cases

Here are some things I might do while testing an ruby-on-rails application locally.
In an (my) ideal world, I would be able to write my type-safe C#, but still get the interactive workflow of ruby-on-rails.

1. I want my local user to be an admin

    In the ruby console, I would write `User.where(email: 'test@test.org').first.update(role: 'admin')`.

    In C#, I would *like to write* 
    
    ```cs
    var user = context.Set<ApplicationUser>().Where(u => u.Email == "test@test.org").First();
    user.role = "Admin";
    context.SaveChanges();
    ```

2. I want to remove the last objecct I created via the UI so I can try again.

    Ruby: `Bookmark.last.destroy`.

    C#: `var bookmark = context.Bookmarks.Last(); context.Bookmarks.Remove(bookmark); context.SaveChanges();`

3. I want to find a set of objects in some 'invalid' state.

    Ruby: `User.where(status: 'invalid').pluck(:id)`

    C#: `context.Users.Where(u => u.status == "invalid").Select(u => u.Id)`

None of these are very complicated, but I've found it incredibly handy to be able to instantly evaluate code I'm writing against real data.
And, with dotnet interactive, its pretty easy to set that up in a C# project.

## Building a notebook 

The steps to connect a notebook to an API context are fairly strait forward:

1. Build the project
2. Load the required entity framework nuget packages
3. Load the project DLL (containing the reference to the DbContext you want to use)
4. Create a DbContext object
5. Use the DbContext however you see fit!


### 0. Create the notebook

To get started, install the [dotnet interactive vscode extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.dotnet-interactive-vscode).
Then, from the command menu select `> .NET Interactive: Create new blank notebook`.
This will create a new blank notebook file that you can save in the root of your project.

If you don't want to commit this to version control, you can add an entry to your `.gitignore` or global gitignore file.

You can verify the extension is working and the version of dotnet being used is the latest by creating a cell with the follow code and click the `Run all cells` button.

```cs
Environment.Version
```

### 1. Build the project

The dotnet interactive can switch the language type of a cell by using the `#!` magic string.
So, I created a powershell code block and execute `dotnet build` in the project.

```powershell
#! pwsh

dotnet build
```

This should produce some output indicating which DDL was built, which will be useful in the next step

```
Microsoft (R) Build Engine version 16.8.0+126527ff1 for .NET
Copyright (C) Microsoft Corporation. All rights reserved.
  Determining projects to restore...
  All projects are up-to-date for restore.
  BookyApi -> /home/saahil/gitProjects/BookyApi/API/bin/Debug/net5.0/BookyApi.dll # <-- This is the DLL
```

### 2. Load the required entity framework nuget packages

Create a new `C#` cell and add a reference to each nuget package that you would like to use.
Each line should be in the format `#r "nuget:<package name>,<version>`.

```
#r "nuget:Microsoft.EntityFrameworkCore.Design,5.0.0"
#r "nuget:Npgsql,5.0.0"
#r "nuget:Npgsql.EntityFrameworkCore.PostgreSQL,5.0.0"
```

Clicking `Execute cell` will install these packages as normal.

### 3. Load your project DLL

To actually access your project code, you need to load the DLL much like the nuget packages.
This is done simply by padding a `#r` directive pointing to the DLL compiled in step 1.

My sample project has a line like this:
```
#r "./API/bin/Debug/net5.0/BookyApi.dll"
```

> Note: Each time you build your project, you will need to restart the Jupyter kernel powering the notebook and reload this DLL.


### 4. Create a DbContext option

I created a single cell with all the helper objects and other boilerplate down the road.
Any namespace included here with a `using` statement will be usable in all the future cells.

Creating the DbContext object is easy enough to do via a connecction string.
Once this object is created, it is globally accessible to othe other code blocks as long as the kernal is running.
(Generally, not disposing a DbContext isn't great, but for a simple interactive notebook it should be fine. This could be replaced with a factory & `using` statements).

> Note to self: it would probably be useful to set up the entire service provider here so any service could be tested via
> dependency injection.
> For example, testing a some MediatR handler could be done by simple running `serviceProvider.GetService<MyHandler>().Handle(new())`

```cs
using System.Linq;
using System.Text.Json;
using Microsoft.EntityFrameworkCore;
using BookyApi.API.Db;
using BookyApi.API.Models;
using BookyApi.API.Services.Extensions;

var options = new DbContextOptionsBuilder<BookyContext>();
options.UseNpgsql("Host=localhost;Database=BookyApI;Username=postgres;Password=example;Port=5432");
var context = new BookyContext(options.Options);
```

### 5. Use the DbContext as you see fit!

Now that there is a globally available `context` object, it is easy to use Linq and your codebase to query, update, and test your code.
Because this is all just 'plain old C# code', you can copy and paste code from parts of your project into this interactive window, or just write simple queries as top-level queries.
The last statement not ending with a semi-colon will be printed in a human-readable manner to the output window.

```cs
var books = context.Bookmarks.ToList();
books.Select(b => b.Content)

/* Output: 
index	value
0	Here comes santa
*/
```