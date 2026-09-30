# Tomcat 10 Upgrade — Impacted Repositories

Migration of Tomcat to version 10 for use with Java 17 and the Jakarta EE namespace.

---

## Overview

Tomcat 10 is the first Tomcat release to implement the Jakarta EE 9+ specifications, meaning all `javax.*` servlet/security namespaces are replaced by `jakarta.*`. The changes required span version management (BOM), source code imports, integration adapters, embedded Tomcat overrides in Spring Boot modules, and test container profiles.

---

## 1. `droolsjbpm-build-bootstrap` — Central version management 🔴 Critical

This is the BOM (Bill of Materials) and the most critical starting point. All version properties are centralized here.

| File | What needs changing |
|---|---|
| `pom.xml` (line 155) | `version.org.apache.tomcat` is already `10.1.56` ✅ — verify Spring Boot version alignment |
| `narayana-integration-bom/pom.xml` (line 28) | `version.org.apache.tomcat.tomcat-dbcp` is still `10.1.34` — must be aligned to `10.1.56` |
| `narayana-integration-bom/pom.xml` (line 24) | `version.org.jboss.integration.narayana-tomcat` — Narayana's Tomcat integration module must be compatible with Tomcat 10 |
| `kie-bom/pom.xml` (line 2617) | References `kie-tomcat-integration` — needs review once that module is migrated |

---

## 2. `droolsjbpm-integration` — Most impacted repo 🔴 Critical

### 2a. `kie-tomcat-integration` — Core integration module, code changes required

This module directly implements Tomcat Valves and Realms using `org.apache.catalina.*` APIs.

| File | Issue |
|---|---|
| `kie-tomcat-integration/src/main/java/org/kie/integration/tomcat/JACCValve.java` (line 57) | Uses `"javax.security.auth.Subject.container"` key — in Tomcat 10 / Jakarta EE 10 this key is **unchanged** (it is a JAAS/JSSE constant, not a Jakarta one) but the code comment at line 88 notes a Tomcat 10 removal; needs review |
| `kie-tomcat-integration/src/main/java/org/kie/integration/tomcat/TomcatRealmLoginModule.java` (line 23) | Imports `javax.security.auth.*` — these are from the JDK, not Jakarta EE, so they stay as `javax.*` ✅ |
| `kie-tomcat-integration/pom.xml` (line 20) | Depends on `tomcat-catalina` — version must come from BOM |

### 2b. `kie-spring-boot` and sub-modules — Embedded Tomcat upgrade

These modules explicitly override Spring Boot's embedded Tomcat with their own version to fix CVEs. With Tomcat 10, the override pattern must continue but targeting `10.x`.

> ⚠️ **Key blocker**: Tomcat 10 (embedded) requires **Spring Boot 3.x**, which in turn requires **Spring Framework 6.x** and **Java 17**. If these modules still use Spring Boot 2.x, this is a major migration dependency.

| File | Issue |
|---|---|
| `kie-spring-boot/pom.xml` (lines 59–138) | `tomcat-embed-core`, `tomcat-embed-websocket`, `tomcat-embed-el` overrides |
| `kie-spring-boot-autoconfiguration/kie-server-spring-boot-autoconfiguration/pom.xml` (line 40) | Same Tomcat embed overrides |
| `kie-spring-boot-starters/kie-server-spring-boot-starter/pom.xml` (line 26) | Same Tomcat embed overrides |
| `kie-spring-boot-starters/kie-server-spring-boot-starter-drools/pom.xml` (line 24) | Same Tomcat embed overrides |
| `kie-spring-boot-starters/kie-server-spring-boot-starter-jbpm/pom.xml` (line 26) | Same Tomcat embed overrides |
| `kie-spring-boot-samples/jbpm-spring-boot-sample-basic/pom.xml` (line 55) | Same Tomcat embed overrides |

### 2c. `kie-server-wars` — Servlet API namespace

| File | Issue |
|---|---|
| `kie-server-parent/kie-server-wars/kie-server/pom.xml` (line 37) | Still has `javax.servlet:javax.servlet-api` — must become `jakarta.servlet:jakarta.servlet-api` |
| `kie-server-parent/kie-server-wars/kie-server/src/main/webc-resources/WEB-INF/web.xml` (line 8) | References `NarayanaJtaServletContextListener` — needs Narayana Tomcat 10 compatible version |
| `kie-server-parent/kie-server-wars/kie-server/src/main/webc-resources/META-INF/context.xml` (line 4) | Uses Narayana JTA factory classes — version compatibility check needed |

### 2d. `kie-server-tests` — Integration test container profile

| File | Issue |
|---|---|
| `kie-server-parent/kie-server-tests/pom.xml` (line 559) | Has a `tomcat9` Cargo profile (`tomcat9x`); needs a `tomcat10` profile with `cargo.container.id=tomcat10x` |

---

## 3. `jbpm` — Container tests and transaction config 🟠 High

