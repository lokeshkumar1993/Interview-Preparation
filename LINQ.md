# LINQ Practice Interview Session (Complete + Deep Explanation)

This document contains a full guided LINQ interview practice session with:
- Questions
- Solutions
- Fixes
- Deep explanations
- Interview mental models

---

# Question 1: Filtering + Select

### Task
Return names of employees whose age > 30.

### Solution
```csharp
var res = employees.Where(e => e.Age > 30)
                   .Select(e => e.Name);
```

---

# Question 2: Filtering + Ordering

### Task
Age > 25, ordered by age descending.

### Solution
```csharp
var res = employees.Where(e => e.Age > 25)
                   .OrderByDescending(e => e.Age)
                   .Select(e => e.Name);
```

---

# Concept: OrderBy vs MinBy

```csharp
employees.OrderBy(e => e.Age).First()
```

- O(n log n) due to sorting

```csharp
employees.MinBy(e => e.Age)
```

- O(n) single scan
- More efficient

---

# Question 3: GroupBy Count

### Task
Count employees per department

### Solution
```csharp
var res = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        DeptName = g.Key,
        Count = g.Count()
    });
```

---

# Question 4: GroupBy Average

### Task
Average salary per department

### Solution
```csharp
var res = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        AverageSalary = g.Average(e => e.Salary)
    });
```

---

# Question 5: Inner Join

## Method Syntax
```csharp
var res = employees.Join(
    departments,
    e => e.DepartmentId,
    d => d.Id,
    (e, d) => new
    {
        EmployeeName = e.Name,
        DepartmentName = d.Name
    });
```

---

## Query Syntax
```csharp
var res =
    from e in employees
    join d in departments
        on e.DepartmentId equals d.Id
    select new
    {
        EmployeeName = e.Name,
        DepartmentName = d.Name
    };
```

---

# Question 6: LEFT JOIN (Advanced)

## Goal
Return all employees even if department is missing.

---

# Select vs SelectMany

## Select
- 1 → 1 mapping
- No flattening

## SelectMany
- 1 → many → flatten

---

## Example
```csharp
new List<List<int>>
{
    new List<int>{1,2},
    new List<int>{3}
}
.SelectMany(x => x);
```

Output:
```
1, 2, 3
```

---

# LEFT JOIN USING GROUPJOIN + SELECTMANY

---

# Step 1: GroupJoin

```csharp
employees.GroupJoin(
    departments,
    e => e.DepartmentId,
    d => d.Id,
    (e, deps) => new { e, deps }
);
```

This produces:

Employee → List<Departments>

Example:
```
Alice → [HR]
Bob   → []
Charlie → [IT]
```

So now we have:

> Each employee mapped to a collection of departments.

---

# Step 2: SelectMany (IMPORTANT CORE CONCEPT)

We use:

```csharp
.SelectMany(
    x => x.deps.DefaultIfEmpty(),
    (x, d) => new
    {
        EmployeeName = x.e.Name,
        DepartmentName = d != null ? d.Name : "Unknown"
    }
);
```

---

# 🔥 SelectMany parameters (important concept)

You are using this overload:

```csharp
SelectMany(
    collectionSelector,
    resultSelector
)
```

---

# 🧠 Parameter 1: collectionSelector

```csharp
x => x.deps.DefaultIfEmpty()
```

### What it means:

> “For each employee-group object x, give me the collection I should expand into rows.”

---

### Break it down:

- `x` = `{ e, deps }`
- `x.deps` = list of departments for that employee
- `DefaultIfEmpty()` ensures:
  - if empty → return `[null]` instead of `[]`

---

### Think of it like:

```
Alice -> [HR]
Bob   -> []
```

becomes:

```
Alice -> [HR]
Bob   -> [null]
```

✔ This is what enables LEFT JOIN behavior.

---

# 🧠 Parameter 2: resultSelector

```csharp
(x, d) => new
{
    EmployeeName = x.e.Name,
    DepartmentName = d != null ? d.Name : "Unknown"
}
```

---

### What it means:

> “Now take:
- the original group object (x)
- each item from the flattened collection (d)
and produce the final output row.”

---

### Mapping:

| Variable | Meaning |
|----------|--------|
| x | original grouped object (employee + departments list) |
| d | single department (or null) |
| result | final output row |

---

# 🔥 Full mental model

## Step 1: GroupJoin
```
Employee → List<Departments>
```

## Step 2: SelectMany input
```
{
  e = Employee,
  deps = [Department1, Department2]
}
```

## Step 3: SelectMany transformation
```
flatten into rows
```

---

# 🧠 One-line interview explanation

If asked:

> What do the two parameters of SelectMany do?

### Answer:

> The first parameter selects and flattens the inner collection (with DefaultIfEmpty for left join behavior), and the second parameter defines how each flattened element combines with the original outer element to produce the final result.

---

# Final Left Join Code

```csharp
var res = employees
    .GroupJoin(
        departments,
        e => e.DepartmentId,
        d => d.Id,
        (e, deps) => new { e, deps }
    )
    .SelectMany(
        x => x.deps.DefaultIfEmpty(),
        (x, d) => new
        {
            EmployeeName = x.e.Name,
            DepartmentName = d != null ? d.Name : "Unknown"
        }
    );
```

---

# Final Takeaways

- Where → filtering
- Select → projection
- OrderByDescending → sorting
- GroupBy → grouping
- Join → inner join
- GroupJoin → grouped join
- SelectMany → flattening engine
- Left Join = GroupJoin + SelectMany + DefaultIfEmpty

---

# End of Session
