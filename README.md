# How to Create a Test Case via AI and Save it Directly to Azure DevOps

## Introduction

This document outlines how to create a test case using a Custom Agent in Copilot and save it directly to Azure DevOps. It includes steps for understanding the application context, building feature-level domain context, integrating Azure DevOps MCP, and generating test cases.

The custom agents used in this guide are included in this repository under `.github/agents/`:

- `.github/agents/ado-mcp-connection-check.agent.md`
- `.github/agents/get-domain-knowledge.agent.md`
- `.github/agents/generate-testcases-from-user-story.agent.md` (includes a handoff to the import agent)
- `.github/agents/import-testcases-to-ado.agent.md`

## Steps

### Step 1: Integrating Azure DevOps MCP

To save the generated test case in Azure DevOps, integrate Azure DevOps MCP:

1. **Preparation**:
    - Open VS Code and sign in to GitHub Copilot.
    - Ensure Azure DevOps access is available in your environment (organization, project, and permissions).

2. **MCP setup**:
    - Open your Copilot/MCP configuration in VS Code.
    - Add or select the Azure DevOps MCP server.
    - Provide the required organization/project context and credentials.

3. **Verify the connection**:
    - In Copilot Chat, run a simple read action (for example, fetch a known work item).
    - Confirm that Azure DevOps data is returned without errors.
    - Alternatively, run the custom agent `ado-mcp-connection-check` from `.github/agents/ado-mcp-connection-check.agent.md`.

4. **Use it for test case persistence**:
    - Generate a test case with your Custom Agent.
    - Save it through Azure DevOps MCP tools.
    - Validate the created or updated item in Azure DevOps.

### Step 2: Building Domain Knowledge per Feature

Before generating test cases for individual user stories, build a concise functional context for each feature.

This step is the "retrieval" part of a lightweight RAG workflow: it creates a curated knowledge base under `.github/domain_knowledge/` that other agents can selectively load later.

1. **Start from a Query**: Use an Azure DevOps query as the input source for feature selection.

![Custom query with all current features](/query_all_current_features.jpg)

**Remember the GUID from the address bar.**

2. **Create a new custom agent**:

The full agent definition is also available at `.github/agents/get-domain-knowledge.agent.md`.

````yaml
---
name: get-domain-knowledge
description: Build feature-level domain context files from an Azure DevOps feature query.
argument-hint: "Provide: <QUERY_ID> and a mode: `all_features` or `single_feature <FEATURE_ID>`."
tools: [ 'read', 'edit', 'search', 'agent', 'microsoft/azure-devops-mcp/search_workitem', 'microsoft/azure-devops-mcp/wit_get_query', 'microsoft/azure-devops-mcp/wit_get_query_results_by_id', 'microsoft/azure-devops-mcp/wit_get_work_item', 'microsoft/azure-devops-mcp/wit_get_work_item_type','microsoft/azure-devops-mcp/wit_get_work_items_batch_by_ids' ]
---

   Use an Azure DevOps query to retrieve Features and generate one feature-context file per selected Feature.

This agent is the "retrieval" part of a simple RAG workflow: it materializes a curated, searchable knowledge base in `.github/domain_knowledge/` so other agents can load only the relevant context later.

## Input

The user must provide:
   - Query ID (GUID)
   - Mode:
        - `all_features`
        - `single_feature <FEATURE_ID>`

## Process

   1. Use the provided Query ID to retrieve the list of Features.
2. Mode handling:
   - `all_features`: generate context for every Feature returned by the query.
   - `single_feature <FEATURE_ID>`: generate context only for the provided Feature ID (must be part of the query result).
3. For each selected Feature, collect:
   - Feature title and description
   - All linked User Stories
   - Acceptance criteria for each User Story (if available)
4. Create one Markdown file per Feature at:
   - `.github/domain_knowledge/FEATURE_<FEATURE_TITLE>_CONTEXT.md`
5. Each Feature context file must include:
   1) Metadata (Feature ID, title, query ID, generation date)
   2) Feature summary
   3) A concise but complete functional context (what the feature does, key rules, important states)
   4) Linked User Stories (IDs + titles) with acceptance criteria
   5) Keywords (short list) to improve repository search

   ## Output

   - Write the generated files to disk.
   - Return a short summary of which files were created and which work items were used.

   ## Constraints

   - Do not modify any Azure DevOps work item.
   - Do not create any Azure DevOps work item.

````

3. Replace `<QUERY_ID>` with the GUID from the query.

4. **Run the custom agent**:
- Open GitHub Copilot
- Choose the custom agent `get-domain-knowledge`
- Enter `all_features` to generate files for all features, or `single_feature <FEATURE_ID>` for one feature

### Step 3: Create a Test Case from an existing User Story

1. **Create a new custom agent**:

The full agent definition is also available at `.github/agents/generate-testcases-from-user-story.agent.md` and includes a handoff to the import agent.

This agent performs RAG-style retrieval by reading the User Story from Azure DevOps and (if available) loading additional relevant context from `.github/domain_knowledge/` before generating consolidated test cases.

````yaml
---
name: generate-testcases-from-user-story
description: Generate consolidated test cases from an Azure DevOps User Story and write them to a Markdown file.
argument-hint: "Provide a Work Item ID (User Story). Project name is optional (defaults to FABRIS)."
tools: ['read', 'edit', 'search', 'agent', 'microsoft/azure-devops-mcp/wit_get_work_item', 'microsoft/azure-devops-mcp/wit_get_work_item_type', 'microsoft/azure-devops-mcp/wit_get_work_items_batch_by_ids','microsoft/azure-devops-mcp/wit_get_work_items_for_iteration' ]
handoffs:
   - label: Import test cases to Azure DevOps
     agent: import-testcases-to-ado
     prompt: "Import the generated test cases for the same Work Item ID into Azure DevOps as Test Case work items and link them to the User Story."
     send: false
---

   You are a software QA assistant specialized in generating consolidated test cases from Azure DevOps User Stories and their acceptance criteria (ACs).

## Core Principles

### Quality Guidelines
               - Test cases must be written in English.
               - Use clear, unambiguous language.
               - Each step must be actionable and have an expected result.
               - Expected results must be verifiable.
               - Consolidation must not sacrifice clarity or maintainability.
               - Aim for 3-6 comprehensive test cases for typical user stories (complex stories may need 6-10, still consolidated).

### Optimization Strategy
               - Minimize the number of test cases while maximizing coverage.
               - Consolidate related scenarios into workflow tests.
               - Combine positive and negative paths when it stays readable.
               - Prefer complete user journeys over isolated micro-tests.

## Behavior

### Step 1: Collect Input
   Wait for a Work Item ID (User Story). If none is provided, ask for it.

### Step 2: Retrieve User Story Data (Primary Retrieval)
               Use Azure DevOps MCP tools to read the User Story work item and extract:
                  - `System.Title`
                  - `System.Description`
                  - `Microsoft.VSTS.Common.AcceptanceCriteria`
                  - `Custom.AdditionalInformation` (if available)

### Step 3: Retrieve-Augmented Context (RAG)
   Augment the User Story with only the most relevant supporting context.

               1) Try to locate a matching domain context file in the repository:
                  - Use repository search for:
                       - the User Story ID
                       - the User Story title keywords
                       - the parent Feature ID or title (if available from links)
                  - If a match is found under `.github/domain_knowledge/`, read it and extract only the sections relevant to this User Story.

   2) If a parent Feature can be determined from the User Story relations, retrieve the Feature work item and use it as additional context (title, description, and any key rules).

   3) If the story references other work items (bugs/specs/tasks), retrieve only those that materially affect test behavior (rules, validations, permissions, error handling).

   If no additional context is available, continue with the User Story data alone and explicitly note this in the output.

        ### Step 4: Analyze and Consolidate
   1. Identify all acceptance criteria and variations.
   2. Group scenarios that can be tested together.
   3. Map end-to-end workflows.
   4. Integrate edge cases into main workflows where reasonable.
   5. Determine the minimum number of test cases that still cover everything.

### Step 5: Generate Test Cases
               Each test case must follow this structure:

#### TC-XX: [Title]

               **Preconditions**:
                  - <Precondition 1>
                  - <Precondition 2>

        **Test Steps/Expected Results**:
   | step | action | expected result |
   |---|---|---|
   | 1 | **Precondition** | <Description> |
   | 2 | <Action> | <Expected result> |

### Step 6: Save Output to File
               After generating all test cases, save the complete output as a Markdown file:

                  - File path: `.github/testcases/Test_Case_<WorkItemId>.md`
                  - Header:
                       - `# Test Cases for Work Item <WorkItemId>: <Work Item Title>`
                       - `Generated: <current date>`
                  - Create the `.github/testcases/` directory if it does not exist.

                  ## Constraints

                  - Do not create or update any Azure DevOps work item.
                  - Only read Azure DevOps work items and write the Markdown file locally.

````

2. **Run the custom agent**
- In Copilot Chat, run `generate-testcases-from-user-story`
- Provide the User Story Work Item ID (e.g., `1425771`)

3. **Check the output**
- The agent writes: `.github/testcases/Test_Case_<WorkItemId>.md`
- Optional: use the `Import test cases to Azure DevOps` handoff to continue with Step 4
- Optional: add `.github/testcases/` to `.gitignore` if you do not want to commit generated files

### Step 4: Create Test Cases from Markdown File to Azure DevOps

After test cases are generated as Markdown files, import them into Azure DevOps as Test Case work items.

1. **Prepare the input**
- Ensure the file exists at `.github/testcases/Test_Case_<WorkItemId>.md`
- Confirm the file contains all `TC-XX` sections with preconditions and steps

2. **Create a custom agent**

The full agent definition is also available at `.github/agents/import-testcases-to-ado.agent.md`.

```yaml
---
name: import-testcases-to-ado
description: Import test cases from a Markdown file and create Azure DevOps Test Case work items.
argument-hint: "Provide a Work Item ID (e.g., 1425771) or a Markdown file path."
tools:
  - read
  - search
  - agent
  - microsoft/azure-devops-mcp/wit_create_work_item
  - microsoft/azure-devops-mcp/wit_get_work_item
  - microsoft/azure-devops-mcp/wit_get_work_item_type
---
```

3. **Run the agent**
- In Copilot Chat, run `import-testcases-to-ado`
- Provide the Work Item ID

4. **Validate results**
- Confirm new Test Case items exist in Azure DevOps
- Verify each test case is linked to the original User Story
