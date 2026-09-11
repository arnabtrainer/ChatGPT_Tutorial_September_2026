# Product Requirements Document (PRD)
# HighlightHub: Smart Web Highlighter Chrome Extension

**Version:** 1.0 (MVP)
**Author:** Codebasics AI Toolkit Webinar
**Date:** March 2026
**Status:** Draft

---

## 1. Product Overview

### What Is HighlightHub?

HighlightHub is a Chrome browser extension that lets users highlight text on any webpage, automatically saves those highlights, and provides a searchable dashboard to revisit them. When a user clicks on a saved highlight, the extension navigates back to the original page and visually re-highlights the saved text.

### Who Is It For?

Students, researchers, content consumers, and anyone who reads extensively on the web and wants a simple way to save and revisit important information without switching to a separate note-taking app.

### Why Does It Matter?

People read dozens of articles, blog posts, and documentation pages daily. The useful bits get lost in browser history. HighlightHub solves this by turning the browser itself into a personal knowledge capture tool with zero friction: just highlight, and it is saved.

---

## 2. User Personas

### Persona 1: Priya, the Online Learner

- **Age:** 24
- **Background:** Learning data analytics through online courses and blogs
- **Pain Point:** She reads 10+ articles a day and cannot remember where she read specific concepts
- **Goal:** Save key definitions, formulas, and insights while reading and find them later quickly

### Persona 2: Rahul, the Content Researcher

- **Age:** 30
- **Background:** Works in marketing, researches competitor content and industry trends
- **Pain Point:** Uses bookmarks but they save entire pages, not specific text
- **Goal:** Save specific paragraphs and quotes with the ability to trace them back to the source

---

## 3. User Stories (MVP)

| ID | User Story | Priority |
|----|-----------|----------|
| US-01 | As a user, I want to highlight text on any webpage so that I can mark important information | Must Have |
| US-02 | As a user, I want my highlights to be automatically saved so that I do not have to manually copy-paste | Must Have |
| US-03 | As a user, I want to see a dashboard of all my highlights so that I can browse everything I have saved | Must Have |
| US-04 | As a user, I want my highlights grouped by webpage so that I can see all highlights from a specific article together | Must Have |
| US-05 | As a user, I want to click on a saved highlight and go back to the original page so that I can see the full context | Must Have |
| US-06 | As a user, I want the original text to be re-highlighted when I navigate back so that I can quickly locate it on the page | Must Have |
| US-07 | As a user, I want to search through my highlights so that I can find specific information quickly | Must Have |
| US-08 | As a user, I want to filter highlights by date or by source page so that I can narrow down what I am looking for | Must Have |
| US-09 | As a user, I want to delete individual highlights so that I can keep my dashboard clean | Must Have |
| US-10 | As a user, I want to see the date and time when I saved a highlight so that I know when I captured it | Should Have |

---

## 4. MVP Feature Scope

### What Gets Built in Version 1

**Highlight Capture**
- User selects text on any webpage
- A small floating button appears near the selection (or right-click context menu option)
- Clicking the button saves the highlight instantly
- A brief visual confirmation appears (e.g., the text flashes with a background color)

**Chrome Extension Popup**
- Shows a quick summary: total highlights count, recent 3 highlights
- Has a button to open the full dashboard

