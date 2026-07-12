---
type: prompt
id: write-aggregate-summary
title: Write Aggregate Summary
description: "Narrates the computed aggregate into a report."
tags: [Production, Reporting]
connections:
  - target: aggregate-report
    type: derived_from
metadata:
  output_format: markdown
  prompt_type: task
---

## Prompt

You are a business analyst. Every figure below was computed deterministically — do not recount or change any number.

### Unique shipped customers

{{steps.dedupe-customers.output}}

### Count

{{steps.count-customers.output}}

Write a short narrative report of the shipped-order activity by customer, using exactly the figures above. Do not invent totals.
