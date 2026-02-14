---
title: Introduction
description: Learn Effect's Schedule type for defining recurring execution patterns for retry and repeat workflows with configurable policies in Effect programs.
---

# Introduction

Source: extracted from `llms-full.txt` (`Introduction`).

## Overview

Scheduling in Effect defines recurring execution patterns for effects.
It is centered on the `Schedule` type, an immutable description of when an
effect may continue and when it should stop.

## The `Schedule` Type

```text
          ┌─── The type of output produced by the schedule
          │   ┌─── The type of input consumed by the schedule
          │   │     ┌─── Additional requirements for the schedule
          ▼   ▼     ▼
Schedule<Out, In, Requirements>
```

A schedule consumes `In` values (for example, retry errors or repeat inputs),
produces `Out` values, and uses internal state plus timing to decide whether to
continue or halt recurrence.

The `Requirements` parameter allows a schedule to depend on additional services
or resources when needed.

## Intervals and Timing

Schedules are modeled as intervals over time.
Each interval represents a window where the next recurrence can happen.

## Retrying vs Repetition

- **Retrying** reruns an effect after failure.
- **Repetition** reruns an effect to keep producing results.

In both cases, the start boundary of each interval determines when the next run
is attempted.

## Composability

Schedules are composable.
You can combine simple schedules into more complex policies using operators such
as `Schedule.union` and `Schedule.intersect`.
