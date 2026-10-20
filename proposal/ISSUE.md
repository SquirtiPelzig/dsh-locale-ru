# Feature: ship a Russian (ru) UI language

## Summary

DeepSeek Harness currently ships two interface languages (`en`, `zh`). I translated the
whole web UI into Russian — **63 namespaces, 2730 strings**, covering every client-owned
dictionary in `0.2.1-rc.2` — and validated it against the shipped `LocaleRuntime`.

Please ship `ru` as a selectable language so users can pick it in
**Settings → General → Language**.

## What is included

- `dictionaries/<namespace>.json` — `{ en, ru }` pairs for every namespace
  (`common`, `chat`, `conversation`, `trajectory`, `settings*`, `sidebar*`, `job`,
  `question`, `plan`, `goal`, `pluginManager`, `voice-input`, …).
- `dsh-locale-ru/` — a ready external language pack built on the documented API:
  `ctx.locale.addLanguage({ id: 'ru', label: 'Русский', fallback: 'en' })` +
  `ctx.locale.register(ns, 'ru', dict)`. It works today by dropping it into a profile's
  `plugins/` directory and adding one loader row.
- `coverage.json`, `proposal/VERIFICATION.md` — coverage and test evidence.

## Suggested integration

**Option A — ship it as an official language pack (small, safe, reversible).**
Publish the pack as `@deepseek-ai/dsh-locale-ru`, add its loader row to the browser
roster in `packages/bundle/web-app/cordis.patch.yml`, and leave `locale.preference`
unset. «Русский» then appears in the language selector for every install, and
untranslated future keys fall back to English. No changes to the 63 UI packages.

**Option B — make `ru` a built-in locale.**
Move each `ru` dictionary into the owning package's `src/client/locale.ts`, add `ru` to
`LOCALE_IDS` and to the `LocaleNamespaceMap` contract in `dsh-client-locale`. Larger
diff and a stricter compile-time contract; `verify-client-ui-i18n` will then require
complete `ru` dictionaries for every namespace.

Either way the dictionaries are drop-in: keys match the English source exactly.

## Notes for review

- Placeholders and formatting seams are preserved verbatim. Two values are deliberately
  invisible: `chat/message.stepProcess.sharedPrefix` is `""` and
  `pluginManager/sentenceSeparator` is `" "` — both are concatenation seams, not copy.
- The locale registry has no plural rules (documented limitation), so Russian strings
  avoid number agreement instead of guessing at it: `Файлов: {count}`,
  `Ходов: {turns}, шагов: {steps}`. If plural support lands, a handful of strings can be
  made more idiomatic.
- Terminology follows one glossary across namespaces (session → сессия, workspace →
  рабочая область, tool → инструмент, subagent → субагент, approval → подтверждение,
  sandbox → песочница, …) so the UI reads consistently.
- I did not touch `app.asar` or any package source; the pack is additive and the host
  half is a no-op. Removing the loader row returns the UI to English.

## Testing

Verified on Windows x64 against `0.2.1-rc.2` (`332895e`): the pack mounts as a client
row, the bundle is served at `/plugins/…/client.js`, and the shipped `LocaleRuntime`
serves Russian with English fallback (see `proposal/VERIFICATION.md`).

Happy to convert this into a PR in whichever shape you prefer (official pack vs.
per-package dictionaries).
