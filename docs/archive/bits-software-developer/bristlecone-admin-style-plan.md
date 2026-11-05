# Bristlecone Admin Style — v1 Plan

**Status:** Plan only (no implementation, commits, or PRs until Mahler approval)  
**Product owner:** Corey J. Mahler / BITS  
**Repo:** https://github.com/Zyniker13/bits-wp-admin-styles  
**Date:** 2026-09-26 (ET)

---

## 1. Research findings

### 1.1 `Zyniker13/bits-wp-admin-styles` (current state)

| Fact | Detail |
|------|--------|
| Visibility | Public |
| Default branch | `main` |
| Contents | **Empty scaffold** — only `README.md` (“bits-wp-admin-styles / Style control for the WordPress admin (/wp-admin/).”) |
| Size / language | 0 / none |
| License | None yet (Markdown uses GPL-2.0) |
| Issues / PRs | None |
| Description | “Style control for the WordPress admin (/wp-admin/).” |

**Implication:** Greenfield plugin. Bootstrap file, `readme.txt`, LICENSE, and package layout all need to be created. Keep the GitHub repo name `bits-wp-admin-styles`; WordPress plugin folder / slug should follow Bristlecone display naming (see §2), matching Markdown’s split between repo `bits-markdown` and plugin `bristlecone-markdown`.

### 1.2 `Zyniker13/bits-markdown` (BITS style to align with)

| Convention | Markdown value | Proposed for Admin Style |
|------------|----------------|---------------------------|
| Plugin Name (header) | `Bristlecone Markdown` | `Bristlecone Admin Style` |
| Bootstrap file | `bristlecone-markdown.php` | `bristlecone-admin-style.php` |
| Text domain | `bristlecone-markdown` | `bristlecone-admin-style` |
| PHP namespace | `Bristlecone\Markdown` | `Bristlecone\AdminStyle` |
| Requires PHP | `8.4` (composer `>=8.4.1`) | `8.4` (align; no Composer needed for v1) |
| Requires WP | `6.4` | `6.4` |
| Author | Bristlecone IT Services | Same |
| License | GPL-2.0-or-later | Same |
| `readme.txt` | WordPress.org-style header + sections | Same shape, shorter |
| Settings today | `add_options_page` → **Settings → Bristlecone Markdown** (slug `bristlecone-markdown`) | **Not** under Settings — top-level **Bristlecone** (see §5) |
| Option key | `bristlecone_markdown_settings` | `bristlecone_admin_style_settings` |
| Strict types | `declare(strict_types=1);` | Same |
| Package `@package` | `BristleconeMarkdown` | `BristleconeAdminStyle` |

Do **not** copy Markdown features, Composer stack, blocks, or Tools pages. This plugin is intentionally thin (settings + admin CSS).

### 1.3 WordPress admin CSS / list-table hooks

| Mechanism | Use in v1 |
|-----------|-----------|
| `admin_enqueue_scripts` | Enqueue stylesheet; `$hook_suffix` for `edit.php` is a first filter |
| `get_current_screen()` | Restrict further: `base === 'edit'`, `post_type === 'post'` → screen id `edit-post` |
| Body classes | `wp-admin`, `edit-php`, `post-type-post` (useful as CSS selectors) |
| List table | `.wp-list-table` (plus `.widefat`, `.fixed`, `.striped`, `.posts`) |
| Row title / cells | `.wp-list-table .row-title`, `th`, `td`, `.row-actions` for readable monospace |

**Recommended enqueue gate (conceptual):**

1. Hook `admin_enqueue_scripts`.
2. Bail unless `$hook === 'edit.php'` **or** screen `id === 'edit-post'` / (`base === 'edit'` && `post_type === 'post'`).
3. Bail unless settings say fonts are enabled and a non-default family is selected.
4. `wp_enqueue_style( 'bristlecone-admin-style-list', … )` with version = plugin version.
5. Prefer a small static CSS file plus `wp_add_inline_style` for the chosen `font-family` (sanitized from an allowlist), so presets stay maintainable without shipping N CSS files.

**Out of scope for targeting in v1:** `post.php` / `post-new.php` (single editor), Pages list (`edit-page`), CPT lists, media library, dashboard widgets.

---

## 2. Identity & versioning

