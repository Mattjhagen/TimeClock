The implementation plan is directionally approved, but it requires the following revisions before implementation begins.

Required Plan Revisions

1. Correct the 1099 Contractor Model

This is a 1099 independent-contractor time and earnings system, not a W-2 employee payroll system.

Update the plan title from:

Time Clock & Payroll Module

to:

Time Clock & Contractor Earnings Module

Replace employee/payroll terminology throughout the plan:

* employee_settings → contractor_settings
* Employee Settings → Contractor Settings
* Employee Name → Contractor Name
* Payroll Calculations → Earnings Calculations
* Estimated Pay → Estimated Gross Earnings
* Net Pay → Do not use
* Employment Record → Contractor Timesheet Record
* Employee Signature → Contractor Signature

The system must not calculate, estimate, deduct, display, or withhold:

* Federal income tax
* State income tax
* Local income tax
* Social Security
* Medicare
* Unemployment insurance
* Workers’ compensation
* Benefits
* Any other tax or payroll deduction

All financial values must be labeled as gross contractor earnings before taxes and expenses.

Add this disclaimer to the dashboard, emailed timesheets, and PDFs:

“Earnings shown are gross 1099 contractor earnings before taxes, business expenses, or other deductions. No federal, state, or local taxes are calculated or withheld by this application.”

2. Do Not Assume Statutory Overtime

Because this is a 1099 contract, the system must not imply that overtime is legally required.

Rename:

* Overtime Enabled → Additional Rate Enabled
* Overtime Threshold → Additional Rate Threshold
* Overtime Multiplier → Additional Rate Multiplier
* Overtime Earnings → Additional Rate Earnings

The default may be 40 hours and 1.5×, but it must be clearly described as a configurable contractual compensation rule.

The calculation engine should support:

* Base-rate hours
* Additional-rate hours
* Base earnings
* Additional-rate earnings
* Total gross contractor earnings

3. Resolve Authentication Before Creating RLS

Do not implement production time-entry tables or Row Level Security against a mock or ambiguous authentication model.

Before implementation:

1. Inspect the current authentication system.
2. Identify whether Supabase Auth is already active.
3. Identify how the current authenticated user ID is obtained.
4. Identify any existing profiles, users, admins, or role tables.
5. Document how contractor records are associated with an authenticated user.
6. Document whether the current application is single-user or multi-user.

If SchmidtAdmin is currently a single-owner administrative application, do not introduce a complex multi-employee role system without justification.

Prefer the simplest architecture that securely supports the actual current user model.

The implementation plan must specify the exact RLS identity expression, such as:

auth.uid() = user_id

Do not leave this unresolved while writing the migration.

4. Confirm Deployment Before Selecting Vercel Cron

Do not assume Vercel deployment merely because the application uses Next.js.

Inspect and document:

* Current hosting provider
* Existing deployment configuration
* Existing scheduled-job support
* Whether vercel.json already exists
* Whether the deployed Vercel plan supports the required cron frequency
* How CRON_SECRET will be configured

If the project is not deployed on Vercel, choose a scheduler that matches the real hosting environment.

The selected automatic clock-out system must run at least hourly and independently of the browser.

5. Strengthen Automatic Clock-Out Concurrency Safety

The hourly route must be idempotent and safe if two scheduler invocations overlap.

Do not implement this as an unsafe sequence of:

1. Select all open entries.
2. Update each entry.
3. Insert audit rows separately.

Use a database transaction or atomic database function.

Recommended approach:

* Create a Supabase/Postgres RPC function that atomically claims or closes a qualifying entry.
* Update only where clock_out IS NULL.
* Return whether the entry was actually modified.
* Insert the audit record in the same transaction.
* Close active breaks in the same transaction.
* Ensure a retry cannot create duplicate AUTO_CLOCK_OUT audit events.

At minimum, the update must include a conditional predicate equivalent to:

WHERE id = ? AND clock_out IS NULL

A unique or partial constraint should prevent duplicate unresolved automatic-clock-out audit events for the same entry.

The API route may perform timezone evaluation in TypeScript, but the final mutation must be atomic at the database layer.

6. Use One Clear Midnight Boundary Convention

Use the next day’s local 00:00:00 instant as the clock-out boundary, stored in UTC.

Do not store 11:59:59 PM, because that introduces an artificial missing second and makes duration calculations less clean.

Example:

* Contractor clocks in July 18 at 8:00 PM local time.
* Contractor forgets to clock out.
* Entry is closed at July 19, 12:00:00 AM local time.
* The stored UTC timestamp must represent that exact local instant.

The UI may describe this as:

“Automatically clocked out at midnight.”

Document how ambiguous and nonexistent daylight-saving-time transitions are handled.

7. Add a Secure Mutation Layer

The plan currently lists pages, but it does not fully identify how sensitive writes will occur.

Add explicit server actions, services, RPC functions, or API routes for:

* Clock in
* Clock out
* Start break
* End break
* Create manual entry
* Edit time entry
* Resolve review flag
* Void an entry
* Submit timesheet
* Amend a submitted timesheet
* Send timesheet email

