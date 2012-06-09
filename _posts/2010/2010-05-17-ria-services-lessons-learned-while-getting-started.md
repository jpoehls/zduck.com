---
layout: post
title: RIA Services - Lessons Learned While Getting Started
categories: ria-services
---
Recently my company has been working on a rewrite of one of our line-of-business applications. We have decided to leverage RIA Services in the new application. At the moment we are also using Entity Framework; that may or may not change.

Here are a few lessons I've learned so far when working with RIA Services.

### Potentially massive (in terms of lines of code & logic) domain services

RIA doesn't handle multiple domain services well. What I mean by this is that an entity cannot be shared between multiple services. This means that in most cases you are going to have domain services with lots of methods. Worst case, think having an insert, update and delete method for each entity in your domain.

**Lesson:** Accept that having lots of methods in your domain service is inevitable.

**Aid:** Keep the methods slim by not putting any business or data access logic in them.

### Shared database entity temptation

`LinqToEntitiesDomainService<>` wraps up your Entity Framework ObjectContext and lets you access it directory in your domain service. There is also a `LinqToSqlDomainService<>`. This makes it really tempting to simply return your Entity Framework or LINQ to SQL entities from your domain service methods. This is [a very bad idea](http://davybrion.com/blog/2010/05/why-you-shouldnt-expose-your-entities-through-your-services/).

**Lesson:** Don't return your database entities from your domain service.

**Aids:**

* Create a new DTO (Data Transfer Object) for each entity you need to return from your domain service.
* Make sure your domain service operations return these DTOs and _not_ the entities that Entity Framework or LINQ to SQL generate for you.

### `DomainService.Submit()` summary

When your client application makes changes and then calls `SubmitChanges()` on the client-side, RIA Services sends this set of changes to the server as a ChangeSet. This ChangeSet will include deletes, updates and inserts - basically anything your client did before calling `SubmitChanges()`.

[`DomainService`][1] is the base class that all RIA Services inherit from. This includes the `LinqToEntitiesDomainService<>` and `LinqToSqlDomainService<>` classes.

This base class has a `Submit()` method. This is basically what is being called when your client calls `Submit()`. The DomainService's `Submit()` method handles parsing that set of changes and invoking the methods on your domain service that you have written to handle the inserts, updates and deletes of your entities.

Think of the `Submit()` method as a router that takes all the changes and routes them to the specific operations that need to happen in order to persist those changes to your backing data store.

### Using DAOs (or Repositories)

I realized early on that I didn't want to shove all my data access code directly into my domain service methods. I like to isolate the code that talks to the database in DAOs (Data Access Objects). These DAOs act as a weld point between the world of my database and the world of my application. This makes it easier for me to handle inevitable database changes or to switch out my entire data access strategy without having to rewrite the whole application. ([See why.](http://davybrion.com/blog/2010/05/why-you-shouldnt-expose-your-entities-through-your-services/))

At first I thought that to use my DAOs I couldn't/shouldn't use a `LinqToEntitiesDomainService<>` and should instead just inherit from `DomainService`. So this is how I started and I had my DAO methods handle the creation (and disposing) of my `ObjectContext` as needed.

Come to find out, this works; however, it isn't the best way to do things. As I said before, RIA Services will ask your domain service to process a whole set of changes at once. If you have 5 changes to save, do you really want to create a new `ObjectContext` for each one? Or would you rather create the `ObjectContext` once, do the 5 changes and then save them all at once to the database?

Here's what I ended up doing:

* Went back to using `LinqToEntitiesDomainService<>` as my base class.
* Updated my DAO constructors to take an instance of the `ObjectContext`.

The `LinqToEntitiesDomainService<>` will handle creating my `ObjectContext` as well as calling `SaveChanges()` on it when all the changes have been processed. This is much more efficient.

### In code

Here's a code example of what I discovered:

This is bad.

<pre data-language="generic">
public class PersonDomainService : DomainService
{
    public void UpdatePerson(Person dto)
    {
        var dao = new PersonDao();
        dao.Update(dto);
    }
}

public class PersonDao
{
    public void Update(Person dto)
    {
        using (var context = new MyObjectContext())
        {
            var existingPerson = _context.People.Where(p =&gt; p.ID == dto.ID).SingleOrDefault();
            if (existingPerson != null)
            {
                existingPerson.Name = dto.Name;
            }
        }
    }
}
</pre>

This is good.

<pre data-language="generic">
public class PersonDomainService : LinqToEntitiesDomainService&lt;MyObjectContext&gt;
{
    public void UpdatePerson(Person dto)
    {
        var dao = new PersonDao(ObjectContext);
        dao.Update(dto);
    }

    public override bool Submit(ChangeSet changeSet)
    {
        //  base.Submit() is what will take all the changes in the ChangeSet
        //  and call your insert, update and delete methods for each one.
        //  When this is all done, the ObjectContext.SaveChanges() method
        //  will  be called.
        return base.Submit(changeSet);
    }
}

public class PersonDao
{
    private readonly MyObjectContext _context;
    public PersonDao(MyObjectContext context)
    {
        _context = context;
    }

    public void Update(Person dto)
    {
        var existingPerson = _context.People.Where(p =&gt; p.ID == dto.ID).SingleOrDefault();
        if (existingPerson != null)
        {
            existingPerson.Name = dto.Name;
        }
    }
}
</pre>

### Further Reading

Here are some things I've found helpful thus far:

* [Brad Abrams' entire series on Building Amazing Business Applications with Silverlight 3](http://blogs.msdn.com/brada/archive/2009/03/17/mix09-building-amazing-business-applications-with-silverlight-3.aspx)
* [Brad Abrams' Silveright 4 + RIA Services series](http://blogs.msdn.com/brada/archive/2010/03/15/silverlight-4-ria-services-ready-for-business-index.aspx)
* [Example of using a DataSet with RIA Services](http://bradabrams.sys-con.com/node/1049931/mobile) - This is what opened my eyes to the true purpose of the `Submit()` method in the `DomainService`.
* [Example of returning DTOs from your domain service](http://blogs.msdn.com/deepm/archive/2009/11/20/wcf-ria-services-presentation-model-explained.aspx)
* [Example of Composition support in RIA Services](http://blogs.msdn.com/digital_ruminations/archive/2009/11/18/composition-support-in-ria-services.aspx)
* [Nikhil Kothari's](http://www.nikhilk.net)
* [MIX10 talk on Developing with WCF RIA Services Quickly and Effectively](http://live.visitmix.com/MIX10/Sessions/CL09)
* [Good blog post describing Change Sets in detail](http://kevindockx.blogspot.com/2009/07/working-with-ria-services-2-change-sets.html)
* [Google](http://www.google.com) --  'nuff said.

    [1]: http://msdn.microsoft.com/en-us/library/ee707373(VS.91).aspx