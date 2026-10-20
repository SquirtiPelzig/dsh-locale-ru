# Verification

Environment: Windows x64, DSH Desktop `0.2.1-rc.2` (`332895e`), bundled Node 24.21.0.

## 1. Dictionary integrity

Script over `dictionaries/*.json`:

- 63 namespaces, 2730 keys — every key present in the English source exists in `ru`;
- no extra keys, no empty values where the source has text;
- placeholder sets match the source **exactly** (sorted comparison of every `{…}` token);
- whitespace-only and empty formatting seams match the source byte for byte.

## 2. Bundle format

- `dsh-locale-ru/client.js` parses (`node --check`);
- executing it with a stub `window.__ModuleLoader__` yields one registration with
  `id === "dsh-locale-ru"`, `apply` and `inject = ["locale"]`;
- calling `apply` with a recording context performs exactly two effects: one
  `addLanguage({ id: "ru", label: "Русский", fallback: "en" })` and one registration set
  covering all 63 namespaces / 2730 keys.

## 3. Host wiring (isolated instance)

Booted a separate `dsh web` profile (own `DSH_HOME`, port 53177) with the pack installed
and the loader row added:

- boot completed with an empty stderr;
- the boot manifest (`globalThis.__DSH_BOOT__`) contains
  `{"id":"dsh-locale-ru","url":"plugins/??dsh-locale-ru/client.js&rev=…"}`;
- `GET /plugins/??dsh-locale-ru/client.js&rev=…` returns `200 text/javascript` with the
  bundle verbatim.

## 4. Runtime behaviour (shipped `LocaleRuntime`)

Loaded `@deepseek-ai/dsh-client-locale/lib/client.js` from the installed application,
stubbed its module imports and DOM, and drove the real class with a host scope holding
`preference: "ru"`:

```
PASS  active locale is ru
PASS  ru text used
PASS  en fallback for missing key
PASS  params interpolate
PASS  unknown key shows key
PASS  selector lists ru
snapshot: {"active":"ru","locales":[{"id":"zh",…},{"id":"en",…},{"id":"ru","label":"Русский","fallback":"en"}],"revision":3}
```

## 5. Live desktop check

The desktop app mounts the pack from
`~/.dsh/profiles/desktop/plugins/dsh-locale-ru/` with `locale.preference: ru` in the
profile patch. Reloading the window applies the language; the pack is additive, and
deleting the loader row restores English.

## Reproducing the checks

The validator compares a translation file against the English source and rejects
missing/extra keys, empty values and placeholder drift; the runtime check executes the
shipped `LocaleRuntime` against a host scope with a stored preference. Both are small
Node scripts and can be contributed as fixtures if useful.
