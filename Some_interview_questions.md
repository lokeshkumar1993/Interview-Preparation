# .NET + Angular + SQL Interview Questions (5–7 Years Experience)

---

# 1. C# Output-Based Questions

---

## Question 1 — Short Circuit Evaluation

### Code

```csharp
class Program
{
    static bool SomeMethod1()
    {
        Console.WriteLine("Method 1");
        return false;
    }

    static bool SomeMethod2()
    {
        Console.WriteLine("Method 2");
        return true;
    }

    static void Main()
    {
        if (SomeMethod1() && SomeMethod2())
        {
            Console.WriteLine("Inside If");
        }
    }
}
```

---

## Expected Output

```text
Method 1
```

---

## Answer

`&&` is a short-circuit operator.

Since `SomeMethod1()` returns `false`, the second condition is never evaluated.

So:
- `SomeMethod1()` executes
- `SomeMethod2()` does NOT execute
- `if` block does not execute

---

## Follow-Up Questions

### Difference between `&&` and `&`

| Operator | Behavior |
|---|---|
| `&&` | Short-circuit |
| `&` | Evaluates both conditions |

---

### What happens if `&` is used?

Output:

```text
Method 1
Method 2
```

---

---

# Question 2 — var Keyword

## Code

```csharp
var a = 1;
a = "Hello";

Console.WriteLine(a);
```

---

## Expected Answer

Compilation Error

---

## Reason

`var` is resolved at compile time.

So:

```csharp
var a = 1;
```

becomes:

```csharp
int a = 1;
```

Later assigning a string causes a type mismatch.

---

## Follow-Up

### Difference between `var`, `dynamic`, and `object`

| Type | Compile Time | Runtime |
|---|---|---|
| var | Strongly typed | Compile time |
| dynamic | Resolved at runtime | Runtime |
| object | Base type of all | Needs casting |

---

---

# Question 3 — Frequency Count Using LINQ

## Code

```csharp
string[] arr = { "apple", "banana", "orange", "orange", "apple" };
```

---

## Expected Output

```text
apple - 2
banana - 1
orange - 2
```

---

## Solution

```csharp
var result = arr
    .GroupBy(x => x)
    .Select(g => new
    {
        Name = g.Key,
        Count = g.Count()
    });

foreach (var item in result)
{
    Console.WriteLine($"{item.Name} - {item.Count}");
}
```

---

## Answer

`GroupBy()` groups identical values together.

`Count()` counts elements inside each group.

---

## Follow-Up Questions

- Can this be done without LINQ?
- Difference between `GroupBy` and `ToLookup`
- Time complexity?

---

---

# Question 4 — Generics

## Code

```csharp
Pair<int> pairs = new Pair<int>(1, 2);

List<int> listData = new List<int>();
```

---

## Answer

Generics provide:
- Type safety
- Better performance
- Reusability

---

## Why `List<int>` over `ArrayList`?

### Advantages

- No boxing/unboxing
- Compile-time type checking
- Better performance

---

## Follow-Up

### What is Boxing?

Converting value type to reference type.

```csharp
int a = 10;
object obj = a;
```

---

### What is Unboxing?

Converting object back to value type.

```csharp
int b = (int)obj;
```

---

# 2. SOLID Principles + Clean Code

---

# Question 5 — SOLID Principle Violation

## Code

```csharp
public class ReportService
{
    public string Generate()
    {
        var data = File.ReadAllText("input.json");

        var sender = new SmtpClient("smtp.example.com");

        sender.Send(
            "noreply@example.com",
            "ops@example.com",
            "report",
            data);

        return "OK";
    }
}
```

---

## Problems in Code

### Violations

- SRP violation
- Tight coupling
- Hardcoded values
- Not testable
- No exception handling

---

## Why SRP is Violated?

This class:
- Reads file
- Generates report
- Sends email

Multiple responsibilities.

---

## Better Design

Use:
- Dependency Injection
- Interfaces
- Configuration
- Logging

---

## Better Design Example

```csharp
public interface IEmailService
{
    Task SendAsync(string message);
}
```

---

# Question 6 — Resource Leak

## Code

