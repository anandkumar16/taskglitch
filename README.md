# taskglitch

**Bug 1: Double Task Fetching on App Load**

Root Causes
- Injected second useEffect in useTasks.ts that fetched /tasks.json again with setTimeout(0)
- Malformed data injection - Random invalid task rows were being appended to the dataset
- No guard against React StrictMode double-invoke in development mode

Solution
File: useTasks.ts
- Removed the duplicate useEffect
- Removed injected malformed rows that corrupted task data
- Added fetchedRef.current guard to prevent double-invocation from React StrictMode

**Bug 2: Undo Snackbar Not Resetting State**

Root Causes
- Empty handleCloseUndo function in App.tsx did nothing when snackbar closed
- Missing clearLastDeleted method in the tasks context
- No mechanism to reset lastDeleted state when snackbar auto-closes

Solution
File: useTasks.ts
- Added new clearLastDeleted() callback function that resets lastDeleted to null
- Exported it in the context return value
File: TasksContext.tsx
- Added clearLastDeleted: () => void to the TasksContextValue interface
File: App.tsx
- Implemented proper handleCloseUndo function that calls clearLastDeleted()

**Bug 3: Unstable Task Sorting**

Root Cause
File: logic.ts 

Solution
File: logic.ts
- Replaced random tie-breaker with stable alphabetical sort by title
- Sort Hierarchy
- Higher ROI comes first
- Higher priority weight comes second (High → Medium → Low)
 -Alphabetical title (A-Z) as final tie-breaker

**Bug 4: Double Dialog Opening** 

Root Cause
File: TaskTable.tsx
- TableRow had onClick={() => setDetails(t)} to open View dialog
- Edit and Delete buttons had no stopPropagation() call
- Clicks bubbled up from buttons to parent row, triggering multiple handlers

Solution
File: TaskTable.tsx
- Added handleEditClick function with e.stopPropagation()
- Added handleDeleteClick function with e.stopPropagation()
- Updated Edit button: onClick={(e) => handleEditClick(e, t)}
- Updated Delete button: onClick={(e) => handleDeleteClick(e, t.id)}
- Added handleDeleteClick function with e.stopPropagation()
- Updated Edit button: onClick={(e) => handleEditClick(e, t)}
- Updated Delete button: onClick={(e) => handleDeleteClick(e, t.id)}

**Bug 5: Invalid ROI Calculations**

Root Causes
- No validation in computeROI - didn't check for zero division or invalid types
- Direct display of ROI without checking for Infinity/NaN
- TaskForm allowed timeTaken = 0 through validation
- ChartsDashboard bucketing didn't handle null/invalid ROI values
Inconsistent formatting - mixed use of toFixed(1) and toFixed(2)

Solution
File: logic.ts
Rewrote computeROI with comprehensive validation:

