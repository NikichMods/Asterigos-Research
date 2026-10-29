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

- **Resource-free divider runtime case:** same Steam/UE4.27/UE4SS 83330635
  baseline, one Open prompt. New UImage Brush initially DrawAs=3(Image),
  ResourceObject absent, ResourceName=None, dynamic=false, has_uobject=false.
  Explicit Box/NoImage/zero-margin/specified white 140×2 setup and readback
  succeeded; user screenshot shows the divider beneath OPEN with original
  action/input hint present. No texture/material/imported resource required.
  Both own children RemoveChild/GetParent checks passed; CLEANED reason=user,
  remaining=0 after ~5.5s, not timeout. User was uncertain about visually seeing
  disappearance; native pair detachment is established, not invented visual
  confirmation. [Source, log/image hashes and limits](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/8db6e87472257744e83edb29800515ab0870060e/research/interaction-prompts-owner/DIVIDER_READINESS.md).
  Probe disabled; no GC/full-lifecycle/final-design/Aphes acceptance. These
  defaults are observed for this constructed UImage, not every FSlateBrush.

- **Узкое runtime-подтверждение добавления UI:** Steam 1.09/build12996889,
  UE4.27, UE4SS 83330635; один TipHUD с «Открыть». StaticConstructObject создал
  отдельный transient TextBlock с Outer=WidgetTree; synchronous typed Font copy
  и CanvasPanel_6.AddChildToCanvas/свой slot дошли до ATTACHED. RemoveChild
  вернул true, GetParent после него отсутствовал, CLEANED reason=user.
  Пользователь увидел OPEN и его удаление, исходные действие/иконка остались
  на месте. Это один render/manual detach case, не полный lifecycle/GC proof.
  [Точные source/log hashes и границы](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/a89ab33358df4e905f97942dc7deb64d59b5f16a/research/interaction-prompts-owner/ui-child/README.md).
  Probe отключён после выхода. Не подтверждены Aphes/divider, timeout/смена цели,
  другие действия/устройства, DPI/длинные строки или immediate UObject destruction.

- Следующий диагностический прогон установил конкретные live slot значения:
  Position table `(38,0)`, Alignment table `(0,0.5)` у естественного Open prompt.
  Designer X=88 не подходит как безусловный live invariant; Y/alignment совпали.
  Писатель X и применимость к другим actions/devices не установлены. Новый ребёнок
  снова не создавался из-за старого guard. [Точное свидетельство и log hash](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/6e55c184dded60eb1e2a1c0f8753f37bdeda18fe/research/interaction-prompts-owner/ui-child/README.md).

- Позднейший узкий UI-прогон на том же baseline прошёл live проверки
  TextBlock_89 → CanvasPanel_6 → WidgetTree → текущий TipHUD, три canvas children
  и Font.Size=22. Остановился до создания нового ребёнка на строгом условии
  Position/Alignment; сами значения/формат getters не были залогированы.
  Это не доказательство native изменения координат или работоспособности
  создания/рендера/RemoveChild. [Точный результат](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/ea85872892090c8bd13fdc09a275b14ec9bfa94d/research/interaction-prompts-owner/ui-child/README.md).

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

### R-007 Dynamic TTF без cooker — source-only, 2026-10-28

- Ограниченный offline разбор 17 известных game Fonts / EngineFonts packages
  подтвердил 5 UFont и 12 FontFace. Garamond используется Umg_Font и HudUmg
  DamageDigits0…19; пять игровых FontFace входят в Umg_Font. Независимого
  in-place donor не установлено; отсутствие ссылок на engine Font в ограниченном
  корпусе JSON не доказывает отсутствие native/style/soft-reference потребителей.
- Upstream UE4.27.2 source (mirror AlexMercer-MA/UnrealEngine-4.27,
  `306a7e9f7fb53f14c2cb3bfd3854ad906710a17e`, Build.version4.27.2)
  подтверждает runtime FString filename constructor FSlateFontInfo и legacy cache,
  создающий FStandaloneCompositeFont с LazyLoad. Нативный CompositeFont shared
  pointer не UPROPERTY и отсутствует в installed UHT; Lua SetFont marshalling
  не вызывает такой конструктор. Это engine capability, не подтверждённый ABI
  Asterigos или готовый UE4SS C++-мост. Обычная engine plugin сборка не равна
  загружаемой в Shipping игру UE4SS DLL.
- **Новый source-supported кандидат, не runtime-факт:** LuaMod.cpp1916–2023
  pinned UE4SS83330635 принимает Template в StaticConstructObject. UE4.27 native
  InitProperties + struct/array copy поддерживают создание собственной копии UFont.
  Installed UHT exposes FFontData.FontFilename и FontFaceAsset; native FFontData
  читает filename, если face asset отсутствует. Копирование маленького Runtime
  UFont с существующим typeface entry позволяет исследовать собственный provider,
  обходя конструирование native shared pointer и новой typeface array из Lua.
- Это не подмена donor object/file. До записи в копию необходимо доказать её
  изоляцию в реальной игре; optional Template через installed binding, decoder,
  rendering/cache/lifetime ещё не проверены. Новый payload/DLL не создавался,
  runtime не выполнялся. Исходники, хэши, границы dumpbin-проверки и сравнение
  альтернатив: [IIP dynamic font report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/872929ce07ef19d504c14a977eb6cfe04cf17ccc/research/interaction-prompts-owner/DYNAMIC_FONT_READINESS.md).
  Шрифты, raw JSON и engine source snapshots остаются локальными.

### R-007 Read-only native storage identity — UE4SS83330635 source

- LuaTArray.cpp13–17,79–109 хранит remote FScriptArray из params.data;
  GetArrayAddress возвращает адрес native header, GetArrayDataAddress — GetData.
  Это адреса хранилища, не identity Lua wrappers. Num/Max можно читать без индекса;
  indexed Get требует проверки границ заранее, поскольку способен AddZeroed.
- LuaUObject.cpp748–826 ordinary StructProperty Get помещает params.data в
  ScriptStructWrapper.start_of_struct. LuaUScriptStruct.cpp86–94 возвращает его
  через GetStructAddress; nested Get получает адрес поля в той же структуре.
  GetPropertyAddress возвращает общую metadata, непригодную для доказательства
  независимости значений. GetNonTrivialLocal использует другой table path.
- Эти source-контракты дают средство сравнения native array/struct storage без
  изменения данных и без чтения raw memory по возвращённым числам. Разные адреса
  не доказывают независимость скрытых FString buffers/всех heap allocations;
  FontFace UObject reference в копии может намеренно оставаться общим.
- [Узкий IIP creation/storage probe](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/6c75063300c9aa84ec946f64ca108cc4d686e1e3/research/interaction-prompts-owner/font-template/README.md)
  подготовлен с count-before-index и donor post-check. Десять новых mock groups
  проверяют control flow; в игре не запускался. Никакой runtime deep-copy,
  FFontData-mutation, font rendering или GC acceptance здесь не добавлено.

### R-007 Runtime Template creation/storage result — 2026-10-28

- Один пользовательский запуск на established Steam1.09/build12996889,
  UE4.27/UE4SS83330635 подтвердил StaticConstructObject с Template argument8
  для naturally loaded `/Game/Fonts/EBGaramond-SemiBold_Font`.
  Создан отдельный RF_Transient UFont; passed guards: Runtime cache, one Default
  entry, empty fallback/subtypes, matching selected FontData values.
- Native array header/data, entry/FontData и остальные inspected struct/header
  addresses различались у донора и копии. Общая FontFace reference ожидаема.
  DONOR_UNCHANGED относится к выбранному snapshot, не ко всем скрытым данным.
  Один CREATE_BEGIN/CREATED завершён DONE result=STORAGE_CHECK_PASS;
  font_data_writes=0, ui_writes=0, gc_forced=false. Probe отключён после выхода.
- Source `6c75063300c9aa84ec946f64ca108cc4d686e1e3`, installed SHA256
  `f771481709412b2bdf5bc9f54e262b1d8cd7f69db23b1e9c72b0dca7b3b9cc7d`
  совпал после закрытия. Лог строки697–702 /16:32:16 МСК, SHA256
  `c8c06876154954e94d507e7af1ebc683e37830ea42329564329d46a935f22453`.
  [Точный результат и границы](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/4f660f41fd6564cfa0197b15e069ea23440a1089/research/interaction-prompts-owner/font-template/README.md).
- Это runtime-свидетельство construction/observed storage, не full deep copy,
  FString backing independence, setter safety, Aphes decoding/render или GC
  acceptance. Не переносить результат на другие классы/Templates/версии без
  проверки. Raw log остаётся локальным; никаких шрифтовых payload не опубликовано.

### R-007 Setter/ownership follow-up — source/static only, 2026-10-28

- UE4SS83330635 LuaUScriptStruct.cpp167–218 NewIndex вычисляет адрес поля
  существующего native struct и вызывает Operation::Set. LuaUObject.cpp445–480
  object Set с nil пишет nullptr в этот slot, не меняя целевой UObject.
- LuaUObject.cpp1758–1790 Lua-string branch создаёт temporary FString и
  copy-assigns destination. В exact installed DLL SHA256
  `ce5e3c46a920cc00b2817ac61e438c9162830290379d0b5b9c4dfc2b29459290`
  bounded static dumpbin inspection экспортированных FString constructor/copy/
  assignment и resize/copy paths подтвердил owning destination character copy.
  Это не borrowed temporary char pointer. FString-userdata SetCharArray branch
  этим выводом не сертифицирован; числовые setters сами не проверяют enum range.
- UE4.27.2 source Class.cpp/UnrealString.h/Array.h показывает native/non-POD
  property copying и owning FString Data TArray. В совокупности с раздельным
  native storage это позволяет готовить запись в существующий FFontData частной
  копии, но не разрешает модификацию общих font assets или raw offsets.
- FontInfo.FontObject — отражённая reference и часть AddReferencedObjects.
  Для own TextBlock font удерживается этой ссылкой; Outer/Lua wrappers не roots.
  После detach typed native SetFont(stock read) может заменить own reference и
  Slate state вместе. No forced GC/immediate collection claim; реальная очистка,
  font-cache lifetime и Aphes decoding/render ещё не проверены.
- [Точный setter review и исходник](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/94770eddf8ec9c74d75fa4ca145c4e81d1940fe4/research/interaction-prompts-owner/aphes-text/README.md):
  отдельный OPEN→READ probe, 10 новых offline mock groups pass. Не установлен/
  не запускался. Raw tool/source evidence, TTF и config остаются локальными.

### R-007 Native Aphes UMG render/change — bounded runtime PASS, 2026-10-28

- Established Steam1.09/build12996889 / UE4.27 / UE4SS83330635: exact source
  `94770eddf8ec9c74d75fa4ca145c4e81d1940fe4`, SHA256
  `9042d9daebe65681c80e445a963e15ec6bd0b4c1a68d9825cb1b62284b8a6306`,
  config/local original Aphes.ttf and DLL identities matched after exit.
- Один natural Open prompt: private RF_Transient UFont constructed with Template,
  только его FFontData configured with nil face/absolute TTF/LazyLoad, native
  UTextBlock FontObject bound. FONT_CONFIGURED/FONT_BOUND/DONOR_UNCHANGED at21:07:54;
  OPEN→READ same_element=true at21:08:11; оба own children REMOVED, FONT_RELEASED,
  CLEANED reason=user own_children_remaining=0 at21:08:21, до30s timeout. Ошибок probe нет.
- Пользователь подтвердил визуальный успех; один кадр2560×1440 показывает Aphes
  glyph shapes, original «Открыть»/F и divider. Это техническое принятие dynamic
  local TTF→private UFont→native UMG в одном случае, без editor/cook/PNG/new DLL,
  не финальная композиция или семантический перевод OPEN/READ на язык Афеса.
- Raw log SHA256 `b83f9bbe387b4894b07938d121913ca2d957a4fc08b2008262481847c6313c7c`;
  frame SHA256 `b8a9397ee1bb42992d207a328495585cf8ae8a63464103305fa5d20a7340e6de`.
  [Полный результат и границы](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/db09e4d79daacb44a225b2c7854c1ae3ebb8970e/research/interaction-prompts-owner/aphes-text/README.md).
- Probe disabled after closed session. User exit Alt+F4; manual cleanup logged
  before exit, normal-menu shutdown and full GC/cache deallocation не приняты.
  Широкий lifecycle/другие prompts/DPI/input coverage и final design остаются
  отдельными gates. Raw log/frame, TTF и config остаются локальными/ignored.

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

- Additional pinned-source contract: LuaUScriptStruct.cpp 63–65 routes field
  __newindex to Operation::Set; 163–230 resolves reflected fields/type handlers.
  Nested owned structs can therefore be configured through their actual fields;
  this does not establish native defaults or substitute for constructors.
  LuaUObject.cpp 229–232 excludes ConstParm from OutParm table/writeback handling,
  relevant to reflected const-reference inputs such as UImage.SetBrush.
  Source snapshot SHA256 `d7896437d23331d799a1cdce0225df7cd03cc02e3052693c6d646e0a4d09d277`.
  [Exact review and limits](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/180ffbfa8d0ba6e0e6c5da8aa8efe9f818152f3c/research/interaction-prompts-owner/DIVIDER_READINESS.md).
  At that source-review checkpoint Image Box rendering/defaults were unobserved;
  subsequent limited runtime evidence is in R-006. A source-ready prototype alone
  is not runtime proof. No game resources were published.

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
  That source review alone did not prove UMG runtime behavior. The subsequent
  one-label construction/render/manual-detach case is recorded in R-006;
  GC and broader lifecycle remain unproved. Mod readiness belongs to the mod.

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

## R-012 — Runtime PNG import contract, source-only (2026-10-28)

- **Offline contract, no new game run:** retained UHT output for Steam 1.09 /
  build12996889, UE4.27, UE4SS83330635 declares runtime Engine static
  `UTexture2D* ImportFileAsTexture2D(UObject* WorldContextObject, const FString& Filename)`
  on KismetRenderingLibrary. UMG Image declares
  `SetBrushFromTexture(UTexture2D* Texture, bool bMatchSize)` and
  `SetBrushResourceObject(UObject* ResourceObject)`. Generated cpp bodies are
  stubs, not evidence of native implementation success/failure.
- Pinned UE4SS LuaMod.cpp1245–1275 / LuaUObject.cpp373–386 returns an unbound
  UFunction from exact StaticFindObject. LuaUObject.cpp93–145 consumes its
  explicit calling UObject before reflected parameters; 210–280 marshals then
  ProcessEvent, 342–361 returns the object result. Correct source-confirmed form:
  `fn(cdo, current_widget, absolute_filename)`, where fn is
  `/Script/Engine.KismetRenderingLibrary:ImportFileAsTexture2D` and cdo is
  `/Script/Engine.Default__KismetRenderingLibrary`. CDO and WorldContextObject
  are separate arguments. No forced lookup/load is needed by the prepared test.
- Reflected UImage.Brush → FSlateBrush.ResourceObject provides the engine object
  reference for the imported texture after binding. Lua wrappers and Outer alone
  are not GC roots. Detached owned Image can release its resource reference via
  the nil-capable object pusher and SetBrushResourceObject(nil); native operation
  and rendering still require runtime confirmation. This does not prove immediate
  texture/GPU deallocation or certify GC/full HUD lifecycle.
- Original research source, API provenance, exact local font/PNG hashes and
  failure/one-import/cleanup policy are in the
  [IIP PNG report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/135d0ed33c2b697051b63e079a6190da862e8862/research/interaction-prompts-owner/png-header/README.md).
  Thirty mock/offline tests pass there. No PNG import, alpha, GPU upload,
  Aphes-in-UMG render or texture unbind is runtime-verified. No game installation
  occurred. The original TTF and one derived transparent PNG remain local/ignored;
  neither is distributed. This route tests a raster image, not a cooked UFont,
  and does not establish Aphes translation or a finished Nexus setup.

### R-012 runtime follow-up — stock ornament, 2026-10-29

A separate bounded IIP run now confirms the native import path in this exact
baseline, using the stock Coherent ornament rather than the untested glyph PNG.
Exact unchanged source `996ee47f06f6cf88d0440d235d13acb9daf05284`; main SHA256
`3c8eb86e695b716616d89f143dd6d2f66ce4ff40c98db1aad50d99738f604abe`, helper
`332ffbdda46c7ac9edc245dfc42913529e8ee9f20752a9ee9e077a367525ecc6`.
Config/TTF/PNG/DLL matched the installation manifest; baseline UE4SS83330635.

00:33:10 Moscow one ImportFileAsTexture2D→transient Texture2D570×16→UImage brush
binding, two own children attached under TipHUD.CanvasPanel_6. User reports visual
success and supplies a2560×1440 frame showing the ornament between native Aphes
and stock localized text.00:33:27 state-change cleanup restores stock text's
render translation, detaches both children and clears image/font resource refs;
CLEANED remaining0. Native getter readback confirms restoration, not a separate
post-cleanup visual frame. Later manual cleanups were empty; no READ/manual
removal while attached in this run. No full GC/GPU deallocation, general PNG
compatibility, lifecycle or public distribution-rights claim. Probe disabled
with host closed; game resources remain local.

Raw log SHA256 `04e8e943db379971e5b40ffcdb16ccf931fea03e020b32e980f58d3169451cbb`;
user frame `a6374d4c45bab919ff4e63b516b43181862c31e90568e0b1cc47389fe22ff33b`.
[Owning result and limits](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/17c2080b4baa7dfca09f18fdbd9d1e64406720cd/research/interaction-prompts-owner/composition/README.md).
Original raster-glyph PNG remains untested; native Aphes font render has its
separate R-007 evidence. Final IIP design/production acceptance belongs to IIP.

## R-013 — Coherent location-title ornament, source-only (2026-10-28)

Installed Steam1.09/build12996889 base containers contain
`Genesis/Content/uiresources/Hud.html`: PlaceName DOM5676–5682 includes an Ani2 img
referencing `images/menu/ele_loctitleline.png`. Active CSS297–301 gives absolute
width40.5vw, height1.48vh and bottom3.88vh inside a centered column. Earlier
background/pseudo-element alternatives are commented out. The actual separator
is one transparent 570×16 RGBA PNG, alpha0–228, with symmetric end ornaments and
a central motif. It is not a CSS border/font glyph.

SHA256: Hud.html `f3edde51ec5d285ef242968a66f97f8da207cf5ec65b3c229e573f938937c87b`;
commonObj/basic.css `5449a256f3d1b44bd286ecb37592ee25c6136c86671f8cefca47a0480da30a24`;
PNG `901b72602a69a535e6a3d54bce2a13f18d0badc20d59abfca351eec70161e668`.
Only those three entries were extracted with the existing CUE4Parse1.2.2 reader;
raw host bytes remain local/ignored. Exact-basename lookup in retained paths
found no matching uasset/uexp; this does not exclude differently named textures.
Coherent HTML/URL is not a native UMG UTexture2D. Local import under R-012 is a
candidate, not runtime-verified reuse or permission to redistribute game images.

Retained game UHT Widget.h/WidgetTransform.h reflect GetDesiredSize,
SetRenderTranslation(FVector2D), and RenderTransform.Translation. UE4.27 mirror
`306a7e9f7fb53f14c2cb3bfd3854ad906710a17e`, Widget.cpp: GetDesiredSize reads cached
SWidget extent or returns zero; SetRenderTranslation changes only Translation
and updates rendering. This can preserve slot/layout desired size, but neither
cached extent freshness nor absence of native competing transform writes follows.
Opaque FGeometry is unnecessary for this limited approach. Image.cpp's
SetBrushFromTexture also changes the imported texture's residency flags; brush
binding is not a promise of no transient engine activity.

Provenance and limits: [IIP composition report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/996ee47f06f6cf88d0440d235d13acb9daf05284/research/interaction-prompts-owner/composition/README.md).
No game run/install occurred in the original source investigation. The separate
R-012 runtime follow-up above now covers native reuse of this exact stock PNG;
IIP-specific appearance and broader lifecycle acceptance remain owned by IIP.

## R-015 — projected screen components vs viewport HUD root (2026-10-29)

Source/static only, no game run for this investigation. Retained actual UHT output
`Genesis/Private/ExtendedWidgetComponent.cpp` initializes inherited Space=Screen;
base UMG WidgetComponent initializes World. Pinned UEHeaderGenerator.cpp789ff
reads class CDO values to generate constructor assignments; this is a default
snapshot, not recovered C++ implementation. Gameplay method bodies/returns in
these dumps remain stubs. BaseGameEventObject_BP's component template has no
Space override, so inherits the Screen default. Exact live/placed overrides and
native Genesis position writer remain unobserved.

UE4.27 source mirror306a7e9f7fb53f14c2cb3bfd3854ad906710a17e,
SWorldWidgetScreenLayer.cpp92–145, projects SceneComponent.GetComponentLocation
through the player view and converts into layer-local coordinates. Thus Screen
WidgetComponent rendering can still move with its world actor; SetWidgetSpace
alone does not make a contextual prompt viewport-fixed.

Actual UHT UserWidget is Abstract. WidgetBlueprintLibrary.Create(WorldContext,
WidgetType,OwningPlayer), AddToViewport/AddToPlayerScreen and viewport anchor/
position APIs are reflected; UE4.27 ValidateUserWidgetClass rejects Abstract
classes, Initialize constructs/initializes WidgetTree, AddToScreen owns the Slate
layer. Presence of these methods does not justify bypassing initialization or
constructing an arbitrary gameplay UserWidget class without reviewing callbacks.

Existing R-004/005 identifies in-viewport HUDMainUMG_BP_C and gameplay pawn guard.
Retained HUDMainUMG_BP tree has Canvas Main_Panel as its root; the same export
SHA2566c26eb0e427490bf9e55b2d27d2bb81d5bc2dbb1353c2a9645585c99d1f3aeaa is
owned by CombatHUD's evidence. A bounded independent-screen candidate can attach
one owned Canvas subtree here through native AddChildToCanvas, leaving original
children/slots untouched. Root-normalized anchors avoid manual DPI conversion;
host viewport stretch/transforms/margins and lifecycle still bound applicability.
Exact viewport-pixel mapping has not been established by this source review.