```csharp
public class FileExporter
{
    private StreamWriter _writer;

    public void Export(string path, IEnumerable<string> lines)
    {
        _writer = new StreamWriter(File.Open(path, FileMode.Create));

        foreach (var line in lines)
            _writer.WriteLine(line);
    }
}
```

---

## Problem

`StreamWriter` is never disposed.

This causes:
- Memory leak
- File lock issue

---

## Correct Solution

```csharp
public void Export(string path, IEnumerable<string> lines)
{
    using var writer =
        new StreamWriter(File.Open(path, FileMode.Create));

    foreach (var line in lines)
    {
        writer.WriteLine(line);
    }
}
```

---

## Follow-Up Questions

### Difference between `using` statement and `using` declaration

#### Traditional

```csharp
using(var writer = new StreamWriter(...))
{
}
```

#### Modern

```csharp
using var writer = new StreamWriter(...);
```

---

# 3. LINQ Questions

---

# Question 7 — FirstOrDefault vs SingleOrDefault

---

## Difference

| Method | Behavior |
|---|---|
| FirstOrDefault | Returns first matching record |
| SingleOrDefault | Expects exactly one record |

---

## Example

```csharp
var result = users.FirstOrDefault(x => x.Id == 1);
```

Returns first match.

---

```csharp
var result = users.SingleOrDefault(x => x.Id == 1);
```

Throws exception if multiple records exist.

---

## Exception Case

```text
Sequence contains more than one matching element
```

---

## When to Use?

| Method | Use Case |
|---|---|
| FirstOrDefault | Multiple records possible |
| SingleOrDefault | Exactly one record expected |

---

# Question 8 — Select vs SelectMany

## Code

```csharp
var list = new List<List<int>>
{
    new List<int> {1,2},
    new List<int> {3,4}
};
```

---

## Using Select

```csharp
var result = list.Select(x => x);
```

### Output Type

```text
IEnumerable<List<int>>
```

Nested collection.

---

## Using SelectMany

```csharp
var result = list.SelectMany(x => x);
```

### Output

```text
1 2 3 4
```

Flattened collection.

---

## Real-Time Example

Used in:
- EF Core joins
- Nested collection flattening

---

# 4. Async/Await Questions

---

# Question 9 — Sequential vs Parallel Execution

## Code

```csharp
public async Task<IActionResult> GetUserDetails()
{
    var a = await GetUserAsync();
    var b = await GetOrderAsync();
    var c = await GetPaymentAsync();

    return Ok(new { a, b, c });
}
```

---

## Answer

This executes sequentially.

Flow:
1. Wait for User
2. Wait for Order
3. Wait for Payment

Total time = sum of all operations.

---

## Optimized Version

```csharp
public async Task<IActionResult> GetUserDetails()
{
    var userTask = GetUserAsync();
    var orderTask = GetOrderAsync();
    var paymentTask = GetPaymentAsync();

    await Task.WhenAll(
        userTask,
        orderTask,
        paymentTask);

    return Ok(new
    {
        a = userTask.Result,
        b = orderTask.Result,
        c = paymentTask.Result
    });
}
```

---

## Benefits

- Parallel execution
- Faster response time

---

## Follow-Up Questions

### Difference between Async and Multithreading

| Async | Multithreading |
|---|---|
| Non-blocking | Multiple threads |
| IO bound | CPU bound |

---

### Why avoid `.Result`?

Can cause:
- Deadlock
- Thread blocking

---

# 5. Interface Questions

---

# Question 10 — Interface Basics

## Code

```csharp
interface IMyInterface
{
    void Display();

    void NewDisplay();
}
```

---

## Questions to Ask

- Can interface have implementation?
- Difference between abstract class and interface?
- Can interface contain variables?

---

## Answers

### Can interface have implementation?

Before C# 8 → No

After C# 8 → Default implementations allowed.

---

## Interface vs Abstract Class

| Interface | Abstract Class |
|---|---|
| Multiple inheritance | Single inheritance |
| No constructor | Can have constructor |
| Pure contract | Partial implementation |

---

# 6. ASP.NET Core Questions

---

# Question 11 — Middleware

## Ask Candidate

What is middleware?

---

## Answer

Middleware is a component in ASP.NET Core request pipeline.

