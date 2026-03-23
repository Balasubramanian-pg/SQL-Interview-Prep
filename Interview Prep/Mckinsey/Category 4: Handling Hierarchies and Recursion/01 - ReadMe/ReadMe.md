# Handling Hierarchies and Recursion

*   **Recursive CTE Fundamentals**
> [!IMPORTANT]
> Recursive CTEs allow querying hierarchical data with unknown depth.
*   Consists of anchor member (base case) and recursive member.
*   Must include termination condition to prevent infinite loops.
*   Essential for organizational charts, bill of materials, category trees.

**Basic Recursive CTE Structure**

```sql
WITH RECURSIVE hierarchy_path AS (
    -- Anchor member: Top-level nodes
    SELECT 
        id,
        name,
        parent_id,
        1 AS level,
        name AS path
    FROM employees 
    WHERE parent_id IS NULL  -- Root nodes
    
    UNION ALL
    
    -- Recursive member: Child nodes
    SELECT 
        e.id,
        e.name,
        e.parent_id,
        hp.level + 1,
        hp.path || ' -> ' || e.name
    FROM employees e
    INNER JOIN hierarchy_path hp ON e.parent_id = hp.id
    -- Termination condition (optional but recommended)
    WHERE hp.level < 10  -- Prevent infinite recursion
)
SELECT * FROM hierarchy_path;
```

**Organizational Hierarchy**

*   **Employee Management Chain**
> [!TIP]
> Use recursive CTEs to build complete reporting structures.
```sql
WITH RECURSIVE org_chart AS (
    -- Anchor: CEO and top executives (no manager)
    SELECT 
        employee_id,
        employee_name,
        title,
        manager_id,
        0 AS level,
        employee_name AS management_chain,
        ARRAY[employee_id] AS path_ids
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Subordinates
    SELECT 
        e.employee_id,
        e.employee_name,
        e.title,
        e.manager_id,
        oc.level + 1,
        oc.management_chain || ' → ' || e.employee_name,
        oc.path_ids || e.employee_id
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
    WHERE oc.level < 8  -- Reasonable depth limit
      AND e.employee_id != ALL(oc.path_ids)  -- Prevent cycles
)
SELECT 
    employee_id,
    employee_name,
    title,
    level,
    management_chain,
    -- Indentation for visual hierarchy
    LPAD('', level * 4, ' ') || employee_name AS indented_name
FROM org_chart
ORDER BY path_ids;
```

**Product Category Hierarchy**

*   **Nested Categories with Aggregation**
```sql
WITH RECURSIVE category_tree AS (
    -- Anchor: Root categories
    SELECT 
        category_id,
        category_name,
        parent_category_id,
        1 AS depth,
        category_name AS full_path,
        category_id AS root_category_id
    FROM product_categories 
    WHERE parent_category_id IS NULL
    
    UNION ALL
    
    -- Recursive: Subcategories
    SELECT 
        pc.category_id,
        pc.category_name,
        pc.parent_category_id,
        ct.depth + 1,
        ct.full_path || ' > ' || pc.category_name,
        ct.root_category_id
    FROM product_categories pc
    INNER JOIN category_tree ct ON pc.parent_category_id = ct.category_id
    WHERE depth < 5
)
SELECT 
    ct.category_id,
    ct.category_name,
    ct.depth,
    ct.full_path,
    -- Aggregate product counts across hierarchy
    COUNT(p.product_id) AS product_count,
    SUM(p.price) AS total_value
FROM category_tree ct
LEFT JOIN products p ON ct.category_id = p.category_id
GROUP BY ct.category_id, ct.category_name, ct.depth, ct.full_path
ORDER BY ct.full_path;
```

**Bill of Materials (BOM) Explosion**

