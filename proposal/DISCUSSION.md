## Summary

I translated the whole DSH web UI into Russian and would love to see `ru` shipped as a selectable language in a future release, so users can pick it in **Settings → General → Language**.

**Coverage:** 63 namespaces, 2730 strings — every client-owned dictionary in `0.2.1-rc.2` (`332895e`): `common`, `chat`, `conversation`, `trajectory`, `settings*`, `sidebar*`, `job`, `question`, `plan`, `goal`, `pluginManager`, `pluginMarket`, `voice-input`, `workspace`, `schedule.*`, `deliverables`, `feedback`, `skill`, `subagent`, `workflowRun`, …

## It already works today

The pack is built on the documented external-locale API, so it needs no core changes:

```js
ctx.locale.addLanguage({ id: 'ru', label: 'Русский', fallback: 'en' })
ctx.locale.register(ns, 'ru', dictionary)
```

Dropping it into a profile's `plugins/` directory and adding one loader row makes «Русский» appear in Settings → General → Language. Missing keys fall back to English, so keys added by future releases degrade gracefully instead of showing raw keys.

## Links

- Pack, install steps, structure: https://github.com/SquirtiPelzig/dsh-locale-ru
- `{ en, ru }` dictionaries per namespace, ready for review: https://github.com/SquirtiPelzig/dsh-locale-ru/tree/main/dictionaries
- Coverage report: https://github.com/SquirtiPelzig/dsh-locale-ru/blob/main/coverage.json
- Two integration options for the team: https://github.com/SquirtiPelzig/dsh-locale-ru/blob/main/proposal/INTEGRATION.md

## Notes for whoever reviews it

- Placeholders (`{count}`, `{name}`, `{version}`, …) are preserved verbatim. Two values are formatting seams rather than copy: `chat` / `message.stepProcess.sharedPrefix` is `""` and `pluginManager` / `sentenceSeparator` is `" "`.
- The locale registry deliberately has no plural rules (documented limitation), so Russian strings avoid number agreement instead of guessing at it: `Файлов: {count}`, `Ходов: {turns}, шагов: {steps}`. If plural support ever lands, a handful of strings can be made more idiomatic.
- One glossary is used across all namespaces (session → сессия, workspace → рабочая область, tool → инструмент, subagent → субагент, approval → подтверждение, sandbox → песочница, …), so the UI reads consistently.
- Verified against the shipped `LocaleRuntime`: `ru` becomes active from a stored preference, serves Russian, falls back to English for a missing key, interpolates parameters, and lists «Русский» in the selector.

## Two ways to ship it

1. **Official language pack** — publish the pack as a package and add its row to the browser roster in `packages/bundle/web-app/cordis.patch.yml`, leaving `locale.preference` unset. Small, additive, reversible; no changes to the 63 UI packages.
2. **Built-in locale** — fold each `ru` dictionary into the owning package's `src/client/locale.ts` and add `ru` to `LOCALE_IDS`. Larger diff and a stricter compile-time contract, but `ru` becomes a first-class locale.

I am glad to reshape the contribution into whichever form the team prefers, or provide the dictionaries in any other layout.
