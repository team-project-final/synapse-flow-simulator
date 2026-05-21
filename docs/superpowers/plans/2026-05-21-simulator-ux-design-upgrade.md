# Simulator UX/Design Upgrade Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade the Synapse Flow Simulator with search/filter navigation, a UX view for user journeys, a 3-mode system (learning/presentation/reference), architecture view readability improvements, and design polish.

**Architecture:** Single-file SPA (index.html) with vanilla HTML/CSS/JS, data from scenarios.json. All changes go into these two files. The approach is layered: each task produces a complete, testable increment. State management uses existing global variables extended with new state for mode, search, and UX view.

**Tech Stack:** HTML5, CSS3 (custom properties), vanilla JavaScript, SVG, Tabler Icons, Google Fonts (IBM Plex Mono, Space Grotesk)

---

## File Structure

All changes target exactly two files:

- **Modify:** `index.html` — CSS styles (lines 12-178), HTML structure (lines 180-235), JavaScript (lines 237-805)
- **Modify:** `data/scenarios.json` — Add optional `guide` field to steps for learning mode

No new files are created. No build tools. No dependencies.

**Deferred to follow-up:** The following spec items add significant complexity and are better as a separate iteration after the core upgrade lands:
- **Minimap (spec 4-4):** Requires a second miniaturized SVG render + viewport tracking. Moderate value vs high complexity.
- **Actor grouping (spec 4-1):** Requires defining group relationships in scenarios.json and a grouping layout algorithm. Worth doing but independent of other changes.
- **Play button SVG morphing (spec 5-2):** Cosmetic enhancement, current icon swap works fine.
- **`guide` field in scenarios.json (spec 3-2):** The learn mode currently uses enhanced annotations styling. Adding the `guide` field to all 25 scenarios' steps is a data authoring task, not a code task.

---

## Task 1: Search Bar + Real-time Filtering

**Files:**
- Modify: `index.html` (CSS + HTML sidebar + JS `renderSidebar` / new `filterScenarios`)

- [ ] **Step 1: Add search bar CSS styles**

Add after the `.sidebar .sc-item.active` rule (around line 56):

```css
.sidebar .search-wrap{padding:var(--sp-2) var(--sp-3);position:sticky;top:0;background:var(--bg-secondary);z-index:2}
.sidebar .search-input{width:100%;padding:var(--sp-2) var(--sp-3);padding-left:32px;background:var(--bg);border:1px solid var(--border);border-radius:6px;color:var(--text);font-family:inherit;font-size:12px;outline:none;transition:border-color .15s}
.sidebar .search-input:focus{border-color:var(--cyan)}
.sidebar .search-input::placeholder{color:var(--text-dim)}
.sidebar .search-icon{position:absolute;left:calc(var(--sp-3) + 8px);top:50%;transform:translateY(-50%);color:var(--text-dim);font-size:14px;pointer-events:none}
.sidebar .search-clear{position:absolute;right:calc(var(--sp-3) + 6px);top:50%;transform:translateY(-50%);color:var(--text-dim);font-size:14px;cursor:pointer;display:none;background:none;border:none;padding:2px}
.sidebar .search-clear.visible{display:block}
.sidebar .no-results{padding:var(--sp-4) var(--sp-3);text-align:center;color:var(--text-dim);font-size:12px}
.sidebar .no-results button{color:var(--cyan);background:none;border:none;cursor:pointer;font-size:12px;text-decoration:underline;margin-top:var(--sp-2);display:block;width:100%;font-family:inherit}
```

- [ ] **Step 2: Add search state and filter function**

Add after the `let selectedBranch = null;` line (around line 248):

```javascript
let searchQuery = '';
let activeFilters = { categories: [], statuses: [], priorities: [] };
let debounceTimer = null;

function filterScenarios() {
  const q = searchQuery.toLowerCase().trim();
  return data.scenarios.filter(sc => {
    // Text search
    if (q) {
      const searchable = [
        sc.title, sc.description,
        ...(sc.ui?.tags || []),
        sc.id, sc.category
      ].join(' ').toLowerCase();
      if (!searchable.includes(q)) return false;
    }
    // Category filter
    if (activeFilters.categories.length && !activeFilters.categories.includes(sc.category)) return false;
    // Status filter
    if (activeFilters.statuses.length && !activeFilters.statuses.includes(sc.status)) return false;
    // Priority filter
    if (activeFilters.priorities.length && !activeFilters.priorities.includes(sc.priority)) return false;
    return true;
  });
}
```

- [ ] **Step 3: Update renderSidebar to include search bar and use filtering**

Replace the entire `renderSidebar` function:

```javascript
function renderSidebar() {
  const sb = document.getElementById('sidebar');
  const filtered = searchQuery || activeFilters.categories.length || activeFilters.statuses.length || activeFilters.priorities.length
    ? filterScenarios()
    : data.scenarios;
  const filteredIds = new Set(filtered.map(s => s.id));

  let html = '';
  // Search bar
  html += `<div class="search-wrap" style="position:relative">
    <i class="ti ti-search search-icon"></i>
    <input class="search-input" id="search-input" type="text" placeholder="시나리오 검색 (Ctrl+K)" value="${escapeHtml(searchQuery)}" oninput="onSearchInput(this.value)" aria-label="시나리오 검색">
    <button class="search-clear ${searchQuery ? 'visible' : ''}" onclick="clearSearch()" aria-label="검색 초기화"><i class="ti ti-x"></i></button>
  </div>`;

  // Scenario list
  if (filtered.length === 0) {
    html += '<div class="no-results"><p>검색 결과 없음</p><button onclick="clearSearch()">필터 초기화</button></div>';
  } else {
    data.categories.forEach(cat => {
      const catScenarios = data.scenarios.filter(s => s.category === cat.id && filteredIds.has(s.id));
      if (catScenarios.length === 0) return;
      const catColor = COLOR_MAP[cat.color] || '#6b7280';
      html += `<div class="cat-group"><div class="cat-header"><span class="cat-dot" style="background:${catColor}"></span>${cat.label}</div>`;
      catScenarios.forEach(sc => {
        const stepCount = sc.steps.length;
        html += `<button class="sc-item ${currentScenario?.id === sc.id ? 'active' : ''}" data-id="${sc.id}" onclick="selectScenario('${sc.id}')" type="button">
          <span class="status-dot status-${sc.status}" title="${sc.status}"></span>${highlightMatch(sc.title, searchQuery)}
          <span style="float:right;font-size:10px;color:var(--text-dim)">${stepCount}</span>
        </button>`;
      });
      html += '</div>';
    });
  }
  sb.innerHTML = html;
}

function highlightMatch(text, query) {
  if (!query) return escapeHtml(text);
  const escaped = escapeHtml(text);
  const q = query.toLowerCase();
  const idx = text.toLowerCase().indexOf(q);
  if (idx === -1) return escaped;
  const before = escapeHtml(text.slice(0, idx));
  const match = escapeHtml(text.slice(idx, idx + query.length));
  const after = escapeHtml(text.slice(idx + query.length));
  return `${before}<mark style="background:rgba(79,195,247,.25);color:var(--text);border-radius:2px;padding:0 1px">${match}</mark>${after}`;
}

function onSearchInput(value) {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    searchQuery = value;
    renderSidebar();
  }, 150);
}

function clearSearch() {
  searchQuery = '';
  activeFilters = { categories: [], statuses: [], priorities: [] };
  renderSidebar();
  const input = document.getElementById('search-input');
  if (input) input.focus();
}
```

