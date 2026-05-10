# Data Directory

This directory stores the structured state for the trading Harness system.

The current project is in the system-design and MVP-kernel phase. These folders are intentionally data-oriented rather than UI-oriented, so the system can first prove the strategy, context-loading, review, and audit loops.

## Directory Layout

```text
data/
  raw/
    daily_review_files/
    human_notes/
    trade_records/
    screenshots/
    external_market_data/
    external_sentiment/
    external_capital/
  extracted/
    source_documents/
    note_sections/
    atomic_insights/
    case_references/
    strategy_update_requests/
    external_signals/
    quant_validation_results/
  assets/
    terms/
    principles/
    market_regime_rules/
    strategy_cards/
    risk_rules/
    failure_patterns/
  runtime/
    conversations/
    tasks/
    market_snapshots/
    context_packs/
    premarket_plans/
    intraday_observations/
    postmarket_reviews/
    strategy_update_queue/
    order_intents/
    approvals/
    execution_reports/
    audit_logs/
  strategy_versions/
    changelog/
    proposals/
  controls/
    risk_limits/
    kill_switch/
    permission_profiles/
  adapters/
    market_data/
    sentiment/
    quant/
    execution/
```

## Rules

- Raw inputs are evidence sources and must not be overwritten.
- Agent decisions must use reviewed assets and compiled context packs, not raw notes directly.
- External market, sentiment, quant, and capital-flow outputs must be normalized into structured signals before use.
- Strategy changes must enter the update queue before becoming active.
- Trading-related requests must be recorded as `order_intent` or research output until the system is explicitly promoted beyond MVP.
- Runtime outputs should be append-only whenever possible, so conversations, reviews, approvals, and audit logs remain traceable.