| File | Issue |
|---|---|
| `jbpm-container-test/jbpm-in-container-test/jbpm-container-test-suite/pom.xml` (line 705) | `tomcat9` Cargo profile + `arquillian-tomcat-remote-7` adapter — needs update to Tomcat 10 equivalent Arquillian adapter |
| `jbpm-container-test/jbpm-in-container-test/shrinkwrap-war-profiles/pom.xml` (line 200) | Profile ID `shrinkwrap-ews-tomcat-9` — needs a Tomcat 10 equivalent |
| `jbpm-container-test/jbpm-in-container-test/jbpm-container-test-suite/src/test/resources/org/jbpm/test/container/archive/localtransactions/tomcat-context.xml` | Narayana JTA factory bindings — version compatibility check required |
| `jbpm-test-util/pom.xml` (line 56) | Uses `tomcat-dbcp` — version comes from BOM, verify alignment |
| `jbpm-xes/pom.xml` (line 107) | Uses `tomcat-dbcp` + Java source imports `org.apache.tomcat.dbcp.dbcp2.*` |

---

## 4. `jbpm-work-items` — Embedded Tomcat + javax.servlet 🟠 High

| File | Issue |
|---|---|
| `jbpm-workitem-itests/pom.xml` (line 85) | Explicit `tomcat-embed-*` overrides, same pattern as `kie-spring-boot` |
| `repository-springboot/pom.xml` (line 79) | Same Tomcat embed overrides |
| `long-running-rest-workitem/src/test/java/org/jbpm/process/longrest/mockserver/WorkItems.java` (line 20) | `import javax.servlet.ServletContext` — must become `import jakarta.servlet.ServletContext` |
| `long-running-rest-workitem/src/test/java/org/jbpm/process/longrest/demoservices/Service.java` (line 33) | Same `javax.servlet` → `jakarta.servlet` import |

---

## 5. `drools` — DBCP connection pooling (test utilities) 🟡 Medium

| File | Issue |
|---|---|
| `kie-test-util/pom.xml` (line 52) | Uses `tomcat-dbcp` — version managed by BOM, no local override needed |
| `kie-test-util/src/main/java/org/kie/test/util/db/internal/PoolingDataSourceFactory.java` (line 35) | Imports `org.apache.tomcat.dbcp.dbcp2.*` — package name is **unchanged** in Tomcat 10 ✅ |

---

## 6. `kie-soup` — DBCP connection pooling (test utilities) 🟡 Medium

| File | Issue |
|---|---|
| `kie-soup-dataset/kie-soup-dataset-sql-tests/pom.xml` (line 45) | Has its **own local** `version.tomcat.dbcp=9.0.13` — this is **hardcoded to Tomcat 9** and must be updated to a `10.x` version |
| `kie-soup-dataset/kie-soup-dataset-sql-tests/src/test/java/.../PoolingDataSourceFactory.java` (line 35) | Imports `org.apache.tomcat.dbcp.dbcp2.*` — package name unchanged ✅ |

---

## Summary Table

| Repo | Priority | Type of change |
|---|---|---|
| `droolsjbpm-build-bootstrap` | 🔴 Critical | Version alignment in BOM / `narayana-integration-bom` |
| `droolsjbpm-integration` | 🔴 Critical | Source code (Catalina APIs, `javax.servlet` → `jakarta.servlet`), Cargo profile (tomcat9 → tomcat10x), embedded Tomcat overrides, Spring Boot 3 dependency |
| `jbpm` | 🟠 High | Cargo/Arquillian adapter for Tomcat 10, Narayana config files |
| `jbpm-work-items` | 🟠 High | `javax.servlet` → `jakarta.servlet` in Java source, embedded Tomcat overrides |
| `drools` | 🟡 Medium | Version via BOM only, no code changes expected |
| `kie-soup` | 🟡 Medium | Hardcoded `tomcat-dbcp` version `9.0.13` must be updated to `10.x` |

---

## Key Migration Constraints

1. **Spring Boot version**: Tomcat 10 embedded requires Spring Boot 3.x. Spring Boot 3.x requires Spring Framework 6.x and Java 17. Any module using `tomcat-embed-*` with Spring Boot 2.x **cannot** use Tomcat 10 without first upgrading Spring Boot.
2. **Narayana Tomcat integration**: The `narayana-tomcat` library must publish a Tomcat 10-compatible release. Verify `org.jboss.integration:narayana-tomcat` and `org.jboss.narayana.tomcat:narayana-tomcat` are available for Tomcat 10.
3. **Arquillian adapter**: There is no `arquillian-tomcat-remote-9` released (noted in existing `pom.xml` comments); check whether a Tomcat 10 adapter exists before updating container tests.
4. **`javax.security.auth.*` stays as-is**: These are JDK packages (JAAS), not Jakarta EE, and are not renamed in the migration.
5. **`org.apache.tomcat.dbcp.dbcp2` package**: The package path is unchanged between Tomcat 9 and 10 — no Java source refactoring needed for DBCP consumers.
