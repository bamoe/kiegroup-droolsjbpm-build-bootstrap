# EAP 8.1 Migration Guide

> Target: Red Hat EAP 8.1 → WildFly 35  
> Baseline: EAP 7.x (WildFly 23/27, Jakarta EE 8/9)  
> Current branch (`7.81.x-blue-jakarta`): already at WildFly 35 / Jakarta EE 10 — this document covers the **delta** required to make all library versions coherent with EAP 8.1 / WildFly 35.

---

## 1. Platform & Runtime

| Aspect | EAP 7.x (baseline) | EAP 8.1 / WF 35 (target) | Status |
|---|---|---|---|
| Java | 11 | 17 (min), 21 supported | ✅ Done (`maven.compiler.release=17`) |
| Jakarta EE | 8 (EE 7 APIs under `javax.*`) | 10 (`jakarta.*`) | ✅ Done |
| WildFly | 23.0.0.Final | **35.0.1.Final** | ✅ Done |
| WildFly Core | 22.0.0.Final | **27.0.0.Final** | ✅ Done |
| jboss-parent | 36 | **51** | ✅ Done |
| Servlet API | 4.0 (javax) | **6.1.0** (jakarta) | ✅ Done |

---

## 2. Jakarta EE API Versions — EAP 8.1 aligned

| Property | EAP 7.x (7.67.x-blue) | Current 7.81.x | EAP 8.1 / WF 35 ships | Compatible? |
|---|---|---|---|---|
| `jakarta.servlet-api` | 4.0.3 | **6.1.0** | 6.1 | ✅ |
| `jakarta.persistence-api` | 2.2.3 | **3.2.0** | 3.2 | ✅ |
| `jakarta.ws.rs-api` (JAX-RS) | 2.1.6 | **4.0.0** | 4.0 | ✅ |
| `jakarta.ejb-api` | 3.2.6 | **4.0.1** | 4.0 | ✅ |
| `jakarta.transaction-api` | 1.3.3 | **2.0.1** | 2.0 | ✅ |
| `jakarta.annotation-api` | 1.3.5 | **2.1.1** | 2.1 | ✅ |
| `jakarta.enterprise.cdi-api` | 2.0.2 | **4.1.0** | 4.1 | ✅ |
| `jakarta.inject-api` | 1.0.3 | **2.0.1** | 2.0 | ✅ |
| `jakarta.interceptor-api` | 1.2.5 | **2.2.0** | 2.2 | ✅ |
| `jakarta.jms-api` | 2.0.3 | **3.1.0** | 3.1 | ✅ |
| `jakarta.xml.bind-api` (JAXB) | 2.3.3 | **4.0.5** | 4.0 | ✅ |
| `jakarta.xml.ws-api` (JAX-WS) | 2.3.3 | **4.0.3** | 4.0 | ✅ |
| `jakarta.websocket-api` | 1.1.2 | **2.2.0** | 2.2 | ✅ |
| `jakarta.activation-api` | 1.2.2 | **2.1.4** | 2.1 | ✅ |
| `jakarta.mail-api` | 1.6.8 | **2.1.5** | 2.1 | ✅ |
| `jakarta.json-api` | 1.1.6 | **2.1.3** | 2.1 | ✅ |
| `jakarta.json.bind-api` | 1.0.2 | **3.0.0** | 3.0 | ✅ |
| `jakarta.validation-api` | _(none)_ | **3.1.1** | 3.1 | ✅ |
| `jakarta.servlet.jsp-api` | 2.3.6 | **3.1.1** | 3.1 | ✅ |
| `jakarta.enterprise.concurrent-api` | 1.1.2 | **3.1.1** | 3.1 | ✅ |
| `jakarta.resource-api` | 1.7.4 | **2.1.0** | 2.1 | ✅ |

> **Note:** `jakarta.security.jacc-api` is still at `1.6.1` (EE 8-era). EAP 8.1 ships JACC 3.0 (`jakarta.authorization-api:3.0`). This needs updating.

---

## 3. Core Framework Dependencies

