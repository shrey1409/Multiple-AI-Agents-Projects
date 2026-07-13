# RESOURCES.md — Agent Observability Stack

Paths verified 2026-07-13 (HTTP 200 or present in clone). ✅ = confirmed.

## 1. What to clone / read

### OpenTelemetry GenAI semantic conventions (the schema you follow)
```bash
git clone --depth 1 https://github.com/open-telemetry/semantic-conventions.git /tmp/ref-otel-semconv
ls /tmp/ref-otel-semconv/docs/gen-ai/   # ✅ gen-ai conventions live here
```
`docs/gen-ai/README.md` ✅ (200) — the authoritative `gen_ai.*` attribute definitions. This is the single most important reference: follow these names exactly (`gen_ai.request.model`, `gen_ai.usage.input_tokens/output_tokens`, `gen_ai.tool.name`, `gen_ai.operation.name`).

### OpenTelemetry Python SDK
```bash
git clone --depth 1 https://github.com/open-telemetry/opentelemetry-python.git /tmp/ref-otel-python   # ✅
```
The SDK you instrument with (tracer, spans, OTLP exporter). For auto-instrumentation helpers see `open-telemetry/opentelemetry-python-contrib` (repo ✅; its README is `.rst`, browse the repo root).

### OTel Collector (export target)
- Run the collector via its official Docker image (`otel/opentelemetry-collector`); configure an OTLP receiver + your chosen backend exporter. For a self-contained portfolio build, a SQLite/file exporter or Jaeger all-in-one Docker image is enough.

### "Buy" alternatives to read (README-level, for the product-awareness note)
```bash
# Read-only — study how mature products model agent traces, then justify your build-vs-buy in the README.
# Langfuse:  https://github.com/langfuse/langfuse   ✅
# Arize Phoenix: https://github.com/Arize-ai/phoenix ✅
```

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| OTel GenAI conventions | `semantic-conventions/docs/gen-ai/` | Reuse — attribute schema | 1, 2 |
| opentelemetry-python | repo | Reuse — instrumentation SDK | 1, 2, 3 |
| OTel Collector (Docker) | `otel/opentelemetry-collector` image | Reuse — export target | 0 |
| Jaeger/Tempo or SQLite exporter | official images | Reuse — trace store + viewer | 4 |
| Langfuse / Phoenix | repos | Read-only — build-vs-buy framing | 6 |

## 3. Instrumented agents (internal)
- Project 01 (primary) and Project 02 (reuse proof) — both already return the Target Agent Contract shape; this project makes the `trajectory` real by deriving it from OTel spans. Build 01 first.