- [ ] **Step 4: Add Ctrl+K keyboard shortcut**

Add to the existing `keydown` event listener, inside the switch (before the closing `}`):

```javascript
    case 'k':
      if (e.ctrlKey || e.metaKey) {
        e.preventDefault();
        const searchInput = document.getElementById('search-input');
        if (searchInput) searchInput.focus();
      }
      break;
```

- [ ] **Step 5: Verify search works**

Open `index.html` in a browser. Test:
- Type "로그인" → only login scenarios visible
- Type "kafka" → scenarios with Kafka tags visible
- Type nonsense → "검색 결과 없음" with reset button
- Click reset → full list restored
- Press Ctrl+K → search input focused
- Clear search → all scenarios visible again

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add search bar with real-time filtering to sidebar"
```

---

## Task 2: Filter Chips (Category / Status / Priority)

**Files:**
- Modify: `index.html` (CSS + sidebar HTML + JS)

- [ ] **Step 1: Add filter chip CSS**

Add after the search bar styles:

```css
.sidebar .filter-area{padding:0 var(--sp-3) var(--sp-2);display:flex;flex-wrap:wrap;gap:4px}
.sidebar .filter-chip{padding:2px 8px;border-radius:12px;font-size:10px;font-weight:500;border:1px solid var(--border);background:var(--bg);color:var(--text-muted);cursor:pointer;transition:.15s;white-space:nowrap;font-family:inherit}
.sidebar .filter-chip:hover{border-color:var(--text-muted)}
.sidebar .filter-chip.active{background:rgba(79,195,247,.15);border-color:var(--cyan);color:var(--cyan)}
.sidebar .filter-count{display:inline-flex;align-items:center;justify-content:center;min-width:16px;height:16px;border-radius:8px;font-size:9px;font-weight:700;background:var(--cyan);color:var(--bg);margin-left:4px}
```

- [ ] **Step 2: Add filter chip rendering in renderSidebar**

Insert the filter chip HTML after the search bar `</div>` in `renderSidebar`, before the scenario list:

```javascript
  // Filter chips
  const totalActive = activeFilters.categories.length + activeFilters.statuses.length + activeFilters.priorities.length;
  html += '<div class="filter-area">';
  // Category chips (show all categories)
  data.categories.forEach(cat => {
    const isActive = activeFilters.categories.includes(cat.id);
    html += `<button class="filter-chip ${isActive ? 'active' : ''}" onclick="toggleFilter('categories','${cat.id}')">${cat.label}</button>`;
  });
  // Status chips
  ['implemented', 'partial', 'pending'].forEach(s => {
    const isActive = activeFilters.statuses.includes(s);
    const labels = { implemented: '구현완료', partial: '부분구현', pending: '대기' };
    html += `<button class="filter-chip ${isActive ? 'active' : ''}" onclick="toggleFilter('statuses','${s}')">${labels[s]}</button>`;
  });
  if (totalActive > 0) {
    html += `<button class="filter-chip" onclick="clearSearch()" style="border-color:var(--red);color:var(--red)">초기화<span class="filter-count" style="background:var(--red)">${totalActive}</span></button>`;
  }
  html += '</div>';
```

- [ ] **Step 3: Add toggleFilter function**

Add after the `clearSearch` function:

```javascript
function toggleFilter(type, value) {
  const arr = activeFilters[type];
  const idx = arr.indexOf(value);
  if (idx === -1) arr.push(value);
  else arr.splice(idx, 1);
  renderSidebar();
}
```

- [ ] **Step 4: Verify filter chips work**

Open `index.html` in a browser. Test:
- Click "인증 및 보안" chip → only auth scenarios visible, chip highlighted cyan
- Click again → deactivated
- Combine: "인증 및 보안" + "구현완료" → intersection filter
- Click "초기화" → all filters cleared

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add filter chips for category, status, and priority"
```

---

## Task 3: Scenario Card Info + Recent Scenarios

**Files:**
- Modify: `index.html` (CSS + JS)

- [ ] **Step 1: Add CSS for enhanced scenario items and recent section**

```css
.sidebar .sc-item .sc-meta{display:flex;gap:4px;align-items:center;margin-top:2px}
.sidebar .sc-item .sc-tag{font-size:9px;padding:0 4px;border-radius:3px;background:var(--bg-card);border:1px solid var(--border);color:var(--text-dim)}
.sidebar .sc-item .sc-steps{font-size:10px;color:var(--text-dim)}
.sidebar .recent-section{padding:0 var(--sp-3) var(--sp-1);margin-bottom:var(--sp-1);border-bottom:1px solid var(--border)}
.sidebar .recent-label{font-size:10px;font-weight:600;color:var(--text-dim);text-transform:uppercase;letter-spacing:.5px;margin-bottom:var(--sp-1)}
```

- [ ] **Step 2: Add recent scenario tracking state**

Add after `let debounceTimer = null;`:

```javascript
function getRecentScenarios() {
  try {
    const raw = sessionStorage.getItem('recent-scenarios');
    return raw ? JSON.parse(raw) : [];
  } catch { return []; }
}

function addRecentScenario(id) {
  let recent = getRecentScenarios().filter(r => r !== id);
  recent.unshift(id);
  if (recent.length > 3) recent = recent.slice(0, 3);
  sessionStorage.setItem('recent-scenarios', JSON.stringify(recent));
}
```

- [ ] **Step 3: Update renderSidebar to show recent section and enriched cards**

In `renderSidebar`, after the filter chips HTML and before the scenario list, add the recent section:

```javascript
  // Recent scenarios
  const recentIds = getRecentScenarios();
  const recentScenarios = recentIds.map(id => data.scenarios.find(s => s.id === id)).filter(Boolean);
  if (recentScenarios.length > 0 && !searchQuery && !totalActive) {
    html += '<div class="recent-section"><div class="recent-label">최근</div>';
    recentScenarios.forEach(sc => {
      html += `<button class="sc-item ${currentScenario?.id === sc.id ? 'active' : ''}" data-id="${sc.id}" onclick="selectScenario('${sc.id}')" type="button" style="padding-left:var(--sp-3)">
        <span class="status-dot status-${sc.status}" title="${sc.status}"></span>${escapeHtml(sc.title)}
      </button>`;
    });
    html += '</div>';
  }
```

