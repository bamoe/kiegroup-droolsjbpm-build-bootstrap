# EAP 8.1 / WildFly 35 — Impacted Repositories

> Source: `eap_migration.md`. Only items with **❌ / ⚠️** status are listed; ✅-aligned items are omitted.

---

## 1. Version Upgrades — `droolsjbpm-build-bootstrap` (BOM)

All version properties below live in [`droolsjbpm-build-bootstrap/pom.xml`](../pom.xml) and propagate to every downstream repo via BOM import.

| Property | Current | Required | Risk |
|---|---|---|---|
| `version.org.infinispan` | ~~14.0.27.Final~~ → **15.0.20.Final** | **15.0.x.Final** | ✅ Done |
| `version.io.undertow` | ~~2.2.38.Final~~ → **2.3.24.Final** | **2.3.x.Final** | ✅ Done |
| `version.org.keycloak` | ~~20.0.3~~ → **26.6.4** | **24.0.x** | ✅ Done (upgraded past 24.x) |
| `version.jakarta.persistence-api` | **3.1.0** | **3.2.0** | ⚠️ Still at 3.1.0 — consider upgrading to align with EAP 8.1 |
| `version.jakarta.enterprise.cdi-api` | ~~4.0.1~~ → **4.1.0** | **4.1.0** | ✅ Done |
| `version.jakarta.websocket-api` | ~~2.1.1~~ → **2.2.0** | **2.2.0** | ✅ Done |
| `version.jakarta.enterprise.concurrent-api` | ~~3.0.4~~ → **3.1.1** | **3.1.0** | ✅ Done |
| `version.jakarta.authorization.api` | ~~absent~~ → **3.0.0** | **3.0.0** (`jakarta.authorization-api`) | ✅ Done (property `version.jakarta.authorization.api`) |
| `version.wildfly.jar.maven.plugin` (bootable) | ~~6.1.2.Final~~ → **13.0.0.Final** | **13.0.x.Final** | ✅ Done |
| `wildfly-maven-plugin` (pluginManagement) | ~~1.2.0.Final~~ → **5.1.5.Final** | **5.0.x.Final** | ✅ Done |

---

## 2. Spring Upgrade — `droolsjbpm-integration` + `droolsjbpm-knowledge` + `jbpm-work-items`

Spring Boot 3.x with `jakarta.*` namespaces is now in use. All three upgrades are complete.

| Property | Previous | Current | Status |
|---|---|---|---|
| `version.org.springframework.boot` | 2.7.18 | **3.5.16** | ✅ Done |
| `version.org.springframework` | 5.3.39 | **6.2.19** | ✅ Done |
| `version.org.springframework.security.web` | 5.7.14 | **6.5.11** | ✅ Done |

**Affected modules** (own Spring Boot usage):

| Repository | Module path |
|---|---|
| `droolsjbpm-integration` | `kie-spring-boot/**` (starters, autoconfiguration, samples) |
| `droolsjbpm-knowledge` | `kie-archetypes/kie-service-spring-boot-archetype` |
| `jbpm-work-items` | `repository-springboot`, `jbpm-workitem-itests` |

**Breaking changes at code level:**

| Area | Spring 5→6 / Boot 2→3 breaking change |
|---|---|
| Namespace | All `javax.*` → `jakarta.*` in auto-config classes |
| Security | `WebSecurityConfigurerAdapter` removed → `SecurityFilterChain` bean |
| Data/JPA | `JpaRepository` bootstrap changes; Hibernate 6 column naming strategy |
| Web MVC | `HttpMethod` is now an interface; `MockMvc` import changed |
| Actuator | Many endpoint paths renamed under `/actuator` |
| Properties | Several `spring.*` config key names changed |
| Embedded Tomcat | Must be ≥ 10.1.x for `jakarta.*` servlet support |

---

## 3. Process Migration Service — `process-migration-service`

| File | Change | Detail |
|---|---|---|
| `process-migration-service/pom.xml` | `version.org.wildfly` | ✅ Done — now `35.0.1.Final` |
| `process-migration-service/pom.xml` | Cargo container ID | ✅ Done — now `wildfly35x` |
| `process-migration-service/pom.xml` | Arquillian container | ✅ Done — `wildfly-arquillian-container` at `5.1.0.Final` |

**Breaking changes:** WildFly 35 subsystem XML schemas differ; any `standalone.xml` / `domain.xml` snippets in integration tests must be reviewed.

---

## 4. WildFly Camel — `droolsjbpm-integration`

| Module | Risk |
|---|---|
| `droolsjbpm-integration/kie-camel` | 🔴 **High** — `org.wildfly.camel` subsystem dropped after WF 26; entire module unusable on EAP 8.1 |
| `droolsjbpm-integration/camel-container-tests` | 🔴 Same — camel container tests rely on the removed subsystem |

