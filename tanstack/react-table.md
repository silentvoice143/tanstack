# TanStack Table – Server-Side Pagination & Sorting

This project demonstrates how to implement server-side pagination and sorting using **TanStack Table (React Table v8)**. 



## 📌 Features
* **Fake Database:** Local generation of 100 users.
* **Simulated API:** Mock call with an 800ms delay.
* **Server-side Sorting:** Logic handled by the API simulator.
* **Server-side Pagination:** Data slicing handled on the "server."
* **Controlled State:** Syncing React state with table internals.
* **Loading State:** Feedback during data fetching.
* **Nested Headers:** Support for grouped column layouts.

---

## 🏗 Tech Stack
* React & TypeScript
* TanStack Table v8
* Tailwind CSS

---

## 1️⃣ Data Model
```typescript
export type User = {
  id: string;
  firstName: string;
  lastName: string;
  age: number;
  visits: number;
  status: "active" | "inactive";
};
```
`
2️⃣ Fake Database Generator

```
export function generateUsers(count: number): User[] {
  const firstNames = ["Tanner", "Kevin", "Aman", "Rahul", "John", "Emma"];
  const lastNames = ["Linsley", "Vandy", "Sharma", "Smith", "Doe"];

  return Array.from({ length: count }, () => ({
    id: crypto.randomUUID(),
    firstName: firstNames[Math.floor(Math.random() * firstNames.length)],
    lastName: lastNames[Math.floor(Math.random() * lastNames.length)],
    age: Math.floor(Math.random() * 50) + 18,
    visits: Math.floor(Math.random() * 1000),
    status: Math.random() > 0.5 ? "active" : "inactive",
  }));
}

const FAKE_DB = generateUsers(100);
```
3️⃣ Fake API (Server Simulation)
Types

```
type FetchParams = {
  pageIndex: number;
  pageSize: number;
  sorting: SortingState;
};

type FetchResponse = {
  rows: User[];
  total: number;
};
```
**Server-Side Logic**

```
function fetchUsersFromApi({
  pageIndex,
  pageSize,
  sorting,
}: FetchParams): Promise<FetchResponse> {
  return new Promise((resolve) => {
    setTimeout(() => {
      let data = [...FAKE_DB];

      // Server-side sorting
      if (sorting.length > 0) {
        const { id, desc } = sorting[0];

        data.sort((a: any, b: any) => {
          if (a[id] < b[id]) return desc ? 1 : -1;
          if (a[id] > b[id]) return desc ? -1 : 1;
          return 0;
        });
      }

      const total = data.length;

      // Server-side pagination
      const start = pageIndex * pageSize;
      const end = start + pageSize;

      resolve({
        rows: data.slice(start, end),
        total,
      });
    }, 800);
  });
}
```
----
4️⃣ Table State Management

```
const [data, setData] = useState<User[]>([]);
const [loading, setLoading] = useState(false);

const [sorting, setSorting] = useState<SortingState>([]);
const [pagination, setPagination] = useState({
  pageIndex: 0,
  pageSize: 20,
});

const [totalRows, setTotalRows] = useState(0);
```
5️⃣ Column Definition (ColumnDef)

```
const columns = useMemo<ColumnDef<User>[]>(
  () => [
    { accessorKey: "firstName", header: "First Name" },
    { accessorKey: "lastName", header: "Last Name" },
    { accessorKey: "age", header: "Age" },
    { accessorKey: "visits", header: "Visits" },
    {
      accessorKey: "status",
      header: "Status",
      cell: ({ getValue }) => (
        <span
          className={`px-2 py-1 rounded text-white ${
            getValue() === "active" ? "bg-green-500" : "bg-red-500"
          }`}
        >
          {getValue<string>()}
        </span>
      ),
    },
  ],
  []
);
```
6️⃣ Fetch Data on State Change

```
useEffect(() => {
  setLoading(true);

  fetchUsersFromApi({
    pageIndex: pagination.pageIndex,
    pageSize: pagination.pageSize,
    sorting,
  }).then((res) => {
    setData(res.rows);
    setTotalRows(res.total);
    setLoading(false);
  });
}, [pagination, sorting]);
```
7️⃣ Server-Side Controlled Table (Manual Mode)

```
const table = useReactTable({
  data,
  columns,

  state: {
    sorting,
    pagination,
  },

  onSortingChange: setSorting,
  onPaginationChange: setPagination,

  manualSorting: true,
  manualPagination: true,

  pageCount: Math.ceil(totalRows / pagination.pageSize),

  getCoreRowModel: getCoreRowModel(),
});
```
Why Manual Mode?

Because:

- Sorting is handled by the API
- Pagination is handled by the API
- The table only renders returned rows

8️⃣ Client-Side Alternative (Automatic Mode)

```
const table = useReactTable({
  data,
  columns,
  state: { sorting },
  onSortingChange: setSorting,

  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  getPaginationRowModel: getPaginationRowModel(),

  initialState: {
    pagination: {
      pageSize: 20,
    },
  },
});
```
In Client Mode:

- Sorting happens internally
- Pagination slices data internally
- No API required

9️⃣ Header Rendering

```
{table.getHeaderGroups().map((headerGroup) => (
  <tr key={headerGroup.id}>
    {headerGroup.headers.map((header) => (
      <th
        key={header.id}
        onClick={header.column.getToggleSortingHandler()}
        className="cursor-pointer p-3 border-b text-left"
      >
        {flexRender(
          header.column.columnDef.header,
          header.getContext()
        )}

        {{
          asc: " 🔼",
          desc: " 🔽",
        }[header.column.getIsSorted() as string] ?? null}
      </th>
    ))}
  </tr>
))}
```
🔟 Nested Header Example

```
const columns = [
  {
    header: "User Info",
    columns: [
      { accessorKey: "name", header: "Name" },
      { accessorKey: "email", header: "Email" },
    ],
  },
  {
    header: "Details",
    columns: [
      { accessorKey: "age", header: "Age" },
      { accessorKey: "salary", header: "Salary" },
    ],
  },
];
```
### 👤 User Data Structure

| User Info | | Details | |
| :--- | :--- | :--- | :--- |
| **Name** | **Email** | **Age** | **Salary** |
| — | — | — | — |

---


---

## 📊 Client vs. Server Mode Comparison

When building data tables (like TanStack Table or Flutter DataGrids), choosing the right processing mode is critical for app performance.

| Feature | Client-Side | Server-Side |
| :--- | :--- | :--- |
| **Sorting** | `getSortedRowModel` | `manualSorting` |
| **Pagination** | `getPaginationRowModel` | `manualPagination` |
| **Page Count** | **Automatic** | Must provide `pageCount` |
| **Large Data** | ❌ Not ideal | ✅ Recommended |

---

### 🛠 Implementation Logic





Use it when:

- Data comes from backend
- You have large datasets
- You want scalable architecture
- Sorting must happen in database
🧠 Summary

- ColumnDef controls how columns render
- manualPagination enables controlled mode
- manualSorting disables internal sorting
- getHeaderGroups() enables nested headers
- Client mode is easier but not scalable``

--- 

## Sorting 

### 1. Client- Side Sorting

The table sorts the data internally.

Setup Sorting State
```
import { useState } from "react";
import {
  useReactTable,
  getCoreRowModel,
  getSortedRowModel,
  SortingState,
} from "@tanstack/react-table";

