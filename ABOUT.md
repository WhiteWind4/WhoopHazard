# WhoopHazard (RotorHazard Fork)

> Fork: [WhiteWind4/WhoopHazard](https://github.com/WhiteWind4/WhoopHazard)
> Upstream: [RotorHazard/RotorHazard](https://github.com/RotorHazard/RotorHazard)

## Что это

RotorHazard — open-source таймерная система для FPV дрон-рейсинга на Raspberry Pi. Считывает сигнал с IR/RF датчиков на гейтах, фиксирует круги, генерирует результаты.

WhoopHazard — наш форк с доработками для российской экосистемы и ATMOS-платформы.

## Наши доработки поверх upstream

- **Lap Annotations** (`out_of_score`, `need_review`) — аннотации кругов судьями
- **Pilot Status** (`remark`, `disqualified`, `left_competition`) — статусы пилотов в гонке
- **Round renumbering** — корректная перенумерация раундов при переносе заезда между хитами

## Ключевые файлы для интеграции с ATMOS

| Файл | Что делает |
|------|-----------|
| `src/server/Database.py` | Модели БД: Pilot, Heat, HeatNode, RaceClass, RaceFormat, SavedRaceMeta, SavedPilotRace, SavedRaceLap, LapSplit |
| `src/server/RHRace.py` | Управление гонкой: RHRace class, Crossing dataclass |
| `src/server/Results.py` | Лидерборды, ранжирование, результаты |
| `doc/RHAPI.md` | API плагинов |
| `src/server/bundled_plugins/rh_data_export_json/` | JSON экспорт данных |
| `src/server/bundled_plugins/rh_data_import_json/` | JSON импорт данных |

## Протокол связи

Flask-SocketIO (Socket.IO). Основные events:
- `race_start`, `race_stop`, `race_save`
- `lap_data` — данные о пересечении
- `frequency_set`, `node_data`

ATMOS подключается к RH через `apps/rh-bridge` (Socket.IO client → relay в платформу).

## Как запустить

Подробности в `FAST_BOOT.md` и `doc/`.
