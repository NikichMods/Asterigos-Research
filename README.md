# Asterigos Research

Общая база исследований для **Asterigos: Curse of the Stars** и модов NikichMods.

Здесь хранятся повторно используемые сведения об устройстве игры, наблюдения игроков, проверяемые источники, принятые продуктовые решения и открытые вопросы. Это **не репозиторий исходного кода конкретного мода**, не хранилище сборок и не стенограмма переписки.

## С чего начать

- [AGENTS.md](AGENTS.md) — правила работы с базой и границы достоверности.
- [Индекс общих исследований](docs/RESEARCH_INDEX.md) — каноническая отправная точка для всех модов Asterigos.
- [Проверенная общая среда](docs/VERIFIED_RUNTIME.md) — ранее принятые runtime-факты из AsterigosMods с указанием ограничений.
- [Внешняя инвентаризация боевой системы](docs/COMBAT_SYSTEM_EXTERNAL_INVENTORY.md) — атаки шести видов оружия, таланты, ресурсы, прогрессия и границы технической доступности.
- [Асимметричный баланс оружия — исследование на примерах других игр](docs/WEAPON_BALANCE_DESIGN_RESEARCH.md) — переносимая методика, первоисточники и гипотезы для Asterigos.
- [Исследование геймплея и качества жизни](docs/GAMEPLAY_RESEARCH.md) — наблюдения, жалобы сообщества, статус доказательств и ссылки.
- [Приоритеты модификаций](docs/MOD_IDEAS.md) — 18 направлений, решения об исключении и условия для исследования.
- [Технические зацепки](docs/MODDING_EVIDENCE.md) — подтверждённые способы модификации и важные неизвестные.
- [Глоссарий](docs/GLOSSARY.md) — игровые и исследовательские термины.

## Как пользоваться

1. Для новой идеи сначала проверить существующие выводы и источники.
2. Отличать факт о конкретной версии, пользовательское наблюдение, сообщение из сообщества и ещё не проверенную гипотезу.
3. При исследовании игровых файлов добавлять точные имена объектов, владельцев поведения, путь данных, версию игры и воспроизводимую проверку; не угадывать их.
4. При создании отдельного мода хранить его исходный код, решения об устройстве и приёмку **в репозитории этого мода**. Сюда переносить только знания, полезные нескольким проектам.
5. Не публиковать игровые ассеты, декомпилированные материалы, сторонний код и персональные данные без отдельного основания.

Первая запись: **27 октября 2026** — результаты обсуждений игрового баланса, обратной связи, навигации, интерфейса, визуальных улучшений и предварительной реализуемости.

Состояние: собрана **исследовательская база**, но игровые файлы и точная установленная версия ещё не изучались в рамках этого репозитория. Приоритеты отражают текущую оценку полезности, **не** проверенную трудоёмкость.

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
