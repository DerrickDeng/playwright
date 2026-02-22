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

#### 2.6 MCP (Model Context Protocol) — 60 source files, 65 test files, but 15 tools untested

**Source**: `packages/playwright/src/mcp/` (60 source files, ~220 KB)
**Tests**: `tests/mcp/` (65 spec files, ~266 KB)

MCP has good overall coverage (77.3% of browser tools tested), but 15 specific tools have **zero test coverage**:

**Untested browser tools (high priority):**
- `browser_check` / `browser_uncheck` — Checkbox interaction
- `browser_navigate_forward` / `browser_reload` — Navigation actions
- `browser_keydown` / `browser_keyup` — Low-level keyboard events
- `browser_mouse_down` / `browser_mouse_up` / `browser_mouse_wheel` — Low-level mouse events
- `browser_console_clear` — Console management

**Untested browser tools (medium priority):**
- `browser_network_clear` — Network log management
- `browser_devtools_start` / `browser_devtools_stop` — DevTools server control
- `browser_start_video` / `browser_stop_video` — Video recording

**Minimally tested tools (1-2 references only):**
- `browser_press_sequentially`, `browser_generate_locator`, `browser_navigate_back`, `browser_hover`, `browser_drag`, `browser_get_config`

**Well covered:** Test tools are 100% tested (9/9). Navigation, form interaction, cookies, storage, and verification are well-tested. CLI terminal tests provide good secondary coverage via 26+ `cli-*.spec.ts` files.

**Untested source modules:**
- `packages/playwright/src/mcp/extension/cdpRelay.ts` (15K) — CDP relay for extensions, limited coverage
- `packages/playwright/src/mcp/browser/watchdog.ts` — Session watchdog, no dedicated tests
- `packages/playwright/src/mcp/browser/sessionLog.ts` — Session logging, minimal coverage

**Recommendation**: Add tests for the 15 untested browser tools (especially checkbox, navigation forward/reload, and video recording). Add tests for the extension CDP relay. Add dedicated keyboard and mouse low-level event tests.

---

#### 2.7 HTML Reporter — 26 source files, 1 test file

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

#### 2.8 Trace Viewer — 43 source files, 1 test file

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

#### 2.9 Recorder UI — 7 source files, indirect coverage only

**Source**: `packages/recorder/src/` (7 files including `recorder.tsx` at 1,959 lines)
**Tests**: Tested indirectly through `tests/library/inspector/cli-codegen-*.spec.ts` and `recorder-api.spec.ts`, but no direct component tests.

**Risk**: The recorder UI is a complex interactive component. Code generation in the recorder interacts with many edge cases (iframes, shadow DOM, dynamic content).

**Recommendation**: Add tests specifically for the recorder UI state management and code generation output for edge cases: shadow DOM, iframes, web components, dynamic selectors.

---

#### 2.10 Injected Scripts — Large files with mixed coverage

**Source**: `packages/injected/src/` (22 files, 7,719 LOC total, several very large)

| File | Lines | Direct Tests? | Notes |
|------|-------|---------------|-------|
| `recorder/recorder.ts` | 1,959 | No | Largest file, only indirect coverage |
| `injectedScript.ts` | 1,813 | No | Core evaluation logic, only indirect |
| `roleUtils.ts` | 1,235 | **Yes** | `role-utils.spec.ts` (24.9K lines) |
| `clock.ts` | 792 | **Yes** | `clock.spec.ts` (unit test) |
| `ariaSnapshot.ts` | 747 | **Yes** | `page-aria-snapshot.spec.ts` + 13 test files |
| `selectorEvaluator.ts` | 602 | No | No explicit test coverage |
| `selectorGenerator.ts` | 581 | **Yes** | `selector-generator.spec.ts` (26.7K lines) |
| `webSocketMock.ts` | 363 | No | No dedicated tests |
| `highlight.ts` | 288 | Partial | Only `locator-highlight.spec.ts` |
| `storageScript.ts` | 210 | No | No dedicated tests |
| `roleSelectorEngine.ts` | 198 | No | No dedicated tests |
| `domUtils.ts` | 175 | No | No dedicated tests |
| `consoleApi.ts` | 147 | No | Only 1 reference |

