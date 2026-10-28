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

## R-005 — Native HUD menu owner и границы opacity-групп

- **Статус/среда:** локально проверено 2026-10-27 в baseline R-001, UE4SS `83330635`; пользователь подтвердил ожидаемое разделение яркости и корректное скрытие/возврат HUD в доступных меню.
- **Источник:** [CombatHUD visual report](https://github.com/NikichMods/AsterigosMods/blob/8e95840ade14f8ef17cffe0ddb86176f2799942d/CombatHUD/docs/VERIFIED_RUNTIME.md), commit `8e95840ade14f8ef17cffe0ddb86176f2799942d`. Immutable controller source `4445482eaed5ddc4f67d21509c00c61930f561fe`, Lua SHA256 `aeaf1caf747c08358d70fbd93b93bc3b03064590322361a60f80e1469f3c0a2a`; финальный log SHA256 `877bf17e6ae64554ee0f19e802f9432d557441201783a176b3600f1f2fc318a2`. Raw logs не публикуются.
- **Native consumer/owner:** общий `HUDMainUMG_BP_C` → `HUD_Panel` управляет нативным menu attenuation/hiding. Пока неперекрывающиеся дочерние группы сохраняли экспериментальные opacity, родитель сам менял alpha `1→0.5→1` и raw Visibility `3→2→3`. Подмена общего предка постоянной alpha1/Visibility нарушила бы этот boundary; оставлять его под native управлением. C++ метод-писатель не установлен по имени.
- **Проверенная граница:** 17 disjoint дочерних targets, one-time writes по выбору режима и read-only монитор вместо постоянного reassertion. В menu-сессии 5848 сравнений без расхождения target alpha; прямой rollback из split восстановил17 значений с readback, ошибок0. После закрытия игры исследовательский мод удалён, исходный loader list и production payload hashes восстановлены. Это не доказательство отсутствия transient writes между 50-мс samples, всех меню/кат-сцен и input switching.
- **Identity против display spelling:** у live item content `slot.Bg_Image` и объекта с `GetFName():ToString() == "BG_Image"` совпал адрес. Для адресного сопоставления reflected-ссылки с фактическими детьми нельзя требовать одинаковое строковое написание field key и displayed FName. Это конкретное наблюдение baseline, не универсальный список Lua aliases.
- **Неактивные ветки:** keyboard `CrystalList_Panel` в наблюдённых сессиях уже был native alpha0/Visibility2; альтернативная pad-группа была активна. Успешный опыт на активной ветке не подтверждает visible keyboard layout и не оправдывает фиксацию захваченного нуля как универсальной baseline. Не включать скрытую native ветку ради доказательства.
- **Потребители:** CombatHUD; будущие HUD-моды должны переиспользовать menu-owner/identity evidence, оставляя свой состав targets и acceptance в документах мода.

## R-006 — Interaction prompts: статическая цепочка и живые поля CanShow

- **Первоначальный статус:** статически подтверждено на ресурсах Steam
  1.09/build12996889, UE4.27. Отдельное runtime-дополнение ниже не закрывает
  production gate и не превращает всю статическую цепочку в live owner proof.
- **Первичный источник:** [Immersive Interaction Prompts static report](https://github.com/NikichMods/AsterigosMods/blob/207489fed083acb54e0fa5acae75990bfcdd5a57/ImmersiveInteractionPrompts/docs/VERIFIED_RUNTIME.md),
  commit `207489fed083acb54e0fa5acae75990bfcdd5a57`. Там находятся tool/input hashes,
  bytecode offsets, точные разрывы и воспроизведение; raw exports остаются локально.
- **Owner:** `/Game/MainContent/Template/BaseBlueprint/BaseGameEventObject_BP`
  содержит `ExtendedWidget:ExtendedWidgetComponent`, назначающий
  `/Game/MainContent/Template/UMG/TipHUD.TipHUD_C`, и `TipHUDArgumentComponent`.
  BeginPlay действительно получает widget из компонента, cast к TipHUD и передаёт
  `HoldPress` в `CanHoldPress` (ubergraph offsets 102/144/219). Это положительная
  executable связь, а не вывод из имени.
- **Consumer и локализация:** `TipHUD_C:SetupTipMessage_Proxy` →
  `ConvertMsgTagToTextArray` → `InteractTipToText` DataTable → FText StringTable
  `InteractTipToText_cs`, namespace `NewStringTable` → native
  `/Script/Genesis.ExtendedUserWidget:SetupTipMessage2` (ubergraph 713).
  Consumer получает массив `[TextBlock_89]`, ImagePanels с RetainerBox_0,
  display mode, native FText и textures. C++ final writer/поздние перезаписи
  **не установлены**; не называть Blueprint proxy final writer.
- **Native UHT-декларация (2026-10-28):** в одном разрешённом игровом сеансе
  штатный генератор UE4SS `83330635` вывел
  `void SetupTipMessage2(TArray<UTextBlock*>, TArray<UPanelWidget*>, ETipWidgetMode, TArray<FText>, TArray<UTexture2D*>)`.
  Все пять параметров — входные по значению, без const/ref/out; возврата нет.
  Это подтверждённая отражённая декларация, но не выполнение Lua-hook и не
  наблюдение final writer. [Источник, хэши и границы](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/946fbf61dc40e7018f5a5c5047e9e904f16a745c/research/interaction-prompts-owner/UHT_SIGNATURE.md).
  Подробная версия проверки генератора и ограничений — в R-010 ниже.
- **Доказанные базы:** BaseLoot2_BP → tag Take → RU UI.locres «Взять»;
  BaseLoot_BP → Open → «Открыть»; BaseDocumentObject через BaseInteractiveItem_BP
  → Check → «Проверить». Это component overrides и локализационные ресурсы,
  не утверждение о каждом placed/live объекте. Ladder_BP наследует отдельный
  native BaseClimbObject; общий prompt path и «спуститься» не доказаны.
- **Input/lifecycle:** base TipHUDArgument задаёт semantic Interact и
  IconNameIndex=InteractButton. DefaultInputMapping имеет F и
  Gamepad_FaceButton_Bottom, но это не текущий remapping игрока. Device→texture
  producer ещё неизвестен. Tick TipHUD пишет собственную opacity по DebugHideUMG
  и обновляет material scalar t прогресса удержания; CanShow зависит от native
  parent и input mode. Не фиксировать alpha или glyph и не заменять удержание
  предположением. Реальный show/hide/target/respawn lifecycle не наблюдался.
- **Применимость:** будущие interaction/UI моды могут переиспользовать точку
  исследования; это не доказательство родства с HUD_Panel CombatHUD или
  Coherent Hud.html. Мод-специфический дизайн/пробу/приёмку хранит исходный модуль.

### R-006 runtime-дополнение, 2026-10-28

- На том же Steam/UE4.27 baseline, UE4SS 83330635, zero-input Blueprint post-hook
  `/Game/MainContent/Template/UMG/TipHUD.TipHUD_C:CanShow` реально зарегистрировался
  и дошёл до 948 callbacks. В трёх разных TipHUD непосредственно прочитаны
  TextBlock_89.Text и stored Visibility/RenderOpacity/bIsEnabled: локализованные
  «Взаимодействовать», «Открыть», «Взять»; шесть changed snapshots. Это подтверждает
  этот узкий маршрут чтения без SetupTipMessage2 array arguments, не native hook.
- В каждой identity сначала прочитано `???`, затем действие; raw Visibility
  4(SelfHitTestInvisible) → 0(Visible), opacity=1, enabled=true.
  SelfHitTestInvisible не означает Hidden. Нельзя выводить из этого видимый
  placeholder, полный hide/show lifecycle, тип actor или final writer.
- [Точное свидетельство и пределы](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/1d0983ec76a33bdf396932949f2164e5848e1bb0/docs/VERIFIED_RUNTIME.md):
  runtime source `4cda59f27f4719a728a88d671e04cca901e08ab8`, script SHA256
  `ae75d8419d988ea91be288ba1aa33760ab980b354cd4d74d791564eec5109e09`,
  log SHA256 `c847c8ca032ede4c0540c6c4e1f2bbffbf2b2a0e70bf6565fb40cd5bcf2f7a7c`.
  Raw log stays local. Probe disabled after normal exit. Input icons, actor/
  component linkage, ladder, menu transitions and visual acceptance remain open.

### R-006 layout-дополнение, 2026-10-28

- На прежнем Steam 1.09/build12996889 / UE4.27 baseline offline проверены семь
  reciprocal Parent/Content/Slot связей: CanvasPanel_0 → HorizontalBox_4 →
  CanvasPanel_6; отдельные ветви TextBlock_89, RetainerBox_0 → allowed image,
  CanvasPanel_2 → forbidden image. Text slot AutoSize=true, anchor `(0,0.5)`,
  Left=88; root-child canvas slot centered, 1000×100. Это сериализованные
  параметры, не измеренные live geometry/desired size/clipping.
- Blueprint native consumer получает ровно исходный TextBlock_89 и RetainerBox_0
  в своих массивах. Hold material, lock overlay и handheld SetFont(size56) —
  отдельные preserved boundaries. Добавление sibling не доказывает сохранение
  panel desired size или individual visibility; полный final writer не установлен.
- [Точный отчёт, дерево, хэши и воспроизводимый offline audit](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/e6592818452b007159be9d84a407e865483f5b11/research/interaction-prompts-owner/VISUAL_READINESS.md).
  Runtime/mod-specific implementation и визуальная приёмка не заявляются.

## R-007 — Aphes.ttf и граница между глифами, UFont и языком

- **Статус/источник:** техническая offline-проверка stock ресурсов того же baseline,
  [точный отчёт и SHA](https://github.com/NikichMods/AsterigosMods/blob/207489fed083acb54e0fa5acae75990bfcdd5a57/ImmersiveInteractionPrompts/docs/VERIFIED_RUNTIME.md).
  Не проверено новое отображение в игре.
- `Genesis/Content/Fonts/Aphes.ttf`, SHA256
  `443fcae9fd3d3588a259d6c7746145b6c2a343cfb0c2f572fb330234c0f90ed0`:
  внутреннее имя Aphes Medium; 72 glyphs, 97 cmap entries (U+0000, U+001D,
  U+0020..U+007E); lowercase и uppercase ссылаются на одинаковые глифы.
  A–Z/0–9 имеют непустые outlines и просмотрены на offline contact sheet.
  Кириллицы, греческого Unicode и PUA в cmap нет. Mapping пунктуации не
  гарантирует непустой outline.
- В stock inventory нет отдельного Aphes UFont/FontFace uasset. Наличие TTF
  не доказывает пригодность в UMG FontObject. `/Game/Fonts/Umg_Font` — другой
  runtime composite: Default=EBGaramond-SemiBold, Desc=FiraSans-Regular,
  fallback=традиционный китайский Noto, culture subfaces ja/zh-Hans.
  Нельзя считать этот composite активным шрифтом любого TextBlock по соседству.
- **Coherent, статический путь (2026-10-28):** stock `uiresources/commonObj/basic.css`
  задаёт `.AphesText { font-family: var(--font-family-aphes), monospace; }`,
  переменная равна `Aphes, EBGBold`. Названия локаций в Hud.html используют этот
  класс; `ShowPlaceName` отдельно получает `stageAphes`/`placeAphes` и локализованные
  имена. Глифы шрифта и содержание этих строк — разные источники. Это позволяет
  исследовать штатный CSS-путь вместо изобретения alias/font URL, но не доказывает
  применимость к UMG или результат конкретного нового мода в игре.
  [Источник и границы проверки](https://github.com/NikichMods/SoulsEnemyHUD/blob/9f2cf4e6e75b665c3f1cb5573acf701583c98a1c/docs/BOSS_IMPLEMENTATION.md).
- **Runtime-подтверждение Coherent (2026-10-28):** на Steam 1.09/build12996889
  пользователь принял отображение новой надписи с тем же `.AphesText` в штатном
  Hud.html; скриншот показывает требуемые глифы над локализованным именем.
  [Точное свидетельство и границы](https://github.com/NikichMods/SoulsEnemyHUD/blob/19286fe213653ca8caeb34399b88c137ca87edb5/docs/VERIFIED_RUNTIME.md).
  Это подтверждает применение штатного класса в этом Coherent-контексте,
  но не UMG FontObject, не перевод слов и не причину сбоя отдельного alias
  в прежней реализации. Успешная браузерная подстановка шрифта сама по себе
  не доказывает его загрузку в игре.
- **Граница:** cmap кодирует символы, не перевод слов. В исследованных
  prompt/string/font ресурсах словарь действий на языке Афеса не найден;
  это не аудит всего игрового лора. Транслитерация/декоративная маркировка —
  отдельный продуктовый выбор. Font rendering, semantic translation и право
  публичной дистрибуции — три разные проверки; извлечённый TTF не опубликован.

### R-007 UMG readiness-дополнение, 2026-10-28

- Installed-build UHT подтверждает UTextBlock::SetFont(FSlateFontInfo),
  UFont.CompositeFont и reflected FFontData.FontFilename, но у FSlateFontInfo
  нет reflected filename member. UE4SS 83330635 marshals struct fields, а не
  C++ filename constructor FFontData. Наличие этих полей не доказывает корректную
  инициализацию полного font state или чтение packaged Aphes.ttf.
- UFontFace.SourceFilename не заменяет импорт raw payload. Стандартный
  [UE4.27 UMG font path](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-fonts-with-umg?application_version=4.27)
  использует runtime composite/font-face assets. Валидный Aphes UMG font object
  ещё не получен; это пробел доказательств, не невозможность UMG-рендера.
  Generated UHT `.cpp` bodies — stubs, не engine implementations. Точные поля
  проверять по installed dump; страницы API даже с version=4.27 могут содержать
  новые объявления. [Источник и границы](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/e6592818452b007159be9d84a407e865483f5b11/research/interaction-prompts-owner/VISUAL_READINESS.md).
  Ни TTF, ни cooked assets не опубликованы; новый runtime не выполнялся.

## R-008 — UE4SS callback boundaries for observation (2026-10-28)

- **Evidence level:** source inspection of installed baseline revision `83330635`,
  not a new runtime test. Steam 1.09/build12996889, UE4.27; installed UE4SS DLL
  SHA256 `ce5e3c46a920cc00b2817ac61e438c9162830290379d0b5b9c4dfc2b29459290`.
- [Version-pinned LuaMod.cpp](https://github.com/UE4SS-RE/RE-UE4SS/blob/83330635/UE4SS/src/Mod/LuaMod.cpp):
  RegisterHook selects native/script handling from the loaded UFunction pointer
  and flags. On UE4.27 the script path runs after ProcessLocalScriptFunction;
  it skips ReturnValue during input marshalling. A Blueprint no-input bool query
  therefore does not deliver that original result as a Lua callback argument.
  A non-nil callback return can override the host result; observers return nil.
- Hook parameters are marshalled before entering the Lua callback, including
  out/reference address resolution. A context-only Lua callback does not bypass
  that prerequisite; inspect the exact reflected signature before hooking it.
- **Consumers/limits:** relevant to interaction-prompt observation and future
  Asterigos UE4SS diagnostics. This establishes source contract, not successful
  registration of a game function, support for all property shapes or final UI
  writer ownership. Other UE4SS revisions require separate checks. Mod-specific
  missing metadata and procedure remain in the
  [ImmersiveInteractionPrompts report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/afb5b4500b460734d6f76542926ffb8593d42231/research/interaction-prompts-owner/README.md).

### R-008 object/struct lifetime clarification, 2026-10-28

- Source-only, same revision `83330635`: LuaMod.cpp 1916–2023 supplies
  StaticConstructObject with UClass/Outer/name/flags, not a reflected UMG factory.
  LuaUObject.cpp 754–830 supports same-type ScriptStructWrapper CopyCompleteValue;
  a synchronous copy of a live property differs from fabricating a partial struct
  or retaining a borrowed native parameter for later use.
- LuaUObject.hpp 761–775 checks non-null/sentinel, global object-map membership
  and IsUnreachable. LuaUObject.cpp 25–92 contains map/deletion-listener handling.
  This is neither a strong GC root nor a serial-number weak handle. Outer or a
  Lua wrapper must not be assumed to keep an object alive. Fresh exact lookup and
  identity checks avoid reliance on retained raw wrappers, but are not a complete
  engine-lifetime proof.
- [Source contract review and limits](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/c9ecc0064eae59e8f6fc2a6557a1721b28d80534/research/interaction-prompts-owner/ui-child/README.md).
  hpp SHA256 `1b011de1b90390abd02c9f11ebf9da7eb7b55b7d8b2c3f7040f749bd4649e157`.
  No new successful UMG construction/render/GC result is claimed; source-only
  prototype readiness belongs to the mod, not shared runtime acceptance.

## R-009 — Lua metadata completeness limit in UE4SS 83330635 (2026-10-28)

- **Evidence:** source inspection for the same Steam 1.09/build12996889, UE4.27,
  installed UE4SS revision/hash as R-008; no new game execution.
- UFunction exposes GetFunctionFlags; UStruct exposes ForEachProperty;
  XArrayProperty exposes GetInner. However,
  [XProperty](https://github.com/UE4SS-RE/RE-UE4SS/blob/83330635/UE4SS/src/LuaType/LuaXProperty.cpp)
  and its actual RemoteObjectBase/RemoteObject binding hierarchy expose no
  EPropertyFlags getter/predicate. UObject HasAnyFlags is not FProperty access.
  Native HasAnyPropertyFlags calls inside the loader do not make that API
  available to Lua.
- **Consequence:** property enumeration alone cannot certify Parm, OutParm,
  ReferenceParm or ReturnParm, nor separate Blueprint locals from parameters.
  Do not infer an exact native signature from property order/offsets or another
  function's flags. A complete metadata reader needs another verified typed
  access path or an existing trustworthy declaration. This is a binding limit,
  not evidence that reflection reads themselves crashed or mutated game state.
- **One-shot source contract:** LuaMod ExecuteInGameThread queues a simple
  action; process_simple_actions removes it from the queue and unrefs the callback
  after execution, with reentrant execution deferred. Scheduler availability and
  naturally loaded target objects still require runtime evidence. Do not confuse
  that shared engine dispatch with a need to hook the inspected game function.
- **Consumers/limits:** interaction metadata capture and future exact-signature
  diagnostics on this UE4SS revision. See the owning mod's
  [metadata API report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/46d220083263dd12017f071b4af9b48d9d710bf1/research/interaction-prompts-owner/METADATA_CAPTURE.md);
  this does not establish actual Genesis signatures or final UI writers, and
  does not generalize to newer UE4SS bindings.

## R-010 — Built-in UHT declaration evidence (2026-10-28)

- **Verified environment:** Steam 1.09/build12996889 accepted EXE hash,
  UE4.27, UE4SS 83330635. A separately authorized user-operated session completed
  built-in Generate UHT Compatible Headers; generated native-class declarations
  were obtained without a custom C++ mod or Lua FProperty flag reader.
- `/Script/Genesis.ExtendedUserWidget:SetupTipMessage2` is emitted as void with
  ordered inputs `TArray<UTextBlock*> utexts`, `TArray<UPanelWidget*> imgContainers`,
  `ETipWidgetMode display_mode`, `TArray<FText> msgs`,
  `TArray<UTexture2D*> base_textures`, all without const or reference qualifiers.
  This agrees with the TipHUD call at 713 in R-006. It closes declaration access,
  not final-writer or hook-runtime evidence.
- [UEHeaderGenerator parameter generation](https://github.com/UE4SS-RE/RE-UE4SS/blob/83330635/UE4SS/src/SDKGenerator/UEHeaderGenerator.cpp#L3292)
  filters Parm/non-ReturnParm and emits const/& from ConstParm and
  ReferenceParm/OutParm. Its forced string const-reference exception does not
  apply to these array/enum parameters. Therefore the selected parameter flags
  can be inferred here, while full numeric masks remain uncaptured.
- **Limits:** UHT generated macros may be normalized by settings, notably
  MakeAllFunctionsBlueprintCallable; BlueprintCallable text alone does not prove
  the original bit or native hookability. Generated .cpp bodies are stubs, not
  game implementation. UHT removes the old UHTHeaderDump first; preserve any
  existing output. The distinct CXX generator's asset load/free behavior is not
  interchangeable with UHT. This broad built-in generation requires its own
  authorized scope; it does not invalidate the earlier narrow Lua API limit.
- Exact signature, source mapping, output/environment hashes, actual activation
  count and cleanup belong in the
  [owning UHT report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/946fbf61dc40e7018f5a5c5047e9e904f16a745c/research/interaction-prompts-owner/UHT_SIGNATURE.md).

## R-011 — TArray indexed reads can mutate storage in UE4SS 83330635 (2026-10-28)

- **Source-level fact, not a new runtime test:** installed Asterigos baseline
  UE4SS 83330635, same DLL identity as R-008. In
  [LuaTArray.cpp](https://github.com/UE4SS-RE/RE-UE4SS/blob/83330635/UE4SS/src/LuaType/LuaTArray.cpp),
  indexed Get uses prepare_to_handle; if the converted index is beyond Num or the
  array is empty, it calls AddZeroed before reading. Operation::Get does not
  exempt that branch. An apparent Lua read is therefore not inherently read-only.
- Observers must validate count and enforce bounded in-range indexes in the same
  valid callback; never use read-until-nil/ipairs on borrowed TArray userdata.
  TArray IsValid checks a non-null header pointer, not frame/storage lifetime.
  Wrappers and element addresses must not outlive the memory they reference.
- ForEach has an unresolved source comment about stack corruption with large
  arrays. This is not evidence of a crash on this game's small arrays; it does
  not by itself certify or disprove a specific observer route. Guarded indexing
  still requires valid native parameter memory before any element read.
- Relevant to interaction and future UE4SS diagnostics, not production behavior
  or a new compatibility claim. Snapshot SHA256
  `c5c4be252fc9a026152a70d0964d5a7dc4889b68f609e37baa8eab216d80bbac`;
  mod-specific native-frame evidence gap remains in the
  [observer readiness report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/d52939b65fe5b31d52738f2dcabe9ac6d7edbda8/research/interaction-prompts-owner/OBSERVER_READINESS.md).

## Что не скопировано сюда

Не импортированы все исходники, журналы экспериментов и мод-специфические ID из подпроектов. При первом повторном использовании выявить конкретный применимый факт, сохранить доказательства и перенести только нужный переиспользуемый вывод в этот документ.
