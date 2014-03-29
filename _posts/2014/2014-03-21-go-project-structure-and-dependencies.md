---
title: Organizing Go Projects and Dependencies
layout: post
categories: golang
description: "My boilerplate recipe for organizing your Go project files and managing dependencies."
---

Go is build around the concept of a [GOPATH][gopath] which is a common workspace for most (or all) of your Go source code. This works well but sometimes you don't want the third party packages you depend on to be in your primary GOPATH. I can think of a few reasons for this:

- You have multiple Go apps and each depend on different versions of the same third party package. Ideally these packages would use a versioned URL so that the import paths are different but this isn't always the case. (See [http://gopkg.in](http://gopkg.in), a great tool for package authors to use for providing versioned URLs.)

	> If you want to know more about how package versioning works in Go, you should read my other post: [Go and Package Versioning]({{site.url}}/2014/go-and-package-versioning)

- You want to commit your third party packages as part of your application's code so that everything needed for a build is in one place. This guarantees that your app will always build regardless of the state of the third party repo and elminates a lot of headaches when working with a build server or multiple developers.

	> This is equivelant to committing your NuGet `./packages` folder in .NET or your npm `./node_modules` folder in Node.

I looked at many Go projects to find a solution I liked. There are a lot of approaches to this problem including [many attempts](https://code.google.com/p/go-wiki/wiki/PackageManagementTools) at a package manager for Go. The community is [very much in flux](https://groups.google.com/d/topic/golang-nuts/PLTY792AVzc/discussion) around this topic.

Personally, I felt any package manager tool was overkill for my needs. I was also discouraged by the fact that the community hasn't rallied behind any single package management solution.

In the end, this is my recipe for Go application structure.

- It uses all the standard `go get`, `go build`, and `go test` commands.

- It doesn't involve copying your source code into a temporary folder during the build.

- It doesn't require you to rewrite the import paths of third party packages.

- It modifies your GOPATH environment variable in a very simple and isolated way. This is how we prioritize our `_vendor` directory so that third party packages are pulled from there instead of our primary GOPATH.

- It wraps common commands in a Makefile. This can easily be replaced by a batch file on Windows or a `make.go` file that you can run with `go run make.go`.

	> [Camlistore](http://camlistore.org) uses the `make.go` approach ([see here](https://camlistore.googlesource.com/camlistore/+/master)) and I really like its cross platform consistency. I may switch to it at some point.

**Solution:**

For context, my system `GOPATH` is `~/mygo`.

My project is at `~/mygo/src/bitbucket.org/USERNAME/PROJECT`.

```
~/mygo/src/bitbucket.org/USERNAME/PROJECT
|-- .gitignore
|-- README
|-- Makefile
|-- _vendor # Third party packages. This is a typical GOPATH workspace.
|   `-- src
|       `-- github.com
|           |-- codegangsta
|           |   |-- inject
|           |   `-- martini
|           `-- jpoehls
|               `-- gophermail
|-- bin  # My app's binary output.
|-- docs # My app's documentation.
`-- src  # My app's source code.
    |-- main_app
    |-- some_pkg
    `-- some_other_pkg
```

My Makefile looks like this:

```
.PHONY: build doc fmt lint run test vendor_clean vendor_get vendor_update vet

# Prepend our _vendor directory to the system GOPATH
# so that import path resolution will prioritize
# our third party snapshots.
GOPATH := ${PWD}/_vendor:${GOPATH}
export GOPATH

default: build

build: vet
	go build -v -o ./bin/main_app ./src/main_app

doc:
	godoc -http=:6060 -index

# http://golang.org/cmd/go/#hdr-Run_gofmt_on_package_sources
fmt:
	go fmt ./src/...

# https://github.com/golang/lint
# go get github.com/golang/lint/golint
lint:
	golint ./src

run: build
	./bin/main_app

test:
	go test ./src/...

vendor_clean:
	rm -dRf ./_vendor/src

# We have to set GOPATH to just the _vendor
# directory to ensure that `go get` doesn't
# update packages in our primary GOPATH instead.
# This will happen if you already have the package
# installed in GOPATH since `go get` will use
# that existing location as the destination.
vendor_get: vendor_clean
	GOPATH=${PWD}/_vendor go get -d -u -v \
	github.com/jpoehls/gophermail \
	github.com/codegangsta/martini

vendor_update: vendor_get
	rm -rf `find ./_vendor/src -type d -name .git` \
	&& rm -rf `find ./_vendor/src -type d -name .hg` \
	&& rm -rf `find ./_vendor/src -type d -name .bzr` \
	&& rm -rf `find ./_vendor/src -type d -name .svn`

# http://godoc.org/code.google.com/p/go.tools/cmd/vet
# go get code.google.com/p/go.tools/cmd/vet
vet:
	go vet ./src/...
```

**Notes:**

- The standard `go get`, `go build`, and `go test` commands are used. All we do is prepend our `_vendor` workspace to the GOPATH to prioritize our snapshot of the third party packages during import path resolution.

- `vendor_update` task in the Makefile is a shotgun approach. It will update all of your dependencies to their latest version. 80% of the time this is what I want. In the 20% cases, I simply manually update the individual packages.

- The `_vendor` directory can be called anything you like but the underscore prefix is important. The [`go`][gocmd] tool will ignore any files or directories that begin with `.` or `_` ([read more](http://golang.org/cmd/go/#hdr-Description_of_package_lists)). This means we can run things like `go test` in our repo root without it picking up those third party packages.

- I've included some useful ancillary tasks as well. Such as the [`doc`][doc], [`fmt`][fmt], [`lint`][lint], and [`vet`][vet] tasks, and the fact that `build` runs [`go vet`][vet] first.

**How do you structure your Go projects?** What does your build script look like? I'm always interested in improving my methods, let me know what's working for you!

[vet]: http://godoc.org/code.google.com/p/go.tools/cmd/vet
[doc]: http://godoc.org/code.google.com/p/go.tools/cmd/godoc
[fmt]: http://golang.org/cmd/go/#hdr-Run_gofmt_on_package_sources
[lint]: https://github.com/golang/lint
[gocmd]: http://golang.org/cmd/go
[gopath]: http://golang.org/cmd/go/#hdr-GOPATH_environment_variable