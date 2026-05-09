# prompts
Test prompts

When a JAR version upgrade breaks your code, it is usually due to **Breaking Changes** (deprecated methods, removed classes) or **Dependency Hell** (transitive dependency conflicts).
Using an LLM to fix this is highly effective because LLMs have been trained on vast amounts of library documentation and GitHub migration guides. Here is the best step-by-step strategy to automate the fix.
## Phase 1: Diagnostic Extraction
Before the LLM can fix the code, it needs to understand *why* it broke.
 1. **Capture the Build Log:** Run your build (e.g., mvn clean compile) and pipe the output to a text file.
 2. **Isolate the Symbols:** Identify the specific classes or methods that are "missing" or throwing errors.
 3. **Dependency Tree:** Generate a dependency tree (e.g., mvn dependency:tree) to see if a sub-dependency is conflicting with your new version.
## Phase 2: The LLM "Migration Agent" Strategy
Don't just ask the LLM "how to fix this." Use a **Context-Aware Prompting** strategy.
### 1. The "Code + Documentation" Prompt
Feed the LLM the old code and the error message, but also mention the library's migration guide if you have it.
> **Prompt Template:**
> "I upgraded library $X from version $A to $B. The following code is now breaking with error: $ERROR_MESSAGE. Looking at the migration guide for version $B, $METHOD_Y was removed. Please refactor the code below to use the new $METHOD_Z or the recommended alternative while maintaining the same logic."
> 
### 2. The "Bridge" Pattern Generation
If the new JAR version completely changes the API architecture, ask the LLM to write a **Wrapper** or **Adapter**.
 * This allows you to keep your business logic untouched while the LLM writes a translation layer that communicates with the new, breaking JAR version.
## Phase 3: Automated Remediation Loop
If you are building a tool to do this, implement a **feedback loop**:
 1. **Step A:** Send the broken code + error log to the LLM.
 2. **Step B:** Apply the LLM’s suggested fix to a temporary branch.
 3. **Step C:** Trigger a build.
   * **If it fails:** Pipe the *new* error back to the LLM (it will now have more context).
   * **If it passes:** Move to Step D.
 4. **Step D:** Run Unit Tests. A successful compile doesn't mean the logic is still correct. The LLM might have swapped a method for one that behaves slightly differently.
## Phase 4: Dealing with Transitive Conflicts (The "Exclusion" Strategy)
Sometimes the "break" isn't in your code, but in a conflict between two JARs.
 * **Strategy:** Provide the LLM with your pom.xml and the dependency:tree output.
 * **The Request:** "Identify the conflict causing the NoSuchMethodError. Suggest the correct <exclusion> tags for my pom.xml to ensure only version $X of the library is loaded."
## Summary of LLM Roles in JAR Upgrades
| Task | LLM Technique |
|---|---|
| **Deprecated Methods** | Direct Refactoring (Search and Replace with context) |
| **Removed Classes** | Architectural Mapping (Mapping old logic to new classes) |
| **Dependency Conflicts** | Config Analysis (Generating pom.xml exclusions) |
| **Behavioral Changes** | Test-Driven Repair (Fixing code based on failing unit tests) |
### Pro-Tip: Use "Diff" Mode
When using an LLM tool, ask it to output the fix in **Unified Diff format**. This makes it much easier for your automated tool (or a human reviewer in GitLab) to see exactly what changed without re-reading the entire file.
Would you like a sample Python script that parses a Maven error log and sends the relevant code snippets to an LLM for fixing?
