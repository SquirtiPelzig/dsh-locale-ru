# Integration guide

Two paths. Both use the dictionaries in `../dictionaries/`, whose keys match the English
source exactly.

## Option A — official language pack (recommended)

The pack in `../dsh-locale-ru/` is a normal DSH client plugin: it mounts as a loader row,
registers the language and dictionaries as owned effects, and disappears cleanly when the
row is removed.

### A.1 Package layout

```
packages/client/locale-ru/
  package.json     # name, exports "./client", dsh.client.platform = "web"
  src/index.ts     # host half: export function apply() {}
  src/client.ts    # language + dictionaries registration
```

`package.json` keeps the same contract every browser package uses:

```json
{
  "name": "@deepseek-ai/dsh-locale-ru",
  "type": "module",
  "main": "lib/index.js",
  "exports": {
    ".": "./lib/index.js",
    "./client": "./lib/client.js",
    "./package.json": "./package.json"
  },
  "dsh": { "client": { "platform": "web" } }
}
```

### A.2 Client registration

```ts
export const inject = ['locale']

export function apply(ctx) {
  ctx.effect(
    () => ctx.locale.addLanguage({ id: 'ru', label: 'Русский', fallback: 'en' }),
    'locale-ru: language',
  )
  ctx.effect(() => {
    const disposers = Object.keys(dictionaries).map((ns) =>
      ctx.locale.register(ns, 'ru', dictionaries[ns]),
    )
    return () => { for (const dispose of disposers.reverse()) dispose() }
  }, 'locale-ru: dictionaries')
}
```

The generated `client.js` in `../dsh-locale-ru/` is exactly this, compiled to the
`window.__ModuleLoader__.load({ id, factory })` form the client module system expects.

### A.3 Roster row

In `packages/bundle/web-app/cordis.patch.yml`, next to the other browser rows:

```yaml
    - id: locale-ru
      name: '@deepseek-ai/dsh-locale-ru'
```

Do **not** set `locale.preference` in a shipped layer: the language should appear in
Settings → General → Language and stay opt-in. Note that `detectBrowserLocale` matches the
browser's primary subtag, so a `ru-*` browser selects Russian automatically once the
language is registered — decide whether that is the desired default.

### A.4 Install paths

- Bundled with the desktop app: add the package to the runtime dependencies that ship in
  `resources/app.asar` (`dsh/node_modules/@deepseek-ai/…`).
- Self-hosted profiles: `dsh plugin --profile <name> add @deepseek-ai/dsh-locale-ru`.

## Option B — built-in locale

Makes `ru` a first-class locale instead of a pack.

1. `packages/client/locale/src/locale-settings.ts`: add `'ru'` to `LOCALE_IDS`.
2. `packages/client/locale/src/client/…`: add the `{ id: 'ru', label: 'Русский', fallback: 'en' }`
   entry to `BUILT_IN_LOCALES` (it currently maps `LOCALE_IDS`).
3. Widen the `LocaleNamespaceMap` / typed key contract so `register(ns, { zh, en, ru })`
   type-checks; today the object form requires both shipped locales.
4. Each owning package: add `ru` next to `zh`/`en` in `src/client/locale.ts` and pass it to
   `ctx.locale.register(NS, { zh, en, ru })`. `dsh-client-ui-sidebar-documentpreview`
   registers six namespaces this way; the per-locale form already used by
   `dsh-client-ui-directory-picker-browse` is another option.
5. `verify-client-ui-i18n` will require a complete `ru` dictionary for every namespace —
   `../dictionaries/` satisfies that today.

Trade-off: one bundle and no extra loader row, but a multi-package diff, a stricter
compile-time contract, and every future key must be translated in the same PR.

## Strings that need care

| Namespace / key | Value | Why |
| --- | --- | --- |
| `chat` / `message.stepProcess.sharedPrefix` | `""` | prefix stripped from `done.*` titles; a non-empty value changes the comparison |
| `pluginManager` / `sentenceSeparator` | `" "` | joined between complete sentences |
| `common` / `number.thousand`, `number.million` | `{value} тыс.`, `{value} млн` | unit suffix |
| `chat` / `clock.md`, `clock.ymd` | `{d}.{m}`, `{d}.{m}.{y}` | date order |
| `workspace` / `date.ymd` | `{y}-{m}-{d}` | technical format, intentionally unchanged |

Everything else is ordinary copy. Placeholders must stay byte-identical; a validator for
that is described in `VERIFICATION.md`.
