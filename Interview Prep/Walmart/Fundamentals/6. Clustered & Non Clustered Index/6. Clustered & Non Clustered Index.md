# 6. Clustered vs non-clustered index – what actually changes in storage and lookup?

*   **Clustered Index**
> [!IMPORTANT]
> The table's data is physically sorted and stored in the order of the index key.
*   Defines the physical storage order of the entire table.
*   **One clustered index per table** (the "table" is the clustered index).
*   Leaf nodes of the index contain the actual data pages.
*   Fast for range queries (e.g., `BETWEEN`, `>` , `<`) on the index key.

*   **Non-Clustered Index**
> [!NOTE]
> A separate structure from the table data, like an index in a book.
*   Creates a copy of the indexed columns and a pointer to the data.
*   **Multiple non-clustered indexes per table** are possible.
*   Leaf nodes contain the indexed columns + a pointer (the clustered key).
*   Efficient for finding specific values or small ranges.

**Storage Differences**

*   **Clustered Index Storage**
> [!TIP]
> Think of a clustered index as a dictionary sorted alphabetically.
*   The data rows themselves are stored in the B-tree's leaf level.
*   No separate storage is needed for the data – the index *is* the data.
*   The logical and physical order of the rows is the same.

*   **Non-Clustered Index Storage**
> [!CAUTION]
> A non-clustered index is a separate copy, requiring additional disk space.
*   Creates a separate B-tree structure.
*   The leaf nodes contain:
    *   The values from the indexed columns.
    *   A pointer (row locator) back to the main data.
    *   If a clustered index exists, the pointer is the clustered key.
    *   If no clustered index exists (a heap), the pointer is a Row ID (RID).

**Lookup Process Differences**

*   **Clustered Index Seek**
*   The database navigates the B-tree to find the starting row directly.
*   Since data is physically sorted, subsequent rows are read sequentially.
*   This is the most efficient data retrieval method for the indexed range.

*   **Non-Clustered Index Seek with Key Lookup**
> [!IMPORTANT]
> A seek on a non-clustered index often requires a second, costly step.
1.  **Seek:** The database finds the value in the non-clustered index B-tree.
2.  **Key Lookup (RID Lookup for a heap):** It uses the pointer (clustered key) to retrieve the full row from the main table data.
*   This lookup is expensive if many rows are returned, as it involves random I/O.

**Visual Analogy**

*   **Clustered Index = A Phone Book**
*   Sorted by last name, then first name.
*   To find all people named "Smith," you find the first entry and read sequentially.
*   The data (name, address, phone number) is right there on the page.

*   **Non-Clustered Index = A Book's Index**
*   Sorted by keyword in the back of the book.
*   To find information about "database indexing," you look it up in the index.
*   The index gives you a page number (a pointer), and you must then turn to that page to get the full text (the data).