Each middleware:
- Handles request
- Calls next middleware
- Can modify response

---

## Custom Middleware Example

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        Console.WriteLine("Before Request");

        await _next(context);

        Console.WriteLine("After Response");
    }
}
```

---

## Follow-Up

### Difference between `Use`, `Run`, and `Map`

| Method | Behavior |
|---|---|
| Use | Calls next middleware |
| Run | Terminates pipeline |
| Map | Branches pipeline |

---

# Question 12 — Dependency Injection Lifetimes

---

## Types

| Lifetime | Behavior |
|---|---|
| Singleton | One instance entire app |
| Scoped | One instance per request |
| Transient | New instance every time |

---

## Important Scenario

### What happens if Scoped service injected into Singleton?

Problem:
- Scoped service lives shorter
- Singleton lives entire app

This causes runtime exception.

---

# Question 13 — HttpContext

## Questions

- What is HttpContext?
- How to access headers?
- How to access current user?

---

## Example

```csharp
var user = HttpContext.User.Identity.Name;

var token = HttpContext.Request.Headers["Authorization"];
```

---

# 7. REST API + Security

---

# Question 14 — OAuth vs JWT

---

## JWT

JWT is a token format.

Contains:
- Header
- Payload
- Signature

---

## OAuth

OAuth is an authorization framework.

Used for:
- Login with Google
- Third-party authorization

---

## JWT Structure

```text
xxxxx.yyyyy.zzzzz
```

---

## Follow-Up Questions

- What is Refresh Token?
- Why JWT is stateless?
- Where should JWT be stored?

---

# Question 15 — API Performance Optimization

## Expected Answers

- Caching
- Pagination
- Compression
- Async operations
- Database indexing
- Avoid unnecessary joins

---

# 8. EF Core Questions

---

# Question 16 — Tracking vs NoTracking

---

## Tracking

EF tracks entity changes.

```csharp
context.Users.ToList();
```

---

## NoTracking

Improves read performance.

```csharp
context.Users
    .AsNoTracking()
    .ToList();
```

---

## When to Use?

Use `AsNoTracking()` for:
- Read-only queries
- Large datasets

---

# Question 17 — Lazy Loading vs Eager Loading

---

## Lazy Loading

Loads related data when accessed.

---

## Eager Loading

Loads related data immediately.

```csharp
context.Users.Include(x => x.Orders);
```

---

## Problem with Lazy Loading

Can cause:
- N+1 query problem
- Performance issues

---

# 9. Angular Questions

---

# Question 18 — Angular Output Question

## Code

```typescript
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