Update the scenario button rendering in the category loop to include tags and step count:

```javascript
      catScenarios.forEach(sc => {
        const stepCount = sc.steps.length;
        const tags = (sc.ui?.tags || []).slice(0, 3);
        html += `<button class="sc-item ${currentScenario?.id === sc.id ? 'active' : ''}" data-id="${sc.id}" onclick="selectScenario('${sc.id}')" type="button">
          <div><span class="status-dot status-${sc.status}" title="${sc.status}"></span>${highlightMatch(sc.title, searchQuery)}</div>
          <div class="sc-meta">
            ${tags.map(t => `<span class="sc-tag">${escapeHtml(t)}</span>`).join('')}
            <span class="sc-steps">${stepCount}steps</span>
          </div>
        </button>`;
      });
```

- [ ] **Step 4: Call addRecentScenario in selectScenario**

At the top of the `selectScenario` function, after `currentScenario = sc;`, add:

```javascript
  addRecentScenario(sc.id);
```

- [ ] **Step 5: Verify recent scenarios and enriched cards**

Test:
- Select 3 different scenarios → "최근" section appears with all 3
- Select a 4th → oldest drops off (max 3)
- Each scenario card shows tags and step count
- Search active → recent section hidden

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add recent scenarios section and enriched scenario cards"
```

---

## Task 4: UX View — Tab Extension + Step Filtering

**Files:**
- Modify: `index.html` (HTML tabs + JS view switching + step filtering)

- [ ] **Step 1: Add the UX view tab to HTML**

In the `.view-tabs` div (around line 188), add the third tab button:

```html
    <div class="view-tabs">
      <button id="tab-arch" class="active" onclick="switchView('arch')">아키텍처 뷰</button>
      <button id="tab-seq" onclick="switchView('seq')">시퀀스 뷰</button>
      <button id="tab-ux" onclick="switchView('ux')">사용자 경험 뷰</button>
    </div>
```

- [ ] **Step 2: Update switchView to handle 'ux'**

Replace the `switchView` function:

```javascript
function switchView(view) {
  currentView = view;
  document.getElementById('tab-arch').classList.toggle('active', view === 'arch');
  document.getElementById('tab-seq').classList.toggle('active', view === 'seq');
  document.getElementById('tab-ux').classList.toggle('active', view === 'ux');
  if (currentScenario) renderView();
}
```

- [ ] **Step 3: Add step filtering helpers**

Add after the `getScenarioActors` function:

```javascript
function getFilteredSteps() {
  if (!currentScenario) return [];
  if (currentView === 'ux') return currentScenario.steps; // UX view shows all, highlights ui_action
  // Arch and Seq views: exclude ui_action steps and screen actors
  return currentScenario.steps.filter(s => s.type !== 'ui_action');
}

function getFilteredActors() {
  if (!currentScenario) return [];
  if (currentView === 'ux') return getScenarioActors();
  // Arch and Seq views: exclude screen-kind actors
  return getScenarioActors().filter(a => a.kind !== 'screen');
}

function hasScreenActors() {
  if (!currentScenario) return false;
  return getScenarioActors().some(a => a.kind === 'screen');
}
```

- [ ] **Step 4: Update renderView to route to ux**

Replace the `renderView` function:

```javascript
function renderView() {
  if (!currentScenario) return;
  if (currentView === 'arch') renderArchView();
  else if (currentView === 'seq') renderSeqView();
  else if (currentView === 'ux') renderUxView();
}
```

- [ ] **Step 5: Update renderArchView to use filtered actors/steps**

In `renderArchView`, change `const actors = getScenarioActors();` to:

```javascript
  const actors = getFilteredActors();
```

And change `const step = currentScenario.steps[currentStep];` to:

```javascript
  const allSteps = getFilteredSteps();
  const step = allSteps[Math.min(currentStep, allSteps.length - 1)];
```

- [ ] **Step 6: Update renderSeqView to use filtered actors/steps**

In `renderSeqView`, change `const actors = getScenarioActors();` to:

```javascript
  const actors = getFilteredActors();
```

And change `const steps = currentScenario.steps;` to:

```javascript
  const steps = getFilteredSteps();
```

Also update references to `currentStep` in the loop to clamp: change `const isCurrent = si === currentStep;` and `const isFuture = si > currentStep;` to use a mapped index that accounts for filtered steps:

```javascript
  const mappedStep = Math.min(currentStep, steps.length - 1);
```

Then use `mappedStep` instead of `currentStep` in the comparisons.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add UX view tab and step/actor filtering for view separation"
```

---

## Task 5: UX View — Card Flow Rendering

**Files:**
- Modify: `index.html` (CSS for UX cards + JS `renderUxView`)

- [ ] **Step 1: Add UX view CSS**

```css
/* UX View */
.ux-view{width:100%;height:100%;overflow-x:auto;overflow-y:auto;padding:var(--sp-4);display:flex;flex-direction:column}
.ux-empty{display:flex;flex-direction:column;align-items:center;justify-content:center;height:100%;color:var(--text-dim);gap:var(--sp-3);text-align:center}
.ux-empty i{font-size:48px}
.ux-empty a{color:var(--cyan);cursor:pointer;text-decoration:none}
.ux-empty a:hover{text-decoration:underline}
.ux-flow{display:flex;gap:var(--sp-4);align-items:flex-start;min-height:200px;padding-bottom:var(--sp-4)}
.ux-card{min-width:200px;max-width:240px;background:var(--bg-card);border:2px solid var(--border);border-radius:12px;padding:var(--sp-3);flex-shrink:0;transition:border-color .3s,box-shadow .3s}
.ux-card.active{border-color:var(--ui-action);box-shadow:0 0 12px rgba(74,222,128,.2)}
.ux-card-header{display:flex;align-items:center;gap:var(--sp-2);margin-bottom:var(--sp-2);padding-bottom:var(--sp-2);border-bottom:1px solid var(--border)}
.ux-card-header i{font-size:18px;color:var(--ui-action)}
.ux-card-header span{font-size:12px;font-weight:600;color:var(--text)}
.ux-card-actions{margin-bottom:var(--sp-2)}
.ux-action-item{display:flex;align-items:flex-start;gap:var(--sp-2);font-size:11px;color:var(--text-muted);padding:3px 0;border-radius:4px;transition:background .15s}
.ux-action-item.active{background:rgba(74,222,128,.1);color:var(--ui-action)}
.ux-action-item::before{content:'○';font-size:10px;color:var(--text-dim);margin-top:1px;flex-shrink:0}
.ux-action-item.active::before{content:'●';color:var(--ui-action)}
.ux-card-apis{border-top:1px solid var(--border);padding-top:var(--sp-2);display:flex;flex-wrap:wrap;gap:4px}
.ux-api-badge{font-size:9px;padding:1px 6px;border-radius:3px;border:1px solid var(--border);color:var(--text-muted);transition:border-color .3s,color .3s}
.ux-api-badge.active{border-color:var(--blue);color:var(--blue);animation:glowPulse 2s ease-in-out infinite;--glow-color:rgba(79,195,247,.15)}
.ux-connector{display:flex;align-items:center;flex-shrink:0;color:var(--text-dim);font-size:18px;margin:0 -4px}
```

