# <p style="text-align:center;"> **Домашнее задание к занятию «GitLab»  <Фамилия Имя>** </p>

## Задание №1

**Установка GitLab и регистрация ранера**

```
- Обновление пакетов и репозитория скриптом
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

- Устанавливаем Gitlab
apt install gitlab-ce

- Задаем имя в конфигурации GitLab
nano /etc/gitlab/gitlab.rb
external_url 'http://gitlab.netology.ru'

- Первый запуск GitLab
gitlab-ctl reconfigure

- Получаем root-пароль
cat /etc/gitlab/initial_root_password | grep Password:

- Запускаем ранер в докере 

docker run -ti --rm --name gitlab-runner \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest register

- Вводим по очереди то что просит ранер
http://gitlab.netology.ru/
<токен>
new
<enter>
Enter an executor: docker
golang:1.17

- Запускаем экзепляр ранера
docker run -d --name gitlab-runner --restart always      --network host      -v /srv/gitlab-runner/config:/etc/gitlab-runner      -v /var/run/docker.sock:/var/run/docker.sock      gitlab/gitlab-runner:latest

- Добавляем в конфигурацию ранера
  [runners.docker]
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    extra_hosts = ["gitlab.netology.ru:192.168.1.82"]

- Перезапускаем ранер для применения настроек
docker exec -it gitlab-runner gitlab-runner restart

```
![Новый проект с зарегистрированным ранером](https://github.com/AlexandeAbel/8-3-hv/img/runner_project.png)


## Задание №2

- Клонируем репозиторий
```
git clone https://github.com/netology-code/sdvps-materials.git
```

- Меняем ссылку репозитория на наш
```
git remote set-url origin http://gitlab.netology.ru/root/netology
```

- Создаем ветку
```
git checkout -b changed
```

- Добавляем, коммитим и отправляем изменения в GitLab
```
git add .
git commit -m "test"
git push origin changed
```

![Новая ветка в GitLab](https://github.com/AlexandeAbel/8-3-hv/img/pushed_branch.png)

- Создаем файл .gitlab-ci.yml с кодом:
```
stages:
  - test
  - build

test:
  stage: test
  image: golang:1.17
  script: 
   - go test .
  tags:
    - new


build:
  stage: build
  image: docker:latest
  script:
   - docker build .
  tags:
    - new
```

![Успешный пайплайн](https://github.com/AlexandeAbel/8-3-hv/img/success_pipeline_1.png)

![Успешный пайплайн](https://github.com/AlexandeAbel/8-3-hv/img/success_pipeline_2.png)

![Успешный пайплайн](https://github.com/AlexandeAbel/8-3-hv/img/success_pipeline_3.png)

## Задание №3

- Добавляем в задачу test следующий код, чтобы она запускалась только при изменении файлов .go

``` 
only:
    changes:
      - "*.go"
```

- Добавляем в задачу build следующий код, чтобы она не ждала результатов тестов

```
needs: []
```

- Итоговый пайп:

```
stages:
  - test
  - build

test:
  stage: test
  image: golang:1.17
  script: 
   - go test .
  tags:
    - new
  only:
    changes:
      - "*.go"


build:
  stage: build
  image: docker:latest
  script:
   - docker build .
  tags:
    - new
  needs: []
```

- С помощью тега определяем какой ранер использовать в работе
```
tags:
    - new
```
![Третье задание](https://github.com/AlexandeAbel/8-3-hv/img/task_3.png)