Migrate all functionality currently built on the Lead object to a new custom 
object, so the custom object fully replaces Lead's role. We can use the Groves sandbox to retrieve metadata if needed .This includes, but is 
not limited to:
- Fields (standard + custom)
- Flows (record-triggered, screen, scheduled)
- Apex Triggers and trigger-handler classes
- Validation rules, workflow rules, process builders (if any remain)
- Lightning Web Components (LWC) that reference Lead
- Visualforce pages/controllers that reference Lead
- Page layouts, record types, and assignment/escalation rules
- Reports, list views, and any automation tied to Lead conversion

Requirements:
1. The custom object must replicate all current Lead functionality — no 
   feature loss.
2. While migrating, apply Salesforce best practices. Where you identify 
   something on the current Lead implementation that is inefficient, 
   outdated, or not aligned with best practices (e.g., logic that should be 
   in Flow vs. Apex, bulkification issues, hardcoded IDs, missing error 
   handling), flag it as a recommended improvement rather than copying it 
   as-is.

Process (do this before writing any implementation code):

Step 1 — Discovery
Review the org and produce an inventory of everything currently built on the 
Lead object (fields, automations, LWC, VF pages, etc.).
Deliverable: docs/lead-features-inventory.md — a categorized list of every 
feature/component found, with a short description of what each one does.

Step 2 — Open Questions
While reviewing, list anything that needs clarification from the client before 
migration can be scoped accurately (e.g., "Is Lead Conversion logic in scope, 
or only pre-conversion Lead behavior?", "Should we keep field API names or 
rename them?", "Any integrations/external systems that reference the Lead 
object by ID or API name?").
Deliverable: docs/open-questions.md

Step 3 — Review checkpoint (stop here)
Do not proceed to design or implementation yet. Wait for:
 a) Client answers to docs/open-questions.md
 b) My approval of docs/lead-features-inventory.md

Step 4 — Implementation planning
Once both are approved, create the technical design docs needed to execute 
the migration (data model, migration/mapping plan, list of components to 
build/refactor, and the improvement recommendations from Step 1). We'll only 
start actual build work after this is approved too.