- [ ] **Step 2: Implement renderUxView**

```javascript
function renderUxView() {
  const mv = document.getElementById('main-view');
  if (!hasScreenActors()) {
    // Empty state for non-journey scenarios
    const journeyScenarios = data.scenarios.filter(s =>
      s.category === 'journey-learner' || s.category === 'journey-admin'
    );
    let links = journeyScenarios.slice(0, 3).map(s =>
      `<a onclick="selectScenario('${s.id}')">${escapeHtml(s.title)}</a>`
    ).join('<br>');
    mv.innerHTML = `<div class="ux-empty">
      <i class="ti ti-device-mobile-off"></i>
      <p style="font-size:14px">이 시나리오에는 사용자 경험 흐름이<br>정의되어 있지 않습니다</p>
      <div style="font-size:12px;margin-top:var(--sp-2)">
        <p style="margin-bottom:var(--sp-2)">다음 여정 시나리오를 확인해보세요:</p>
        ${links}
      </div>
    </div>`;
    return;
  }

  // Build screen cards from steps
  const screens = buildScreenCards();
  const step = currentScenario.steps[currentStep];

  let html = '<div class="ux-view"><div class="ux-flow">';

  screens.forEach((screen, idx) => {
    const isActive = screen.actions.some(a => a.stepIdx === currentStep) ||
                     screen.apis.some(a => a.stepIdx === currentStep);

    html += `<div class="ux-card ${isActive ? 'active' : ''}">`;
    // Header
    html += `<div class="ux-card-header"><i class="${screen.icon}"></i><span>${escapeHtml(screen.label)}</span></div>`;
    // UI Actions
    if (screen.actions.length) {
      html += '<div class="ux-card-actions">';
      screen.actions.forEach(a => {
        const isCurrentAction = a.stepIdx === currentStep;
        html += `<div class="ux-action-item ${isCurrentAction ? 'active' : ''}">${escapeHtml(a.action)}</div>`;
      });
      html += '</div>';
    }
    // API calls
    if (screen.apis.length) {
      html += '<div class="ux-card-apis">';
      screen.apis.forEach(a => {
        const isCurrentApi = a.stepIdx === currentStep;
        html += `<span class="ux-api-badge ${isCurrentApi ? 'active' : ''}">${escapeHtml(truncate(a.action, 25))}</span>`;
      });
      html += '</div>';
    }
    html += '</div>';

    // Connector arrow between cards
    if (idx < screens.length - 1) {
      html += '<div class="ux-connector"><i class="ti ti-chevron-right"></i></div>';
    }
  });

  html += '</div></div>';
  mv.innerHTML = html;

  // Auto-scroll to active card
  const activeCard = mv.querySelector('.ux-card.active');
  if (activeCard) {
    activeCard.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' });
  }
}

function buildScreenCards() {
  const steps = currentScenario.steps;
  const cards = [];
  let currentScreen = null;

  steps.forEach((step, idx) => {
    if (step.type === 'ui_action') {
      // This is a UI action — find or create the target screen card
      const toActor = actorMap[step.to];
      const fromActor = actorMap[step.from];
      // The target screen is step.to if it's a screen, or step.from if it's a screen
      const screenActor = (toActor?.kind === 'screen') ? toActor : (fromActor?.kind === 'screen' ? fromActor : null);
      if (screenActor) {
        // Find existing card for this screen, or create new one
        let card = cards.find(c => c.id === screenActor.id && c === cards[cards.length - 1]);
        if (!card || card.id !== screenActor.id) {
          card = {
            id: screenActor.id,
            label: screenActor.label,
            icon: screenActor.icon || 'ti ti-app-window',
            actions: [],
            apis: []
          };
          cards.push(card);
        }
        card.actions.push({ action: step.action, stepIdx: idx });
        currentScreen = card;
      }
    } else if (currentScreen) {
      // Non-UI step — attach as API call to the current screen card
      currentScreen.apis.push({ action: step.action, stepIdx: idx });
    }
  });

  return cards;
}
```

- [ ] **Step 3: Verify UX view works**

Test:
- Select "P1 김시냅스" journey → UX view shows screen cards with actions and API badges
- Step through → active card highlights, current action shows filled circle
- Select "auth-login-basic" (no screens) → empty state with journey links
- Click a journey link → switches to that scenario

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add UX view with screen flow card visualization"
```

---

## Task 6: Mode System — State + UI + Layout Switching

**Files:**
- Modify: `index.html` (CSS + HTML header + JS mode system)

- [ ] **Step 1: Add mode system CSS**

```css
/* Mode System */
.mode-switcher{display:flex;gap:2px;background:var(--bg-tertiary);border-radius:8px;padding:2px;margin-left:var(--sp-3)}
.mode-switcher button{padding:4px 10px;border-radius:6px;font-size:11px;font-weight:500;transition:.15s;white-space:nowrap;min-height:28px;display:inline-flex;align-items:center;gap:4px}
.mode-switcher button.active{background:rgba(79,195,247,.15);color:var(--cyan)}

/* Presentation mode */
.app.mode-presentation{grid-template-columns:0 1fr 0;grid-template-rows:var(--header-h) 1fr 0}
.app.mode-presentation .sidebar{transform:translateX(-100%);position:fixed;z-index:20}
.app.mode-presentation .detail-panel{position:fixed;bottom:0;left:0;right:0;height:auto;max-height:60px;padding:var(--sp-2) var(--sp-4);border-left:none;border-top:1px solid var(--border);z-index:10;overflow:hidden;grid-row:auto;top:auto;width:100%}
.app.mode-presentation .log-panel{display:none}
.app.mode-presentation .main-view{grid-column:1/-1}
.app.mode-presentation .scenario-title-overlay{position:absolute;top:var(--sp-4);left:50%;transform:translateX(-50%);font-family:'Space Grotesk',sans-serif;font-size:24px;font-weight:600;color:var(--text);z-index:5;text-align:center;text-shadow:0 2px 8px rgba(0,0,0,.5)}

/* Reference mode */
.app.mode-reference{grid-template-columns:300px 1fr var(--detail-w)}
.app.mode-reference .sidebar{width:300px}
```

- [ ] **Step 2: Add mode state and URL hash support**

Add after `let selectedBranch = null;`:

```javascript
let currentMode = 'learn'; // 'learn' | 'presentation' | 'reference'

