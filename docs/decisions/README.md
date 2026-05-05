# Architecture Decision Records (ADRs)

This directory holds **per-decision** records. **Memory file `decisions.md` is the living TL;DR**; ADRs here preserve reasoning history when a decision is **reversed or superseded**.

## When to add an ADR

ADRs are **on-demand**, not exhaustive. Add one only when:
- A previous decision is being **reversed** or **superseded** (preserve original reasoning + new path)
- A choice has **rich context** that doesn't fit a 2-3 line memory entry
- Future readers will likely re-litigate without it

For decisions that are stable and unsurprising, leave them only in `decisions.md` (memory). No ADR needed.

## Existing ADRs

v0.2.0 baseline 7 결정 중 **rich context 있는 3개**를 backfill:

| # | Title | 메모리 항목 |
|---|---|---|
| [ADR-0001](0001-progressive-disclosure.md) | Adopt Progressive Disclosure for Agent Knowledge | `decisions.md` #2 |
| [ADR-0002](0002-visibility-protocol-3-tier.md) | Adopt 3-Tier Visibility Protocol for Operating Principles | `decisions.md` #3 |
| [ADR-0003](0003-adopt-openai-harness-patterns.md) | Selectively Adopt OpenAI Harness Engineering Patterns | `decisions.md` #7 |

나머지 4개 결정(#1 Hybrid 구조, #4 Computational sensors next-ts, #5 Sync 자동화 보류, #6 Architecture guide opt-in)은 **2~3줄로 충분히 표현됨** — backfill 안 함, [memory/decisions.md](../../memory/decisions.md)에서만 관리.

새 reversal 또는 rich context decisions이 나오면 다음 번호(0004부터)로 추가.

## Numbering

`<NNNN>-<short-slug>.md`. NNNN starts at `0001`. Numbers don't reset, never reuse.

## Template

```markdown
# ADR <NNNN>: <Short Title>

- **Status**: Proposed | Accepted | Superseded by ADR-NNNN | Deprecated
- **Date**: YYYY-MM-DD
- **Supersedes**: (optional) ADR-NNNN

## Context

배경. 이 결정이 왜 필요해졌는지. 이전 상태가 뭐였는지.

## Decision

확정한 내용. 한 문장으로.

## Consequences

- 긍정: ...
- 부정: ...
- 후속 작업: ...

## Alternatives considered

- A: <기각 사유>
- B: <기각 사유>

## Related

- 코드 위치: <files>
- 메모리: `decisions.md` 항목 N
- 출처: <links>
```

## ADR 작성 후 해야 할 일

1. 이 디렉토리에 `<NNNN>-<slug>.md`로 commit
2. `memory/decisions.md`의 해당 항목 in-place 편집:
   - `~~기존 결정~~ → 새 결정. ADR-NNNN 참조.`
   - "Last updated" 헤더 갱신
3. README의 "Existing decisions" 섹션에 카운트 갱신(필요 시)
