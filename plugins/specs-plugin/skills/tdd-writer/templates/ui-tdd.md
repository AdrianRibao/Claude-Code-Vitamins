# TDD: [Feature Name] - UI

| Document    | Link                                       |
| ----------- | ------------------------------------------ |
| Parent PRD  | [prd-name.md](../prds/prd-name.md)         |
| Backend TDD | [feature-backend.md](./feature-backend.md) |

| Domain       | Priority | Status   | Last Updated   |
| ------------ | -------- | -------- | -------------- |
| `UI.Feature` | P0-P3    | Planning | [Month] [Year] |

______________________________________________________________________

## Overview

### Purpose

[2-3 sentences: What user problem does this UI solve?]

### Scope

| Included             | Out of Scope            |
| -------------------- | ----------------------- |
| ✅ List view         | ❌ Bulk operations      |
| ✅ Create/Edit forms | ❌ Advanced filtering   |
| ✅ Detail view       | ❌ Export functionality |

______________________________________________________________________

## Routes

| Route                | Page Component | Description            |
| -------------------- | -------------- | ---------------------- |
| `/features`          | FeatureList    | List all user features |
| `/features/new`      | FeatureForm    | Create new feature     |
| `/features/:id`      | FeatureDetail  | View feature details   |
| `/features/:id/edit` | FeatureForm    | Edit existing feature  |

______________________________________________________________________

## Screens

### Feature List

**Purpose**: Display paginated list of user's features

**Components**:

| Component    | Description                                |
| ------------ | ------------------------------------------ |
| Header       | Page title + "New Feature" button          |
| FilterBar    | Status filter, search input                |
| FeatureTable | Sortable columns: name, status, created_at |
| Pagination   | Page navigation with page size selector    |
| EmptyState   | Shown when no features exist               |

**User Actions**:

- Click row → Navigate to detail view
- Click "New Feature" → Navigate to create form
- Filter by status → Update list
- Sort by column → Reorder list

### Feature Form (Create/Edit)

**Purpose**: Create or edit a feature

**Fields**:

| Field       | Type     | Validation        | Notes                |
| ----------- | -------- | ----------------- | -------------------- |
| name        | text     | Required, max 255 | Auto-focus on load   |
| description | textarea | Optional          | Markdown supported   |
| status      | select   | Required          | Options from backend |

**Actions**:

| Action | Behavior                             |
| ------ | ------------------------------------ |
| Submit | Validate → Save → Navigate to detail |
| Cancel | Navigate back without saving         |

**Validation Display**:

- Inline errors below each field
- Form-level errors in alert component
- Submit button disabled until valid

### Feature Detail

**Purpose**: Display feature information

**Sections**:

| Section       | Content                            |
| ------------- | ---------------------------------- |
| Header        | Name, status badge, action buttons |
| Metadata      | Created, updated, owner            |
| Description   | Rendered markdown                  |
| Related Items | List of associated items           |

**Actions**:

| Action  | Condition                      | Behavior              |
| ------- | ------------------------------ | --------------------- |
| Edit    | User is owner                  | Navigate to edit form |
| Archive | User is owner, status = active | Confirm → Archive     |
| Delete  | User is admin                  | Confirm → Delete      |

______________________________________________________________________

## Authorization

### Layer 1: Action Permissions (UI Actions)

| Actor   | View List | Create | Edit    | Archive | [Domain Action] |
| ------- | --------- | ------ | ------- | ------- | --------------- |
| Owner   | ✅ Own    | ✅     | ✅ Own  | ✅ Own  | [scope]         |
| Manager | ✅ Team   | ✅     | ✅ Team | ✅ Team | [scope]         |
| Admin   | ✅ All    | ✅     | ✅ All  | ✅ All  | ✅ All          |
| Guest   | ✅ Public | ❌     | ❌      | ❌      | ❌              |

### Layer 2: Data Permissions (UI Visibility)

| Field/Section  | Owner  | Manager | Admin | Guest  |
| -------------- | ------ | ------- | ----- | ------ |
| [public field] | RW     | RW      | RW    | R      |
| [restricted]   | R      | RW      | RW    | Hidden |
| [sensitive]    | Hidden | R       | RW    | Hidden |

> **RW** = Editable field, **R** = Displayed read-only, **Hidden** = Component not rendered. UI must hide actions the user cannot perform.

### Layer 3: Permission Conditions

| Rule ID | Condition           | UI Effect                    | Actors |
| ------- | ------------------- | ---------------------------- | ------ |
| PC-01   | `status = 'draft'`  | Show edit button             | Owner  |
| PC-02   | `status != 'draft'` | Disable edit, show read-only | Owner  |

### Policy Rules

- Buttons and menu items for unauthorized actions must be hidden (not just disabled)
- Navigation to unauthorized routes redirects to appropriate fallback
- Field visibility adapts per role — sensitive fields never rendered for unauthorized roles

______________________________________________________________________

## Components

### Shared Components

