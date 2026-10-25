# Sentry Operations Skills

Reusable agent skills for governed building-operations workflows.

This repository is a sanitized public showcase of a private operational
assistant design. It demonstrates how an agent can route messages into narrow
tools, preserve audit context, gather structured evidence, and stop at explicit
approval boundaries.

## What is included

- building and equipment query routing;
- direct inspection and work-order commands;
- technician closeout as a stateful checklist workflow;
- intervention-only health monitoring;
- optimization recommendation review;
- fault-code and parts-supplier lookup patterns;
- complaint classification, call logging, and technician notification.

The skills are instruction-layer assets. They expect a consumer project to
provide its own tools, API, authentication, storage, notification channel, and
authority model. The examples use synthetic building, equipment, technician,
and chat identifiers.

## Design principles

1. Route first: classify the request before selecting a tool.
2. Use tools for facts: never invent equipment status, alarms, or work-order results.
3. Keep actions narrow: inspection, notification, approval, and reset are separate capabilities.
4. Preserve evidence: retain the original technician response and structured context.
5. Make interpretation explicit: diagnosis is a bounded step, not an invisible side effect.
6. Fail closed: unavailable APIs and missing authority stop the workflow cleanly.

## Relationship to the portfolio

This is the operational skill layer related to the broader
[Development Platform Core](https://github.com/Bederf/development-platform-core)
and the [GSD Master](https://github.com/Bederf/gsd-master) lifecycle skill.
It is intentionally separate from the SENTINEL-AI product repository and does
not include private BMS data, production credentials, deployment files, or
consumer implementation code.

## Adaptation

Before using a skill, replace the synthetic tool paths and environment
placeholders with a consumer-owned adapter. Do not place secrets in skill
files. Inject them at runtime through the consumer's approved secret manager.

## Project status

**Maturity: sanitized skill-library showcase.** The instruction-layer assets are
complete enough to study and adapt, but they are not a deployable BMS product.
Runtime tools, authentication, storage, notifications and authority remain
consumer-owned integration work.

## Fastest tour

Read the skill files alongside [Development Platform Core](https://github.com/Bederf/development-platform-core)
and [GSD Master](https://github.com/Bederf/gsd-master), then adapt one narrow
workflow with synthetic identifiers before connecting any real system.

## License

Copyright (c) Pieter van Rooyen. Released under the MIT License; see
[`LICENSE`](LICENSE).
