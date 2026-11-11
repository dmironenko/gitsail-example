<p align="right">
  <img align="left" src="https://hits.sh/github.com/dmironenko/gitsail-example.svg?style=flat&amp;label=views&amp;color=007ec6" alt="views" />
</p>
<br clear="left" />

# Пример GitSail

Готовый репозиторий для GitOps с GitSail. Он разворачивает API из исходников, сайт и NGINX с Certbot. Отдельные экземпляры NGINX обслуживают production и stage. Общий прокси выбирает окружение по cookie.

## Выберите задачу

- [Запустить пример локально](#run-example)
- [Проверить изменение и перенести его в production](#promote-change)
- [Добавить персональное окружение](#personal-environment)
- [Открыть настройки NGINX и HTTPS](apps/ingress/nginx/vars/base.yaml)
- [Найти рецепт или справочник полей](#gitsail-help)

<a id="run-example"></a>
## Запустить пример

Нужны Git и Docker Engine или Docker Desktop. Установите GitSail:

```sh
curl -fsSL https://raw.githubusercontent.com/dmironenko/gitsail/v1.0.0/install.sh | sh -s -- v1.0.0
```

Склонируйте репозиторий и запустите пример:

```sh
git clone --branch master https://github.com/dmironenko/gitsail-example.git
cd gitsail-example
git branch dev master
cp host.yaml.example ../gitsail.local.yaml
install -m 600 demo.agekey ../gitsail-example.agekey
gitsail apply --config ../gitsail.local.yaml --wait --timeout 15m
```

При первой сборке GitSail скачает и скомпилирует API. Ingress слушает `127.0.0.1:18080` и `127.0.0.1:18443`. Проверьте, что порты свободны. Затем проверьте production и stage:

```sh
curl -ki https://localhost:18443/api/
curl -ki --cookie 'gitsail_stage=gitsail-example:stage' https://localhost:18443/api/
gitsail run --config ../gitsail.local.yaml
```

В ответах есть заголовки `X-GitSail-Environment: prod` и `stage`. Флаг `-k` принимает локальный сертификат. Ctrl-C останавливает GitSail, контейнеры продолжают работать. Пути в конфигурации хоста отсчитываются от `gitsail.local.yaml`. Пример проверяет локальный Git каждые две секунды. Для удаленного репозитория задайте `source.poll_interval: 1m`.

Ключ `demo.agekey` и токен stage опубликованы как демонстрационные значения. Используйте их только локально. Замените оба значения перед запуском реального сервиса.

<a id="promote-change"></a>
## Проверить изменение и перенести его в production

Измените приложение в ветке `dev` и создайте коммит:

```sh
git switch dev
# Измените apps/demo/api/vars/base.yaml.
git add apps/demo/api/vars/base.yaml
git commit -m 'feat: update API settings'
```

Проверьте stage и перенесите изменение в production:

```sh
git switch master
git merge dev
```

Локальный режим читает коммиты без push. Удаленный режим требует push. Изменения только в конфигурации тоже применяются. Для отката отмените коммит. Чтобы собрать другую версию исходников, измените `build.tag`. Если входные данные не изменились, GitSail использует готовый образ.

<a id="personal-environment"></a>
## Добавить окружение

Создайте ветку и добавьте ее в конфигурацию хоста:

```sh
git branch users/alice dev
```

```yaml
source:
  branches:
    prod: master
    stage: dev
    alice: users/alice
```

Для выбора окружения передайте cookie `gitsail_stage=<token>:alice`. Используйте токен из `app.stage_token`. Окружение получает отдельные проекты Compose и тома. Данные production туда не копируются. Имя сервиса `nginx` получает суффикс `alice`. Если файла `stage.yaml` нет, приложение не запускается вне production. Удаление окружения удаляет контейнеры и сохраняет тома и образы.

<a id="gitsail-help"></a>
## Найти рецепт или справочник

- [Установить GitSail](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/install.md)
- [Создать приложение](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/create-an-app.md)
- [Разрешить приложениям отправлять запросы друг другу](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/connect-apps.md)
- [Создать stage или персональное окружение](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/stage.md)
- [Передать переменные окружения](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/environment-variables.md)
- [Добавить настройки приложения и параметры `x-gitsail`](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/application-config.md)
- [Отключить приложение или окружение](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/disable-app.md)
- [Зашифровать и использовать секреты](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/secrets.md)
- [Собрать приложение из исходников](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/build-from-source.md)
- [Найти все поля приложения](https://github.com/dmironenko/gitsail/blob/master/docs/ru/reference/app-configuration.md)
- [Найти все поля конфигурации хоста](https://github.com/dmironenko/gitsail/blob/master/docs/ru/reference/host-configuration.md)
- [Посмотреть команды CLI](https://github.com/dmironenko/gitsail/blob/master/docs/ru/reference/cli.md)
- [Установить GitSail на сервер](https://github.com/dmironenko/gitsail/blob/master/docs/ru/recipes/server.md)

В каталоге `apps/` находятся настройки приложений для production и stage. `host.yaml.example` служит шаблоном. Скопируйте его за пределы Git. Ключ `demo.agekey` открыт и не подходит для защиты настоящих секретов. Только `apps/ingress/nginx` публикует порты хоста. Остальные приложения общаются по сети установки.

## Участие в разработке

Поддерживайте пример в рабочем состоянии. Документация должна быть на русском и английском языках. Не добавляйте конфигурацию хоста и отрендеренное состояние в Git. Используйте два пробела в YAML. Сохраняйте порядок блоков `build`, `app`, `compose`. Настройки приложения и секреты помещайте в `app`. Настройки Compose помещайте в `compose`. Параметры GitSail задавайте в `compose.x-gitsail`.

Проверяйте изменения GitSail на отдельной копии с отдельным каталогом состояния и свободными портами.
