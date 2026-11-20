---
name: newo-develop
description: Develop, review or migrate Newo agent code, event subscriptions and reusable Python WASM skills. Use for runtime actions, SDK usage, project drafts and code regression tests.
---

# Develop Newo agent code

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md) and the [development guide](../../references/agent-development.md). Use the [Python action reference](../../references/python-actions.md) to check signatures, return types and runtime requirements. Use [newo-configure](../newo-configure/SKILL.md) for attribute-only changes and [newo-external-mcp](../newo-external-mcp/SKILL.md) for connector OAuth2 setup.

1. Identify the requested outcome and the source or customer version. Read the active files for a customer change. Trace the subscription, flow, registered skill, Python imports and downstream side effects. Resolve the actual event, integration, connector, persona scope and actor context before changing routing.
2. Write new code in Python using `runner_type: python_wasm`. Use NSL only when repairing existing NSL or when the user chooses it for migration of existing Guidance. When a requested change affects Guidance, warn the user and offer a rewrite in Python or NSL. Apply the requested change after migrating the affected code. Never write or edit Guidance.
3. Use typed `newo.*` actions from Python. Keep regular helper functions in project modules. Give host entry points a synchronous `main()` and checked `newo.arguments`. Preserve payload names, event IDNs, context, model settings, outputs and side effects. For a migration, verify flow-state defaults, data scope and backfill before removing old fields.
4. Compile and validate the changed files, imports and registered paths. For an authorized customer edit, fork the accessible version, submit complete changed files to the draft and read them back. Follow the [version procedure](../../references/versions.md) for publication, activation and rollback. Record the original activation and registry mode before switching. Keep activation within the requested scope.
5. For authorized runtime verification, use an identified write-safe test customer, a fresh sandbox session or the exact test actor event. Use [newo-quality](../newo-quality/SKILL.md) to correlate the event, skill and downstream result. Verify a positive execution marker before treating absence of an effect as success. Report static checks, runtime evidence and any unverified paths separately.
