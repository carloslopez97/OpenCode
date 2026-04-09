# Review Rules

## Purpose

These rules keep review focused on real delivery risk: correctness, security, reliability, maintainability, and verification quality.

## Review Priorities

Review in this order:

1. correctness and data integrity
2. security and privacy
3. architecture and boundary compliance
4. reliability and failure handling
5. performance regression risk
6. maintainability and clarity
7. style and consistency

## Required Checks

Before marking work as ready, verify:

- the change matches the requested behavior
- the important acceptance criteria are addressed
- artifact references are present and coherent
- verification evidence is appropriate for the risk level
- unresolved risks or gaps are called out explicitly

Do not depend on PR metadata alone. Use the available artifacts, contracts, and evidence.

## Severity Model

### Critical

Must block progress. High likelihood of severe failure or exposure.

Examples:

- functional bug in the requested behavior
- security vulnerability
- major architecture violation
- missing verification for high-risk behavior

### High

Should be fixed before merge or rollout.

Examples:

- likely regression
- unsafe error handling
- broken edge-case behavior
- meaningful observability or operational gap

### Medium

Useful to address soon. Moderate risk or maintainability drag.

Examples:

- confusing structure
- avoidable duplication
- weak but non-blocking test coverage
- local performance concern without immediate failure risk

### Low

Small improvement with limited near-term impact.

Examples:

- naming polish
- minor cleanup
- consistency nits

## Feedback Standard

Every finding should include:

- severity
- concrete risk
- likely impact
- precise location
- actionable recommendation

If there are no findings, say so explicitly and mention any remaining uncertainty or testing gaps.

## Review Outcome Format

The review outcome should make these clear:

- overall assessment
- merge readiness: `ready`, `ready_with_minor_changes`, or `not_ready`
- severity-ranked findings
- verification gaps
- recommended next steps

## Anti-Patterns to Catch

Avoid approving work that has:

- vague evidence claims
- silent failure paths
- hidden coupling across boundaries
- unowned follow-up risks
- unreviewed sensitive flows
- parallel changes that create unresolved merge risk

## Enforcement

- critical findings block merge-readiness claims
- high findings require explicit resolution or conscious risk acceptance
- reviewers should optimize for signal, not volume
- absence of evidence is not evidence of correctness