| Property | 7.67.x-blue | Current 7.81.x | EAP 8.1 / WF 35 provides | Action needed |
|---|---|---|---|---|
| `org.jboss.resteasy` | 3.15.6.Final | **6.2.16.Final** | 6.2.x | ✅ Aligned |
| `org.hibernate` (ORM) | 5.4.24.Final | **6.6.18.Final** | 6.6.x | ✅ Aligned |
| `org.hibernate.validator` | 6.2.0.Final | **8.0.2.Final** | 8.0.x | ✅ Aligned |
| `org.jboss.weld.weld` (CDI) | 3.1.6.Final | **5.1.7.Final** | 5.1.x | ✅ Aligned |
| `org.jboss.weld.weld-api` | 3.1.SP3 | **5.0.SP3** | 5.0.x | ✅ Aligned |
| `org.jboss.ejb.client` | 4.0.39.Final | **5.0.0.Final** | 5.0.x | ✅ Aligned |
| `org.infinispan` | 14.0.27.Final | **15.0.20.Final** | **15.0.x** | ✅ Aligned |
| `org.keycloak` | 20.0.3 | **26.6.4** | **24.0.x** | ✅ Done (upgraded past 24.x) |
| `io.undertow` | 2.2.38.Final | **2.3.24.Final** | **2.3.x** | ✅ Aligned |
| `org.eclipse.jetty` | 9.4.57 | **12.1.10** | _not embedded_ | ✅ Aligned (Jetty used standalone only) |
| `org.apache.cxf` | 3.6.8 | **4.2.2** | 4.1.x | ✅ Compatible (minor gap) |
| `org.jboss.remoting` | 5.0.30.Final | **5.0.30.Final** | 5.0.x | ✅ Aligned |
| `org.glassfish.jaxb` (JAXB RI) | 2.3.2 | **4.0.5** | 4.0.x | ✅ Aligned |
| `org.eclipse.parsson` (JSON-P impl) | _(none)_ | **1.1.9** | 1.1.x | ✅ Aligned |

---

## 4. Maven Build / Plugin Changes

| Property | 7.67.x-blue | Current 7.81.x | EAP 8.1 required | Action |
|---|---|---|---|---|
| `maven.compiler.release` | 8 | **17** | 17+ | ✅ Done |
| `wildfly-jar-maven-plugin` (`bootable`) | n/a | **13.0.0.Final** | **13.0.x** | ✅ Done |
| `wildfly-maven-plugin` | 1.2.0.Final | **5.1.5.Final** | **5.x** | ✅ Done |
| `org.wildfly.arquillian` | 3.0.1.Final | **5.1.0.Final** | 5.1.x | ✅ Done |
| `io.takari.maven.plugins.compiler` | 1.13.5 | **2.3.0** | any | ✅ Done |
| `version.compiler.plugin` | 3.8.1 | **3.15.0** | 3.10+ | ✅ Done |

---

## 5. Process Migration Service (Quarkus / WildFly profile)

| Property | Current | EAP 8.1 target | Action |
|---|---|---|---|
| `version.io.quarkus` | 3.20.6 | 3.x (LTS) | ✅ Compatible |
| `version.org.wildfly` (test profile) | **35.0.1.Final** | **35.0.1.Final** | ✅ Done |
| Cargo `wildfly35x` container ID | `wildfly35x` | `wildfly35x` | ✅ Done |
| `cargo.container.id` | `wildfly35x` | `wildfly35x` | ✅ Done — updated in `process-migration-service/pom.xml` |

---

## 6. Spring-based Modules (droolsjbpm-integration, jbpm)

Spring Boot 3.x (with `jakarta.*` namespaces) is now in use across all modules that previously used Spring Boot 2.7.x.

| Property | Current | EAP 8.1 target | Status |
|---|---|---|---|
| `org.springframework.boot` | **3.5.16** | **3.3.x** | ✅ Done |
| `org.springframework` | **6.2.19** | **6.1.x** | ✅ Done |
| `org.springframework.security.web` | **6.5.11** | **6.3.x** | ✅ Done |
| `org.apache.tomcat` (embedded in Spring) | **10.1.60** (jakarta) | 10.1.x / 11.x | ✅ Jakarta-compatible |

> **Note:** Spring Boot 3.x uses `jakarta.*` throughout and requires Java 17+ — fully aligned with EAP 8.1. The upgrade from 2.7 to 3.x introduced significant API changes in auto-configuration, security, and data access that have been addressed in the relevant modules.

---

## 7. Other Incompatibilities / Breaking Changes

