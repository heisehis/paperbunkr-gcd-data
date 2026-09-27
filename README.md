# Paperbunkr GCD data

A compact extract of the [Grand Comics Database™](https://www.comics.org) (GCD), published as an optional download for
[Paperbunkr](https://github.com/heisehis/PaperBunkr), the comic and manga library and reader.

Paperbunkr uses it to link series that continue each other (renumberings, renames, publisher changes, merges and reboots)
and to order story events by when issues went on sale. You don't need this repository to use it: in Paperbunkr, open
**Preferences → Connections → Grand Comics Database data** and click **Download**.

## What's here

| File | What it is |
|---|---|
| [`gcd-data.json`](gcd-data.json) | The manifest Paperbunkr reads: the data's dump date, the download address, its SHA-256 and size, and the extract's schema version. |
| [Releases](https://github.com/heisehis/paperbunkr-gcd-data/releases) | One release per GCD data dump (`gcd-YYYY-MM-DD`), with the extract as `gcd-YYYY-MM-DD.zip`. |
| [`LICENSE`](LICENSE) | Creative Commons Attribution-ShareAlike 4.0 International, the licence of the data. |

## Current data

| Dump | Series | Issues | Series bonds | Zip | Extract |
|---|---|---|---|---|---|
| [2026-09-15](https://github.com/heisehis/paperbunkr-gcd-data/releases/tag/gcd-2026-09-15) | 229,836 | 2,203,939 | 6,798 | 33.7 MB | 97.7 MB |

## Attribution and licence

This data comes from the **Grand Comics Database™** (<https://www.comics.org>) and is licensed under the
[Creative Commons Attribution-ShareAlike 4.0 International licence](https://creativecommons.org/licenses/by-sa/4.0/)
(CC BY-SA 4.0), the same licence as the GCD data it is derived from. Every series and issue carries its GCD id, and its page
is at `https://www.comics.org/series/{id}/` or `https://www.comics.org/issue/{id}/`. Paperbunkr links GCD-derived lines to
those pages and credits the GCD in its Preferences and in About → Legal & notices.

The Grand Comics Database does not endorse Paperbunkr or this extract. "Grand Comics Database" is a trademark of the Grand
Comics Database project.

### Changes made to the GCD data

The extract is a subset of the GCD's SQLite data dump, reshaped for matching. It is not a copy of the dump:

- **Series:** comic publications only (not deleted). The extract keeps the id, name, first and last year, publisher name,
  language and issue count, plus a normalized `name_key` added for name matching.
- **Issues:** not deleted, and variant covers left out. The extract keeps the id, series, number, key date and on-sale date.
- **Series bonds:** all of them, with the bond type's name in place of its id.
- **Left out:** stories, credits, characters, covers, indexer notes, story arcs, and everything else.
- **Added:** a `meta` table recording the source dump date, the build time, the schema version, and this licence and
  attribution.

The extract is built from the dump by `tools/Paperbunkr.GcdExtract` in the
[Paperbunkr repository](https://github.com/heisehis/PaperBunkr).

## Format

`gcd.sqlite` (schema version 1), inside the zip:

```sql
series      (id, name, name_key, year_began, year_ended, publisher, language, issue_count)
issue       (id, series_id, number, key_date, on_sale_date)   -- dates "yyyy-mm-dd", 00 for unknown parts
series_bond (origin_id, target_id, origin_issue_id, target_issue_id, bond_type)   -- origin = older, target = newer
meta        (key, value)   -- dump_date, built_at, schema_version, licence, licence_url, attribution, notice
```

Bond types: `minor_name_numbering_continues`, `major_name_numbering_continues`, `publisher_numbering_continues`,
`subnumbering_continues`, `merge_numbering_continues`, `merge`, `reboot`.

## Updating

1. Download a newer GCD data dump from <https://www.comics.org/download/>. You need to be signed in and accept the GCD's terms.
2. In the Paperbunkr repository, run
   `dotnet run --project tools/Paperbunkr.GcdExtract -c Release -- <dump.db> <outDir> --url https://github.com/heisehis/paperbunkr-gcd-data/releases/download/gcd-<date>/gcd-<date>.zip`.
3. Create a release tagged `gcd-<date>` here with `gcd-<date>.zip` attached, then replace `gcd-data.json` in this repository
   with the one the tool wrote. Paperbunkr's **Check for update** reads it from here.
