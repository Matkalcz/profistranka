# profistranka - Pokyny pro agenty

**Repozitář:** https://github.com/matkalcz/profistranka
**Poslední aktualizace:** 2026-03-21

---

## OBSAH

1. [Nastavení MCP](#nastavení-mcp)
2. [Základní pravidla](#základní-pravidla)
3. [Git a repozitář](#git-a-repozitář)
4. [Workflow pro nový web](#workflow-pro-nový-web)
5. [Správa stránek a sekcí](#správa-stránek-a-sekcí)
6. [Vzhled webu (styly, menu, patička)](#vzhled-webu)
7. [Příspěvky a kategorie](#příspěvky-a-kategorie)
8. [Smart tagy](#smart-tagy)
9. [Soubory](#soubory)
10. [Import a export](#import-a-export)
11. [Troubleshooting](#troubleshooting)
12. [Dostupné nástroje](#dostupné-nástroje)

---

## NASTAVENÍ MCP

### Servery

| Server | URL | Použití |
|---|---|---|
| **profistranka** (produkce) | `https://mcp.profistranka.cz/sse` | Ostré weby, produkční změny |
| **profistranka-test** (test) | `https://mcp.test.profistranka.cz/sse` | Testování, ověřování |

- **API klíč** ve formátu `msk_...` — získáš od administrátora
- Před každou prací ověř, zda pracuješ proti produkci nebo testu

### Claude Code

Přidej do `.mcp.json` v kořeni projektu nebo globálně do `~/.claude/.mcp.json`:

```json
{
  "mcpServers": {
    "profistranka": {
      "type": "sse",
      "url": "https://mcp.profistranka.cz/sse",
      "headers": {
        "X-Api-Key": "msk_TVUJ_PRODUKČNÍ_KLÍČ"
      }
    },
    "profistranka-test": {
      "type": "sse",
      "url": "https://mcp.test.profistranka.cz/sse",
      "headers": {
        "X-Api-Key": "msk_TVUJ_TESTOVACÍ_KLÍČ"
      }
    }
  }
}
```

Po uložení restartuj Claude Code.

### Claude Desktop

Jdi do Nastavení → Developer → Edit Config, přidej do `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "profistranka": {
      "type": "sse",
      "url": "https://mcp.profistranka.cz/sse",
      "headers": {
        "X-Api-Key": "msk_TVUJ_PRODUKČNÍ_KLÍČ"
      }
    }
  }
}
```

### OpenAI Codex

Přidej do `~/.codex/config.toml` nebo `.codex/config.toml`:

```toml
[mcp_servers.profistranka]
url = "https://mcp.profistranka.cz/sse"
http_headers = { "X-Api-Key" = "msk_TVUJ_PRODUKČNÍ_KLÍČ" }
```

### Ověření připojení

Zavolej: *„Zavolej list_websites a ukaž mi dostupné weby."*

Pokud nefunguje: zkontroluj formát API klíče (`msk_...`), dostupnost serveru a restart klienta.

---

## ZÁKLADNÍ PRAVIDLA

1. **Nikdy nezačínej destruktivní operaci** bez načtení kontextu — pokud nemáš jistá ID a stav webu.
2. **Pokud neznáš websiteId**, nejdřív zavolej `list_websites`.
3. **Před úpravami webu** vždy preferuj `get_context`, aby ses opřel o skutečný stav webu.
4. **Po obsahových nebo designových změnách** preferuj `preview` dřív než `publish`.
5. **Destruktivní operace** (`delete_page`, `delete_section`, `delete_post`, import v režimu `replace`) dělej jen na výslovný pokyn uživatele.
6. **U importu** vždy nejdřív proveď `validate`, teprve pak případný `import`.
7. **Neprováděj `publish` automaticky** jen proto, že jsi něco upravil — publish udělej jen když to uživatel chce.
8. **Nehádej si ID** (`pageId`, `sectionId`, `postId`, `blockId`, `categoryId`). Získej je z `get_context`.
9. **CSS patří do `set_styles`** — nikdy nedávej CSS do HTML sekcí.
10. **Zachovávej existující styl webu**, strukturu navigace a vizuální konzistenci, pokud uživatel neřekne jinak.
11. **Ověř doménu**: pokud uživatel specifikuje doménu, nejdřív zkontroluj přes `get_context` → `metadata.primaryDomain`.
12. **HTML obsah sekcí musí být čistý** — bez `<!DOCTYPE>`, `<html>`, `<head>`, `<body>` tagů a bez inline CSS.

---

## GIT A REPOZITÁŘ

**Repozitář:** https://github.com/matkalcz/profistranka

**Lokální cesta (tento stroj):** `C:/Users/marti/vibecoding/profistranka/`

Veškeré lokální soubory, projekty a klony repozitářů se ukládají do `C:/Users/marti/vibecoding/`. Klon profistranka repo je na:
```
C:/Users/marti/vibecoding/profistranka/
```

**Struktura projektu:**
```
matkalcz/profistranka/
├── AGENT_GUIDE.md          # Tento návod
└── projects/
    ├── bondsky/            # Projekt Bondsky
    │   ├── index.html
    │   ├── styles.css
    │   └── images/
    ├── muj-projekt/        # Tvůj projekt
    └── README.md
```

**Všechny projektové soubory patří do složky `projects/`.**

**Workflow po každé změně:**
```bash
cd C:/Users/marti/vibecoding/profistranka
git add projects/nazev-projektu/
git commit -m "ProjektXY: popis změny"
git push origin main
```

---

## WORKFLOW PRO NOVÝ WEB

### Kompletní postup od začátku:

```
1. list_websites           → zjisti websiteId
2. get_context             → načti stav webu
3. set_styles              → nastav barvy, menu, patičku, globální CSS
4. create_page             → vytvoř stránky
5. add_section             → přidej HTML obsah (čisté HTML, bez CSS)
6. preview                 → vygeneruj URL pro kontrolu
7. publish                 → zveřejni (jen na výslovný pokyn)
```

**Weby se publikují na adrese `nazev.profistranka.cz`** — konkrétní subdoménu určí administrátor při zakládání webu.

### Workflow pro novou kategorii s blogem:
```
create_category → create_page (articlepage) → create_page (listpage)
→ assign_page_to_category (obě stránky) → add_section se smart tagem <posts>
→ create_post → assign_post_categories → publish_post → preview → publish
```

### Workflow pro úpravu existující stránky:
```
get_context → najdi sectionId → get_section (přesné HTML) → edit_section → preview
```

### Workflow pro změnu stylů:
```
get_context → set_styles → preview
```

---

## SPRÁVA STRÁNEK A SEKCÍ

### Typy stránek

| Typ | Kdy použít |
|---|---|
| **Unikátní** | Kontakt, O nás, Domovská stránka |
| **Articlepage** (univerzální) | Šablona pro zobrazení jednoho článku v kategorii |
| **Listpage** (univerzální) | Šablona pro výpis článků v kategorii |

**Pravidlo:** Pokud stránka může mít kategorii → použij kategorii + univerzální stránky.

### Vytvoření stránky (`create_page`)

```bash
mcporter call mstranka.create_page \
  websiteId="ID_WEBU" \
  name="o-nas" \
  slug="o-nas" \
  title="O nás" \
  showInMenu=true \
  description="Stránka O nás"
```

Parametry: `websiteId`\*, `name`\*, `slug`\*, `title`\*, `sortOrder?`, `showInMenu?`, `description?`, `language?`

**NEMÁ** parametry `template` ani `isPublished`.

### Přidání sekce (`add_section`)

```bash
mcporter call mstranka.add_section \
  websiteId="ID_WEBU" \
  pageId="ID_STRANKY" \
  name="hero" \
  htmlContent="<div class=\"hero\"><h1>Nadpis</h1><p>Text</p></div>" \
  title="Hero sekce" \
  showOnPage=true
```

### Úprava sekce (`edit_section`)

```bash
# VŽDY nejdřív získej sectionId z get_context!
mcporter call mstranka.edit_section \
  websiteId="ID_WEBU" \
  sectionId="ID_SEKCE" \
  name="hero" \
  htmlContent="<div class=\"hero\"><h1>Nový nadpis</h1></div>" \
  title="Hero sekce" \
  showOnPage=true
```

> `get_context` vrací sekce **bez HTML obsahu**. Pro přesné HTML vždy nejdřív zavolej `get_section` se správným `sectionId`, teprve pak edituj.

### Pravidla pro HTML obsah sekcí

- ✅ Čisté HTML bez `<!DOCTYPE>`, `<html>`, `<head>`, `<body>`
- ✅ Uvozovky uvnitř HTML escapuj zpětným lomítkem (`\"`)
- ✅ HTML neescapuj — posílej čisté `<tagy>`, ne `&lt;tagy&gt;`
- ✅ Pro dlouhý obsah používej `--raw-strings` flag

```bash
# Načtení HTML ze souboru
HTML_CONTENT=$(cat muj_obsah.html)
mcporter call mstranka.add_section --raw-strings \
  websiteId="ID_WEBU" \
  pageId="ID_STRANKY" \
  name="sekce" \
  htmlContent="$HTML_CONTENT"
```

---

## VZHLED WEBU

### Barvy a globální styly (`set_styles`)

```bash
mcporter call mstranka.set_styles \
  websiteId="ID_WEBU" \
  primaryColor="#2563eb" \
  primaryHoverColor="#1d4ed8" \
  secondaryColor="#64748b" \
  secondaryHoverColor="#475569"
```

Dostupné parametry `set_styles`: `primaryColor`, `primaryHoverColor`, `secondaryColor`, `secondaryHoverColor`, `styleContent`, `headerContent`, `footerContent`, `htmlHead`, `googleAnalyticsCode`

### Barvové placeholdery v CSS

V `styleContent` vždy používej placeholdery místo konkrétních hex hodnot:

| Placeholder | Popis |
|---|---|
| `[WebPrimaryColor]` | Hlavní barva |
| `[WebPrimaryHoverColor]` | Hover stav hlavní barvy |
| `[WebSecondaryColor]` | Sekundární barva |
| `[WebSecondaryHoverColor]` | Sekundární hover |

```bash
mcporter call mstranka.set_styles --raw-strings \
  websiteId="ID_WEBU" \
  styleContent="$(cat styles.css)"
```

### Menu a patička

Menu (`headerContent`) a patička (`footerContent`) jsou **globální** — nastavují se přes `set_styles`, **ne** jako běžné sekce. Nemají `sectionId`.

```bash
# Nastavení menu
mcporter call mstranka.set_styles --raw-strings \
  websiteId="ID_WEBU" \
  headerContent="$(cat menu.html)"

# Nastavení patičky
mcporter call mstranka.set_styles --raw-strings \
  websiteId="ID_WEBU" \
  footerContent="$(cat footer.html)"
```

| | Běžné sekce | Menu / Patička |
|---|---|---|
| Nástroj | `add_section` / `edit_section` | `set_styles` |
| Scope | Konkrétní stránka | Celý web |
| Má sectionId | ✅ | ❌ |

---

## PŘÍSPĚVKY A KATEGORIE

### Vytvoření kategorie

```bash
mcporter call mstranka.create_category \
  websiteId="ID_WEBU" \
  name="Novinky" \
  slug="novinky"
```

### Vytvoření příspěvku (draft)

```bash
mcporter call mstranka.create_post \
  websiteId="ID_WEBU" \
  title="Název článku" \
  slug="nazev-clanku" \
  perex="Krátký úvod článku..." \
  htmlContent="$HTML_CONTENT"
```

Nový příspěvek je vždy **draft** — pro zveřejnění je třeba `publish_post` a pak `publish` webu.

### Přiřazení příspěvku ke kategorii

```bash
mcporter call mstranka.assign_post_categories \
  websiteId="ID_WEBU" \
  postId="ID_PRISPEVKU" \
  categoryIds='["ID_KATEGORIE"]'
```

### Publikace příspěvku

```bash
mcporter call mstranka.publish_post \
  websiteId="ID_WEBU" \
  postId="ID_PRISPEVKU"
```

### Každá kategorie musí mít:
- ✅ **Articlepage** — univerzální stránka pro zobrazení jednoho článku
- ✅ **Listpage** — univerzální stránka pro výpis článků (obsahuje smart tag `<posts>`)
- ✅ Obě stránky přiřazené ke kategorii přes `assign_page_to_category`

---

## SMART TAGY

Smart tagy se vkládají do HTML obsahu sekcí.

### `<posts>` — výpis příspěvků

```html
<posts category="novinky" take="10" order="date">
  <div class="card">
    <h3>[Title]</h3>
    <p>[Perex]</p>
    <span>[PublishedDate]</span>
    <a href="[Url]">Číst dále →</a>
  </div>
</posts>
```

**Atributy:** `category`\* (slug), `take`, `skip`, `order` (`date`/`date asc`/`title`), `paging`, `pageSize`

**Proměnné:** `[Title]`, `[Perex]`, `[Content]`, `[PublishedDate]`, `[Url]`, `[TitleImageUrl]`, `[Slug]`, `[CategoryName]`, `[CategorySlug]`, `[ExternalUrl]`, `[HeaderVideoUrl]`, `[TextField1]`–`[TextField4]`

### Stránkování `<posts>`

Stránkování se kreslí **mimo** tag `<posts>` jako samostatné placeholdery:

| Placeholder | Popis |
|---|---|
| `[NextPageUrl]` | URL další stránky (prázdný na poslední) |
| `[PrevPageUrl]` | URL předchozí stránky (prázdný na první) |
| `[PageNumber]` | Číslo aktuální stránky |
| `[PageCount]` | Celkový počet stránek |

```html
<posts paging="true" pageSize="9" take="9" category="novinky">
  <div class="card">[Title]</div>
</posts>

<nav>
  <a href="[PrevPageUrl]">← Předchozí</a>
  <span>Stránka [PageNumber] z [PageCount]</span>
  <a href="[NextPageUrl]">Další →</a>
</nav>
```

### `<menu>` — navigace

```html
<menu><a href="[PageUrl]" class="[ActiveClass]">[PageTitle]</a></menu>
```

### `<block>` — sdílený blok

```html
<block name="nazev-bloku" />
```

Správa bloků přes `manage_block`.

### `<field>` — systémová hodnota

```html
<field name="ContactEmail" />
```

Podporovaná pole: `ContactEmail`, `Title`, `Subtitle`, `Description`

---

## SOUBORY

```bash
# Nahrání souboru z URL
mcporter call mstranka.upload_file \
  websiteId="ID_WEBU" \
  fileName="obrazek.jpg" \
  folder="images" \
  sourceUrl="https://example.com/obrazek.jpg"

# Nahrání souboru jako base64
mcporter call mstranka.upload_file \
  websiteId="ID_WEBU" \
  fileName="logo.png" \
  folder="images" \
  base64Content="$(base64 logo.png)"

# Výpis složek
mcporter call mstranka.list_folders \
  websiteId="ID_WEBU"
```

---

## IMPORT A EXPORT

```bash
# Export dat webu
mcporter call mstranka.export \
  websiteId="ID_WEBU" \
  parts='["pages","posts","categories","appearance"]'

# Validace před importem (VŽDY nejdřív!)
mcporter call mstranka.validate \
  websiteId="ID_WEBU" \
  data="$(cat export.json)"

# Import (merge = sloučí, replace = přepíše)
mcporter call mstranka.import \
  websiteId="ID_WEBU" \
  mode="merge" \
  data="$(cat export.json)"
```

**Části pro export:** `pages`, `blocks`, `posts`, `categories`, `appearance`, `redirects`, `settings`, `files`

> Import v režimu `replace` je vysoce rizikový — dělej jen na výslovný pokyn.

---

## TROUBLESHOOTING

### "Unauthorized" nebo "Authentication failed"
Zkontroluj API klíč v konfiguraci MCP — musí být ve formátu `msk_...`.

### "Website not found"
```bash
mcporter call mstranka.list_websites --output json
```

### "An error occurred invoking 'edit_section'"
Příčina: špatné `sectionId` nebo chybějící parametry. Vždy získej `sectionId` z `get_context`.

### "Invalid HTML content" nebo JSON parse error
HTML nesmí být escapované. Zkontroluj obsah:
```bash
echo "$HTML_CONTENT" | head -5
# Správně: <h1>Nadpis</h1>
# Špatně:  &lt;h1&gt;Nadpis&lt;/h1&gt;
```

### `create_page` nefunguje nebo "Unknown tool"
`create_page` funguje i když není vždy vidět v seznamu nástrojů — použij ho přímo.

### Každý web má jiná ID!
❌ Nepoužívej ID z příkladů v tomto návodu — jsou ilustrační.
✅ Vždy získej aktuální ID přes `get_context`.

---

## DOSTUPNÉ NÁSTROJE (23 MCP nástrojů)

### Workflow
| Nástroj | Popis |
|---|---|
| `list_websites` | Seznam dostupných webů |
| `get_context` | Kompletní stav webu (stránky, sekce, příspěvky, kategorie, styly) |
| `preview` | Vygeneruje dočasnou URL pro kontrolu — **vždy před publish** |
| `publish` | Zveřejní změny na živém webu (`nazev.profistranka.cz`) |
| `export` | Exportuje data webu |
| `import` | Importuje data do webu |
| `validate` | Validuje data před importem |

### Stránky
| Nástroj | Popis |
|---|---|
| `create_page` | Vytvoří novou stránku |
| `edit_page` | Upraví existující stránku |
| `delete_page` | Smaže stránku **včetně všech sekcí** (nevratné!) |

### Sekce
| Nástroj | Popis |
|---|---|
| `add_section` | Přidá sekci na stránku |
| `get_section` | Stáhne kompletní HTML obsah sekce |
| `edit_section` | Upraví sekci |
| `delete_section` | Smaže sekci (nevratné!) |

### Příspěvky
| Nástroj | Popis |
|---|---|
| `create_post` | Vytvoří draft příspěvku |
| `edit_post` | Upraví příspěvek |
| `delete_post` | Smaže příspěvek |
| `publish_post` | Publikuje příspěvek |

### Kategorie
| Nástroj | Popis |
|---|---|
| `create_category` | Vytvoří kategorii |
| `edit_category` | Upraví kategorii |
| `delete_category` | Smaže kategorii |
| `assign_post_categories` | Přiřadí příspěvek ke kategoriím |

### Vzhled
| Nástroj | Popis |
|---|---|
| `set_styles` | Barvy, CSS, menu, patička, analytika |
| `manage_block` | Spravuje sdílené bloky |

### Soubory
| Nástroj | Popis |
|---|---|
| `upload_file` | Nahraje soubor (base64 nebo URL) |
| `list_folders` | Zobrazí složky s obrázky/soubory |

---

## ROZHODOVACÍ STROM NÁSTROJŮ

```
Nový web od začátku:
  list_websites → get_context → set_styles (barvy+menu+patička+CSS)
  → create_page → add_section → preview → [publish na pokyn]

Doplnění sekce na existující stránku:
  get_context → najdi pageId → add_section → preview

Úprava existující sekce:
  get_context → najdi sectionId → get_section → edit_section → preview

Nový blogový článek:
  get_context → [create_category] → create_post → assign_post_categories
  → preview → publish_post → [publish na pokyn]

Změna stylů / menu / patičky:
  get_context → set_styles → preview

Audit webu:
  get_context → analyzuj → vrať doporučení (bez změn!)

Import dat:
  validate → [pokud OK] → import (merge/replace dle zadání)
```

---

*Repozitář: https://github.com/matkalcz/profistranka*
*Poslední aktualizace: 2026-03-21*
