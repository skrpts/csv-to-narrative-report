---
type: workflow
id: csv-to-narrative-report
title: CSV to Narrative Report
description: "Parse CSV, filter and dedupe deterministically, count, then summarise the aggregate."
tags: [Production, Reporting]
connections:
  - target: aggregate-report
    type: uses
  - target: llm-service
    type: runs_on
metadata:
  estimated_duration: "1-2 minutes"
  trigger: manual
output_step: "aggregate-report"
execution:
  - id: "parse-csv"
    step_type: "local.builtin"
    input: "{{input.csv}}"
    context:
      operation: "csv-to-json"
    output: { name: "rows", type: "text" }
  - id: "filter-rows"
    step_type: "local.builtin"
    input: "{{steps.parse-csv.output}}"
    context:
      operation: "json-filter"
      expression: ". | select(status == \"shipped\")"
    output: { name: "shipped_rows", type: "text" }
  - id: "dedupe-customers"
    step_type: "local.builtin"
    input: "{{steps.filter-rows.output}}"
    context:
      operation: "dedup-merge"
      key: "customer"
    output: { name: "unique_customers", type: "text" }
  - id: "count-customers"
    step_type: "local.builtin"
    input: "{{steps.dedupe-customers.output}}"
    context:
      operation: "count"
      mode: "json-array"
    output: { name: "customer_count", type: "text" }
  - skill: "aggregate-report"
    step_type: "generation"
    prompt: "write-aggregate-summary"
    output: { name: "report", type: "text" }
---

## Overview

Asking a model to "count the shipped orders per customer" from a raw CSV is where hallucinated arithmetic creeps in. This skrpt does every number **deterministically** with zero-token builtins — parse the CSV, filter to shipped rows, dedupe by customer, count the unique customers — and only then hands the model the computed result to narrate. The maths is exact; the model just writes the prose.

## Pipeline Stages

### Stage 1: Parse CSV (builtin)
**csv-to-json** turns the CSV into a JSON array of row objects.

### Stage 2: Filter (builtin)
**json-filter** keeps only the rows where `status == "shipped"`.

### Stage 3: Dedupe (builtin)
**dedup-merge** collapses the shipped rows to unique customers.

### Stage 4: Count (builtin)
**count** reports the number of unique customers — exact, no model.

### Stage 5: Narrative Report
The model writes a short report from the computed figures.

## Inputs

| Name | Required | Description | Example |
|------|----------|-------------|---------|
| `{{input.csv}}` | Yes | CSV with at least `customer` and `status` columns | `order_id,customer,status\n1,Acme,shipped` |

## Example Input

```
CSV:
order_id,customer,status
1,Acme,shipped
2,Acme,shipped
3,Globex,pending
4,Initech,shipped
```
