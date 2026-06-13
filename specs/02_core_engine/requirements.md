# Requirements — 02_core_engine

**Feature:** Selectors + config service + email service + the central sequence engine
**Source:** Target_Sequence_Solution_Design.md §4 (Apex Components), §4.1 (Engine flow), §3.5, §8, §9
**Depends on:** 01_data_model

## Purpose

Build the callable core of the cadence: the selectors, the metadata config reader, the
email-send service (which is the only place that can attach the uploaded file), and the
`SequenceEngineService.processStep` method that sends an email, creates the call + logged
tasks, advances the step, and schedules the next action. This feature has **no triggers
or jobs** — it is verified in isolation with unit + bulk tests.

## In scope

- `TargetSelector`, `TaskSelector`, `ContentSelector` (`inherited sharing`, USER_MODE SOQL).
- `SequenceStepConfigService` — reads `Sequence_Step_Config__mdt`.
- `SequenceEmailService` — builds/sends `Messaging.SingleEmailMessage` (template, "RE:",
  attachment, Org-Wide Email Address); plus an `@InvocableMethod` wrapper.
- `SequenceEngineService` — `processStep`, helpers, kill-switch guard, next-step scheduling.

## Out of scope

- `TargetTrigger`/`TaskTrigger` + queueable (03), batch/schedulable (04), before-update
  stop (05), LWC (06). Those invoke this engine; they are not built here.

## Requirements (EARS)

**R1 (Ubiquitous):** `SequenceStepConfigService` shall expose `Sequence_Step_Config__mdt` as a `Map<Integer, Sequence_Step_Config__mdt>` keyed by `Step_Number__c`.
**R2 (Ubiquitous):** `TargetSelector`, `TaskSelector`, and `ContentSelector` shall centralize all SOQL for their objects, declared `inherited sharing`, querying `WITH USER_MODE` and selecting every field the engine references (Rule A1).
**R3 (Event-driven):** When `SequenceEmailService.send` runs for a target+step, it shall build a `Messaging.SingleEmailMessage` from the step's Lightning Email Template (`Email_Template_Dev_Name__c`) using `setTargetObjectId(Primary_Contact__c)` and `setWhatId(Target__c.Id)`, and set the From to the Org-Wide Email Address resolved by name.
**R4 (Optional):** Where the step config's `Is_Reply__c` is true, the email subject shall be prefixed with `"RE: "`.
**R5 (State-driven):** While `Target__c.Sequence_Attachment_Id__c` is populated, the email shall include that ContentDocument's latest `ContentVersion` via `setEntityAttachments`.
**R6 (State-driven):** While `Target__c.Sequence_Attachment_Id__c` is blank, the email shall be sent with no attachment.
**R7 (Unwanted behavior):** If `Target__c.Sequence_Active__c` is false, then `processStep` shall return without sending email, creating tasks, or updating the record.
**R8 (Event-driven):** When `processStep(target, stepToSend)` runs for an active target, the engine shall (a) send `Email[stepToSend]`, (b) create the `Call N` Task (`ActivityDate = today + Call_Due_Offset_Days__c`, `Is_Sequence_Call__c=true`, `Sequence_Step__c=stepToSend`, `WhatId=target.Id`, `Status='Open'`), (c) insert a completed Email Task (`Type='Email'`, `Status='Completed'`, `WhatId=target.Id`), and (d) set `target.Sequence_Step__c=stepToSend`.
**R9 (Event-driven):** When `processStep` completes a step whose config `Next_Trigger_Type__c='Timer'`, the engine shall set `Next_Action_Date__c = now + Next_Wait_Days__c` (days).
**R10 (Event-driven):** When `processStep` completes a step whose config `Next_Trigger_Type__c='CallCompleted'`, the engine shall set `Next_Action_Date__c = null`.
**R11 (Event-driven):** When `processStep` completes a step whose config `Next_Trigger_Type__c='None'` (step 10), the engine shall leave `Next_Action_Date__c` null and make no further scheduling.
**R12 (Ubiquitous):** All engine DML shall be bulkified — collected into lists and executed as a single DML per object across all targets in the transaction (200-record safe), via `Database.*(records, AccessLevel.USER_MODE)` with partial-success handling.
**R13 (Optional):** Where a flow needs to call the send, `SequenceEmailService` shall expose an `@InvocableMethod` (with `label`+`description`) that delegates to the same send logic.
**R14 (Ubiquitous):** All classes in this feature shall be `with sharing` (services) or `inherited sharing` (selectors); no hardcoded Org-Wide Email Address or template Ids — resolved by developer name / metadata.

## Acceptance

- `processStep` on an active single target produces: 1 outbound email (test context), an
  open `Call N` task, a completed Email task, `Sequence_Step__c` advanced, and the correct
  `Next_Action_Date__c` per the step's `Next_Trigger_Type__c`.
- An inactive target produces nothing (R7).
- A 200-target bulk call stays within 1 DML per object and sends without governor errors.
- Solution Design §12 unit-test assertions for the engine pass.

## Open items

- **Apex single-email cap** (5,000/org/day, §10 item 2): the engine respects it; high
  volume is a monitoring/operational concern, not a code branch.
- **"RE:" subject only** (§8, confirmed): no `In-Reply-To`/`References` threading — the
  send service does not attempt it.