const [sorting, setSorting] = useState<SortingState>([]);
```

Create Table Instance

```
const table = useReactTable({
  data,
  columns,

  state: {
    sorting,
  },

  onSortingChange: setSorting,

  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
});
```

Enable Sorting in Columns

```
const columns = [
  {
    accessorKey: "name",
    header: "Name",
  },
  {
    accessorKey: "age",
    header: "Age",
  },
];
```

Add Click Sorting to Header

```
<th
  onClick={header.column.getToggleSortingHandler()}
>
  {flexRender(header.column.columnDef.header, header.getContext())}

  {{
    asc: " 🔼",
    desc: " 🔽",
  }[header.column.getIsSorted() as string] ?? null}
</th>
```

Sorting Order Cycle

Each click cycles through:

not sorted → asc → desc → not sorted

### 2. Server-Side Sorting

Used when data comes from an API.

Example: fetch sorted data from backend.

```
const table = useReactTable({
  data,
  columns,

  state: {
    sorting,
  },

  onSortingChange: setSorting,

  manualSorting: true,

  getCoreRowModel: getCoreRowModel(),
});
```

Send Sorting to API

```
useEffect(() => {
  fetchUsers({
    sortBy: sorting[0]?.id,
    order: sorting[0]?.desc ? "desc" : "asc",
  });
}, [sorting]);
```

### 3. Multi-Column Sorting

Hold Shift while clicking headers: Need to cliling shift else evertime the sorting will reset.

Example state of sorting:
```
[
  { id: "name", desc: false },
  { id: "age", desc: true }
]
```

### 4. Disable Sorting for a Column

```
{
  accessorKey: "actions",
  header: "Actions",
  enableSorting: false,
}
```

### 5. Custom Sorting Function

```
type Task = {
  id: number
  title: string
  status: "Pending" | "In Progress" | "Completed"
}

const data: Task[] = [
  { id: 1, title: "Task 1", status: "Completed" },
  { id: 2, title: "Task 2", status: "Pending" },
  { id: 3, title: "Task 3", status: "In Progress" },
]
```

Column Definition:

```
const columns = [
  {
    accessorKey: "title",
    header: "Title",
  },
  {
    accessorKey: "status",
    header: "Status",
    sortingFn: (rowA, rowB, columnId) => {
      const order = {
        Pending: 1,
        "In Progress": 2,
        Completed: 3,
      }

      const a = order[rowA.getValue(columnId)]
      const b = order[rowB.getValue(columnId)]

      return a - b
    },
  },
]
```

Result:

| Title | Status |
| :--- | :--- |
| Task 2 | Pending |
| Task 3 | In Progress |
| Task 1 | Completed |
---

## Column Visiblity

TanStack Table controls visibility through a state object.
```
const [columnVisibility, setColumnVisibility] = useState({})
```

Structure:

```
{
  email: false,
  salary: false
}
```

Connect State to Table

```
const table = useReactTable({
  data,
  columns,

  state: {
    columnVisibility,
  },

  onColumnVisibilityChange: setColumnVisibility,

  getCoreRowModel: getCoreRowModel(),
})
```

Now the table automatically reacts when visibility changes.

**Toggle Column Visibility**

Hide a column:
```
table.getColumn("email")?.toggleVisibility(true)
```

Show a column
```
table.getColumn("email")?.toggleVisibility(true)
```

Automatically
```
table.getColumn("email")?.toggleVisibility()
```
---

## Column Ordering

TanStack stores the order as an array of column IDs.

```
const [columnOrder, setColumnOrder] = useState([
  "name",
  "email",
  "role",
  "salary"
])
```

Structure:

```
[
  "name",
  "role",
  "email",
  "salary"
]
```

```
Name | Role | Email | Salary
```

### Connect Column Order to Table

```
const table = useReactTable({
  data,
  columns,

  state: {
    columnOrder,
  },

  onColumnOrderChange: setColumnOrder,

  getCoreRowModel: getCoreRowModel(),
})
```
### Change Column Order Programmatically

```
setColumnOrder([
  "name",
  "salary",
  "email",
  "role"
])
```

Result:

```
Name | Salary | Email | Role
```

