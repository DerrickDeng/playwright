# Playwright Test Coverage Analysis

**Date**: 2026-02-22
**Scope**: Full codebase analysis of source-to-test ratios and gap identification

---

## Executive Summary

The Playwright repository has a comprehensive test suite with **526+ spec files** spanning page automation, the test runner, MCP tools, component testing, and more. However, several areas show significant coverage gaps — particularly around **BiDi protocol internals**, **dispatchers/RPC layer**, **utility functions (unit tests)**, **UI packages** (HTML reporter, trace viewer, recorder), and **newer features** like the AI agent framework. This document identifies the top areas where additional testing would reduce risk and improve confidence.

---

## 1. Codebase Overview

### Source Packages (734 source files total)

| Package | Source Files | Key Functionality |
|---------|-------------|-------------------|
| `playwright-core` | 304 | Browser automation engine (server, client, CLI) |
| `playwright` | 172 | Test runner, reporters, MCP, workers, matchers |
| `injected` | 22 | Scripts injected into pages (selectors, ARIA, clock) |
| `trace-viewer` | 43 | Trace visualization UI (React/TSX) |
| `html-reporter` | 26 | HTML report generation UI (React/TSX) |
| `web` | 24 | Shared web UI utilities |
| `playwright-ct-core` | 12 | Component testing core |
| `recorder` | 7 | Test recorder UI |
| `protocol` | 3 | RPC protocol definitions |
| `trace` | 3 | Trace format definitions |

### Test Suite (526+ spec files)

| Test Directory | Spec Files | Coverage Target |
|---------------|-----------|-----------------|
| `tests/page/` | 124 | Page API (locators, events, navigation, etc.) |
| `tests/library/` | 142 | Core library features, inspector, codegen |
| `tests/playwright-test/` | 102 | Test runner, reporters, fixtures, UI mode |
| `tests/mcp/` | 68 | MCP tools and CLI |
| `tests/components/` | 56 | Component testing (React, Vue, Svelte) |
| `tests/installation/` | 20 | Package installation and setup |
| `tests/android/` | 5 | Android automation |
| `tests/stress/` | 4 | Performance and stress testing |
| `tests/electron/` | 3 | Electron integration |
| `tests/image_tools/` | 2 | Image comparison utilities |
| `tests/bidi/` | 0 spec files | BiDi protocol (config only, no spec files) |

---

## 2. Coverage Gap Analysis

### CRITICAL: Areas with No or Very Weak Test Coverage

#### 2.1 BiDi (WebDriver BiDi) Protocol — 18 source files, 0 dedicated spec files

**Source**: `packages/playwright-core/src/server/bidi/` (18 files)
**Tests**: `tests/bidi/` contains only configuration files (`playwright.config.ts`, reporter utilities) but **zero `.spec.ts` files**.

Key untested modules:
- `bidiBrowser.ts` — BiDi browser lifecycle management
- `bidiPage.ts` — BiDi page implementation
- `bidiNetworkManager.ts` — BiDi network interception
- `bidiConnection.ts` — WebSocket connection handling
- `bidiSerializer.ts` / `bidiDeserializer.ts` — Data serialization
- `bidiExecutionContext.ts` — JavaScript execution via BiDi
- `bidiInput.ts` / `bidiKeyboard.ts` — Input handling
- `bidiPdf.ts` — PDF generation
- `bidiOverCdp.ts` — BiDi-over-CDP bridge

**Risk**: BiDi is the future of browser automation (replacing CDP for Firefox/cross-browser). Without dedicated tests, regressions in protocol handling, serialization edge cases, and connection lifecycle will go undetected.

**Recommendation**: Add spec files for at least BiDi connection lifecycle, serialization/deserialization round-trips, network interception, and input handling. Consider a dedicated `tests/bidi/` spec suite mirroring the page tests for BiDi-specific behavior.

---

#### 2.2 Dispatchers / RPC Layer — 23 source files, 0 dedicated tests

**Source**: `packages/playwright-core/src/server/dispatchers/` (23 files)
**Tests**: No test files specifically targeting the dispatcher layer.

Key untested modules:
- `dispatcher.ts` — Core dispatcher framework (message routing, object lifecycle)
- `pageDispatcher.ts` — Page object RPC handling
- `browserContextDispatcher.ts` — Context-level dispatching
- `networkDispatchers.ts` — Network request/response dispatching
- `frameDispatcher.ts` — Frame navigation dispatching
- `elementHandlerDispatcher.ts` — Element handle RPC
- `webSocketRouteDispatcher.ts` — WebSocket route handling
- `electronDispatcher.ts` — Electron-specific dispatching
- `androidDispatcher.ts` — Android-specific dispatching
- `pageAgentDispatcher.ts` — AI agent dispatching

