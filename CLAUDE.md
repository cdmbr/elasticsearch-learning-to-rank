# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Elasticsearch plugin (`esplugin` name: `ltr`) that adds learning-to-rank query types, a feature store, and feature-logging hooks. Plugin entrypoint is `com.o19s.es.ltr.LtrQueryParserPlugin` (`src/main/java/com/o19s/es/ltr/LtrQueryParserPlugin.java`), registered via the `elasticsearch.esplugin` Gradle plugin in `build.gradle`.

The plugin does NOT train models. It stores features (ES query templates) and pre-trained models (RankLib / linear / XGBoost) inside Elasticsearch, logs feature values for offline training, and executes models at query time. See `docs/fits-in.rst` for the boundary.

## Build & test commands

Java 21 is required (see `.github/workflows/test.yml`). Version coupling is strict: the produced artifact is `ltr-<ltrVersion>-es<elasticsearchVersion>.zip`, both pinned in `gradle.properties`.

| Task | Command |
|---|---|
| Full build + all checks (what CI runs) | `./gradlew clean check` |
| Compile only | `./gradlew assemble` |
| Unit tests (`src/test`) | `./gradlew test` |
| One unit test class | `./gradlew test --tests 'com.o19s.es.ltr.query.LtrQueryBuilderTests'` |
| One unit test method | `./gradlew test --tests 'com.o19s.es.ltr.query.LtrQueryBuilderTests.testSomething'` |
| Java REST integration tests (`src/javaRestTest`) — boots a real ES node with the plugin | `./gradlew javaRestTest` |
| YAML REST tests (`src/yamlRestTest`) | `./gradlew yamlRestTest` |
| Code-style check | `./gradlew spotlessCheck` |
| Auto-fix code style | `./gradlew spotlessApply` |
| Build plugin zip only | `./gradlew assemble` (output: `build/distributions/`) |
| Install locally | `./bin/elasticsearch-plugin install file:///…/build/distributions/ltr-<LTR-VER>-es<ES-VER>.zip` |

Spotless uses `googleJavaFormat('1.31.0')` and removes unused imports — wildcard imports will fail the build.

## Repository layout (only the non-obvious bits)

Four source sets sharing the same Gradle module:

- `src/main/java` — plugin code, all under `com.o19s.es.*`.
- `src/test/java` — pure unit tests using ES test framework.
- `src/javaRestTest/java` — integration tests; the `elasticsearch.java-rest-test` plugin spins up an ES cluster with the plugin installed. Tests here are named `*IT` and live alongside the package they exercise (e.g. `action/AddFeaturesToSetActionIT.java`).
- `src/yamlRestTest/java` + `src/yamlRestTest/resources/rest-api-spec` — YAML-driven REST tests dispatched by `LtrQueryClientYamlTestSuiteIT`. Add new YAML test files under `rest-api-spec/test/`, not new Java.

`publish/` is a separate Gradle subproject containing only Maven Central publication / GPG signing config — only relevant when cutting a release.

## Architecture: how a request flows

The plugin extends `Plugin` and implements `SearchPlugin`, `ScriptPlugin`, `ActionPlugin`, `AnalysisPlugin`. Wiring lives in `LtrQueryParserPlugin`:

1. **Query DSL extensions** (`getQueries()` in `LtrQueryParserPlugin`): registers `sltr` (`StoredLtrQueryBuilder`), `ltr` (`LtrQueryBuilder`), `match_explorer` (`ExplorerQueryBuilder`), `term_stat` (`TermStatQueryBuilder`), and an internal `validating_sltr` used during feature/model upload.
2. **Feature store** = a hidden ES index named `.ltrstore` (default) or `.ltrstore_<name>` for named stores. Constants in `IndexFeatureStore.DEFAULT_STORE` / `STORE_PREFIX`. Index mapping/analysis lives in `src/main/resources/com/o19s/es/ltr/feature/store/index/fstore-index-{mapping,analysis}.json`. Three element types stored: `featureset`, `feature`, `model` (subclasses of `StorableElement`).
3. **Caching**: `CachedFeatureStore` wraps `IndexFeatureStore` (constructed in `getFeatureStoreLoader()`). The shared `Caches` instance is created once at plugin construction and evicted on index deletion via a cluster listener. Cache size / TTL are settings: `ltr.caches.max_mem`, `ltr.caches.expire_after_read`, `ltr.caches.expire_after_write`.
4. **Model parsing**: `LtrRankerParserFactory` is built once at plugin construction with parsers for `model/ranklib`, `model/linear`, `model/xgboost+json`, `model/xgboost+json/v2`. RankLib's `RankerFactory` is heavy and is constructed lazily via `Suppliers.memoize`. All ranker parsers live under `com.o19s.es.ltr.ranker.parser`.
5. **Query execution**: `StoredLtrQueryBuilder` looks up a model+featureset from the store (cached), compiles features into a `RankerQuery` (`com.o19s.es.ltr.query.RankerQuery`) that scores documents using the ranker. `LtrRewritableQuery` / `LtrRewriteContext` carry features through Lucene rewrite.
6. **Feature logging** (for building training sets): `LoggingFetchSubPhase` + `LoggingSearchExtBuilder` (registered via `getFetchSubPhases()` / `getSearchExts()`) extract per-doc feature scores into the search response so they can be exported as training data.
7. **REST + transport actions**: REST handlers (`com.o19s.es.ltr.rest`) handle CRUD for features/featuresets/models, cache stats, and `_ltr/_stats`. Each REST handler dispatches to a transport action pair (`Action` + `TransportAction`) under `com.o19s.es.ltr.action`. The script engine `RankLibScriptEngine` (registered via `getScriptEngine()`) lets stored RankLib models also be invoked as scripts.

Side packages with no LTR-specific knowledge needed in most cases: `com.o19s.es.explore` (the `match_explorer` query computing IDF / term-count stats), `com.o19s.es.termstat` (the `term_stat` query), `com.o19s.es.template.mustache` (custom Mustache rendering for feature templates).

## Version branching

`master` tracks the latest ES upgrade. Maintenance branches are named per ES major.minor (`es_6_6`, `feat/9.4.x`, etc.) — when adding ES compatibility, bump `elasticsearchVersion` and `luceneVersion` in `gradle.properties`, branch off, and fix breakage. CONTRIBUTING.md has the upgrade playbook.

## Cutting a release (this fork — manual)

Releases on this fork are created by hand, not by `.github/workflows/release.yml` (which is inherited from upstream and unused here — no Maven Central secrets configured). The full step-by-step procedure lives in **CONTRIBUTING.md → [Cutting a release in this fork](CONTRIBUTING.md#cutting-a-release-in-this-fork)** — that is the source of truth for both Claude and human contributors.

Key facts to remember without re-reading:
- Tag format: `v<ltrVersion>-es<elasticsearchVersion>` (e.g. `v1.5.12-es9.4.1`).
- Gradle produces `ltr-<ltrVersion>-es<esVersion>.zip`; the release asset must be renamed to `ltr-plugin-<tag>.zip` (add `ltr-plugin-` prefix and leading `v`).
- Release title equals the tag name; release body is empty. Match this format.
- Do not push the tag and assume CI will publish — it won't. You must `gh release create` yourself.

## Known operational caveat

ES 5.5.3 ≤ version < 6.3.0 has a cache deadlock bug (see `KNOWN_ISSUES.md`) — irrelevant for current ES 9.x work but the `ltr.caches.*` settings exist partly because of it.
