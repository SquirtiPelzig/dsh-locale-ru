# dsh-locale-ru — русский язык для DeepSeek Harness

Языковой пакет для веб-интерфейса [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness):
**63 пространства имён, 2730 строк**. Собран под DSH Desktop `0.2.1-rc.2` (`332895e`).

Пакет использует документированный API внешних локалей, поэтому не требует правок ядра:

```js
ctx.locale.addLanguage({ id: 'ru', label: 'Русский', fallback: 'en' })
ctx.locale.register(ns, 'ru', dictionary)
```

Поиск идёт по цепочке `ru → en` в каждом пространстве имён, затем в `common`, затем
показывается сам ключ. Поэтому строки, добавленные в новых версиях DSH, будут показаны
по-английски, а не сырым ключом.

## Установка

Скопируйте пакет в каталог плагинов профиля и добавьте одну строку загрузчика.

```
<DSH_HOME>/profiles/<имя>/plugins/dsh-locale-ru/
  index.js  client.js  package.json
```

`<DSH_HOME>/profiles/<имя>/cordis.patch.yml`:

```yaml
- id: locale
  name: '@deepseek-ai/dsh-client-locale'
  config:
    preference: ru
- insert:
    - id: locale-ru
      name: ./plugins/dsh-locale-ru/index.js
```

Перезагрузите окно (`Ctrl+R`); если интерфейс остался английским — перезапустите DSH. После
этого язык появится и в «Настройки → Общие → Язык» как «Русский», рядом с English и 中文.

Чтобы вернуть английский, поставьте `preference: en` или удалите обе строки.

## Что покрыто

`dictionaries/<namespace>.json` содержит `{ namespace, packages, en, ru }` для каждого
словаря клиента: `common`, `chat`, `conversation`, `trajectory`, `settings*`, `sidebar*`,
`job`, `question`, `plan`, `goal`, `pluginManager`, `pluginMarket`, `voice-input`,
`workspace`, `schedule.*`, `deliverables`, `feedback`, `skill`, `subagent`, `workflowRun` и др.
Полный список — в [`coverage.json`](coverage.json).

## Особенности перевода

- Плейсхолдеры (`{count}`, `{name}`, `{version}` и т. д.) сохранены дословно.
- Сохранены «швы» форматирования: `chat` / `message.stepProcess.sharedPrefix` — пустая
  строка, `pluginManager` / `sentenceSeparator` — пробел; это склейки, а не текст.
- В реестре локалей нет правил множественного числа (документированное ограничение),
  поэтому формы построены без числового согласования: `Файлов: {count}`,
  `Ходов: {turns}, шагов: {steps}`.
- Единый глоссарий: сессия, рабочая область, инструмент, субагент, подтверждение,
  песочница, пресет, навык, плагин.

## Проверено

- структура: все английские ключи есть в `ru`, лишних нет, пустых значений нет, наборы
  плейсхолдеров совпадают;
- бандл: парсится, регистрирует один модуль `dsh-locale-ru`, `apply` даёт ровно два эффекта;
- хост: монтируется строкой загрузчика, `/plugins/…/client.js` отдаётся сервером;
- рантайм: штатный `LocaleRuntime` включает `ru` из сохранённого предпочтения, отдаёт
  русский текст, падает в английский для отсутствующего ключа, подставляет параметры и
  показывает «Русский» в списке языков.

Подробности — в [`proposal/VERIFICATION.md`](proposal/VERIFICATION.md).

## Предложение для апстрима

[`proposal/ISSUE.md`](proposal/ISSUE.md) — готовый текст запроса, чтобы `ru` вошёл в поставку
как выбираемый язык; [`proposal/INTEGRATION.md`](proposal/INTEGRATION.md) описывает два способа
интеграции (официальный языковой пакет или встроенная локаль).

## Лицензия

MIT. Строки — оригинальный перевод английских словарей из пакетов, перечисленных в
`coverage.json`.