**Risk**: The dispatcher layer is the core RPC backbone connecting the client API to the server. Bugs here can cause silent data corruption, connection leaks, or race conditions that are extremely hard to diagnose.

**Recommendation**: Add unit tests for the core `dispatcher.ts` framework (object creation, lifecycle, message serialization). Add integration tests for complex dispatchers like `networkDispatchers.ts` and `frameDispatcher.ts` focusing on edge cases (concurrent operations, error propagation, object disposal).

---

#### 2.3 Server Utilities — 30 source files, minimal unit tests

**Source**: `packages/playwright-core/src/server/utils/` (30 files)
**Tests**: Only 3 unit test files exist under `tests/library/unit/`

Key untested utilities:
- `happyEyeballs.ts` — Dual-stack TCP connection algorithm
- `socksProxy.ts` — SOCKS proxy implementation
- `wsServer.ts` — WebSocket server implementation
- `httpServer.ts` — HTTP server utilities
- `zones.ts` — Async context zones
- `crypto.ts` — Cryptographic utilities
- `comparators.ts` — Value comparison logic
- `imageChannel.ts` / `imageUtils.ts` — Image processing
- `colorUtils.ts` — Color manipulation
- `processLauncher.ts` — Process lifecycle management
- `env.ts` — Environment variable handling
- `fileUtils.ts` — File system operations

**Risk**: Utility functions are foundational building blocks. Bugs in Happy Eyeballs, SOCKS proxy, or crypto utilities could cause hard-to-diagnose network failures, security issues, or flaky tests.

**Recommendation**: Create a `tests/library/unit/` suite with focused unit tests for each utility module. Prioritize `happyEyeballs.ts`, `socksProxy.ts`, `comparators.ts`, `zones.ts`, and `crypto.ts`.

---

#### 2.4 Core Utils (`packages/playwright-core/src/utils/`) — 39 files, minimal dedicated tests

**Source**: `packages/playwright-core/src/utils/` (39 files including critical parsers)
**Tests**: Very few dedicated unit tests.

Key untested modules:
- `cssParser.ts` / `cssTokenizer.ts` — CSS selector parsing
- `selectorParser.ts` — Playwright selector syntax parsing
- `locatorParser.ts` / `locatorGenerators.ts` / `locatorUtils.ts` — Locator string parsing and generation
- `ariaSnapshot.ts` — ARIA snapshot comparison
- `urlMatch.ts` — URL pattern matching
- `yaml.ts` — YAML parsing
- `timeoutRunner.ts` — Timeout management
- `manualPromise.ts` — Manual promise resolution
- `multimap.ts` — Multimap data structure
- `lruCache.ts` — LRU cache implementation
- `snapshotRenderer.ts` / `snapshotStorage.ts` — Snapshot handling
- `traceLoader.ts` / `traceModel.ts` — Trace file loading and parsing (multiple version modules: `traceV3.ts` through `traceV8.ts`)

**Risk**: Parsers are notoriously error-prone with edge cases. The CSS parser, selector parser, and locator parser handle user-facing input and are critical to correct test execution. Trace loaders need to handle all historical trace versions correctly.

**Recommendation**: Add comprehensive unit tests for all parsers (CSS, selector, locator, ARIA, YAML). Test edge cases: empty inputs, malformed syntax, unicode, deeply nested structures. Add backward-compatibility tests for each trace version (V3-V8).

---

### HIGH: Areas with Partial or Thin Coverage

#### 2.5 AI Agent Framework — 8 source files, limited test variety

**Source**: `packages/playwright-core/src/server/agent/` (8 files)
**Tests**: Some JSON fixture-based tests exist in `tests/library/` but coverage is narrow.

Key source files:
- `pageAgent.ts` — Core agent orchestration
- `actionRunner.ts` — Action execution engine
- `performTools.ts` — Tool execution for agent
- `expectTools.ts` — Expectation/assertion tools
- `context.ts` — Agent context management
- `codegen.ts` — Code generation from agent actions

Also: `packages/playwright/src/agents/generateAgents.ts` (386 lines) and `agentParser.ts` (100 lines).

**Risk**: The agent framework is a newer feature that likely sees rapid iteration. Thin test coverage means regressions in tool execution, timeout handling, error recovery, and context management will go undetected.

**Recommendation**: Expand tests for agent error recovery, timeout cascading, concurrent agent operations, and edge cases in action execution (e.g., stale elements, navigation during action).

---

#### 2.6 HTML Reporter — 26 source files, 1 test file

**Source**: `packages/html-reporter/src/` (26 TSX/TS/CSS files)
**Tests**: `tests/playwright-test/reporter-html.spec.ts` (single file)

