## Компоненты и их назначение (To‑Be)

| Компонент | Назначение |
| --- | --- |
| **Operational Systems** (Core Banking, CRM, Cards, Loans) | Мастер‑системы: создают и меняют состояние сущностей. В них живут state machines сущностей (карта, кредит, счёт). |
| **MDM** | Единый источник истины по клиентам и справочникам. Выдаёт global_client_id, хранит правила маппинга и дедупликации. |
| **Integration Layer** | Нормализует форматы, обогащает данными из MDM, контролирует качество, маршрутизирует в Kafka/DWH, применяет политики безопасности. |
| **Kafka** | Потоковая шина: события жизненного цикла, CDC из Core, статусы карт/кредитов. |
| **DWH** | Структурированные витрины, регламентированные отчёты, данные для регуляторики, витрины для BI. |
| **Data Lakehouse** (S3/MinIO + Delta/Iceberg) | Сырые события, исторические логи, ML‑датасеты, аудит. |
| **Airflow** | Оркестрация batch‑процессов: ночные загрузки, пересчёт витрин, выгрузки, проверки качества. |
| **BI** | Дашборды, аналитика, self‑service, отчётность для бизнеса. |
| **DataHub** | Каталог данных, lineage, владельцы, бизнес‑глоссарий, поиск. |

1. **Месяцы 1–3**: внедрить Integration Layer и Kafka, настроить CDC из Core и потоки в MDM, запустить базовый DataHub (метаданные от Operational Systems), развернуть DWH и базовые витрины.
2. **Месяцы 4–6**: настроить Airflow для ночных пайплайнов, подключить BI к витринам, зафиксировать владельцев данных.
3. **Месяцы 7–9**: развернуть Data Lakehouse для сырых событий и аудита, настроить lineage и контроль качества в Integration Layer, внедрить mTLS и rate limiting.
4. **Месяцы 10–12**: автоматизировать выгрузки для регуляторики, стабилизировать SLA витрин и событий, оформить модель управления данными (владельцы, stewards, процессы).

@startuml
title C4 Context: Целевая архитектура данных (To-Be, 12 мес)
left to right direction

rectangle "Operational Systems\n(Core, CRM, Cards, Loans)" as Ops
rectangle "MDM\n(global_client_id,\nсправочники)" as MDM
rectangle "Integration Layer\n(нормализация,\nконтроль качества)" as INT
rectangle "Kafka\n(события,\nCDC)" as KAFKA
rectangle "DWH\n(витрины,\nотчётность)" as DWH
rectangle "Data Lakehouse\n(сырые,\nисторические)" as LAKE
rectangle "Airflow\n(оркестрация)" as AIR
rectangle "BI\n(дашборды)" as BI
rectangle "DataHub\n(каталог,\nlineage)" as DH

Ops --> INT : Capture (CDC/Async)
INT --> MDM : Enrich / Dedup
MDM --> INT : Return global_id
INT --> KAFKA : Publish events
KAFKA --> DWH : Micro-batch load
KAFKA --> LAKE : Raw append
DWH --> BI : Read (JDBC)
DWH --> AIR : Triggers for reports
AIR --> DWH : Nightly recalcs
DWH --> DH : Metadata / Lineage
Ops --> DH : Publish metadata

note right of INT
  - mTLS
  - rate limiting
  - circuit breaker
  - валидация схем
end note

note bottom of KAFKA
  - append-only
  - порядок по timestamp_utc
  - основа для трассировки
end note
@enduml