# Combat contracts — статические данные 2026-11-05

**Уровень:** cooked tables / Blueprint bytecode / native declarations. Нового runtime-теста нет. Это общий источник знаний о боевой структуре игры; поведение, параметры и готовность отдельных модов принадлежат их репозиториям.

Первичный отчёт: [StaffRebalance Static Combat Research 01, source 9348a6f](https://github.com/NikichMods/StaffRebalance/blob/9348a6ff7d8206b725f089428652ccbec8012675/docs/STATIC_COMBAT_RESEARCH_01.md). Точная идентичность 67 выбранных входов и reader: [EVIDENCE.json](https://github.com/NikichMods/StaffRebalance/blob/9348a6ff7d8206b725f089428652ccbec8012675/research/static-combat-01/EVIDENCE.json). Сырые игровые экспорты, ключи и машинные пути не опубликованы.

## Baseline и границы метода

- Установленный Steam manifest: app 1731070, build 12996889, depot 1731071 / manifest 4552465433899076789. Принятый проектный контекст — Steam 1.09, UE4.27, Windows x64. Manifest не доказывает неизменность каждого файла установки.
- UE4SS DLL SHA-256 `ce5e3c46a920cc00b2817ac61e438c9162830290379d0b5b9c4dfc2b29459290`, ранее исследованный `83330635`.
- Переиспользован прежний UHTHeaderDump; 17 выбранных headers совпали с сохранённой генерацией IIP. Generated `.cpp` — stubs. Подробности происхождения/ограничений генерации — R-010 в [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md).
- CUE4Parse 1.2.2, GAME_UE4_27, ReadScriptData=true, существующий reader; выборочное чтение из верхнего уровня штатного Paks. Ни игра, ни новый dumper не запускались.

## Реальные владельцы данных

| Область | Статически установленный контракт | Предел применения |
| --- | --- | --- |
| Skill | `FSkillData` имеет DamageRate, Tags, SkillBeginTimelineData/TimelineData/SkillEndTimelineData; реальные таблицы `/Game/MainContent/Tables/Player/Skill_Player`, `TimelineData_PlayerSkill[_1/_2]`, `TimelineData_Status` | Декларация/строка не доказывает live commit или порядок native-интерпретатора |
| Источник повреждения | `/Script/Genesis.BaseProjectile`: DamageRate, DamageAttributeName, HitStatus, HitGainAstralPool; GetCharacterOwner, GetBaseDamage/GetBaseDamageRate/GetBaseTypeDamage/GetTypeDamage и SetBaseDamage/SetBaseDamageRate; `GameGlobal.SpawnProjectile/SpawnProjectileEX` | Не установлены native snapshot/caches, итоговая формула и полнота ProcessEvent callbacks |
| Цели | Разные ProjectileOverlapCharacter, ProjectileOverlapScene, ProjectileOverlapGameEventObject; в прочитанных source rows IsCanHitScene и IsCanHitCharacter могут быть одновременно true. Для исследованного Astral hit нативная ветвь персонажа проверяет GenesisCharacter перед расчётом | Общий DamageRate может затронуть источники, которые взаимодействуют и со сценой. Другие ветви и типы целей отдельно не исследованы |
| HP | CharacterAttributeComponent.HP/MaxHP/OnCurHPChanged; GameGlobal HP accessors и отдельные BaseGameObject.GetHP/AddHP. В обычной положительной ветви Astral дельта доходит до нативного писателя CharacterAttributeComponent | Установлено статически на одном билде. Доступный мутабельный перехват и live порядок не установлены |
| Замены | `/Game/MainContent/Tables/Remapping`, `Status_Talent`, `Status_Skill`, weapon ModuleBuffList и SwitchWeaponSkill_Enter | Нельзя классифицировать происхождение по одному текущему оружию или предполагать отсутствие DLC/module/talent путей |

## Действие не равно projectile и не равно hit

В `staff_atk1/2/3` таблицы содержат соответственно 1/2/3 отдельных `fire object` события (кадры 23; 22/25; 26/29/32, FPS=30). Fourth Astral использует общий первый source и timeline, создающую шесть дочерних sources. Это значения cooked timelines, не измеренные timings.

Melee также использует эту модель: `hilda_dual_atk1` → `dual_atk1` имеет два fire события, `hilda_dual_attack` на кадрах 6 и 18; hammer/spear rows создают собственные projectile sources. Более того, **обычные staff skill rows имеют `Melee=true`**, как и прочитанные melee rows. Этот field не является достаточным семантическим фильтром оружия ближнего боя; его native назначение здесь не установлено.

BeginSkillStateEvent/EndSkillStateEvent и GetDoingSkill существуют, но skill name не уникальный ID исполнения, а начало состояния не доказано как момент фактически выполненного выстрела. Табличный Group и SetGroup(int32) не доказаны как идентификатор пользовательской атаки.

## Дочерние эффекты и штатные damage writers

Fire hit timeline создаёт отдельный `hilda_staff_bolt_fire_explosion`; Ice BP выбирает hit timeline по тегам/`talent_e804`, затем создаёт `hilda_staff_bolt_ice_hit`. Fourth имеет отдельные beam/main/hit источники. В bytecode есть child `SpawnProjectile` как с creator=self, так и с creator=player; универсальное наследование parent attack через Owner не установлено. Полная Thunder chain/retarget и частота DoT/ticks не выведены из имён.

В `hilda_staff_bolt_astral` реально прочитаны сохранение `Original Damage Rate`, умножение DamageRate на 1.1 при `talent_e804` и последующее восстановление; аналогичная логика есть у Thunder. `hilda_staff_arrow` отдельно меняет DamageProjectile.DamageRate. Однократная внешняя запись этого поля не гарантирует сохранение значения или корректное stacking. Точные offsets и SHA — в первичном отчёте.

Charged Rays разделён на aim/charge/shoot skills; shoot timeline создаёт carrier `hilda_staff_arrow`, который сам создаёт damage-projectile. Rapid Fire (`p802`) использует Remapping и другие переходы/SP cost. Fourth Quadrant (`e805`) подключает переход к четвёртому удару; Staff Mastery (`e809`) имеет SkillDamageRate status function. English localization связывает Judgment (`u805`) с `hilda_ult_com05`; stance использует staff fourth sources через отдельный `hilda_ult_com05_atk1`, без префикса `hilda_staff_`. Это data/bytecode факты, не доказательство полного живого поведения или готового способа изменения.

## Нативное попадание обычного Astral-снаряда

Статическая проверка Steam 1.09 / build 12996889, Windows x64, UE 4.27. SHA-256 исполняемого файла `ddea6b5ca5d34836f01702993d20209c2595367287e6cd5a135a7ace1958b1c0`. [Отчёт с RVA, переходами и независимой сверкой HP](https://github.com/NikichMods/StaffRebalance/blob/76a52a4e4b282f9bfd74d8ee495b13d5f4523b62/docs/ASTRAL_NATIVE_HP_PATH.md).

- Контакт с персонажем проходит отдельную ветвь с проверкой GenesisCharacter и передаёт исходный projectile и hit component в компонент цели. Для сцены обнаружены отдельные обработчик и уведомления.
- Компонент цели рассчитывает сумму по типам повреждения и в обычной положительной ветви вызывает прямой писатель HP (`0x10C1C60`). `GameGlobal.DamageCharacter_Cpp` использует общий механизм изменения HP по другому маршруту. Путь попадания через отражённую функцию не проходит.
- `GenesisCharacter.ProjectileHitEvent` вызывается с целью, исходным projectile, hit component, позицией и копией урона перед штатной записью HP. Последующие нативные условия могут изменить фактическую дельту. `BaseProjectile.ProjectileOverlapCharacter` в этом маршруте вызывается после обработки.
- `ProjectileHitEvent` — кандидат для read-only наблюдения. Доставка на конкретном Blueprint цели и обработка аргумента `FVector_NetQuantize` текущим UE4SS требуют проверки. Параметр `Damage` передаётся по значению, а штатный писатель получает величину из нативного расчёта. Связь с экземпляром projectile прямая, идентификатор породившего его выполнения skill остаётся открытым.

## Conduit: rest, recovery и transport различимы статически

`ExtendedCoherentUIGTGameHUD` имеет отдельные SavePointOnRest/OnTransport/OnExit/OnFail. `BaseSavePoint_BP` связывает rest с `Interact_Rest`; эта ветвь запускает `use_savepoint_rest`, отдельные таймеры 1.5 и 3.5 и восстановление через `GameGlobalBlueprintLibrary.RecoverPlayerAndScene`. Конкретный `SavePoint_BP.UseRest` обслуживает animation/sound/VFX.

Library отправляет GenesisGameInstance.ExecuteAction с именем RecoverPlayerAndScene; найден соответствующий `MainGameInstance_BP.RecoverPlayerAndScene` и его BP-путь: player HP=MaxHP, Stamina=MaxStamina, AstralPool=MaxAstralPool, ResetCharacterCreators(false). В rest branch отдельно вызывается HUD EventOnRestEnd. Native dispatch между library и instance event и реальный порядок успешного отдыха не наблюдались.

Не считать menu/open/transport отдыхом и не объявлять общий recovery event rest-exclusive без проверки других вызывающих контекстов. Полезный следующий вопрос — live порядок rest/recovery/end и source/hit/HP на узкой выборке; никакого нового gameplay-механизма эти сведения сами по себе не разрешают.
