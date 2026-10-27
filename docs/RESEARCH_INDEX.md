# Research Index — Asterigos: Curse of the Stars

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
| [VERIFIED_RUNTIME.md](VERIFIED_RUNTIME.md) | Поддерживаемый в `AsterigosMods` Steam-билд, версия UE/UE4SS, DX11, runtime-контракт; R-004: HP-getters и startup pawn; R-005: native HUD menu owner, opacity rollback и identity/display-spelling boundary; R-006: статическая компонентная цепочка interaction prompts; R-007: Aphes cmap/UFont/language boundaries | Перенесено из принятой документации и точного отчёта локальной runtime-сессии с commit/hash; не повторно проверено в этом репозитории; R-006 — статическая проверка; R-007 — статические свойства шрифта плюс ограниченная игровая приёмка штатного Aphes-класса в Coherent |
| [GAMEPLAY_RESEARCH.md](GAMEPLAY_RESEARCH.md) | Баланс посоха/кинжалов/копья, выносливость, реакция на урон, ориентирование, UX/визуальные отзывы, исправления патчей | Наблюдения пользователя, датированные сообщения игроков и официальные notes; это **не** runtime-верификация |
| [MOD_IDEAS.md](MOD_IDEAS.md) | Согласованные 18 приоритетов, исключения и открытые продуктовые направления | Продуктовые решения, не обещание реализуемости |
| [MODDING_EVIDENCE.md](MODDING_EVIDENCE.md) | Публичные примеры `.pak`, UE4SS, 3Dmigoto; технические неопределённости для будущих исследований | Внешние опубликованные примеры, не проверенные в текущей установленной игре |
| [GLOSSARY.md](GLOSSARY.md) | Игровые обозначения и уровни достоверности | Справочник |

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