**Action:** Remove or replace `kie-camel` module; Camel must be embedded (Camel Quarkus / Spring Camel) rather than provided by the server subsystem.

---

## 5. Errai — `kie-wb-common`, `appformer`, `drools-wb`, `jbpm-wb`, `kie-wb-distributions`

| Risk | Detail |
|---|---|
| 🔴 **High** | `org.jboss.errai` is now at `5.0.0-SNAPSHOT` (Jakarta EE 10 / CDI 4.x target), but remains a SNAPSHOT |
| Code-level | `@Inject`, `@Portable`, `@Dependent` Errai equivalents — ongoing compatibility work with CDI 4.1 |
| GWT integration | Errai's GWT-CDI bridge stability under Errai 5.x / CDI 4.1 should be verified through build/test runs |

**Affected repos** (all repos importing `jboss.errai`): `kie-wb-common`, `appformer`, `drools-wb`, `jbpm-wb`, `kie-wb-distributions`, `kie-uberfire-extensions`.

---

## 6. PicketLink — `appformer`, `drools-wb`, `kie-wb-common`

| File | Change required |
|---|---|
| `jboss-deployment-structure.xml` (in WARs) | Remove `org.picketlink` module dependency |
| Security config classes | Replace PicketLink identity/auth with **Elytron** security domain |
| `beans.xml` | Update CDI schema to version `4.0` |

**Affected repos:** `appformer` (`uberfire-showcase`), `drools-wb` (`drools-wb-webapp`), `kie-wb-common`.

**Breaking:** PicketLink `SecurityContext`, `Identity`, `Credentials` APIs removed entirely; full migration to Elytron / `jakarta.security.enterprise` required.

---

## 7. Keycloak Adapter — `appformer`, `kie-wb-distributions`, `droolsjbpm-integration`

| Risk | Detail |
|---|---|
| ⚠️ Verify | Keycloak upgraded to `26.6.4`; legacy WildFly adapter was removed from Keycloak 24+ |
| Required action | Verify migration to `keycloak-servlet-oauth-client` or native OIDC in Elytron is complete |
| `jboss-deployment-structure.xml` | Verify `org.keycloak.keycloak-adapter-subsystem` dependency has been removed |

---

## 8. Deployment Descriptor Changes (all WAR-producing repos)

Applies to `kie-wb-distributions`, `droolsjbpm-integration`, `jbpm-wb`, `kie-wb-common` and any module producing a deployable WAR/EAR.

| Descriptor | Required change |
|---|---|
| `jboss-deployment-structure.xml` | Remove `org.picketlink`, remove `org.jboss.as.camel` |
| `persistence.xml` | Namespace → `https://jakarta.ee/xml/ns/persistence`, version `3.2` |
| `beans.xml` | Namespace → `https://jakarta.ee/xml/ns/cdi`, version `4.0` |
| Security subsystem config | Replace PicketLink / legacy security with Elytron security domain |

---

## Summary Matrix

> **Status updated**: most POM changes are now complete. Remaining open items are noted below.

| Repository | POM change status | Code-level breaking change | Severity |
|---|---|---|---|
| `droolsjbpm-build-bootstrap` | ✅ Done — all version bumps applied (⚠️ `jakarta.persistence-api` still at 3.1.0) | None (BOM only) | Low |
| `droolsjbpm-integration` | ✅ Spring Boot 3.x done; Camel removal done | Namespace done; Security API done; Camel subsystem removed | ✅ Addressed |
| `droolsjbpm-knowledge` | ✅ Spring Boot 3.x done | Archetype template `javax.*` → `jakarta.*` done | ✅ Addressed |
| `jbpm-work-items` | ✅ Spring Boot 3.x done | Security/Web MVC API changes done | ✅ Addressed |
| `process-migration-service` | ✅ WildFly 35 + Cargo container ID done | Integration test config done | ✅ Done |
| `kie-wb-common` | None (inherits BOM) | PicketLink removal; Errai CDI 4.1 compat | 🔴 High (ongoing) |
| `appformer` | None (inherits BOM) | PicketLink removal; Errai CDI 4.1 compat | 🔴 High (ongoing) |
| `drools-wb` | None (inherits BOM) | PicketLink removal; Errai CDI 4.1 compat | 🔴 High (ongoing) |
| `jbpm-wb` | None (inherits BOM) | Errai CDI 4.1 compat | High (ongoing) |
| `kie-wb-distributions` | None (inherits BOM) | Keycloak adapter removal; descriptors | High (ongoing) |
| `jbpm` | None (inherits BOM) | Keycloak adapter; `persistence.xml` v3.1 | Medium (ongoing) |
