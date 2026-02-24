# How to Create a Test Case via AI and Save it Directly to Azure DevOps

## Introduction

This document outlines how to create a test case using a Custom Agent in Copilot and save it directly to Azure DevOps. It includes steps for understanding the application context, building feature-level domain context, integrating Azure DevOps MCP, and generating test cases.

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

4. **Use it for test case persistence**:
    - Generate a test case with your Custom Agent.
    - Save it through Azure DevOps MCP tools.
    - Validate the created or updated item in Azure DevOps.

### Step 2: Engaging the AI to Understand the Application for Test Case Creation

To create comprehensive test cases, it's essential to understand how the AI perceives the application. Use the following prompt to engage the AI effectively:

**Prompt:**

"**Analyze this repository and generate a comprehensive PROJECT_CONTEXT.md file. The goal is to create a source of truth that I can provide to other LLMs in the future to give them full context of this project. Please include:**
1. **Project Overview:** A high-level summary of what the application does and the core tech stack.
2. **Architecture & Design Patterns:** Explain the overall structure (e.g., MVC, Microservices, Hexagonal) and key patterns used.
3. **Module Breakdown:** A list of the most important directories/files and their specific responsibilities.
4. **Key Workflows:** Describe how data flows through the system for a primary use case.
5. **Entry Points:** Identify where the application starts and where the main logic resides.
6. **Development Guidelines:** Mention any specific naming conventions, state management patterns, or constraints visible in the code.

**Please format the output in clean Markdown with clear headings and concise descriptions.**"

---

This prompt is designed to extract detailed information from the AI, enabling the creation of thorough and effective test cases that address various aspects of the application.

### Step 3: Building Functional Context per Feature

Before generating test cases for individual user stories, build a concise functional context for each feature.

1. **Start from a Query**: Use an Azure DevOps query as the input source for feature selection.

![Custom query with all current features](/query_all_current_features.jpg)

**Remember the GUID from the address bar.**

2. **Create a new custom query agent**:

```yaml
name: get-domain-knowledge
description: Build feature-level domain context files from an Azure DevOps feature query.
argument-hint: To consider all features, enter 'all_features'. To consider a single feature, enter 'single_feature <FEATURE_ID>'.
tools: ['agent', 'read', 'search', 'microsoft/azure-devops-mcp/search_workitem', 'microsoft/azure-devops-mcp/wit_get_query', 'microsoft/azure-devops-mcp/wit_get_query_results_by_id', 'microsoft/azure-devops-mcp/wit_get_work_item', 'microsoft/azure-devops-mcp/wit_get_work_item_type', 'microsoft/azure-devops-mcp/wit_get_work_items_batch_by_ids']
---
Use query <QUERY_ID> to retrieve features.
Mode:
- `all_features`: Create one context Markdown file for every feature in the query result.
- `single_feature`: Create one context Markdown file only for feature ID <FEATURE_ID> from the same query result.

For each selected feature, collect the feature description, all linked user stories, and all acceptance criteria. Create a `domaincontext/FEATURE_<FEATURE_TITLE>_CONTEXT.md` document containing:
1) Feature summary,
2) A concise but complete overall functional context.
Do not modify any Azure DevOps work item. Return only the generated Markdown content.
```

3. Replace `<QUERY_ID>` with the GUID from the query.

4. **Run the custom agent**:
- Open GitHub Copilot
- Choose the custom agent `get-domain-knowledge`
- Enter `all_features` to generate files for all features, or `single_feature <FEATURE_ID>` for one feature

### Step 4: Setting Up the Custom Agent

To create a Custom Agent in Copilot, follow these steps:

1. **Access Copilot**: Log into Copilot and navigate to the agent settings.
2. **Create a New Agent**: Select the option to create a new Custom Agent.
3. **Configure the Agent**: Enter the required parameters and configurations for the agent, including specific agents for test cases.

```
tools: ['vscode', 'execute', 'read', 'edit', 'search', 'web', 'agent', 
'microsoft/azure-devops-mcp/search_workitem', 'microsoft/azure-devops-mcp/testplan_add_test_cases_to_suite', 
'microsoft/azure-devops-mcp/testplan_list_test_cases', 'microsoft/azure-devops-mcp/testplan_list_test_plans', 
'microsoft/azure-devops-mcp/testplan_list_test_suites', 'microsoft/azure-devops-mcp/testplan_show_test_results_from_build_id', 
'microsoft/azure-devops-mcp/testplan_update_test_case_steps', 'microsoft/azure-devops-mcp/wit_add_artifact_link', 
'microsoft/azure-devops-mcp/wit_add_work_item_comment', 'microsoft/azure-devops-mcp/wit_get_query', 
'microsoft/azure-devops-mcp/wit_get_query_results_by_id', 'microsoft/azure-devops-mcp/wit_get_work_item', 
'microsoft/azure-devops-mcp/wit_get_work_item_type', 'microsoft/azure-devops-mcp/wit_get_work_items_batch_by_ids', 
'microsoft/azure-devops-mcp/wit_get_work_items_for_iteration', 'microsoft/azure-devops-mcp/wit_update_work_item', 
'microsoft/azure-devops-mcp/wit_update_work_items_batch', 'microsoft/azure-devops-mcp/wit_work_item_unlink', 
'microsoft/azure-devops-mcp/wit_work_items_link', 'todo']
```

4. **Save and Activate**: Save the agent and activate it for test case generation.

### Step 5: Generating a Description of the Test Case Agent

Once the Custom Agent is created, you can generate a description of the test case agent:

1. **Create Description**: Use the Custom Agent to generate a detailed description of the test case. This should include:
    - The acceptance criteria from the user story.
    - Any specific scenarios or conditions that need to be tested based on the user story.
    - The structure of the test case, including Test Case ID, Title, Description, Preconditions, Test Steps, Expected Results, and Postconditions.
2. **Include User Story**: Ensure that the dependent user story is incorporated into the test case description. This will help in aligning the test case with the specific requirements and expectations outlined in the user story.
3. **Review**: Check the generated description for completeness and accuracy.
4. **Adjustments**: Make adjustments as needed to ensure the description meets the requirements.

### Step 6: Generating a Test Case for a Specific User Story

To generate a test case based on a specific user story, use the following prompt. This prompt assumes the Custom Agent has been selected within the Copilot application and will fetch the story details based on the user story number.

**Prompt:**

"I would like to generate a test case for the following user story:

**User Story Number**: [Insert user story number here]

Using the capabilities of the Custom Agent, please generate a test case that aligns with this user story. Ensure the test case is comprehensive and includes all necessary elements as defined by the Custom Agent.

Please provide the generated test case in a structured format, ensuring clarity and comprehensiveness. Once completed, share the test case for review."

---

This prompt is designed for use after selecting the Custom Agent within Copilot, focusing on the AI's task of generating a structured test case based on the user story number provided.

## Conclusion

By utilizing a Custom Agent in Copilot and integrating with Azure DevOps MCP, test cases can be efficiently generated and saved. This saves time and improves the accuracy of test case management.
