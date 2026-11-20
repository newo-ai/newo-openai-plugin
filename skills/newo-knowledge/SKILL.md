---
name: newo-knowledge
description: Read, search, create and delete Newo agent AKB topics, and import text and pages through supported attributes. Use for FAQs, outdated facts and retrieval diagnostics using fields_score and agent responses.
---

# AKB topics and agent answers

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md), [knowledge guide](../../references/knowledge.md) and [customer attributes](../../references/configuration.md) for business facts.

1. Identify the knowledge source and exact `customer_idn`, `project_idn` and `agent_idn` using the [source and agent procedure](../../references/knowledge.md#choose-the-source-and-agent). For registry projects, check the version using [versions.md](../../references/versions.md).
2. `list_knowledge_base_topics` lists topics newest first and accepts no text query. Reduce `per` if summaries or facts are truncated. For semantic search, use `search_knowledge_base_topics` and inspect actual `fields_score`. Label filters differ between these tools.
3. `create_knowledge_base_topic` accepts a summary and an array of single-line facts. Creating a topic with the same name leaves the old one in place. To replace a topic, create the new one and delete the exact old topic with `delete_knowledge_base_topic` when authorized. Verify both operations. They are separate writes. The next conversation can retrieve the new topic without publication.
4. Update short facts through customer attributes. Import text or pages through existing append fields and check logs. Append adds content, so confirm the outcome before retrying. Reverting an attribute value will not undo the import. An import does not update a different active knowledge source.
5. For an incorrect answer, compare the session, logs and AKB search using the actual `SearchFuzzyAkb` arguments. MCP defaults to comparing four fields and returning up to five topics. The agent action usually compares facts and takes one result. MCP searches agent topics only, so it cannot reproduce a search that also used caller topics.
6. Test a new answer and nearby cases. Knowledge Studio, external index publication and index version recovery are outside this catalog. When replacing a topic, choose the order of the separate create and delete calls based on data retention and whether temporary duplication is acceptable. Agent AKB tools remain available with a signed BAA. Restrictions on caller data still apply.