| Library | Current | Required for EAP 8.1 | Risk |
|---|---|---|---|
| `org.jboss.errai` | 5.0.0-SNAPSHOT | No official EE 10 release | 🔴 **High** — Errai 5.x is still a SNAPSHOT targeting Jakarta EE 10; GWT integration stability is uncertain |
| `org.wildfly.camel` | 11.0.1 | Not supported in WF 35 | 🔴 **High** — Camel subsystem dropped after WF 26 |
| `org.keycloak` adapter | ~~20.0.3~~ **26.6.4** | Keycloak 24+ uses OIDC adapters only; legacy WildFly adapter removed | ⚠️ Upgraded; verify OIDC adapter migration complete |
| `org.apache.deltaspike.core` | 2.0.1 | CDI 4.0 compatible | ✅ OK |
| `com.github.tomakehurst.wiremock` | replaced by `com.github.wiremock:3.13.2` | ✅ Done | ✅ |
| `org.picketlink` | 2.7.1.Final | Not compatible with EE 10 | 🔴 **High** — PicketLink is EOL; replace with Elytron |
| `com.sun.xml.bind` (JAXB RI) | removed | `org.glassfish.jaxb:4.0.5` | ✅ Done |
| `org.eclipse.yasson` (JSON-B) | 3.0.4 | 3.0.x | ✅ Aligned |

---

## 8. Deployment Descriptor / Configuration Changes

| File/Descriptor | Change required |
|---|---|
| `jboss-deployment-structure.xml` | Remove `org.picketlink` module dependency; remove `org.jboss.as.camel` |
| `jboss-web.xml` | No namespace changes needed (not Jakarta-namespaced) |
| `persistence.xml` | Schema must be `https://jakarta.ee/xml/ns/persistence` version `3.2` |
| `beans.xml` | Schema must be `https://jakarta.ee/xml/ns/cdi` version `4.0` |
| Security configuration | Replace PicketLink / legacy security subsystem with **Elytron** security domain |
| Messaging | ActiveMQ Artemis subsystem unchanged; JMS API → `jakarta.jms:3.1` ✅ |
| Datasources | IronJacamar 1.5.x included in WF 35; `jakarta.resource-api:2.1` required ✅ |

---

## 9. Summary of Required Version Changes

The following properties in `droolsjbpm-build-bootstrap/pom.xml` need updating:

```xml
<!-- EAP 8.1 / WildFly 35 alignment — current pom.xml state -->
<!-- <version.org.infinispan>15.0.20.Final</version.org.infinispan>                              ✅ Done -->
<!-- <version.io.undertow>2.3.24.Final</version.io.undertow>                                     ✅ Done -->
<!-- <version.org.keycloak>26.6.4</version.org.keycloak>                                         ✅ Done (upgraded past 24.x) -->
<!-- <version.jakarta.persistence-api>3.1.0</version.jakarta.persistence-api>                    ⚠️  At 3.1.0; EAP 8.1 ships 3.2 — consider upgrading to 3.2.0 -->
<!-- <version.jakarta.enterprise.cdi-api>4.1.0</version.jakarta.enterprise.cdi-api>              ✅ Done -->
<!-- <version.jakarta.websocket-api>2.2.0</version.jakarta.websocket-api>                        ✅ Done -->
<!-- <version.jakarta.enterprise.concurrent-api>3.1.1</version.jakarta.enterprise.concurrent-api> ✅ Done -->
<!-- <version.jakarta.authorization.api>3.0.0</version.jakarta.authorization.api>                ✅ Done (replaces jacc-api) -->
<!-- <version.wildfly.jar.maven.plugin>13.0.0.Final</version.wildfly.jar.maven.plugin>           ✅ Done -->
<!-- <version.wildfly.maven.plugin>5.1.5.Final</version.wildfly.maven.plugin>                    ✅ Done -->

<!-- Spring 3.x upgrade — DONE -->
<!-- <version.org.springframework>6.2.19</version.org.springframework>                           ✅ Done -->
<!-- <version.org.springframework.boot>3.5.16</version.org.springframework.boot>                 ✅ Done -->
<!-- <version.org.springframework.security.web>6.5.11</version.org.springframework.security.web> ✅ Done -->

<!-- process-migration-service/pom.xml — DONE -->
<!-- <version.org.wildfly>35.0.1.Final</version.org.wildfly>                                     ✅ Done -->
<!-- cargo.container.id = wildfly35x                                                              ✅ Done -->
```
