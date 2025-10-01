## Cheat Sheet: Ranking Functions in Interviews

| Interview Question Pattern                                                                   | What They’re Really Testing                             | Best Function to Use                                        | Why                                            |
| -------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------- |
| **Find the 2nd highest salary** (all employees at that salary)                               | Do you know how to handle ties                          | `DENSE_RANK()`                                              | Groups ties together → clean “2nd highest” set |
| **Find the 2nd highest salary** (pick exactly one employee, even if multiple tied)           | Can you force uniqueness                                | `ROW_NUMBER()`                                              | Each row is unique; you can pick one           |
| **Show employees ranked by salary with gaps in ranking numbers**                             | Do you know `RANK()`’s “skipping” behavior              | `RANK()`                                                    | If two are tied at 2, next one is 4            |
| **Show employees ranked by salary without gaps in ranking numbers**                          | Spot the difference between `RANK()` and `DENSE_RANK()` | `DENSE_RANK()`                                              | Compacts ranking numbers (1,2,2,3)             |
| **Top N employees** (e.g., top 3 people, even if same salary)                                | Do you know how to cut rows                             | `ROW_NUMBER()`                                              | Forces only N rows, ignores ties               |
| **Top N salaries** (e.g., top 3 distinct salary levels)                                      | Do you know how to cut groups                           | `DENSE_RANK()`                                              | Includes ties, but only N salary bands         |
| **Nth row in a partition** (e.g., each department’s highest-paid employee)                   | Partitioning knowledge                                  | `ROW_NUMBER() OVER(PARTITION BY dept ORDER BY salary DESC)` | Picks one per group cleanly                    |
| **Top N per partition** (e.g., top 2 salaries per department, all ties allowed)              | Both partitioning + ties                                | `DENSE_RANK() OVER(PARTITION BY dept …)`                    | Keeps all employees with rank ≤ N              |
| **Find duplicates** (e.g., which employees have same salary)                                 | Can you detect repetition                               | `ROW_NUMBER()` with `> 1` in filter                         | Easy duplicate flag                            |
| **Competition / Olympics style ranking** (gold, silver, bronze, but skipping places if tied) | Spot when to use `RANK()`                               | `RANK()`                                                    | Mimics competition ranking logic               |

### Quick Decision Tree in Your Head

* Do you need **unique rows**? → `ROW_NUMBER()`
* Do you want **ties grouped, no gaps**? → `DENSE_RANK()`
* Do you want **ties grouped, with gaps**? → `RANK()`
