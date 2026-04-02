# Locale Specification — Agent Lab Workshops

**Target locales:** Brazilian Portuguese (`pt_BR`), Spanish (`es`)  
**Scope:** All four `agent-lab-*` repositories (dotnet, java, python, typescript)

---

## 1. Overview

Each agent-lab repo is a self-contained workshop built around the same "Soc Ops" Social Bingo game. They share an identical content structure but use different tech stacks. This specification defines a **unified localization strategy** that works consistently across all four repos while respecting each stack's idiomatic patterns.

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

---

## 2. Workshop Documentation (`workshop/`)

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

## 3. Docs Site (`docs/`)

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

## 4. Root `README.md`

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

## 5. App UI Strings

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

## 6. Game Data (Questions / Prompts)

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

## 7. Copilot Configuration Files (`.github/`)

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

## 8. Solution Checkpoints (`.solutions/`)

### Strategy: Do not localize

Solution checkpoints (`.solutions/step-*`) are reference snapshots of the application at various stages. They should remain in English because:

- They serve as code diffs/comparisons, not instructional content.
- Maintaining translated checkpoints across all steps × all locales is impractical.
- Workshop participants compare their code against these — language-neutral code is clearer.

If a fully localized workshop experience is desired in the future, a separate branch per locale is a better fit than duplicating `.solutions/`.

---

## 9. Implementation Order

The recommended implementation sequence, prioritized by impact:

1. **Workshop markdown** (`workshop/pt_BR/`, `workshop/es/`) — the primary content attendees read.
2. **Docs site** (`docs/pt_BR/`, `docs/es/`, `locale.js`) — the entry point for the workshop.
3. **Root README** (`README.pt_BR.md`, `README.es.md`) — first thing visitors see on GitHub.
4. **App UI strings** — per-stack i18n setup + string catalogs.
5. **Game data** — translated bingo question sets.
6. **Copilot config** — locale-awareness note in instructions.

---

## 10. File Naming & Locale Codes

| Locale | IETF tag | File suffix | HTML `lang` | Directory name |
|--------|----------|-------------|-------------|----------------|
| English | `en` | (none, default) | `en` | (root) |
| Brazilian Portuguese | `pt-BR` | `pt_BR` | `pt-BR` | `pt_BR` |
| Spanish | `es` | `es` | `es` | `es` |

Use underscores (`pt_BR`) in file/directory names for cross-platform compatibility. Use hyphens (`pt-BR`) only in HTML `lang` attributes and IETF contexts.

---

## 11. Cross-Repo Consistency Rules

Since all four repos share the same workshop structure and game content:

1. **Translate once, apply four times.** Workshop markdown, docs HTML, README, and game questions are identical across repos (modulo stack-specific references). Translate the content once and adapt only stack-specific fragments (e.g., "Python 3.13 & uv" → "Java 21 & Maven").
2. **Keep locale directory structure identical** across all four repos.
3. **Use the same canonical string keys** (Section 5.1) across all stacks for the app UI.
4. **Coordinate releases** — all four repos should ship a new locale at the same time.

---

## 12. Quality Checklist

Before shipping a locale:

- [ ] All 7 workshop markdown files translated and link-checked
- [ ] Docs site pages translated with correct `lang` attribute
- [ ] Language switcher works on docs site
- [ ] README translated with cross-locale badges
- [ ] App UI strings render correctly (no truncation, no encoding issues)
- [ ] Game questions make cultural sense in the target locale
- [ ] Code blocks, file paths, and CLI commands left untranslated
- [ ] Navigation (Previous/Next) works in localized step viewer
- [ ] No broken links in translated content
