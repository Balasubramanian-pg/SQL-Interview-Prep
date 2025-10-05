## TRIM, LTRIM, RTRIM Cheat Sheet Table

| Function | Purpose | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **`TRIM`** | Removes characters from **both the beginning and end** of a string. | `$TRIM( [ [LEADING | TRAILING | BOTH] [characters] FROM ] string)$` | The ANSI standard function; allows specifying *which* characters to remove (default is space). |
| **`LTRIM`** | Removes characters only from the **beginning (Left)** of a string. | `$LTRIM(string [, characters])$` | Useful for removing leading spaces, prefixes, or unwanted symbols. |
| **`RTRIM`** | Removes characters only from the **end (Right)** of a string. | `$RTRIM(string [, characters])$` | Useful for removing trailing spaces, line breaks, or suffixes. |
| **Simple Space Removal** | The most common use: removing leading/trailing **whitespace**. | `$TRIM(string)$` | Most databases use this simple syntax for removing only spaces. |

## TRIM, LTRIM, RTRIM Mini Playbook (Realistic Queries)

These snippets illustrate the common use cases for trimming functions, emphasizing general syntax and platform variations.

