Alright Balu, here’s a clean, sharp README. Straightforward, practical, Gen Z skepticism sprinkled in, no fluff. All links intact.

---

# SQL Interview Preparation – Pattern Recognition (1–9)

This collection covers the core SQL patterns interviewers love to spring on you. Each topic focuses on a specific analytical pattern that shows whether a candidate can think structurally rather than just write queries. These nine files form a tight prep set that builds real muscle for data work.

## Files

### [1. Finding Top N Records per Group](1.%20Finding%20Top%20N%20Records%20per%20Group.md)

This pattern tests whether someone understands window functions and ranking logic. The goal is to extract the highest-value rows within each category without resorting to messy subqueries. Mastering this demonstrates control over partitioning, ordering, and row-level precision.

### [2. Calculating Running Totals-Cumulative Sums](2.%20Calculating%20Running%20Totals-Cumulative%20Sums.md)

Running totals are the backbone of time-series and financial reporting. Interviewers use this to check if you can compute cumulative metrics while keeping order stable. It separates people who know `SUM() OVER` from those who brute-force with correlated queries.

### [3. Identifying Gaps in Sequences](3.%20Identifying%20Gaps%20in%20Sequences.md)

Gaps in sequences reveal missing events, broken pipelines, or dirty data. This pattern shows whether you can compare each row to its predecessor and detect structural anomalies. It’s a clever test of logical thinking with ordering functions.

### [4. Finding Islands of Consecutive Values](4.%20Finding%20Islands%20of%20Consecutive%20Values.md)

“Islands and gaps” questions are a classic stress test. They check if you can group continuous ranges of values without explicit grouping keys. It basically evaluates whether you understand difference-based grouping and row-number alignment.

### [5. Calculating Moving Averages](5.%20Calculating%20Moving%20Averages.md)

Moving averages are used in forecasting, smoothing volatility, and building trend analysis. Interviewers ask this to see if you know how to apply frame clauses like `ROWS BETWEEN X PRECEDING AND CURRENT ROW` for rolling calculations.

### [6. Pivoting Data (Rows to Columns)](6.%20Pivoting%20Data%20%28Rows%20to%20Columns%29.md)

Pivoting checks your ability to reshape datasets for reporting. It’s all about transforming long format inputs into wide outputs. This pattern reflects whether you can navigate aggregate pivots or dynamic SQL pivot generation.

### [7. Unpivoting Data (Columns to Rows)](7.%20Unpivoting%20Data%20%28Columns%20to%20Rows%29.md)

The opposite of pivoting, unpivoting tests whether you can normalize overly wide data back into row-oriented structure. It’s useful when cleaning messy tables that store attributes horizontally instead of vertically.

### [8. Finding Duplicate Records](8.%20Finding%20Duplicate%20Records.md)

Duplicate detection comes up constantly in ETL and data quality work. Interviewers ask this to verify that you can group, count, and isolate data inconsistencies without destroying the original detail. It’s basic but widely mis-handled.

### [9. Navigating Hierarchical Data](9.%20Navigating%20Hierarchical%20Data.md)

Hierarchical queries check if you can manage parent-child relationships using recursion. These show up in org charts, category trees, and nested structures. Candidates who fail this usually avoid CTE recursion, which is exactly why interviewers love it.

## Repository