**Risk**: These scripts run inside the browser and are the foundation of element selection, ARIA snapshot generation, and recording. While `roleUtils.ts` and `selectorGenerator.ts` have good dedicated tests, the two largest files — `injectedScript.ts` (1,813 lines of core evaluation logic) and `recorder/recorder.ts` (1,959 lines) — have zero direct tests. `selectorEvaluator.ts` (602 lines) also lacks explicit coverage.

**Recommendation**: Add unit tests for `injectedScript.ts` core evaluation paths. Add tests for `selectorEvaluator.ts` with complex selector expressions. Add tests for `webSocketMock.ts`, `domUtils.ts`, and `storageScript.ts`.

---

#### 2.11 Stress and Performance Testing — 4 spec files

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

#### 2.12 Test Runner Internals — 20 files, 4,494 LOC, many with zero test references

The runner module has extensive code with many files having **zero direct test references**:

**Zero test references (12 files):**
- `failureTracker.ts` (89 lines) — Failure tracking logic
- `projectUtils.ts` (250 lines) — Complex project filtering/matching
- `testGroups.ts` (173 lines) — Test grouping/batching
- `taskRunner.ts` (146 lines) — Task execution framework
- `workerHost.ts` (116 lines) — Worker process management
- `processHost.ts` (186 lines) — Process lifecycle
- `loaderHost.ts` (84 lines) — Test file loading
- `sigIntWatcher.ts` (105 lines) — Signal handling
- `lastRun.ts` (77 lines) — Last run persistence
- `storage.ts` (80 lines) — Storage utilities
- `vcs.ts` (58 lines) — Version control integration
- `rebase.ts` (205 lines) — Snapshot rebasing logic

**Thin coverage:**
- `loadUtils.ts` (382 lines) — Core test discovery, only 1 test reference
- `testServer.ts` (342 lines) — Limited coverage

**Well covered:**
- `watchMode.ts` (433 lines) — `watch.spec.ts` (31.5K lines)
- `dispatcher.ts` (670 lines) — 115 test references via integration tests
- `testRunner.ts` (496 lines) — `runner.spec.ts` (30.7K lines)

**Recommendation**: Add focused unit tests for `projectUtils.ts` (project dependency graphs, filtering), `testGroups.ts` (batching logic), `loadUtils.ts` (test discovery edge cases), and `rebase.ts` (snapshot rebasing). Add tests for `failureTracker.ts` state management.

---

#### 2.13 Reporter Edge Cases — 17 files, 4,562 LOC

While most reporters have dedicated test files, several source modules lack coverage:

**Well tested (10 reporters with dedicated spec files):**
- `base.ts`, `html.ts`, `json.ts`, `junit.ts`, `list.ts`, `blob.ts`, `dot.ts`, `line.ts`, `github.ts`, `markdown.ts`

**No dedicated coverage (7 files):**
- `merge.ts` (681 lines) — Complex report merging logic, only indirect coverage through `reporter-blob.spec.ts`
- `teleEmitter.ts` (352 lines) — Event serialization, zero direct test references
- `reporterV2.ts` (148 lines) — Reporter interface adapter, no tests
- `internalReporter.ts` (149 lines) — Internal framework, no tests
- `multiplexer.ts` (131 lines) — Multi-reporter coordination, no tests
- `listModeReporter.ts` (70 lines) — List mode output, minimal references
- `blobV1.ts` — Legacy blob format, no backward-compat tests

**Recommendation**: Add dedicated tests for `merge.ts` (report merging edge cases: conflicting data, partial reports, format versions). Add tests for the `multiplexer.ts` (concurrent reporters, error isolation). Add backward-compatibility tests for `blobV1.ts`.

---

#### 2.14 Common Utilities (`packages/playwright/src/common/`) — 14 files, 2,673 LOC

**Zero direct test references (8 files):**
- `validators.ts` (70 lines) — Zod schema validation, untested in isolation
- `ipc.ts` (197 lines) — Inter-process communication logic
- `process.ts` (154 lines) — Process management utilities
- `suiteUtils.ts` (139 lines) — Test suite utilities
- `testLoader.ts` (100 lines) — Module/file loading
- `poolBuilder.ts` (93 lines) — Worker pool construction
- `esmLoaderHost.ts` (93 lines) — ESM loader hosting
- `globals.ts` (44 lines) — Global state management