console.log(3);
```

---

## Expected Output

```text
1
3
2
```

---

## Answer

`setTimeout` goes into callback queue.

JavaScript executes synchronous code first.

---

## Follow-Up

- Explain Event Loop
- Microtask vs Macrotask

---

# Question 19 — Observable vs Promise

---

## Observable

- Multiple values
- Lazy execution
- Can cancel subscription

---

## Promise

- Single value
- Executes immediately
- Cannot cancel

---

## Example

```typescript
this.http.get('/api/users')
.subscribe(data => {
    console.log(data);
});
```

---

# Question 20 — Subject vs BehaviorSubject

---

## Difference

| Subject | BehaviorSubject |
|---|---|
| No initial value | Requires initial value |
| No latest value stored | Stores latest value |

---

## Example

```typescript
const subject = new BehaviorSubject(0);
```

---

# 10. SQL Questions

---

# Question 21 — Second Highest Salary

## Query

```sql
SELECT MAX(Salary)
FROM Employees
WHERE Salary <
(
    SELECT MAX(Salary)
    FROM Employees
);
```

---

# Question 22 — Find Duplicate Records

## Query

```sql
SELECT Name, COUNT(*)
FROM Employees
GROUP BY Name
HAVING COUNT(*) > 1;
```

---

# Question 23 — DELETE vs TRUNCATE vs DROP

| Command | Behavior |
|---|---|
| DELETE | Removes rows |
| TRUNCATE | Removes all rows quickly |
| DROP | Removes table completely |

---

# Question 24 — GROUP BY vs DISTINCT

| GROUP BY | DISTINCT |
|---|---|
| Aggregation possible | No aggregation |
| Groups data | Removes duplicates |

---

# Question 25 — Clustered vs Non-Clustered Index

---

## Clustered Index

- Physically sorts table data
- One per table

---

## Non-Clustered Index

- Separate structure
- Multiple allowed

---

# 11. Scenario-Based Questions

---

# Question 26 — API Response Suddenly Slow

## Expected Answer

Check:
- SQL query performance
- Logs
- CPU/memory
- API dependencies
- Database indexes
- External services

---

# Question 27 — Export Millions of Records

## Expected Answer

Use:
- Streaming
- Batch processing
- Pagination
- Background jobs

Avoid loading all data into memory.

---

# Question 28 — Memory Usage Increasing

## Possible Reasons

- Memory leaks
- Undisposed objects
- Static references
- Caching issue

---

# 12. Rapid Fire Questions

---

# Question 29 — IEnumerable vs IQueryable

| IEnumerable | IQueryable |
|---|---|
| In-memory | Database query |
| Executes immediately | Deferred execution |

---

# Question 30 — const vs readonly

| const | readonly |
|---|---|
| Compile-time constant | Runtime constant |
| Static by default | Instance allowed |

---

# Question 31 — string vs StringBuilder

| string | StringBuilder |
|---|---|
| Immutable | Mutable |
| Slower for repeated changes | Faster |

---

# Question 32 — ref vs out

| ref | out |
|---|---|
| Must initialize before passing | No need to initialize |

---

# Question 33 — virtual vs abstract

| virtual | abstract |
|---|---|
| Optional override | Mandatory override |

---

# Question 34 — Stack vs Heap

| Stack | Heap |
|---|---|
| Stores value types | Stores reference types |
| Faster | Slower |

---

# Question 35 — Delegates vs Events

| Delegate | Event |
|---|---|
| Function pointer | Restricted delegate |

---

# 13. Coding Questions

---

# Question 36 — Reverse Words

## Input

```text
I love dotnet
```

---

## Output

```text
dotnet love I
```

---

## Solution

```csharp
string input = "I love dotnet";

var result = string.Join(
    " ",
    input.Split(' ').Reverse());

Console.WriteLine(result);
```

---

# Question 37 — Palindrome

## Solution

```csharp
string str = "madam";

bool isPalindrome = true;

for(int i = 0; i < str.Length / 2; i++)
{
    if(str[i] != str[str.Length - 1 - i])
    {
        isPalindrome = false;
        break;
    }
}

Console.WriteLine(isPalindrome);
```

---

# Question 38 — Remove Duplicate Characters

## Input

```text
aabbccdde
```

---

## Output

```text
abcde
```

---

## Solution

```csharp
string input = "aabbccdde";

string result = "";

foreach(char c in input)
{
    if(!result.Contains(c))
    {
        result += c;
    }
}

Console.WriteLine(result);
```

---

# Question 39 — Top 3 Salaries

## Query

```sql
SELECT TOP 3 Salary
FROM Employees
ORDER BY Salary DESC;
```

---

# Question 40 — Department Wise Employee Count

## Query

```sql
SELECT DepartmentId,
       COUNT(*) AS EmployeeCount
FROM Employees
GROUP BY DepartmentId;
```

---

# Additional LINQ Interview Questions

---

# Question 41 — Deferred Execution in LINQ

## Code

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4 };

var result = numbers.Where(x => x > 2);

numbers.Add(5);

foreach (var item in result)
{
    Console.WriteLine(item);
}
```

---

## Expected Output

```text
3
4
5
```

---

## Answer

LINQ uses deferred execution.

The query is executed only when iterated.

Since `5` was added before iteration, it appears in output.

---

## Follow-Up Questions

### How to force immediate execution?

```csharp
var result = numbers.Where(x => x > 2).ToList();
```

---

# Question 42 — Immediate Execution

## Code

```csharp
List<int> numbers = new List<int> { 1, 2, 3 };

var result = numbers.Where(x => x > 1).ToList();

numbers.Add(4);

foreach (var item in result)
{
    Console.WriteLine(item);
}
```

---

## Expected Output

```text
2
3
```

---

## Answer

`ToList()` forces immediate execution.

Query result is stored immediately.

Later modifications do not affect output.

