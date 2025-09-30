# RFC 0000: A Textual Format for HTTP API Test Scenarios

> WIP.

> TODO : bibliography ? (format http).

> TODO : global variable that MUST be available

> TODO : usual section about MUST/SHOULD/etc...

**Abstract**

This document specifies a textual file format designed for defining and executing test scenarios against Application Programming Interfaces (APIs) that communicate over the Hypertext Transfer Protocol (HTTP). The format emphasizes readability and simplicity, leveraging Github Flavored Markdown (GFM) for its structure. It provides a clear separation between global setup, per-step execution, and individual HTTP requests, along with optional scripting capabilities.

## 1. Introduction

Modern software development heavily relies on robust Application Programming Interfaces (APIs). Ensuring the correctness and reliability of these APIs requires comprehensive testing. This document proposes a human-readable and machine-parsable file format for defining test scenarios. The format aims to standardize how test sequences, including HTTP requests, pre-request logic, and post-request validations, are described in a plain text file. This approach facilitates version control, collaboration, and automated execution of API tests.

## 2. Core Concepts

### 2.1 Structured document using "key expressions"

A test scenario file is composed of distinct sections, identified by "key expressions" in a header line. A key expression is a string enclosed by colons (e.g., `:KeyExpression:`), and they are case-insensitive. Any text within a section that does not begin with a key expression is considered a comment and is ignored by parsers.

The primary structural elements (header line starting with 2 hash characters `##`) of a test scenario are:

*   **:Preamble:**: Defines global configurations and hooks that execute before, between, or after test steps.
*   **:Step:**: Defines individual test actions, typically involving an HTTP request and associated logic.

Sections are declared sequentially within the file.

### 2.2 Scripting language support

Testing an API using a scenario, i.e. a sequence of API calls, requires some processing in between the actual call. 

_For the sake of simplicity, this document will use Javascript. However, any programming language is usable._

There are two constraints :

* There is only one programming language used throughout the scenario. E.g., one cannot find some fenced blocks in Javascript, and other fenced blocks in Python. _The document is malformed in such case._
* The fenced blocks containing statements in the chosen programming language MUST target that programming language. E.g. a scenario that uses the Java programming language MUST use the keyword `java` for its fenced blocks describing the processings.

## 3. The Preamble Section

The `:Preamble:` section is an optional, single section at the beginning of the file. It serves to configure the testing environment and define reusable setup/teardown actions. Any text directly within the `:Preamble:` section but outside its sub-sections is considered a comment.

The `:Preamble:` section may contain the following sub-sections (header line starting with 3 hash characters `###`), which must appear in the order listed below if present:

*   **:Globals:**
*   **:Before all:**
*   **:Before each:**
*   **:After each:**
*   **:After all:**

Each of these sub-sections defines a specific execution hook or configuration scope.

### 3.1. The :Globals: Sub-section

The `:Globals:` sub-section is optional and may appear at most once within the `:Preamble:`. Its primary purpose is to define global variables or import necessary JavaScript modules that will be accessible throughout the test scenario.

The content of the `:Globals:` sub-section consists of a single, mandatory fenced code block, which MUST be of type `javascript`. Any text outside this specific JavaScript block within the `:Globals:` sub-section is treated as a comment.

Example:

```markdown
## :Preamble:
This is a comment about the preamble.

### :Globals:
These are global definitions.
```js
// specify javascript imports
// Define "Environment variables"
const baseUrl = "https://api.example.com/v1";
let authToken = ""; // This will be populated later.
```

### 3.2. The :Before all: Sub-section

The `:Before all:` sub-section is optional and may appear at most once within the `:Preamble:`. It contains JavaScript code that is executed once before any test step begins, and prior to the first execution of any `:Before each:` script.

The content of the `:Before all:` sub-section consists of a single, mandatory fenced code block, which MUST be of type `javascript`. Any text outside this JavaScript block is treated as a comment.

Example:

```markdown
### :Before all:
This script runs once before all steps.
```js
// Javascript to execute once before starting any step
// (before the first "before each" if any)
console.log("Starting test suite...");
// e.g., obtain an initial global token
```

### 3.3. The :Before each: Sub-section

The `:Before each:` sub-section is optional and may appear at most once within the `:Preamble:`. It contains JavaScript code that is executed before each individual test step is processed.

The content of the `:Before each:` sub-section consists of a single, mandatory fenced code block, which MUST be of type `javascript`. Any text outside this JavaScript block is treated as a comment.

Example:

```markdown
### :Before each:
This script runs before every step.
```js // Javascript to execute before each step
console.log(`Executing step ${currentStepNumber}...`);
// e.g., reset a state or log step start
```

### 3.4. The :After each: Sub-section

The `:After each:` sub-section is optional and may appear at most once within the `:Preamble:`. It contains JavaScript code that is executed after each individual test step has completed its HTTP request and post-request script.

The content of the `:After each:` sub-section consists of a single, mandatory fenced code block, which MUST be of type `javascript`. Any text outside this JavaScript block is treated as a comment.

Example:

```markdown
### :After each:
This script runs after every step.
```js 
// Javascript to execute after each step
console.log(`Step ${currentStepNumber} finished.`);
// e.g., log step duration or clean up per-step resources
```

### 3.5. The :After all: Sub-section

The `:After all:` sub-section is optional and may appear at most once within the `:Preamble:`. It contains JavaScript code that is executed once after all test steps have completed, including their respective `:After each:` scripts.

The content of the `:After all:` sub-section consists of a single, mandatory fenced code block, which MUST be of type `javascript`. Any text outside this JavaScript block is treated as a comment.

Example:

```markdown
### :After all:
This script runs once after all steps.
```js
// Javascript to execute once after the last step
// (after the last "before each" if any)
console.log("Test suite finished.");
// e.g., generate a summary report or clean up global resources
```

## 4. The Step Section

The `:Step:` section (header line starting with 2 hash characters `##`) defines an individual test action. A test scenario file consists of one or more `:Step:` sections, which are processed in the order they appear in the file.

Each `:Step:` section begins with the `:Step:` key expression, optionnally followed by a step number and a descriptive text. The step number and the descriptive text after it are considered a comment and can be used for clarity, logging, or generating test function names.

Example:

```markdown
## :Step: 1. Authenticate User
```

Within a `:Step:` section, the following components can appear in sequence:

1.  **Optional Pre-request Script**: A fenced code block of type `javascript`.
2.  **Mandatory HTTP Request**: A fenced code block of type `http`.
3.  **Optional Post-request Script**: A fenced code block of type `javascript`.

Any text within a `:Step:` section that is not part of these fenced code blocks is treated as a comment.

### 4.1. Pre-request Script

This optional JavaScript block executes immediately before the HTTP request for the current step is sent. It can be used to prepare request parameters, calculate dynamic values, or set up local state for the step.

Example:

```markdown
```js
// optionnal pre-request script
const queryParams = new URLSearchParams({
    status: "active",
    limit: 10
});
pm.variables.set("productsQuery", queryParams.toString());
```

### 4.2. HTTP Request

This is a mandatory section within each `:Step:`. It defines the actual HTTP request to be made. The content must be a fenced code block of type `http`. The block is _malformed_ when it contains more than one request.

Variables defined in `:Globals:` or set within JavaScript scripts (e.g., pre-request or post-request scripts) can be used within the HTTP request block by enclosing their names in double curly braces (e.g., `{{my_variable}}`).

The format of the HTTP request block generally follows common conventions for defining HTTP messages, including the method, Uniform Resource Identifier (URI), headers, and body.

Example:

```markdown
```http
GET {{baseUrl}}/products?{{productsQuery}}
Accept: application/json
Authorization: Bearer {{authToken}}
```

### 4.3. Post-request Script

This optional JavaScript block executes immediately after the HTTP response for the current step has been received. It is typically used for:

*   Validating the response (e.g., status code, body content, headers).
*   Extracting data from the response to be used in subsequent steps.
*   Logging or reporting test outcomes.

Example:

```markdown
```js 
// optionnal post-request script
// Check status code
if (response.status !== 200) {
    throw new Error(`Expected status 200 but got ${response.status}`);
}
// Store a value for next step
const responseBody = JSON.parse(response.body);
pm.variables.set("firstProductId", responseBody.data[0].id);
```

## 5. File Structure and Comments

The test scenario file uses Github Flavored Markdown (GFM). Key expressions are put in headers. Any line or block of text that is not explicitly part of a key expression's defined content (like a JavaScript or HTTP block) or another key expression is considered a comment. This allows for liberal use of descriptive text to enhance readability.

The order of sections is strict:
1.  Optional `:Preamble:` section (if present, its sub-sections must be in order).
2.  One or more `:Step:` sections, in sequential order.

## Annex A. Acronyms

*   **API**: Application Programming Interface. A set of definitions and protocols for building and integrating application software.
*   **GFM**: Github Flavored Markdown. A dialect of Markdown that is used on GitHub and other platforms.
*   **HTTP**: Hypertext Transfer Protocol. An application-layer protocol for transmitting hypermedia documents, such as HTML. It was designed for communication between web browsers and web servers, but it can be used for other purposes as well.
*   **IETF**: Internet Engineering Task Force. An organization that develops and promotes Internet standards.
*   **RFC**: Request For Comments. A formal document series from the IETF that describes the Internet's technical specifications and organizational notes.
*   **URI**: Uniform Resource Identifier. A sequence of characters that identifies a logical or physical resource.

