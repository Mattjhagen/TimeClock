SchmidtAdmin: Professional Time Clock & Payroll Module

Objective

Implement a production-quality Time Clock module for SchmidtAdmin that allows employees to accurately track work hours, calculate estimated gross pay, correct mistakes, and submit professional timesheets to management.

This is not a basic stopwatch. It should feel like a polished commercial time-tracking product comparable to QuickBooks Time, Clockify, Homebase, or ADP Time.

⸻

1. Time Clock Navigation

Add a new primary navigation item:

* Time Clock

The page must match the existing SchmidtAdmin visual language and application architecture.

⸻

2. Employee Settings

Create persistent employee settings stored in Supabase.

Fields:

* Employee name
* Hourly pay rate
* Boss or manager email
* Time zone
* Company name
* Overtime enabled
* Overtime threshold, defaulting to 40 hours
* Overtime multiplier, defaulting to 1.5x
* Automatic clock-out enabled
* Automatic clock-out time, defaulting to midnight in the employee’s configured time zone

These settings should only need to be configured once but must remain editable.

⸻

3. Clock In and Clock Out

Create a professional time-clock interface displaying:

* Current status: Working, On Break, or Off Duty
* Current clock-in time
* Large live elapsed-time display
* Current project or work assignment
* Clock In button
* Clock Out button
* Start Break button
* End Break button

Button availability must respond intelligently to the employee’s current state.

Prevent:

* Clocking in twice
* Clocking out without an active shift
* Starting multiple simultaneous breaks
* Ending a break that is not active

⸻

4. Server-Based Time Tracking

Do not trust the browser or device clock as the authoritative time source.

Use server-generated timestamps through Supabase or secure server routes.

Each time entry must support:

* Employee or user ID
* Clock-in timestamp
* Clock-out timestamp
* Original clock-in timestamp
* Original clock-out timestamp
* Break duration
* Project
* Customer or job-site address
* Work notes
* Entry status
* Automatic clock-out flag
* Needs-review flag
* Review reason
* Created timestamp
* Updated timestamp

All calculations must use the employee’s configured time zone.

⸻

5. Automatic Midnight Clock-Out

If an employee remains clocked in at midnight in their configured time zone, automatically close the active time entry.

The system must:

1. Set the clock-out time to exactly 11:59:59 PM or midnight at the end of the workday, using one consistent documented convention.
2. Mark the entry as automatically clocked out.
3. Set needs_review to true.
4. Add a review reason such as:
    * “Automatically clocked out at midnight because no manual clock-out was recorded.”
5. Clearly flag the affected day in the daily and weekly timesheet views.
6. Display a warning to the employee the next time they open the Time Clock.
7. Allow the employee to correct the clock-out time.
8. Require a reason when the employee corrects the entry.
9. Preserve the original automatic clock-out value in the audit trail.
10. Never automatically continue the shift into the next day.

Use a reliable server-side scheduled process rather than relying on the browser remaining open.

Use the project’s existing deployment architecture where possible. Determine whether this should be implemented with:

* A Supabase scheduled function
* A database cron job
* A secure scheduled API route
* Another reliable server-side scheduler already supported by the project

The implementation plan must explain the selected approach and why it is reliable.

The automatic clock-out process must be idempotent so that retrying the scheduled job cannot create duplicate entries or repeatedly modify the same shift.

⸻

6. Editable Time Entries

Employees must be able to edit past and current time entries.

Editable fields:

* Clock-in time
* Clock-out time
* Break duration
* Project
* Customer or job-site address
* Work notes

Every edit must require an edit reason.

Example reasons:

* Forgot to clock out
* Forgot to clock in
* Incorrect break duration
* Entered the wrong project
* Automatic midnight clock-out correction
* Other

For “Other,” require a written explanation.

Time edits must never silently replace historical data.

For every edit, store:

* Entry ID
* User who made the edit
* Field changed
* Previous value
* New value
* Edit reason
* Timestamp
* Whether the entry had previously been submitted
* Whether the edit caused the entry to require resubmission or manager review

Visually mark edited entries with an “Edited” badge.

Allow the employee to view an entry’s complete edit history.

⸻

7. Manual Time Entry

Allow employees to manually add a missed shift.

Required fields:

* Date
* Clock-in time
* Clock-out time
* Break duration
* Project or assignment
* Reason for manual entry

Manual entries must:

* Be clearly labeled “Manual Entry”
* Be included in payroll calculations
* Be recorded in the audit trail
* Be flagged for review if the timesheet workflow includes manager approval

Validate that:

* Clock-out occurs after clock-in
* Break duration does not exceed shift duration
* Entries do not unintentionally overlap
* The entry does not produce impossible or unreasonable hours

Warn the user about overlapping entries and require confirmation or correction.

⸻

8. Work Details

