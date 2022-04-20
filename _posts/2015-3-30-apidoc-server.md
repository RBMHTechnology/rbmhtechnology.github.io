---
layout: post
title: APIDOC SERVER
author: fkranawetter
tag: rbmhtech
---

All of our projects are continuously built on our continuous integration servers, like most of us do. Beside publishing the regular artifacts into our maven repository, the documentation artifacts get  published as well.
Our goal was to give all our development teams a convenient access to each and every published documentation artifact, including the frequent changing snapshot artifacts.

The obvious way to accomplish this was to start up a webserver and ensure that each and every build job uploads its documentation artifacts to the webserver. But that would require a lot of work
maintaining all build jobs. As we had already all documentation artifacts in one place, namely our maven repository, we came up with the idea to serve all our documentation artifacts directly from there eliminating the need to actively change any existing or future build job. We looked for a project that does exactly that, however, to our surprise, we couldn't find any.

Today, we are happy to announce the launch of our [ApiDoc Server](https://github.com/RBMHTechnology/apidoc-server) as open source from the very beginning.

It is a web-application that transparently extracts api documentation artifacts from a maven repository and serves the content to the user. The ApiDoc Server

- serves documentation artifacts from [JCenter](http://jcenter.bintray.com/), but can be configured to other repositories with optional basic authentication support
- transparently resolves the `latest` and `release` versions for artifacts using the information provided by the maven repository
- lets you browse through the list of available versions of an artifact specified by its `groupId` and `artifactId`
- supports all kinds of zip/jar packaged documentation artifacts like javadoc, groovydoc, scaladoc and others
- downloads all documentation artifacts to a temporary local storage, but can be configured to store in a dedicated location
- caches resolved and downloaded snapshot artifacts for 30 minutes, but can be configured to any other cache timeout
- serves all artifacts from the configured repository, but can be restricted by providing a whitelist of `groupId` prefixes
- can be restricted to serve release versions only.

We hope that the ApiDoc Server serves you the same way as it helps us in our daily work.