Coherent UHT exposes CreateJSEvent, AddString/AddBool and TriggerJSEvent plus
readiness APIs. Retained base Hud.html ShowTipMessage/HideTipMessage bindings
2678–2686 and TipMessage DOM5987ff are commented out. The bounded inspected inputs
do not confirm an active TipHUD→localized-action/current-input bridge. A custom
bridge is possible in principle but would add state/input synchronization and
shared Hud.html integration; absence is not proved for every native subsystem.
The concurrent R-014 action-contract entry below separately confirms native
NotifyCoherentHudInput→UI_Input/UI_InputReleased forwarding; it does not establish
a contextual TipHUD/action-text/current-hint payload.

[Owning IIP comparison/prototype](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/242a5f0a6f5807dd4cdc3953f1f51acc02b0be06/research/interaction-prompts-owner/viewport/README.md)
initially had source/mocks only for root attachment/tuning/hint copy. The separate
R-015 runtime follow-up below exercises that root route with explicit limits;
acceptance is not inferred from prior TipHUD-child/font/import experiments.

### R-015 runtime follow-up — existing HUD-root sample, 2026-10-29

A separate user-run IIP test now exercised the existing viewport-root route.
Exact source `242a5f0a6f5807dd4cdc3953f1f51acc02b0be06`, main SHA256
`3fa10606000f9b44406d1ddf5a798d1634483e6809570c8fc9b242ed12da63fa`, helper
`e2e83b83e2a8b4d1efdef5b01b7125383de99d784f15319ecbbc5ce89bc59620`;
private config/TTF/PNG/DLL and18 previous mod/loader files matched after exit.

02:28:05 Moscow own Canvas with five children attached under in-viewport
HUDMainUMG.Main_Panel, initial normalized anchors(.50,.78), viewport2560×1440.
User reports the screen sample worked apart from arrow movement and supplies one
frame showing the original actor-relative prompt plus the lower-screen sample,
including the copied current F hint alongside localized text. The user's report
supports stability during the camera check; a single still does not establish
that by itself.02:30:43 CLEANED reason=user remaining0: five own children detached,
image resource refs cleared, private font released and own empty Canvas detached.
No probe failures. No full lifecycle/device-switch/remapping/GC/other-resolution
acceptance. Probe disabled after closed-host inspection; host assets/log/frame
remain local. This result does not transfer production behavior between mods.

POSITION move/reset/commit records are absent. User saw camera motion during
arrow attempts, not sample movement. This is an unsuccessful control subtest,
not proof of UE4SS key interception/routing cause or a universal arrow-key defect.
Home reset and F7 commit remain unverified. The project's user chose the initial
position directly; that product decision and future layout stay owned by IIP.

Log SHA256 `1f2f787adc0d82bfcdc005981c6ed513c0b218dafb90eb1bdefc0c37bdeb3096`;
frame SHA256 `f49ecdafb7d1b635fa8f36b57704c4d1497bc0051ca052ba36b38f3f1daa2b07`.
[Owning result/limits](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/1320ad57665124f18b61244b10c5c7c6980388ea/research/interaction-prompts-owner/viewport/README.md).

## R-016 — World WidgetComponent renderer/depth/lifecycle limits (2026-10-29)

Source/static only on Steam1.09/build12996889, UE4.27, UE4SS83330635;
installed DLL still SHA256ce5e3c46a920cc00b2817ac61e438c9162830290379d0b5b9c4dfc2b29459290.
No game run, installation or game/settings/save/other-mod writes in this study.

- R-015's reflected ExtendedWidgetComponent CDO Screen value and projected
  component-world location remain distinct from World geometry. Actual UHT
  EWidgetSpace is World0/Screen1. CDO/serialized inheritance does not certify
  every placed live component or recover Genesis' native transform writer.
- UE4.27 mirror306a7e9f7fb53f14c2cb3bfd3854ad906710a17e WidgetComponent.h337:
  SetWidgetSpace only assigns Space. OnRegister861–895 allocates FWidgetRenderer
  for non-Screen mode; Tick1139ff calls UpdateWidget but does not perform this
  allocation. DrawWidgetToRenderTarget1259ff returns if renderer is null.
  Therefore changing an already registered Screen component's enum is not a
  verified World transition. Cached screen-layer/window/material/renderer state
  also prevents declaring exact rollback from property readbacks alone.
- World scene proxy draws a local YZ plane (+X normal) from render-target size
  and Pivot through component LocalToWorld, in SDPG_World. It is not automatic
  billboarding. Physical size depends on component scale; perspective size
  changes with depth. Two-sided rendering does not keep text camera-facing.
- Exactly seven installed base EngineMaterials/Widget3DPassThrough packages were
  read with existing CUE4Parse1.2.2. All six instances resolve to Unlit base
  Widget3DPassThrough (Translucent), with Masked/Opaque/Translucent and two-sided
  overrides. Base lacks a serialized disable-depth=true; actual UMaterial CDO
  sets bDisableDepthTest=false. This is static default/asset evidence, not active
  material/shader/Hilda occlusion measurement. GetMaterial(0) gives override
  precedence; GetBaseMaterial and bDisableDepthTest are reflected read candidates.
  No shared base-material mutation or CustomDepth/post-process is required by
  this evidence. Ordinary scene depth can also admit walls/chests/masked foliage;
  it is not a character-only mask.
- AddComponentByClass/FinishAddComponent and K2_DestroyComponent are reflected.
  Engine ActorConstruction creates actor ownership/BlueprintCreatedComponents,
  optionally attaches/promotes root and registers; AddComponentByClass has no
  RF_Transient parameter. A candidate can instead investigate explicit transient
  native allocation with actor Outer + manual FinishAddComponent, configure own
  World/no-collision/no-focus before registration, and self/owner-allowed destroy.
  ActorComponent PostInitProperties/DestroyComponent manage native membership;
  WidgetComponent destruction releases resources via deferred engine cleanup.
  These paths are source-reviewed, not runtime registration/teardown acceptance.