*   **Multi-Level Product Assembly**
> [!CAUTION]
> BOM queries require careful quantity roll-up calculations.
```sql
WITH RECURSIVE bom_explosion AS (
    -- Anchor: Final product
    SELECT 
        component_id,
        component_name,
        parent_component_id,
        quantity,
        1 AS level,
        component_name AS assembly_path,
        quantity AS total_quantity
    FROM bill_of_materials 
    WHERE parent_component_id IS NULL  -- Final product
      AND product_id = 123
    
    UNION ALL
    
    -- Recursive: Subcomponents
    SELECT 
        bom.component_id,
        bom.component_name,
        bom.parent_component_id,
        bom.quantity,
        be.level + 1,
        be.assembly_path || ' -> ' || bom.component_name,
        be.total_quantity * bom.quantity AS total_quantity
    FROM bill_of_materials bom
    INNER JOIN bom_explosion be ON bom.parent_component_id = be.component_id
    WHERE be.level < 10  -- Prevent infinite recursion in cyclic BOMs
)
SELECT 
    component_id,
    component_name,
    level,
    assembly_path,
    quantity AS unit_quantity,
    total_quantity AS total_required,
    -- Calculate total cost
    total_quantity * c.unit_cost AS total_cost
FROM bom_explosion be
JOIN components c ON be.component_id = c.component_id
ORDER BY level, assembly_path;
```

**Path Finding in Graphs**

*   **Network and Route Analysis**
```sql
WITH RECURSIVE path_finder AS (
    -- Anchor: Starting node
    SELECT 
        node_id,
        node_name,
        ARRAY[node_id] AS path,
        0 AS total_distance,
        FALSE AS cycle_detected
    FROM network_nodes 
    WHERE node_id = 'A'  -- Start node
    
    UNION ALL
    
    -- Recursive: Connected nodes
    SELECT 
        nn.node_id,
        nn.node_name,
        pf.path || nn.node_id,
        pf.total_distance + e.distance,
        nn.node_id = ANY(pf.path)  -- Cycle detection
    FROM network_nodes nn
    JOIN network_edges e ON nn.node_id = e.to_node_id
    JOIN path_finder pf ON e.from_node_id = pf.node_id
    WHERE NOT pf.cycle_detected
      AND pf.total_distance < 1000  -- Distance limit
      AND array_length(pf.path, 1) < 10  -- Path length limit
)
SELECT 
    node_id,
    node_name,
    path,
    total_distance,
    array_to_string(path, ' -> ') AS path_string
FROM path_finder
WHERE node_id = 'Z'  -- Target node
ORDER BY total_distance
LIMIT 5;  -- Top 5 shortest paths
```

**Hierarchical Aggregation**

*   **Roll-up Summaries with Recursion**
```sql
WITH RECURSIVE sales_hierarchy AS (
    -- Anchor: Leaf-level sales territories
    SELECT 
        territory_id,
        territory_name,
        parent_territory_id,
        territory_id AS leaf_territory,
        sales_amount,
        1 AS level
    FROM sales_territories st
    JOIN sales_data sd ON st.territory_id = sd.territory_id
    WHERE NOT EXISTS (
        SELECT 1 FROM sales_territories sub 
        WHERE sub.parent_territory_id = st.territory_id
    )  -- Leaf nodes only
    
    UNION ALL
    
    -- Recursive: Roll up to parent territories
    SELECT 
        st.territory_id,
        st.territory_name,
        st.parent_territory_id,
        sh.leaf_territory,
        sh.sales_amount,
        sh.level + 1
    FROM sales_territories st
    INNER JOIN sales_hierarchy sh ON st.territory_id = sh.parent_territory_id
)
SELECT 
    territory_id,
    territory_name,
    level,
    SUM(sales_amount) AS rolled_up_sales,
    COUNT(DISTINCT leaf_territory) AS leaf_territories_count
FROM sales_hierarchy
GROUP BY territory_id, territory_name, level
ORDER BY level, territory_id;
```

**Cycle Detection and Prevention**

*   **Handling Circular References**
> [!WARNING]
> Always implement cycle detection in recursive queries.
```sql
WITH RECURSIVE employee_hierarchy AS (
    SELECT 
        employee_id,
        employee_name,
        manager_id,
        1 AS level,
        ARRAY[employee_id] AS path,
        FALSE AS has_cycle
    FROM employees 
    WHERE employee_id = 1  -- Starting employee
    
    UNION ALL
    
    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        eh.level + 1,
        eh.path || e.employee_id,
        e.employee_id = ANY(eh.path)  -- Cycle detection
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
    WHERE NOT eh.has_cycle
      AND eh.level < 15  -- Safety depth limit
)
SELECT 
    employee_id,
    employee_name,
    level,
    has_cycle,
    array_to_string(path, ' -> ') AS management_chain
FROM employee_hierarchy
WHERE NOT has_cycle
ORDER BY level, employee_id;
```

**Multiple Root Handling**

