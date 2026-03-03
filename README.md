# How to Create a Test Case via AI and Save it Directly to Azure DevOps

## Introduction

This document outlines how to create a test case using a Custom Agent in Copilot and save it directly to Azure DevOps. It includes steps for understanding the application context, building feature-level domain context, integrating Azure DevOps MCP, and generating test cases.

The custom agents used in this guide are included in this repository under `.github/agents/`:

- [`ado-mcp-connection-check`](.github/agents/ado-mcp-connection-check.agent.md)
- [`get-domain-knowledge`](.github/agents/get-domain-knowledge.agent.md)
- [`retrieve-domain-context`](.github/agents/retrieve-domain-context.agent.md)
- [`generate-testcases-from-user-story`](.github/agents/generate-testcases-from-user-story.agent.md) (includes a handoff to the import agent)
- [`import-testcases-to-ado`](.github/agents/import-testcases-to-ado.agent.md)

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

Use the agent definition from:
- [`get-domain-knowledge`](.github/agents/get-domain-knowledge.agent.md)

3. Replace `<QUERY_ID>` with the GUID from the query.

4. **Run the custom agent**:
- Open GitHub Copilot
- Choose the custom agent `get-domain-knowledge`
- Enter `all_features` to generate files for all features, or `single_feature <FEATURE_ID>` for one feature

#### Recommended structure (for better RAG)

For higher-quality retrieval later, ensure each generated Feature context uses stable headings (rules, workflows/states, validations, permissions). The `get-domain-knowledge` agent definition includes a recommended template.

### Step 3: Create a Test Case from an existing User Story

1. **Create a new custom agent**:

Use the agent definition from:
- [`generate-testcases-from-user-story`](.github/agents/generate-testcases-from-user-story.agent.md)

This agent performs RAG-style retrieval by reading the User Story from Azure DevOps and (if available) loading additional relevant context from `.github/domain_knowledge/` before generating consolidated test cases.

To keep retrieval consistent and maintainable, a dedicated retrieval-only agent is included:
- [`retrieve-domain-context`](.github/agents/retrieve-domain-context.agent.md)

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

Use the agent definition from:
- [`import-testcases-to-ado`](.github/agents/import-testcases-to-ado.agent.md)

3. **Run the agent**
- In Copilot Chat, run `import-testcases-to-ado`
- Provide the Work Item ID

4. **Validate results**
- Confirm new Test Case items exist in Azure DevOps
- Verify each test case is linked to the original User Story
