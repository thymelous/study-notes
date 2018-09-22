---
layout: post
title: "JSP, Servlets, and Other Outdated Tech We Use"
date: 2018-09-15
---

_"If I don't have to do it, I won't. If I have to do it, I'll make it quick."_

If you haven't been scared off by the anime reference yet, I'm sure you'll
manage to get through the rest of the post.

JSP and Servlets seem like really outdated tech to me, but with my study
department's inclination toward Enterprise Java, it's no surprise we're
asked to use it for an assignment. I'll stay true to the opening quote
and try to limit my exposure to it as much as possible.

# Scala

I've wanted to try Scala out for a long time now, and this time I'm actually
going to — otherwise there's nothing to look forward to in this assignment
besides _JavaEE_.

First, I needed to [install `sbt`](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Linux.html),
Scala's build tool. On Fedora, it takes the following two commands to do it
(copypasted from the linked page):

```bash
curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
sudo dnf --enablerepo=bintray--sbt-rpm install sbt
```

Project creation is done via `sbt`, with several templates available. I couldn't
find one for Servlets, though, so I settled on a simple hello-world template:

```bash
sbt new scala/scala-seed.g8
```

# Dependencies

I included the following dependencies in `build.sbt`:

```scala
libraryDependencies += "javax.servlet" % "javax.servlet-api" % "4.0.1" % "provided",
libraryDependencies += "org.scala-lang.modules" %% "scala-xml" % "1.1.0"
```

The syntax is `groupID % artifactID % revision` (the `%%` operator appends the
Scala version to the artifact ID).

In addition to that, I added `xsbt-web-plugin`, a plugin that eases J2EE web app
development and release packaging, to `project/build.sbt`:

```scala
addSbtPlugin("com.earldouglas" % "xsbt-web-plugin" % "4.0.2")
```

# Hello, World

Here's a simple servlet I pieced together from other introductory guides:

```scala
package example

import javax.servlet.annotation.WebServlet
import javax.servlet.http.{
  HttpServlet,
  HttpServletRequest => HttpReq,
  HttpServletResponse => HttpResp}

@WebServlet(Array("/*"))
class ControllerServlet extends HttpServlet {
  def message =
    <html>
      <head><title>h</title></head>
      <body>Hello, World</body>
    </html>

  override def doGet(req: HttpReq, resp: HttpResp) =
    resp.getWriter().print(message)
}
```

It shows off a couple of neat Scala features, like import aliases and
impressive XML sugar that feels a lot like JSX (I haven't had the opportunity
to push it to the limits yet; there are likely to be some pain points).

I put the source file in `src/main/scala/example/ControllerServlet.scala`.
In addition to that, I had to declare a minimal configuration in `src/main/webapp/WEB-INF/web.xml`:

```xml
<web-app 
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0">
</web-app>
```

This instructs the application server that we're using version 3.0 of the Servlet
specification (which is kinda outdated but this is what I have to target, see
the Deployment section below). On the upside, this version already supports
annotations (`WebServlet`) — prior to it, I'd have to declare controllers and
URL mappings in XML.

# Local development

`xsbt-web-plugin` provides `jetty:start` and `jetty:stop` commands to run
a local instance of Jetty, a servlet engine and an HTTP server.

The commands are run in `sbt`:

```bash
sbt
> jetty:start
> jetty:stop
```

Jetty's stdout is piped to sbt, but you're still able to enter commands.
I didn't realize it at first and had to `^C` every time I wanted to stop
the server (`^C` also kills the sbt session).

# Preparing the release

```bash
sbt
> package
```

This produces a WAR (web application archive) file at `target/scala-2.12`.

# Deployment

My deployment target is a GlassFish application server — the 3.1.2.2 version of it,
to be precise, which was released way back in 2012 — and I've found a neat Docker image
of it, [ucalgary/glassfish](https://github.com/ucalgary/docker-glassfish). There's
just one little problem: the image is based on JRE7, so there's no support for Java 8.

The following one-liner [tweaks GlassFish to use JRE8](https://stackoverflow.com/a/22517955/1726690)
and builds a local image tagged `glassfish-3.1.2.2`:

```bash
curl -s https://raw.githubusercontent.com/ucalgary/docker-glassfish/master/Dockerfile \
  | sed "s/7u151-jdk-alpine/8u171-jdk-alpine/;/^ENTRYPOINT/d;/^COPY/d" \
  | sed "/# Ports being exposed/i RUN echo \'jre-1.8=$\{jre-1.7\}\' >> /usr/local/glassfish3/glassfish/config/osgi.properties" \
  | docker build -t glassfish-3.1.2.2 -
```

When creating a container, expose port `8080`, map the current working
directory to `/srv`, and boot an interactive `sh` session:
```
docker run -p 8080:8080 -v `pwd`:/srv --rm -it glassfish-3.1.2.2 sh
```

The first to do inside the container is to start the administration server
with `asadmin start-domain`. With the current working directory available at `/src`, GlassFish
can access the WAR file to deploy it: `asadmin deploy /srv/target/scala-2.12/example.war`.

The application is now available at `http://localhost:8080/example/`.
