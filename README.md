        # redis — Репликация в деталях

        Homework-шаблон для урока **l1_replication** (Репликация в деталях) на платформе Vibe Learn.

        ## Что делать

        docker-compose: master + replica. Скрипт: load-generator на master (10k RPS write).
Эмулирует разрыв сети (iptables drop), измеряет backlog usage и lag при разных
repl-backlog-size (1mb, 32mb, 256mb). Отчёт: при каких параметрах partial resync
выживает 5с разрыв, 30с разрыв.

## Контекст (из transfer-задачи урока)

Конфиг прода: master + 1 replica, async, `repl-backlog-size 1mb` (default). Write-нагрузка
~50MB/sec. Каждые ~2 минуты сеть моргает на ~5 секунд. В мониторинге наблюдается шторм
BGSAVE-ов и memory spike. Объясни причину и три конкретных изменения конфига чтобы починить.

## Recap из урока

- PSYNC при подключении реплики решает partial vs full resync через replid + offset в backlog.
- Default `repl-backlog-size 1mb` слишком мал для production write-heavy — увеличь до 32-64mb.
- `min-replicas-to-write` защищает от split-brain: мастер отказывает в записи если реплик слишком мало или они отстают.
- Diskless replication (Redis 6+) шлёт RDB прямо в сокет минуя диск — полезно при слабом IO.
- `REPLICAOF NO ONE` — manual failover. Старый мастер нужно вручную перевести в реплики нового, иначе split-brain.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node Redis 7 на `localhost:6379` (с включёнными keyspace-notifications и AOF). Адрес переопределяется через env `REDIS_ADDR`.

        ## Запуск

        ```bash
        # Поднять локальный Redis
        docker compose up -d

        # Прогнать тесты (интеграционный включается через REDIS_INTEGRATION=1)
        go test ./...
        REDIS_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