*   **Forest of Hierarchies**
```sql
WITH RECURSIVE multi_root_hierarchy AS (
    -- Anchor: All root nodes
    SELECT 
        node_id,
        node_name,
        parent_id,
        node_id AS root_id,
        1 AS level,
        ARRAY[node_id] AS path,
        node_name AS full_path
    FROM hierarchy_table 
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive: Child nodes
    SELECT 
        ht.node_id,
        ht.node_name,
        ht.parent_id,
        mrh.root_id,
        mrh.level + 1,
        mrh.path || ht.node_id,
        mrh.full_path || ' / ' || ht.node_name
    FROM hierarchy_table ht
    INNER JOIN multi_root_hierarchy mrh ON ht.parent_id = mrh.node_id
    WHERE mrh.level < 10
)
SELECT 
    root_id,
    node_id,
    node_name,
    level,
    full_path,
    -- Count nodes per root
    COUNT(*) OVER (PARTITION BY root_id) AS tree_size,
    -- Level-based indentation
    RPAD('', (level - 1) * 3, ' ') || node_name AS visual_hierarchy
FROM multi_root_hierarchy
ORDER BY root_id, path;
```

**Materialized Path Pattern**

*   **Alternative to Recursive CTEs**
> [!NOTE]
> Materialized paths store the hierarchy as a string for faster querying.
```sql
-- Querying materialized path (e.g., '1.5.12.42')
SELECT 
    category_id,
    category_name,
    materialized_path,
    -- Extract depth from path
    array_length(string_to_array(materialized_path, '.'), 1) AS depth,
    -- Extract parent path
    CASE 
        WHEN string_to_array(materialized_path, '.') > 1 
        THEN array_to_string(
            string_to_array(materialized_path, '.')[:array_length(string_to_array(materialized_path, '.'), 1)-1], 
            '.'
        )
        ELSE NULL
    END AS parent_path
FROM product_categories
-- Find all descendants of a category
WHERE materialized_path LIKE '1.5.%'
ORDER BY materialized_path;

-- Find all ancestors of a category
SELECT *
FROM product_categories
WHERE '1.5.12.42' LIKE materialized_path || '%'
ORDER BY materialized_path;
```

**Performance Optimization**

*   **Efficient Hierarchy Queries**
```sql
-- Optimized recursive query with early filtering
WITH RECURSIVE optimized_hierarchy AS (
    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        1 AS level,
        ARRAY[e.employee_id] AS path
    FROM employees e
    WHERE e.department_id = 5  -- Early filtering
      AND e.manager_id IS NULL
    
    UNION ALL
    
    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        oh.level + 1,
        oh.path || e.employee_id
    FROM employees e
    INNER JOIN optimized_hierarchy oh ON e.manager_id = oh.employee_id
    WHERE e.department_id = 5  -- Maintain filter in recursion
      AND oh.level < 6
)
SELECT 
    employee_id,
    employee_name,
    level,
    array_to_string(path, ' -> ') AS hierarchy_path
FROM optimized_hierarchy
-- Create index on: (department_id, manager_id) for performance
ORDER BY path;
```

**Advanced Use Case: Permission Inheritance**

*   **Role-Based Access Control Hierarchy**
```sql
WITH RECURSIVE permission_inheritance AS (
    -- Anchor: Base roles with direct permissions
    SELECT 
        r.role_id,
        r.role_name,
        r.parent_role_id,
        p.permission_id,
        p.permission_name,
        1 AS inheritance_level,
        ARRAY[r.role_id] AS role_chain
    FROM roles r
    JOIN role_permissions rp ON r.role_id = rp.role_id
    JOIN permissions p ON rp.permission_id = p.permission_id
    
    UNION ALL
    
    -- Recursive: Inherit permissions from parent roles
    SELECT 
        cr.role_id,
        cr.role_name,
        cr.parent_role_id,
        pi.permission_id,
        pi.permission_name,
        pi.inheritance_level + 1,
        pi.role_chain || cr.role_id
    FROM roles cr
    INNER JOIN permission_inheritance pi ON cr.parent_role_id = pi.role_id
    WHERE pi.inheritance_level < 8
      AND cr.role_id != ALL(pi.role_chain)  -- Prevent cycles
)
SELECT DISTINCT
    role_id,
    role_name,
    permission_id,
    permission_name,
    inheritance_level,
    array_to_string(role_chain, ' ← ') AS inheritance_chain
FROM permission_inheritance
ORDER BY role_id, inheritance_level, permission_id;
```