---

# Question 43 — OrderBy vs ThenBy

## Code

```csharp
var employees = new List<Employee>
{
    new Employee { Name = "John", Department = "IT" },
    new Employee { Name = "Adam", Department = "HR" },
    new Employee { Name = "Sam", Department = "IT" }
};

var result = employees
    .OrderBy(x => x.Department)
    .ThenBy(x => x.Name);
```

---

## Answer

- `OrderBy()` performs primary sorting
- `ThenBy()` performs secondary sorting

---

## Expected Order

```text
HR - Adam
IT - John
IT - Sam
```

---

# Question 44 — Any() vs All()

## Code

```csharp
List<int> numbers = new List<int> { 2, 4, 6, 8 };

bool result1 = numbers.Any(x => x > 5);

bool result2 = numbers.All(x => x > 5);

Console.WriteLine(result1);
Console.WriteLine(result2);
```

---

## Expected Output

```text
True
False
```

---

## Answer

### Any()

Returns true if at least one element matches condition.

### All()

Returns true only if all elements satisfy condition.

---

# Question 45 — Count() vs LongCount()

---

## Answer

| Method | Return Type |
|---|---|
| Count() | int |
| LongCount() | long |

---

## When to Use LongCount?

Use when collection size may exceed:
- 2,147,483,647 records

---

# Question 46 — Skip() and Take()

## Code

```csharp
List<int> numbers = Enumerable.Range(1, 10).ToList();

var result = numbers.Skip(3).Take(4);

foreach (var item in result)
{
    Console.WriteLine(item);
}
```

---

## Expected Output

```text
4
5
6
7
```

---

## Answer

### Skip(3)

Skips first 3 records.

### Take(4)

Takes next 4 records.

---

## Real-Time Usage

Used in:
- Pagination
- Infinite scrolling

---

# Question 47 — Distinct()

## Code

```csharp
List<int> numbers = new List<int>
{
    1, 1, 2, 3, 3, 4
};

var result = numbers.Distinct();

foreach(var item in result)
{
    Console.WriteLine(item);
}
```

---

## Expected Output

```text
1
2
3
4
```

---

## Answer

`Distinct()` removes duplicate values.

---

## Follow-Up Question

### How does Distinct work internally?

Uses hashing internally.

---

# Question 48 — GroupBy with Multiple Columns

## Code

```csharp
var employees = new List<Employee>
{
    new Employee { Department = "IT", City = "Delhi" },
    new Employee { Department = "IT", City = "Delhi" },
    new Employee { Department = "HR", City = "Mumbai" }
};

var result = employees
    .GroupBy(x => new
    {
        x.Department,
        x.City
    });
```

---

## Answer

Groups data using composite keys.

---

## Real-Time Example

Used for:
- Reporting
- Dashboard aggregation

---

# Question 49 — ToDictionary()

## Code

```csharp
var employees = new List<Employee>
{
    new Employee { Id = 1, Name = "John" },
    new Employee { Id = 2, Name = "Sam" }
};

var dict = employees.ToDictionary(x => x.Id);

Console.WriteLine(dict[1].Name);
```

---

## Expected Output

```text
John
```

---

## Answer

`ToDictionary()` converts collection into key-value pairs.

---

## Important Follow-Up

### What happens if duplicate keys exist?

Throws exception.

---

# Question 50 — DefaultIfEmpty()

## Code

```csharp
List<int> numbers = new List<int>();

var result = numbers.DefaultIfEmpty(100);

foreach(var item in result)
{
    Console.WriteLine(item);
}
```

---

## Expected Output

```text
100
```

---

## Answer

Returns default value if collection is empty.

---

# Question 51 — Union vs Concat

## Code

```csharp
List<int> list1 = new List<int> { 1, 2, 3 };
List<int> list2 = new List<int> { 3, 4, 5 };

var unionResult = list1.Union(list2);

var concatResult = list1.Concat(list2);
```

---

## Answer

| Method | Behavior |
|---|---|
| Union | Removes duplicates |
| Concat | Keeps duplicates |

---

## Expected Results

### Union

```text
1 2 3 4 5
```

### Concat

```text
1 2 3 3 4 5
```

---

