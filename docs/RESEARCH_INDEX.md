# Research Index — Asterigos: Curse of the Stars

**2026-10-30 — R-018 bounded runtime:** own transient PointLightComponent on
Hilda's root completed EngineTick creation, dark registration/attachment,
ON/OFF, guarded movement and native manual teardown; user confirms visible light.
[Exact limits](VERIFIED_RUNTIME.md#r-018-owned-pointlight--bounded-runtimemanual-teardown-pass-2026-10-30):
automatic timeout/Glimmer/full lifecycle untested; visual settings remain mod-owned.

**2026-10-30 — R-018 static lighting:**
[VERIFIED_RUNTIME.md R-018](VERIFIED_RUNTIME.md#r-018--hilda-stock-flash-light-component-chain-and-independent-lighting-apis-static-2026-10-30)
records Hilda's serialized `light`→PointLight subtree, `flash_light`/Lights
Blueprint control and reflected lighting/attachment APIs. Source/static evidence;
no new light rendering, level lifecycle or runtime acceptance. Mod-owned report:
[HildaSoftLight](https://github.com/NikichMods/HildaSoftLight/tree/research/lighting-feasibility).

**2026-10-30 — дизайн-контракты шести видов оружия:**
[WEAPON_BALANCE_CONTRACTS.md](WEAPON_BALANCE_CONTRACTS.md) фиксирует
согласованные принципы и фантазии, отделяя их от ещё не принятых
предложений по агро, stagger, knockdown, AP и наведению.

**2026-10-30 — публичная инвентаризация боевой системы:**
[COMBAT_SYSTEM_EXTERNAL_INVENTORY.md](COMBAT_SYSTEM_EXTERNAL_INVENTORY.md) —
оружие, обычные/переходные атаки, таланты, AP/SP, стихии и технические
прецеденты. Внешние сведения не равны runtime-фактам Steam 1.09;
точные формулы и владельцы данных не установлены.

**2026-10-30 — самостоятельное cross-game дизайн-исследование:**
[асимметричный баланс оружия](WEAPON_BALANCE_DESIGN_RESEARCH.md) — первичные
интервью и материалы разработчиков, проектные принципы и методика проверки;
это не доказательство текущих чисел или механик Asterigos.

**2026-10-30 — R-006 partial runtime:** borrowed retained MID/Icon device4→1
updated visible glyph to keyboard F; guard cleanup passed. Small/faded/bluish
output reported, pixel equivalence/full cycle not accepted.
[Evidence](VERIFIED_RUNTIME.md#borrowed-retained-hint--partial-runtime-proof-2026-10-30).

**2026-10-30 — R-006 source-only:** native Retainer/MID texture getters and
Shipping global-mode boundary reviewed;0 from cvar getter is ambiguous. No new
current-hint/device/rendering PASS.
[Evidence](VERIFIED_RUNTIME.md#retainer-readiness-getters-and-global-mode-boundary--source-only-2026-10-30).

**2026-10-30 — R-006 source-only:** current text/FText marshalling limits and
Retainer hint material stage reviewed. Shared MID in a second Retainer can gain
another render-target writer; no new live input-hint synchronization PASS.
[Evidence](VERIFIED_RUNTIME.md#current-text--retained-input-hint--source-only-integration-boundary-2026-10-30).

**2026-10-30 — R-016 bounded frame-loop/manual-stop runtime PASS:**1068 own
rotation updates, explicit handle cancellation and user-triggered native cleanup
completed. User visual confirmation; no FPS benchmark/full lifecycle claim.
[Evidence](VERIFIED_RUNTIME.md#frame-loop-and-explicit-user-cleanup--bounded-runtime-pass-2026-10-30).

**2026-10-30 — R-016 source-only:** pinned frame-loop ownership, zero-result
callback contract and cancellation/removal reviewed. Runtime cadence/cancellation
not accepted by this entry.
[Exact limits](VERIFIED_RUNTIME.md#pinned-frame-loop-ownership-and-cancellation--source-only-2026-10-30).

**2026-10-30 — R-016 bounded rotation runtime addition:** reflected native setter
returned on one owned registered World component; target-loss native cleanup
completed. No final follow metrics/manual F7/design acceptance.
[Evidence and limits](VERIFIED_RUNTIME.md#registered-scene-rotation--bounded-runtime-addition-2026-10-30).

**2026-10-30 — R-016 source-only:** registered scene rotation setter and pinned
OutParm table/scheduler contracts reviewed. Actual native rotation/marshalling
and visual smoothness remain untested.
[Evidence and limits](VERIFIED_RUNTIME.md#registered-scene-rotation-and-reflected-outparm--source-only-2026-10-30).

**2026-10-30 — R-016 bounded runtime PASS:** one independent actor-owned World
WidgetComponent/private host rendered; user confirmed Hilda partial-letter
occlusion and reappearance. Native state and45s timeout cleanup completed; probe
disabled. Manual F7/dynamic billboard/broader lifecycle remain unverified.
[Exact scope/evidence](VERIFIED_RUNTIME.md#r-016-runtime-follow-up--one-owned-world-rendererocclusiontimeout-cleanup-pass-2026-10-30).

**2026-10-30 — R-016 pinned execution-context follow-up, source-only:** script
callbacks retain the UE4SS global recursive mutex; explicit EngineTick actions
execute outside the queue-lock scope. LoadAsset may synchronously LoadObject.
[Contracts and limits](VERIFIED_RUNTIME.md#pinned-script-callback-versus-enginetick-execution--source-only-2026-10-30).
This does not diagnose a particular hang or establish a runtime fix.

**2026-10-29 — R-016 source follow-up:** explicit transient native ownership and
manual FinishAddComponent registration/teardown reviewed. UE4SS Lua IsValid does
not reject PendingKill; reflected native Kismet validity is the distinct logical
death check. [Contracts and limits](VERIFIED_RUNTIME.md#r-016--world-widgetcomponent-rendererdepthlifecycle-limits-2026-10-29).
No World registration/render/occlusion runtime result is inferred.

**2026-10-29 — R-006 natural Open join/pose live PASS:** BeginPlay acquisition
plus CanShow matched one real item_lootbox02_C actor/component/TipHUD and read2
finite world poses after one manual reload. World rendering/occlusion is untested.
[Exact scope and evidence](VERIFIED_RUNTIME.md#natural-open-actorcomponentwidget-join-and-world-pose--live-pass-2026-10-29).

**2026-10-29 — R-006 acquisition live partial PASS:** parent BP hook fired after
manual reload;3 Open actor/component/TipHUD pairs passed reciprocal getter checks.
Initialization budget stopped observation before CanShow/pose, so active selection
remains unproved. [Live evidence and limits](VERIFIED_RUNTIME.md#parent-beginplay-acquisition--live-partial-pass-2026-10-29).

**2026-10-29 — R-006 script-hook lifetime, source-only:** pinned registrations
use function/Outer name chains at mod scope; reviewed LoadMap handlers preserve
them, uninstall clears them. This supports a bounded reload test, not live reload
acceptance. [Lifetime and cleanup limits](VERIFIED_RUNTIME.md#script-hook-lifetime-across-map-changes--source-contract-2026-10-29).

**2026-10-29 — R-006 positive initialization callsite, source-only:** Open-tagged
BaseLoot BeginPlay explicitly calls parent BaseGameEventObject BeginPlay, which
reads its own component's widget. Exact non-native post-hook contract reviewed;
live binding remains unproved. [Chain and limits](VERIFIED_RUNTIME.md#open-tagged-baseloot-initialization-chain--source-only-2026-10-29).

**2026-10-29 — R-006 SetShow NO_EVENTS:** native hook registration succeeded,
but0 callbacks/samples in31.965s before user stop. No new live owner/pose proof;
missing-event cause unknown. [Negative result and limits](VERIFIED_RUNTIME.md#setshow-event-route--bounded-no_events-observation-2026-10-29).

**2026-10-29 — R-006 component-binding source follow-up:** exact widget/owner/
pose getter contracts and reciprocal BP object fields checked. No new live owner
mapping; natural reflected SetShow dispatch remains an untested event route.
[Contracts and limits](VERIFIED_RUNTIME.md#component-side-binding-contracts--source-only-follow-up-2026-10-29).

**2026-10-29 — R-017 host-only runtime PASS:** shipped PostionItemUMG_BP_C loaded
through its exact generated-class registry path, created with an independent tree,
shown after Construct Hidden and manually released from viewport. Original HUD
fingerprint unchanged; user confirmed visible show/removal and normal original UI.
See [host facts and limits](VERIFIED_RUNTIME.md#r-017--concrete-independent-userwidget-host-candidate-2026-10-29).
This is one viewport cycle, not World rendering/occlusion or production lifecycle.


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
| [WEAPON_BALANCE_CONTRACTS.md](WEAPON_BALANCE_CONTRACTS.md) | Желаемые роли шести видов оружия, принятые принципы, кандидатные механизмы, риски и дальнейшие проверки | Дизайн: общая цель согласована, конкретные изменения остаются гипотезами |
| [COMBAT_SYSTEM_EXTERNAL_INVENTORY.md](COMBAT_SYSTEM_EXTERNAL_INVENTORY.md) | Внешняя инвентаризация механик шести видов оружия, талантов, AP/SP, стихий, прогрессии и классов возможных правок | Патчи, обзоры, wiki, сообщения игроков и моды; локальный runtime и поля данных **не исследованы** |
| [WEAPON_BALANCE_DESIGN_RESEARCH.md](WEAPON_BALANCE_DESIGN_RESEARCH.md) | Межигровые принципы асимметричного баланса оружия, источники создателей, методика сравнения, гипотезы для Asterigos | Проверенные публичные источники + собственный синтез; внутриигровые параметры не измерялись |
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

R-006 bounded runtime follow-up (2026-10-30): [live Canvas slot size](VERIFIED_RUNTIME.md#live-canvaspanelslot-size-seam--source-cross-check-2026-10-30)
changed/read back and visibly enlarged one owned Image with unchanged projected
center/component/material/RT. No native Screen-size, rebind or color acceptance.

R-006 source addition (2026-10-30): [live CanvasPanelSlot size seam](VERIFIED_RUNTIME.md#live-canvaspanelslot-size-seam--source-cross-check-2026-10-30)
updates own live Slate offsets without re-registration; source/readback contract,
not new visual acceptance or an adopted mod size.

R-006 bounded runtime addition (2026-10-30): [own registered World material refresh](VERIFIED_RUNTIME.md#registered-owned-world-material-refresh--bounded-runtime-2026-10-30)
succeeded with new own MID and unchanged RT/source. No color-fix or general
blend-mode/occlusion equivalence is inferred.

R-006 bounded runtime addition (2026-10-30): [viewport/projected point numeric out tables](VERIFIED_RUNTIME.md#player-viewport-and-projected-point-out-tables--bounded-runtime-2026-10-30)
worked on the pinned baseline. No FGeometry or material/pixel equivalence claim.

R-006 source addition (2026-10-30): [opaque geometry return / own material update](VERIFIED_RUNTIME.md#lua-geometry-return-and-owned-material-refresh--source-only-2026-10-30).
Pinned Lua FGeometry return loses unreflected payload; projected numeric-vector
out tables are a distinct route. Registered owned WidgetComponent material
refresh has a source-confirmed reflected seam, not new runtime acceptance.

R-006 source follow-up (2026-10-30), [retained UI→World distinctions](VERIFIED_RUNTIME.md#retained-ui-material--world-surface--source-only-distinctions-2026-10-30):
material-specific AlphaComposite versus draw flags, default gamma1 versus target
encoding, explicit material override precedence and world units versus pixels.
No new runtime acceptance or universal washed-out-color fix is claimed.

[R-014](VERIFIED_RUNTIME.md#r-014--native-actionmapping-contracts-and-skill-topology-static-2026-10-29) records static semantic action IDs, mapping/default-vs-user boundaries, native held/context getters, absent Blueprint action override, Coherent/UI forwarding and keyboard/pad skill topology. Runtime dispatch, remapping behavior and focus/menu release semantics remain unverified; concrete CombatHUD readiness stays in its owned report. The R-014 startup addition records successful hook registration, two Released callbacks and primitive getter reads, followed by an observer widget-contract stop; this does not establish gameplay dispatch or held behavior.

R-014 observer02 addition now demonstrates the native modifier getter in gameplay, both skill-root bindings and native menu-ancestor hiding. Semantic Interact dispatch, remapping and lost-release/focus behavior remain OPEN; capture ended at the observer's240-line budget. Exact evidence and limits stay linked to CombatHUD.

R-014 scalar continuation and live addition document Interact virtual-key identities plus demonstrated current-PC GetKeyByVirtualKey/FKey/native key-edge queries, avoiding unnecessary out arrays. Live before/after rebinding, chords and other platform namespaces remain unverified; see the exact observer03 result and its scope.

R-008 source/static addition2026-10-30: [delayed GameThread timer contract](VERIFIED_RUNTIME.md#r-008-delayed-gamethread-actions--sourcestatic-2026-10-30)
records pinned83330635 single-loop/rearm/handle cancellation, restore-before-cancel
ordering and later upstream lifecycle fixes. Installed DLL identity/API strings
were checked; new timer execution and native-crash safety remain unverified.
