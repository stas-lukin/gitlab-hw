**Студент:** Лукин Станислав
**Дата:** 2026-03-01

## Задание 1. Развертывание GitLab и настройка runner

### Инфраструктура
Для выполнения задания были созданы 3 виртуальные машины в Яндекс.Облаке с помощью Terraform:
- **GitLab сервер**: 93.77.190.230 (2 vCPU, 4 RAM, 20 ГБ диск)
- **GitLab Runner сервер**: 93.77.185.120 (2 vCPU, 2 RAM, 15 ГБ диск)
- **Backup сервер**: 89.169.139.146 (2 vCPU, 2 RAM, 15 ГБ диск)

### Установка GitLab
GitLab был установлен на первом сервере с помощью официального пакета:

```bash
# Установка зависимостей
sudo apt update
sudo apt install -y curl openssh-server ca-certificates

# Добавление репозитория GitLab
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

# Установка GitLab CE
sudo apt install -y gitlab-ce

# Настройка внешнего URL
sudo sed -i "s/external_url 'http:\/\/gitlab.example.com'/external_url 'http:\/\/93.77.190.230'/" /etc/gitlab/gitlab.rb

# Применение конфигурации
sudo gitlab-ctl reconfigure
### Настройка GitLab Runner
На втором сервере (93.77.185.120) был установлен и зарегистрирован GitLab Runner:

```bash
# Установка Docker
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER

# Установка GitLab Runner
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start

# Регистрация runner
sudo gitlab-runner register
**Параметры регистрации:**
- **URL**: http://93.77.190.230
- **Token**: glrt-VBvfJfNYGlhEG7HBYYkdfm86MQpwOjEKdDozCnU6MQ8.01.171ob1yne
- **Описание**: docker-runner
- **Теги**: docker, linux
- **Executor**: docker
- **Образ по умолчанию**: alpine:latest

### Результат настройки runner
![GitLab Runner](screenshots/runner.png)

## Задание 2. CI/CD пайплайн

### Файл .gitlab-ci.yml
```yaml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  tags:
    - docker
    - linux
  script:
    - echo "Building application..."
    - mkdir -p build
    - echo "Hello World" > build/index.html
  artifacts:
    paths:
      - build/

test-job:
  stage: test
  tags:
    - docker
    - linux
  script:
    - echo "Testing application..."
    - test -f build/index.html
  dependencies:
    - build-job

deploy-job:
  stage: deploy
  tags:
    - docker
    - linux
  script:
    - echo "Deploying application..."
    - ls -la build/
  dependencies:
    - build-job
  only:
    - main
### Результат выполнения пайплайна
![GitLab CI Pipeline](screenshots/pipeline.png)

### Содержимое .gitlab-ci.yml в репозитории
![GitLab CI Configuration](screenshots/gitlab-ci.png)

## Ссылки
- **Репозиторий с решением**: https://github.com/stas-lukin/gitlab-hw
- **GitLab сервер**: http://93.77.190.230

## Вывод
Все задания выполнены в полном объеме, результаты подтверждены скриншотами.