# Question 52 — First vs FirstOrDefault

---

## Difference

| Method | Empty Collection |
|---|---|
| First | Throws exception |
| FirstOrDefault | Returns default value |

---

## Example

```csharp
var result = list.First();
```

Throws exception if no records.

---

```csharp
var result = list.FirstOrDefault();
```

Returns:
- null for reference type
- 0 for int

---

# Question 53 — Join in LINQ

## Code

```csharp
var employees = new List<Employee>
{
    new Employee { Id = 1, Name = "John", DepartmentId = 1 }
};

var departments = new List<Department>
{
    new Department { Id = 1, Name = "IT" }
};

var result = employees.Join(
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

## Answer

`Join()` combines collections based on matching keys.

---

## SQL Equivalent

```sql
INNER JOIN
```

---

# Question 54 — Left Join in LINQ

## Solution

```csharp
var result =
    from e in employees
    join d in departments
    on e.DepartmentId equals d.Id
    into deptGroup
    from dg in deptGroup.DefaultIfEmpty()
    select new
    {
        Employee = e.Name,
        Department = dg?.Name
    };
```

---

## Answer

`DefaultIfEmpty()` is used to perform LEFT JOIN.

---

# Question 55 — Deferred Execution Performance Issue

## Code

```csharp
var result = employees.Where(x => x.Salary > 50000);

foreach(var item in result)
{
    Console.WriteLine(item.Name);
}

foreach(var item in result)
{
    Console.WriteLine(item.Name);
}
```

---

## Question

What is the issue?

---

## Answer

Query executes multiple times.

This may cause:
- Performance issue
- Multiple DB calls

---

## Better Solution

```csharp
var result = employees
    .Where(x => x.Salary > 50000)
    .ToList();
```

---

# Question 56 — IQueryable vs IEnumerable

---

## Difference

| IEnumerable | IQueryable |
|---|---|
| Executes in memory | Executes in database |
| LINQ to Objects | LINQ to Entities |

---

## Example

### IEnumerable

```csharp
var data = context.Users.ToList()
    .Where(x => x.Age > 18);
```

Filtering happens in memory.

---

### IQueryable

```csharp
var data = context.Users
    .Where(x => x.Age > 18);
```

Filtering happens in database.

---

## Important Follow-Up

Which is better for performance?

Answer:
- IQueryable for database filtering

---

# Question 57 — Projection in LINQ

## Code

```csharp
var result = employees.Select(x => new
{
    x.Name,
    x.Salary
});
```

---

## Answer

Projection means selecting only required fields.

---

## Benefits

- Improves performance
- Reduces memory usage
- Reduces DB load

---

# Question 58 — Aggregate Function

## Code

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4 };

var sum = numbers.Aggregate((a, b) => a + b);

Console.WriteLine(sum);
```

---

## Expected Output

```text
10
```

---

## Answer

`Aggregate()` performs custom accumulation operation.

---

# Question 59 — SequenceEqual()

## Code

```csharp
List<int> list1 = new List<int> { 1, 2, 3 };
List<int> list2 = new List<int> { 1, 2, 3 };

bool result = list1.SequenceEqual(list2);

Console.WriteLine(result);
```

---

## Expected Output

```text
True
```

---

## Answer

Checks:
- Order
- Values

Both must match.

---

# Question 60 — Chunk()

## Code

```csharp
var numbers = Enumerable.Range(1, 10);

var chunks = numbers.Chunk(3);

foreach(var chunk in chunks)
{
    Console.WriteLine(string.Join(",", chunk));
}
```

---

## Expected Output

```text
1,2,3
4,5,6
7,8,9
10
```

---

## Answer

`Chunk()` splits collection into smaller groups.

---

## Real-Time Usage

Used in:
- Batch processing
- Bulk operations
- Parallel processing

---


# Interview Evaluation Guidelines

---

# Strong Candidate Indicators

- Explains WHY behind output
- Understands async deeply
- Knows performance implications
- Understands SOLID principles
- Can optimize queries
- Gives practical examples

---

# Weak Candidate Indicators

- Memorized answers only
- Cannot explain execution flow
- Weak syntax knowledge
- Confuses LINQ methods
- Poor async understanding
- No real-world examples

---