Untested UI components:
- `reportView.tsx` — Main report layout
- `testResultView.tsx` — Individual test result rendering
- `metadataView.tsx` — Metadata display
- `gantt.tsx` — Timeline visualization
- `filter.ts` — Filtering logic
- `headerView.tsx` — Navigation/header

**Risk**: The HTML reporter is a primary user-facing output. UI rendering bugs, filtering issues, or layout problems degrade user experience.

**Recommendation**: Add component-level tests for the filter logic (pure function, easily testable). Consider snapshot tests for key UI views. The `gantt.tsx` timeline visualization needs tests for time scale calculations and rendering edge cases (zero-duration tests, very long runs).

---

#### 2.7 Trace Viewer — 43 source files, 1 test file

**Source**: `packages/trace-viewer/src/` (43 TSX/TS/CSS files)
**Tests**: `tests/library/trace-viewer.spec.ts` (single file)

Untested UI components:
- Filter components
- Gantt chart visualization
- Test tree navigation
- Network tab
- Metadata/attachment views
- Service worker logic (`sw/`)

**Risk**: Similar to the HTML reporter — the trace viewer is a critical debugging tool. Bugs in trace playback, timeline rendering, or filtering directly impact developer productivity.

**Recommendation**: Add tests for trace file loading with various versions, timeline rendering, and filter logic. The service worker (`sw/`) directory needs specific tests for caching and trace file serving.

---

#### 2.8 Recorder UI — 7 source files, indirect coverage only

**Source**: `packages/recorder/src/` (7 files including `recorder.tsx` at 1,959 lines)
**Tests**: Tested indirectly through `tests/library/inspector/cli-codegen-*.spec.ts` and `recorder-api.spec.ts`, but no direct component tests.

**Risk**: The recorder UI is a complex interactive component. Code generation in the recorder interacts with many edge cases (iframes, shadow DOM, dynamic content).

**Recommendation**: Add tests specifically for the recorder UI state management and code generation output for edge cases: shadow DOM, iframes, web components, dynamic selectors.

---

#### 2.9 Injected Scripts — Large files with indirect-only coverage

**Source**: `packages/injected/src/` (22 files, several very large)

| File | Lines | Direct Tests? |
|------|-------|---------------|
| `recorder/recorder.ts` | 1,959 | No |
| `injectedScript.ts` | 1,813 | No |
| `roleUtils.ts` | 1,235 | No |
| `clock.ts` | 792 | Partial (clock.spec.ts) |
| `ariaSnapshot.ts` | 747 | Partial |
| `selectorEvaluator.ts` | 602 | No |
| `selectorGenerator.ts` | 581 | No |
| `webSocketMock.ts` | 363 | No |

**Risk**: These scripts run inside the browser and are the foundation of element selection, ARIA snapshot generation, and recording. `roleUtils.ts` (1,235 lines) implements complex ARIA role computation. `selectorGenerator.ts` generates the locators users see. Bugs here directly cause incorrect test results.

**Recommendation**: Add unit tests for `roleUtils.ts` ARIA role computation (test with complex ARIA patterns, dynamic roles, implicit vs explicit roles). Add tests for `selectorGenerator.ts` covering all selector strategies. Add tests for `selectorEvaluator.ts` with complex selector expressions.

---

#### 2.10 Stress and Performance Testing — 4 spec files

**Source**: `tests/stress/` (4 spec files, 233 total lines)
- `heap.spec.ts` (118 lines)
- `frames.spec.ts` (60 lines)
- `contexts.spec.ts` (24 lines)
- `browsers.spec.ts` (31 lines)

**Risk**: Memory leaks and performance degradation are common in long-running automation. The stress tests are minimal and don't cover:
- Memory leaks with large numbers of pages/tabs
- Network interception memory overhead
- Trace recording memory growth
- Worker pool exhaustion in the test runner
- Large test suite scalability

**Recommendation**: Expand stress tests to cover network interception with many routes, trace recording over long sessions, and test runner scalability with 1000+ tests. Add heap snapshot analysis for common automation patterns.

---

### MODERATE: Areas Worth Improving

#### 2.11 Test Runner Internals

Some runner modules have thin coverage:
- `watchMode.ts` — Only 1 test file (`watch.spec.ts`)
- `sigIntWatcher.ts` — Signal handling (hard to test but important)
- `failureTracker.ts` — Failure tracking logic
- `testGroups.ts` — Test grouping/batching
- `projectUtils.ts` — Multi-project configuration handling
- `vcs.ts` — Version control integration

**Recommendation**: Add focused tests for watch mode edge cases (file rename detection, config change handling). Add tests for test grouping with complex project dependency graphs.

