---
title: Using enum flags to model user roles in C#, SQL, and Go
layout: post
categories: c# sql golang
description: "Using an enum flags property to hold users' roles is just one trick I've learned from @SeiginoRaikou, one of my co-geniuses at InterWorks."
---

Using an enum flags property to hold users' roles is just one trick
I've learned from [@SeiginoRaikou](https://twitter.com/SeiginoRaikou),
one of my co-geniuses at [InterWorks](https://www.interworks.com/).

Historically I have always modeled user roles with a `Role` table
and a many-to-many relationship with my `User` table.

<pre>    +-------------------+
    | User              |
    |-------------------|
    | UserId   : int    +-------------+---------------+
    | UserName : varchar|             | UserRole      |
    +-------------------+             |---------------|
                                      + UserId : int  |
    +-------------------+             + RoleId : int  |
    | Role              |             |---------------+
    |-------------------|             |
    | RoleId   : int    +-------------+
    | RoleName : varchar|
    +-------------------+</pre>

Compared to a flags enum approach this is downright wasteful.
With flags, there is only ever one column to work with and it is an integer.
No second table. No joins or second query.

<pre>    +--------------------+
    | User               |
    |--------------------|
    | UserId   : int     |
    | UserName : varchar |
    | Roles    : int     |
    +--------------------+</pre>

Modeling this in C# is easy.

<pre data-language="csharp">
[Flags] // Values must always be powers of two
enum UserRole : int {
  Employee = 1,
  Manager = 2,
  Admin = 4
}

class User {
  public UserRole Role { get; set; }
 
  public void AddRole(UserRole roleToAdd) {
    this.Role |= roleToAdd;
  }
 
  public void RemoveRole(UserRole roleToRemove) {
    this.Role &amp;= ~roleToRemove;
  }
  
  public bool IsInRole(UserRole roleToCheck) {
    return this.Role.HasFlag(roleToCheck);
  }
}
</pre>

Flag enumerations are just as accessible in SQL.

<pre data-language="sql">
-- Add users to the Admin role
UPDATE [User] SET [Roles] = ([Roles] | 4)

-- Remove users from the Admin role
UPDATE [User] SET [Roles] = ([Roles] - ([Roles] &amp; 4))

-- Find all users in the Admin role
SELECT * FROM [User] WHERE [Roles] &amp; 4 &lt;&gt; 0
</pre>

Go forth and simplify, my friends.

**Update for Go**

Bonus! Here is how you might model this in [Go](http://www.golang.org).

<pre data-language="golang">
type User struct {
    Roles    Roles
}

type Roles uint

const (
    // Roles must be powers of two.
    RoleEmployee Roles = 1
    RoleManager        = 2
    RoleAdmin          = 4
)

func (user *User) AddRole(role Roles) {
  user.Roles |= role
}

func (user *User) RemoveRole(role Roles) {
  user.Roles &amp;^= role
}

func (user *User) IsInRole(role Roles) bool {
  return (user.Roles &amp; role) != 0
}
</pre>

An even cleaner way would be to use [`iota`](http://golang.org/pkg/builtin/#iota) to have Go automatically assign the flag values in powers of two. You can see this method used by Go's own [text/tabwriter](http://golang.org/pkg/text/tabwriter/#pkg-constants) package.

<pre data-language="golang">
const (
  // Roles must be powers of two.
  RoleEmployee Roles = 1 &lt;&lt; iota
  RoleManager
  RoleAdmin
)
</pre>