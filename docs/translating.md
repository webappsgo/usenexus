# Translating

## Current status

The shared `src/common/i18n` package embeds six complete locale files and
provides language normalization, translation lookup, formatting, plural rules,
and validation. The client CLI and TUI use the locale package, and the
server-preferences page lets visitors choose a language. Server-rendered page
copy and notification emails have not yet been migrated to translation keys,
so those surfaces remain English. New locale data belongs in the embedded
locale directory and must be added to the shared language registry.

The current supported locales are English (`en`), Spanish (`es`), French
(`fr`), Arabic (`ar`), German (`de`), and Italian (`it`).

## Translation contribution rules

The English file is the source of truth and each locale is a UTF-8 JSON
translation with the same message keys. Translators change values, never keys,
use two-space indentation,
and preserve every `{placeholder}` exactly. Plural messages must use the CLDR
categories required by the language. The build will validate keys,
placeholders, empty values, and plural categories.

Contributions should use one locale file per change. AI-generated translations
are an initial draft; human corrections take precedence. Reviewers should
check meaning, glossary consistency, keyboard and screen-reader use, text
expansion, and layout at 320, 768, and 1280 pixels. Arabic also needs a
right-to-left layout review. Native-speaker review is welcome, but the required
review and test checks must still pass.

Translators may be credited in the locale file and on this page using the name
or handle they choose. No contributor is credited without their consent.

## Source-language glossary

These meanings should remain consistent across future translations. There
are no approved translated terms yet.

| English term | Meaning in GoSpool |
|---|---|
| Article | A Usenet message stored and displayed in a newsgroup archive. |
| Newsgroup | A named Usenet discussion group. |
| Mailing list | Email subscription and delivery behavior planned for the product. |
| Peer | Another Usenet server exchanging articles with GoSpool. |
| Feed | An NNTP transfer of articles between servers. |
| Subscriber | A user who receives mailing-list delivery for a group. |
| Message-ID | The RFC message identifier for an article. |
| Moderation | Review of a post before it is accepted for a moderated group. |
| Recovery key | A single-use code that can replace a TOTP code during administrator login. |

## Keep untranslated

Product and brand names; protocol and technology names such as API, Tor, I2P,
NNTP, JSON, and Message-ID; code, filenames, configuration keys, URLs, and
literal values users enter (for example `dark` and `auto`) remain unchanged.
Translate the surrounding sentence while preserving those tokens.

Follow each language's typography, punctuation, register, and word order.
French spacing, quotation marks, number formats, dates, and time conventions
must follow French usage; do not build sentences by concatenating fragments.

## Adding a locale

1. Copy the complete English locale file to the new BCP 47 locale code.
2. Translate every message and provide the metadata, direction, plural
   categories, and optional translator credit.
3. Add the language to the shared registry; the locale files and registry must
   represent exactly the same supported set.
4. Run `make test` and rebuild every binary so the embedded locale is included.
5. Update the language list and translator credits here and in the README.

The runtime language-selection order is `?lang=`, the `lang` cookie,
`Accept-Language`, then English. Unsupported languages fall back to English.
The `/server/preferences` selector stores a supported language choice in the
`lang` cookie. Continue migrating server page and email text to translation
keys before describing those surfaces as localized.
