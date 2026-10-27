# Verified Runtime — Asterigos (cross-mod)

**Создано:** 2026-10-27. **Статус записей:** *промотированные сведения из `NikichMods/AsterigosMods`: принятая база и отдельно обозначенные локальные runtime-наблюдения; повторных тестов этим Research-репозиторием не проводилось*. Это не универсальная таблица совместимости.

## R-001 — Подтверждённая в проекте базовая среда

- **Источник:** [`AsterigosMods/AGENTS.md`](https://github.com/NikichMods/AsterigosMods/blob/main/AGENTS.md) и [`AsterigosMods/docs/VERIFIED_RUNTIME.md`](https://github.com/NikichMods/AsterigosMods/blob/main/docs/VERIFIED_RUNTIME.md).
- **Среда, указанная проектом:** Asterigos: Curse of the Stars **Steam 1.09 / build 12996889**, **UE 4.27**, **Windows x64**.
- **Интеграция:** внешний **UE4SS v3.0.1-1181-g83330635** и UEHelpers; в исходном проекте это зарегистрированный рабочий baseline.
- **Область применимости:** только указанная среда. Другие сборки Steam/GOG, DX-режимы и версии UE4SS **не объявляются совместимыми автоматически**.
- **Практическое следствие:** новые исследования конкретных хуков, объектов и UI должны привязываться к этому baseline либо явно указывать отличие.

## R-002 — Мерцающие чёрные участки и графический API

- **Источник:** [`AsterigosMods/docs/VERIFIED_RUNTIME.md`](https://github.com/NikichMods/AsterigosMods/blob/main/docs/VERIFIED_RUNTIME.md), принятое пользовательское игровое наблюдение.
- **Наблюдение:** переключение в **DX11** устранило стационарное чёрное мерцание на шероховатых скалах в испытанной конфигурации.
- **Не установлено:** настоящая первопричина поведения в DX12, распространённость на других драйверах/сценах и связь с **иными** визуальными артефактами. Не превращать это в универсальный диагноз «DX12 сломан».
- **Практическое следствие:** перед созданием визуального мода сверять выбранный API и точный симптом; не дублировать диагностику уже решённого в указанном случае мерцания.

## R-003 — Общий runtime-контракт проекта

- **Источник:** [`AsterigosMods/docs/VERIFIED_RUNTIME.md`](https://github.com/NikichMods/AsterigosMods/blob/main/docs/VERIFIED_RUNTIME.md).
- **Принятое интеграционное ограничение:** изменения runtime должны исполняться на игровом потоке, с проверкой класса/жизненного цикла UObject и безопасным выходом при неподдерживаемом состоянии.
- **Важно:** это зарегистрированное требование `AsterigosMods`, а **не** доказательство, что каждый гипотетический UE4SS-хук обладает одинаковыми контрактами. Точные владельцы и доступные события проверяются отдельно.
- **Для производства:** версии, тесты, установку и приёмку всегда определяет локальный мод внутри `AsterigosMods`.

## R-004 — Нативное чтение HP и startup owning pawn через UE4SS

- **Уровень:** технически подтверждено в локальной read-only сессии 2026-10-27, Steam 1.09/build12996889, UE4.27, Windows x64, UE4SS Git SHA `83330635`.
- **Источник и точная идентичность:** [CombatHUD runtime report](https://github.com/NikichMods/AsterigosMods/blob/2c6c959f9fbf901616815e6b23df18b8c8dc1c4e/CombatHUD/docs/VERIFIED_RUNTIME.md), commit `2c6c959f9fbf901616815e6b23df18b8c8dc1c4e`. Probe source `9f1514058029aa77928faf03daf3f94ea9c7fed0`; SHA256 Lua `a4407110e1aeb95f12ba04cf906d860228b149401aafbfbfcebb1db9d0381ab6`; log SHA256 `cb62a34b09212c75f26146dfd3e33b744c7decad0a77d2fdf5a2337ba8aa1694`. Полный лог остаётся локальным; исследовательский probe удалён.
- **Подтверждённый путь чтения:** `/Script/Genesis.GameGlobal:GetHP` и `GetMaxHP` имеют один `ObjectProperty Character` и `FloatProperty ReturnValue`; flags обеих `335684609`, static bit `0x2000`. На `GameGlobal /Script/Genesis.Default__GameGlobal` успешны Lua-вызовы `global:GetHP(character)` / `global:GetMaxHP(character)`, Lua type `number`. Аргумент — валидный owning pawn HUD, проверенный по ancestry до `GenesisCharacter`, в сессии `hilda_BP_C`.
- **Наблюдение:** `680/680 → 532.07415771484/680 → 680/680`, 577 samples; current HP дробное, max оставался 680. В проверенном цикле неполное HP различалось через `hp < max`, полное — точным равенством, без округления. Не заменять реальные HP сглаженным fill или целочисленным текстом UI.
- **Жизненный цикл:** ранний in-viewport `HUDMainUMG_BP_C` в `UIOnlyMap` принадлежал `DefaultPawn`; позже другой HUD получил `hilda_BP_C`. Сам по себе viewport HUD не доказывает игровой pawn. Неподходящего владельца следует пропускать до вызова getters, не искать первого персонажа в мире. Вложенный UserWidget также разрывает непрерывный обход GetParent: status root `CanvasPanel_0:GetParent()` = nil; связь к owning status восстанавливается через verified owner references. Конкретные слои/требования HUD принадлежат CombatHUD.
- **Пределы:** C++ final writer HP и события commit не установлены; sampled delta не доказывает каждое изменение между samples. Не проверены playable pawn replacement/respawn, buffs/max changes, все виды урона, FPS, иные сборки и версии UE4SS. Успешное чтение и readonly widget traversal не доказывают отсутствие native opacity writer и визуальную корректность будущей модификации.
- **Потребители:** CombatHUD (непрерывное чтение HP); возможные другие моды реакции на урон могут переиспользовать accessor/lifecycle evidence, но обязаны отдельно установить нужную полноту событий и свой consumer.

## Что не скопировано сюда

Не импортированы все исходники, журналы экспериментов и мод-специфические ID из подпроектов. При первом повторном использовании выявить конкретный применимый факт, сохранить доказательства и перенести только нужный переиспользуемый вывод в этот документ.
