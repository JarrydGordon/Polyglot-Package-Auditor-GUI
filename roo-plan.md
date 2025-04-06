```markdown
# Roo Code Execution Plan: Polyglot Package Auditor GUI

**Concise Goal:** Develop a cross-platform "Polyglot Package Auditor GUI" desktop application using Electron, Node.js, and JavaScript/TypeScript. The application must scan project directories, detect npm and pip dependencies, resolve the full dependency tree using CLI tools, check dependencies against the OSV vulnerability database and registry license information, and report findings (vulnerabilities, license compliance based on user config) in a sortable/filterable UI, implementing caching for external API calls and ensuring UI responsiveness.

1.  <switch_mode>lead-developer</switch_mode>

## Phase 1: Project Setup and Core Structure

2.  <new_task delegate="electron-main-setup">
    <mode>electron-main-setup</mode>
    <message>{
        "goal": "Initialize the Electron application structure, focusing on the main process entry point (`src/main/main.js`) and the preload script (`src/main/preload.js`). Create the main browser window and ensure basic application lifecycle events are handled.",
        "contextSummary": "The project requires an Electron application. This task sets up the foundation. Key files: `src/main/main.js`, `src/main/preload.js`. `main.js` initializes the app, creates the `BrowserWindow`, loads `index.html`, and sets up webPreferences (contextIsolation: true, nodeIntegration: false). `preload.js` will use `contextBridge` to expose specific IPC functionality safely to the renderer. See provided `src/main/main.js` outline for initialization logic.",
        "detailedInstructions": [
            "Create the directory structure: `src/main`, `src/renderer`.",
            "Implement `src/main/main.js` based on the provided outline:",
            "  - Require necessary Electron modules (`app`, `BrowserWindow`, `ipcMain`) and `path`.",
            "  - Define `createWindow` function to instantiate `BrowserWindow`.",
            "  - Set appropriate `webPreferences` including `preload: path.join(__dirname, 'preload.js')`, `contextIsolation: true`, `nodeIntegration: false`.",
            "  - Load `src/renderer/index.html` (this file will be created later).",
            "  - Handle `app.on('ready')`, `app.on('window-all-closed')`, and `app.on('activate')` events.",
            "  - Include placeholders for initializing and registering IPC handlers (they will be implemented in separate tasks).",
            "Implement a basic `src/main/preload.js`:",
            "  - Require `contextBridge`, `ipcRenderer` from Electron.",
            "  - Use `contextBridge.exposeInMainWorld('electronAPI', { ... });` to create the API namespace.",
            "  - Initially, expose placeholder functions or leave the object empty; it will be populated by the `ipc-handler` and `electron-renderer-base` tasks. See provided `preload.js` outline for examples.",
            "Ensure code adheres to `.clinerules-code` (e.g., use `require`, follow JS conventions)."
        ],
        "toolNotes": "Standard Node.js/Electron APIs. No specific MCP required for this setup task.",
        "completionCriteria": "Electron application window launches successfully loading a blank (or placeholder) `index.html` page. `main.js` and `preload.js` exist with the basic structure defined. Use `attempt_completion` with summary: 'Implemented initial Electron main process setup (main.js, preload.js) and window creation.'"
    }</message>
    </new_task>

3.  <new_task delegate="config-manager">
    <mode>config-manager</mode>
    <message>{
        "goal": "Implement the configuration management module (`src/main/configManager.js`) responsible for loading and saving application settings, specifically license allow/deny lists.",
        "contextSummary": "The application needs to store user-configurable settings like allowed/denied licenses. This module provides `getConfig` and `saveConfig` methods using a JSON file (`app-config.json`) in the user data directory. Default settings should be provided. See provided `src/main/configManager.js` outline for implementation details.",
        "detailedInstructions": [
            "Create the file `src/main/configManager.js`.",
            "Implement the `ConfigManager` class based on the provided outline.",
            "  - Use `electron.app.getPath('userData')` to determine the storage location for `app-config.json`.",
            "  - Use Node.js `fs.promises` for asynchronous file read/write operations.",
            "  - Define `DEFAULT_CONFIG` with initial sensible lists for `allowedLicenses` and `deniedLicenses`.",
            "  - Implement `loadConfig()`: reads the JSON file, merges with defaults, handles file-not-found errors by saving defaults.",
            "  - Implement `getConfig()`: returns the current configuration object (ensure it's loaded first).",
            "  - Implement `saveConfig(newConfig)`: merges the provided `newConfig` with the current config and writes the complete configuration back to the JSON file.",
            "Ensure error handling for file operations.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Requires Filesystem (FS) MCP capabilities for reading/writing the config file.",
        "completionCriteria": "`src/main/configManager.js` is implemented with `loadConfig`, `getConfig`, `saveConfig` methods operating correctly on a JSON file. Default config is handled. Use `attempt_completion` with summary: 'Implemented ConfigManager for loading/saving license configurations to app-config.json.'"
    }</message>
    </new_task>

4.  <new_task delegate="cache-manager">
    <mode>cache-manager</mode>
    <message>{
        "goal": "Implement the cache management module (`src/main/cacheManager.js`) to store and retrieve data from external APIs (OSV, registries), reducing redundant calls.",
        "contextSummary": "Caching is crucial for performance and API rate limits. This module needs `get`, `set`, and `clear` methods. Initial implementation should use a JSON file (`app-cache.json`) in the user data directory with a Time-To-Live (TTL) mechanism. See provided `src/main/cacheManager.js` outline (JSON file example).",
        "detailedInstructions": [
            "Create the file `src/main/cacheManager.js`.",
            "Implement the `CacheManager` class based on the provided JSON file outline.",
            "  - Use `electron.app.getPath('userData')` for the cache file location.",
            "  - Use `fs.promises` for file operations.",
            "  - Implement an internal `_loadCache()` and `_saveCache()` mechanism.",
            "  - Implement `get(key)`: checks if key exists and if its timestamp is within `CACHE_TTL_MS` (e.g., 24 hours). Returns data or null.",
            "  - Implement `set(key, data)`: stores the data along with the current timestamp.",
            "  - Implement `clear()`: empties the cache and saves.",
            "Handle file read/write errors gracefully.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Requires Filesystem (FS) MCP capabilities for reading/writing the cache file. SQLite is a future enhancement possibility.",
        "completionCriteria": "`src/main/cacheManager.js` is implemented with `get`, `set`, `clear` methods operating correctly on a JSON file, including TTL logic. Use `attempt_completion` with summary: 'Implemented CacheManager using JSON file storage with TTL.'"
    }</message>
    </new_task>

5.  <new_task delegate="ipc-handler">
    <mode>ipc-handler</mode>
    <message>{
        "goal": "Implement the central IPC handling logic (`src/main/ipcHandlers.js`) to manage communication between the main and renderer processes, and integrate it into `main.js`.",
        "contextSummary": "Electron relies on IPC for process communication. This module centralizes the setup of `ipcMain.handle` listeners for requests initiated from the renderer via the preload script. It orchestrates calls to backend services (Scanner, Resolver, etc.). See provided `src/main/ipcHandlers.js` outline.",
        "detailedInstructions": [
            "Create the file `src/main/ipcHandlers.js`.",
            "Implement the `registerIpcHandlers(ipcMain, services)` function as outlined.",
            "  - The `services` object will contain instances of Scanner, Resolver, Checkers, ConfigManager, CacheManager, and the `mainWindow`.",
            "  - Define handlers for:",
            "    - `dialog:openDirectory`: Use `electron.dialog.showOpenDialog`.",
            "    - `scan:start`: Orchestrate the scanning workflow (scan -> resolve -> check vulns -> check licenses -> combine results). Send progress/error updates back to renderer using `mainWindow.webContents.send('scan:progress', ...)` and `mainWindow.webContents.send('scan:error', ...)`. This handler will be complex.",
            "    - `config:get`: Call `configManager.loadConfig()` or `configManager.getConfig()`.",
            "    - `config:save`: Call `configManager.saveConfig(newConfig)`.",
            "    - `cache:clear`: Call `cacheManager.clear()`.",
            "Update `src/main/main.js`:",
            "  - Instantiate `ConfigManager`, `CacheManager` (potentially async initialization).",
            "  - Instantiate placeholder classes/objects for `Scanner`, `Resolver`, `VulnChecker`, `LicenseChecker` for now (they will be implemented later).",
            "  - Import `registerIpcHandlers` from `./ipcHandlers.js`.",
            "  - Call `registerIpcHandlers(ipcMain, { /* pass instances here */ });` within the `app.on('ready')` block *before* `createWindow()`.",
            "Update `src/main/preload.js` to expose the functions corresponding to the handlers defined here (e.g., `openDirectoryDialog`, `startScan`, `getConfig`, `saveConfig`, `clearCache`) and the event listeners (`onScanProgress`, `onScanError`, etc.) using `contextBridge`. Ensure `removeAllListeners` is also exposed for cleanup.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Uses Electron's `ipcMain`, `dialog` modules. Interacts with services that will use FS, CLI, API MCPs later.",
        "completionCriteria": "`ipcHandlers.js` is implemented, handlers are registered in `main.js`, and corresponding functions/listeners are exposed via `preload.js`. Basic IPC communication flow is established. Use `attempt_completion` with summary: 'Implemented IPC handlers, integrated into main process, and updated preload script.'"
    }</message>
    </new_task>

## Phase 2: Backend Logic Implementation

6.  <new_task delegate="directory-scanner">
    <mode>directory-scanner</mode>
    <message>{
        "goal": "Implement the directory scanning module (`src/main/scanner.js`) to find project manifest files (`package.json`, `requirements.txt`, `pyproject.toml`) within a selected directory.",
        "contextSummary": "The first step of the analysis is finding projects. This module needs a `scanDirectory(directoryPath)` method that searches for specified manifest files and returns an array of `Project` objects ({ path, type, projectName }). See provided `src/main/scanner.js` outline.",
        "detailedInstructions": [
            "Create the file `src/main/scanner.js`.",
            "Implement the `Scanner` class with the `async scanDirectory(directoryPath)` method based on the outline.",
            "  - Use `fs.promises.readdir` and potentially `fs.promises.access` to find manifest files.",
            "  - Map found filenames (`package.json`, etc.) to project types (`npm`, `pip`).",
            "  - Determine `projectName` (e.g., based on the directory name).",
            "  - Handle potential errors during directory reading.",
            "  - The current outline focuses on the immediate directory; consider if recursive scanning is needed (initially, no).",
            "  - Return an array of project objects.",
            "Update `src/main/main.js` to instantiate the actual `Scanner` class instead of a placeholder.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Requires Filesystem (FS) MCP capabilities for directory listing and file checking.",
        "completionCriteria": "`src/main/scanner.js` implemented correctly, finds specified manifest files in a given directory, and returns structured project data. Integrated into `main.js`. Use `attempt_completion` with summary: 'Implemented directory scanner for npm/pip projects.'"
    }</message>
    </new_task>

7.  <new_task delegate="dependency-resolver">
    <mode>dependency-resolver</mode>
    <message>{
        "goal": "Implement the dependency resolution module (`src/main/resolver.js`) using CLI tools (`npm ls --json`, `pipdeptree --json-tree`) to get the full dependency tree for found projects.",
        "contextSummary": "Requires executing external CLI commands and parsing their JSON output. Needs `resolveDependencies(project)` method. Handles both 'npm' and 'pip' project types. See provided `src/main/resolver.js` outline. This is noted as a potential Boomerang Task candidate due to complexity and external process dependency.",
        "detailedInstructions": [
            "Create the file `src/main/resolver.js`.",
            "Implement the `Resolver` class with the `async resolveDependencies(project)` method based on the outline.",
            "  - Use `child_process.exec` (or `execPromise`) to run the appropriate command (`npm ls --json --all` or `pipdeptree --json-tree`) in the project's directory (`cwd`).",
            "  - Implement the `parseOutput(output, projectType, manifestPath)` method:",
            "    - Parse JSON output from `npm ls` (recursive structure).",
            "    - Parse JSON output from `pipdeptree` (list of trees).",
            "    - Extract dependency `name`, `version`, `dependencyType` ('direct'/'transitive'), and `sourceManifest`.",
            "    - Handle potential parsing errors.",
            "    - Deduplicate dependencies (e.g., by name + version).",
            "  - Implement robust error handling for CLI command execution (command not found, non-zero exit code, stderr parsing). Increase `maxBuffer` if needed.",
            "Update `src/main/main.js` to instantiate the actual `Resolver` class.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Requires Command Line Interface (CLI) MCP capabilities. Address ambiguities: ensure `pipdeptree` is noted as a prerequisite; consider strategies or notes about Python virtual environments.",
        "completionCriteria": "`src/main/resolver.js` implemented, correctly executes `npm` and `pipdeptree` commands, parses their JSON output, handles errors, and returns a flat list of unique `Dependency` objects. Integrated into `main.js`. Use `attempt_completion` with summary: 'Implemented dependency resolver using npm/pipdeptree CLI tools.'"
    }</message>
    </new_task>

8.  <new_task delegate="vulnerability-checker">
    <mode>vulnerability-checker</mode>
    <message>{
        "goal": "Implement the vulnerability checking module (`src/main/vulnChecker.js`) to query the OSV API for known vulnerabilities based on dependency names and versions.",
        "contextSummary": "Requires querying the OSV batch API (`https://api.osv.dev/v1/querybatch`). Needs `checkVulnerabilities(dependencies)` method. Must interact with the `CacheManager`. See provided `src/main/vulnChecker.js` outline. Noted as potential Boomerang Task candidate.",
        "detailedInstructions": [
            "Create the file `src/main/vulnChecker.js`.",
            "Implement the `VulnChecker` class, accepting `cacheManager` in its constructor.",
            "Implement `async checkVulnerabilities(dependencies)`:",
            "  - Iterate through dependencies, check cache first using key like `vuln-name@version`.",
            "  - For cache misses, prepare queries for the OSV batch API.",
            "  - Map project types ('npm', 'pip') to OSV ecosystems ('npm', 'PyPI').",
            "  - Make a POST request to `OSV_API_BATCH_URL` with the collected queries.",
            "  - Process the response:",
            "    - Map results back to corresponding dependencies.",
            "    - Parse vulnerability details (id, summary, severity, affected versions, references). Implement basic `determineSeverity` helper.",
            "    - Store successful results in the cache via `cacheManager.set()`.",
            "  - Handle API errors gracefully.",
            "  - Return an array of objects `{ dependency, vulnerabilities: VulnObject[] }`.",
            "Update `src/main/main.js` to instantiate the actual `VulnChecker` class, passing the `cacheManager` instance.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Requires API MCP for HTTP POST requests (e.g., using `axios` or `node-fetch`). Requires interaction with `CacheManager` (passed in constructor). Address ambiguity: clarify mapping from internal project type to OSV ecosystem name.",
        "completionCriteria": "`src/main/vulnChecker.js` implemented, interacts with cache, queries OSV batch API, parses results, handles errors, and returns structured vulnerability data. Integrated into `main.js`. Use `attempt_completion` with summary: 'Implemented vulnerability checker using OSV API and caching.'"
    }</message>
    </new_task>

