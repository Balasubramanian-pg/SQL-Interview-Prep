Dealing with **date range overlaps** is a common and complex task in SQL, especially when scheduling, booking, or validating time periods. The fundamental challenge is that a date range $[A_{start}, A_{end}]$ and a second range $[B_{start}, B_{end}]$ can overlap in four primary ways, all of which must be captured by a single, concise condition.

## The Non-Overlap Condition (The Simplest Approach) 🧠

The easiest and most reliable way to check for an overlap is to determine when **two ranges *do not* overlap** and then negate that condition.

Two date ranges, Range A and Range B, **do not overlap** if one ends before the other starts.

1.  Range A ends before Range B starts: $$\text{A}_{\text{end}} < \text{B}_{\text{start}}$$2.  Range B ends before Range A starts:$$\text{B}_{\text{end}} < \text{A}_{\text{start}}$$

### The Master Formula for Overlap

A date range overlap **occurs** if it is **NOT** the case that they *don't* overlap.

$$\text{Overlap} \iff \text{NOT} ( (\text{A}_{\text{end}} < \text{B}_{\text{start}}) \text{ OR } (\text{B}_{\text{end}} < \text{A}_{\text{start}}) )$$

In SQL, this translates to the most common and robust condition:

```sql
WHERE NOT (
    A.end_date < B.start_date OR 
    B.end_date < A.start_date
);
```

-----

## The Direct Overlap Condition (Alternative)

A slightly less intuitive but also powerful condition checks directly for the overlap by ensuring the latest start date is before the earliest end date.

1.  The later of the two start dates ($\text{MAX}(\text{A}_{\text{start}}, \text{B}_{\text{start}})$)
2.  Must be less than the earlier of the two end dates ($\text{MIN}(\text{A}_{\text{end}}, \text{B}_{\text{end}})$)

### The Direct Overlap Formula

$$\text{Overlap} \iff (\text{A}_{\text{start}} < \text{B}_{\text{end}}) \text{ AND } (\text{B}_{\text{start}} < \text{A}_{\text{end}})$$

In SQL:

```sql
WHERE 
    A.start_date < B.end_date AND 
    B.start_date < A.end_date;
```

-----

## Date Range Overlaps Mini Playbook

This playbook demonstrates finding conflicts in a table of existing bookings (`Bookings`) when trying to insert a new, potential booking (represented by the variables `@New_Start` and `@New_End`).

### Scenario: Checking for a Booking Conflict

We want to find all existing bookings that conflict with a new request from '2025-11-15' to '2025-11-20'.

```sql
-- Variables representing the new proposed booking range
DECLARE @New_Start DATE = '2025-11-15';
DECLARE @New_End DATE = '2025-11-20';

SELECT
    b.booking_id,
    b.start_date AS existing_start,
    b.end_date AS existing_end
FROM
    Bookings AS b
WHERE
    -- Using the robust NOT (No Overlap) logic:
    -- Find existing bookings where it is NOT true that the ranges DON'T overlap.
    NOT (
        b.end_date < @New_Start OR -- Existing booking ends before new one starts
        b.start_date > @New_End    -- Existing booking starts after new one ends (Note: Use > for start_date check)
    )
    AND b.room_id = 4; -- Filter by the specific resource
```

### Important Boundary Consideration 🛑

The choice between using $\mathbf{<}$ (strictly less than) or $\mathbf{\le}$ (less than or equal to) depends on whether the boundary dates are **inclusive or exclusive**.

  * **Inclusive (Standard):** If a booking on Day 1-3 **can** be immediately followed by a booking on Day 3-5 (meaning the end date is inclusive), use the formulas as shown ($\mathbf{<}$).
  * **Exclusive/Consecutive:** If a booking on Day 1-3 means the resource is free *starting* on Day 4, then your boundary conditions might need slight adjustment. The standard conditions above handle **inclusive** boundaries correctly, where the overlap check is based on the actual time units.
