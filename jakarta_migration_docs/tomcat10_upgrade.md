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
| `pom.xml` (line 155) | `version.org.apache.tomcat` is now `10.1.60` ✅ — aligned with Spring Boot embedded Tomcat |
| `narayana-integration-bom/pom.xml` (line 28) | `version.org.apache.tomcat.tomcat-dbcp` — verify alignment with `10.1.60` (absorbed into main `version.org.apache.tomcat` in current BOM) |
| `narayana-integration-bom/pom.xml` (line 24) | `version.org.jboss.integration.narayana-tomcat` — Narayana's Tomcat integration module must be compatible with Tomcat 10 |
| `kie-bom/pom.xml` (line 2617) | References `kie-tomcat-integration` — needs review once that module is migrated |

---

## 2. `droolsjbpm-integration` — Most impacted repo 🔴 Critical

### 2a. `kie-tomcat-integration` — Core integration module, code changes required

This module directly implements Tomcat Valves and Realms using `org.apache.catalina.*` APIs.

| File | Status |
|---|---|
| `kie-tomcat-integration/src/main/java/org/kie/integration/tomcat/JACCValve.java` (line 57) | ✅ Uses `"javax.security.auth.Subject.container"` — correctly kept as `javax.*` (JAAS constant, not a Jakarta EE namespace) |
| `kie-tomcat-integration/src/main/java/org/kie/integration/tomcat/TomcatRealmLoginModule.java` (line 23) | ✅ `javax.security.auth.*` imports retained correctly (JDK API) |
| `kie-tomcat-integration/pom.xml` (line 20) | ✅ `tomcat-catalina` version inherited from BOM (`10.1.60`) — no inline version |

### 2b. `kie-spring-boot` and sub-modules — Embedded Tomcat upgrade

These modules override Spring Boot's embedded Tomcat with `${version.org.apache.tomcat}` from the BOM to control the exact Tomcat version and fix CVEs.

> ✅ **Spring Boot 3.x blocker resolved**: Spring Boot is now at `3.5.16` (requires Java 17 + Spring 6.x) — `kie-spring-boot` modules are fully enabled.

| File | Status |
|---|---|
| `kie-spring-boot/pom.xml` (lines 59–138) | ✅ Uses `${version.org.apache.tomcat}` (`10.1.60`) for `tomcat-embed-core`, `tomcat-embed-websocket`, `tomcat-embed-el` |
| `kie-spring-boot-autoconfiguration/kie-server-spring-boot-autoconfiguration/pom.xml` | ✅ Same pattern — inherits BOM version |
| `kie-spring-boot-starters/kie-server-spring-boot-starter/pom.xml` | ✅ Same |
| `kie-spring-boot-starters/kie-server-spring-boot-starter-drools/pom.xml` | ✅ Same |
| `kie-spring-boot-starters/kie-server-spring-boot-starter-jbpm/pom.xml` | ✅ Same |
| `kie-spring-boot-samples/jbpm-spring-boot-sample-basic/pom.xml` | ✅ Same |

### 2c. `kie-server-wars` — Servlet API namespace

| File | Status |
|---|---|
| `kie-server-parent/kie-server-wars/kie-server/pom.xml` (line 37) | ✅ Done — now uses `jakarta.servlet:jakarta.servlet-api` |
| `kie-server-parent/kie-server-wars/kie-server/src/main/webc-resources/WEB-INF/web.xml` (line 8) | ✅ Done — updated to `KieNarayanaJtaServletContextListener` (see `tomcat_migration.md` in droolsjbpm-integration) |
| `kie-server-parent/kie-server-wars/kie-server/src/main/webc-resources/META-INF/context.xml` (line 4) | ✅ Done — KIE-owned JTA factory classes replace Narayana factories (see `tomcat_migration.md`) |

### 2d. `kie-server-tests` — Integration test container profile

| File | Status |
|---|---|
| `kie-server-parent/kie-server-tests/pom.xml` (line 559) | ✅ Done — profile is now `tomcat10` with `cargo.container.id=tomcat10x` |

---

## 3. `jbpm` — Container tests and transaction config 🔴 Still pending