function initMode() {
  const hash = window.location.hash;
  const modeMatch = hash.match(/mode=(learn|presentation|reference)/);
  if (modeMatch) currentMode = modeMatch[1];
  else {
    const saved = sessionStorage.getItem('simulator-mode');
    if (saved && ['learn', 'presentation', 'reference'].includes(saved)) currentMode = saved;
  }
  applyMode();
}

function setMode(mode) {
  currentMode = mode;
  sessionStorage.setItem('simulator-mode', mode);
  applyMode();
  if (currentScenario) {
    renderView();
    renderDetail();
  }
}

function applyMode() {
  const app = document.querySelector('.app');
  app.classList.remove('mode-learn', 'mode-presentation', 'mode-reference');
  app.classList.add('mode-' + currentMode);
  // Update mode buttons
  document.querySelectorAll('.mode-switcher button').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.mode === currentMode);
  });
  // Presentation mode: auto-play
  if (currentMode === 'presentation' && currentScenario && !isPlaying) {
    startPlay();
  }
}
```

- [ ] **Step 3: Add mode switcher to header HTML**

After the `.view-tabs` div and before the `.nav-controls` div in the header, add:

```html
    <div class="mode-switcher">
      <button data-mode="learn" onclick="setMode('learn')">📖 학습</button>
      <button data-mode="presentation" onclick="setMode('presentation')">🎬 발표</button>
      <button data-mode="reference" onclick="setMode('reference')">🔍 레퍼런스</button>
    </div>
```

- [ ] **Step 4: Update renderDetail for mode-aware content**

Add at the top of `renderDetail`, after the null check:

```javascript
  // Presentation mode: minimal detail
  if (currentMode === 'presentation') {
    const step = sc.steps[currentStep];
    if (step) {
      dc.innerHTML = `<div style="font-size:12px;color:var(--text-muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis">
        <strong>${step.n}</strong> ${escapeHtml(step.action)}
      </div>`;
    } else {
      dc.innerHTML = '';
    }
    return;
  }
```

For reference mode, add JSON auto-expand after the payload section:

```javascript
    // Reference mode: expanded payload, Kafka topic info
    if (currentMode === 'reference' && step.payload) {
      // Payload already shown above, but in reference mode show full JSON
    }
    if (currentMode === 'reference' && step.topic) {
      const topicData = data.topics?.find(t => t.name === step.topic);
      if (topicData) {
        html += `<div class="section"><h3>토픽 상세</h3>
          <div style="font-size:11px;color:var(--text-muted);line-height:1.6">
            <div><strong>프로듀서:</strong> ${topicData.producers.join(', ')}</div>
            <div><strong>컨슈머:</strong> ${topicData.consumers.join(', ')}</div>
            <div><strong>호환성:</strong> ${topicData.compatibility}</div>
            ${topicData.description ? `<div style="margin-top:4px">${escapeHtml(topicData.description)}</div>` : ''}
          </div>
        </div>`;
      }
    }
```

For learn mode, enhance annotations display:

```javascript
    // Learn mode: enhanced annotations
    if (step.annotations?.length) {
      html += '<div class="section"><h3>주석</h3>';
      step.annotations.forEach(a => {
        if (currentMode === 'learn') {
          html += `<div class="annotation" style="background:var(--bg);padding:var(--sp-2);border-radius:4px;border-left:3px solid var(--amber);margin-bottom:var(--sp-2)">${escapeHtml(a)}</div>`;
        } else {
          html += `<div class="annotation">${escapeHtml(a)}</div>`;
        }
      });
      html += '</div>';
    }
```

- [ ] **Step 5: Add presentation mode title overlay**

In `renderArchView` and `renderSeqView`, when in presentation mode, add a title overlay at the top of the view:

```javascript
  // After setting mv.innerHTML, before the closing of the function:
  if (currentMode === 'presentation') {
    const overlay = document.createElement('div');
    overlay.className = 'scenario-title-overlay';
    overlay.textContent = currentScenario.title;
    mv.querySelector('.arch-view,.seq-view,.ux-view')?.prepend(overlay);
  }
```

- [ ] **Step 6: Add fullscreen toggle (F key) for presentation mode**

Add to the keyboard listener:

```javascript
    case 'f':
    case 'F':
      if (currentMode === 'presentation') {
        e.preventDefault();
        if (document.fullscreenElement) document.exitFullscreen();
        else document.documentElement.requestFullscreen();
      }
      break;
```

- [ ] **Step 7: Call initMode in the boot sequence**

In the boot section at the bottom of the script, add `initMode();` after `initColors();`:

```javascript
initColors();
initMode();
init();
```

- [ ] **Step 8: Verify mode system**

Test:
- Click "📖 학습" → full layout with enhanced annotations
- Click "🎬 발표" → sidebar hidden, detail minimized, auto-play starts, title overlay
- Press F → fullscreen toggle
- Click "🔍 레퍼런스" → sidebar 300px, topic details shown
- Refresh → mode persisted via sessionStorage
- Add `#mode=presentation` to URL → starts in presentation mode

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "feat: add 3-mode system (learn/presentation/reference)"
```

---

## Task 7: Architecture View — Arrow Trails + Action Labels

**Files:**
- Modify: `index.html` (JS `renderArchView` + `drawArchArrow`)

- [ ] **Step 1: Add state for step history trails**

Add after `let currentMode = 'learn';`:

```javascript
let stepTrail = []; // array of past step indices for arrow trails
```

Update `onStepChange` to track trail:

```javascript
function onStepChange() {
  stepTrail.push(currentStep);
  if (stepTrail.length > 20) stepTrail.shift(); // keep last 20
  renderView();
  renderDetail();
  updateStepInfo();
  logStep(currentStep);
}
```

Update `resetSteps` to clear trail:

```javascript
  stepTrail = [];
