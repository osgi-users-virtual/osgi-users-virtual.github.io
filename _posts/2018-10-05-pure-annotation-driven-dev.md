---
layout: post
title:  Pure Annotation-Driven Bundle Development
date:   2018-10-05 11:00:00
categories: tooling
tags:
    - tooling
    - bnd
author:
    name: Neil Bartlett
    email: njbartlett@gmail.com
    twitter: nbartlett
    github: njbartlett
---
# Pure Annotation-Driven Bundle Development

This article desribes how to move to a purely annotation and code-driven model for building OSGi bundles. Under this model, the content of the bundle manifest is generated both from information inherent in the Java code — such as package-level dependencies — and from Java source annotations. No separate descriptor is needed, either as a separate file or as a configuration section in a Maven POM. This ensures that crucial information about the bundle stays close to where it arises: the Java code.

<!-- more -->

Contents <!-- omit in toc -->
--------

- [Introduction](#introduction)
- [Setup](#setup)
- [Package Exports and Versioning](#package-exports-and-versioning)
- [Custom Headers](#custom-headers)
- [Bundle Activator](#bundle-activator)
- [Providing and Requiring Capabilities](#providing-and-requiring-capabilities)
- [What's Left?](#whats-left)

Introduction
------------

When developing OSGi bundles with [bnd](https://bnd.bndtools.org) or any of its derivative tools (such as the various Maven and Gradle plugins), we have always generated some or most of the metadata required in the manifest. For example bnd generates the `Import-Package` header by inspecting the types referenced from the `.class` files inside your bundle. This gives bnd a massive advantage over tools that require such information to be specified manually.

Nevertheless there is often a need for additional metadata to go into the manifest, which cannot be derived directly from the bytecode. For example, bnd needs information such as the bundle version, the declaration of an activator (if present), any required and provided capabilities, and any arbitrary manifest headers that the developer wants to include. Traditionally this information was provided in a separate file in some text format. Suppose we wanted to declare a `Bundle-Activator` header. With the bnd-maven-plugin or the Gradle plugin we would create a file named `bnd.bnd` with the following content:

    Bundle-Activator: org.example.MyActivator

Alternatively with the maven-bundle-plugin, we would have to add the following to the configuration section of the POM:

```xml
<Bundle-Activator>org.example.MyActivator</Bundle-Activator>
```

Neither of these are particularly satisfactory: if we change the class name then our IDE is unlikely to know to change the name in this text file. There is also very little guidance from the IDE as to what fields are available, and what values can be set.

For these reasons, the bnd developers have been pushing for the past several years for more information to be specified with Java annotations. This makes your Java code the sole source of truth about your bundle, and it also allows referenced names to survive refactoring operations. This work started with experimental annotations in the bnd package namespace, but through the OSGi standards process they have now be standardised in OSGi Release 7.

As a result, it is now entirely practical to build a full OSGi application with zero `bnd.bnd` files in any of the projects. In this article I will describe how to achieve using annotations some of the tasks that previously called for a `bnd.bnd` file.

Setup
-----

Before starting you will need to have the OSGi R7 Annotations library on your build classpath:

```xml
<dependency>
    <groupId>org.osgi</groupId>
    <artifactId>osgi.annotation</artifactId>
    <version>7.0.0</version>
    <scope>provided</scope>
</dependency>
```

**N.B.**: you can use this dependency even if you are not yet ready to upgrade to OSGi Release 7 as your runtime platform. The annotations are only retained in the class files, so they do not create a runtime dependency.

Some of the functionality mentioned below requires bnd version 4.0.0 (the latest stable release in October 2018) or higher.

Package Exports and Versioning
------------------------------

To export a package we used to have to put something like this in the `bnd.bnd` file:

    Export-Package: org.example.api; version="1.0.0"

We can replace this with the new `@Export` and `@Version` annotations. These need to be placed on the *package*, not any single type inside the package, so the ideal place to put them is on the `package-info.java` file: 

```java
@org.osgi.annotation.bundle.Export
@org.osgi.annotation.versioning.Version("1.0.0")
package org.example.api;
```

Custom Headers
--------------

Sometimes you want to put your own custom header into the manifest, i.e. a header with an arbitrary name that is not defined by OSGi.

There are many reasons to do this. Sometimes these headers are used purely for documentation purposes, e.g. `Code-Author: Neil Bartlett`. Alternatively we can use them to declare information that will be picked up elsewhere in the application, e.g. `Help-Index: index.html; language=en_US`.

You can now declare these headers using the `@Header` annotation. This can be placed on any type in the bundle, but it's sensible to add it to a `package-info.java`, preferably on the "base" package of the bundle:

```java
@org.osgi.annotation.bundle.Header(
    name = Constants.AUTHOR_HEADER,
    value = "Neil Bartlett")
@org.osgi.annotation.bundle.Header(
    name = HelpSystem.INDEX_HEADER,
    value = "index.html;" + HelpSystem.LANGUAGE + "=" + Constants.DEFAULT_LANGUAGE)
package org.example;
```

Sometimes we want to reference the class or the package name from the value field. This can be done using macros. The following are available:

* `{@class}`: the fully-qualified class name of the type to which the annotation is attached;
* `{@class-short}`: the short class name (i.e. without package name) of the type to which the annotation is attached;
* `{@package}`: the fully qualified name of the package containing the type to which the annotation is attached;
* `{@version}`: the version of the package containing the type to which the annotation is attached.

Note that you can combine multiple macros along with fixed strings, for example:

```java
@org.osgi.annotation.bundle.Header(
    name = "Base-Package",
    value = "${@package};version='${@version}'")
@org.osgi.annotation.bundle.Export
@org.osgi.annotation.versioning.Version("1.2.3")
package org.example.api;
```

This will result in the following header (in addition to the package itself being exported with a proper version):

    Base-Package: org.example.api; version='1.2.3'

Bundle Activator
----------------

Declaring a bundle activator is simply a matter of using the `@Header` annotation with the `${@class}` macro:

```java
@Header(name = Constants.BUNDLE_ACTIVATOR, value = "${@class}")
public class MyActivator implements BundleActivator {
    // ...
}
```

This is very handy because you can now rename the activator class or move it to another package (or even bundle) and it will still be found by OSGi.

Providing and Requiring Capabilities
------------------------------------

The OSGi Capabilities and Requirements model is immensely powerful in terms of expressing dependencies between modules that are not necessarily encoded as code-level dependencies. These are the kinds of dependencies where two modules need to cooperate in order to produce useful functionality.

A good example is a servlet and a web engine. It's certainly possible to install a servlet bundle into an OSGi Framework without any resolution errors... but it won't do anything if there is no web engine to recognise it and direct HTTP requests to it. Likewise you can install a web engine but if there are no servlets present then it will not do anything useful. However if you put them together, magic happens! Typically the servlet would want to depend on there being a web engine present, but it should not specify any one particular engine such as Tomcat or Jetty. It should be able to work with whatever implementation is provided. This is done in OSGi by having the servlet bundle require a capability in the `osgi.implementation` namespace with a value of `osgi.http`. The web engine(s) should provide a corresponding capability. Now tooling such as Bndtools or the bnd-resolver-maven-plugin can construct a complete application.

Writing a provided capability used to look like this in `bnd.bnd`:

    Provide-Capability: osgi.implementation; \
        osgi.implementation=osgi.http; \
        version:Version=1.0.0

And writing a required capability looked like this:

    Require-Capability: osgi.implementation; \
        filter:='(&(osgi.implementation=osgi.http)(version>=1.0.0)(!(version>=2.0.0)))'

The complexity of this syntax has not exactly encouraged adoption of this fantastic OSGi feature. With the new `@Capability` and `@Requirement` annotations, it becomes much easier to use. We can use these annotations directly on our own classes, but they are most convenient when used as *meta-annotations*. That is, we apply these annotations to our own annotations when defining an API. For example, we can define the following annotation:

```java
@Requirement(
    namespace = ImplementationNamespace.IMPLEMENTATION_NAMESPACE,
    name = "osgi.http", version = "1.0")
public @interface RequireHttp {}
```

Now when we write a servlet class, we can annotate it simply with:

```java
@RequireHttp
public class MyServlet extends HttpServlet { ... }
```

... and the OSGi resolver knows that a bundle containing `MyServlet` should be provisioned along with an implementation of the HTTP specification, or in other words a web engine.

By the way, if you are using the HTTP Whiteboard specification from OSGi Release 7, you don't need to define your own requirement annotation as I have done above because the specification [defines one already: `@RequireHttpWhiteboard`](https://osgi.org/specification/osgi.cmpn/7.0.0/service.http.whiteboard.html#org.osgi.service.http.whiteboard.annotations.RequireHttpWhiteboard). Also you will get the requirement if you use any of the [component property types](https://osgi.org/specification/osgi.cmpn/7.0.0/service.http.whiteboard.html#org.osgi.service.http.whiteboard.propertytypes) defined by the same spec, because they are annotated with `@RequireHttpWhiteboard` (in fact this is an example of a *meta-meta-annotation*!).

Defining custom annotations for providers is less compelling than for requirers because there are usually fewer providers. But of course it is possible:

```java
@Capability(
    namespace = ImplementationNamespace.IMPLEMENTATION_NAMESPACE,
    name = "osgi.http", version = "1.0")
public @interface ProvideHttp {}

// ...

@ProvideHttp
public class JettyLauncher { ... }
```

With the availability of this feature, it is a great idea to include a set of annotations with any API you design, making it easy for both implementers and consumers to declare provided and required capabilities.

What's Left?
------------

Reviewing the `bnd.bnd` files that I could find in my various projects, the above topics – exports, versioning, activators, provided and required capabilities, custom headers – cover at least 95% of the reason for those files to exist. So what's left over?

Just occasionally you need to customise the `Import-Package` header generated by bnd. This usually happens when you are wrapping a third-party library which has messy dependencies or dead code. You can use the `@Header` annotation for this, but I wouldn't recommend it. In these cases it's better to write a `bnd.bnd` file.

Finally there is `-conditionalpackage`. Rather than defining a manifest header, this is an instruction to bnd to conditionally include extra packages in the bundle. As such it cannot be driven by an annotations currently, and you probably wouldn't want it to be. Build instructions belong in the build files!