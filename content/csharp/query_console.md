---
title: "Dotnet query console"
date: 2020-07-11T18:09:50-04:00
draft: false
---

As I've developed with Rails, the feature I've come to appreciate the most is the `rails console`.
It gives you an immediately connected view into your application - fully hooked up to all of your code and to your database.

<!--more-->


At my day-job, I spend much of my time writing MVC-based `Ruby-on-Rails` applications.
I've grown quite accostomed to the framework and, while I think rails does *many* things right, I like to poke my head out of the rails world every once in a while to see how other frameworks handle different features.
One of the first things that I struggled with in Asp.net core land was the lack of a console.

Here is a quick overview of the rails console: it is a REPL that loads up your entire codebase and connects to your database via whatever connection strings your development environment would normally use.
At first, I didn't rely on this repl for much - maybe just checking some logic or syntax interactively - but as I developed more features I found it *invaluable* for looking at the data.
For example, let's say you have some `Student` model and a `Assignment` model referencing the student model. 
For some reason, you need to query all of the students who have an average assignment score above 80%.
I may define `filter_by_average_score` as `Student.find_each {|student| average_score(student) > 0.8 }` where `average_score` is defined as `student.assignments.map(:score).sum / student.assignments.count`.
So - does this work? 

I could write a unit test here, testing whether the `average_score` function works, but I *probably* already have some data in my development environment that would be nice to query and check out.
In the `rails console`, that's exactly what I would do - I would simply run

```sh
rails console
> average_score(Student.first) # ignoring namespacing
... 0.6
> filter_by_average_score
... []
```

Assuming I find an issue with this function, it is super easy to reload the environment and try again[^1].

### So how do I do this in Asp.net core?

Generally, there isn't an interactive environment for asp.net core like there is for ruby.
There are solutions like [LinqPad](https://www.linqpad.net/), but after looking (briefly) into these I was not convinced of their ease of integration with an existing asp.net app.
Thus I decided to find my own solution.

The requirements for this solution are as follows: 

1. Able to run commands independent from the rest of the Asp.net jiggamrole.
2. Use the exact same database connection & models as the rest of the application.
3. No extra changes to existing code.
4. Fast.

I 'solved' this dillemma by doing the follwing: 
First, I created a `Scratch` file that would serve as the entrypoint for my 1-off queries.
Second, I wrote the necessary functions to load up the application settings (create database contexts, load connection strings).
Third, I re-wrote the `Main` method using *[Bullseye](https://github.com/adamralph/bullseye)* to look like this: 

```cs
public static void Main(string[] args) 
{
    Target("default", () => WebMain(args)); // by default, run the web app
    Target("scratch", () => new Scratch().Perform().Wait()); // 'dotnet run scratch' will run the `Scratch` class
    RunTargetsAndExit(args);
}
```

Finally, I can write and run whatever queries I want to test by attaching to a live db context.
Better yet, I can wrap all the calls into a transaction and roll that back so I can test destructive functions without worrying about the consequences.
The entire `Scratch` code is show below.

```cs
class Scratch 
{
    private void LoadConfiguration()
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json", optional: false)
            .AddEnvironmentVariables();
        Configuration = builder.Build();
    }

    public Scratch()
    {
        LoadConfiguration();
        var contextOptions = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseNpgsql(Configuration.GetConnectionString("DefaultConnection"),
            o => o.MigrationsAssembly("SharpListUI"))
                .Options;
        ContextOptions = contextOptions;
        _connection = RelationalOptionsExtension.Extract(ContextOptions).Connection;
    }

    public async Task Perform()
    {
        System.Console.WriteLine(Configuration.GetConnectionString("DefaultConnection"));
        Context = new ApplicationDbContext(ContextOptions, DummyOptions);
        using var transaction = Context.Database.BeginTransaction();

        var score = await Context.Students.First().average_score;
        Console.WriteLine($"score: {score}");

        var students = await Context.StudentsByAverage.ToListAsync();
        Console.WriteLine(students.ToJson(pretty: true));

        System.Console.WriteLine("Rolling back changes");
    }
}

```

All of this may be over engineered, and if I spent more time developing c# applications I might not feel the need to reach for the familiar interactiveness of ruby.
But, while I transition at least, I find this provides with *just enough* interaction with my application to keep me happy[^3].


[^1]: These interactive tests definitly do not replace unit tests - but for the first quick iterations I think they are faster.
[^3]: Being comfortable with SQL also goes a long way into letting you get to know your data models - more on that later.



