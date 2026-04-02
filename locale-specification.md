# Locale Specification — Agent Lab Workshops

**Target locales:** Brazilian Portuguese (`pt_BR`), Spanish (`es`)  
**Scope:** All four `agent-lab-*` repositories (dotnet, java, python, typescript) **and** the `copilot-dev-days.github.io` landing page repository  
**Design reference:** [javaevolved.github.io](https://javaevolved.github.io) locale picker

---

## 1. Overview

Each agent-lab repo is a self-contained workshop built around the same "Soc Ops" Social Bingo game. They share an identical content structure but use different tech stacks. The central landing page at [copilot-dev-days.github.io](https://copilot-dev-days.github.io/) serves as the entry point linking to all workshops.

This specification defines a **unified localization strategy** that works consistently across all five repositories (four workshops + landing page) while respecting each stack's idiomatic patterns. It also defines a **consistent locale selection UI** inspired by [javaevolved.github.io](https://javaevolved.github.io)'s language picker, to be shared across all pages.

### Content categories requiring localization

| Category | Files | Priority |
|----------|-------|----------|
| **Workshop docs** | `workshop/*.md`, `README.md` | 🔴 High |
| **Docs site** | `docs/index.html`, `docs/step.html` | 🔴 High |
| **App UI strings** | Components/templates with hardcoded text | 🟡 Medium |
| **Game data** | Questions/prompts data files | 🟡 Medium |
| **Copilot agent definitions** | `.github/agents/*.md` | 🟢 Low |
| **Copilot instructions** | `.github/instructions/*.md` | 🟢 Low |
| **Copilot prompts** | `.github/prompts/*.md` | 🟢 Low |
| **Solution checkpoints** | `.solutions/step-*/**` | ⚪ Optional |
| **Landing page** | `copilot-dev-days.github.io/index.html` | 🔴 High |

---

## 2. Locale Picker UI Design

All pages across all repos must share a **consistent language-selection UI** modeled after the [javaevolved.github.io](https://javaevolved.github.io) locale picker. This ensures a familiar, professional experience regardless of which workshop or page the user is on.

### 2.1 Design Reference

The reference implementation is the globe-icon dropdown at [javaevolved.github.io](https://javaevolved.github.io):

- A **🌐 globe button** in the top-right of the navigation bar.
- Clicking it reveals a **dropdown listbox** anchored to the button's bottom-right.
- Each option shows a **country flag emoji + language name** in its native script.
- The currently active locale is **highlighted with accent color and bold weight**.
- Clicking an option navigates to the locale-prefixed version of the current page.
- Pressing `Escape` or clicking outside closes the dropdown.

### 2.2 Supported Locales

| Flag | Label | `data-locale` | URL prefix |
|------|-------|---------------|------------|
| 🇬🇧 | English | `en` | `/` (root) |
| 🇧🇷 | Português (Brasil) | `pt-BR` | `/pt_BR/` |
| 🇪🇸 | Español | `es` | `/es/` |

### 2.3 HTML Markup

Use the same semantic structure across all pages. The markup uses ARIA `listbox`/`option` roles for accessibility:

```html
<div class="locale-picker" id="localePicker">
  <button type="button" class="locale-toggle"
          aria-haspopup="listbox" aria-expanded="false"
          aria-label="Select language">🌐</button>
  <ul role="listbox" aria-label="Language">
    <li role="option" data-locale="en" aria-selected="true" class="active">🇬🇧 English</li>
    <li role="option" data-locale="es" aria-selected="false">🇪🇸 Español</li>
    <li role="option" data-locale="pt-BR" aria-selected="false">🇧🇷 Português (Brasil)</li>
  </ul>
</div>
```

### 2.4 CSS

Use CSS custom properties (already present in all repos' stylesheets) for theming. The picker must work in both dark and light themes:

```css
/* --- Locale Picker --- */
.locale-picker {
  position: relative;
  display: inline-flex;
}

.locale-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
  border-radius: var(--radius-sm, 10px);
  border: 1px solid var(--border, #25252f);
  background: var(--surface, #14141a);
  color: var(--text-muted, #6b6b76);
  font-size: 1rem;
  line-height: 1;
  transition: all 0.2s;
  cursor: pointer;
}

.locale-toggle:hover {
  background: var(--accent, #fb923c);
  color: #fff;
  border-color: var(--accent, #fb923c);
}

.locale-picker ul {
  display: none;
  position: absolute;
  top: 100%;
  right: 0;
  margin: 4px 0 0;
  padding: 4px 0;
  list-style: none;
  background: var(--surface, #14141a);
  border: 1px solid var(--border, #25252f);
  border-radius: var(--radius-sm, 10px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 100;
  min-width: 180px;
}

.locale-picker li {
  padding: 8px 16px;
  cursor: pointer;
  font-size: 0.9rem;
  color: var(--text, #e4e4e7);
  white-space: nowrap;
  transition: background 0.15s;
}

.locale-picker li:hover {
  background: var(--accent, #fb923c);
  color: #fff;
}

.locale-picker li.active {
  font-weight: 600;
  color: var(--accent, #fb923c);
}

.locale-picker li.active:hover {
  color: #fff;
}
```

### 2.5 JavaScript Behavior

The picker requires a small script that handles detection, toggle, and navigation:

```js
/* --- Locale Picker --- */
(function () {
  // Detect current locale from URL path
  const detectLocale = () => {
    const match = location.pathname.match(/^\/(pt_BR|es)\//);
    return match ? match[1] : 'en';
  };

  const locale = detectLocale();

  const picker = document.getElementById('localePicker');
  if (!picker) return;

  const toggleBtn = picker.querySelector('.locale-toggle');
  const list = picker.querySelector('ul');

  // Mark the active locale
  list.querySelectorAll('li').forEach(li => {
    const liLocale = li.dataset.locale === 'pt-BR' ? 'pt_BR' : li.dataset.locale;
    if (liLocale === locale) {
      li.classList.add('active');
      li.setAttribute('aria-selected', 'true');
    } else {
      li.classList.remove('active');
      li.setAttribute('aria-selected', 'false');
    }
  });

  const open = () => {
    list.style.display = 'block';
    toggleBtn.setAttribute('aria-expanded', 'true');
  };
  const close = () => {
    list.style.display = 'none';
    toggleBtn.setAttribute('aria-expanded', 'false');
  };

  toggleBtn.addEventListener('click', (e) => {
    e.stopPropagation();
    list.style.display === 'block' ? close() : open();
  });

  document.addEventListener('click', close);
  document.addEventListener('keydown', (e) => { if (e.key === 'Escape') close(); });

  list.querySelectorAll('li').forEach(li => {
    li.addEventListener('click', (e) => {
      e.stopPropagation();
      const target = li.dataset.locale === 'pt-BR' ? 'pt_BR' : li.dataset.locale;
      if (target === locale) { close(); return; }

      let path = location.pathname;

      // Strip current locale prefix
      if (locale !== 'en') {
        path = path.replace(new RegExp('^/' + locale), '');
      }

      // Add target locale prefix
      if (target !== 'en') {
        path = '/' + target + path;
      }

      localStorage.setItem('preferred-locale', target);
      window.location.href = path + location.search;
    });
  });
})();
```

### 2.6 Placement Rules

The locale picker must appear in a consistent location across all pages:

| Page type | Placement | Adjacent to |
|-----------|-----------|-------------|
| **Landing page** (`copilot-dev-days.github.io`) | Fixed top-right toolbar | Theme toggle button |
| **Workshop `docs/index.html`** | Fixed top-right toolbar | Theme toggle button |
| **Workshop `docs/step.html`** | Header navigation bar, right side | Previous/Next buttons |

In all cases, the globe button sits **next to the existing theme-toggle button**, using the same size and border styling. For pages with a fixed top-right toolbar:

```html
<div style="position:fixed;top:1rem;right:1rem;z-index:1000;display:flex;gap:0.5rem;">
    <div class="locale-picker" id="localePicker">
      <!-- ... markup from 2.3 ... -->
    </div>
    <button class="theme-toggle" onclick="toggleTheme()">☀️ Light</button>
</div>
```

For pages with a header nav bar (e.g., `step.html`):

```html
<nav class="header-nav">
    <div class="locale-picker" id="localePicker">
      <!-- ... markup from 2.3 ... -->
    </div>
    <button class="theme-toggle" onclick="toggleTheme()">☀️ Light</button>
    <button class="nav-btn" id="prevBtn" disabled>← Previous</button>
    <button class="nav-btn" id="nextBtn">Next →</button>
</nav>
```

### 2.7 SEO: `hreflang` Tags

All HTML pages must include `<link rel="alternate" hreflang="...">` tags in `<head>` for each locale, plus an `x-default`:

```html
<link rel="alternate" hreflang="en" href="https://copilot-dev-days.github.io/agent-lab-python/">
<link rel="alternate" hreflang="pt-BR" href="https://copilot-dev-days.github.io/agent-lab-python/pt_BR/">
<link rel="alternate" hreflang="es" href="https://copilot-dev-days.github.io/agent-lab-python/es/">
<link rel="alternate" hreflang="x-default" href="https://copilot-dev-days.github.io/agent-lab-python/">
```

---

## 3. Workshop Documentation (`workshop/`)

### Strategy: Locale subdirectories with full markdown copies

Each workshop has 7 markdown files that form the core instructional content. These must be fully translated — not partially or via string substitution — because prose, code comments, and inline examples all need to read naturally in the target language.

### Directory layout

```
workshop/
├── .ignored-by-agents
├── GUIDE.md                    ← English (default)
├── 00-overview.md
├── 01-setup.md
├── 02-design.md
├── 03-quiz-master.md
├── 04-multi-agent.md
├── 05-complete.md
├── pt_BR/
│   ├── GUIDE.md
│   ├── 00-overview.md
│   ├── 01-setup.md
│   ├── 02-design.md
│   ├── 03-quiz-master.md
│   ├── 04-multi-agent.md
│   └── 05-complete.md
└── es/
    ├── GUIDE.md
    ├── 00-overview.md
    ├── 01-setup.md
    ├── 02-design.md
    ├── 03-quiz-master.md
    ├── 04-multi-agent.md
    └── 05-complete.md
```

### Translation guidelines for workshop markdown

- Translate all prose, headings, tips, callouts, and table content.
- **Do not translate** code blocks, file paths, CLI commands, tool names, or GitHub Copilot feature names (e.g., "Agent Mode", "Plan Mode").
- Keep the original file names (numbered `00-` through `05-` scheme).
- Preserve all markdown formatting, links, and images.
- Translate alt text for any images.
- Keep emoji usage consistent with the English originals.

---

## 4. Docs Site (`docs/`)

### Strategy: Locale-specific HTML pages + locale-aware step viewer

The `docs/` folder is a static GitHub Pages site with two pages: a landing page (`index.html`) and a step viewer (`step.html`). Since this is a static site with no build step, the simplest reliable approach is locale-specific copies with a shared JS locale router.

### Directory layout

```
docs/
├── index.html                  ← English (default)
├── step.html
├── styles.css                  ← Shared (no localization needed)
├── step.css
├── light-theme.css
├── theme-toggle.js
├── locale.js                   ← NEW: locale detection & switcher
├── pt_BR/
│   ├── index.html
│   └── step.html
└── es/
    ├── index.html
    └── step.html
```

### `locale.js` — Locale detection and switcher

Add a shared script that:

1. Reads `?lang=pt_BR` or `?lang=es` query parameter (highest priority).
2. Falls back to `localStorage.getItem('locale')`.
3. Falls back to `navigator.language` mapping (`pt-BR` → `pt_BR`, `es-*` → `es`).
4. Defaults to `en` (serves files from `docs/` root).
5. Provides a visible language-switcher dropdown on all pages.

```js
// docs/locale.js
const SUPPORTED_LOCALES = ['en', 'pt_BR', 'es'];
const LOCALE_LABELS = { en: 'English', pt_BR: 'Português (BR)', es: 'Español' };

function detectLocale() {
  const params = new URLSearchParams(location.search);
  if (SUPPORTED_LOCALES.includes(params.get('lang'))) return params.get('lang');
  const stored = localStorage.getItem('locale');
  if (SUPPORTED_LOCALES.includes(stored)) return stored;
  const nav = navigator.language;
  if (nav.startsWith('pt')) return 'pt_BR';
  if (nav.startsWith('es')) return 'es';
  return 'en';
}
```

### Localized `step.html` changes

The step viewer fetches workshop markdown via raw GitHub URLs. Localized copies must adjust the fetch path:

```js
// English (default)
const markdownUrl = `${repoBase}/workshop/${stepFile}.md`;

// Localized (e.g., pt_BR)
const markdownUrl = `${repoBase}/workshop/pt_BR/${stepFile}.md`;
```

### Translation guidelines for docs HTML

- Translate all visible text content (headings, paragraphs, button labels, stat labels, card titles, descriptions, footer text).
- Set `<html lang="pt-BR">` or `<html lang="es">` appropriately.
- Update `<title>` tags.
- Keep CSS class names, JavaScript logic, analytics tags, and external URLs unchanged.
- Update the "Also available in" links to cross-reference other language editions.

---

## 5. Root `README.md`

### Strategy: Locale badge links + translated README copies

```
README.md                       ← English (default)
README.pt_BR.md                 ← Brazilian Portuguese
README.es.md                    ← Spanish
```

Add locale badges at the top of each README:

```markdown
<!-- In English README.md -->
🌐 [Português (BR)](README.pt_BR.md) | [Español](README.es.md)

<!-- In README.pt_BR.md -->
🌐 [English](README.md) | [Español](README.es.md)
```

---

## 6. App UI Strings

### Strategy: Per-stack idiomatic i18n with a shared string catalog

All four apps display the same UI text. Define a canonical string catalog, then implement it using each stack's idiomatic approach.

### 5.1 Canonical string catalog

These strings appear across all four app variants:

| Key | English (`en`) | Portuguese (`pt_BR`) | Spanish (`es`) |
|-----|---------------|----------------------|----------------|
| `app.title` | Soc Ops | Soc Ops | Soc Ops |
| `app.subtitle` | Social Bingo | Bingo Social | Bingo Social |
| `game.howToPlay` | How to play | Como jogar | Cómo jugar |
| `game.instruction1` | Find people who match the questions | Encontre pessoas que correspondam às perguntas | Encuentra personas que coincidan con las preguntas |
| `game.instruction2` | Tap a square when you find a match | Toque em um quadrado quando encontrar uma correspondência | Toca un cuadro cuando encuentres una coincidencia |
| `game.instruction3` | Get 5 in a row to win! | Consiga 5 em linha para vencer! | ¡Consigue 5 en línea para ganar! |
| `game.startButton` | Start Game | Iniciar Jogo | Iniciar Juego |
| `game.backButton` | ← Back | ← Voltar | ← Volver |
| `game.tapInstruction` | Tap a square when you find someone who matches it. | Toque em um quadrado quando encontrar alguém que se encaixe. | Toca un cuadro cuando encuentres a alguien que coincida. |
| `bingo.title` | BINGO! | BINGO! | ¡BINGO! |
| `bingo.message` | You completed a line! | Você completou uma linha! | ¡Completaste una línea! |
| `bingo.keepPlaying` | Keep Playing | Continuar Jogando | Seguir Jugando |
| `bingo.celebration` | 🎉 BINGO! You got a line! | 🎉 BINGO! Você conseguiu uma linha! | 🎉 ¡BINGO! ¡Conseguiste una línea! |
| `board.freeSpace` | FREE SPACE | ESPAÇO LIVRE | ESPACIO LIBRE |

### 5.2 Python (FastAPI + Jinja2 + HTMX)

Use a simple JSON-based approach that Jinja2 templates can consume:

```
app/
├── locales/
│   ├── en.json
│   ├── pt_BR.json
│   └── es.json
```

**Implementation:**
- Load the locale JSON at startup based on an environment variable (`APP_LOCALE=pt_BR`) or a query parameter.
- Pass the strings dict to Jinja2 template context as `t`.
- In templates: `{{ t.game.startButton }}` instead of hardcoded `Start Game`.

### 5.3 Java (Spring Boot + Thymeleaf)

Use Spring's built-in `MessageSource` with properties files:

```
src/main/resources/
├── messages.properties            ← English (default)
├── messages_pt_BR.properties
└── messages_es.properties
```

**Implementation:**
- Define a `LocaleResolver` bean (e.g., `SessionLocaleResolver` or parameter-based).
- In Thymeleaf templates: `th:text="#{game.startButton}"` instead of hardcoded text.

### 5.4 .NET (Blazor)

Use .NET resource files (`.resx`):

```
SocOps/
├── Resources/
│   ├── Strings.resx               ← English (default)
│   ├── Strings.pt-BR.resx
│   └── Strings.es.resx
```

**Implementation:**
- Inject `IStringLocalizer<Strings>` into Razor components.
- In templates: `@Localizer["game.startButton"]` instead of hardcoded text.
- Configure `RequestLocalizationMiddleware` in `Program.cs`.

### 5.5 TypeScript (React + Vite)

Use a lightweight JSON + React Context approach (no heavy i18n library needed for this scope):

```
src/
├── locales/
│   ├── en.json
│   ├── pt_BR.json
│   └── es.json
├── hooks/
│   └── useLocale.ts               ← NEW: locale context hook
```

**Implementation:**
- Create a `LocaleProvider` React context that loads the appropriate JSON.
- Expose a `useLocale()` hook returning a `t()` function.
- In components: `{t('game.startButton')}` instead of hardcoded text.
- Locale selection via `?lang=` query param or browser detection.

---

## 7. Game Data (Questions / Prompts)

### Strategy: Locale-keyed data files alongside existing data

The bingo card questions should be translated into standalone locale files, parallel to the existing English data.

### Python

```
app/
├── data.py                        ← English (keep as default)
├── data_pt_BR.py
└── data_es.py
```

Or consolidate into a single `data.py` with a dict keyed by locale:

```python
QUESTIONS = {
    "en": ["bikes to work", ...],
    "pt_BR": ["vai de bicicleta ao trabalho", ...],
    "es": ["va en bicicleta al trabajo", ...],
}
```

### Java

```
src/main/java/com/socops/data/
├── IcebreakerPrompts.java         ← Refactor to load from resources
src/main/resources/
├── prompts_en.json
├── prompts_pt_BR.json
└── prompts_es.json
```

### .NET

```
SocOps/Data/
├── Questions.cs                   ← Refactor to load from JSON resources
SocOps/Data/
├── questions.en.json
├── questions.pt_BR.json
└── questions.es.json
```

### TypeScript

```
src/data/
├── questions.ts                   ← Keep as default English
├── questions.pt_BR.ts
└── questions.es.ts
```

Or a consolidated export:

```typescript
export const questionsByLocale: Record<string, string[]> = {
  en: ["bikes to work", ...],
  pt_BR: ["vai de bicicleta ao trabalho", ...],
  es: ["va en bicicleta al trabajo", ...],
};
```

---

## 8. Copilot Configuration Files (`.github/`)

### Strategy: Keep English originals, add locale-aware guidance

Copilot agents, instructions, and prompts should **remain in English** because:
- GitHub Copilot processes English instructions effectively regardless of the user's language.
- Translating agent definitions would create maintenance burden with minimal benefit.
- The agent behavior (code generation) should work the same regardless of locale.

### Recommended addition

Add a note to `.github/instructions/general.instructions.md`:

```markdown
## Localization

This project supports multiple locales: en (English), pt_BR (Brazilian Portuguese), es (Spanish).
When generating user-facing text, respect the active locale configuration.
Never hardcode English strings in UI components — use the locale string system instead.
```

---

## 9. Solution Checkpoints (`.solutions/`)

### Strategy: Do not localize

Solution checkpoints (`.solutions/step-*`) are reference snapshots of the application at various stages. They should remain in English because:

- They serve as code diffs/comparisons, not instructional content.
- Maintaining translated checkpoints across all steps × all locales is impractical.
- Workshop participants compare their code against these — language-neutral code is clearer.

If a fully localized workshop experience is desired in the future, a separate branch per locale is a better fit than duplicating `.solutions/`.

---

## 10. Landing Page (`copilot-dev-days.github.io`)

### Strategy: Locale-specific HTML pages + shared locale picker

The central landing page at `copilot-dev-days.github.io` lives in a **separate repository** (`copilot-dev-days/copilot-dev-days.github.io`). It must also be localized and must use the **exact same locale picker UI** (Section 2) as the individual workshop pages.

### Directory layout

```
copilot-dev-days.github.io/
├── index.html                  ← English (default)
├── styles.css                  ← Shared
├── light-theme.css
├── theme-toggle.js
├── locale-picker.css           ← NEW: locale picker styles (Section 2.4)
├── locale-picker.js            ← NEW: locale picker script (Section 2.5)
├── locale-specification.md
├── pt_BR/
│   └── index.html
└── es/
    └── index.html
```

### Content to translate

The landing page contains:

| Element | English | Translate? |
|---------|---------|------------|
| Page `<title>` | "GitHub Copilot Dev Days" | ✅ Yes |
| Hero badge | "🎓 GitHub Copilot Workshops" | ✅ Yes |
| Hero heading | "GitHub Copilot Dev Days" | ✅ Yes |
| Hero subtitle | "Hands-on workshops to master GitHub Copilot..." | ✅ Yes |
| "Browse Workshops" button | | ✅ Yes |
| Stats labels | "Workshops", "Languages", "IDEs" | ✅ Yes |
| Category section titles | "GitHub Learn Labs", "VS Code — Agent Lab", etc. | ✅ Yes |
| Workshop card titles & descriptions | | ✅ Yes |
| Card metadata labels | "~1hr", "4 parts", "9 steps", etc. | ✅ Yes |
| Prerequisites section | "General Prerequisites", items | ✅ Yes |
| Footer links and credit text | | ✅ Yes |
| GitHub Copilot / tool names | "GitHub Copilot", "VS Code", etc. | ❌ Keep English |
| Workshop URLs | | ❌ Keep (link to locale-specific workshop pages when available) |

### Workshop card links in localized pages

When the landing page is viewed in a locale, workshop cards should link to the localized version of each workshop's docs site:

```html
<!-- English -->
<a href="https://copilot-dev-days.github.io/agent-lab-python/">

<!-- Portuguese -->
<a href="https://copilot-dev-days.github.io/agent-lab-python/pt_BR/">

<!-- Spanish -->
<a href="https://copilot-dev-days.github.io/agent-lab-python/es/">
```

### Locale picker placement

The existing fixed-position theme toggle at `position:fixed;top:1rem;right:1rem` must be extended to include the locale picker alongside it (see Section 2.6).

---

## 11. Implementation Order

The recommended implementation sequence, prioritized by impact:

1. **Locale picker UI** (Section 2) — implement the shared CSS/JS component first; all pages need it.
2. **Landing page** (`copilot-dev-days.github.io/pt_BR/`, `es/`) — the central entry point for all workshops.
3. **Workshop markdown** (`workshop/pt_BR/`, `workshop/es/`) — the primary content attendees read.
4. **Docs site** (`docs/pt_BR/`, `docs/es/`) — the entry point for each individual workshop.
5. **Root README** (`README.pt_BR.md`, `README.es.md`) — first thing visitors see on GitHub.
6. **App UI strings** — per-stack i18n setup + string catalogs.
7. **Game data** — translated bingo question sets.
8. **Copilot config** — locale-awareness note in instructions.

---

## 12. Translation Governance

### English is the source of truth

All content is authored in English first. English files are the **canonical source** — translated files are derived from them. When English content changes, translated content must be updated to match.

### Content classification

Content in these repos falls into two categories with different translation workflows:

| Type | Examples | Translation approach |
|------|----------|---------------------|
| **Technical** | Code blocks, CLI commands, file paths, configuration snippets, step-by-step instructions that reference IDE features, API names, tool names | **Coding agents** should handle these translations. They can reliably propagate structural changes, update code references, and keep technical accuracy consistent across locales. |
| **Non-technical** | Prose descriptions, workshop introductions, motivational text, "What You'll Learn" summaries, cultural references in bingo questions, hero subtitles | **Human contributions welcome.** Native speakers produce more natural, engaging prose than machine translation. Community pull requests improving these sections should be encouraged. |

### Keeping translations in sync

When English content is updated:

1. **Use coding agents** (e.g., GitHub Copilot coding agent, Copilot CLI) to propagate changes to translated files. Agents should:
   - Identify which English files changed (via diff against the previous version).
   - Apply the corresponding structural changes to each locale's files.
   - Translate new or modified technical content (steps, commands, UI references).
   - Flag non-technical prose changes for human review rather than auto-translating them.

2. **Open a PR per locale** with the agent-generated updates. Mark non-technical prose sections with `<!-- NEEDS_HUMAN_REVIEW -->` comments so reviewers know which parts would benefit from a native speaker's touch.

3. **Label PRs** with `localization` and the locale code (e.g., `l10n:pt_BR`, `l10n:es`) for easy filtering.

### Welcoming community contributions

Each repo's `CONTRIBUTING.md` should include a section like:

```markdown
## 🌐 Translations

We welcome contributions to improve translations! If you're a native speaker
of Portuguese (BR) or Spanish and notice awkward phrasing, cultural mismatches,
or opportunities to make the content more natural, please open a PR.

**Guidelines:**
- English is the source of truth — do not add content that doesn't exist in English.
- Focus on improving prose quality, not restructuring content.
- Do not translate code blocks, file paths, CLI commands, or tool/feature names.
- Keep the same markdown structure and formatting as the English original.
```

### Translation freshness tracking

To track whether translated files are up to date, each translated file should include a comment at the top referencing the English source commit it was last synced with:

```markdown
<!-- l10n-sync: english-commit-sha="abc1234" -->
```

This allows coding agents to quickly determine which files are stale by comparing the recorded SHA against the latest English file's history.

---

## 13. File Naming & Locale Codes

| Locale | IETF tag | File suffix | HTML `lang` | Directory name |
|--------|----------|-------------|-------------|----------------|
| English | `en` | (none, default) | `en` | (root) |
| Brazilian Portuguese | `pt-BR` | `pt_BR` | `pt-BR` | `pt_BR` |
| Spanish | `es` | `es` | `es` | `es` |

Use underscores (`pt_BR`) in file/directory names for cross-platform compatibility. Use hyphens (`pt-BR`) only in HTML `lang` attributes and IETF contexts.

---

## 14. Cross-Repo Consistency Rules

Since all five repos share the same locale picker UI and the four workshop repos share identical content structure:

1. **Shared locale picker.** The locale picker HTML/CSS/JS (Section 2) must be identical across all five repos. Consider extracting it into a shared snippet or copying verbatim.
2. **Translate once, apply four times.** Workshop markdown, docs HTML, README, and game questions are identical across repos (modulo stack-specific references). Translate the content once and adapt only stack-specific fragments (e.g., "Python 3.13 & uv" → "Java 21 & Maven").
3. **Keep locale directory structure identical** across all repos.
4. **Use the same canonical string keys** (Section 6.1) across all stacks for the app UI.
5. **Landing page links.** Localized landing page cards must link to the corresponding locale path on each workshop's docs site.
6. **Coordinate releases** — all five repos should ship a new locale at the same time.

---

## 15. Quality Checklist

Before shipping a locale:

- [ ] Locale picker (🌐) renders and functions on all pages
- [ ] Locale picker design matches javaevolved.github.io reference
- [ ] Landing page (`copilot-dev-days.github.io`) translated with correct `lang` attribute
- [ ] Landing page workshop cards link to localized workshop docs
- [ ] All 7 workshop markdown files translated and link-checked
- [ ] Docs site pages translated with correct `lang` attribute
- [ ] Language switcher works on docs site (landing + step viewer)
- [ ] `hreflang` tags present in all HTML `<head>` sections
- [ ] README translated with cross-locale badges
- [ ] App UI strings render correctly (no truncation, no encoding issues)
- [ ] Game questions make cultural sense in the target locale
- [ ] Code blocks, file paths, and CLI commands left untranslated
- [ ] Navigation (Previous/Next) works in localized step viewer
- [ ] No broken links in translated content
- [ ] Locale picker appearance is consistent across light and dark themes