---

#### 2.12 Reporter Edge Cases

While most reporters have test files, several source modules lack dedicated coverage:
- `teleEmitter.ts` — Telemetry event emission
- `multiplexer.ts` — Multi-reporter coordination
- `internalReporter.ts` — Internal reporting framework
- `listModeReporter.ts` — List mode output
- `blobV1.ts` — Legacy blob format

**Recommendation**: Add tests for the multiplexer (concurrent reporters, error in one reporter not blocking others). Add backward-compatibility tests for `blobV1.ts`.

---

#### 2.13 Electron Integration — 2 source files, 3 test files

**Source**: `packages/playwright-core/src/server/electron/` (2 files)
**Tests**: `tests/electron/` (3 spec files)

Electron testing is inherently complex and the current tests appear thin. Consider adding tests for:
- Electron app lifecycle (launch, crash, restart)
- IPC communication mocking
- DevTools protocol integration
- Multi-window Electron scenarios

---

#### 2.14 Android Automation — 2 source files, 5 test files

**Source**: `packages/playwright-core/src/server/android/` (2 files)
**Tests**: `tests/android/` (5 spec files)

The ratio is decent but given the complexity of Android automation (device discovery, WebView integration, screen capture), more edge case testing would be valuable.

---

#### 2.15 Component Testing Frameworks — Uneven Coverage

While `tests/components/` has 56 spec files, the coverage is split across 6 framework variants. Verify that each framework gets equal attention:
- React (Vite) — likely best covered
- React 17 — legacy compatibility
- Vue (CLI and Vite) — two build targets
- Svelte (basic and Vite) — two build targets

**Recommendation**: Ensure parity tests exist across all framework variants, especially for mounting, props, events, and slot/children handling.

---

## 3. Prioritized Recommendations

### Tier 1 — Critical (High risk, foundational code)

| # | Area | Action | Estimated Effort |
|---|------|--------|-----------------|
| 1 | **BiDi Protocol** | Create `tests/bidi/*.spec.ts` covering connection, serialization, page lifecycle, network, input | Large |
| 2 | **Dispatchers/RPC** | Add unit tests for core dispatcher framework + integration tests for complex dispatchers | Large |
| 3 | **Parser Unit Tests** | Unit tests for CSS parser, selector parser, locator parser, ARIA snapshot parser | Medium |
| 4 | **Server Utilities** | Unit tests for happyEyeballs, socksProxy, comparators, zones, crypto | Medium |

### Tier 2 — High (User-facing features, newer code)

| # | Area | Action | Estimated Effort |
|---|------|--------|-----------------|
| 5 | **AI Agent Framework** | Expand agent tests: error recovery, timeouts, concurrent ops, stale elements | Medium |
| 6 | **Injected Scripts** | Unit tests for roleUtils (ARIA), selectorGenerator, selectorEvaluator | Medium |
| 7 | **HTML Reporter** | Component tests for filter logic, gantt rendering, edge cases | Small |
| 8 | **Trace Viewer** | Tests for trace loading (all versions), timeline, filtering | Medium |

### Tier 3 — Moderate (Improve robustness)

| # | Area | Action | Estimated Effort |
|---|------|--------|-----------------|
| 9 | **Stress Tests** | Expand to cover memory leaks, network interception overhead, large suites | Medium |
| 10 | **Watch Mode** | Edge cases: file renames, config changes, error recovery | Small |
| 11 | **Reporter Multiplexer** | Concurrent reporter tests, error isolation | Small |
| 12 | **Trace Version Compat** | Backward-compat tests for trace V3-V8 formats | Small |
| 13 | **Recorder UI** | Edge cases: shadow DOM, iframes, web components | Medium |

---

## 4. Key Metrics Summary

| Metric | Value |
|--------|-------|
| Total source files | ~734 |
| Total test spec files | ~526 |
| Source-to-test ratio | 1.4:1 |
| Modules with zero dedicated tests | BiDi (18 files), Dispatchers (23 files) |
| Largest untested files | `injectedScript.ts` (1,813 lines), `recorder.ts` (1,959 lines), `roleUtils.ts` (1,235 lines) |
| Unit test files | 3 (under `tests/library/unit/`) |
| Test directories | 14 |

---

## 5. Methodology

This analysis was conducted by:
1. Mapping all source packages and their file counts
2. Mapping all test directories and their spec file counts
3. Cross-referencing source modules against test files by name and import patterns
4. Identifying source modules with no corresponding test files
5. Evaluating coverage breadth by comparing source complexity (line count) to test variety

Note: This is a structural analysis based on file presence and naming conventions, not runtime code coverage data. Actual line/branch coverage may differ from these findings.
