# TimeClock


I’d push AG toward something that feels like a commercial product (QuickBooks Time, Clockify, Homebase), not a simple start/stop timer.

⸻

Phase X: Professional Employee Time Clock & Payroll

Objective

Implement a production-quality Time Clock module for SchmidtAdmin that allows employees to accurately track work hours, calculate estimated gross pay, and submit professional timesheets to management.

This is not a stopwatch.

This should feel like a polished SaaS payroll/time tracking product.

⸻

Requirements

1. New Time Clock section

Add a new navigation item:

* Time Clock

The page should feel premium and match the rest of SchmidtAdmin.

⸻

2. Employee Settings

Create persistent employee settings stored in Supabase.

Fields:

* Employee Name
* Hourly Pay Rate
* Boss / Manager Email
* Time Zone
* Company Name
* Overtime Enabled
* Overtime Threshold (default 40 hrs)
* Overtime Multiplier (default 1.5x)

These settings should only need configured once.

⸻

3. Clock In / Clock Out

Large professional interface.

Display:

Current Status

• Working
or

• Off Duty

Large live timer

Examples

Working for

3h 42m

Buttons

Clock In

Clock Out

Button states should intelligently switch.

⸻

4. Accurate Time Tracking

Never trust the browser clock.

Use server timestamps stored in Supabase.

Each entry stores:

* clock_in
* clock_out
* break minutes
* project
* notes
* created_at

⸻

5. Work Details

When clocking in allow:

Project (optional)

Examples

General Construction

Office

Concrete

Roofing

Cleanup

Customer Address (optional)

Work Notes

⸻

6. Break Tracking

Support

Start Break

End Break

Track:

* break duration
* total paid hours
* total unpaid time

⸻

7. Today’s Summary Card

Display

Clock In

Clock Out

Breaks

Hours Worked

Estimated Earnings

⸻

8. Weekly Timesheet

Professional table.

Columns

Date

Clock In

Clock Out

Break

Hours

Hourly Rate

Daily Gross

Status

Totals at bottom.

⸻

9. Payroll Calculations

Automatically calculate

Regular Hours

Overtime Hours

Gross Pay

Example

36 Regular Hours

5 Overtime Hours

Total Gross

$1,187.50

Everything updates live.

⸻

10. Professional Dashboard Cards

Cards:

Hours Today

Hours This Week

Estimated Gross Pay

Current Status

Last Clock In

Weekly Progress

⸻

11. Email Timesheet

Button

Submit Timesheet

Generate a professional HTML email using the existing Resend infrastructure already used elsewhere in the application.

Email should include

Employee Name

Company

Pay Period

Daily Entries

Weekly Total

Hourly Rate

Regular Hours

Overtime Hours

Estimated Gross Pay

Notes

The email should look professional enough to send directly to an employer.

⸻

12. PDF Export

Generate printable timesheets.

Include:

Company

Employee

Pay Period

Signature Line

Manager Signature

Totals

Professional formatting.

⸻

13. Audit Trail

Never silently edit time.

If a user edits:

Clock In

Clock Out

Breaks

Notes

Store:

Old Value

New Value

Reason

Timestamp

This provides accountability.

⸻

14. Manual Corrections

Allow requesting corrections.

Example

Forgot to clock out yesterday.

Manager can approve.

⸻

15. Offline Support

If internet is unavailable:

Store entries locally.

Automatically sync when online.

Never lose punches.

⸻

16. Mobile Experience

The interface should feel like a native mobile time clock.

Large buttons.

Minimal typing.

One-tap clock in/out.

Responsive layouts.

⸻

17. Database

Create Supabase tables:

employee_settings

time_entries

timesheets

time_entry_audit

Use Row Level Security.

Authenticated users only.

⸻

18. Code Quality

Follow existing SchmidtAdmin architecture.

Keep components modular.

Separate:

UI

Business Logic

Payroll calculations

Database

Email templates

Reusable hooks

⸻

19. Design

Use the existing SchmidtAdmin design language.

Premium.

Modern.

Minimal.

No toy stopwatch UI.

Think:

QuickBooks Time

Clockify

Homebase

ADP Time

⸻

Success Criteria

The finished feature should feel like a real commercial employee time tracking system, not a demo.

It should be capable of being used by a real employee every day to:

* Clock in
* Clock out
* Track breaks
* Calculate pay
* Review timesheets
* Submit professional reports to a manager
* Export printable records

Before writing any code:

1. Inspect the existing project architecture.
2. Reuse the existing Supabase client, authentication, and Resend email infrastructure.
3. Produce an implementation plan identifying where each new component, hook, API route, database migration, and UI screen will live.
4. Do not begin implementation until the plan is complete and internally consistent.
