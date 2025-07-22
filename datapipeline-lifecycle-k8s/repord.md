Данное тестовое задание служит для отработки жизненного цикла приложений в Kubernetes (Labels, Annotations, Probes, Jobs, CronJobs).

Labels (метки)
Используются для идентификации и группировки ресурсов Kubernetes.
Примеры в проекте:

component: frontend|api|database – определяет тип компонента

tier: frontend|backend|database|batch – архитектурный уровень

data-sensitivity: public|internal|confidential – уровень конфиденциальности данных

backup-policy: daily|weekly|none – политика резервного копирования

Для чего?

Фильтрация (kubectl get pods -l component=api)

Селекторы Service (selector: { tier: backend })

Логическая группировка (мониторинг, логирование)

Annotations (аннотации)
Содержат дополнительную метаинформацию, не влияющую на работу Kubernetes.
Примеры в проекте:

kubernetes.io/change-cause: "Обновление до v2.2.0" – история изменений

prometheus.io/scrape: "true" – настройки мониторинга

documentation: "https://..." – ссылки на документацию

Для чего?

Документирование (maintainer, runbook)

Интеграция с инструментами (Prometheus, CI/CD)

Хранение служебной информации (дата деплоя, причина обновления)

Различия между startup, readiness и liveness probes
Тип пробы	Когда выполняется?	Что проверяет?	Последствия ошибки	Пример использования
Startup	При старте контейнера	Готовность приложения к работе	Перезапуск контейнера	Долгая инициализация (БД, кеши)
Readiness	Регулярно после старта	Способность принимать трафик	Исключение из балансировки нагрузки	Временная недоступность (миграции)
Liveness	Регулярно после старта	"Живость" приложения	Перезапуск контейнера	Зависшие процессы
Пример из проекта:

Database использует exec-проверку (pg_isready) для readiness/liveness.

API-Server имеет startupProbe с большим failureThreshold (30 попыток).

Сравнение поведения Job'ов
Параметр	retry-job	data-transform-job	conditional-job
restartPolicy	Never	OnFailure	OnFailure
backoffLimit	5 попыток	–	–
completions	1 (по умолчанию)	6	–
parallelism	–	3	–
Условия	Случайная ошибка	Параллельные задачи	Проверка данных перед запуском
Выводы:

backoffLimit ограничивает число попыток для retry-job.

parallelism и completions позволяют распараллелить задачи (как в data-transform-job).

Анализ CronJob'ов
CronJob	Расписание	ConcurrencyPolicy	Условия выполнения
database-backup	0 2 * * * (2:00)	Forbid	–
data-cleanup	0 1 * * 0 (воскресенье)	Replace	–
health-report	*/15 * * * * (каждые 15 мин)	Allow	Проверка API (exit 0/1)
Политики параллелизма:

Forbid: Запрещает параллельные запуски (важно для бэкапов).

Replace: Отменяет текущий запуск, если начат новый.

Allow: Разрешает параллельные выполнения (для частых задач).

Результаты тестирования Health Checks
Компонент	Startup Time	Readiness Time	Проблемы
web-frontend	5 сек	5 сек	Нет
api-server	До 150 сек	5 сек	Долгий старт (исправлено startupProbe)
database	60 сек	15 сек	Требует pg_isready
Выводы:

startupProbe критичен для долго стартующих приложений.

Readiness-пробу нужно настраивать с учетом реального времени инициализации.

Рекомендации по оптимизации
Для долгих стартов:

Увеличить failureThreshold для startupProbe.

Использовать initContainers для предварительной настройки.

Для отказоустойчивости:

Добавить livenessProbe для всех stateful-сервисов (БД, очереди).

Настроить preStop-хуки для graceful shutdown.

Для CronJob'ов:

Использовать Forbid для задач с эксклюзивным доступом (бэкапы).

Логировать результаты выполнения (exit 0/1).

Troubleshooting Guide
Проблема	Диагностика	Решение
Pod в CrashLoopBackOff	kubectl logs <pod> --previous	Проверить логи и livenessProbe.
Сервис не маршрутизирует трафик	kubectl describe endpoints <service>	Проверить readinessProbe и labels.
CronJob не запускается	kubectl get cronjobs -o yaml	Проверить расписание и политики.
Health checks падают	kubectl describe pod <pod>	Настроить корректные порты/paths.
