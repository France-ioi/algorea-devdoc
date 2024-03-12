---
layout: page
title: IDE Configuration and Processes
nav_order: 50
parent: Backend
---

# IDE Configuration and Processes

## Jetbrains GoLand or IntelliJ IDEA with Go plugin

If you're using IntelliJ IDEA, you need to install the Go plugin. Go to `File` > `Settings` > `Plugins` and search for `Go`. Install it and restart the IDE.


### Plugins

Go to `File` > `Settings` > `Plugins` and install the following plugins:
- `Cucumber+`, helps to edit the `*.feature` files, especially the tables.
- `Gherkin`, for the syntax of `*.feature` files.
- `Makefile language support`, for the syntax of the `Makefile` file.
- `Go linter`, to get the lint warnings in the IDE while editing files.
- `File Watchers`, to automatically format the code on save.


### For Windows: Add `diff` executable in the PATH

If you use Windows, you need to add the `diff` executable in the PATH.

The easiest way to do this is to:

1. Install [Git for Windows](https://git-scm.com/download/win)
2. Add the `bin` folder of Git for Windows to the PATH. The default path is `C:\Program Files\Git\usr\bin`. See the details below.
3. Restart the IDE.


#### Details:

Search "environment" in Windows Search, then select "Edit the system environment variables":

![Search "environment" in Windows Search, then select "Edit the system environment variables"]({{ site.url }}{{ site.baseurl }}/assets/intellij-idea-windows-step1.png)

Click on "Environment Variables...":

![Click on "Environment Variables...]({{ site.url }}{{ site.baseurl }}/assets/intellij-idea-windows-step2.png)

Select the `Path` variable in the `User variables` or in `System variables` section, then click on "Edit...":

![Select the `Path` variable in the `User variables` or in `System variables` section, then click on "Edit..."]({{ site.url }}{{ site.baseurl }}/assets/intellij-idea-windows-step3.png)

Click on "New", then type the path (default is C:\Program Files\Git\usr\bin), then click on "OK":

![Click on "New", then type the path (default is C:\Program Files\Git\usr\bin), then click on "OK"]({{ site.url }}{{ site.baseurl }}/assets/intellij-idea-windows-step4.png)


### Automatic code formatting on file save

Go to `File` > `Settings` > `Tools` > `File Watchers` and add a new watcher (if it is not there yet) with the following settings:
- Name: `gofumpt`
- File type: `Go`
- Scope: `Project Files`
- Program: `go`

Note: `gofumpt` is a tool that formats Go code. It is stricter than `gofmt`.
This will make sure the code and the imports are formatted correctly to pass the lint checks,
when you save a file.
