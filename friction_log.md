# Friction Log for firebase-functions-basics

This document outlines potential friction points an LLM might encounter when following the instructions in `skills/firebase-functions-basics/SKILL.md` and its associated reference files.

## 1. Setup Instructions Conflict
- **Issue**: `SKILL.md` instructs the user to "Initialize in your code" (step 1) by adding code to `index.ts`. However, step 2 refers to `references/node_setup.md`, which explicitly instructs the user to "Replace the contents of `src/index.ts`" with a provided code block.
- **Result**: If an LLM follows step 1 and modifies `index.ts`, step 2 will completely overwrite those changes, removing the `firebase-admin` initialization.
- **Impact**: The user ends up with a function that cannot interact with other Firebase services (like Firestore or Auth) despite following the "Provisioning & Setup" instructions.

## 2. Inconsistent Admin SDK Setup
- **Issue**: The `references/python_setup.md` guide includes `firebase_admin.initialize_app()` in its code example, but `references/node_setup.md` does not.
- **Result**: Python users get a complete setup out-of-the-box, while Node.js users are left with a function that only handles HTTP requests but lacks the initialization required for other Firebase interactions (as promised in `SKILL.md`).
- **Impact**: Confusion for Node.js users and potential runtime errors if they try to add Firestore logic later without re-adding the initialization code.

## 3. Ambiguity in Initialization Placement (Node.js)
- **Issue**: `SKILL.md` uses `onInit` for initialization, which is a specific V2 feature. However, many existing resources and even some V2 documentation show top-level `initializeApp()`.
- **Result**: An LLM might generate code using the older top-level pattern if not strictly guided, potentially leading to inconsistent behavior or warnings in future V2 versions.
- **Impact**: Mixing patterns can lead to confusion about best practices for V2 functions.

## 4. Interactive Commands vs Automation
- **Issue**: The deployment section mentions that the CLI will prompt for secret values.
- **Result**: An LLM cannot interact with CLI prompts. If a secret is required but not set, the deployment will hang or fail.
- **Impact**: The process is not fully automated for an agent. The guide mentions `firebase functions:secrets:set` as an alternative, but it should be emphasized as the *primary* method for agents.

## 5. Dependency Management
- **Issue**: `node_setup.md` assumes `firebase-admin` is installed because `SKILL.md` mentioned it. However, `node_setup.md` doesn't explicitly list it in `package.json` creation steps (it relies on `npm install` working with an existing `package.json`).
- **Result**: If the user skips `SKILL.md` step 1 or starts directly from `node_setup.md`, they will miss the `firebase-admin` dependency.
- **Impact**: Build errors when trying to run the code if dependencies are missing.

## 6. Long-Running Processes
- **Issue**: The command `firebase emulators:start` is long-running.
- **Result**: An LLM might run this and wait indefinitely for it to finish.
- **Impact**: The agent hangs. The guide mentions "A human should run this command", which is good, but an agent might still attempt it if not careful.
