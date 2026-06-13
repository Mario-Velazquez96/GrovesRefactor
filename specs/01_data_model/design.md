# Design — 01_data_model

**Source:** Target_Sequence_Solution_Design.md §3 (Data Model), §3.5, §9, §10

## Approach

Pure declarative metadata — custom fields, two Custom Metadata Types (+ rows), 10
Lightning Email Templates, a custom index, and two permission sets. No Apex or LWC. The
configuration that varies across the 10 near-identical steps lives entirely in
`Sequence_Step_Config__mdt`, so re-timing/re-templating the cadence is a metadata edit
(Solution Design §1, §3.3).

## Fields — `Target__c` (§3.1)

| Field | Type | Notes |
|---|---|---|
| `Sequence_Active__c` | Checkbox, default true | Kill switch (R1) |
| `Sequence_Step__c` | Number(2,0), default 0 | Counter 0–10 (R2) |
| `Days_Until_Next_Email__c` | Number(3,0), default 4 | Wait for call-driven steps 1–6 (R3) |
| `Sequence_Stop_Reason__c` | Text(255) | Auto-stop reason (R4) |
| `Next_Action_Date__c` | DateTime | Scheduler due-date; indexed (R5, R15) |
| `Primary_Contact__c` | Lookup(Contact) | Recipient + `[Primary Contact]` merge (R6) |
| `Sequence_Attachment_Id__c` | Text(18) | `ContentDocumentId` set by LWC (R7) |
| `Billing_City__c` | Text Formula = `Company_Name__r.BillingCity` | `[Billing City]` merge (R8) |

## Fields — `Task` (§3.2)

| Field | Type | Notes |
|---|---|---|
| `Is_Sequence_Call__c` | Checkbox | Robust detection of engine call tasks (R9) |
| `Sequence_Step__c` | Number(2,0) | Which call number 1–10 (R10) |

## CMDT — `Sequence_Step_Config__mdt` rows (§3.3, §6)

| DeveloperName | Step | Email_Template_Dev_Name | Is_Reply | Call_Task_Subject | Call_Due_Offset_Days | Next_Trigger_Type | Next_Wait_Days |
|---|---|---|---|---|---|---|---|
| Step_1 | 1 | Sequence_Email_1 | false | Call 1 | 2 | CallCompleted | — |
| Step_2 | 2 | Sequence_Email_2 | true | Call 2 | 2 | CallCompleted | — |
| Step_3 | 3 | Sequence_Email_3 | true | Call 3 | 2 | CallCompleted | — |
| Step_4 | 4 | Sequence_Email_4 | true | Call 4 | 2 | CallCompleted | — |
| Step_5 | 5 | Sequence_Email_5 | true | Call 5 | 2 | CallCompleted | — |
| Step_6 | 6 | Sequence_Email_6 | true | Call 6 | 2 | Timer | 14 |
| Step_7 | 7 | Sequence_Email_7 | true | Call 7 | 2 | Timer | 7 |
| Step_8 | 8 | Sequence_Email_8 | true | Call 8 | 2 | Timer | 14 |
| Step_9 | 9 | Sequence_Email_9 | true | Call 9 | 2 | Timer | 14 |
| Step_10 | 10 | Sequence_Email_10 | true | Call 10 | 2 | None | — |

`Next_Trigger_Type__c` describes how the **next** step is reached: steps 1–5 wait on the
call completion (then `Days_Until_Next_Email__c`), steps 6–9 advance on a fixed timer,
step 10 is terminal. (Engine logic that reads this lives in `02_core_engine`.)

## CMDT — `Sequence_Terminal_Status__mdt` rows (§3.4, Feature 11)

| DeveloperName | Status_Value | Stop_Reason |
|---|---|---|
| Converted | Converted | Converted |
| Meeting_Booked | Meeting Booked | Meeting Booked |
| Do_Not_Contact | Do Not Contact | Do Not Contact |
| Replied | Replied | Replied |

## Email Templates (§3.5)

10 Lightning Email Templates `Sequence_Email_1..10` from the doc copy (Document.md
"Email templates" section). Merge fields use HML so the Apex send in `02` can supply both
recipient (Contact) and related-to (Target). The "RE:" subject prefix is **not** baked
into the templates — it is applied in Apex from `Is_Reply__c` (one template per step).

## Security (§9)

- `Login_Sequence_User`: FLS read/edit on all new `Target__c` + `Task` fields. (Apex
  class access is added by later features as their classes deploy — D2 in the dev rules.)
- `Login_Sequence_Admin`: additionally manage the two CMDTs.
- No object/field is introduced that isn't listed above (metadata hygiene).

## Bulkification

N/A — declarative metadata only.

## Test approach

No Apex in this feature. Verification is deployment + data checks:
- Deploy-validate the metadata.
- Anonymous Apex / query: assert `Sequence_Step_Config__mdt.getAll().size() == 10` and
  spot-check Step_6 (`Next_Trigger_Type__c='Timer'`, `Next_Wait_Days__c=14`); assert
  `Sequence_Terminal_Status__mdt.getAll().size() == 4`.
- Render each email template against a sample Target+Contact and confirm merge output.

## Resolved decisions / discrepancies

- **`Company_Name__c` targets `Account`** (confirmed 2026-06-12) — `Billing_City__c =
  Company_Name__r.BillingCity` (R8). No remaining ambiguity on the formula path.
- `Next_Action_Date__c` custom index (R15) may need a Support request; flagged as a
  prerequisite, not auto-deployable (operational, not a decision).
