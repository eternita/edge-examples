# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Runnable examples for Frontcache, a reverse-proxy / servlet-filter page-fragment cache. They moved here from the
Frontcache repository's `examples/` (`eternita/frontcache`, checked out beside this one as `../frontcache`), which
keeps the product source, the docs and the log-analytics stack. Apache-2.0.

Each top-level directory is a **standalone project**: `frontcache-jsp/` (Gradle, its own wrapper),
`frontcache-spring/` (Maven), `frontcache-php/` (nothing to build), `front-door/` (nginx, Docker / VM scripts), plus
`images/` (screenshots `frontcache-jsp/README.md` shows by relative path). There is no build at the root.

## Examples consume the published artifacts, never the source

They resolve `org.frontcache:*` from `https://repo.eternita.co/maven2` and run the published
`pavlikovskiy/frontcache-server` image, exactly as an application would - so each stays copyable on its own.
`frontcache-jsp` once declared `includeBuild('../..')` to compile against the source; its `settings.gradle` says why
that was removed. **Never reintroduce a composite build, or a relative path out of a project directory.** To try an
example against an unreleased core, `./gradlew publishToMavenLocal` in `../frontcache` and point the example at
`mavenLocal()` locally - and do not commit that.

Links into the Frontcache docs are absolute `https://github.com/eternita/frontcache/blob/main/docs/...` URLs; a
relative path out of this repository resolves to nothing on GitHub. The `docs/...` mentions in the
`FRONTCACHE_HOME/conf/*` comments are copies of the files Frontcache ships and name its docs the way those do.

## Running them

The two filter-mode examples need **JDK 25** - Frontcache is Java 25 bytecode and Jakarta EE 10 (`jakarta.servlet`),
so it will not load on an older JVM or in a `javax.servlet` container.

```bash
cd frontcache-jsp && ./gradlew appRun          # embedded Jetty 12 :8080 -> /example/index.jsp
```
```bash
cd frontcache-spring && mvn spring-boot:run    # Spring Boot 3.x :8080 -> /example
```

`frontcache-php` has nothing to build: copy the directory into a `DocumentRoot` and run a downloaded
`frontcache-server` in front of it (use case #2 - see its README).

The front-door example has an executable check of its own (it needs Docker, so nothing runs it in CI):

```bash
cd front-door && ./smoke-test.sh
```

It brings the two-container compose stack up on 8080/8443 against a stand-in origin and asserts the five things the
nginx template exists to get right (cache hit on the second request, unbuffered `/hystrix.stream`, gzip on proxied
responses, `x-frontcache-*` headers surviving the hop, `ORIGIN_PATHS` bypass). `KEEP=1` leaves the stack up;
`FC_IMAGE=…` pins a different image. Run it after touching `render-nginx-site.sh`, `docker/`, or `vm/`.

Render the nginx config without installing anything:

```bash
FRONTCACHE_HOST=fc-server ORIGIN_HOST=origin.example.com front-door/render-nginx-site.sh | less
```

Jetty in the JSP example is pinned to **12.0.38** deliberately: 12.0.30+ bundles ASM ≥ 9.9, which is required to
scan Java 25 (class file v69) bytecode. An older Jetty crashes the embedded launcher during annotation scanning.

## Conventions that are easy to break

**The Frontcache version is hard-coded here, and moves at a Frontcache release.** Every README,
`frontcache-jsp/build.gradle` (`frontcacheVersion`), `frontcache-spring/pom.xml` (`frontcache.version`), and in
`front-door/` the `docker/.env.example`, `docker/docker-compose.yml` and `smoke-test.sh` image defaults. Grep for the
old version and change every hit, together with the Frontcache repository's own docs. The examples deliberately
**pin an exact version**, so keep `latest` out of them.

**Topologies and ports are Frontcache's numbering**: **#1** filter inside a Java app, **#2** standalone proxy in
front of an app in any language, **#3** GSLB → N standalone edges → Java origin that also runs the filter. Standalone
server 9080, console 7080, the filter examples 8080/8443. `front-cache.http-port` / `https-port` are the
**client-facing** ports (80/443 behind a TLS terminator), not the port Frontcache listens on, because they drive
redirect rewriting. The server image publishes **9080** and terminates no TLS - that is why `front-door/` exists.

**The `x-frontcache-component-*` header contract** is owned by Frontcache's `docs/concept.md`;
`frontcache-php/README.md` echoes it. When it changes there, it changes here.

**Nothing here needs credentials.** API keys and TLS material are placeholders (`CHANGE_ME`, `YOUR_API_KEY`,
self-signed certs from `front-door/tls/make-self-signed.sh`).

**Build output stays out of git** - see `.gitignore` (`build/`, `target/`, `.gradle/`, each example's
`FRONTCACHE_HOME/{cache,logs}/`, and `front-door/docker/.env`).
