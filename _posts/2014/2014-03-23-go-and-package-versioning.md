---
title: Go and Package Versioning
layout: post
categories: golang
description: "An explanation of how Go handles package dependencies and multiple versions of packages."
---

*I've been working with Go on personal projects for several months. During that time, Go's intended strategy for package management and versioning has largely eluded me. My understanding really started to come together over the past week which is why I wrote about my [Go Project Structure][mygoprojectstructure] and now am writing about package versioning.*

*My goal with this post is to help others who, like myself, have had a hard time reconciling Go's approach to package dependencies and versioning with that of other languages.*

## Import Paths and `go get`

Other languages, particularly recent ones, tend to have a package manager that the community has adopted as defacto standard. Ruby has [gems][rubygems]. Node has [npm][npm]. .NET has [NuGet][nuget]. Perl has [CPAN][cpan]. Haskell has [Hackage][hackage]. All of these have something in common; **they are all a central package hosting repository.**

**Go's package manager is the `go get` command and it is completely decentralized.**

How does this work?

When you reference a package in your Go source code you use an import path that usually looks like a URL. e.g. `github.com/jpoehls/gophermail`. When you `go build` your code the Go tool uses this path to find the package in your [GOPATH][gopath]. If it can't find the package the build fails. So how do you pull down the package?

1. Manually. You can `git clone` the package into your GOPATH.

2. Use [`go get`][goget]. This is where the URL import path convention is leveraged. `go get` simply [treats the import path as a URL][remoteimportpaths] and attempts to retrieve it via HTTP or HTTPS. It is smart enough to handle Git, Mercurial, Bazaar, and Subversion. Go has special support for common hosts like GitHub and Bitbucket, but you can use it with any host and even custom URLs.

## Finding Packages

As a [Baby Gopher][babygopher] you may find it hard to locate packages that you need. e.g. "What's a good image resizing package?" You feel lost without a central repository to search.

Sites like [GoDoc.org][godoc] [Go-Search.org][gosearch] fill the role of a central repository in this regard. They don't host the packages but they index all packages stored on various hosting sites. GitHub, Bitbucket, etc. They even include packages from your custom servers.

So to answer the question, "what's a good image resizing package", all you have to do is a simple search.

- [http://godoc.org/?q=image+resize](http://godoc.org/?q=image+resize)
- [http://go-search.org/search?q=image+resize](http://go-search.org/search?q=image+resize)

> GoDoc.org was recently [adopted into the Go project][godocjoinsgo] which is very, very cool.

## Package Versioning

`go get` *is* the Go package manager. We've seen how it works in a completely decentralized way and how package discovery still possible without a central package hosting repository.

Besides locating and downloading packages, the other big role of a package manager is handling multiple versions of the same package. Go takes the most minimal and pragmatic approach of any package manager. There is no such thing as multiple versions of a Go package.

`go get` always pulls from the HEAD of the default branch in the repository. Always. This has two important implications.

> This isn't always true. If your repository has certain special branches then `go get` will pull from them instead. Specifically, if you have a `go1` branch and are running Go 1+ it will pull from that branch.


1. As a package author, you must adhere to the stable HEAD philosophy. Your default branch must always be the stable, released version of your package. You must do work in feature branches and only merge when ready to release.

2. New major versions of your package must have their own repository. Put simply, each major version of your package (following [semantic versioning][semver]) would have its own repository and thus its own import path. e.g. `github.com/jpoehls/gophermail-v1` and `github.com/jpoehls/gophermail-v2`.

It's that simple.

Another way to phrase this is that each import path must point to a stable API. Major version bumps are for backwards incompatible API changes and thus each major version is a distinct stable API. In the Go world this warrants a distinct import path and because `go get` ties import paths to VCS repositories, it means a distinct repository.

> This is exactly the explanation I wish was in the Go docs. They kind of dance around the idea but it isn't stated as clearly as it could be IMO. This is the epiphany that I needed to really understand how Go works.

## Version 2.0? New Repository.

As someone building an application in Go, the above philosophy really doesn't have a downside. Every import path is a stable API. There are no version numbers to worry about. Awesome!

> I think this must have been the Go author's focus when designing `go get`. It is a solid and simple design for Go applications. I love it. It's only frustrating from the package authoring point of view, which I'll elaborate on below.

The frustration lies with the authoring of packages. Putting different versions of your package in separate repositories is simply not the standard workflow. The standard flow is to maintain tags or branches for each version of your package.

As a package author, you think about your repository as the top level of your project and the ecosystem supports this (think GitHub). I'll use [gophermail][gophermail] as an example. I can tag bugs with version numbers and organize them into release milestones, moving features from V1 to V2, etc. I might use GitHub Pages to serve a website for the project and host the documentation.

Go shatters this world view by forcing you to break your project into multiple repositories as soon as you want a version 2.0. The level of abstraction is simply moved from the branch level to the repository level and the ecosystem doesn't cater to this methodology.

The biggest hurdle is just understanding that this is the case. That this is how Go works.

The downside is all in code organization preference. The industry standard is to use tags and branches for marking multiple versions. Not separate repositories. `go get` thinks of branches as being used solely to target different versions of Go (i.e. the special `go1` branch) and, honestly, I think that's a pretty ugly hack on their part.

## Multiple Versions in a Single Repository

There is a workaround that will allow you to keep multiple versions of your package in the same repository and use branches/tags for differentiating between them.

`go get` supports custom URLs and you can use this to insert a version number into your package's import path. Granted, this is non-trivial. It means writing and hosting a proxy service that parses URL and proxies the requests to the applicable branch/tag of your repository.

Fortunately, someone has already done the hard work for us. [GoPkg.in][gopkgin] does exactly what I've described.

I'm taking advantage of this for my gophermail package. All it means is that instead of people using `github.com/jpoehls/gophermail` to import my package, they use `gopkg.in/jpoehls/v0/gophermail`. `/v0` is because gophermail hasn't reached 1.0 yet. When I release 1.0 and declare a stable API, the import path will change to `gopkg.in/jpoehls/v1/gophermail`.

[GoPkg.in][gopkgin] is a fantastic service and I encourage anyone with a Go package to use it. All you have to do is tell your users to use the `gopkg.in` import path for your package. If this gets wide enough attention, hopefully similar functionality will be adopted into `go get` itself.

> My dream is for `go get` to support a version number component in the import path. So `github.com/jpoehls/gophermail` would fetch the HEAD of the repository as it does today. `github.com/jpoehls/gophermail/v1` would fetch the `v1` branch or tag.

[mygoprojectstructure]: {{site.url}}/2014/go-project-structure-and-dependencies
[gopath]: http://golang.org/cmd/go/#hdr-GOPATH_environment_variable
[goget]: http://golang.org/cmd/go/#hdr-Download_and_install_packages_and_dependencies
[remoteimportpaths]: http://godoc.org/code.google.com/p/go/src/cmd/go#hdr-Remote_import_paths
[rubygems]: http://rubygems.org
[npm]: https://www.npmjs.org
[nuget]: http://www.nuget.org
[cpan]: http://www.cpan.org
[hackage]: http://hackage.haskell.org
[babygopher]: http://babygopher.org
[godoc]: http://godoc.org
[gosearch]: http://go-search.org
[godocjoinsgo]: https://groups.google.com/d/topic/golang-nuts/_rbVuzl-OqA/discussion
[semver]: http://semver.org
[gophermail]: http://github.com/jpoehls/gophermail
[gopkgin]: http://gopkg.in