# Quran metadata (M1)

The app bundles three tables describing the Madani (Hafs) 604-page mushaf, 15 lines per page.
They contain page, surah, juz and hizb numbers and surah names only: no Quran text.

| File                      | Entries | Fields                                                                             |
| ------------------------- | ------- | ---------------------------------------------------------------------------------- |
| `assets/data/surahs.json` | 114     | `number`, `nameAr`, `nameEn`, `nameFr`, `ayahCount`, `startPage`, `endPage`, `juz` |
| `assets/data/pages.json`  | 604     | `page`, `juz`, `hizbQuarter` (1-240), `surahsOnPage`, `firstSurah`, `firstAyah`    |
| `assets/data/juz.json`    | 30      | `number`, `startPage`, `endPage`, `nameAr`                                         |

The app reads them through `src/data/mushaf.ts` (a `MushafLayout` from `src/domain/mushaf.ts`).
Another layout, such as Warsh, can be added later as a second `MushafLayout` with its own spec.

## Source and licence

- **Tanzil Quran Metadata v1.0**, `scripts/sources/tanzil/quran-data.js`, from
  <https://tanzil.net/res/text/metadata/quran-data.js>. Copyright (C) 2008-2009 Tanzil.info,
  licensed under [Creative Commons Attribution 3.0](https://creativecommons.org/licenses/by/3.0/).
  SHA-256 `9e9930c592aaa34b5c7a79ff4e99ffdc782263a266090bfe2c49e4ed4bac4ca4`, retrieved 2026-10-15.
- Attribution is required: it is in the README now and goes on the About screen in M11.

**Never edit the JSON by hand.** Change the generator, then run `npm run data:build`. A test
fails if the committed JSON differs from what the source produces.

## How the tables are derived

`scripts/build-quran-data.js` reads Tanzil's per-ayah boundaries (the first ayah of each page,
juz and hizb quarter) and derives:

- **Surah page range:** the page of ayah 1 to the page of the last ayah.
- **Surahs on a page:** every surah with at least one ayah on that page.
- **Juz and hizb quarter of a page:** the ones in effect at the page's **last** ayah. So a juz
  that begins part-way down a page owns that page, as in the printed mushaf, where the juz marker
  is on that page. This makes the 30 juz page ranges contiguous and non-overlapping, which the
  map's one-row-per-juz layout needs. It matches every juz start page listed in PLAN.md section 3.
- **Juz of a surah:** the juz of its first ayah.
- **Juz names:** Arabic ordinals ("الجزء الأول" to "الجزء الثلاثون"). The traditional names
  ("عم", "تبارك"...) are the juz's opening words, so they count as Quran text, which v1 leaves out.

## Verification

1. **Anchor tests** (`src/data/__tests__/mushaf.test.ts`): Al-Fatiha alone on page 1,
   Al-Baqarah on pages 2-49, juz 30 starting on page 582, page 604 holding surahs 112-114, and
   all 30 juz start pages from PLAN.md section 3. **All match.**
2. **Structural validator** (`validateMushafData`): counts (604 / 114 / 30 / 6236 ayahs),
   ordering, and consistency between the three tables. **No problems.**
3. **Independent second source** (`npm run data:verify`, needs network): every one of the 6236
   ayahs compared against the quran.com API (v4) for page, juz and hizb quarter, plus each surah's
   ayah count, Arabic name and page range.
   - Pages: 6236 of 6236 match. Juz: 6236 of 6236 match.
   - Surahs: all 114 ayah counts, Arabic names and page ranges match.
   - Hizb quarters: **1 mismatch**, reported below.

## Mismatches and notes (all reviewed and confirmed 2026-10-15)

1. **The uploaded file was double-encoded.** Its Arabic surah names were mojibake (UTF-8 read as
   Windows-1252 and saved again). The bytes could be recovered, and after repair the file
   matched Tanzil's original byte for byte, except for a trailing newline. I replaced it with the
   original from tanzil.net (hash above).
2. **Hizb quarter 106 starts at 15:49 on quran.com but at 15:50 in Tanzil.** Both ayahs are on
   page 265, so the page-level data is the same either way. Kept Tanzil's value. Nothing to
   decide unless ayah-level quarters are ever needed.
3. **Four juz begin part-way down a page** in both sources: juz 4 at 3:93 (page 62), juz 7 at
   5:82 (page 121), juz 11 at 9:93 (page 201) and juz 26 at 46:1 (page 502). Under the
   convention above, those pages belong to the new juz. PLAN.md's anchor list implies the same.
   These are the Madani boundaries; some other divisions start juz 4 at 3:92 ("لن تنالوا").
4. **Latin surah names are Tanzil's transliteration**, which writes long vowels as doubled
   letters ("Al-Faatiha", "Aal-i-Imraan"). Tanzil has no French names, so `nameFr` currently
   equals `nameEn`. Confirmed: keep them, the same for English and French.
5. **Arabic surah names follow Tanzil's spelling**, which quran.com uses too: for example "ابراهيم"
   without a hamza, "سبإ", "النبإ", "الانسان". Confirmed: no changes.