When clocking in or editing a shift, support:

* Project or job
* Customer name
* Customer address or job-site address
* Work category
* Work notes

Provide useful preset work categories while allowing custom values.

Example categories:

* General Construction
* Office
* Concrete
* Roofing
* Cleanup
* Travel
* Materials
* Customer Meeting
* Other

⸻

9. Break Tracking

Support:

* Start Break
* End Break
* Multiple breaks in one shift
* Total unpaid break time
* Net paid hours

Store individual break periods rather than only a single combined number when possible.

Each break should contain:

* Break start
* Break end
* Duration
* Whether it was manually edited

An active break must also be closed when a shift is automatically closed at midnight.

⸻

10. Today’s Summary

Display a summary card containing:

* First clock-in
* Current or final clock-out
* Total break time
* Paid hours
* Estimated earnings
* Current status
* Project
* Whether the entry needs review

If the previous day was automatically clocked out, display a prominent review alert.

Example:

“Yesterday’s shift was automatically clocked out at midnight. Review and correct the entry before submitting your timesheet.”

⸻

11. Weekly Timesheet

Create a professional weekly timesheet table.

Columns:

* Date
* Status
* Clock In
* Clock Out
* Break
* Paid Hours
* Hourly Rate
* Daily Gross
* Project
* Flags
* Actions

Possible flags:

* Auto Clock-Out
* Needs Review
* Edited
* Manual Entry
* Submitted
* Approved

Actions:

* View
* Edit
* View History
* Resolve Flag

Include totals for:

* Regular hours
* Overtime hours
* Break time
* Gross pay
* Number of flagged entries

Flagged entries should be visually prominent without making the interface feel alarming or unfinished.

⸻

12. Payroll Calculations

Automatically calculate:

* Gross shift duration
* Unpaid break time
* Net paid hours
* Regular hours
* Overtime hours
* Regular pay
* Overtime pay
* Estimated gross pay

Example:

* 36 regular hours
* 5 overtime hours
* Regular pay: $900.00
* Overtime pay: $187.50
* Estimated gross pay: $1,087.50

Use decimal-safe currency calculations. Avoid floating-point rounding errors.

Clearly label all pay amounts as estimates unless the application is serving as the employer’s authoritative payroll system.

⸻

13. Dashboard Cards

Create polished dashboard cards for:

* Hours Today
* Hours This Week
* Estimated Gross Pay
* Current Status
* Last Clock In
* Weekly Progress
* Entries Needing Review

The “Entries Needing Review” card should link directly to flagged entries.

⸻

14. Timesheet Review and Submission

Before a timesheet can be submitted:

* Show all entries in the pay period
* Highlight automatic clock-outs
* Highlight unresolved review flags
* Require the employee to review flagged entries
* Prevent submission while unresolved required corrections remain, unless the workflow intentionally allows submission with flags

Submission should:

* Lock the submitted snapshot
* Record submission date and time
* Store totals at the time of submission
* Preserve the exact entries included
* Allow later corrections through a documented amendment workflow

Editing a submitted entry must not silently change the original submitted timesheet.

Instead:

1. Preserve the submitted snapshot.
2. Record the correction.
3. Mark the timesheet as amended or requiring resubmission.
4. Record the reason for the change.

⸻

15. Email Timesheet

Add a “Submit and Email Timesheet” action.

Use the existing Resend infrastructure already present in SchmidtAdmin.

Send a professional branded HTML email containing:

* Employee name
* Company
* Pay period
* Submission date
* Daily entries
* Regular hours
* Overtime hours
* Total paid hours
* Hourly rate
* Estimated gross pay
* Work notes
* Edited-entry indicators
* Automatic clock-out indicators
* Unresolved or resolved flags
* Link or reference to the timesheet inside SchmidtAdmin

Do not email the manager after every individual clock-out.

The primary workflow should send a polished weekly or pay-period report. An optional daily summary setting may be included separately.

⸻

16. Printable and PDF Timesheet

Generate a clean printable timesheet or PDF containing:

* Company
* Employee
* Pay period
* Daily entries
* Projects
* Break time
* Regular hours
* Overtime hours
* Estimated gross pay
* Flags and correction notes
* Employee signature line
* Manager signature line
* Submission timestamp

The PDF must look professional enough to serve as an employment record.

⸻

17. Audit Trail

Create a complete immutable audit trail.

Audit events should include:

* Clock in
* Clock out
* Start break
* End break
* Automatic clock-out
* Manual entry creation
* Time edit
* Entry deletion or voiding
* Flag resolution
* Timesheet submission
* Timesheet amendment

Never permanently delete historical time data through the normal UI.

Use soft deletion or a voided status with a required reason.

⸻

18. Database Design

Create appropriate Supabase migrations for tables such as:

employee_settings

Stores:

* User ID
* Employee name
* Hourly rate
* Manager email
* Company name
* Time zone
* Overtime settings
* Automatic clock-out settings

time_entries

Stores:

* User ID
* Clock-in and clock-out timestamps
* Break totals
* Project and notes
* Status
* Automatic clock-out flag
* Needs-review flag
* Review reason
* Manual-entry flag
* Submission state

time_entry_breaks

Stores individual break periods.

timesheet_periods

Stores:

* Period start and end
* Submission status
* Submission timestamp
* Totals snapshot
* Email-delivery status
* Amendment status

time_entry_audit

Stores immutable event and edit history.

Use proper foreign keys, indexes, constraints, and Row Level Security.

Authenticated employees should only be able to access their own entries unless a manager or administrator role is explicitly supported.

Sensitive payroll and employee data must never be publicly readable.

⸻

19. Reliable Scheduled Processing

The midnight auto-clock-out feature must run independently of the user’s device.

The scheduled process must:

* Identify open shifts whose local workday has ended
* Respect each employee’s configured time zone
* Close open breaks
* Close the shift
* Add the automatic clock-out and review flags
* Add an audit event
* Avoid processing the same entry more than once
* Log failures
* Be safe to retry

Account for daylight-saving-time transitions and time-zone changes.

Document how the scheduler is deployed, authenticated, monitored, and tested.

⸻

20. Offline and Recovery Behavior

If the internet is unavailable:

* Preserve attempted clock-in and clock-out actions locally
* Show that the action is pending synchronization
* Sync safely when connectivity returns
* Prevent duplicate punches
* Reconcile local intent with server time
* Never silently lose a punch

Because the server is authoritative, clearly define how offline timestamps are recorded and reviewed.

Offline or delayed punches may need to be labeled and audited rather than silently accepted as normal server-time punches.

⸻

21. Mobile Experience

The interface should feel like a native mobile time clock.

Use:

* Large touch targets
* One-tap primary actions
* Minimal required typing
* Clear current status
* Responsive layout
* Accessible contrast
* Keyboard-accessible controls
* Helpful confirmation and error states

Avoid a toy stopwatch appearance.

⸻

22. Code Quality

Follow the existing SchmidtAdmin architecture.

Separate:

* UI components
* Time-clock state
* Server actions and API routes
* Database access
* Time-zone handling
* Payroll calculations
* Validation
* Audit logging
* Scheduled jobs
* Email templates
* PDF generation

Create reusable hooks and services where appropriate.

All payroll and duration calculations should be covered by automated tests.

⸻

23. Required Tests

Include tests for:

* Normal clock-in and clock-out
* Duplicate clock-in prevention
* Break tracking
* Multiple breaks
* Manual time entry
* Editing an entry
* Audit-history creation
* Midnight automatic clock-out
* Automatic closure of an active break
* Time-zone boundaries
* Daylight-saving-time boundaries
* Scheduler retry and idempotency
* Regular-hour calculation
* Overtime calculation
* Currency rounding
* Overlapping entries
* Submitted-timesheet amendments
* Row Level Security expectations

⸻

24. Design Direction

Use the existing SchmidtAdmin design system.

The experience should be:

* Premium
* Modern
* Minimal
* Trustworthy
* Professional
* Easy to use from a phone at a job site

Take inspiration from:

* QuickBooks Time
* Clockify
* Homebase
* ADP Time

Do not copy another product directly.

⸻

Success Criteria

The completed feature must be usable by a real employee every day to:

* Clock in
* Clock out
* Track breaks
* Correct forgotten or inaccurate punches
* Automatically close forgotten shifts at midnight
* Review flagged days
* Calculate estimated pay
* Review weekly timesheets
* Submit reports to a manager
* Email professional timesheets
* Export printable employment records
* View a complete history of all changes

A forgotten clock-out must never allow a shift to run indefinitely or silently inflate payroll calculations.

⸻

Required Workflow Before Coding

Before writing any implementation code:

1. Inspect the existing SchmidtAdmin project architecture.
2. Inspect the existing Supabase schema, authentication, Row Level Security policies, server actions, API routes, Resend integration, design system, and deployment configuration.
3. Identify the existing user and administrator role model.
4. Determine the most reliable supported method for server-side midnight automatic clock-out.
5. Produce a detailed implementation plan.
6. Identify every new or modified:
    * Page
    * Component
    * Hook
    * Service
    * API route
    * Server action
    * Database migration
    * Row Level Security policy
    * Email template
    * Scheduled job
    * Test
7. Explain the data model and time-zone strategy.
8. Explain how submitted timesheets remain historically accurate after edits.
9. Explain how automatic clock-out is made idempotent.
10. Identify any architectural risks or unresolved decisions.
11. Do not begin implementation until the plan is complete and internally consistent.
