# Pulse-Point — UI Style Guide

**Stack:** TailwindCSS • MUI (Material UI) • Mantine React Table (MRT)

> Goal: A crisp, professional, enterprise-ready UI that is fast, accessible (WCAG 2.1 AA), and consistent across pages.

---

## 1) Design Principles

* **Clarity first:** clear hierarchy, generous whitespace, predictable placement.
* **Consistency:** shared tokens/components; never redefine per page.
* **Comfortable density:** default medium, optional compact for data-heavy screens.
* **A11y:** color contrast ≥ 4.5:1; focus states always visible; keyboard-first.
* **Motion with purpose:** small, fast, easing for feedback (200–250ms).

---

## 2) Brand & Theme Tokens

Use tokens in Tailwind & MUI theme to ensure parity.

### 2.1 Color Palette

* **Primary**: `600`= #0B6BCA, `500`= #1976D2, `700`= #0A58A2
* **Secondary**: `600`= #5B5BD6, `500`= #6D6DE3
* **Success**: `#2E7D32`  • **Warning**: `#ED6C02`  • **Error**: `#D32F2F`  • **Info**: `#0288D1`
* **Gray scale**: 50–900 based on Tailwind neutral with minor tweaks
* **Background**: light `#F7F9FC` / dark `#0B0F19`
* **Surface**: light `#FFFFFF` / dark `#0F1525`
* **Border**: light `#E5EAF2` / dark `#22304C`
* **Focus ring**: `#80BFFF` (AA compliant)

### 2.2 Typography

* **Font family:** Inter, system-ui, Segoe UI, Roboto, Apple Color Emoji
* **Scale:**

  * Display: `text-3xl` (32/40)
  * H1: `text-2xl` (24/32)
  * H2: `text-xl` (20/28)
  * H3: `text-lg` (18/28)
  * Body: `text-base` (16/24)
  * Secondary: `text-sm` (14/20)
  * Micro: `text-xs` (12/16)
* **Weights:** 500 for headings, 400 body, 600 for emphasis.

### 2.3 Spacing & Radius

* **Spacing scale:** Tailwind default; prefer `space-y-3/4/6` for vertical rhythm.
* **Containers:** max-w-7xl, page gutters `px-6 md:px-8`.
* **Radius:** cards/buttons `rounded-2xl` for hero, `rounded-xl` standard, `rounded-lg` dense.
* **Shadows:** subtle; use `shadow-sm` on interactive surfaces, `shadow` for modals.

### 2.4 Elevation

* **0:** page bg  • **1:** cards, tables  • **2:** popovers/menus  • **3:** dialogs/drawers

---

## 3) Tailwind Configuration

Add tokens and dark mode.

```ts
// tailwind.config.ts
import type { Config } from 'tailwindcss'
export default {
  darkMode: 'class',
  content: [
    './index.html',
    './src/**/*.{ts,tsx,js,jsx}',
    './node_modules/@mui/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#E6F1FC',
          100: '#CCE3F9',
          200: '#99C7F3',
          300: '#66AAED',
          400: '#338EE7',
          500: '#1976D2',
          600: '#0B6BCA',
          700: '#0A58A2',
          800: '#083F75',
          900: '#062A50',
        },
        surface: {
          DEFAULT: '#FFFFFF',
          dark: '#0F1525',
        },
        bg: {
          DEFAULT: '#F7F9FC',
          dark: '#0B0F19',
        },
        border: {
          DEFAULT: '#E5EAF2',
          dark: '#22304C',
        },
        focus: '#80BFFF',
      },
      boxShadow: {
        subtle: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
      },
    },
  },
  plugins: [],
} satisfies Config
```

---

## 4) MUI Theme (Light/Dark + Component Overrides)

Create a single source of truth and wrap the app with `ThemeProvider`.

