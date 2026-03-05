# TanStack Table (v8) — Complete Reference

> Server-side pagination, sorting, row selection, filtering, column pinning, column resizing, global search, and more — with React & TypeScript.

---

## Table of Contents

1. [Data Model & Setup](#1-data-model--setup)
2. [Fake Database & API Simulator](#2-fake-database--api-simulator)
3. [Table State Management](#3-table-state-management)
4. [Column Definitions](#4-column-definitions)
5. [Fetching Data on State Change](#5-fetching-data-on-state-change)
6. [Server-Side vs Client-Side Mode](#6-server-side-vs-client-side-mode)
7. [Sorting](#7-sorting)
8. [Column Visibility](#8-column-visibility)
9. [Column Ordering](#9-column-ordering)
10. [Rendering: Headers & Rows](#10-rendering-headers--rows)
11. [Row Selection](#11-row-selection)
12. [Column Filtering](#12-column-filtering)
13. [Global Search / Filtering](#13-global-search--filtering)
14. [Column Pinning](#14-column-pinning)
15. [Column Resizing](#15-column-resizing)
16. [Row Expanding (Subrows)](#16-row-expanding-subrows)
17. [Row Grouping](#17-row-grouping)
18. [Pagination Controls (UI)](#18-pagination-controls-ui)
19. [Virtualization (Large Lists)](#19-virtualization-large-lists)

---

## 1. Data Model & Setup

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

**Tech Stack:** React + TypeScript · TanStack Table v8 · Tailwind CSS

---

## 2. Fake Database & API Simulator

### Generate Fake Data

```typescript
export function generateUsers(count: number): User[] {
  const firstNames = ["Tanner", "Kevin", "Aman", "Rahul", "John", "Emma"];
  const lastNames  = ["Linsley", "Vandy", "Sharma", "Smith", "Doe"];

  return Array.from({ length: count }, () => ({
    id:        crypto.randomUUID(),
    firstName: firstNames[Math.floor(Math.random() * firstNames.length)],
    lastName:  lastNames[Math.floor(Math.random() * lastNames.length)],
    age:       Math.floor(Math.random() * 50) + 18,
    visits:    Math.floor(Math.random() * 1000),
    status:    Math.random() > 0.5 ? "active" : "inactive",
  }));
}

const FAKE_DB = generateUsers(100);
```

### Simulated API (800ms delay)

```typescript
type FetchParams = {
  pageIndex: number;
  pageSize:  number;
  sorting:   SortingState;
};

type FetchResponse = {
  rows:  User[];
  total: number;
};

function fetchUsersFromApi({ pageIndex, pageSize, sorting }: FetchParams): Promise<FetchResponse> {
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

      // Server-side pagination
      const start = pageIndex * pageSize;
      resolve({
        rows:  data.slice(start, start + pageSize),
        total: data.length,
      });
    }, 800);
  });
}
```

---

## 3. Table State Management

```typescript
const [data,       setData]       = useState<User[]>([]);
const [loading,    setLoading]    = useState(false);
const [totalRows,  setTotalRows]  = useState(0);

const [sorting,    setSorting]    = useState<SortingState>([]);
const [pagination, setPagination] = useState({ pageIndex: 0, pageSize: 20 });
```

---

## 4. Column Definitions

### Basic Columns

```typescript
const columns = useMemo<ColumnDef<User>[]>(() => [
  { accessorKey: "firstName", header: "First Name" },
  { accessorKey: "lastName",  header: "Last Name"  },
  { accessorKey: "age",       header: "Age"        },
  { accessorKey: "visits",    header: "Visits"     },
  {
    accessorKey: "status",
    header: "Status",
    cell: ({ getValue }) => (
      <span className={`px-2 py-1 rounded text-white ${
        getValue() === "active" ? "bg-green-500" : "bg-red-500"
      }`}>
        {getValue<string>()}
      </span>
    ),
  },
], []);
```

### Nested / Grouped Headers

```typescript
const columns = [
  {
    header: "User Info",
    columns: [
      { accessorKey: "name",  header: "Name"  },
      { accessorKey: "email", header: "Email" },
    ],
  },
  {
    header: "Details",
    columns: [
      { accessorKey: "age",    header: "Age"    },
      { accessorKey: "salary", header: "Salary" },
    ],
  },
];
```

Renders as:

| User Info | | Details | |
|---|---|---|---|
| **Name** | **Email** | **Age** | **Salary** |

---

## 5. Fetching Data on State Change

```typescript
useEffect(() => {
  setLoading(true);

  fetchUsersFromApi({
    pageIndex: pagination.pageIndex,
    pageSize:  pagination.pageSize,
    sorting,
  }).then((res) => {
    setData(res.rows);
    setTotalRows(res.total);
    setLoading(false);
  });
}, [pagination, sorting]);
```

---

## 6. Server-Side vs Client-Side Mode

| Feature | Client-Side | Server-Side |
|---|---|---|
| Sorting | `getSortedRowModel()` | `manualSorting: true` |
| Pagination | `getPaginationRowModel()` | `manualPagination: true` |
| Page Count | Automatic | Must provide `pageCount` |
| Large Datasets | ❌ Not ideal | ✅ Recommended |

### Server-Side (Manual) Mode

```typescript
const table = useReactTable({
  data,
  columns,
  state:               { sorting, pagination },
  onSortingChange:     setSorting,
  onPaginationChange:  setPagination,
  manualSorting:       true,
  manualPagination:    true,
  pageCount:           Math.ceil(totalRows / pagination.pageSize),
  getCoreRowModel:     getCoreRowModel(),
});
```

> **Use when:** data comes from a backend, datasets are large, or sorting must happen at the database level.

### Client-Side (Automatic) Mode

```typescript
const table = useReactTable({
  data,
  columns,
  state:                  { sorting },
  onSortingChange:        setSorting,
  getCoreRowModel:        getCoreRowModel(),
  getSortedRowModel:      getSortedRowModel(),
  getPaginationRowModel:  getPaginationRowModel(),
  initialState:           { pagination: { pageSize: 20 } },
});
```

> **Use when:** the full dataset is already in memory and scalability isn't a concern.

---

## 7. Sorting

### Client-Side Sorting

```typescript
const [sorting, setSorting] = useState<SortingState>([]);

const table = useReactTable({
  data,
  columns,
  state:             { sorting },
  onSortingChange:   setSorting,
  getCoreRowModel:   getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
});
```

### Server-Side Sorting

```typescript
const table = useReactTable({
  data,
  columns,
  state:           { sorting },
  onSortingChange: setSorting,
  manualSorting:   true,
  getCoreRowModel: getCoreRowModel(),
});

// Send sort params to the API
useEffect(() => {
  fetchUsers({
    sortBy: sorting[0]?.id,
    order:  sorting[0]?.desc ? "desc" : "asc",
  });
}, [sorting]);
```

### Multi-Column Sorting

Hold **Shift** while clicking headers to sort by multiple columns simultaneously.

```typescript
// Resulting sorting state:
[
  { id: "name", desc: false },
  { id: "age",  desc: true  },
]
```

> ⚠️ Without Shift, each click resets sorting to a single column.

### Disable Sorting for a Column

```typescript
{
  accessorKey:   "actions",
  header:        "Actions",
  enableSorting: false,
}
```

### Custom Sorting Function

Sort by a custom priority order (e.g. task status):

```typescript
type Task = {
  id:     number;
  title:  string;
  status: "Pending" | "In Progress" | "Completed";
};

const columns = [
  { accessorKey: "title",  header: "Title"  },
  {
    accessorKey: "status",
    header:      "Status",
    sortingFn: (rowA, rowB, columnId) => {
      const order = { Pending: 1, "In Progress": 2, Completed: 3 };
      return order[rowA.getValue(columnId)] - order[rowB.getValue(columnId)];
    },
  },
];
```

Result when sorted ascending:

| Title  | Status      |
|--------|-------------|
| Task 2 | Pending     |
| Task 3 | In Progress |
| Task 1 | Completed   |

---

## 8. Column Visibility

### Setup State

```typescript
const [columnVisibility, setColumnVisibility] = useState({});
// e.g. { email: false, salary: false }
```

### Connect to Table

```typescript
const table = useReactTable({
  data,
  columns,
  state:                   { columnVisibility },
  onColumnVisibilityChange: setColumnVisibility,
  getCoreRowModel:         getCoreRowModel(),
});
```

### Toggle Visibility

```typescript
table.getColumn("email")?.toggleVisibility(false); // hide
table.getColumn("email")?.toggleVisibility(true);  // show
table.getColumn("email")?.toggleVisibility();      // auto-toggle
```

---

## 9. Column Ordering

### Setup State

```typescript
const [columnOrder, setColumnOrder] = useState([
  "name", "email", "role", "salary"
]);
```

### Connect to Table

```typescript
const table = useReactTable({
  data,
  columns,
  state:               { columnOrder },
  onColumnOrderChange: setColumnOrder,
  getCoreRowModel:     getCoreRowModel(),
});
```

### Reorder Programmatically

```typescript
setColumnOrder(["name", "salary", "email", "role"]);
// Renders: Name | Salary | Email | Role
```

---

## 10. Rendering: Headers & Rows

### Sortable Header Row

```tsx
{table.getHeaderGroups().map((headerGroup) => (
  <tr key={headerGroup.id}>
    {headerGroup.headers.map((header) => (
      <th
        key={header.id}
        onClick={header.column.getToggleSortingHandler()}
        className="cursor-pointer p-3 border-b text-left"
      >
        {flexRender(header.column.columnDef.header, header.getContext())}
        {{ asc: " 🔼", desc: " 🔽" }[header.column.getIsSorted() as string] ?? null}
      </th>
    ))}
  </tr>
))}
```

**Sort cycle:** unsorted → `asc 🔼` → `desc 🔽` → unsorted

---

---

## 11. Row Selection

Row selection is tracked via a state object mapping row IDs to `true`.

### Setup State

```typescript
import { RowSelectionState } from "@tanstack/react-table";

const [rowSelection, setRowSelection] = useState<RowSelectionState>({});
// e.g. { "row-1": true, "row-4": true }
```

### Connect to Table

```typescript
const table = useReactTable({
  data,
  columns,
  state:                { rowSelection },
  onRowSelectionChange: setRowSelection,
  enableRowSelection:   true,           // enable for all rows
  // enableRowSelection: (row) => row.original.status === "active", // conditional
  getCoreRowModel:      getCoreRowModel(),
});
```

### Checkbox Column Definition

Add a dedicated checkbox column as the first column:

```typescript
import { createColumnHelper } from "@tanstack/react-table";

{
  id: "select",
  header: ({ table }) => (
    <input
      type="checkbox"
      checked={table.getIsAllPageRowsSelected()}
      ref={(el) => {
        if (el) el.indeterminate = table.getIsSomePageRowsSelected();
      }}
      onChange={table.getToggleAllPageRowsSelectedHandler()}
    />
  ),
  cell: ({ row }) => (
    <input
      type="checkbox"
      checked={row.getIsSelected()}
      disabled={!row.getCanSelect()}
      onChange={row.getToggleSelectedHandler()}
    />
  ),
  enableSorting: false,
  size: 40,
}
```

### Reading Selected Rows

```typescript
// Get selected row objects
const selectedRows = table.getSelectedRowModel().rows;
// => Row<User>[]

// Get selected original data
const selectedData = table.getSelectedRowModel().rows.map(r => r.original);

// Count selected rows
const selectedCount = Object.keys(rowSelection).length;
```

### Selection Variants

```typescript
// Select all rows across ALL pages (not just current page)
table.getToggleAllRowsSelectedHandler();

// Select only current page rows
table.getToggleAllPageRowsSelectedHandler();

// Check states
table.getIsAllRowsSelected();       // true if every row is selected
table.getIsSomeRowsSelected();      // true if some (but not all) are selected
table.getIsAllPageRowsSelected();   // scoped to current page
```

> ⚠️ For server-side pagination, prefer `getToggleAllPageRowsSelectedHandler()` — only current-page rows are loaded in memory.

### Highlight Selected Rows (UI)

```tsx
{table.getRowModel().rows.map((row) => (
  <tr
    key={row.id}
    className={row.getIsSelected() ? "bg-blue-50" : ""}
  >
    {row.getVisibleCells().map((cell) => (
      <td key={cell.id} className="p-3">
        {flexRender(cell.column.columnDef.cell, cell.getContext())}
      </td>
    ))}
  </tr>
))}
```

---

## 12. Column Filtering

Column filters apply independently per column.

### Setup State

```typescript
import { ColumnFiltersState } from "@tanstack/react-table";

const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
// e.g. [{ id: "status", value: "active" }, { id: "age", value: [18, 30] }]
```

### Connect to Table

```typescript
const table = useReactTable({
  data,
  columns,
  state:                { columnFilters },
  onColumnFiltersChange: setColumnFilters,
  getCoreRowModel:      getCoreRowModel(),
  getFilteredRowModel:  getFilteredRowModel(), // required for client-side filtering
});
```

### Enable Filter Input per Column

```typescript
// In column definition
{
  accessorKey: "status",
  header: "Status",
  enableColumnFilter: true,   // default: true
}

// Disable filter for a column
{
  accessorKey: "id",
  enableColumnFilter: false,
}
```

### Filter Input UI

```tsx
// Render a filter input below each header
{header.column.getCanFilter() && (
  <input
    value={(header.column.getFilterValue() as string) ?? ""}
    onChange={(e) => header.column.setFilterValue(e.target.value)}
    placeholder={`Filter ${header.column.id}...`}
    className="mt-1 p-1 border rounded text-sm w-full"
  />
)}
```

### Built-in Filter Functions

| Filter Fn | Behaviour |
|---|---|
| `auto` | Inferred from column data type (default) |
| `includesString` | Case-insensitive substring match |
| `equalsString` | Exact string match |
| `arrIncludes` | Value exists in array |
| `inNumberRange` | `[min, max]` range filter |

```typescript
{
  accessorKey: "age",
  header: "Age",
  filterFn: "inNumberRange",  // filter value must be [min, max]
}
```

### Custom Filter Function

```typescript
{
  accessorKey: "status",
  header: "Status",
  filterFn: (row, columnId, filterValue) => {
    return row.getValue(columnId) === filterValue;
  },
}
```

### Programmatic Filtering

```typescript
// Set filter
table.getColumn("status")?.setFilterValue("active");

// Clear filter
table.getColumn("status")?.setFilterValue(undefined);

// Clear all filters
table.resetColumnFilters();
```

### Server-Side Column Filtering

```typescript
const table = useReactTable({
  data,
  columns,
  state:                { columnFilters },
  onColumnFiltersChange: setColumnFilters,
  manualFiltering:      true,           // disable client-side filter logic
  getCoreRowModel:      getCoreRowModel(),
});

// Send filters to API
useEffect(() => {
  fetchUsers({ filters: columnFilters });
}, [columnFilters]);
```

---

## 13. Global Search / Filtering

Global filter applies a single search term across all columns.

### Setup State

```typescript
const [globalFilter, setGlobalFilter] = useState<string>("");
```

### Connect to Table

```typescript
const table = useReactTable({
  data,
  columns,
  state:               { globalFilter },
  onGlobalFilterChange: setGlobalFilter,
  globalFilterFn:      "includesString",  // default
  getCoreRowModel:     getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
});
```

### Search Input UI

```tsx
<input
  value={globalFilter}
  onChange={(e) => setGlobalFilter(e.target.value)}
  placeholder="Search all columns..."
  className="p-2 border rounded w-64"
/>
```

### Custom Global Filter Function

```typescript
const table = useReactTable({
  // ...
  globalFilterFn: (row, columnId, filterValue) => {
    const value = String(row.getValue(columnId)).toLowerCase();
    return value.includes(String(filterValue).toLowerCase());
  },
});
```

> 💡 Combine `globalFilter` + `columnFilters` simultaneously — TanStack Table applies both independently.

---

## 14. Column Pinning

Pin columns to the left or right so they stay visible during horizontal scroll.

### Setup State

```typescript
import { ColumnPinningState } from "@tanstack/react-table";

const [columnPinning, setColumnPinning] = useState<ColumnPinningState>({
  left:  ["select", "firstName"],
  right: ["actions"],
});
```

### Connect to Table

```typescript
const table = useReactTable({
  data,
  columns,
  state:                { columnPinning },
  onColumnPinningChange: setColumnPinning,
  getCoreRowModel:      getCoreRowModel(),
});
```

### Programmatic Pinning

```typescript
// Pin to left
table.getColumn("firstName")?.pin("left");

// Pin to right
table.getColumn("actions")?.pin("right");

// Unpin
table.getColumn("firstName")?.pin(false);
```

### Rendering Pinned Columns

Use `getLeftVisibleLeafColumns()`, `getCenterVisibleLeafColumns()`, and `getRightVisibleLeafColumns()` to render three groups:

```tsx
<tr>
  {/* Left pinned */}
  {table.getLeftVisibleLeafColumns().map((col) => (
    <th key={col.id} style={{ position: "sticky", left: col.getStart("left"), background: "white" }}>
      {flexRender(col.columnDef.header, col.getContext())}
    </th>
  ))}

  {/* Scrollable center */}
  {table.getCenterVisibleLeafColumns().map((col) => (
    <th key={col.id}>
      {flexRender(col.columnDef.header, col.getContext())}
    </th>
  ))}

  {/* Right pinned */}
  {table.getRightVisibleLeafColumns().map((col) => (
    <th key={col.id} style={{ position: "sticky", right: col.getAfter("right"), background: "white" }}>
      {flexRender(col.columnDef.header, col.getContext())}
    </th>
  ))}
</tr>
```

---

## 15. Column Resizing

Allow users to drag column borders to resize them.

### Enable in Table

```typescript
const table = useReactTable({
  data,
  columns,
  columnResizeMode: "onChange",   // resize live as user drags
  // columnResizeMode: "onEnd",   // resize only when drag ends
  getCoreRowModel:  getCoreRowModel(),
});
```

### Set Initial Column Sizes

```typescript
const columns = [
  {
    accessorKey: "firstName",
    header:      "First Name",
    size:        150,    // initial width in px
    minSize:     80,     // minimum width
    maxSize:     300,    // maximum width
  },
];
```

### Resize Handle UI

```tsx
{headerGroup.headers.map((header) => (
  <th
    key={header.id}
    style={{ width: header.getSize(), position: "relative" }}
  >
    {flexRender(header.column.columnDef.header, header.getContext())}

    {/* Drag handle */}
    <div
      onMouseDown={header.getResizeHandler()}
      onTouchStart={header.getResizeHandler()}
      className={`absolute right-0 top-0 h-full w-1 cursor-col-resize bg-gray-300 hover:bg-blue-500 ${
        header.column.getIsResizing() ? "bg-blue-500" : ""
      }`}
    />
  </th>
))}
```

### Apply Column Width to Cells

```tsx
{row.getVisibleCells().map((cell) => (
  <td
    key={cell.id}
    style={{ width: cell.column.getSize() }}
  >
    {flexRender(cell.column.columnDef.cell, cell.getContext())}
  </td>
))}
```

---

## 16. Row Expanding (Subrows)

Expand rows to show nested sub-rows or custom detail panels.

### Option A — Nested Subrows (Tree Data)

```typescript
// Data must have a `subRows` field
type Category = {
  name:    string;
  subRows?: Category[];
};

const table = useReactTable({
  data,
  columns,
  getSubRows:        (row) => row.subRows,
  getCoreRowModel:   getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
});
```

### Option B — Custom Detail Panel (No Subrows)

```typescript
const [expanded, setExpanded] = useState<ExpandedState>({});

const table = useReactTable({
  data,
  columns,
  state:             { expanded },
  onExpandedChange:  setExpanded,
  getCoreRowModel:   getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
});
```

### Expand Toggle in Column

```typescript
{
  id: "expand",
  header: () => null,
  cell: ({ row }) =>
    row.getCanExpand() ? (
      <button onClick={row.getToggleExpandedHandler()}>
        {row.getIsExpanded() ? "▼" : "▶"}
      </button>
    ) : null,
}
```

### Rendering the Detail Panel

```tsx
{table.getRowModel().rows.map((row) => (
  <>
    <tr key={row.id}>
      {row.getVisibleCells().map((cell) => (
        <td key={cell.id}>
          {flexRender(cell.column.columnDef.cell, cell.getContext())}
        </td>
      ))}
    </tr>

    {row.getIsExpanded() && (
      <tr key={`${row.id}-detail`}>
        <td colSpan={columns.length} className="bg-gray-50 p-4">
          {/* Custom detail panel */}
          <pre>{JSON.stringify(row.original, null, 2)}</pre>
        </td>
      </tr>
    )}
  </>
))}
```

---

## 17. Row Grouping

Group rows by a column value, collapsing them under a group header.

### Setup

```typescript
import { getGroupedRowModel, getAggregatedRowModel } from "@tanstack/react-table";

const [grouping, setGrouping] = useState<GroupingState>(["status"]);

const table = useReactTable({
  data,
  columns,
  state:              { grouping },
  onGroupingChange:   setGrouping,
  getCoreRowModel:    getCoreRowModel(),
  getGroupedRowModel: getGroupedRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
  getAggregatedRowModel: getAggregatedRowModel(),
});
```

### Aggregation in Column Definitions

```typescript
{
  accessorKey: "visits",
  header:      "Visits",
  aggregationFn: "sum",          // built-in: sum, min, max, mean, count, extent
  aggregatedCell: ({ getValue }) => (
    <strong>Total: {getValue<number>()}</strong>
  ),
}
```

### Built-in Aggregation Functions

| Function | Description |
|---|---|
| `sum` | Sum of all values |
| `min` / `max` | Minimum / maximum value |
| `mean` | Average value |
| `count` | Number of rows in group |
| `extent` | `[min, max]` tuple |
| `unique` | Array of unique values |

---

## 18. Pagination Controls (UI)

```tsx
<div className="flex items-center gap-2 mt-4">

  {/* First / Prev */}
  <button onClick={() => table.firstPage()}    disabled={!table.getCanPreviousPage()}>««</button>
  <button onClick={() => table.previousPage()} disabled={!table.getCanPreviousPage()}>‹</button>

  {/* Page indicator */}
  <span>
    Page{" "}
    <strong>
      {table.getState().pagination.pageIndex + 1} of {table.getPageCount()}
    </strong>
  </span>

  {/* Next / Last */}
  <button onClick={() => table.nextPage()} disabled={!table.getCanNextPage()}>›</button>
  <button onClick={() => table.lastPage()}  disabled={!table.getCanNextPage()}>»»</button>

  {/* Go to page */}
  <input
    type="number"
    min={1}
    max={table.getPageCount()}
    defaultValue={table.getState().pagination.pageIndex + 1}
    onChange={(e) => table.setPageIndex(Number(e.target.value) - 1)}
    className="border p-1 rounded w-16"
  />

  {/* Page size selector */}
  <select
    value={table.getState().pagination.pageSize}
    onChange={(e) => table.setPageSize(Number(e.target.value))}
    className="border p-1 rounded"
  >
    {[10, 20, 50, 100].map((size) => (
      <option key={size} value={size}>Show {size}</option>
    ))}
  </select>

</div>
```

---

## 19. Virtualization (Large Lists)

For tables with thousands of rows, use `@tanstack/react-virtual` to only render visible rows.

### Install

```bash
npm install @tanstack/react-virtual
```

### Setup

```typescript
import { useVirtualizer } from "@tanstack/react-virtual";
import { useRef } from "react";

const tableContainerRef = useRef<HTMLDivElement>(null);
const rows = table.getRowModel().rows;

const rowVirtualizer = useVirtualizer({
  count:           rows.length,
  getScrollElement: () => tableContainerRef.current,
  estimateSize:    () => 35,       // estimated row height in px
  overscan:        10,             // render extra rows above/below viewport
});
```

### Rendering Virtual Rows

```tsx
<div ref={tableContainerRef} style={{ height: "600px", overflow: "auto" }}>
  <table>
    <thead>...</thead>
    <tbody style={{ height: `${rowVirtualizer.getTotalSize()}px`, position: "relative" }}>
      {rowVirtualizer.getVirtualItems().map((virtualRow) => {
        const row = rows[virtualRow.index];
        return (
          <tr
            key={row.id}
            style={{
              position:  "absolute",
              top:       0,
              transform: `translateY(${virtualRow.start}px)`,
              height:    `${virtualRow.size}px`,
            }}
          >
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        );
      })}
    </tbody>
  </table>
</div>
```

> 💡 Virtualization works independently of TanStack Table — the table computes all rows, and the virtualizer only renders the visible slice to the DOM.

---

## Quick Reference

| Concept | Key Option / Method |
|---|---|
| **Sorting** | |
| Server sorting | `manualSorting: true` |
| Disable sort on column | `enableSorting: false` |
| Custom sort logic | `sortingFn: (rowA, rowB, colId) => number` |
| Multi-column sort | Hold **Shift** + click headers |
| **Pagination** | |
| Server pagination | `manualPagination: true` |
| Provide page count | `pageCount: Math.ceil(total / pageSize)` |
| **Row Selection** | |
| Enable selection | `enableRowSelection: true` |
| Checkbox column | Add `id: "select"` column with `getToggleSelectedHandler()` |
| Read selected rows | `table.getSelectedRowModel().rows` |
| Conditional selection | `enableRowSelection: (row) => boolean` |
| **Filtering** | |
| Client column filter | `getFilteredRowModel()` |
| Server filter | `manualFiltering: true` |
| Global search | `globalFilterFn`, `onGlobalFilterChange` |
| **Columns** | |
| Hide a column | `toggleVisibility(false)` |
| Reorder columns | `setColumnOrder([...ids])` |
| Pin column | `column.pin("left" \| "right" \| false)` |
| Resize columns | `columnResizeMode: "onChange"` |
| Nested headers | Nest `columns` arrays inside a parent `header` group |
| **Rows** | |
| Expand subrows | `getExpandedRowModel()` + `getSubRows` |
| Group rows | `getGroupedRowModel()` + `grouping` state |
| Aggregate values | `aggregationFn: "sum" \| "mean" \| ...` |
| **Performance** | |
| Virtualize rows | `useVirtualizer` from `@tanstack/react-virtual` |
