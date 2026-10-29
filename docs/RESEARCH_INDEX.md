# Research Index — Asterigos: Curse of the Stars

**2026-10-29 — R-016 source/static:** World WidgetComponent uses normal scene
geometry/depth; stock Widget3DPassThrough family is present. SetWidgetSpace is
assignment-only and does not allocate the World renderer needed at registration.
[Renderer/material/ownership limits](VERIFIED_RUNTIME.md#r-016--world-widgetcomponent-rendererdepthlifecycle-limits-2026-10-29)
record exact sources and defaults; no World payload/game render/rollback acceptance.


**2026-10-29 — R-015 runtime follow-up:** one own Canvas in existing viewport
HUD root rendered and manually cleaned up, original TipHUD preserved. Arrow
tuning did not pass; no input-routing cause inferred. Source distinction between
projected screen components and viewport HUD, Abstract UserWidget restriction
and Coherent bridge limits remain recorded in
[VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md#r-015--projected-screen-components-vs-viewport-hud-root-2026-10-29).
A bounded2560×1440 user-run result now exists; full lifecycle and other resolutions
are unaccepted. Exact source/log/frame and partial input outcome in R-015 follow-up.


**2026-10-29 — R-012 runtime follow-up / R-013:** stock Coherent PNG570×16
успешно импортирован и показан в native TipHUD; state-change cleanup и возврат
translation подтверждены логом. Probe отключён; общий lifecycle/финальный дизайн
не принимаются этим узким результатом.
См. [VERIFIED_RUNTIME.md#r-013](VERIFIED_RUNTIME.md#r-013--coherent-location-title-ornament-source-only-2026-10-28).
Raw игровые изображения остаются локальными; точный runtime source/log/frame
и пределы доказательства приведены в R-012 follow-up.


R-007: **native Aphes OPEN→READ/manual cleanup runtime PASS** в одном natural
Open prompt, с пользовательским визуальным подтверждением. Private transient
UFont/local TTF работает без cooker; probe отключён. Final design, semantic
translation и полный GC/lifecycle не приняты этим ограниченным тестом.

R-007 дополнен source/static review nested setters, owning FString copy в exact
UE4SS83330635 DLL и FontObject retention. Это основа следующего ограниченного
Aphes writer source, не подтверждение реальных записей/глифов/GC в игре.

R-007 дополнен pinned getter-контрактами native array/struct storage addresses:
они позволяют готовить read-only isolation checks, но не доказывают сами по себе
независимость всех внутренних буферов. Последующий IIP run подтвердил отдельный
transient UFont/observed storage через Template; probe отключён. Setters/Aphes
rendering и полная deep-copy independence этим не подтверждены.

Уточнение R-007 в [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md): ограниченная font
inventory и UE4.27 filename-constructor source; private Template UFont copy —
новый непроверенный runtime-кандидат без shared font replacement. Не объявлять
его Aphes-in-UMG acceptance или готовым C++-мостом.

R-012 в [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md) фиксирует source-only контракт
runtime PNG import / explicit CDO call / Image texture reference для UE4SS83330635.
Это готовность узкого теста, без подтверждения импорта/прозрачности/Aphes в UMG.

**Назначение:** короткий вход в общие знания, переиспользуемые несколькими модами NikichMods. Детали и первичные доказательства остаются в документах-владельцах. Дата первоначальной консолидации: **2026-10-27**.

## Репозитории модов после организационного разделения

Общая группа игры содержит шесть самостоятельных репозиториев и этот единственный Research. Актуальные исходники, версии и приёмка принадлежат модам; исходный [AsterigosMods](https://github.com/NikichMods/AsterigosMods) сохранён доступным, без архивирования. Первичные ссылки на старые точные коммиты в исследовательских выводах остаются историческими доказательствами.

| Мод | Источник текущего состояния |
| --- | --- |
| [CombatHUD](https://github.com/NikichMods/CombatHUD) | [docs/STATUS.md](https://github.com/NikichMods/CombatHUD/blob/main/docs/STATUS.md) |
| [SoulsEnemyHUD](https://github.com/NikichMods/SoulsEnemyHUD) | [docs/STATUS.md](https://github.com/NikichMods/SoulsEnemyHUD/blob/main/docs/STATUS.md) |
| [SubtitleSoftEdges](https://github.com/NikichMods/SubtitleSoftEdges) | [docs/STATUS.md](https://github.com/NikichMods/SubtitleSoftEdges/blob/main/docs/STATUS.md) |
| [ProfileRender](https://github.com/NikichMods/ProfileRender) | [docs/STATUS.md](https://github.com/NikichMods/ProfileRender/blob/main/docs/STATUS.md) |
| [AnimationRate](https://github.com/NikichMods/AnimationRate) | [docs/STATUS.md](https://github.com/NikichMods/AnimationRate/blob/main/docs/STATUS.md) |
| [ImmersiveInteractionPrompts](https://github.com/NikichMods/ImmersiveInteractionPrompts) | [docs/STATUS.md](https://github.com/NikichMods/ImmersiveInteractionPrompts/blob/main/docs/STATUS.md) |

## Действующие источники

| Раздел | Что известно | Уровень доказательств |
| --- | --- | --- |
| [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md) | Поддерживаемый Steam-билд и UE/UE4SS; R-004: HP/startup pawn; R-005: native HUD menu owner; R-006: статическая цепочка, slot tree и живые поля CanShow; R-007: Aphes cmap и предел UMG font bridge; R-010: native UHT-сигнатура; R-011: TArray read-side growth | Точные источники/хэши в разделах. CanShow реально прочитал три TipHUD, но full lifecycle/final writer не установлены. Aphes принят в отдельном Coherent-контексте; UMG-рендер остаётся непроверенным. UHT-декларация не равна исполнению hook |
| [GAMEPLAY_RESEARCH.md](GAMEPLAY_RESEARCH.md) | Баланс посоха/кинжалов/копья, выносливость, реакция на урон, ориентирование, UX/визуальные отзывы, исправления патчей | Наблюдения пользователя, датированные сообщения игроков и официальные notes; это **не** runtime-верификация |
| [MOD_IDEAS.md](MOD_IDEAS.md) | Согласованные 18 приоритетов, исключения и открытые продуктовые направления | Продуктовые решения, не обещание реализуемости |
| [MODDING_EVIDENCE.md](MODDING_EVIDENCE.md) | Публичные примеры `.pak`, UE4SS, 3Dmigoto; технические неопределённости для будущих исследований | Внешние опубликованные примеры, не проверенные в текущей установленной игре |
| [GLOSSARY.md](GLOSSARY.md) | Игровые обозначения и уровни достоверности | Справочник |

Дополнение: [R-008 в VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md#r-008--ue4ss-callback-boundaries-for-observation-2026-10-28)
фиксирует post-only Blueprint callbacks и маршалинг до Lua в UE4SS `83330635`.
Это проверка исходника точной версии, не новый игровой прогон.

Дополнение: R-009 в [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md) описывает отсутствие
EPropertyFlags readers в Lua FProperty binding UE4SS `83330635` и границу полноты
metadata capture. Подтверждено исходником, без игрового запуска.

Уточнение R-008: добавлены source-only границы StaticConstructObject, typed struct
copy и UObject wrapper/GC lifetime. Они не доказывают успешное создание/рендер UMG.

Дополнение: R-010 в [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md) — штатный UHT
генератор дал декларацию SetupTipMessage2; правила вывода параметров позволяют
установить отдельные флаги, но normalized macros и stub bodies имеют ограничения.

Дополнение: R-011 в [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md) — индексный Get
TArray UE4SS `83330635` может расширять массив; read-only diagnostics обязаны
проверять границы и время жизни исходной памяти. Проверено исходником, без runtime.

Уточнение R-006: к статической цепочке добавлено отдельное runtime-свидетельство
zero-input CanShow post-hook — чтение текста/состояния трёх живых TipHUD.
Это не подтверждение actor mapping, полного hide/show или final writer.

Последующее R-006 UI-свидетельство: один transient TextBlock добавлен в native
CanvasPanel_6, видимо отрисован штатным шрифтом и вручную удалён; исходная строка/
иконка остались на месте по наблюдению пользователя. Это не Aphes или GC/lifecycle
acceptance; точные исходники, log hash и ограничения находятся в R-006.

R-006 divider follow-up: resource-free UImage Box виден на пользовательском кадре;
native detachment обоих добавленных элементов подтверждён, reason=user, не timeout.
Не обобщать на GC/lifecycle или уверенное визуальное наблюдение исчезновения пары.

Offline-уточнение R-006/R-007: установлены семь slot edges штатного TipHUD,
text/icon layout branches и граница FontFilename/SourceFilename против готового
UMG font resource. Aphes UMG render/cook-load ещё не подтверждён; дерево из
экспорта не равно live geometry. Точный источник и хэши добавлены в эти разделы.

## Откуда взят проверенный технический baseline

- [AsterigosMods/AGENTS.md](https://github.com/NikichMods/AsterigosMods/blob/main/AGENTS.md) — точная версия Steam/UE/UE4SS и правила установки.
- [AsterigosMods/docs/VERIFIED_RUNTIME.md](https://github.com/NikichMods/AsterigosMods/blob/main/docs/VERIFIED_RUNTIME.md) — ранее принятый контрольный контекст и наблюдение DX11.
- `docs/VERIFIED_RUNTIME.md` в каждом новом репозитории и соответствующие исторические `<ModName>/docs/VERIFIED_RUNTIME.md` в `AsterigosMods` сохраняют принадлежащие отдельным модам точные доказательства. При необходимости извлекать общий факт с ссылкой и пределами применимости, а не переносить их документы целиком.

## Автоматическая запись новых результатов

При каждой работе над модом или исследованием Asterigos исполнитель сам проверяет, появилось ли новое **переиспользуемое** знание. Если да:

1. Вносит короткий точный вывод в существующий подходящий файл (или создаёт один обоснованный новый владелец).
2. Указывает версию, источник, доказанный результат, незнакомые/непроверенные части и ограничения.
3. Обновляет этот индекс, если появился новый раздел или изменился важный статус.
4. Оставляет конкретные решения по реализации, тестовые версии, релизы и приёмку **владеющему моду**.

Отсутствие нового принятого факта не требует фиктивного коммита. Если GitHub недоступен, запись остаётся явно ожидающей публикации — она не считается сохранённой здесь, пока не подтверждён коммит.

## Input/action contracts — 2026-10-29

[R-014](VERIFIED_RUNTIME.md#r-014--native-actionmapping-contracts-and-skill-topology-static-2026-10-29) records static semantic action IDs, mapping/default-vs-user boundaries, native held/context getters, absent Blueprint action override, Coherent/UI forwarding and keyboard/pad skill topology. Runtime dispatch, remapping behavior and focus/menu release semantics remain unverified; concrete CombatHUD readiness stays in its owned report. The R-014 startup addition records successful hook registration, two Released callbacks and primitive getter reads, followed by an observer widget-contract stop; this does not establish gameplay dispatch or held behavior.
