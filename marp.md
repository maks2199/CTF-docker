---
marp: true
theme: default
paginate: true
style: |
  .timeline {
    position: fixed;
    top: 12px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 6px;
    font-size: 20px;
    z-index: 1000;
  }
  .timeline-item {
    padding: 5px 14px;
    border-radius: 3px;
    background: #ddd;
    color: #888;
    font-weight: 400;
    white-space: nowrap;
  }
  .timeline-item.done {
    background: #eee;
    color: #aaa;
  }
  .timeline-item.current {
    background: #000;
    color: #fff;
    font-weight: 600;
  }
  .timeline-item.future {
    background: #eee;
    color: #aaa;
  }
  header {
    color: #9e9e9eff;
    font-size: 16px;
    text-align: center;
    padding: 10px 0;
  }
  footer {
    color: #9e9e9eff;
    font-size: 14px;
    text-align: center;
    padding: 10px 0;
  }

header: "Побег из докер-контейнера"
footer: "Тачков Максим | Новогодний CTF 2026 - Взламывай, Защищай, Побеждай!"
---

# Побег из докер-контейнера

## CTF Workshop

**Образовательный воркшоп по безопасности контейнеров**

---

<div class="timeline">
  <div class="timeline-item future">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## План воркшопа

1. **Реальные инициденты**
2. **Векторы атаки**

   - Privileged контейнеры
   - Docker socket escape
   - CAP_SYS_ADMIN побег

3. **Лучшие практики для защиты**
4. **Инструменты диагностики** (Trivy/Scout)

---

<div class="timeline">
  <div class="timeline-item future">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Для кого этот воркшоп?

**Целевая аудитория:**

✅ **DevOps / SRE инженеры** - работаете с Docker в production
✅ **Backend разработчики** - пишете Dockerfile и docker-compose
✅ **Junior Security Engineers** - хотите понять container security
✅ **Студенты DevSecOps** - нужен практический опыт

**Уровень:** Intermediate (базовые знания Docker требуются)

<!-- **Что получите:**

- Понимание реальных векторов атак на контейнеры
- Практический опыт побега из контейнеров
- Навыки сканирования образов (Trivy/Scout)
- Готовые примеры безопасных конфигураций -->

---

<div class="timeline">
  <div class="timeline-item future">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Prerequisites: Что нужно знать?

**Вопрос к аудитории:** Кто из вас запускал Docker контейнеры? 🙋

**Docker за 30 секунд (если вы не уверены):**

```bash
# Docker = платформа для запуска приложений в изолированных контейнерах
docker pull ubuntu          # Скачать образ (шаблон)
docker run -it ubuntu bash  # Запустить контейнер (экземпляр)
docker ps                   # Список запущенных контейнеров
docker exec -it <id> bash   # Подключиться к контейнеру
```

```
docker run -d -p 3030:3000 --name=grafana grafana/grafana-enterprise
```

**Образ** = шаблон с ОС и приложением | **Контейнер** = запущенный процесс из образа

---

<div class="timeline">
  <div class="timeline-item future">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Введение: Что такое контейнер?

- Изолированное окружение для запуска приложений
- Использует namespaces и cgroups ядра Linux
- Не полная виртуализация - разделяет ядро с хостом
- Изоляция ≠ безопасность по умолчанию

```bash
docker run -it ubuntu:latest /bin/bash
```

---

<!-- ## Модель безопасности Docker

**Основные механизмы защиты:**

- **Namespaces** - изоляция процессов, сети, файловой системы
- **Capabilities** - гранулярные привилегии вместо полного root
- **Seccomp** - фильтрация системных вызовов
- **AppArmor/SELinux** - политики доступа

⚠️ **Изоляция ≠ безопасность по умолчанию!**

--- -->

<div class="timeline">
  <div class="timeline-item current">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# Реальные инциденты

---

<div class="timeline">
  <div class="timeline-item current">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Реальные инциденты