| Field | Value |
|-------|--------|
| **Display name** | Bristlecone Admin Styless |
| **Plugin slug / folder** | `bristlecone-admin-style` |
| **GitHub repo** | `bits-wp-admin-styles` (unchanged) |
| **Text domain** | `bristlecone-admin-style` |
| **Bootstrap** | `bristlecone-admin-style.php` |
| **Constants prefix** | `BRISTLECONE_ADMIN_STYLE_*` (e.g. `VERSION`, `FILE`, `DIR`, `URL`) |
| **Initial version** | **1.0.0** |
| **Requires at least** | 6.4 |
| **Requires PHP** | 8.4 |
| **Tested up to** | Match current BITS practice (Markdown: 7.1) at ship time |
| **License** | GPL-2.0-or-later |
| **Plugin URI** | TBD — e.g. `https://bristleconeit.com/bristlecone-admin-style` (open question) |
| **Author** | Bristlecone IT Services / https://bristleconeit.com |

**Versioning:** SemVer. 1.0.0 = first usable release (post-list monospace). Feature additions (more screens, more typography) bump minor; fixes patch.

---

## 3. Suggested package layout (for implementer; not built yet)

```
bristlecone-admin-style.php          # bootstrap + headers
readme.txt                           # WP.org-style
README.md                            # GitHub overview
LICENSE                              # GPL-2.0
uninstall.php                        # delete option on uninstall
index.php                            # silence
includes/
  Plugin.php                         # init wiring
  Settings.php                       # menu, Settings API, render
  Assets.php                         # admin_enqueue_scripts + screen checks
assets/
  css/
    admin-list-fonts.css             # base selectors; family via inline
languages/                           # empty + text domain ready
```

No Composer for v1 unless Mahler wants PSR-4 autoload parity with Markdown for consistency alone (open question — lean **no** Composer for this thin plugin; simple `require_once` is enough).

---

## 4. Settings UX — Bristlecone → Admin Styles

### 4.1 Menu labels

- Top-level: **Bristlecone**
- Submenu (this plugin): **Admin Styles**
- Page title: **Admin Styles** (or “Bristlecone Admin Styles”)
- Menu capability: `manage_options` (site admins only; settings are site-wide)

### 4.2 Settings fields (v1)

| Control | Type | Default | Notes |
|---------|------|---------|--------|
| Enable custom admin fonts | checkbox | off | When off, enqueue nothing |
| Font family | select | System default (no override) | Allowlisted presets only |
| Apply to | read-only text / note | Post list (Posts → All Posts) | Documents v1 scope; expandable later |

**Font family presets (allowlist):**

1. **System default** — no CSS override (control remains visible for future).
2. **System monospace** — primary need:  
   `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace`
3. Optional extra presets (cheap to ship; still allowlisted):  
   - **IBM Plex Mono** stack (system / generic fallback only — **no** webfont CDN in v1)  
   - **Cascadia / Consolas-forward** stack  
   Keep the list short (3–4 options). No free-text CSS, no Google Fonts, no uploads.

**Sanitize:** Store an enum key (`default` | `monospace` | …), never raw CSS from the user. Map key → `font-family` string in PHP.

**Settings API:**

- Option group: `bristlecone_admin_style`
- Option name: `bristlecone_admin_style_settings`
- Shape (example): `{ enabled: bool, font_family: string, scopes: string[] }` with `scopes` fixed to `['edit-post']` in v1 (stored for forward compatibility).

**UX copy (short):** Explain that monospace helps dense title/slug/date columns on All Posts; other admin screens keep core fonts until a later release.

---

## 5. Menu registration & future Markdown coexistence

### 5.1 Parent slug contract (shared Bristlecone menu)

| Item | Value |
|------|--------|
| **Parent menu slug** | `bristlecone` |
| **Capability** | `manage_options` |
| **Icon** | Dashicon TBD (e.g. `dashicons-admin-generic` or a BITS brand icon later) |
| **Position** | TBD — suggest ~58–65 (after Comments, before Appearance) |
| **This plugin’s submenu slug** | `bristlecone-admin-style` |
| **Markdown’s future submenu slug** | Keep existing page slug `bristlecone-markdown` when moved |

### 5.2 How this plugin registers (v1)

1. On `admin_menu` (priority ~9 or 10):
   - If parent is not yet registered (`empty( $GLOBALS['admin_page_hooks']['bristlecone'] )`), call `add_menu_page( …, 'bristlecone', … )`.
   - Call `add_submenu_page( 'bristlecone', …, 'bristlecone-admin-style', … )`.
2. Avoid the classic WP duplicate “Bristlecone / Bristlecone” first submenu:
   - Either use the parent slug as the **first** submenu slug and rename that first item to “Admin Styles”, **or**
   - Register parent + distinct submenu, then `remove_submenu_page( 'bristlecone', 'bristlecone' )` if WP auto-added a duplicate.
3. Document the chosen pattern in code comments so Markdown can copy it.

