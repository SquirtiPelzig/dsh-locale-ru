# dsh-locale-ru

Russian (`ru`) language pack for the [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)
web UI — **63 namespaces, 2730 strings**, prepared against DSH Desktop `0.2.1-rc.2` (`332895e`).

The pack is built on the documented external-locale API, so it needs no core changes:

```js
ctx.locale.addLanguage({ id: 'ru', label: 'Русский', fallback: 'en' })
ctx.locale.register(ns, 'ru', dictionary)
```

Lookup walks `ru → en` per namespace, then `common`, then the key itself, so a key added by a
future DSH release shows English instead of a raw key.

## Install

Copy the package into the profile's plugin directory and add one loader row.

```
<DSH_HOME>/profiles/<name>/plugins/dsh-locale-ru/
  index.js  client.js  package.json
```

`<DSH_HOME>/profiles/<name>/cordis.patch.yml`:

```yaml
- id: locale
  name: '@deepseek-ai/dsh-client-locale'
  config:
    preference: ru
- insert:
    - id: locale-ru
      name: ./plugins/dsh-locale-ru/index.js
```

Reload the window (`Ctrl+R`); if the UI is still English, restart DSH. The language then also
appears in **Settings → General → Language** as «Русский», next to English and 中文.

Set `preference` to `en` (or drop the two rows) to go back.

## What is covered

`dictionaries/<namespace>.json` holds `{ namespace, packages, en, ru }` for every client-owned
dictionary: `common`, `chat`, `conversation`, `trajectory`, `settings*`, `sidebar*`, `job`,
`question`, `plan`, `goal`, `pluginManager`, `pluginMarket`, `voice-input`, `workspace`,
`schedule.*`, `deliverables`, `feedback`, `skill`, `subagent`, `workflowRun`, … See
[`coverage.json`](coverage.json).

## Translation notes

- Placeholders (`{count}`, `{name}`, `{version}`, …) are kept byte-identical.
- Formatting seams are preserved: `chat` / `message.stepProcess.sharedPrefix` is `""` and
  `pluginManager` / `sentenceSeparator` is `" "` — both are concatenation seams, not copy.
- The locale registry has no plural rules (documented limitation), so Russian strings avoid
  number agreement instead of guessing: `Файлов: {count}`, `Ходов: {turns}, шагов: {steps}`.
- One glossary across namespaces: session → сессия, workspace → рабочая область, tool →
  инструмент, subagent → субагент, approval → подтверждение, sandbox → песочница, …

## Verification

- structure: every English key exists in `ru`, no extras, no empty values, placeholder sets match;
- bundle: parses, registers one module `dsh-locale-ru`, `apply` performs exactly two effects;
- host: mounts as a client row, `/plugins/…/client.js` serves it;
- runtime: the shipped `LocaleRuntime` activates `ru` from a stored preference, serves Russian,
  falls back to English for a missing key, interpolates parameters and lists «Русский».

Details in [`proposal/VERIFICATION.md`](proposal/VERIFICATION.md).

## Upstream proposal

[`proposal/ISSUE.md`](proposal/ISSUE.md) is a ready feature request asking DSH to ship `ru` as a
selectable language, and [`proposal/INTEGRATION.md`](proposal/INTEGRATION.md) describes two ways to
do it (official language pack, or folding the dictionaries into each package as a built-in locale).

## License

MIT. Strings are original translations of the English dictionaries shipped in the packages listed
in `coverage.json`.
