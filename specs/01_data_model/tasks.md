# Tasks — 01_data_model

> Each task cites the requirement(s) it satisfies. Mark `[x]` as completed.

- [x] Confirmed: `Company_Name__c` targets `Account` → `Billing_City__c = Company_Name__r.BillingCity` (R8)
- [ ] Create `Target__c` fields: `Sequence_Active__c`, `Sequence_Step__c`, `Days_Until_Next_Email__c`, `Sequence_Stop_Reason__c`, `Next_Action_Date__c`, `Primary_Contact__c`, `Sequence_Attachment_Id__c` (R1–R7)
- [ ] Create `Target__c.Billing_City__c` formula = `Company_Name__r.BillingCity` (R8)
- [ ] Create `Task` fields `Is_Sequence_Call__c`, `Sequence_Step__c` (R9, R10)
- [ ] Create CMDT `Sequence_Step_Config__mdt` with the 7 fields (R11)
- [ ] Load the 10 step rows per Design §Rows table (R12)
- [ ] Create CMDT `Sequence_Terminal_Status__mdt` with its 2 fields (R13)
- [ ] Load the 4 terminal-status rows (R13)
- [ ] Build 10 Lightning Email Templates `Sequence_Email_1..10` from `project-documents/client_requirement.md` copy with the three merge fields (R14)
- [ ] Request/create the custom index on `Next_Action_Date__c` (R15) — note: may require Support
- [ ] Create permission sets `Login_Sequence_Admin` and `Login_Sequence_User` with FLS for all new fields (R16)
- [ ] Set API version 66.0 on all new metadata (R17)
- [ ] Deploy-validate to scratch/sandbox; run the data checks below

## Verification

- **R1–R10:** deploy succeeds; fields visible on `Target__c`/`Task`; defaults correct
  (`Sequence_Active__c=true`, `Sequence_Step__c=0`, `Days_Until_Next_Email__c=4`).
- **R11–R13 (anonymous Apex):**
  `Assert.areEqual(10, Sequence_Step_Config__mdt.getAll().size());`
  spot-check Step_6 → `Timer`, `Next_Wait_Days__c=14`, `Is_Reply__c=true`;
  `Assert.areEqual(4, Sequence_Terminal_Status__mdt.getAll().size());`
- **R14:** render `Sequence_Email_3` against a sample Target (with `Company_Name__c`→Account
  having a Billing City) + Contact → `[Target Name]`, `[Primary Contact]`, `[Billing City]`
  all resolve.
- **R16:** assign `Login_Sequence_User` to a test user; confirm field read/edit access.

> No Apex coverage applies (declarative-only feature).
