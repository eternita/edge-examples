# edge-examples

Runnable examples for [Frontcache](https://github.com/eternita/frontcache): an app with Frontcache in front of it or
inside it, each one a standalone project that consumes the **published** artifacts (`org.frontcache:*` from
`https://repo.eternita.co/maven2`, and the `pavlikovskiy/frontcache-server` container image) - never a Frontcache
source checkout. Copy a directory anywhere and it still runs.

| directory | what | topology |
|---|---|---|
| [frontcache-jsp/](frontcache-jsp/) | Frontcache as a servlet filter in a plain JSP web app, on an embedded Jetty 12 - `./gradlew appRun` | #1 filter |
| [frontcache-spring/](frontcache-spring/) | the same, wired the Spring Boot 3.x way - `mvn spring-boot:run` | #1 filter |
| [frontcache-php/](frontcache-php/) | a PHP app behind a standalone Frontcache: the pages only emit response headers and `<fc:include/>` markers | #2 standalone |
| [front-door/](front-door/) | nginx on 80/443 in front of Frontcache - TLS, gzip, asset bypass, an unbuffered dashboard stream - as two containers or on a VM, with a Docker smoke test | #2 / #3 |
| [images/](images/) | screenshots the JSP example's README shows | |

The topology numbers are the ones Frontcache's
[deployment use cases](https://github.com/eternita/frontcache/blob/main/docs/deployment-usecases.md) use:
**#1** filter inside a Java app, **#2** standalone proxy in front of an app in any language, **#3** GSLB → N
standalone edges → a Java origin that also runs the filter.

## Requirements

The two filter-mode examples need **JDK 25**: Frontcache is Java 25 bytecode and Jakarta EE 10 (`jakarta.servlet`), so
it will not load on an older JVM or in a `javax.servlet` container. The PHP example needs PHP and a Frontcache server
(an archive or the container image); the front door needs Docker, or an Ubuntu host for its VM variant.

```bash
git clone https://github.com/eternita/edge-examples.git
cd edge-examples/frontcache-jsp && ./gradlew appRun      # http://localhost:8080/example/index.jsp
```

Each directory's README has the rest.

## Elsewhere

- Installing Frontcache: [install guide](https://github.com/eternita/frontcache/blob/main/docs/install-guide.md)
- How it works, and the `x-frontcache-component-*` header contract: [concepts](https://github.com/eternita/frontcache/blob/main/docs/concept.md)
- Logs into Elasticsearch + Kibana: [log-analytics](https://github.com/eternita/frontcache/tree/main/log-analytics), in the Frontcache repository
- Plugins, and a template for one of your own: [eternita/edge-plugins](https://github.com/eternita/edge-plugins)

## License

Apache-2.0 - see [LICENSE](LICENSE). The vendored Bootstrap in the JSP and PHP examples keeps its own MIT license;
see [NOTICE](NOTICE).