```

- [ ] **Step 2: Update drawArchArrow to render trail arrows**

Replace the `drawArchArrow` function:

```javascript
function drawArchArrow(currentStepObj, positions) {
  const svg = document.getElementById('arch-svg');
  if (!svg) return;

  let svgContent = '<defs>';
  const drawnArrows = [];
  const steps = getFilteredSteps();

  // Build unique arrow markers
  const markerTypes = new Set();
  steps.forEach(s => markerTypes.add(s.type));
  markerTypes.forEach(type => {
    const ts = TYPE_ARROW[type] || TYPE_ARROW.internal;
    svgContent += `<marker id="ah-${type}" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><polygon points="0 0,8 3,0 6" fill="${ts.color}"/></marker>`;
  });
  svgContent += '</defs>';

  // Draw trail arrows (past steps, faded)
  const trailSteps = stepTrail.slice(0, -1); // exclude current
  const trailSet = new Set();
  trailSteps.forEach(idx => {
    const step = currentScenario.steps[idx];
    if (!step || step.type === 'ui_action') return;
    const key = `${step.from}-${step.to}-${step.type}`;
    if (trailSet.has(key)) return;
    trailSet.add(key);
    const from = positions[step.from];
    const to = positions[step.to];
    if (!from || !to) return;
    const ts = TYPE_ARROW[step.type] || TYPE_ARROW.internal;
    const pathD = buildBezierPath(from, to, step.from === step.to);
    const dashAttr = ts.dash ? `stroke-dasharray="${ts.dash}"` : '';
    svgContent += `<path d="${pathD}" fill="none" stroke="${ts.color}" stroke-width="1.5" ${dashAttr} opacity="0.25" marker-end="url(#ah-${step.type})"/>`;
  });

  // Draw current step arrow (bold)
  if (currentStepObj) {
    const from = positions[currentStepObj.from];
    const to = positions[currentStepObj.to];
    if (from && to) {
      const ts = TYPE_ARROW[currentStepObj.type] || TYPE_ARROW.internal;
      const pathD = buildBezierPath(from, to, currentStepObj.from === currentStepObj.to);
      const dashAttr = ts.dash ? `stroke-dasharray="${ts.dash}"` : '';
      let animDash = '';
      if (ts.dash) animDash = `<animate attributeName="stroke-dashoffset" from="0" to="-20" dur="0.8s" repeatCount="indefinite"/>`;
      svgContent += `<path d="${pathD}" fill="none" stroke="${ts.color}" stroke-width="3" ${dashAttr} opacity="0.9" marker-end="url(#ah-${currentStepObj.type})">${animDash}</path>`;
      svgContent += `<circle r="4" fill="${ts.color}" opacity="0.9"><animateMotion dur="1.2s" repeatCount="indefinite" path="${pathD}"/></circle>`;

      // Action label (always visible)
      const mx = (from.x + to.x) / 2;
      const my = (from.y + to.y) / 2;
      svgContent += `<text x="${mx}" y="${my - 8}" text-anchor="middle" fill="var(--text)" font-size="10" font-weight="500">${escapeHtml(truncate(currentStepObj.action, 40))}</text>`;
      if (currentStepObj.topic) {
        svgContent += `<text x="${mx}" y="${my + 6}" text-anchor="middle" fill="${ts.color}" font-size="9" opacity="0.7">${currentStepObj.topic.split('.').pop()}</text>`;
      }
    }
  }

  svg.innerHTML = svgContent;

  // Error shake
  if (currentStepObj?.type === 'error') {
    [currentStepObj.from, currentStepObj.to].forEach(id => {
      const box = document.querySelector(`[data-actor="${id}"]`);
      if (box) { box.classList.add('shake'); setTimeout(() => box.classList.remove('shake'), 600); }
    });
  }
}

function buildBezierPath(from, to, isSelf) {
  if (isSelf) {
    return `M ${from.x + 50} ${from.y} C ${from.x + 110} ${from.y - 40}, ${from.x + 110} ${from.y + 40}, ${from.x + 50} ${from.y}`;
  }
  const dx = to.x - from.x;
  const dy = to.y - from.y;
  const cx1 = from.x + dx * 0.3;
  const cy1 = from.y + dy * 0.05;
  const cx2 = from.x + dx * 0.7;
  const cy2 = to.y - dy * 0.05;
  return `M ${from.x} ${from.y} C ${cx1} ${cy1}, ${cx2} ${cy2}, ${to.x} ${to.y}`;
}
```

- [ ] **Step 3: Verify arrow trails**

Test:
- Select a scenario, step through → past arrows appear faded
- Current arrow is bold with action label always visible
- Reset → trails cleared

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add arrow trails and always-visible action labels in architecture view"
```

---

## Task 8: Zoom & Pan for Architecture and Sequence Views

**Files:**
- Modify: `index.html` (CSS for zoom controls + JS zoom/pan logic)

- [ ] **Step 1: Add zoom controls CSS**

```css
/* Zoom Controls */
.zoom-controls{position:absolute;bottom:var(--sp-3);right:var(--sp-3);display:flex;flex-direction:column;gap:4px;z-index:6}
.zoom-controls button{width:32px;height:32px;border-radius:6px;background:var(--bg-card);border:1px solid var(--border);color:var(--text);font-size:14px;display:flex;align-items:center;justify-content:center;cursor:pointer;transition:.15s}
.zoom-controls button:hover{background:var(--border)}
```

- [ ] **Step 2: Add zoom/pan state and functions**

```javascript
let zoomLevel = 1;
let panX = 0, panY = 0;
let isPanning = false;
let panStartX = 0, panStartY = 0;

function resetZoom() {
  zoomLevel = 1;
  panX = 0;
  panY = 0;
  applyZoomPan();
}

function zoomIn() {
  zoomLevel = Math.min(3, zoomLevel * 1.2);
  applyZoomPan();
}

function zoomOut() {
  zoomLevel = Math.max(0.5, zoomLevel / 1.2);
  applyZoomPan();
}

function applyZoomPan() {
  const container = document.querySelector('.arch-view, .seq-view');
  if (!container) return;
  const inner = container.querySelector('.arch-actors, .seq-lifeline-area');
  const svg = container.querySelector('svg');
  const transform = `translate(${panX}px, ${panY}px) scale(${zoomLevel})`;
  if (inner) inner.style.transform = transform;
  if (svg) svg.style.transform = transform;
  [inner, svg].forEach(el => { if (el) el.style.transformOrigin = '0 0'; });
}

function initZoomPan() {
  const mv = document.getElementById('main-view');

  mv.addEventListener('wheel', e => {
    if (currentView === 'ux') return;
    e.preventDefault();
    if (e.deltaY < 0) zoomIn();
    else zoomOut();
  }, { passive: false });

  mv.addEventListener('mousedown', e => {
    if (currentView === 'ux') return;
    if (e.button !== 0) return;
    isPanning = true;
    panStartX = e.clientX - panX;
    panStartY = e.clientY - panY;
    mv.style.cursor = 'grabbing';
  });

  window.addEventListener('mousemove', e => {
    if (!isPanning) return;
    panX = e.clientX - panStartX;
    panY = e.clientY - panStartY;
    applyZoomPan();
  });

  window.addEventListener('mouseup', () => {
    if (isPanning) {
      isPanning = false;
      document.getElementById('main-view').style.cursor = '';
    }
  });
}
```

- [ ] **Step 3: Add zoom control buttons to the view**

In `renderArchView` and `renderSeqView`, after setting `mv.innerHTML`, append zoom controls:

```javascript
  // Add zoom controls
  if (currentMode !== 'presentation') {
    const zc = document.createElement('div');
    zc.className = 'zoom-controls';
    zc.innerHTML = `
      <button onclick="zoomIn()" title="확대" aria-label="확대"><i class="ti ti-plus"></i></button>
      <button onclick="zoomOut()" title="축소" aria-label="축소"><i class="ti ti-minus"></i></button>
      <button onclick="resetZoom()" title="맞추기" aria-label="화면 맞추기"><i class="ti ti-arrows-maximize"></i></button>
    `;
    mv.appendChild(zc);
  }
```

