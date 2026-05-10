# Schemas

This directory defines the first stable data contracts for the trading Harness MVP.

The schemas are intentionally small and file-based so the system can start as a local kernel before adding a database, UI, broker integration, or real-time market adapters.

## Core Objects

- `source_document.schema.json`: raw input evidence, including historical images, human notes, trade records, and agent proposals.
- `atomic_insight.schema.json`: the smallest extracted trading experience or rule-like claim.
- `strategy_card.schema.json`: a versioned strategy asset that can be loaded by agents after review.
- `risk_rule.schema.json`: independent risk controls that override opportunity-seeking strategies.
- `context_pack.schema.json`: the compiled context passed into an agent for a specific task.
- `conversation_task.schema.json`: a structured task produced from a user chat message.
- `postmarket_review.schema.json`: daily or trade-level review output.
- `order_intent.schema.json`: a draft trading intent for future L3 workflows; it is not an executable order.
- `external_signal.schema.json`: normalized market, sentiment, news, capital, or tool observations.
- `quant_validation_result.schema.json`: historical sample, simulation, and statistical validation output for strategy conditions.

## Design Rules

- Raw source data should be referenced by id, not copied into every downstream object.
- Strategy and risk assets must carry status fields so draft or pending rules do not accidentally drive trading output.
- Real trading requests must become `order_intent` records first, then pass risk and approval gates outside the model.
- `context_pack` is the main anti-RAG mechanism: it records exactly what the system loaded for a task and which guardrails applied.
- External tools are observers and validators; their raw output must not directly trigger trading behavior.