```ts
// theme.ts
import { createTheme } from '@mui/material/styles'

const base = {
  shape: { borderRadius: 12 },
  typography: {
    fontFamily: 'Inter, system-ui, Segoe UI, Roboto, Apple Color Emoji',
    h1: { fontSize: 24, fontWeight: 600 },
    h2: { fontSize: 20, fontWeight: 600 },
    h3: { fontSize: 18, fontWeight: 600 },
    body1: { fontSize: 16 },
    body2: { fontSize: 14 },
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: { textTransform: 'none', borderRadius: 12, fontWeight: 600 },
      },
      defaultProps: { disableElevation: true },
    },
    MuiPaper: { styleOverrides: { root: { borderRadius: 12 } } },
    MuiInputBase: {
      styleOverrides: {
        root: { borderRadius: 10 },
      },
    },
  },
}

export const lightTheme = createTheme({
  ...base,
  palette: {
    mode: 'light',
    primary: { main: '#1976D2' },
    secondary: { main: '#6D6DE3' },
    success: { main: '#2E7D32' },
    warning: { main: '#ED6C02' },
    error: { main: '#D32F2F' },
    info: { main: '#0288D1' },
    background: { default: '#F7F9FC', paper: '#FFFFFF' },
    divider: '#E5EAF2',
  },
})

export const darkTheme = createTheme({
  ...base,
  palette: {
    mode: 'dark',
    primary: { main: '#80BFFF' },
    secondary: { main: '#9FA9FF' },
    success: { main: '#66BB6A' },
    warning: { main: '#FFA726' },
    error: { main: '#EF5350' },
    info: { main: '#4FC3F7' },
    background: { default: '#0B0F19', paper: '#0F1525' },
    divider: '#22304C',
  },
})
```

**Usage:**

```tsx
// main.tsx
import { ThemeProvider, CssBaseline } from '@mui/material'
import { lightTheme, darkTheme } from './theme'

export function App({ dark }: { dark?: boolean }) {
  return (
    <ThemeProvider theme={dark ? darkTheme : lightTheme}>
      <CssBaseline />
      {/* ... */}
    </ThemeProvider>
  )
}
```

---

## 5) Component Library Conventions

### 5.1 Buttons

* Primary actions: MUI `contained` primary
* Secondary: `outlined` primary or `text`
* Destructive: `contained` error; confirm with dialog
* Sizes: `small` (compact tables), `medium` default, avoid `large`
* Tailwind helpers for layout only: `inline-flex items-center gap-2`

### 5.2 Inputs & Forms

* Use MUI `TextField`, `Select`, `Autocomplete`, `DatePicker`.
* Label always visible; help text for validation hints.
* Group related fields inside a card with `space-y-4`.
* Validation: show inline errors + top-level summary on submit failure.

### 5.3 Cards & Surfaces

* Use MUI `Paper` with `elevation={0}` + Tailwind `shadow-subtle border border-border`.
* Header pattern: title (H2) + right-aligned actions.

### 5.4 Navigation

* Left nav (permanent on desktop, drawer on mobile).
* Active item: primary tint bg (`bg-brand-50`) + bold text.

---

## 6) Mantine React Table (MRT) Guidelines

### 6.1 Table Presets

* **Table container:** MUI `Paper` within Tailwind `p-4`.
* **Row density:** default `md`; compact variant available via class `data-density="sm"` on wrapper.
* **Sticky header:** enabled for long lists.
* **Column config:** define `accessorKey`, `header`, `size`; keep essential columns first.

### 6.2 Interactions

* Sorting single-column by default; multi-sort with Shift.
* Filters: column filters (text/date/select) + top-level quick search.
* Row selection (checkbox) for bulk actions.
* Pagination: 10/25/50 per page; remember last choice per user.

### 6.3 Example Setup

```tsx
import { MantineReactTable, useMantineReactTable } from 'mantine-react-table'
import { Paper } from '@mui/material'

const columns = [
  { accessorKey: 'name', header: 'Name', size: 220 },
  { accessorKey: 'type', header: 'Type', size: 140 },
  { accessorKey: 'company', header: 'Company', size: 180 },
  { accessorKey: 'email', header: 'Email', size: 240 },
]

export function ContactsTable({ data }) {
  const table = useMantineReactTable({
    columns,
    data,
    enableStickyHeader: true,
    initialState: { pagination: { pageSize: 25 }, showGlobalFilter: true },
    enableRowSelection: true,
    mantinePaperProps: { withBorder: true, radius: 'md' },
  })
  return (
    <Paper className="p-4">
      <MantineReactTable table={table} />
    </Paper>
  )
}
```

---

## 7) Layout Patterns

### 7.1 App Shell

* **Header:** app name, global search, user menu.
* **Sidebar:** main nav; collapsible on small screens.
* **Content:** `max-w-7xl mx-auto p-6 md:p-8`.

### 7.2 Page Types