**Well covered through integration tests:**
- `config.ts` (317 lines) — via `config.spec.ts` (23K lines)
- `fixtures.ts` (361 lines) — via `fixtures.spec.ts` (27K lines)
- `configLoader.ts` (376 lines) — via config/loader tests

**Recommendation**: Add unit tests for `validators.ts` (schema edge cases), `ipc.ts` (message serialization), and `poolBuilder.ts` (pool sizing logic).

---

#### 2.15 Matchers/Assertions — Well Covered

The matchers module (9 files, 2,076 LOC in `packages/playwright/src/matchers/`) is one of the **best-tested** areas, with comprehensive coverage through `expect.spec.ts` (45.6K lines), `to-have-screenshot.spec.ts` (63.6K lines), and dedicated ARIA snapshot tests. No significant gaps identified.

---

#### 2.16 Electron Integration — 2 source files, 3 test files

**Source**: `packages/playwright-core/src/server/electron/` (2 files)
**Tests**: `tests/electron/` (3 spec files)

Electron testing is inherently complex and the current tests appear thin. Consider adding tests for:
- Electron app lifecycle (launch, crash, restart)
- IPC communication mocking
- DevTools protocol integration
- Multi-window Electron scenarios

---

#### 2.17 Android Automation — 2 source files, 5 test files

**Source**: `packages/playwright-core/src/server/android/` (2 files)
**Tests**: `tests/android/` (5 spec files)

The ratio is decent but given the complexity of Android automation (device discovery, WebView integration, screen capture), more edge case testing would be valuable.

---

#### 2.18 Component Testing Frameworks — Uneven Coverage

While `tests/components/` has 56 spec files, the coverage is split across 6 framework variants. Verify that each framework gets equal attention:
- React (Vite) — likely best covered
- React 17 — legacy compatibility
- Vue (CLI and Vite) — two build targets
- Svelte (basic and Vite) — two build targets

**Recommendation**: Ensure parity tests exist across all framework variants, especially for mounting, props, events, and slot/children handling.

---

#### 2.19 CLI Commands — Several commands lack dedicated tests

**Source**: `packages/playwright-core/src/cli/program.ts` (744 lines, 14 commands)

**Well covered**: `codegen`, `install`, `open`, `cr`/`ff`/`wk` (via codegen and MCP tests)

**No dedicated CLI tests:**
- `show-trace` — No tests for trace viewer launch via CLI
- `run-driver` / `run-server` / `launch-server` — Hidden server commands, untested in isolation
- `uninstall` — No specific CLI tests for browser uninstallation
- `mark-docker-image` — Hidden Docker command, untested
- `pdf` / `screenshot` — Partial coverage via library tests, no dedicated CLI tests

**Recommendation**: Add CLI-level tests for `show-trace`, `uninstall`, and `screenshot`/`pdf` commands with error cases (invalid URLs, missing permissions, disk full).

---

#### 2.20 Git Commit Info Plugin — 197 lines, zero dedicated tests

**Source**: `packages/playwright/src/plugins/gitCommitInfoPlugin.ts` (197 lines)
**Tests**: No dedicated tests found.

This plugin captures git metadata (commit hash, branch, author) for test reports. It has no test coverage despite interacting with the git CLI and report metadata.

**Recommendation**: Add tests for git metadata capture (normal repos, detached HEAD, shallow clones, missing git), and verify metadata appears correctly in reports.

---

### CROSS-CUTTING: Error Handling and Edge Cases

#### 2.21 Error Handling — Significant gaps across the codebase

While Playwright has good coverage for common error scenarios (target closed, timeouts, page errors, HTTPS certificate errors), several critical error paths lack testing:

**Critical error handling gaps:**
- **PipeTransport JSON.parse()** (`pipeTransport.ts` lines 76-88) — No error handling for malformed JSON messages; could crash the entire transport. The WebSocket transport (`transport.ts`) handles this correctly, but `PipeTransport` does not.
- **Protocol validator unit tests** — `ValidationError` is thrown but never tested. Zero dedicated tests for the validation framework despite complex type checking logic.
- **Serialization edge cases** — Only 2 tests exist (circular objects, BigInt). Missing: invalid ref IDs, circular reference detection, null stack traces, custom Error subclasses, handle validation failures.
- **Remote connection** — `playwrightServer.ts` line 99 has an empty catch block for JSON.parse. WebSocket message callback exceptions are swallowed.

**Network-level error gaps:**
- DNS resolution failures (ENOTFOUND, ECONNREFUSED) — not tested
- Connection reset errors (ECONNRESET) — not tested
- Happy Eyeballs IPv4/IPv6 fallback edge cases — not tested
- Proxy connection failures — not tested

**Process-level error gaps:**
- Missing executable (ENOENT) — not tested
- Permission denied (EACCES) — not tested
- Spawn timeout and signal handling — not tested

**Well covered error scenarios:**
- Page error events (116 test cases)
- Target closed errors (402+ test references)
- Browser crash handling (crash event, navigation cancellation, context closure)
- WebSocket connection errors and reconnection (52 tests)
- Network timeouts and fetch abort scenarios
- Corrupted response bodies (gzip, brotli, deflate)

**Recommendation**: Fix the `PipeTransport` JSON.parse vulnerability (add try-catch). Add unit tests for protocol validators and serialization edge cases. Add network-level error tests for DNS failures and connection resets.

---

## 3. Prioritized Recommendations

### Tier 1 — Critical (High risk, foundational code)

| # | Area | Action | Estimated Effort |
|---|------|--------|-----------------|
| 1 | **BiDi Protocol** | Create `tests/bidi/*.spec.ts` covering connection, serialization, page lifecycle, network, input | Large |
| 2 | **Dispatchers/RPC** | Add unit tests for core dispatcher framework + integration tests for complex dispatchers | Large |
| 3 | **Parser Unit Tests** | Unit tests for CSS parser, selector parser, locator parser, ARIA snapshot parser | Medium |
| 4 | **Server Utilities** | Unit tests for happyEyeballs, socksProxy, comparators, zones, crypto | Medium |
| 5 | **Error Handling** | Fix PipeTransport JSON.parse gap; add protocol validator + serialization edge case tests | Medium |

### Tier 2 — High (User-facing features, newer code)

| # | Area | Action | Estimated Effort |
|---|------|--------|-----------------|
| 6 | **AI Agent Framework** | Expand agent tests: error recovery, timeouts, concurrent ops, stale elements | Medium |
| 7 | **Runner Internals** | Unit tests for projectUtils, testGroups, loadUtils, failureTracker, rebase (12+ files with zero refs) | Large |
| 8 | **Injected Scripts** | Unit tests for injectedScript.ts (1,813 LOC), selectorEvaluator.ts (602 LOC) | Medium |
| 9 | **HTML Reporter** | Component tests for filter logic, gantt rendering, edge cases | Small |
| 10 | **Trace Viewer** | Tests for trace loading (all versions), timeline, filtering | Medium |

### Tier 3 — Moderate (Improve robustness)

| # | Area | Action | Estimated Effort |
|---|------|--------|-----------------|
| 11 | **MCP Untested Tools** | Tests for 15 untested browser tools (checkbox, forward/reload, low-level events, video) | Medium |
| 12 | **Reporter merge.ts** | Dedicated tests for report merging (681 LOC), teleEmitter, multiplexer | Medium |
| 13 | **Common Utilities** | Unit tests for validators.ts, ipc.ts, poolBuilder.ts | Small |
| 14 | **Stress Tests** | Expand to cover memory leaks, network interception overhead, large suites | Medium |
| 15 | **Trace Version Compat** | Backward-compat tests for trace V3-V8 formats | Small |
| 16 | **Recorder UI** | Edge cases: shadow DOM, iframes, web components | Medium |
| 17 | **CLI Commands** | Tests for show-trace, uninstall, pdf/screenshot commands with error cases | Small |
| 18 | **Git Commit Plugin** | Tests for gitCommitInfoPlugin.ts (detached HEAD, shallow clones, missing git) | Small |

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
