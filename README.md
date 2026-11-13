# Итоговый проект курса «DevOps-инженер с нуля» Марина Кукушкина

## 1. Описание проекта

Проект демонстрирует создание и настройку базовой DevOps-инфраструктуры в Yandex Cloud с использованием Terraform, Ansible, Docker и GitHub Actions.

В рамках проекта мной были осуществлены:
- создание облачной инфраструктуры с помощью Terraform;
- хранение Terraform state в S3-совместимом Object Storage;
- автоматическая настройка виртуальной машины с помощью Ansible;
- контейнеризация Flask-приложения с помощью Docker;
- публикация Docker-образа в Docker Hub;
- автоматическая сборка и развёртывание приложения с помощью GitHub Actions.

## 2. Ссылки на проект

| Terraform | https://github.com/kumarina-max/terraform-yandex-cloud |
| Ansible | https://github.com/kumarina-max/ansible-yandex-cloud |
| Docker-приложение | https://github.com/kumarina-max/docker-app-yandex-cloud |
| Docker Hub | https://hub.docker.com/r/kumarina/docker-app-yandex-cloud |

## 3. Архитектура проекта

Инфраструктура включает:

- Yandex Cloud VPC — виртуальная сеть.
- Subnet — подсеть `10.10.0.0/24` в зоне `ru-central1-a`.
- Virtual Machine — Ubuntu 22.04 LTS.
- Docker — запуск приложения в контейнере.
- Docker Hub — хранение Docker-образа.
- GitHub Actions — автоматизация сборки и развёртывания.
- Object Storage — хранение Terraform state.

### Схема работы

```text
Developer
    |
    v
GitHub Repository
    |
    v
GitHub Actions
    |
    +----> Build Docker Image
    |             |
    |             v
    |         Docker Hub
    |             |
    |             v
    +-------> SSH Deploy
                  |
                  v
          Yandex Cloud VM
                  |
                  v
             Docker
                  |
                  v
          Flask Application
```

Terraform создаёт облачную инфраструктуру, Ansible устанавливает Docker, а GitHub Actions отвечает за сборку и доставку приложения.

## 4. Используемые технологии

- Yandex Cloud
- Terraform
- Ansible
- Ubuntu 22.04 LTS
- Docker
- Docker Compose
- Python 3.12
- Flask
- Docker Hub
- GitHub Actions
- Git

## 5. Развёртывание инфраструктуры

### 5.1. Предварительные требования

Перед началом необходимо установить:

- Yandex Cloud CLI;
- Terraform;
- Ansible;
- Git;
- Docker.

Также необходимы:
- аккаунт Yandex Cloud;
- сервисный аккаунт с необходимыми правами;
- SSH-ключ;
- S3-совместимый бакет для Terraform state.

### 5.2. Создание инфраструктуры

Клонирование репозитория:

```bash
git clone git@github.com:kumarina-max/terraform-yandex-cloud.git
cd terraform-yandex-cloud
```

Инициализация Terraform:

```bash
terraform init
```

Проверка конфигурации:

```bash
terraform fmt
terraform validate
```

Просмотр плана:

```bash
terraform plan
```

Создание инфраструктуры:

```bash
terraform apply
```

Получение публичного IP виртуальной машины:

```bash
terraform output -raw vm_public_ip
```

### 5.3. Настройка виртуальной машины через Ansible

Клонирование репозитория:

```bash
git clone git@github.com:kumarina-max/ansible-yandex-cloud.git
cd ansible-yandex-cloud
```

Перед запуском необходимо проверить IP-адрес в файле `inventory/hosts.ini` и при необходимости обновить его.

Проверка соединения:

```bash
ansible -i inventory/hosts.ini web -m ping
```

Установка Docker:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/install-docker.yml
```

Проверка Docker на виртуальной машине:

```bash
docker --version
docker compose version
```

## 6. Контейнеризация приложения

Исходный код Flask-приложения находится в репозитории:

https://github.com/kumarina-max/docker-app-yandex-cloud

Сборка и запуск приложения локально:

```bash
docker compose up -d --build
```

Проверка состояния:

```bash
docker ps
```

Проверка приложения:

```bash
curl http://localhost:8081/
curl http://localhost:8081/health
```

Ожидаемый ответ health endpoint:

```json
{"status":"healthy"}
```

## 7. Публикация Docker-образа

Docker-образ публикуется в Docker Hub:

`kumarina/docker-app-yandex-cloud:latest`

Ручная сборка и публикация:

```bash
docker build -t kumarina/docker-app-yandex-cloud:latest .
docker push kumarina/docker-app-yandex-cloud:latest
```

## 8. Настройка CI/CD

В репозитории Docker-приложения используется GitHub Actions.

Workflow находится в файле:

`.github/workflows/deploy.yml`

При каждом push в ветку `main` автоматически выполняются следующие действия:

1. Получение исходного кода.
2. Настройка Docker Buildx.
3. Авторизация в Docker Hub.
4. Сборка Docker-образа.
5. Публикация образа в Docker Hub.
6. Подключение к виртуальной машине по SSH.
7. Обновление контейнера приложения.

### GitHub Actions Secrets

Для работы workflow используются следующие секреты:

| Имя секрета | Назначение |
|---|---|
| `DOCKERHUB_USERNAME` | Имя пользователя Docker Hub |
| `DOCKERHUB_TOKEN` | Токен Docker Hub |
| `DEPLOY_HOST` | Публичный IP виртуальной машины |
| `DEPLOY_USER` | Пользователь виртуальной машины |
| `DEPLOY_SSH_KEY` | Приватный SSH-ключ для деплоя |

Секретные значения не хранятся в репозитории.

## 9. Проверка работоспособности

После развёртывания приложение доступно по адресу:

```text
http://<VM_PUBLIC_IP>:8081
```

Проверка health endpoint:

```text
http://<VM_PUBLIC_IP>:8081/health
```

Проверка контейнера на виртуальной машине:

```bash
docker ps
```

Проверка логов:

```bash
docker logs devops-diploma-app
```

## 10. Удаление инфраструктуры

Для удаления инфраструктуры необходимо перейти в каталог Terraform:

```bash
terraform destroy
```

Подтвердить операцию, введя `yes`.

Terraform должен удалить созданные им ресурсы: виртуальную машину, подсеть и виртуальную сеть.

S3-бакет с Terraform state создавался отдельно и должен сохраниться для повторного использования.

## 11. Повторное развёртывание

Для проверки воспроизводимости проекта необходимо:

1. Повторно выполнить `terraform apply`.
2. Получить новый публичный IP виртуальной машины.
3. Обновить IP в Ansible inventory.
4. Повторно выполнить Ansible playbook.
5. Обновить GitHub Secret `DEPLOY_HOST`, если IP изменился.
6. Запустить GitHub Actions новым push в ветку `main`.
7. Проверить доступность приложения и endpoint `/health`.

Успешное повторное развёртывание подтвердит воспроизводимость инфраструктуры и автоматизацию доставки приложения.

## 12. Результат проекта

В результате создана базовая DevOps-инфраструктура в Yandex Cloud:

- облачная сеть и виртуальная машина создаются с помощью Terraform;
- состояние Terraform хранится в Object Storage;
- Docker устанавливается с помощью Ansible;
- Flask-приложение запускается в Docker-контейнере;
- Docker-образ хранится в Docker Hub;
- GitHub Actions автоматически собирает и развёртывает приложение.

Проект демонстрирует базовый цикл Infrastructure as Code, Configuration Management, Containerization и CI/CD.