Do not permit direct unvalidated browser writes to sensitive time, rate, audit, or submission tables.

Each mutation must:

* Verify the authenticated user
* Validate ownership
* Validate the current entry state
* Use server timestamps where appropriate
* Create an audit record
* Prevent overlapping entries
* Return structured errors

8. Expand the Data Model

Update the proposed tables.

contractor_settings

Include:

* user_id
* contractor_name
* company_name
* manager_name
* manager_email
* hourly_rate_cents
* time_zone
* additional_rate_enabled
* additional_rate_threshold_minutes
* additional_rate_multiplier
* auto_clock_out_enabled
* created_at
* updated_at

Store currency in integer cents, not floating-point dollars.

time_entries

Include:

* id
* user_id
* clock_in
* clock_out
* project
* customer_name
* job_site_address
* work_category
* notes
* status
* auto_clock_out
* needs_review
* review_reason
* manual_entry
* voided_at
* void_reason
* created_at
* updated_at

Do not store mutable “original clock-in” and “original clock-out” columns as the sole history mechanism. The immutable audit table should preserve previous values.

time_entry_breaks

Include:

* id
* time_entry_id
* start_time
* end_time
* created_at
* updated_at

Duration should normally be derived from timestamps. A stored duration may be used only if required for submitted snapshots or manual corrections.

timesheet_periods

Include:

* id
* user_id
* period_start
* period_end
* status
* submitted_at
* amended_at
* submission_version
* totals_snapshot
* entries_snapshot
* email_status
* email_message_id
* created_at
* updated_at

A totals snapshot alone is insufficient.

Store an immutable snapshot of the exact entries included in the submission so later edits do not rewrite history.

time_entry_audit

Include:

* id
* time_entry_id
* user_id
* event_type
* field_name
* old_value
* new_value
* reason
* metadata
* created_at

Prevent normal UI users from updating or deleting audit records.

9. Add Editing and Manual Entry Details

The implementation plan must explicitly include:

* Edit-entry modal or page
* Mandatory edit reason
* “Other” reason requiring explanation
* Manual missed-shift creation
* Overlap validation
* Break-duration validation
* Clock-out-after-clock-in validation
* Edited badge
* Manual Entry badge
* Full entry-history display

Editing an automatically clocked-out entry should:

1. Preserve the midnight clock-out in the audit history.
2. Record the corrected clock-out value.
3. Require a reason.
4. Clear needs_review only when the user explicitly resolves the flag.
5. Retain auto_clock_out = true as historical information.

Do not erase the fact that automatic clock-out occurred.

10. Define Submission and Amendment Behavior

Before submission:

* Unresolved flagged entries must be clearly shown.
* The application should normally prevent submission until required flags are resolved.

When submitting:

* Store the exact entries snapshot.
* Store rates and earnings calculations as they existed at submission.
* Record submission timestamp and version.
* Email the submitted snapshot, not live mutable entries.

When an entry is edited after submission:

* Preserve the original submitted snapshot.
* Mark the timesheet as amended.
* Increment the submission version.
* Require a correction reason.
* Allow the corrected version to be resubmitted and re-emailed.
* Never silently modify the previously submitted record.

11. Update PDF and Email Language

The PDF and email must use contractor terminology.

Include:

* Contractor name
* Client or company name
* Pay period
* Work dates
* Clock-in and clock-out times
* Breaks
* Base-rate hours
* Additional-rate hours
* Gross contractor earnings
* Projects and notes
* Edit and automatic-clock-out indicators
* Contractor signature
* Manager/client acknowledgment line
* 1099 gross-earnings disclaimer

Do not describe the document as:

* Payroll
* Paycheck
* Net pay
* Tax withholding statement
* Employment record

12. Expand Automated Testing

Add tests for:

* Authentication and ownership checks
* RLS policies
* Duplicate clock-in race conditions
* Duplicate break-start race conditions
* Two simultaneous automatic-clock-out executions
* Audit insertion atomicity
* Automatic clock-out at exactly local midnight
* Half-hour and quarter-hour time zones
* DST transitions
* Time-zone changes while a shift is open
* Manual-entry overlaps
* Editing an automatically clocked-out entry
* Resolving a review flag
* Submitted snapshot immutability
* Timesheet amendments
* Integer-cent earnings calculations
* Additional-rate calculations
* No tax or withholding calculations anywhere in the system

13. Revised Verification Requirements

Manual testing should include:

1. Clock in and refresh the browser.
2. Close and reopen the browser while clocked in.
3. Start and end multiple breaks.
4. Edit a completed entry and verify the audit history.
5. Create a manual missed entry.
6. Attempt an overlapping entry.
7. Simulate midnight in multiple time zones.
8. Invoke the cron endpoint twice and verify only one automatic clock-out and one audit event occur.
9. Correct an automatically closed entry and resolve its flag.
10. Submit a timesheet.
11. Edit a submitted entry and verify the original submission remains unchanged.
12. Generate the contractor PDF.
13. Send the contractor earnings email.
14. Verify no tax, deduction, withholding, or net-pay fields appear anywhere.

After revising the implementation plan with these items, stop and present the updated plan for review before writing implementation code.
