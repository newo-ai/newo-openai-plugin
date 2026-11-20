# AKB, imports and answer diagnostics

Follow [operations.md](operations.md) and use Customer MCP. Short business facts usually belong to customer attributes. Knowledge topics belong to a specific AKB agent.

## Choose the source and agent

Check `project_attributes_setting_knowledge_source`. Depending on the deployment, values can include `akb`, empty, `akb_studio`, `external` and `akb_external_shadow`. If unavailable or unclear, verify the source using accessible project files, retrieval logs and a new answer. Changes to standard AKB affect that knowledge base.

AKB requires `customer_idn`, `project_idn` and `agent_idn`. Locate the project with `list_projects` and identify the version using [versions.md](versions.md). Inspect `agents/` through `read_project_version_files` to confirm the exact agent IDN before changing topics.

## List and search topics

`list_knowledge_base_topics` returns names, summaries, facts, sources and labels newest first. It has no text query. Pages start at 1 and `per` defaults to 20 with a maximum of 100. Reduce `per` on truncated summaries or facts. Report any remaining limit. Label search matches case-insensitive substrings, with multiple terms required within one label.

`search_knowledge_base_topics` returns best matches and per-field `fields_score`. It supports `search_fields`, a threshold from 0 to 1, `topics_number` from 1 to 100 with default 5, and labels. All labels must match exactly.

To reproduce `SearchFuzzyAkb`, use the actual arguments from permitted logs and acts. MCP searches four fields and up to five topics by default. The agent action usually searches facts and takes one result. MCP searches agent topics only. The agent action can include caller topics unless limited by `fromPerson Agent`. MCP scores establish the result for that request. Use agent evidence for the actual retrieval and final answer.

## Add or replace a topic

`create_knowledge_base_topic` requires `topic_summary` and an array of single-line `topic_facts`. It accepts optional `topic_name`, `source` and `labels`. The name defaults to the first fact. Save the returned topic ID and verify it by reading or searching. AKB can retrieve the topic from the next conversation without a separate publication.

A matching name creates a second topic. There is no in-place update. For a requested correction, read the original fully. Prepare and verify the replacement, then delete the explicitly agreed original ID with `delete_knowledge_base_topic`. These writes can succeed independently. Creating first keeps the old content available until verification but can briefly duplicate it. Deleting first requires authorization and an acceptable gap. Verify each step. Deletion is permanent. After an uncertain creation, search for the topic before retrying.

## Import text and pages through existing attributes

Standard deployments may include the following customer attributes.

- `project_attributes_rag_knowledge_base_append_text` and `project_attributes_rag_knowledge_base_append_text_log`
- `project_attributes_rag_knowledge_base_append_websites` and `project_attributes_rag_knowledge_base_append_websites_log`

Use `search_customer_attributes` to find these fields and read them in full. Leave pending batches available for processing. Submit approved material using `set_customer_attribute_value`. For pages, use the user-supplied address and supported Website, Objective and Keywords format. The server retrieves the pages. Verify extraction support for files or photos before claiming import.

Check the new log entry for the batch, the resulting topics and a fresh answer. Clearing the queue alone does not establish success. Check an uncertain batch before resubmitting. Restoring an old nonempty value can re-run import. Avoid reset fields.

## Diagnose an incorrect answer

Find the session and question with `list_sessions` or `get_session`. Use `search_logs` for technical causes and `search_conversations` for events. Compare business attributes, knowledge source, topics and fuzzy results. Inspect a failing connector through `get_connectors` using both IDNs without exposing secrets.

Fix the verified fact and repeat the question in a new sandbox conversation using [testing.md](testing.md). Studio builds, external index publication and index version history or recovery are outside the catalog. See [missing-tools.md](missing-tools.md). Respect access restrictions on conversations.

Customer MCP does not support Knowledge Studio, and the plugin uses one server. For `knowledge_source=akb_studio`, diagnose accessible settings and logs. Standard AKB changes update standard AKB. Agent AKB tools remain available for customers with BAA. Caller data restrictions still apply.