**Dashboard (Full HTML Page)**
- Opens as a new tab (chrome-extension://[id]/dashboard.html)
- Displays all highlights grouped by source webpage
- Each group shows the page title, URL, and favicon
- Each highlight entry shows the saved text, date/time of capture
- Search bar at the top to search across all highlight text
- Filter options: by date range, by source domain
- Click on any highlight to navigate to the source page
- Delete button on each highlight

**Navigate and Re-highlight**
- When user clicks a saved highlight, the extension opens the original URL
- The content script attempts to find and scroll to the exact text on the page
- If found, the text is visually highlighted with a distinct background color
- If the text is not found (page content changed), no fallback action is taken

### What Is NOT in Version 1

- No user accounts or cloud sync
- No sharing or collaboration
- No tags or folders
- No export functionality
- No screenshot capture
- No voice notes
- No AI-powered summarization

---

## 5. Information Architecture

### Data Model

Each saved highlight is stored as a JSON object in Chrome's local storage:

```json
{
  "highlights": [
    {
      "id": "unique-uuid",
      "text": "The highlighted text content",
      "pageUrl": "https://example.com/article",
      "pageTitle": "Article Title",
      "favicon": "https://example.com/favicon.ico",
      "timestamp": "2026-03-05T14:30:00Z",
      "textContext": {
        "prefix": "50 characters before the highlight",
        "suffix": "50 characters after the highlight"
      }
    }
  ]
}
```

**Field Descriptions:**

| Field | Purpose |
|-------|---------|
| id | Unique identifier for each highlight |
| text | The actual highlighted text |
| pageUrl | Full URL of the source page |
| pageTitle | Title of the source page |
| favicon | Favicon URL for visual identification in the dashboard |
| timestamp | When the highlight was saved |
| textContext.prefix | Text appearing before the highlight (helps with re-finding on revisit) |
| textContext.suffix | Text appearing after the highlight (helps with re-finding on revisit) |

### Storage Strategy

- **Chrome Storage API** (`chrome.storage.local`) for all data
- Storage limit: 10 MB for `chrome.storage.local` (sufficient for thousands of text highlights)
- No external database or backend required

---

## 6. User Flows

### Flow 1: Saving a Highlight

```
1. User is reading a webpage
2. User selects text by clicking and dragging
3. A small "Save Highlight" button appears near the selection
4. User clicks the button
5. The extension captures: selected text, page URL, page title, favicon, timestamp, surrounding context
6. Data is saved to Chrome local storage
7. A brief visual flash confirms the save
8. The floating button disappears
```

### Flow 2: Viewing the Dashboard

```
1. User clicks the HighlightHub extension icon in the toolbar
2. A popup appears showing highlight count and recent highlights
3. User clicks "Open Dashboard"
4. A new tab opens with the full dashboard
5. Highlights are displayed grouped by source page
6. User can scroll, search, or filter to find specific highlights
```

### Flow 3: Navigating Back to Source

```
1. User is on the dashboard
2. User clicks on a specific highlight entry
3. The extension opens the source URL in a new tab
4. The content script activates on the page
5. The script searches for the exact text (using the saved text + context)
6. If found: scrolls to the text and applies a highlight background color
7. If not found: the page opens normally without any highlight
```

### Flow 4: Deleting a Highlight

```
1. User is on the dashboard
2. User clicks the delete icon next to a highlight
3. A confirmation prompt appears
4. User confirms deletion
5. The highlight is removed from storage and disappears from the dashboard
```

---

## 7. Technical Architecture

### Extension Components

A Chrome Extension (Manifest V3) consists of several files that work together. Here is what HighlightHub needs:

| Component | File(s) | Purpose |
|-----------|---------|---------|
| Manifest | `manifest.json` | Configuration file that tells Chrome about the extension: permissions, scripts, pages |
| Content Script | `content.js` | Runs on every webpage the user visits. Detects text selection, shows the save button, handles re-highlighting |
| Background Script (Service Worker) | `background.js` | Runs in the background. Handles storage operations, coordinates between content script and dashboard |
| Popup | `popup.html`, `popup.js` | The small window that appears when clicking the extension icon. Shows summary and link to dashboard |
| Dashboard | `dashboard.html`, `dashboard.js`, `dashboard.css` | Full-page interface for browsing, searching, and managing highlights |
| Styles | `content.css` | Styles for the floating save button and re-highlight effects on webpages |
| Icons | `icons/` | Extension icons in multiple sizes (16x16, 48x48, 128x128) |

### Manifest V3 Permissions Required

| Permission | Why It Is Needed |
|------------|-----------------|
| `storage` | To save and retrieve highlights using Chrome Storage API |
| `activeTab` | To access the current tab when the user interacts with the extension |
| `scripting` | To inject content scripts for re-highlighting when navigating back |
| `tabs` | To open new tabs and navigate to saved URLs |

### How Components Communicate

```
Content Script (on webpage)
    |
    | --- sends highlight data via chrome.runtime.sendMessage() --->
    |
Background Script (Service Worker)
    |
    | --- reads/writes data via chrome.storage.local --->
    |
Dashboard Page
    |
    | --- reads data via chrome.storage.local
    | --- sends navigation requests via chrome.tabs.create()
```

---

## 8. UI/UX Requirements

### 8.1 Floating Save Button (on webpages)

- Appears near the mouse cursor when text is selected
- Small, rounded button with a highlighter icon
- Background color: yellow/amber (to match the highlighter metaphor)
- Disappears when user clicks elsewhere or deselects text
- Should not block the selected text

### 8.2 Extension Popup

- Width: 320px, Height: 400px (approx)
- Header: Extension name and logo
- Stats bar: "You have X highlights saved"
- Recent highlights section: Shows the last 3 saved highlights with truncated text
- "Open Dashboard" button at the bottom (prominent, full-width)

### 8.3 Dashboard Page

- Clean, card-based layout
- Top section: Search bar + filter controls (date range picker, domain filter dropdown)
- Main area: Highlights grouped by source page
- Each group has a header showing: page favicon + page title + URL + highlight count
- Each highlight card shows: saved text (full), date/time, delete icon, click-to-navigate action
- Responsive layout that works on different screen widths
- Color scheme: Clean whites and grays with yellow/amber accent (matching the highlighter theme)

---

## 9. Success Metrics

For an MVP, success is measured by whether the core loop works reliably:

| Metric | Target |
|--------|--------|
| Highlight saved successfully on any standard webpage | 95%+ success rate |
| Dashboard loads and displays all saved highlights | 100% reliability |
| Search returns relevant results | Matches should appear for any substring |
| Navigate-back correctly re-highlights text on static pages | 80%+ success rate |
| Extension works without errors on Chrome latest version | Zero crash errors |

---

## 10. Future Enhancements

These are ideas for extending HighlightHub beyond the MVP. Each one can be a standalone project for further learning.

| Enhancement | Brief Description |
|-------------|------------------|
| **Tags and Folders** | Let users organize highlights with custom tags or group them into folders/collections |
| **Export to Markdown/PDF** | Allow users to export all highlights (or filtered highlights) as a Markdown file or PDF |
| **Screenshot Capture** | Save a screenshot of the highlighted area along with the text for visual reference |
| **Voice Notes** | Let users attach a short voice recording to any highlight as a personal annotation |
| **Cloud Sync** | Sync highlights across devices using a backend service (Firebase, Supabase) |
| **AI Summary** | Use an AI API to generate a summary of all highlights from a specific page or topic |
| **Color Coding** | Let users choose different highlight colors to categorize information visually |
| **Keyboard Shortcut** | Allow saving highlights via a keyboard shortcut (e.g., Ctrl+Shift+H) without clicking |
| **Side Panel View** | Show highlights in Chrome's side panel instead of a separate tab for quicker access |
| **Share Highlights** | Generate a shareable link with all highlights from a specific page |

---

## Glossary of Chrome Extension Terms

| Term | Definition |
|------|-----------|
| **Manifest (manifest.json)** | The configuration file every Chrome extension must have. It tells Chrome the extension's name, version, permissions, and which scripts to load |
| **Manifest V3** | The latest version of Chrome's extension platform. It is more secure and performant than the older V2 |
| **Content Script** | JavaScript that runs inside web pages the user visits. It can read and modify the page's content |
| **Background Script / Service Worker** | A script that runs in the background, separate from any webpage. It handles events and coordinates between other parts of the extension |
| **Popup** | The small HTML page that appears when a user clicks the extension icon in the toolbar |
| **Chrome Storage API** | A built-in API that lets extensions save data locally on the user's machine (similar to localStorage but designed for extensions) |
| **chrome.runtime.sendMessage()** | A method to send messages between different parts of the extension (e.g., from a content script to the background script) |
| **activeTab Permission** | Gives the extension temporary access to the current tab when the user interacts with the extension |
| **Content Security Policy (CSP)** | Security rules that control what scripts and resources the extension can load |

---

*This PRD is a reference document for the Codebasics AI Toolkit Webinar. Learners are encouraged to use this as a guide while building the extension with AI coding tools.*
