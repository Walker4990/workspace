# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bucket List** is a simple, client-side web application for managing personal goals and tasks. It's a vanilla JavaScript project with no build tools, frameworks, or backend dependencies. Data is persisted using browser LocalStorage.

## Running the Project

### Quick Start
1. Open `index.html` directly in a browser
2. Or use a simple HTTP server:
   ```bash
   python -m http.server 8000
   # Then visit http://localhost:8000
   ```
3. Or use VS Code Live Server extension: right-click `index.html` → "Open with Live Server"

**No build step required** — this is a static site with no compilation or preprocessing.

## Architecture & Design Patterns

### Core Architecture
The application follows a **two-module separation**:

```
Storage Layer (js/storage.js)
    ↓ (handles data persistence)
    ↓
Business Logic (LocalStorage)
    ↓
UI Layer (js/app.js)
    ↓ (renders UI + handles events)
    ↓
View (index.html)
```

### Module Breakdown

**`js/storage.js` — BucketStorage Object**
- Singleton object pattern (not a class)
- Single responsibility: LocalStorage CRUD operations
- All data operations go through this module
- Methods: `load()`, `save()`, `addItem()`, `updateItem()`, `deleteItem()`, `toggleComplete()`, `getStats()`, `getFilteredList()`
- Storage key: `'bucketList'` (JSON array)

**`js/app.js` — BucketListApp Class**
- Class-based structure for UI management
- Responsibilities:
  - DOM caching (`cacheElements()`) — stores references to DOM nodes for performance
  - Event binding (`bindEvents()`) — attaches all event listeners
  - State management — tracks `currentFilter` and `editingId`
  - Rendering logic (`render()`) — re-renders UI on state changes
  - HTML generation and escaping (`createBucketItemHTML()`, `escapeHtml()`)
- **Key pattern**: Re-render on every state change (no virtual DOM)
- XSS protection via `escapeHtml()` — converts text to safe HTML

**`index.html` — UI Structure**
- Semantic HTML5 structure
- Tailwind CSS CDN for styling
- Custom CSS in `css/styles.css` for animations and overrides
- Modal for editing via display toggle (flex/hidden classes)

### Data Model

```javascript
{
  id: String,              // timestamp (Date.now().toString())
  title: String,           // goal text
  completed: Boolean,      // completion status
  createdAt: ISO String,   // ISO date string
  completedAt: ISO String  // null unless completed
}
```

## Key Implementation Details

### Filtering & Rendering
- Three filter states: `'all'`, `'active'` (incomplete), `'completed'`
- `getFilteredList()` returns filtered data
- `render()` rebuilds entire list HTML on state change (not optimized for large datasets)

### Event Handling
- Inline `onclick` handlers in generated HTML (e.g., `onclick="app.handleToggle('${item.id}'"`)
- Modal control via visibility toggle (class addition/removal)
- Form submission via `addEventListener` (not inline)

### Styling Strategy
- Tailwind CSS via CDN for base styling
- Custom CSS in `styles.css` for:
  - Filter button states
  - Animations (slideIn, fadeIn, scaleIn)
  - Mobile media queries (< 640px)
  - Dark mode support

## Important Notes for Modifications

1. **No external dependencies** — keep it that way. Any feature additions should use vanilla JS.
2. **LocalStorage key is `'bucketList'`** — changing it will orphan existing user data.
3. **HTML escaping is critical** — always use `escapeHtml()` when rendering user input to prevent XSS.
4. **Re-render is inefficient** — rebuilds entire list on every change; acceptable for small lists but consider refactoring if adding >1000 items.
5. **Modal management** — uses CSS class toggling (flex/hidden); no state persistence after page reload (by design).
6. **Timestamps as IDs** — works but could collide if items added simultaneously; acceptable for single-user app.

## Testing

No automated tests exist. Manual testing approach:
- Open `index.html` and test CRUD operations
- Open browser DevTools → Application → LocalStorage to inspect data
- Test in multiple browsers (Chrome, Firefox, Safari, Edge)
- Test mobile via DevTools device emulation

## Common Tasks

| Task | How To |
|------|--------|
| **Add new feature** | 1) Add UI in `index.html`, 2) Add storage logic in `storage.js`, 3) Add event handlers in `app.js` |
| **Change colors** | Edit Tailwind classes in `index.html` or CSS in `styles.css` |
| **Debug data** | Open browser console → check `localStorage.getItem('bucketList')` |
| **Extend functionality** | Keep separation between `BucketStorage` (data) and `BucketListApp` (UI) |

## Browser Compatibility

- Requires LocalStorage support (IE10+)
- Target: modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile: iOS Safari, Chrome for Android

## Future Enhancement Opportunities

See README.md for planned features (categories, images, priority levels, dark mode, drag-drop, export/import, etc.). Follow the two-module pattern: storage changes first, then UI updates.