### 5.3 How Markdown attaches later (without breaking this plugin)

**Do not** change Markdown in this v1 ticket. Document the migration path:

1. Markdown stops using `add_options_page( …, 'bristlecone-markdown', … )`.
2. Markdown uses the same parent guard: create `bristlecone` only if missing, then `add_submenu_page( 'bristlecone', 'Markdown', 'Markdown', 'manage_options', 'bristlecone-markdown', … )`.
3. **Either plugin alone** must be able to create the parent (Admin Style deactivated, Markdown still shows Bristlecone → Markdown; and vice versa).
4. Keep Markdown’s **page slug** `bristlecone-markdown` so existing bookmarks / docs (`options-general.php?page=bristlecone-markdown`) need a one-time note or redirect helper when Markdown migrates (open question for Markdown ticket, not this one).
5. Optional later: tiny shared helper (mu-plugin or `bristlecone-admin` stub) — **out of scope**; two identical parent-guard snippets are enough for two plugins.

**Capability alignment:** Both use `manage_options` so the menu appears consistently for the same users.

---

## 6. CSS enqueue design

| Step | Detail |
|------|--------|
| Hook | `admin_enqueue_scripts` |
| Gate | Settings enabled + font ≠ default + screen is post list (`edit-post` / `edit.php` + `post`) |
| Handle | `bristlecone-admin-style-list` |
| File | `assets/css/admin-list-fonts.css` — selectors only, e.g. `body.wp-admin.post-type-post .wp-list-table, … { /* family set inline */ }` |
| Inline | `wp_add_inline_style( …, '…{ font-family: ' . $stack . '; }' )` |
| Dependencies | None (or empty array); load after core admin CSS by enqueueing normally |
| Front end | Never enqueue |

**Selector intent:** Cover list table cells, column headers, row titles, and row actions so the whole posts table reads as monospace—not the admin menu, not `#wpadminbar`, not screen options chrome (unless unavoidable via inheritance; prefer scoped `.wp-list-table` rules).

---

## 7. Non-goals for v1

- Full admin themes / color schemes / density / spacing
- Custom font upload, Google Fonts, or remote webfonts
- Per-user (profile) font preferences
- Pages list, CPT lists, Comments, Media, Users, Plugins, or editor (`post.php`) fonts
- Changing Markdown’s menu location (document only)
- Composer / PHPUnit suite (optional later; Markdown has both)
- WordPress.org submission process (repo + zip install is enough initially)
- Multisite network-admin UI (site `manage_options` only unless requested)
- JavaScript / React settings UI

---

## 8. Acceptance criteria (v1)

1. Plugin activates on WP ≥ 6.4 / PHP ≥ 8.4 without fatals; headers and `readme.txt` match §2.
2. Top-level **Bristlecone** menu appears for users with `manage_options`; submenu **Admin Styles** opens a Settings API form that saves correctly.
3. With **Enable** on and **System monospace** selected, **Posts → All Posts** list table text uses the monospace stack.
4. With enable off (or default font), All Posts matches core fonts (no leftover CSS).
5. CSS does **not** load on Dashboard, Pages list, single post editor, or front end.
6. Uninstall deletes `bristlecone_admin_style_settings`.
7. Parent slug is `bristlecone`; README/DESIGN note documents how Markdown can `add_submenu_page` later without conflicting.
8. No commits/PRs until this plan is approved (this document only).

---

## 9. Cursor Manager ticket text (AFTER Mahler approval)

> **Title:** Implement Bristlecone Admin Style v1.0.0 (post list monospace fonts)
>
> **Repo:** https://github.com/Zyniker13/bits-wp-admin-styles  
> **Plan:** `[box scratch]/bristlecone-admin-style-plan.md` (approved)
>
> **Do:**
> - Scaffold plugin `bristlecone-admin-style` (bootstrap, includes, assets, readme.txt, LICENSE GPL-2.0-or-later, uninstall.php) aligned with BITS Markdown conventions (PHP 8.4, WP 6.4, text domain `bristlecone-admin-style`, strict types). Version **1.0.0**.
> - Register top-level admin menu slug **`bristlecone`** (`manage_options`) and submenu **Admin Styles** (`bristlecone-admin-style`). Guard parent creation so a future Bristlecone Markdown submenu can attach without breaking this plugin (document in README).
> - Settings: enable toggle + allowlisted font family presets (system default + system monospace stack + 1–2 optional stacks). No free-text CSS / webfonts.
> - Enqueue admin CSS only on Posts list (`edit-post` / `edit.php` + post type `post`) via `admin_enqueue_scripts` + screen checks; apply `font-family` to `.wp-list-table` (and related list-table text) when enabled.
> - Add GitHub README describing install, settings path (Bristlecone → Admin Styles), and Markdown coexistence notes.
>
> **Don’t:** Change `bits-markdown`; no WPORG submit; no Composer unless needed; no commits outside this repo; no scope creep beyond post-list fonts.
>
> **Done when:** Acceptance criteria in the plan §8 pass on a local WP admin smoke test.