- [ ] **Step 4: Reset zoom on scenario change and call initZoomPan on boot**

In `selectScenario`, after `currentStep = 0;`:

```javascript
  resetZoom();
```

In the boot section:

```javascript
initColors();
initMode();
initZoomPan();
init();
```

- [ ] **Step 5: Verify zoom and pan**

Test:
- Scroll wheel on architecture view → zooms in/out
- Click and drag → pans the view
- Click "Fit" button → resets to default
- +/- buttons work
- Switch scenario → zoom resets
- Presentation mode → no zoom controls visible

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add zoom and pan controls for architecture and sequence views"
```

---

## Task 9: Design Polish — Transitions + Microinteractions

**Files:**
- Modify: `index.html` (CSS transitions + JS microinteractions)

- [ ] **Step 1: Add CSS transitions for view/mode switching**

```css
/* Smooth transitions */
.main-view>*{animation:viewFadeIn .2s ease-out}
@keyframes viewFadeIn{from{opacity:0}to{opacity:1}}

.sidebar{transition:width .3s ease,transform .25s}
.detail-panel{transition:width .3s ease,transform .25s,max-height .3s ease}
.log-panel{transition:height .3s ease}
.app{transition:grid-template-columns .3s ease,grid-template-rows .3s ease}

/* Branch toggle animation */
.branch-detail{overflow:hidden;max-height:0;opacity:0;transition:max-height .3s ease,opacity .2s ease,margin .2s ease,padding .2s ease}
.branch-detail.open{max-height:200px;opacity:1}

/* Hover tooltip for scenario items */
.sidebar .sc-item{position:relative}
.sidebar .sc-item .tooltip{display:none;position:absolute;left:100%;top:0;margin-left:8px;background:var(--bg-card);border:1px solid var(--border);border-radius:8px;padding:var(--sp-2) var(--sp-3);width:200px;z-index:30;pointer-events:none;font-size:11px;color:var(--text-muted);line-height:1.5;box-shadow:0 4px 12px rgba(0,0,0,.3)}
.sidebar .sc-item:hover .tooltip{display:block}
```

- [ ] **Step 2: Add actor card kind icons and subtle gradient backgrounds**

Update `renderArchView` actor box rendering — replace the actor box HTML template:

```javascript
      const kindIcons = {
        client: 'ti-device-desktop', screen: 'ti-app-window', edge: 'ti-network',
        service: 'ti-server-2', datastore: 'ti-database', broker: 'ti-broadcast',
        external: 'ti-cloud', ops: 'ti-settings-automation', k8s: 'ti-box'
      };
      const kindIcon = kindIcons[actor.kind] || actor.icon;
      const gradient = `linear-gradient(135deg, ${color}08, transparent)`;
      html += `<div class="${cls.join(' ')}" style="left:${x}px;top:${y}px;border-color:${borderColor};--glow-color:${color}40;background:${gradient}" data-actor="${actor.id}">
        <i class="ti ${kindIcon} actor-icon" style="color:${color}"></i>
        <span class="actor-label">${actor.label}</span>
        <span class="actor-kind">${actor.kind}</span>
      </div>`;
```

- [ ] **Step 3: Enhance log panel with step-type icons**

Update `logStep` to include type-specific icons:

```javascript
function logStep(idx) {
  if (!currentScenario) return;
  const step = currentScenario.steps[idx];
  if (!step) return;
  const lp = document.getElementById('log-panel');
  const type = step.type;
  const typeIcons = {
    http_request: 'ti-arrow-right', http_response: 'ti-arrow-left', error: 'ti-alert-triangle',
    kafka_publish: 'ti-broadcast', kafka_consume: 'ti-broadcast', db_query: 'ti-database',
    internal: 'ti-arrows-exchange', external_api: 'ti-cloud', cache_op: 'ti-database',
    k8s_event: 'ti-box', git_op: 'ti-git-merge', sync_op: 'ti-refresh', ui_action: 'ti-click'
  };
  let dir = 'evt';
  let dirLabel = 'EVT';
  if (type === 'http_request') { dir = 'req'; dirLabel = 'REQ'; }
  else if (type === 'http_response') { dir = 'res'; dirLabel = 'RES'; }
  else if (type === 'error') { dir = 'err'; dirLabel = 'ERR'; }
  else if (type.startsWith('kafka')) { dir = 'evt'; dirLabel = type === 'kafka_publish' ? 'PUB' : 'SUB'; }
  else if (type === 'ui_action') { dir = 'evt'; dirLabel = 'UI'; }

  const time = new Date().toLocaleTimeString('ko-KR', { hour12: false });
  const topicStr = step.topic ? ` [${step.topic}]` : '';
  const icon = typeIcons[type] || 'ti-point';

  const entry = document.createElement('div');
  entry.className = 'log-entry';
  entry.style.animation = 'fadeInUp .2s ease';
  entry.innerHTML = `<span class="log-n">#${String(step.n).padStart(2, '0')}</span><span class="log-time">${time}</span><i class="ti ${icon}" style="font-size:11px;margin-right:4px;color:${(TYPE_ARROW[type]||TYPE_ARROW.internal).color}"></i><span class="log-dir ${dir}">${dirLabel}</span><span class="log-action">${escapeHtml(step.from)} → ${escapeHtml(step.to)}: ${escapeHtml(step.action)}${topicStr}</span>`;
  lp.appendChild(entry);
  lp.scrollTop = lp.scrollHeight;
}
```

- [ ] **Step 4: Add prefers-reduced-motion override for new animations**

The existing rule at line 175-177 already handles `prefers-reduced-motion:reduce` by setting all animation/transition durations to 0.01ms. The new CSS animations (`viewFadeIn`, branch transitions) are covered by this blanket rule. No additional changes needed.

- [ ] **Step 5: Verify design polish**

Test:
- Switch views → smooth fade-in transition
- Switch modes → layout resizes smoothly
- Hover scenario items → shows tooltip (if tooltips added to renderSidebar)
- Actor cards → kind-specific icons and subtle gradient
- Log panel → type icons visible
- Enable `prefers-reduced-motion` in browser → all animations instant

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add design polish — transitions, actor gradients, log icons"
```

---

## Task 10: Responsive — Tablet Layout + Touch Gestures + Mobile UX View

**Files:**
- Modify: `index.html` (CSS media queries + JS touch handlers)

- [ ] **Step 1: Add tablet breakpoint CSS**

Add a new media query between the existing 900px and 700px breakpoints:

```css
@media(min-width:901px) and (max-width:1200px){
  .app{grid-template-columns:180px 1fr 240px}
  .sidebar{width:180px}
  .detail-panel{width:240px}
  .app.mode-reference{grid-template-columns:240px 1fr 240px}
}
```

- [ ] **Step 2: Add mobile UX view vertical layout**