- WidgetComponent.SetWidget needs independent UUserWidget content. CanvasPanel
  is not a UserWidget; TakeWidget uses cached Slate, not a deep clone. Prior
  UFont Template-copy success does not establish UUserWidget/Slate duplication.
  Base UUserWidget is Abstract; game native/BP class callbacks cannot be assumed
  inert from generated UHT stubs. CreateWidget(World) can use GameInstance Outer;
  player ownership is not an inverse pointer to the interaction's WidgetComponent.
  The retained earlier R-006 CanShow identities scope all three TipHUD objects
  under MainGameInstance_BP_C, reinforcing this boundary without a new runtime run.

Material-export SHA256: base50f69813fcca27b7b7b59b41ab9d5bbaadafe0a689851fcd737e29d0b32a2533;
Masked_OneSided b1c11f8f940bce8600d73923ca2fc941fdc995b86246ea1592ca75696bfb2ac1.
Raw exports/source copies remain local/ignored; exact seven-file identities,
source links and mod-specific candidate/blockers live in the
[IIP World-space report](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/21d3fd994002bbcd78293a8833f87c432a73e1b2/research/interaction-prompts-owner/world-space/README.md).
No executable World payload or new rendering acceptance is supplied by this result.

## R-017 — concrete independent UserWidget host candidate (2026-10-29)

### Targeted registry/loading follow-up — offline, not runtime

The installed Genesis/AssetRegistry.bin contains a separate exact generated-class
entry for `/Game/MainContent/Template/UMG/Debug/PostionItemUMG_BP.PostionItemUMG_BP_C`,
AssetClass WidgetBlueprintGeneratedClass, ParentClass/NativeParentClass UMG.UserWidget,
chunk0. The WidgetBlueprint entry is also present; no guessed fallback path is
needed. Raw registry SHA256 `9ba7a655f82877869145de2bc62c2fa76b392fd3d8e7ec67b6b9667784f1ac92`.
Only matching records were inspected with the already cached CUE4Parse1.2.2 reader;
no game execution or tool installation. Raw registry/records remain private.

Pinned UE4SS83330635 LuaMod.cpp3334ff exposes game-thread-only LoadAsset through
AssetRegistry, returning object/was_found/did_load. This establishes a specific
bounded load route, not a successful live load. Static UFunction calls require
the correct explicit receiver; WidgetBlueprintLibrary.Create needs its library
CDO before WorldContextObject/class/OwningPlayer arguments. Own RemoveFromParent
releases viewport ownership; weak Slate liveness can lag, so a pending release
must not be recorded as completed GC or solved by forced collection.
[Exact source/provenance and unrun test boundary](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/a23635b95b029ff8d73dec4f940e1aa54c213d37/research/interaction-prompts-owner/userwidget-host/probe/README.md).
No native loading/Create/render/cleanup PASS is claimed by this addition.

Source/static only, same Steam1.09/build12996889/UE4.27/UE4SS83330635 baseline.
Bounded four-package read from existing base containers found
`/Game/MainContent/Template/UMG/Debug/PostionItemUMG_BP.PostionItemUMG_BP_C`.
It is concrete (no Abstract/Deprecated/NewerVersionExists flags) and directly
inherits UMG.UserWidget, then Widget/Visual/Object. No game native base.

Exactly14 exports: four functions plus own primitive tree/slots/CDO/class.
Construct entry36→10 calls self SetVisibility(Hidden2), then returns; PreConstruct
entry31 returns. CDO script Tick/Paint false; generated-class native-tick-required
field absent with reflected class CDO false. No animations/property/dynamic
bindings or nested UserWidgets. SetText helper only converts/concatenates numbers,
writes own Value_Text and sets self HitTestInvisible3; not a lifecycle callback.
Tree CanvasPanel_0→Main_Panel→{Bg_Image,Value_Text}, three reciprocal slot edges
checked. All callable symbols were enumerated, including EX_CallMath; no gameplay,
input, save/file or actor calls in these Blueprint graphs. Normal engine UMG
bookkeeping is not excluded or misrepresented as zero side effects.

UE4.27 source306a7e9f7fb53f14c2cb3bfd3854ad906710a17e confirms normal
WidgetBlueprintLibrary.Create→NewObject/Initialize, generated-class instancing of
private WidgetTree and per-object Slate caches. This supports an independent
instance without sharing the live TipHUD or mutating class/CDO/other UI. First
OnWidgetRebuilt runs Construct, so setting visibility only before first build
will be overwritten by this class's own Hidden. Do not mistake that for inability
to render; a bounded future test must handle/read back own visibility after build.
Create's host RF_Transactional and the private tree's transient flags differ.

Class residency/loadability, actual Lua Create, native callback timing/Slate and
cleanup have NOT been executed for this candidate. UE4SS LoadAsset exists and
uses AssetRegistry with object/was_found/did_load results; the registry presence
of this exact cooked class path is unverified, so no class-loading recipe is
claimed proven. A separate bounded host-only runtime check is justified by source,
not yet a successful World WidgetComponent test. Other native game classes were
not declared inert from small trees; raw exports remain local/ignored.

Candidate export SHA256
`cac88b3e55127ad3afe5d9d976ed65371afabca9e61586311612aa5db192441f`;
Construct bytecode `d8fd1f391d9cd86eead6ed2e8cda101805a1627cc1626a6dd2dd7bf348c3d682`;
ubergraph `f03baab2734d97441166dc107bb5e7418ef5c7410cb03ac54491c262c2bbe683`.
[Owning report and exact limits](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/8598e4f96da87d536598580fae52be7a938a7c17/research/interaction-prompts-owner/userwidget-host/README.md).
No new payload/game run/install/toolchain or production acceptance.

