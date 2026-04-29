**

# How the App Handles Data: Analysis & Findings

  

**Date:** March 02, 2026  

**Goal:** To understand how the app loads common data (like Business Units and Departments) and what to do with the old, unused Redux code.

  

## 1. WHAT WE FOUND: REPEATED DATA LOADING

  

Right now, the app is loading the exact same information multiple times. This happens whenever you move between different pages.

  

### Common Data Being Loaded Too Often:

The following lists are being requested from the server over and over again:

  

| Data Type | Used In |

| :--- | :--- |

| **Technologies** | Business Units, Departments, Projects, etc. |

| **Industries** | Welcome Form, Org Details, Business Units, etc. |

| **Job Titles** | Employees, Roles, Task Mapping, etc. |

| **Skills** | Skill Management, Teams, Matrix, etc. |

| **Certifications**| Business Units, Departments, Org Details, etc. |

  

### Why This Matters:

- **Too Many Requests:** These lists are called in **15+ different places**.

- **Slow Feeling:** Every time you switch pages, the app has to wait for the server to send the same list again.

- **Old Code (Redux):** There is old code (called "Redux") meant to help with this, but it's **not actually being used** by the current pages. It's basically just taking up space.

  

---

  

## 2. A BETTER WAY: "CONTEXT API" (SHARED DATA)

  

Instead of every page asking for its own list, we recommend a "Shared Data" setup. This is like having a single library where every page can get what it needs instantly.

  

### 2. DETAILED FINDINGS

  

### A. All Reference Data Categories (Exhaustive List)

  

After a 100% thorough scan of the application, I have identified the following categories of data that are fetched repeatedly and are prime candidates for a central "Reference Data" context:

  

| Data Type | Primary Usage Areas | Source API (Redundant) |

| :--- | :--- | :--- |

| **Business Units (BU)** | Departments, Employees, Teams, Cohorts, Org Details | `api/business-unit.jsx` / `api/org-directory.jsx` |

| **Sub-Business Units (SBU)** | Departments, Employees, Teams, Cohorts, Org Details | `api/business-unit.jsx` / `api/org-directory.jsx` |

| **Departments** | Employees, Teams, Cohorts, Job Titles, Skills | `api/department.jsx` / `api/org-directory.jsx` |

| **Technologies** | Business Units, Departments, Projects, Org Details | `api/business-unit.jsx` |

| **Industries** | Welcome Form, Org Details, Business Units, SBU | `api/business-unit.jsx` |

| **Job Titles** | Employees, Roles, Task Mapping, Skill Matrix, Cohorts | `api/job-title.jsx` / `api/org-directory.jsx` |

| **Skills / Competencies** | Skill Catalog, Mapping Engine, Teams, Matrix | `api/skills.jsx` / `api/org-directory.jsx` |

| **Certifications** | Business Units, Departments, Org Details | `api/certifications.jsx` |

| **Proficiency Levels** | Skill Mapping, Competency Mapping, Matrix | `api/proficiency.jsx` / `api/org-directory.jsx` |

| **Importance Levels** | Skill Mapping, Competency Mapping | `api/org-directory.jsx` |

| **Projects** | Tasks, Entity Selectors, Mapping | `api/project.jsx` / `api/org-directory.jsx` |

| **Teams & Cohorts** | Entity Selectors, Task Mapping, Cohort Mapping | `api/teams.jsx` / `api/org-directory.jsx` |

| **Tasks** | Project Management, Task Mapping, Role Mapping | `api/tasks.jsx` / `api/org-directory.jsx` |

| **Countries & Cities** | Locations, Profile Setup, Org Details | `api/location-api.jsx` (Redundant with library) |

  

### B. The "Two APIs" Problem

A major discovery is that the application often has **two different ways** to fetch the same data:

1.  **Specific APIs**: Custom files like `api/skills.jsx` or `api/project.jsx`.

2.  **Org-Directory API**: A generic `api/org-directory.jsx` that has "Org-wide" versions of the same fetches (e.g., `getAllOrgProjects` vs `getAllProjects`).

  

