# Accessibility Audit Tool

A self-contained web application for conducting WCAG 2.2 Level AA accessibility audits. Built as a single HTML file with no dependencies.

---

## Contents

- [What it does](#what-it-does)
- [Requirements](#requirements)
- [Installation](#installation)
- [How to use](#how-to-use)
- [Developer guide](#developer-guide)
  - [File structure](#file-structure)
  - [How the checklist data works](#how-the-checklist-data-works)
  - [Adding a checklist item](#adding-a-checklist-item)
  - [Editing or removing a checklist item](#editing-or-removing-a-checklist-item)
  - [Adding a new section](#adding-a-new-section)
  - [Adding a new group within a section](#adding-a-new-group-within-a-section)
  - [Changing colours and styling](#changing-colours-and-styling)
  - [Running multiple audits simultaneously](#running-multiple-audits-simultaneously)
- [Troubleshooting](#troubleshooting)

---

## What it does

The tool presents a structured checklist of accessibility tests aligned with WCAG 2.2 Level AA. For each test you can:

- Mark the result as **Pass**, **Fail**, or **N/A**
- Add a comment or description of any issue found
- Track overall progress across all 15 sections and ~90 checks

Progress is saved automatically in the browser using `localStorage`. When you are ready, clicking **Generate report** downloads a self-contained HTML file summarising the results, including a dedicated section for all failures and your comments.

---

## Requirements

- Any modern web browser (Chrome, Firefox, Edge, Safari)
- A web server or local server such as WAMP (for local use) or any standard hosting for live use
- No internet connection required once the file is loaded
- No installation of Node.js, PHP, databases, or any other software

---

## Installation

### On WAMP (local)

1. Copy `accessibility-audit-tool.html` into your WAMP `www` folder.  
   The default path on Windows is usually `C:\wamp64\www\`.
2. Make sure WAMP is running (the icon in the system tray should be green).
3. Open your browser and go to:  
   `http://localhost/accessibility-audit-tool.html`

### On a live server

1. Upload `accessibility-audit-tool.html` to any directory on your server using FTP, SFTP, or your hosting control panel.
2. Visit the URL for that file in your browser.  
   For example: `https://yourdomain.com/tools/accessibility-audit-tool.html`

That is all. There are no other files to upload and no configuration needed.

### Opening as a local file (without a server)

You can also open the file directly in a browser by double-clicking it, though some browsers restrict `localStorage` for files opened this way (particularly Chrome). Using a local server like WAMP avoids this issue entirely.

---

## How to use

1. **Fill in the site details** — enter the site name, URL, tester name, date, and optional scope notes. These appear in the generated report.
2. **Click Start testing** — the checklist loads with 15 sections accessible via the tab bar at the top.
3. **Work through each section** — for every check, click **Pass**, **Fail**, or **N/A**. Clicking the same button again deselects it if you change your mind.
4. **Add comments** — a comment box appears automatically when you mark something as Fail. You can type a description of the issue, the affected page, or any other relevant note.
5. **Monitor progress** — the progress bar and counters at the top update as you go. Sections with failures are highlighted in red in the tab bar.
6. **Generate the report** — click **Generate report** at the bottom right. A HTML file downloads automatically. Open it in any browser to view or print it.
7. **Start a new audit** — click **New audit** in the header. You will be asked to confirm before your current progress is cleared.

> **Note:** Progress is saved in your browser automatically. If you close the tab and reopen the tool in the same browser on the same machine, your audit will be restored. Progress is not shared between different browsers or different machines.

---

## Developer guide

The entire application is contained in a single file: `accessibility-audit-tool.html`.

It has three parts:

- **HTML** — the page structure and the two views (site details form and testing interface)
- **CSS** — all styling, inside a `<style>` block in the `<head>`
- **JavaScript** — all logic, inside a `<script>` block at the end of `<body>`

There is no build step, no package manager, and no framework. You can edit the file in any text editor such as Notepad++, VS Code, or Sublime Text.

---

### File structure

Within the JavaScript, the code is organised into clearly labelled sections using comments:

```
// ─── Checklist data ───
// ─── State ────────────
// ─── Storage ──────────
// ─── Initialise ───────
// ─── Start testing ────
// ─── Build sections ───
// ─── Status and comments ───
// ─── Stats ────────────
// ─── Reset ────────────
// ─── Report generation ───
// ─── Utility ──────────
```

The section you will most commonly need is **Checklist data**, which is where all the test items live.

---

### How the checklist data works

All checklist content is stored in a JavaScript array called `CL` near the top of the `<script>` block. It is a nested structure:

```
CL
 └─ section (e.g. "Colour")
     └─ group (e.g. "Check text colour contrast")
         └─ item (e.g. "Normal text below 24px must have a contrast of at least 4.5:1")
```

Here is a simplified example showing the shape:

```javascript
const CL = [
  {
    id: 'colour',          // Unique ID, no spaces, used internally
    title: 'Colour',       // Display name shown in tabs and reports
    groups: [
      {
        title: 'Check text colour contrast',      // Group heading
        tools: 'Silktide, axe DevTools, ...',     // Shown in italics under the heading
        items: [
          {
            id: 'tc1',                            // Unique ID, no spaces
            t: 'Normal text below 24px must...'  // The check text shown to the user
          },
          {
            id: 'tc2',
            t: 'Normal text of 24px and above...'
          }
        ]
      }
    ]
  }
];
```

**Important rules for IDs:**

- Every `id` must be unique across the entire checklist. No two sections, and no two items, can share the same ID.
- IDs must not contain spaces or special characters. Use letters and numbers only (e.g. `tc1`, `fr16`, `w6`).
- If you change an existing ID, any saved audit progress that used the old ID will be lost for that item.

---

### Adding a checklist item

Find the section and group where you want to add the item. Items sit inside the `items` array of a group.

**Before (existing items):**

```javascript
items: [
  { id: 'tc1', t: 'Normal text below 24px must have a colour contrast of at least 4.5:1' },
  { id: 'tc2', t: 'Normal text of 24px and above must have a colour contrast of at least 3:1' }
]
```

**After (new item added at the end):**

```javascript
items: [
  { id: 'tc1', t: 'Normal text below 24px must have a colour contrast of at least 4.5:1' },
  { id: 'tc2', t: 'Normal text of 24px and above must have a colour contrast of at least 3:1' },
  { id: 'tc5', t: 'Your new check text goes here' }
]
```

Note the comma after the previous last item. JavaScript arrays use commas between items, but not after the final item.

If the item relates to a new WCAG 2.2 criterion, you can add `isNew: true` to show the "New" badge:

```javascript
{ id: 'tc5', t: 'Your new check text goes here', isNew: true }
```

---

### Editing or removing a checklist item

To **edit** an item, simply change the value of its `t` property. The ID should stay the same so that any existing saved results still match up.

```javascript
// Before
{ id: 'tc1', t: 'Normal text below 24px must have a colour contrast of at least 4.5:1' }

// After (text corrected)
{ id: 'tc1', t: 'Normal text smaller than 24px must have a colour contrast ratio of at least 4.5:1' }
```

To **remove** an item, delete the entire `{ id: '...', t: '...' }` object and make sure the commas between the remaining items are correct.

---

### Adding a new section

Sections are the top-level entries in the `CL` array. To add a new one, add a new object at the position you want it to appear (sections display in the order they appear in the array).

```javascript
{
  id: 'mysection',           // Unique ID, no spaces
  title: 'My new section',   // Displayed in the tab bar
  groups: [
    {
      title: 'Check something',
      tools: 'Tool name, another tool',
      items: [
        { id: 'ms1', t: 'First check in this section' },
        { id: 'ms2', t: 'Second check in this section' }
      ]
    }
  ]
}
```

If this section relates to new WCAG 2.2 criteria, add `isNew: true` after the `title` property:

```javascript
{
  id: 'mysection',
  title: 'My new section',
  isNew: true,
  groups: [ ... ]
}
```

---

### Adding a new group within a section

Groups are the sub-headings within a section. Find the `groups` array for the section you want to add to, and append a new object:

```javascript
groups: [
  {
    title: 'Existing group',
    tools: '...',
    items: [ ... ]
  },
  {
    title: 'My new group',
    tools: 'Tool one, Tool two',
    items: [
      { id: 'newg1', t: 'First check in the new group' }
    ]
  }
]
```

---

### Changing colours and styling

All colours are defined as CSS custom properties (variables) at the top of the `<style>` block, inside `:root { ... }`. You can change any of them without touching the rest of the CSS.

```css
:root {
  --primary: #1a4a8a;         /* Main blue — header, links, section headings */
  --primary-light: #e8f1fb;   /* Light blue — active tab background */
  --primary-mid: #2e6db4;     /* Mid blue — hover states */

  --text: #1a1a1a;            /* Main body text */
  --text-muted: #5a6270;      /* Secondary text, labels */
  --text-subtle: #8a92a0;     /* Hints, placeholder-level text */

  --bg: #ffffff;              /* Page and card background */
  --bg-secondary: #f5f7fa;    /* Subtle background, form inputs */
  --bg-tertiary: #eef1f5;     /* Progress bar track */

  --border: #dde1e7;          /* Default borders */
  --border-strong: #b8bec8;   /* Stronger borders, button outlines */

  --success: #1e6e38;         /* Pass text colour */
  --success-bg: #e8f8ee;      /* Pass background */
  --success-border: #95d9ae;  /* Pass border */

  --danger: #b52d2d;          /* Fail text colour */
  --danger-bg: #fdf0f0;       /* Fail background */
  --danger-border: #e8a8a8;   /* Fail border */

  --na-text: #555;            /* N/A text colour */
  --na-bg: #f0f0f0;           /* N/A background */
  --na-border: #c0c0c0;       /* N/A border */

  --new-text: #1a4a8a;        /* "New in WCAG 2.2" badge text */
  --new-bg: #deeafb;          /* "New in WCAG 2.2" badge background */

  --radius: 6px;              /* Corner radius for cards and panels */
  --radius-sm: 4px;           /* Corner radius for buttons and inputs */
}
```

For example, to change the header and primary colour from blue to a dark green, you would change:

```css
--primary: #1a5e2a;
--primary-light: #e8f5ec;
--primary-mid: #2e7a3c;
--new-text: #1a5e2a;
--new-bg: #d4edda;
```

---

### Running multiple audits simultaneously

By default, the tool saves one audit at a time under the key `a11y-audit-v2` in `localStorage`. If you want to run two separate instances — for example one for a desktop audit and one for a mobile audit — you can duplicate the HTML file and change the storage key in each copy.

Find this line near the top of the `<script>` block:

```javascript
const STORAGE_KEY = 'a11y-audit-v2';
```

Change it to something unique in each copy:

```javascript
// In accessibility-audit-tool-desktop.html
const STORAGE_KEY = 'a11y-audit-desktop';

// In accessibility-audit-tool-mobile.html
const STORAGE_KEY = 'a11y-audit-mobile';
```

Each copy will then save and restore its own independent progress.

---

## Troubleshooting

**Progress is not being saved.**  
This usually happens when the file is opened directly from the filesystem in Chrome (`file:///...` in the address bar). Chrome restricts `localStorage` for local files. Use WAMP or another local server (`http://localhost/...`) instead.

**I changed an item ID and now its saved results are gone.**  
Results are stored by item ID. Changing an ID breaks the link to any previously saved result for that item. If you are editing an item's text only (not its ID), saved results will be preserved.

**The report downloads but will not open.**  
The report is a standard HTML file. Open it in any browser by double-clicking it, or right-clicking and choosing "Open with".

**I want to clear the saved progress without using the app.**  
Open your browser's developer tools (F12), go to the **Application** tab (Chrome) or **Storage** tab (Firefox), find **Local Storage**, and delete the entry with the key `a11y-audit-v2`.

**I accidentally clicked "New audit" and lost my progress.**  
Unfortunately this cannot be undone. The "New audit" action clears `localStorage` immediately on confirmation. For important audits, it is good practice to generate a report as a backup before starting a new one.

---

*Accessibility Audit Tool — WCAG 2.2 Level AA*
