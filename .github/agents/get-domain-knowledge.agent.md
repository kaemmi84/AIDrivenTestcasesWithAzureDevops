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
