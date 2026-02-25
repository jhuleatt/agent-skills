# Friction Log: Firebase Functions Basics

This log details potential issues an LLM (or a developer) might encounter while following the instructions in `skills/firebase-functions-basics/SKILL.md` and its references.

## 1. Ambiguity in Generation 2 Imports (Node.js)

The guide states to "Use top-level imports (e.g., `firebase-functions/https`). These are 2nd gen by default."

**Issue:**
- Traditionally, `firebase-functions/v2` is the explicit namespace for 2nd-gen functions. Using `firebase-functions/https` often implies 1st-gen functions unless the SDK behavior has changed significantly in version 6.0.0 (which is not standard practice across most documentation).
- An LLM trained on slightly older data will likely generate v1 code if instructed to use `firebase-functions/https`.
- Mixing up v1 and v2 imports leads to runtime errors or unexpected behavior (e.g., cold starts, concurrency settings not applying).

**Recommendation:**
- Explicitly recommend importing from `firebase-functions/v2/https`, `firebase-functions/v2/firestore`, etc.
- Clarify that `firebase-functions/v1` is for 1st-gen functions.

## 2. Incorrect Function Signatures in `node_setup.md`

The example code in `node_setup.md` uses a potentially incorrect signature for `onDocumentCreated`.

```typescript
export const newDoc = onDocumentCreated(
  { maxInstances: scaleLimit },
  "/words/{wordId}",
  async (event) => { ... }
);
```

**Issue:**
- In the `firebase-functions/v2/firestore` API, `onDocumentCreated` typically accepts either a document path string as the first argument, or an options object that *includes* the `document` property.
- Passing options as the first argument and the path as the second is not a standard signature for v2 functions.
- This will cause TypeScript compilation errors or runtime failures.

**Recommendation:**
- Update the example to use the options object correctly:
  ```typescript
  onDocumentCreated({ document: "/words/{wordId}", maxInstances: scaleLimit }, async (event) => { ... })
  ```

## 3. Confusing Initialization Instructions

The `SKILL.md` suggests using `onInit` for `initializeApp`:

```typescript
import { onInit } from "firebase-functions";
onInit(() => { initializeApp(); });
```

**Issue:**
- While `onInit` exists in v2, the standard and most robust way to initialize `firebase-admin` is at the top level of the file to ensure it's ready before any function handlers execute.
- Using `onInit` might be correct for certain advanced use cases, but for basics, it introduces unnecessary complexity and potential timing issues if not understood perfectly.
- The text also says "This should be done once at the top level of your `index.ts` file," which contradicts the code snippet using `onInit`.

**Recommendation:**
- Simplify to standard top-level initialization:
  ```typescript
  import { initializeApp } from "firebase-admin/app";
  initializeApp();
  ```

## 4. Parameter Access in Python

The Python example accesses `.value` at the module level:
```python
SCALE_LIMIT = params.IntParam("MAX_INSTANCES", default=1).value
```

**Issue:**
- While valid in many contexts, accessing `.value` at module scope relies on the environment being fully populated before the module is imported. In Cloud Functions this is generally true, but it can be a source of confusion if testing locally without emulators or in different environments.
- An LLM might hallucinate that `.value` is only available inside a function body.

**Recommendation:**
- Clarify that parameter values are resolved at deployment/runtime initialization.
- Ensure the example works in the emulator.

## 5. Secret Management

The guide mentions `defineSecret` but the example uses `functions.config` in the "Secrets Management" section context (referring to legacy code).

**Issue:**
- The distinction between `defineSecret` (v2/modern) and `functions.config` (legacy) needs to be very sharp to prevent LLMs from hallucinating legacy config usage for secrets.

**Recommendation:**
- Reinforce `defineSecret` usage in v2 examples.