**Current Issue:** Different components use different versions of these APIs, making it impossible to cache data effectively without a central provider.

  

### C. Resource-Intensive Scenarios (Deep Dive)

  

| Scenario | Current Behavior (Inefficient) | Ideal Behavior (with Context) |

| :--- | :--- | :--- |

| **Switching Tabs in Org Directory** | Navigating from BU to SBU to Departments triggers 5-6 API calls every time you switch back and forth. | Data is fetched once on mount; switching tabs is instantaneous with zero network activity. |

| **Entity Selector (Dropdowns)** | Components like the `EntitySelector` often fetch Job Titles, Projects, Teams, and Cohorts all at once just to populate a dropdown. | The dropdowns use the pre-loaded data from the Context API. |

| **Employee Form** | Opening an employee edit form triggers calls for BU, SBU, Dept, City, Country, and Job Title. | All dropdowns are instantly populated from global state. |

  

---

  

## 3. LEGACY REDUX ANALYSIS

  

I have confirmed that the `store/` directory contains 21 slices (BU, SBU, Dept, etc.) that are either incomplete or not used for simple data lookups. They mostly follow a pattern optimized for large paginated tables, which is overkill for "Reference Data" (the simple list of items used in dropdowns).

  

**Decision:** We will completely ignore these Redux slices and remove them. The Context API will be much simpler and more direct.

  

---

  

## 4. PROPOSED SOLUTION: ReferenceDataContext

  

### How it will work:

1.  **Initial Load**: On application start, a single `ReferenceDataProvider` will call the necessary "get all" APIs.

2.  **Internal Storage**: The data will be stored in a simple state object: `{ bus: [], sbus: [], depts: [], jobTitles: [], ... }`.

3.  **The Hook**: Any component can simply call `const { bus, depts } = useReferenceData();`.

4.  **Auto-Refresh**: The provider can include a `refresh()` function that components can call after they add a new item (like adding a new Department) to keep the global list up to date.

  

---

  

## 5. PAGINATION STRATEGY

  

One common question is how to handle pagination if the lists become very large. Here is the proposed strategy:

  

### A. Reference Data (The "Fetch-All" Group)

For lists like **Business Units, Departments, Job Titles, and Technologies**, the current app already uses a `show_all=true` or `per_page=1000` pattern for dropdowns.

- **Implementation**: The `ReferenceDataContext` will use these "fetch-all" versions. Since these lists typically contain tens or hundreds of items (not thousands), storing them in memory is highly efficient and provides an "instant" feel to the UI.

  

### B. Transactional Data (The "Paginated" Group)

For large datasets like **Employees** or **Project Lists** (when shown in tables), we will **not** store the full list in the global context.

- **Implementation**: Components that display these tables will continue to use their own local state and call the API with `page` and `per_page` parameters.

- **Exception**: If an Employee dropdown is needed (e.g., "Select Manager"), we will only fetch the list of *Managers* into the context, not all 10,000 employees.

  

### C. Keeping Everything in Sync

When a user is on a management page (e.g., the Business Units table) and adds a new item:

1.  The management page calls the **Create API**.

2.  Upon success, it calls `refresh('bus')` from our `ReferenceDataContext`.

3.  The context re-fetches the full BU list in the background.

4.  All dropdowns across the app are automatically updated.

  

---

  

## 6. NEXT STEPS (Action Plan)

  

1.  **Clean up Legacy Redux**: Remove the `store/` folder and the Redux Provider in `layout.jsx`.

2.  **Create ReferenceDataContext**: Build the provider in `app/context/ReferenceDataContext.jsx`.

3.  **Global Integration**: Wrap `app/layout.jsx` with the new provider.

4.  **Selective Refactoring**: Start with the most redundant modules (BU, SBU, Departments, Employees).

  

---

  

## 6. FINAL SUMMARY

  

The app is currently doing more work than it needs to by loading the same data repeatedly. By using a simpler "Shared Data" (Context API) setup and removing the old, unused Redux code, the app will be much faster and easier to maintain.

  
  
**