* **Dashboard:** 2–3 column grid, `grid gap-6 md:grid-cols-12`, cards span 4–8 cols.
* **List + Detail Drawer:** list table + right-side drawer for create/edit; page width stays stable.
* **Form Pages:** group fields into sections; show summary sidebar with key info.

---

## 8) States & Feedback

* **Hover:** subtle bg tint `bg-brand-50/40` for clickable rows.
* **Focus:** 2px ring `outline focus-visible:outline-2 focus-visible:outline-focus`.
* **Loading:** skeletons for lists; button spinners for actions.
* **Empty state:** icon + 1–2 lines copy + primary CTA.
* **Errors:** inline + toast; never only a toast.
* **Success:** quiet toast + checkmark icon.

---

## 9) Density Modes

* **Comfortable (default):** table row height \~48px, `text-sm`.
* **Compact:** \~40px rows, condensed padding, use on data-heavy pages.

Tailwind helpers:

```html
<div class="[&_.MuiTableCell-root]:py-2 [&_.MuiTableCell-root]:px-3 text-sm">...</div>
```

---

## 10) Dark Mode

* Use Tailwind `dark:` classes and MUI dark theme together.
* Surfaces switch to `background.paper = #0F1525` and text uses MUI palette.
* Maintain contrast for tables, dividers `#22304C`.

Toggle strategy: store user preference in local storage; follow OS by default.

---

## 11) Iconography & Illustrations

* Use MUI Icons; line style, 24px.
* Status icons: success ✓, warning !, error ⨯, info i.
* Avoid decorative imagery unless for empty states.

---

## 12) Motion

* Standard duration: 200ms (entrances 200–250ms, exits 150–200ms).
* Easing: `cubic-bezier(0.2, 0.0, 0.2, 1)` (MUI standard).
* Reduce motion: respect OS setting; disable non-essential animations.

---

## 13) Accessibility

* Keyboard: all actions reachable; focus order logical.
* ARIA: name/role/state on interactive controls; label form fields.
* Tables: header scope; row selection toggles labeled.
* Color: never rely on color alone; include icon/text cues.

---

## 14) Common Components (Recipes)

### 14.1 Page Header

```tsx
export function PageHeader({ title, actions }) {
  return (
    <div className="mb-6 flex items-center justify-between">
      <h1 className="text-2xl font-semibold tracking-tight">{title}</h1>
      <div className="flex items-center gap-2">{actions}</div>
    </div>
  )
}
```

### 14.2 Card

```tsx
import { Paper } from '@mui/material'
export const Card = ({ title, children, actions }) => (
  <Paper className="p-4 md:p-6 border border-border shadow-subtle" elevation={0}>
    <div className="flex items-center justify-between mb-4">
      <h2 className="text-xl font-semibold">{title}</h2>
      <div className="flex gap-2">{actions}</div>
    </div>
    {children}
  </Paper>
)
```

### 14.3 Form Section

```tsx
export const FormSection = ({ title, description, children }) => (
  <section className="space-y-4">
    <header>
      <h3 className="text-lg font-semibold">{title}</h3>
      {description && <p className="text-sm text-neutral-600 dark:text-neutral-300">{description}</p>}
    </header>
    <div className="grid gap-4 md:grid-cols-2">{children}</div>
  </section>
)
```

---

## 15) Do & Don’t

* **Do:** use MUI for controls, Tailwind for layout/spacing.
* **Do:** keep tables readable; not more than 7–9 visible columns by default.
* **Do:** prefer drawers over full-page modals for create/edit.
* **Don’t:** mix multiple button styles in one area.
* **Don’t:** introduce custom colors not in the palette.
* **Don’t:** put destructive actions next to primary without spacing/confirmation.

---

## 16) Implementation Checklist

* [ ] Tailwind configured with brand tokens and dark mode.
* [ ] MUI `ThemeProvider` with light/dark themes wired.
* [ ] Global layout shell created (header/sidebar/content).
* [ ] Page header, card, and form section components.
* [ ] MRT table preset (density, filters, sticky header).
* [ ] Empty state + loading skeleton components.
* [ ] A11y audit on first three pages.

---

## 17) Visual Examples (suggested)

* Contacts list (MRT) in comfortable and compact densities.
* Meeting detail with attached notes & to-dos.
* To-do Kanban with drag handles and status colors.

> Keep this file updated as components evolve; any new color, spacing, or pattern must be added here before use.