| Component     | Props                     | Description                     |
| ------------- | ------------------------- | ------------------------------- |
| StatusBadge   | status: enum              | Color-coded status indicator    |
| FeatureCard   | feature: object           | Compact feature display         |
| ConfirmDialog | title, message, onConfirm | Destructive action confirmation |

### Form Components

| Component     | Props                | Description                |
| ------------- | -------------------- | -------------------------- |
| TextField     | name, label, error   | Text input with validation |
| SelectField   | name, options, error | Dropdown selector          |
| TextAreaField | name, label, rows    | Multi-line input           |

______________________________________________________________________

## User Flows

### Create Feature Flow

```
List Page → Click "New" → Form Page → Fill fields → Submit → Detail Page
                                    ↓
                              Validation Error → Show inline errors
```

### Archive Feature Flow

```
Detail Page → Click "Archive" → Confirm Dialog → Yes → API Call → Success Toast → Update UI
                              ↓
                              No → Close dialog
```

______________________________________________________________________

## Behavior Specifications

### Form Submission

**Given**: User is on create form with valid data **When**: User clicks "Submit" **Then**:

- Loading indicator appears on button
- Form fields are disabled
- On success: Navigate to detail page with success toast
- On error: Show error message, re-enable form

### Optimistic UI Updates

**Given**: User archives a feature from list view **When**: Archive is confirmed **Then**:

- Item immediately shows "archived" status
- On API failure: Revert status, show error toast

______________________________________________________________________

## Accessibility Requirements

- [ ] All form fields have associated labels
- [ ] Focus management on page navigation
- [ ] Keyboard navigation for all actions
- [ ] Screen reader announcements for status changes
- [ ] Color contrast meets WCAG AA
- [ ] Error messages are announced

______________________________________________________________________

## Acceptance Criteria

### Navigation

- [ ] User can navigate to list from sidebar
- [ ] User can navigate to create form from list
- [ ] User can navigate to detail from list row click
- [ ] Browser back/forward works correctly

### List View

- [ ] List displays user's features with pagination
- [ ] User can filter by status
- [ ] User can sort by column headers
- [ ] Empty state shown when no features

### Form

- [ ] Required fields show validation errors
- [ ] Form remembers values on validation failure
- [ ] Loading state shown during submission
- [ ] Success redirects to detail page

### Authorization

- [ ] Unauthorized actions are hidden (buttons, menu items not rendered)
- [ ] Navigation to unauthorized routes redirects appropriately
- [ ] Field visibility matches data permissions per role
- [ ] Permission conditions (PC-xx) correctly toggle UI state
- [ ] Sensitive fields never rendered in DOM for unauthorized roles

### Responsive

- [ ] Layout adapts to mobile viewport
- [ ] Touch targets are 44px minimum
- [ ] Table converts to card layout on mobile

### Testing

- [ ] Unit tests for all component logic and state management
- [ ] Unit tests validate form validation rules and error handling
- [ ] Component tests verify prop handling and event emissions
- [ ] Integration tests validate navigation flows and route transitions
- [ ] E2E tests cover complete user workflows (create, edit, delete)
- [ ] E2E tests verify optimistic UI updates and error recovery
- [ ] **End-to-end tests render each page/form/component via the framework's real render pipeline (e.g. `Phoenix.LiveViewTest`, Testing Library, Capybara, Playwright) — not just unit-level component mounts**
- [ ] **Every change / live-validation event path has a test, including each validation-error branch**
- [ ] **Every submit success path asserts redirect/toast AND the backend effect: persisted row attributes, job enqueued, or API call payload — NOT just the redirect**
- [ ] **Every submit failure branch has a test: missing field, invalid format, validation rejection, anti-discovery, rate limit, server error**
- [ ] **Failure-branch tests assert the rendered error message or field-level error — NEVER only a static string like page title or header (which passes even when the whole validation pipeline is broken)**
- [ ] **UI bug fixes include a regression test that fails on the buggy code and passes on the fix**
- [ ] **Accessibility tests validate WCAG AA compliance**
- [ ] **Accessibility tests verify keyboard navigation and focus management**
- [ ] **Accessibility tests validate screen reader announcements**
- [ ] **Authorization tests verify hidden actions are not rendered for unauthorized roles**
- [ ] **Authorization tests verify field visibility matches data permissions per role**
- [ ] **Authorization tests verify permission conditions toggle UI correctly (PC-xx)**
- [ ] **Authorization tests confirm sensitive fields are absent from DOM (not just hidden via CSS)**
- [ ] Visual regression tests detect unintended UI changes
- [ ] Test coverage ≥ 70% for UI components
- [ ] Test coverage ≥ 90% for critical user flows
- [ ] Performance tests validate page load < 3s, interaction < 100ms
- [ ] Cross-browser tests verify functionality in Chrome, Firefox, Safari
- [ ] UI tests cover all user-facing components and interactions

### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards

______________________________________________________________________

## Related Documents

| Document     | Link                                       |
| ------------ | ------------------------------------------ |
| Parent PRD   | [prd-name.md](../prds/prd-name.md)         |
| Backend TDD  | [feature-backend.md](./feature-backend.md) |
| Design Specs | [figma-link](https://figma.com/...)        |
