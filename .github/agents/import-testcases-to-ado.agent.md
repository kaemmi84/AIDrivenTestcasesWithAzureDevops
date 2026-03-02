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

Import test cases from a generated Markdown file and create Azure DevOps Test Case work items.

## Input

The user must provide either:
- Work Item ID (preferred), or
- A direct Markdown file path

## Process

1. Resolve input to a Markdown file:
   - If Work Item ID is provided, load `.github/testcases/Test_Case_<WorkItemId>.md`
   - Otherwise, load the provided path
2. Parse each `TC-XX` section into:
   - Title
   - Preconditions
   - A steps table (action + expected result)
3. For each parsed test case, create an Azure DevOps **Test Case** work item:
   - Title = test case title
   - Description = preconditions (and any additional notes)
   - Steps = converted from the Markdown table (prefer the native Test Case steps field when possible)
4. Link each created Test Case to the original User Story referenced by the Work Item ID.
5. Return a summary containing the created Test Case IDs and titles.

## Constraints

- Do not modify the source User Story.
- Do not modify existing Test Case work items unless the user explicitly asks for updates.