**Graboid Worm (2019)** 📚 [Palo Alto Unit 42: Graboid worm](https://unit42.paloaltonetworks.com/graboid-first-ever-cryptojacking-worm-found-in-images-on-docker-hub/)

- Первый worm для Docker
- Сканировал интернет на открытые Docker API (порт 2375)
- Запускал контейнеры с криптомайнерами
- **~2000 хостов заражено**

💡 Docker имеет API для удаленного управления. Многие оставляли его открытым в интернет — червь автоматически находил такие серверы и заражал их

---

<div class="timeline">
  <div class="timeline-item current">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

**Docker Hub Breach (2019)** 📚 [SecurityWeek: Docker Hub Breach](https://www.securityweek.com/docker-hub-breach-hits-190000-accounts/)

- Компрометация 190,000 аккаунтов
- Утечка токенов GitHub и Bitbucket из образов
- Проблема: разработчики хранили секреты в образах

💡 Взломали базу данных Docker Hub (главного репозитория образов). Получили доступ к паролям и токенам для GitHub — теперь злоумышленники могли изменять код проектов жертв

---

<div class="timeline">
  <div class="timeline-item current">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

**TeamTNT Campaign (2020-2023)** 📚 [Sysdig: TeamTNT Analysis](https://www.sysdig.com/blog/real-cost-cryptomining-teamtnt/) | [Unit 42: Hildegard Malware](https://unit42.paloaltonetworks.com/hildegard-malware-teamtnt/)

- Крупнейшая кампания атак на контейнеры
- Эксплуатация открытых Docker API и Kubernetes
- Кража AWS credentials, криптомайнинг, backdoors
- **>10,000 скомпрометированных систем**

💡 Организованная группа хакеров несколько лет сканировала весь интернет в поисках неправильно настроенных контейнеров. Украденные AWS ключи давали доступ к облачным счетам компаний

---

<div class="timeline">
  <div class="timeline-item current">Инциденты</div>
  <div class="timeline-item future">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

**Вывод:** Неправильная конфигурация = реальная угроза

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# Векторы атаки

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## 🏹 Вектор #1: Privileged контейнеры

**Проблема:** `--privileged` флаг отключает большинство защит

```bash
docker run --privileged -it ubuntu /bin/bash
```

**Что получает атакующий:**

- Доступ ко всем устройствам хоста (`/dev`)
- Все Linux capabilities
- Возможность монтирования файловых систем

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

В каких случаях вообще контейнер может быть запущен небезопасно, зачем запускать с privelleged?

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

В каких случаях вообще контейнер может быть запущен небезопасно, зачем запускать с privelleged?

> - Разработка и тестрирование,
> - работа с ядром и драйверами,
> - исследование уязвимостей

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Подготовка: Запуск заданий

```bash
# Задание 1: Privileged контейнер
echo "FLAG{pr1v1leged_m0de_danger}" > /root/flag.txt

docker run -it --privileged --name ctf-task1 ubuntu
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## ✅ Задание 1: Побег через privileged режим

**Сценарий:** Вы получили доступ к контейнеру, запущенному с `--privileged`

**Цель:** Прочитать файл `/root/flag.txt` с хоста

**Пошаговый план:**

1. Установите утилиту для работы с дисками`apt-get update` `apt-get -y install fdisk`
2. Проверьте доступные диски: `fdisk -l`,
3. Найдите основной диск хоста (обычно `/dev/sda1` или `/dev/vda1`)
4. Смонтируйте диск: `mkdir /mnt/host && mount /dev/sda1 /mnt/host`
5. Теперь ФС хоста доступна: `cat /mnt/host/root/flag.txt`

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

Как злоумышленник может получить доступ к конетейнеру?

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

Как злоумышленник может получить доступ к конетейнеру?

> - уязвимости в приложениях внутри контейнера
> - уязвимости в пакетах/библиотеках (Supply Chain Attack)
> - утечка credentials

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## 🏹 Вектор #2: Небезопасные монтирования

**Проблема:** Монтирование критичных директорий хоста

```bash
docker run -v /:/host -it ubuntu /bin/bash
docker run -v /var/run/docker.sock:/var/run/docker.sock
```

**Риски:**

- Прямой доступ к файлам хоста
- Возможность управления Docker API
- Запуск новых привилегированных контейнеров

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

В каких случаях может быть примонтирован docker socket?

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

В каких случаях может быть примонтирован docker socket?

> - В ci/cd, чтобы собирать образы
> - В инструментах управления контейнерами (интерфейсах)
> - Для мониторинга (Prometheus)

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Подготовка: Запуск заданий

```bash
# Задание 2: Docker socket
docker run -it -v /var/run/docker.sock:/var/run/docker.sock \
  --name ctf-task2 docker:latest sh

```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## ✅ Задание 2: Docker socket escape

**Сценарий:** В контейнере примонтирован `/var/run/docker.sock`

**Цель:** Получить shell на хосте с root правами

**Пошаговый план:**

1. Посмотрите запущенные контейнеры: `docker ps`
2. Запустите privileged контейнер с монтированием хост-системы:
   ```bash
   docker run -it --privileged -v /:/host alpine chroot /host
   ```
3. Найдите флаг: `cat /root/flag.txt`
4. Найдите новый контейнер на хосте

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

Как мы полчуили доступ к /root/flag.txt, мы ведь не монтировали его внутрь первого контейнера?

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

Как мы полчуили доступ к /root/flag.txt, мы ведь не монтировали его внутрь первого контейнера?

> Docker socket — это не файл с данными, а API для управления Docker daemon. Когда ты выполняешь docker run ... из контейнера A:

> 1.  Команда идёт через сокет к Docker daemon на хосте
> 2.  Daemon работает с root-правами на хосте
> 3.  Daemon создаёт новый контейнер B (не внутри A, а рядом с ним!)
> 4.  В контейнер B монтируется -v /:/host — вся файловая система хоста
> 5.  chroot /host делает корнем ФС хоста

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## 🏹 Вектор #3: Misconfigured capabilities

**Проблема:** Излишние capabilities

Опасные capabilities:

- `CAP_SYS_ADMIN` - административные операции
- `CAP_SYS_PTRACE` - отладка процессов
- `CAP_SYS_MODULE` - загрузка модулей ядра
- `CAP_DAC_READ_SEARCH` - обход проверок чтения

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

В каких случаях вообще контейнер могут запустить с --cap-add=SYS_ADMIN, --cap-add=DAC_READ_SEARCH?

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❓ Вопрос

В каких случаях вообще контейнер могут запустить с --cap-add=SYS_ADMIN, --cap-add=DAC_READ_SEARCH?

> - Docker-in-Docker (DinD). GitLab CI runners, Jenkins agents часто требуют это для сборки образов.
> - Бэкап-агенты (Restic, Borg, Bacula)
> - NFS-серверы
> - Аудит/мониторинг/антивирусы
> - Файловая индексация и поиск
> - Rsync/синхронизация данных

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Подготовка: Запуск заданий

```bash
# Запускаем контейнер с DAC_READ_SEARCH
services:
  vulnerable-dac:
    build: .
    container_name: vuln-dac
    cap_add:
      - DAC_OVERRIDE      # Игнорирует права на запись
      - DAC_READ_SEARCH   # Игнорирует права на чтение
    volumes:
      - /etc:/host-etc:ro  # Монтируем /etc хоста (типичная ошибка!)
    stdin_open: true
    tty: true
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## ✅ Задание 3: DAC_READ_SEARCH

**Сценарий:** Контейнер запущен с `--cap-add=DAC_READ_SEARCH`

**Цель:** Получить доступ к привилегированному файлу хоста

**Пошаговый план:**

1. Запускаем уязвимый контейнер `docker compose up -d` и подключаемся к консоли контейнера `docker compose exec vulnerable-dac bash`
2. Читаем файл, к которому нет доступа `cat /host-etc/shadow`
3. ! Вне контейнера я не могу получить доступ к этоу файлу `cat /etc/shadow`. Членство в группе docker ≈ root-доступ к хосту. Это не баг, это особенность архитектуры

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item current">Векторы атаки</div>
  <div class="timeline-item future">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Краткая сводка: 3 вектора побега

| Вектор              | Требования                 | Сложность | Защита                             |
| ------------------- | -------------------------- | --------- | ---------------------------------- |
| **Privileged**      | `--privileged` флаг        | Легко     | Никогда не использовать            |
| **Docker socket**   | Монтирование sock          | Легко     | Не монтировать без необходимости   |
| **DAC_READ_SEARCH** | Capability DAC_READ_SEARCH | Средне    | `--cap-drop=ALL`, минимальные caps |

**Общий паттерн:** Неправильная конфигурация → доступ к ресурсам хоста → побег

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

Суть защиты: даже если приложение взломают, контейнер должен быть настолько ограничен, что дальше двигаться некуда.

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# Лучшие практики защиты

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## ❌ Безопасный Dockerfile: Плохой пример

❓ Какие здесь есть уязвимости?

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y python3 curl

ENV API_KEY="sk-1234567890abcdef"

# Установка всех пакетов
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .

CMD ["python3", "app,py"]
```

---

**Проблемы:** latest тег, root, большой образ, секреты в образе

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## ✅ Безопасный Dockerfile: Хороший пример

```dockerfile
FROM python:3.11-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Multi-stage: минимальный финальный образ
FROM python:3.11-alpine

# Non-root пользователь
RUN addgroup -g 1000 appgroup && \
    adduser -D -u 1000 -G appgroup appuser
...
USER appuser

CMD ["python3", "app.py"]
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Безопасный Dockerfile

Попробуем!

```
docker run -it --privileged --name ctf-task1 --user appuser:appuser ubuntu-unpriveleged-user
```

1. Установите утилиту для работы с дисками`apt-get update` `apt-get -y install fdisk`
2. Проверьте доступные диски: `fdisk -l`

> Уменьшение поверхности атаки — даже внутри контейнера непривилегированный пользователь не может менять системные файлы, устанавливать пакеты, менять сетевые настройки.

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## Безопасный Dockerfile: Лучшие практики

✅ Конкретные теги версий (`python:3.11-alpine`, не `latest`)
✅ Минимальные базовые образы (`alpine`, `distroless`)
✅ Multi-stage builds для уменьшения размера
✅ Non-root пользователя
✅ `.dockerignore` для исключения чувствительных файлов

**Избегайте:**

❌ `latest` тега
❌ Запуска от root
❌ Хранения секретов в образе (используйте secrets management)
❌ Установки ненужных пакетов

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

## ✅ docker-compose.yml: Хороший пример

```yaml
services:
  app:
    image: myapp:1.2.3 # Конкретная версия
    read_only: true # Read-only filesystem
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE # Только необходимые capabilities
    security_opt:
      - no-new-privileges:true
    user: "1000:1000" # Non-root
    volumes:
      - ./data:/app/data:ro # Read-only монтирование
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ❌ Все capabilities: Плохой пример

Пробуем!

```yaml
# docker-compose.vulnerable.yml
services:
  vulnerable-app:
    image: alpine
    container_name: vulnerable-app
    command: sh -c "sleep infinity"
    # No capability restrictions - container has default ~14 capabilities
```

---

```bash
docker compose -f docker-compose.vulnerable.yml up -d

# Подключимся к консоли контейнера
docker exec -it vulnerable-app sh

# Проверим capabilities внутри контейнера
cat /proc/1/status | grep Cap

# Контейнер может изменить владельца файла (CAP_CHOWN)
touch /tmp/test
chown 1000:1000 /tmp/test
ls -la /tmp/test

# Контейнер может использовать привилегированный порт
(CAP_NET_BIND_SERVICE)
apk add socat && socat TCP-LISTEN:80,fork STDOUT &
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item current">Лучшие практики</div>
  <div class="timeline-item future">Инструменты</div>
</div>

# ✅ Ограниченные capabilities: Хороший пример

```yaml
# docker-compose.secure.yml
services:
  secure-app:
    image: alpine
    container_name: secure-app
    command: sh -c "sleep infinity"
    cap_drop:
      - ALL # Сначала отключаем всё
    cap_add:
      - NET_BIND_SERVICE # Включаем только то, что требуется
    security_opt:
      - no-new-privileges:true
```

---

```bash
docker compose -f docker-compose.secure.yml up -d

# Подключимся к консоли контейнера
docker exec -it secure-app sh

# Проверим capabilities внутри контейнера
cat /proc/1/status | grep Cap

# Контейнер НЕ может изменить владельца файла (CAP_CHOWN)
touch /tmp/test
chown 1000:1000 /tmp/test
ls -la /tmp/test

# Контейнер может использовать привилегированный порт
(CAP_NET_BIND_SERVICE)
apk add socat && socat TCP-LISTEN:80,fork STDOUT &

```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

# Инструменты диагностики

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Инструменты диагностики

**Зачем сканировать образы?**

- Уязвимости в базовых образах и зависимостях
- Устаревшие пакеты с известными CVE
- Secrets и чувствительные данные в слоях
- Misconfiguration в Dockerfile

**Основные инструменты:**

- **Trivy** - комплексный сканер (наш фокус)
- **Docker Scout** - встроенный инструмент Docker
- Snyk, Anchore, Clair

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Trivy: Универсальный сканер

**Что сканирует Trivy:**

- Образы контейнеров (OS пакеты и зависимости приложений)
- Файловые системы и rootfs
- Git репозитории
- Kubernetes манифесты
- Terraform/CloudFormation

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Trivy: Практическое использование

**Сканирование образа:**

```bash
# Базовое сканирование
trivy image nginx:latest

# Только критичные и высокие уязвимости
trivy image --severity CRITICAL,HIGH nginx:latest

# Вывод в JSON для автоматизации
trivy image -f json -o results.json nginx:latest

# Игнорирование unfixed уязвимостей
trivy image --ignore-unfixed nginx:latest
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Docker Scout: Встроенный анализ

```bash
# Анализ локального образа
docker scout cves nginx:latest
# Сравнение с рекомендациями
docker scout recommendations nginx:latest
# Быстрый обзор
docker scout quickview nginx:latest
```

**Преимущества Scout:**

- Встроен в Docker Desktop и CLI
- Интеграция с Docker Hub
- Рекомендации по обновлению базовых образов
- Policy enforcement для CI/CD

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Dockle: Сканирование на best-practices

```
dockle ubuntu:latest

dockle ubuntu-unpriveleged-user
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Docker Bench for Security: Проверка на best-practices

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh
```

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Методы защиты: Краткий чеклист

✅ Никогда не используйте `--privileged` в продакшене
✅ Минимизируйте capabilities: `--cap-drop=ALL`
✅ Не монтируйте docker.sock без необходимости
✅ Используйте read-only файловую систему где возможно

✅ Регулярно сканируйте образы (Trivy/Scout в CI/CD)
✅ Используйте минимальные базовые образы (alpine, distroless)
✅ Обновляйте Docker и образы
✅ Не храните секреты в образах

<!-- --- -->

<!--
<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Полезные инструменты

**Тестирование безопасности:**

- [CDK (Container Duck)](https://github.com/cdk-team/CDK) - набор эксплойтов
- [botb](https://github.com/brompwnie/botb) - автоматический поиск путей побега
- Docker Bench Security - CIS benchmark проверка

**Сканирование:**

- Trivy - универсальный сканер
- Docker Scout - встроенный в Docker
- Grype, Snyk - альтернативы

--- -->
<!--
<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item current">Инструменты</div>
</div>

## Ресурсы для дальнейшего изучения

**Документация и стандарты:**

- [Docker Security Docs](https://docs.docker.com/engine/security/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

**Практика:**

- HackTheBox - Docker escape challenges
- TryHackMe - Container Security room
- [Play with Docker](https://labs.play-with-docker.com/) - песочница -->

<!-- ---

**Инструменты:**

- Trivy: https://github.com/aquasecurity/trivy
- Docker Bench: https://github.com/docker/docker-bench-security -->

---

<div class="timeline">
  <div class="timeline-item done">Инциденты</div>
  <div class="timeline-item done">Векторы атаки</div>
  <div class="timeline-item done">Лучшие практики</div>
  <div class="timeline-item done">Инструменты</div>
</div>

## Заключение

**Ключевые выводы:**

1. Контейнеры не являются безопасными по умолчанию
2. Privileged режим и docker.sock - критичные векторы атак
3. Capabilities требуют тщательной настройки
4. Регулярное сканирование образов обязательно (Trivy/Scout)
5. Defense in depth - используйте несколько уровней защиты

**Важно:** Все показанные техники только для легального тестирования в контролируемых окружениях!

---

## Спасибо за участие!

**Вопросы и обсуждение**

Практикуйтесь безопасно!
