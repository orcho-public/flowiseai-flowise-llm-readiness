# Documentation–Implementation Mismatches and LLM / AI Risk

**Repository analyzed:** `Flowise`

---
## Executive summary

- **44 findings** were reviewed from the raw Flowise contradictions output.
- **38 findings are included below as verified documentation–implementation contradictions** (items #6, #14, #16, #24, #25, #41 were excluded as non‑verified/overstated/ambiguous).
- The dominant patterns are:
  - **async vs sync contract drift** (missing `await` risk)
  - **return type drift** (`Promise<T>` vs `T`, `string` vs structured object, `void` vs object, arrays vs single value)
  - **parameter naming drift** (e.g., `obj` vs `data`)
- These issues may not break runtime behavior today, but they **materially increase LLM hallucination risk** by misrepresenting runtime contracts.

---

**Note for readers**

This report is a reformatted and filtered version of the “Flowise — Orcho Code & Documentation Consistency Report.” Only findings verified in the accompanying verification list are included.

---
## How LLM and AI risk arises

LLMs rely heavily on comments, JSDoc, and local documentation as high‑confidence signals.  
When documentation contradicts implementation, models must infer intent and guess behavior.

In practice:
- comments outweigh code during inference,
- contradictions force reconciliation,
- reconciliation optimizes for plausibility, not correctness.

This produces plausible‑but‑wrong code:
- missing `await`,
- invalid assumptions about returned fields,
- incorrect iteration and destructuring,
- incorrect usage patterns (e.g., treating strings as objects or vice‑versa).

Each failure increases retries, corrective turns, and token usage—directly increasing cost in AI‑assisted workflows.

---
## Fully verified contradictions (high LLM / AI risk)

---

### 1. `CachePool`

- **Location:** `packages/server/src/CachePool.ts:7`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may generate calling code that treats initialization as immediate and omits `await`, producing subtle runtime failures due to unresolved promises.

---

### 2. `JSONPathExtractorTool`

- **Location:** `packages/components/nodes/tools/JSONPathExtractor/JSONPathExtractor.ts:10`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Models may produce usage patterns that assume immediate return values, causing broken tool execution flows.

---

### 3. `WordLoader`

- **Location:** `packages/components/nodes/documentloaders/MicrosoftWord/WordLoader.ts:11`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may omit `await` and treat the loader output as available immediately, generating pipelines that fail during runtime.

---

### 4. `PowerpointLoader`

- **Location:** `packages/components/nodes/documentloaders/MicrosoftPowerpoint/PowerpointLoader.ts:11`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Models may hallucinate synchronous composition and produce incorrect chaining without awaiting.

---

### 5. `AWSChatBedrock_ChatModels`

- **Location:** `packages/components/nodes/chatmodels/AWSBedrock/AWSChatBedrock.ts:11`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may generate incorrect initialization and invocation patterns, leading to unresolved promises and invalid chatflow execution.

---

### 7. `AWSBedrock_LLMs`

- **Location:** `packages/components/nodes/llms/AWSBedrock/AWSBedrock.ts:12`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may treat the options/initialization as immediate, causing toolchain and config wiring to break.

---

### 8. `LoadOfSheet`

- **Location:** `packages/components/nodes/documentloaders/MicrosoftExcel/ExcelLoader.ts:12`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Models may omit `await` and mis-handle returned values, producing incorrect ingestion pipelines.

---

### 9. `CSVLoader`

- **Location:** `packages/components/nodes/documentloaders/Csv/CsvLoader.ts:19`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may generate synchronous calls and incorrectly assume output is immediately available.

---

### 10. `ChatNemoGuardrailsModel`

- **Location:** `packages/components/nodes/chatmodels/ChatNemoGuardrails/ChatNemoGuardrails.ts:26`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Increases likelihood of missing `await` and generating incorrect response handling in chat pipelines.

---

### 11. `UnstructuredLoader`

- **Location:** `packages/components/nodes/documentloaders/Unstructured/Unstructured.ts:29`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may generate code that expects immediate extraction results and fails when promises propagate.

---

### 12. `ChatFireworks`

- **Location:** `packages/components/nodes/chatmodels/ChatFireworks/core.ts:30`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Models may hallucinate synchronous patterns for model invocation and streaming, causing broken runtime behavior.

---

### 13. `HuggingFaceInference`

- **Location:** `packages/components/nodes/llms/HuggingFaceInference/core.ts:32`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may omit `await` and generate incorrect wiring of inference outputs.

---

### 15. `APIChain`

- **Location:** `packages/components/nodes/chains/ApiChain/postCore.ts:55`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may generate synchronous composition, missing `await` and producing non-deterministic chain execution.

---

### 17. `JSONPathExtractor_Tools`

- **Location:** `packages/components/nodes/tools/JSONPathExtractor/JSONPathExtractor.ts:73`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Encourages hallucinated synchronous tool invocation patterns.

---

### 18. `DynamoDBRetrieveTool`

- **Location:** `packages/components/nodes/tools/AWSDynamoDBKVStorage/AWSDynamoDBKVStorage.ts:100`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may skip `await`, generating code that reads from unresolved promise values.

---

### 19. `AWSDynamoDBKVStorage_Tools`

- **Location:** `packages/components/nodes/tools/AWSDynamoDBKVStorage/AWSDynamoDBKVStorage.ts:183`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Models may hallucinate synchronous option-building and tool registry behavior.

---

### 20. `XMLAgentOutputParser`

- **Location:** `packages/components/src/agents.ts:873`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** LLMs may generate incorrect parser chaining and assume immediate parsing results.

---

### 21. `JsonOutputToolsParser`

- **Location:** `packages/components/src/agents.ts:997`
- **Severity:** MEDIUM
- **Documented:** Synchronous function
- **Runtime:** Asynchronous
- **Risk:** Increases risk of missing `await` in tool-call parsing paths, leading to silent failures.

---

### 22. `JsonOutputToolsParser`

- **Location:** `packages/components/src/agents.ts:997`
- **Severity:** MEDIUM
- **Documented:** Returns tool call id
- **Runtime:** Returns `parsedToolCalls`
- **Risk:** LLMs may generate code that treats the return as a scalar id (string/number) instead of a structured collection, breaking tool dispatch logic.

---

### 23. `extractValues`

- **Location:** `packages/server/src/utils/index.ts:1826`
- **Severity:** MEDIUM
- **Documented:** Expects parameter name `obj` in outer JSDoc
- **Runtime:** Inner helper uses parameter name `data` (no JSDoc for that inner signature)
- **Risk:** Models may hallucinate wrapper signatures or pass the wrong variable, producing pipelines that look right but extract nothing.

---

### 26. `CachePool`

- **Location:** `packages/server/src/CachePool.ts:7`
- **Severity:** HIGH
- **Documented:** Returns `CachePool`
- **Runtime:** Returns `Promise<any | undefined>`
- **Risk:** LLMs may generate immediate method calls on the return value and omit undefined handling, creating brittle cache initialization logic.

---

### 27. `JSONPathExtractorTool`

- **Location:** `packages/components/nodes/tools/JSONPathExtractor/JSONPathExtractor.ts:10`
- **Severity:** HIGH
- **Documented:** Returns various
- **Runtime:** Returns `Promise<string>`
- **Risk:** Vague comments encourage hallucinated return shapes and incorrect branching/serialization code.

---

### 28. `WordLoader`

- **Location:** `packages/components/nodes/documentloaders/MicrosoftWord/WordLoader.ts:11`
- **Severity:** HIGH
- **Documented:** Returns `Document`
- **Runtime:** Returns `Promise<Document[]>`
- **Risk:** LLMs may treat it as a single document and skip iteration, producing incomplete ingestion and silent truncation.

---

### 29. `PowerpointLoader`

- **Location:** `packages/components/nodes/documentloaders/MicrosoftPowerpoint/PowerpointLoader.ts:11`
- **Severity:** HIGH
- **Documented:** Returns `Document`
- **Runtime:** Returns `Promise<Document[]>`
- **Risk:** Models may generate single-document processing code and drop slides/pages without noticing.

---

### 30. `AWSBedrock_LLMs`

- **Location:** `packages/components/nodes/llms/AWSBedrock/AWSBedrock.ts:12`
- **Severity:** HIGH
- **Documented:** Returns not specified
- **Runtime:** Returns `Promise<INodeOptionsValue[]>`
- **Risk:** Underspecified docs force the model to guess return types, increasing hallucinations in option wiring and node initialization.

---

### 31. `LoadOfSheet`

- **Location:** `packages/components/nodes/documentloaders/MicrosoftExcel/ExcelLoader.ts:12`
- **Severity:** HIGH
- **Documented:** Returns `Document`
- **Runtime:** Returns `Promise<Document[]>`
- **Risk:** LLMs may hallucinate a single-document contract and generate incorrect downstream processing.

---

### 32. `CSVLoader`

- **Location:** `packages/components/nodes/documentloaders/Csv/CsvLoader.ts:19`
- **Severity:** HIGH
- **Documented:** Returns an array of strings (`string[]`)
- **Runtime:** Returns `Promise<string[]>` (async)
- **Risk:** Models may hallucinate document-like fields (`pageContent`, `metadata`) and generate code that breaks at runtime.

---

### 33. `ChatNemoGuardrailsModel`

- **Location:** `packages/components/nodes/chatmodels/ChatNemoGuardrails/ChatNemoGuardrails.ts:26`
- **Severity:** HIGH
- **Documented:** Returns `string`
- **Runtime:** Returns `Promise<ChatResult>`
- **Risk:** LLMs may treat the return as raw text, skipping structured handling and causing downstream prompt/response bugs.

---

### 34. `UnstructuredLoader`

- **Location:** `packages/components/nodes/documentloaders/Unstructured/Unstructured.ts:29`
- **Severity:** HIGH
- **Documented:** Returns element object
- **Runtime:** Returns `Promise<Element[]>`
- **Risk:** Models may treat it as a single element object and generate incorrect extraction/iteration logic.

---

### 35. `_HashedDocument`

- **Location:** `packages/components/src/indexing.ts:39`
- **Severity:** HIGH
- **Documented:** Returns HashedDocument
- **Runtime:** Returns `void` (method mutates instance state)
- **Risk:** LLMs may hallucinate chaining and treat the function as a pure transformer returning a new object, producing dead or broken pipelines.

---

### 36. `DynamoDBStoreTool`

- **Location:** `packages/components/nodes/tools/AWSDynamoDBKVStorage/AWSDynamoDBKVStorage.ts:53`
- **Severity:** HIGH
- **Documented:** Returns object
- **Runtime:** Returns `Promise<string>`
- **Risk:** Models may generate property access on the return value and omit `await`, breaking storage workflows.

---

### 37. `APIChain`

- **Location:** `packages/components/nodes/chains/ApiChain/postCore.ts:55`
- **Severity:** HIGH
- **Documented:** Returns `string`
- **Runtime:** Returns `Promise<ChainValues>` (async object result)
- **Risk:** LLMs may generate string operations or treat results as a serialized response rather than iterable structured data.

---

### 38. `JSONPathExtractor_Tools`

- **Location:** `packages/components/nodes/tools/JSONPathExtractor/JSONPathExtractor.ts:73`
- **Severity:** HIGH
- **Documented:** Returns Node
- **Runtime:** Returns `Promise<any>`
- **Risk:** Comment implies a stable node type, but code returns a generic promise—forcing model guessing and increasing hallucination probability.

---

### 39. `DynamoDBRetrieveTool`

- **Location:** `packages/components/nodes/tools/AWSDynamoDBKVStorage/AWSDynamoDBKVStorage.ts:100`
- **Severity:** HIGH
- **Documented:** Returns object
- **Runtime:** Returns `Promise<string>` (serialized string)
- **Risk:** Models may hallucinate an object payload and generate incorrect field access or JSON parsing.

---

### 40. `AWSDynamoDBKVStorage_Tools`

- **Location:** `packages/components/nodes/tools/AWSDynamoDBKVStorage/AWSDynamoDBKVStorage.ts:183`
- **Severity:** HIGH
- **Documented:** Returns object
- **Runtime:** Returns `Promise<INodeOptionsValue[]>`
- **Risk:** Encourages hallucinated shapes and invalid configuration usage.

---

### 42. `LangchainChatGoogleGenerativeAI`

- **Location:** `packages/components/nodes/chatmodels/ChatGoogleGenerativeAI/FlowiseChatGoogleGenerativeAI.ts:557`
- **Severity:** HIGH
- **Documented:** Returns `AIMessageChunk`
- **Runtime:** Returns `Promise<ChatResult>` (async chat result)
- **Risk:** Models may hallucinate streaming chunk handling and attempt to access chunk fields, breaking response handling logic.

---

### 43. `processNodeOutputs`

- **Location:** `packages/server/src/utils/buildAgentflow.ts:795`
- **Severity:** HIGH
- **Documented:** No explicit `@returns` documented
- **Runtime:** Async function returns `Promise<{ humanInput?: IHumanInput }>`
- **Risk:** LLMs may omit `await` and assume the function is purely side-effect–driven, causing agentflow execution ordering bugs.

---

### 44. `CustomStreamingHandler`

- **Location:** `packages/components/src/handler.ts:1833`
- **Severity:** HIGH
- **Documented:** No explicit `@returns` documented
- **Runtime:** Async method returns `Promise<void>`
- **Risk:** Underspecified docs increase guessing and incorrect stream lifecycle handling in AI-generated code.

---

## Research attribution

- Ji et al., 2023 — *A Survey of Hallucination in Large Language Models*  
  https://arxiv.org/abs/2311.05232

- Liu et al., 2024 — *CodeMirage: Hallucinations in Code Generation*  
  https://arxiv.org/abs/2408.08333