9.  <new_task delegate="license-checker">
    <mode>license-checker</mode>
    <message>{
        "goal": "Implement the license checking module (`src/main/licenseChecker.js`) to retrieve dependency licenses from registry APIs (npmjs, PyPI) and check compliance against user configuration.",
        "contextSummary": "Requires querying registry APIs, parsing license information, normalizing license strings, comparing against allow/deny lists from `ConfigManager`, and using `CacheManager`. Needs `checkLicenses(dependencies)` method. See provided `src/main/licenseChecker.js` outline. Noted as potential Boomerang Task candidate.",
        "detailedInstructions": [
            "Create the file `src/main/licenseChecker.js`.",
            "Implement the `LicenseChecker` class, accepting `cacheManager` and `configManager` in its constructor.",
            "Implement `async checkLicenses(dependencies)`:",
            "  - Get current allowed/denied license lists from `configManager.getConfig()`.",
            "  - Iterate through dependencies, check cache first using key like `license-name@version`.",
            "  - For cache misses, determine registry (NPM/PyPI based on `sourceManifest` heuristic) and prepare API lookups.",
            "  - Make GET requests to registry APIs (`registry.npmjs.org`, `pypi.org/pypi/.../json`).",
            "  - Process responses:",
            "    - Extract license information (handle various fields/formats).",
            "    - Normalize the license string using `normalizeLicenseString` helper (implement basic normalization, e.g., uppercase, handle object format).",
            "    - Store successful results (normalized license) in the cache.",
            "  - Implement `evaluateLicense(license, allowedSet, deniedSet)` helper to determine status ('Allowed', 'Denied', 'Unknown').",
            "  - Handle API errors.",
            "  - Return an array of objects `{ dependency, license, licenseStatus }`.",
            "Update `src/main/main.js` to instantiate the actual `LicenseChecker` class, passing `cacheManager` and `configManager` instances.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Requires API MCP for HTTP GET requests. Requires interaction with `CacheManager` and `ConfigManager`. Address ambiguities: strategy for fetching potentially version-specific license info from npm; policy for licenses neither allowed nor denied (treat as 'Unknown').",
        "completionCriteria": "`src/main/licenseChecker.js` implemented, interacts with cache and config, queries registry APIs, parses/normalizes licenses, evaluates compliance, handles errors, and returns structured license data. Integrated into `main.js`. Use `attempt_completion` with summary: 'Implemented license checker using registry APIs, caching, and config.'"
    }</message>
    </new_task>

## Phase 3: Renderer UI Implementation

10. <new_task delegate="electron-renderer-base">
    <mode>electron-renderer-base</mode>
    <message>{
        "goal": "Set up the basic HTML structure (`src/renderer/index.html`) and the renderer process entry point (`src/renderer/renderer.js`). Implement basic UI elements (buttons, status display) and connect them to the main process via the `electronAPI` exposed by the preload script.",
        "contextSummary": "This task creates the user-facing part of the application. `index.html` provides the structure. `renderer.js` handles UI logic and uses `window.electronAPI` (from preload) to trigger actions (select directory, start scan) and listen for events (progress, errors) from the main process. See provided outlines for `index.html` (implied) and `renderer.js`.",
        "detailedInstructions": [
            "Create `src/renderer/index.html`:",
            "  - Include basic HTML structure (doctype, head, body).",
            "  - Add placeholder elements: a button to select directory (`#select-dir-btn`), a span to display selection (`#selected-dir`), a button to start scan (`#start-scan-btn`), a table structure for results (`#results-table-body`), a status bar element (`#status-bar`), and a settings button (`#settings-btn`).",
            "  - Link to `styles.css` (create basic empty file `src/renderer/styles.css`).",
            "  - Include `<script src=\"renderer.js\" defer></script>`.",
            "Create `src/renderer/renderer.js` based on the outline:",
            "  - Add event listener for `DOMContentLoaded`.",
            "  - Get references to the HTML elements.",
            "  - Add click listeners to buttons:",
            "    - Select Dir Btn: Call `window.electronAPI.openDirectoryDialog()`, update UI.",
            "    - Start Scan Btn: Call `window.electronAPI.startScan(currentDirectory)`, disable button, clear results, handle returned results/errors.",
            "    - Settings Btn: Placeholder console log for now.",
            "  - Implement listeners for main process events using `window.electronAPI.onScanProgress`, `onScanError`, `onConfigSaved`, `onCacheCleared` to update the UI (status bar, potentially results).",
            "  - Implement helper functions `updateStatus(message, percentage)` and `displayResults(results)` (basic table row creation for now).",
            "  - Ensure scan button state (enabled/disabled) is managed correctly.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Standard Web APIs (DOM manipulation, events). Interacts with the main process exclusively through the `window.electronAPI` exposed by the preload script.",
        "completionCriteria": "`index.html` and `renderer.js` are created. Basic UI elements are present. Clicking 'Select Directory' opens dialog. Clicking 'Start Scan' triggers IPC call and handles basic response/status updates. Use `attempt_completion` with summary: 'Implemented basic renderer structure (HTML, JS), core UI elements, and IPC connections.'"
    }</message>
    </new_task>

11. <new_task delegate="ui-report-table">
    <mode>ui-report-table</mode>
    <message>{
        "goal": "Implement the main report table component (`src/renderer/components/ReportTable.js`) responsible for displaying the scan results with sorting and filtering capabilities.",
        "contextSummary": "This is a complex UI component. It needs to render the combined analysis results (dependency, license, vulnerability info) in a table. Users should be able to sort by columns and filter the results. See conceptual pseudocode in `src/renderer/components/ReportTable.js` outline. Consider using a simple implementation first, without a heavy framework, unless specified otherwise.",
        "detailedInstructions": [
            "Create `src/renderer/components/ReportTable.js` (and potentially associated CSS).",
            "Implement the report table logic. This can be a class or function that takes the results data array.",
            "  - Render table headers (`<th>`) corresponding to the `DisplayResult` fields (Name, Version, License, License Status, Vuln Count, etc.).",
            "  - Add click handlers to headers for sorting.",
            "  - Implement sorting logic (track sort column/direction, re-render).",
            "  - Add an input field for filtering.",
            "  - Implement filtering logic (filter data based on input text across relevant fields, re-render).",
            "  - Render table rows (`<tr>`/`<td>`) based on the (filtered and sorted) data.",
            "  - Apply basic styling to rows/cells based on data (e.g., red for denied license, orange for vulnerabilities).",
            "Integrate this component/logic into `src/renderer/renderer.js`: Modify the `displayResults` function to use this component/logic to render the data into `#results-table-body`.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "Primarily UI development using standard Web APIs or a chosen framework. Focus on DOM manipulation, event handling, and state management for sorting/filtering. Virtualization for large datasets is an advanced optimization.",
        "completionCriteria": "Report table component is implemented and integrated. Displays scan results correctly. Sorting by clicking headers and basic text filtering works. Use `attempt_completion` with summary: 'Implemented report table component with sorting and filtering.'"
    }</message>
    </new_task>

12. <new_task delegate="ui-settings-view">
    <mode>ui-settings-view</mode>
    <message>{
        "goal": "Implement the settings view component (`src/renderer/components/SettingsView.js`) allowing users to view and edit the license allow/deny lists and potentially clear the cache.",
        "contextSummary": "Provides the UI for configuration. Needs to load current settings via IPC (`config:get`), display them (e.g., in text areas), and save changes via IPC (`config:save`). May include a cache clear button triggering `cache:clear`. See conceptual pseudocode in `src/renderer/components/SettingsView.js` outline.",
        "detailedInstructions": [
            "Create `src/renderer/components/SettingsView.js` (and potentially associated CSS).",
            "Implement the settings view logic (e.g., as functions to show/hide/populate a modal or dedicated view area in `index.html`).",
            "  - On view display, call `window.electronAPI.getConfig()` to fetch current settings.",
            "  - Populate input elements (e.g., text areas for `allowedLicenses`, `deniedLicenses`, joining arrays with newlines).",
            "  - Add a 'Save' button:",
            "    - On click, read values from inputs.",
            "    - Parse text areas back into arrays (split by newline, trim, filter empty).",
            "    - Call `window.electronAPI.saveConfig({ allowedLicenses: ..., deniedLicenses: ... })`.",
            "    - Handle confirmation/feedback using `window.electronAPI.onConfigSaved`.",
            "  - Add a 'Clear Cache' button (optional):",
            "    - Add confirmation dialog.",
            "    - On confirm, call `window.electronAPI.clearCache()`.",
            "    - Handle feedback using `window.electronAPI.onCacheCleared`.",
            "Modify `src/renderer/renderer.js` to trigger the display of this settings view when the settings button is clicked.",
            "Adhere to `.clinerules-code`."
        ],
        "toolNotes": "UI development using standard Web APIs or a chosen framework. Interacts with main process via `window.electronAPI` for config/cache operations.",
        "completionCriteria": "Settings view component implemented and integrated. Loads config, displays allow/deny lists, allows editing and saving via IPC. Cache clear button works if included. Use `attempt_completion` with summary: 'Implemented settings view component for license config and cache management.'"
    }</message>
    </new_task>

## Phase 4: Testing and Documentation

13. <new_task delegate="test-engineer">
    <mode>test-engineer</mode>
    <message>{
        "goal": "Develop unit, integration, and potentially end-to-end (E2E) tests for the application to ensure correctness and stability.",
        "contextSummary": "Testing is required across different levels. Unit tests for individual modules (parsers, checkers, cache). Integration tests for module interactions and IPC. E2E tests for user workflows. The provided analysis outlines specific testing requirements.",
        "detailedInstructions": [
            "Set up a testing framework (e.g., Jest, Mocha/Chai).",
            "Write Unit Tests for:",
            "  - `scanner.js`: Mock `fs` calls, test manifest detection logic.",
            "  - `resolver.js`: Mock `child_process`, test CLI command formation and output parsing (npm/pip).",
            "  - `vulnChecker.js`: Mock API calls and cache, test OSV query formation, response parsing, severity determination, caching logic.",
            "  - `licenseChecker.js`: Mock API calls, cache, config. Test registry query formation, license extraction/normalization, compliance evaluation, caching.",
            "  - `cacheManager.js`: Test get/set/clear/TTL logic (mock `fs` if using files).",
            "  - `configManager.js`: Test load/save/default logic (mock `fs`).",
            "Write Integration Tests for:",
            "  - Scan Workflow: Test interaction between Scanner -> Resolver -> Checkers in the main process.",
            "  - IPC Communication: Test `ipcMain` handlers trigger correct service calls and `preload` script exposes functions that successfully invoke handlers.",
            "Set up E2E Testing Framework (optional but recommended, e.g., Spectron/Playwright):",
            "  - Write basic E2E test stubs for: selecting directory, starting scan, verifying basic results display, opening settings.",
            "Ensure tests adhere to project standards and `.clinerules`."
        ],
        "toolNotes": "Requires knowledge of JavaScript testing frameworks (Jest recommended) and potentially Electron E2E testing tools (Spectron, Playwright). Mocking libraries (like Jest mocks) will be essential.",
        "completionCriteria": "Testing framework set up. Representative unit tests for core backend modules and utilities are implemented. Basic integration tests for IPC and service interaction are present. E2E test stubs created. Use `attempt_completion` with summary: 'Implemented unit tests for core modules, integration test stubs, and set up testing framework.'"
    }</message>
    </new_task>

14. <new_task delegate="docs-writer">
    <mode>docs-writer</mode>
    <message>{
        "goal": "Create comprehensive documentation for the project, starting with the main `README.md` file.",
        "contextSummary": "Documentation is essential for users and future developers. The `README.md` should cover overview, features, installation (including prerequisites like Node, npm, pip, pipdeptree), usage, and build instructions. Code comments are also part of documentation.",
        "detailedInstructions": [
            "Create/Update the `README.md` file in the project root.",
            "Include the following sections:",
            "  - Project Title (`Polyglot Package Auditor GUI`)",
            "  - Brief Overview/Goal",
            "  - Key Features (directory scanning, npm/pip support, vulnerability checking via OSV, license checking, configurable policies, caching, GUI report).",
            "  - Installation Prerequisites (Node.js, npm, Python, pip, `pipdeptree`). Specify version recommendations if known.",
            "  - Installation Steps (e.g., `git clone`, `npm install`).",
            "  - Usage Guide (How to launch, select directory, start scan, interpret results, use settings).",
            "  - Build Instructions (How to package the Electron app for distribution, e.g., using `electron-builder`).",
            "  - (Optional) Architecture overview section.",
            "Review existing code for necessary code comments explaining complex logic, function parameters, etc., and add them where missing.",
            "Ensure Markdown is well-formatted."
        ],
        "toolNotes": "Standard Markdown formatting.",
        "completionCriteria": "`README.md` file created with all specified sections, providing clear instructions for installation, usage, and building. Code comments reviewed/added. Use `attempt_completion` with summary: 'Created comprehensive README.md and reviewed code comments.'"
    }</message>
    </new_task>

**Conclusion:** The `lead-developer` mode will monitor the progress of these delegated tasks, facilitate communication between modes if necessary, and perform final integration and review steps.
```