```css
@media(max-width:700px){
  .ux-flow{flex-direction:column;align-items:stretch}
  .ux-card{max-width:none;min-width:0}
  .ux-connector{transform:rotate(90deg);justify-content:center;margin:0;padding:var(--sp-1) 0}
}
```

- [ ] **Step 3: Add touch gesture handlers**

```javascript
function initTouchGestures() {
  const mv = document.getElementById('main-view');
  let touchStartX = 0;
  let touchStartY = 0;
  let touchStartDist = 0;

  mv.addEventListener('touchstart', e => {
    if (e.touches.length === 1) {
      touchStartX = e.touches[0].clientX;
      touchStartY = e.touches[0].clientY;
    } else if (e.touches.length === 2 && currentView !== 'ux') {
      // Pinch start
      touchStartDist = Math.hypot(
        e.touches[1].clientX - e.touches[0].clientX,
        e.touches[1].clientY - e.touches[0].clientY
      );
    }
  }, { passive: true });

  mv.addEventListener('touchend', e => {
    if (e.changedTouches.length === 1) {
      const dx = e.changedTouches[0].clientX - touchStartX;
      const dy = e.changedTouches[0].clientY - touchStartY;
      // Horizontal swipe for step navigation
      if (Math.abs(dx) > 60 && Math.abs(dx) > Math.abs(dy) * 1.5) {
        if (dx > 0) prevStep();
        else nextStep();
      }
    }
  }, { passive: true });

  mv.addEventListener('touchmove', e => {
    if (e.touches.length === 2 && currentView !== 'ux') {
      const dist = Math.hypot(
        e.touches[1].clientX - e.touches[0].clientX,
        e.touches[1].clientY - e.touches[0].clientY
      );
      if (touchStartDist > 0) {
        const scale = dist / touchStartDist;
        if (scale > 1.05) { zoomIn(); touchStartDist = dist; }
        else if (scale < 0.95) { zoomOut(); touchStartDist = dist; }
      }
    }
  }, { passive: true });

  // Double-tap to reset zoom
  let lastTap = 0;
  mv.addEventListener('touchend', e => {
    const now = Date.now();
    if (now - lastTap < 300 && e.changedTouches.length === 1) {
      resetZoom();
    }
    lastTap = now;
  }, { passive: true });
}
```

- [ ] **Step 4: Call initTouchGestures in boot**

```javascript
initColors();
initMode();
initZoomPan();
initTouchGestures();
init();
```

- [ ] **Step 5: Verify responsive behavior**

Test:
- Resize to 1000px → tablet layout (narrower sidebar/detail)
- Resize to 600px → mobile layout with vertical UX cards
- On touch device: swipe left/right → step navigation
- Pinch → zoom in/out
- Double-tap → reset zoom

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add tablet layout, mobile UX view, and touch gestures"
```

---

## Task 11: Deep Links + URL Hash Routing

**Files:**
- Modify: `index.html` (JS hash routing)

- [ ] **Step 1: Implement URL hash state management**

```javascript
function updateHash() {
  const parts = [];
  if (currentMode !== 'learn') parts.push(`mode=${currentMode}`);
  if (currentScenario) parts.push(`scenario=${currentScenario.id}`);
  if (currentStep > 0) parts.push(`step=${currentStep + 1}`);
  if (currentView !== 'arch') parts.push(`view=${currentView}`);
  const hash = parts.length ? '#' + parts.join('&') : '';
  if (window.location.hash !== hash) {
    history.replaceState(null, '', hash || window.location.pathname);
  }
}

function parseHash() {
  const hash = window.location.hash.slice(1);
  const params = {};
  hash.split('&').forEach(pair => {
    const [k, v] = pair.split('=');
    if (k && v) params[k] = v;
  });
  return params;
}

function restoreFromHash() {
  const params = parseHash();
  if (params.mode && ['learn', 'presentation', 'reference'].includes(params.mode)) {
    currentMode = params.mode;
    applyMode();
  }
  if (params.view && ['arch', 'seq', 'ux'].includes(params.view)) {
    switchView(params.view);
  }
  if (params.scenario && data) {
    selectScenario(params.scenario);
    if (params.step) {
      const stepNum = parseInt(params.step, 10) - 1;
      if (stepNum >= 0 && stepNum < currentScenario?.steps.length) {
        currentStep = stepNum;
        onStepChange();
      }
    }
  }
}
```

- [ ] **Step 2: Call updateHash on state changes**

Add `updateHash();` at the end of:
- `onStepChange()`
- `selectScenario()` (after rendering)
- `setMode()` (after applyMode)
- `switchView()` (after rendering)

- [ ] **Step 3: Call restoreFromHash after data loads**

In the `init` function, after `renderSidebar();`, add:

```javascript
    restoreFromHash();
```

- [ ] **Step 4: Verify deep links**

Test:
- Navigate to `#scenario=auth-login-basic&step=3&view=seq` → loads that scenario at step 3 in sequence view
- Change steps → URL hash updates
- Copy URL, open in new tab → same state restored
- `#mode=presentation&scenario=journey-p1-note-graph-share` → presentation mode for P1

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add URL hash deep linking for scenario, step, view, and mode"
```

---

## Task 12: Final Integration Verification

**Files:**
- No file changes — verification only

- [ ] **Step 1: Cross-feature integration test**

Open `index.html` in a browser and run through this complete test flow:

1. Page loads → learn mode, search bar visible, all 25 scenarios listed
2. Type "김시냅스" in search → P1 scenario visible, other scenarios filtered
3. Clear search → all scenarios back
4. Click "학습자 여정" filter chip → only journey-learner scenarios visible
5. Click "초기화" → all filters cleared
6. Select P1 journey scenario → architecture view renders, recent section appears
7. Switch to "시퀀스 뷰" → sequence view with filtered steps (no ui_action)
8. Switch to "사용자 경험 뷰" → screen flow cards with actions and API badges
9. Play through → cards highlight in sequence, auto-scroll follows
10. Switch to "🎬 발표" mode → full screen, minimal detail, title overlay
11. Press F → fullscreen toggle
12. Switch to "🔍 레퍼런스" mode → wider sidebar, Kafka topic details visible
13. Select "auth-login-basic" → switch to UX view → empty state with journey links
14. Zoom in/out with mouse wheel → view zooms
15. Drag to pan → view pans
16. Click "Fit" → resets zoom
17. Copy URL with hash → open in new tab → same state
18. Resize to 1000px → tablet layout
19. Resize to 600px → mobile layout, UX cards stack vertically

- [ ] **Step 2: Accessibility check**

1. Tab through all interactive elements → focus ring visible
2. Ctrl+K → search focuses
3. Keyboard arrows → step navigation still works
4. Screen reader: check ARIA labels on new elements

- [ ] **Step 3: Commit final adjustments if any**

```bash
git add index.html
git commit -m "fix: integration adjustments from final verification"
```

Only commit if changes were needed. Skip if everything passes.