---

## 10. Open questions for Mahler

1. **Plugin URI / marketing page** — Create `bristleconeit.com/bristlecone-admin-style` now, or leave Plugin URI as the GitHub repo until a product page exists?
2. **Composer / PSR-4** — Match Markdown’s structure for consistency, or keep a no-Composer thin plugin?
3. **Default on install** — Enable + monospace immediately (solves the “immediate need” with zero clicks), or default **off** for safer first activation?
4. **Scope naming** — Confirm v1 is **posts only** (`post`), not all `edit.php` screens (which would include Pages and CPTs). Prefer posts-only for the stated need.
5. **Extra font presets** — Ship only “default + system monospace”, or include 1–2 named stacks now?
6. **Menu icon / position** — Any preference for Dashicon and `$position`?
7. **Markdown menu migration timing** — Separate follow-up ticket after Admin Style ships, or same release train later?
8. **Display name spelling** — Confirm singular **Admin Style** (product brief) vs plural “Admin Styles” (submenu label). Plan uses: Plugin Name singular; submenu **Admin Styles**.

---

## 11. Stop line

**STOP.** No code, commits, or PRs until Mahler approves this plan and a Cursor Manager ticket is cut from §9.

---

## 12. Mahler decisions (2026-09-26 ET) — APPROVED WITH NOTES

| # | Decision |
|---|----------|
| 1 | **Plugin URI:** `https://bristleconeit.com/bristlecone-admin-styles/` |
| 2 | **Composer:** Prefer thin plugin (no Composer) unless PSR-4 becomes necessary — implementer choice: simplest maintainable layout wins |
| 3 | **Defaults:** Enable **off** on install |
| 4 | **Scope roadmap:** Eventual goal = change **all** `/wp-admin/` fonts; **v1** still ships post-list application first, but settings/data model must be **staged** (e.g. scopes allowlist expandable: `edit-post` now → more screens / “all admin” later) |
| 5 | **Fonts:** Monospace was an example, not the only option. Use **Google Fonts or similar**, optimally **cacheable** (prefer **self-hosted / bundled** font files in the plugin for performance + privacy; avoid mandatory third-party CDN on every admin load). Settings: curated allowlist of families (including monospace and common admin-friendly faces), not free-text CSS |
| 6 | **Menu icon:** Bold letter **“B”** as the parent Bristlecone menu icon (custom SVG/`menu_icon`, not a Dashicon) |
| 7 | **Markdown migration:** **Later ticket**. Target labels when moved: plugin/settings page name **Bristlecone Markdown**; submenu item **Markdown** |
| 8 | **Naming:** Plugin display name **Bristlecone Admin Styles** (plural); submenu **Admin Styles** |

**Supersedes earlier plan text:** §2 Plugin Name singular → plural; §7 “no Google Fonts / remote webfonts” → replaced by self-hosted/cacheable webfonts from an allowlist; Plugin URI set; defaults off.

**v1 acceptance updates:**
- Plugin Name header: `Bristlecone Admin Styles`
- Font picker includes multiple curated families (system stacks + bundled/cacheable webfonts); monospace remains one preset
- Still gate CSS to post list in v1, with `scopes` (or equivalent) ready to expand
- Parent menu uses bold “B” icon

---

## 13. Naming amendment (2026-09-26 ET) — plural everywhere

Mahler: pluralize slug and related identifiers for consistency.

| Field | Value |
|-------|--------|
| Plugin Name | Bristlecone Admin Styles |
| Slug / folder | `bristlecone-admin-styles` |
| Bootstrap | `bristlecone-admin-styles.php` |
| Text domain | `bristlecone-admin-styles` |
| Page / submenu slug | `bristlecone-admin-styles` |
| Option group / name | `bristlecone_admin_styles` / `bristlecone_admin_styles_settings` |
| Constants | `BRISTLECONE_ADMIN_STYLES_*` |
| Parent menu slug | `bristlecone` (unchanged) |
| Plugin URI | https://bristleconeit.com/bristlecone-admin-styles/ |

Supersedes earlier singular `bristlecone-admin-style` proposals in §§2–9.