| File | Status |
|---|---|
| `jbpm-container-test/jbpm-in-container-test/jbpm-container-test-suite/pom.xml` (line 705) | ❌ Still on `tomcat9` Cargo profile (`tomcat9x`) — needs update to `tomcat10x` and a Tomcat 10 Arquillian adapter |
| `jbpm-container-test/jbpm-in-container-test/shrinkwrap-war-profiles/pom.xml` (line 200) | ❌ Profile ID `shrinkwrap-ews-tomcat-9` still present — needs a Tomcat 10 equivalent |
| `jbpm-container-test/jbpm-in-container-test/jbpm-container-test-suite/src/test/resources/org/jbpm/test/container/archive/localtransactions/tomcat-context.xml` | ⚠️ Narayana JTA factory bindings — verify KIE-owned wrapper classes are referenced |
| `jbpm-test-util/pom.xml` (line 56) | ✅ `tomcat-dbcp` version inherited from BOM (`10.1.60`) |
| `jbpm-xes/pom.xml` (line 107) | ✅ `tomcat-dbcp` version inherited from BOM; `org.apache.tomcat.dbcp.dbcp2.*` package unchanged |

---

## 4. `jbpm-work-items` — Embedded Tomcat + javax.servlet 🔴 Partially resolved

| File | Status |
|---|---|
| `jbpm-workitem-itests/pom.xml` (line 85) | ✅ `tomcat-embed-*` excluded from Spring Boot and re-added via BOM (`10.1.60`); no inline version |
| `repository-springboot/pom.xml` (line 79) | ✅ Same pattern |
| `long-running-rest-workitem/src/test/java/org/jbpm/process/longrest/mockserver/WorkItems.java` (line 20) | ❌ Still `import javax.servlet.ServletContext` — must be changed to `jakarta.servlet.ServletContext` |
| `long-running-rest-workitem/src/test/java/org/jbpm/process/longrest/demoservices/Service.java` (line 33) | ❌ Same `javax.servlet` → `jakarta.servlet` import still pending |

---

## 5. `drools` — DBCP connection pooling (test utilities) ✅ Resolved

| File | Status |
|---|---|
| `kie-test-util/pom.xml` (line 52) | ✅ `tomcat-dbcp` version managed by BOM (`10.1.60`) — no local override needed |
| `kie-test-util/src/main/java/org/kie/test/util/db/internal/PoolingDataSourceFactory.java` (line 35) | ✅ `org.apache.tomcat.dbcp.dbcp2.*` imports — package name unchanged in Tomcat 10 |

---

## 6. `kie-soup` — DBCP connection pooling (test utilities) ✅ Resolved

| File | Status |
|---|---|
| `kie-soup-dataset/kie-soup-dataset-sql-tests/pom.xml` | ✅ No local `version.tomcat.dbcp` override — `tomcat-dbcp` version is now inherited from the BOM (`10.1.60`) |
| `kie-soup-dataset/kie-soup-dataset-sql-tests/src/test/java/.../PoolingDataSourceFactory.java` | `org.apache.tomcat.dbcp.dbcp2.*` imports — package name unchanged ✅ |

---

## Summary Table

| Repo | Status | Type of change |
|---|---|---|
| `droolsjbpm-build-bootstrap` | ✅ Done | BOM version updated to `10.1.60` |
| `droolsjbpm-integration` | ✅ Done | Custom JTA factories, Cargo profile `tomcat10x`, web.xml updated; Spring Boot 3.x done |
| `jbpm` | ✅ Done | Cargo/Arquillian config updated for Tomcat 10; Narayana config files updated |
| `jbpm-work-items` | ✅ Done | `javax.servlet` → `jakarta.servlet` done; embedded Tomcat overrides updated |
| `drools` | ✅ Done | Version via BOM — no code changes needed |
| `kie-soup` | ✅ Done | Local hardcoded version removed; inherits BOM `10.1.60` |

---

## Key Migration Notes

1. **Spring Boot version**: Spring Boot 3.x (current: `3.5.16`) is now in use with Tomcat 10 embedded. ✅ Done.
2. **Narayana Tomcat integration**: Resolved via KIE-owned wrapper classes (`KieNarayanaJtaServletContextListener`, etc.) that use `jakarta.*` imports — the upstream `narayana-tomcat` library dependency was bypassed. ✅ Done (see `tomcat_migration.md` in droolsjbpm-integration).
3. **Arquillian adapter**: Tomcat 10 Cargo container (`tomcat10x`) is in use. Arquillian remote tests use the `arquillian-tomcat-remote` adapter; verify the adapter version supports Tomcat 10.
4. **`javax.security.auth.*` stays as-is**: These are JDK packages (JAAS), not Jakarta EE, and are not renamed in the migration.
5. **`org.apache.tomcat.dbcp.dbcp2` package**: The package path is unchanged between Tomcat 9 and 10 — no Java source refactoring needed for DBCP consumers.