## Что не скопировано сюда

Не импортированы все исходники, журналы экспериментов и мод-специфические ID из подпроектов. При первом повторном использовании выявить конкретный применимый факт, сохранить доказательства и перенести только нужный переиспользуемый вывод в этот документ.

## R-014 — Native action/mapping contracts and skill topology, static (2026-10-29)

- **Baseline/evidence:** Steam1.09/build12996889, UE4.27; existing UHT declarations and CUE4Parse1.2.2 exports (ReadScriptData=true). No live action callbacks/getters have been exercised by this investigation. [Exact report and evidence SHA-256](https://github.com/NikichMods/CombatHUD/blob/ac526518f2255e30b9726a22e4f00984ee44cd75/docs/VERIFIED_RUNTIME.md#контекстные-игровые-действия-и-skill-targets--статический-этап-2026-10-29).
- **Native semantic contract:** EGenesisInputCommandType has Interact25, SpecialSkill_Switch14 and a separate UI_Enter; EGenesisInputEventType has Pressed0/Released1. GenesisPlayerController declares NotifyActionInput(Command,EventType), onActionInputEvent and native OnActionInputPressed/Released(listener). MainPlayerController_BP has no NotifyActionInput override in the extracted exports. A declared event is not proof of hookability or dispatch; reflected ActionInputMappingListener does not expose a semantic command/key field.
- **Mapping storage shape:** DefaultInputMapping/DefaultInputMappingTypeB use FInputMappingData (ContextType/Command/FKey, modifier/chord/pause flags). FSavedInputMappingData declares saved row/mapping fields, and controller ConfigureAndApply and GameGlobal GetInputMapping/GetInputRemappingData are reflected endpoints. The actual persistence file/manager and current user mapping have not been established. Do not replace semantic actions with shipped default keys or call setters while observing. The two presets assign different default Interact face buttons, while retaining the semantic command.
- **Read candidates:** GenesisPlayerController.GetModifier_SpecialSkill returns bool, GetNowInputControllerType returns input-device enum, GameGlobal.GetInputMode(getAction,getAxis) returns Scene0/UI1/QuestTimeEvent2. These declarations do not establish held/release behavior across menu, device or focus changes. Do not equate Scene or viewport with window focus. Reset/resync of an observer's own held latch requires separate evidence; timeout is not native release proof.
- **Executable forwarding:** MainPlayerController_BP.NotifyCoherentHudInput(cmd,IsPressed) goes via ubergraph entry814 to GTGameHUDBPLocal UI_Input/UI_InputReleased (branches913/927/979). This proves the Coherent/UI forwarding path, not a general gameplay Interact event.
- **Skill families:** HUDSkillUMG_BP has four keyboard skill children under SkillSlotList_Panel plus SP resources; HUDActiveSkillUMG_BP has four pad skill children under Main_Panel and two Image children. Six children are not six proven skill icons. The common HUD_Panel/menu ancestry remains native; visual/gamepad/input switching ownership and independent opacity in new states are unverified. CombatHUD's target selection and runtime acceptance remain owned by CombatHUD.

The original R-014 entry is static; the separately scoped runtime addition below does not turn all contracts into live proof. No diagnostic source, raw UHT dump, game asset, private machine path or settings/save data is published here.

### R-014 runtime startup addition — 2026-10-29

[Exact owning report](https://github.com/NikichMods/CombatHUD/blob/f25fd65ce263382f050cd1727aaa79a6e052d7de/docs/VERIFIED_RUNTIME.md#context-observer01--stopped-before-gameplay-observer02-installed-2026-10-29): UE4SS83330635 log SHA-256 `2e0c91a27b6f0eee98833c80443d05741cc58dd0024dbad07e3c9f8fa75e791f`. Native/base NotifyActionInput and OnActionInputPressed/Released hooks registered; two Released listener callbacks actually ran during startup. No semantic ACTION invocation is established. Primitive getters GetInputMode (0 then1), GetModifier_SpecialSkill (false) and GetNowInputControllerType (4 then1) returned the declared scalar shapes on the early controller.

The observer stopped before any completed GenesisCharacter gameplay snapshot on an invalid assumed direct HUD_Panel widget alias (the pawn type at the exact failed read was not logged); the established previous route is LeftBottom_Panel:GetParent(). Serialized WidgetTree names do not establish Lua fields, and viewport alone is not proof that the gameplay pawn/targets are ready. Optional target diagnostics must not invalidate independent input evidence. The failed observer was removed with the host closed; runtime held/release/remapping/focus behavior, empty-space Interact delivery and visible skill topology remain OPEN. See CombatHUD for corrected diagnostic deployment and later results; registration alone and absence of callbacks after an observer stops are not dispatch proof.

### R-014 gameplay read/binding addition — 2026-10-29

[Observer02 owning evidence and limits](https://github.com/NikichMods/CombatHUD/blob/9860c0ed16167394db5a5db8d7347f266d469723/docs/VERIFIED_RUNTIME.md#observer02-result--live-held-getter-and-skill-binding-interact-open-2026-10-29), log SHA-256 `03cfc94085f8422018181f0b7cd5bcbbbb941fdf7eb20d6e55a441258455e0aa`; same Steam1.09/UE4.27/UE4SS83330635 baseline.

- `GenesisPlayerController:GetModifier_SpecialSkill()` returned live false/true/false under device4 on a GenesisCharacter-owned gameplay HUD, including ~2.74s and ~1.83s true intervals. This demonstrates the semantic native getter on that current preset, not a physical-button constant, remapping round-trip or focus/menu/device-held reset.
- Both HUDSkillUMG_BP_C and HUDActiveSkillUMG_BP_C were bound live with a common CanvasPanel parent read through LeftBottom_Panel:GetParent(). The initial device1 snapshot showed four keyboard slots and four pad slots plus two Images; this is not six visible skill icons or active-pad rendering proof. With accepted CombatHUD active, UI context1 coincided with the common ancestor Visibility2(Hidden); Scene0 restored Visibility3, while both target roots stayed at alpha0.20/Visibility4. Parent hiding remains native; new independent-opacity behavior was not written/tested.
- OnActionInputPressed/Released delivered181 raw listener callbacks; NotifyActionInput registered but delivered zero semantic ACTION callbacks in the recorded interval. Listener pointers cannot identify Interact or distinguish gameplay from UI commands. The tested semantic hook remains unproven as an input source; no general impossibility claim follows.
- Repetitive raw edges exhausted the observer's shared240-line budget; last observer record02:48:09.527 precedes normal host exit02:48:42.878. No terminal budget marker was emitted. Later focus/window actions remain unobserved. Future diagnostics must summarize raw edges separately, reserve state/event budgets and record the stop reason; increasing a global cap alone is not input semantics proof.

The observer was removed with the host closed; exact accepted CombatHUD payload and all19 pre-existing loader files retained hashes. No third probe or production implementation follows automatically. Mod-specific acceptance/readiness and current user button examples remain owned by CombatHUD.

### R-014 scalar virtual-key resolver continuation — source/static only (2026-10-29)

[Owning investigation and exact hashes](https://github.com/NikichMods/CombatHUD/blob/70c62dc63df90d892541339a4c61adfc9f40298e/docs/VERIFIED_RUNTIME.md#current-interact-binding--scalar-resolver-investigation-2026-10-29). Existing UHT declares `GameGlobal:GetKeyByVirtualKey(FName VirtualKeyName) -> FKey`, separately from the default-key getter. Five shipped KeyBinding tables link `K-Interact`, `PC-Joystick-Interact`, `DS4-Interact`, `DS5-Interact`, `X-Interact` to modifiable Game/Game_Normal action `SystemOption.Controller.Interact`; they also match Scene/Interact default input rows. These are virtual configuration identities, not physical keys. Active PC/controller namespace and current saved resolver values remain unverified.

In exact [UE4SS83330635 LuaUObject.cpp](https://github.com/UE4SS-RE/RE-UE4SS/blob/8333063595fa3c49ad307f3b0f732accbb760d25/UE4SS/src/LuaType/LuaUObject.cpp), a scalar struct return uses GetNonTrivialLocal field conversion; a NameProperty is a local value-copy FName. An out array instead fills a Lua table with GetParam element wrappers, so outer-table ownership does not prove deep element ownership. Prefer the scalar route when it supplies the required semantics; no GetInputMapping array call was made by this investigation. Native FKey reconstruction/current-value behavior still needs runtime proof. Exact raw-source SHA-256 `cccf1e63b26fb5a2f009666cf469f19f8eaa655388be462219f3b22b19ae6a0b`.

[UE4.27 WasInputKeyJustPressed](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/GameFramework/APlayerController/WasInputKeyJustPressed?application_version=4.27) is frame-scoped. A50/100ms poll cannot establish reliable event coverage. Sampling from demonstrated native input callbacks is a candidate, but timing, same-frame deduplication, binding namespace/chords and pre/post Scene/UI semantics must be validated before claiming action equivalence. Native pressed/released listeners are wake-ups, not proven semantic commands by themselves. CombatHUD owns the private resolver experiment; this entry records no new live success or production readiness.

### R-014 scalar resolver/native key query — live PC evidence (2026-10-29)

[Exact owning result and readiness limits](https://github.com/NikichMods/CombatHUD/blob/e78d5f734780b826554ebd5369791294b269e018/docs/VERIFIED_RUNTIME.md#observer03-result--current-interact-resolver-and-native-key-queries-pass-2026-10-29); UE4SS session log SHA-256 `e8445de9325cc20c294454f520040b9e96902ed43e3fdddaa8f493a593862cc0`, same Steam1.09/build12996889/UE4.27/UE4SS83330635 baseline.

- GetKeyByVirtualKey actually returned K-Interact→F, PC-Joystick-Interact→Gamepad_FaceButton_Top and DS4/DS5/X-Interact→None. Metadata was platform0(PC), device4 then1, preset0. The gamepad key matched the user's configured Interact and differs from the inspected shipped PC default rows. Device4 is therefore not a valid rule for selecting X-Interact: the active PC mapping namespace in this case was PC-Joystick-Interact.
- The scalar FKey return decoded to a Lua table/local-copy KeyName. Native IsInputKeyDown/WasInputKeyJustPressed accepted an FKey argument reconstructed from that owned name and returned true/true from the demonstrated OnActionInputPressed native post-hook: seven gamepad-positive and two keyboard-positive logged samples, in pre/post Scene0. The user performed empty-space interaction; no interactable actor was required by this read route. Sample count is not a count of physical presses.
- Recorded menu-associated samples had no matching Interact key, with contexts0→1/1→1/1→0. Same-key-in-UI and every input-block condition were not tested; press-context/native UI guards remain necessary. Resolver refresh after UI exit and device change returned unchanged keys across3 generations. A before/after live rebind, chords, other platforms/namespaces and full lost-release/focus semantics are still unverified.
- The specific current-PC resolver/scalar-key-query contract has runtime proof; this is not a universal action-dispatch API or a released HUD feature. Listener Target remains only a wake-up; no fixed physical-key fallback is needed. Observation ended with the normal host exit, without the earlier global-line-budget cutoff. The private observer was removed closed-game and all19 baseline loader files retained hashes. Production/acceptance/readiness for the mod remains owned by CombatHUD.
