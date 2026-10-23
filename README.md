# Дневник исхране — Nutrition Diary

A browser-based food diary that tracks daily protein, carbohydrate, fat and calorie intake. Pick a food, enter the weight in grams, and the macros are computed and totalled per meal and per day.

**Live:** <https://ishrana.onrender.com>

No backend, no build step, no dependencies — everything runs in the browser and persists to `localStorage`.

---

## Features

**Built-in food database** — 324 foods with per-100 g values for protein, carbohydrates, fat and calories. Every entry carries both a Serbian and an English name, so search works in either language.

**Custom foods** — add your own entries with their macro values; they persist across sessions and appear in the same picker as the built-in ones.

**Editable meal structure** — the default day is five meals (breakfast, two snacks, lunch, dinner) with suggested times, but meals can be renamed, retimed, added or removed. The layout is saved as a template and reused for every day.

**Calendar and history** — each day is stored under its own date. Navigate by month, jump to a specific date, or click *Today*. Days with entries are marked in the calendar.

**Bilingual UI with live switching** — Serbian and English, toggled at runtime without a reload. Serbian renders in Cyrillic via a transliteration pass rather than a second set of translation strings (see below).

**Automatic totals** — per-meal and per-day sums for protein, carbs, fat and kcal, recalculated as you type.

---

## Implementation notes

### Latin-to-Cyrillic transliteration

Serbian UI strings are authored once in Latin and converted to Cyrillic at render time by `toCyrillic()`. The conversion handles the three digraphs that map to single Cyrillic letters — `dž → џ`, `lj → љ`, `nj → њ` — before falling back to single-character mapping, and preserves the original casing of each character:

```js
toCyrillic("Užina")   // → "Ужина"
toCyrillic("NJEGOV")  // → "ЊЕГОВ"
```

A small exclusion list keeps certain keys out of the transliteration pass, so labels like `kcal` stay readable.

### Persistence

Four independent `localStorage` namespaces, each versioned so the schema can change without corrupting existing data:

| Key | Contents |
|---|---|
| `dnevnik-ishrane-history-v1` | Diary entries, keyed by ISO date |
| `dnevnik-ishrane-custom-foods-v1` | User-added foods |
| `dnevnik-ishrane-meals-v1` | Meal template — labels, order, times |
| `dnevnik-ishrane-lang-v1` | Selected language |

All data stays in the browser; nothing is sent anywhere.

---

## Structure

```
nutrition-diary/
├── index.html          # markup only
├── css/
│   └── styles.css
└── js/
    ├── data/
    │   ├── foods.js    # 324-item food database
    │   └── i18n.js     # UI strings (sr / en)
    └── app.js          # application logic
```

Data is separated from logic so the food database can be extended without touching application code, and the two data files can be regenerated from an external source if one is ever added.

## Running locally

```bash
git clone https://github.com/nedeljko02/nutrition-diary.git
cd nutrition-diary
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Opening `index.html` directly also works.

## Adding foods to the database

Entries in `js/data/foods.js` use per-100 g values:

```js
{ "name": "Piletina", "p": 21.0, "uh": 0.0, "m": 3.0, "kcal": 111.0, "en": "Chicken" }
```

`p` is protein, `uh` carbohydrates, `m` fat — all in grams. `name` is the Serbian label (written in Latin; transliterated to Cyrillic at render time) and `en` its English equivalent.
