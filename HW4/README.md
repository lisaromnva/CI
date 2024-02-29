# Homework4

## Задание
1. Сделать локальный шаблон CI и отдельный репозиторий с шаблонами, подключить их к своему основному репозиторию через include

> Прислать ссылку на gitlab_ci.yaml либо скриншот gitlab_ci.yaml test stage и скриншот успешно отработанной job-ы


в ветке homework4
## .gitlab-ci.yml
```yaml
include:
  - remote: https://gitlab.com/test4555548/4-dz/-/raw/main/remote-included-file.yml
  - local: local-included-file.yml


```

## local-included-file.yml
```yaml
local included file job:
  script:
    - echo "test"

```
![Alt text](<fotos/Снимок экрана_1.png>)
![Alt text](<fotos/Снимок экрана_2.png>)
![Alt text](<fotos/Снимок экрана_3.png>)
# Репозиторий открытый 4 dz

## .gitlab-ci.yml

```yaml
remote included file job:
  script:
    - echo "test"

```

## remote-included-file.yml
```yaml
remote included file job:
  script:
    - echo "dz4"

```
![Alt text](<fotos/Снимок экрана_4.png>)
![Alt text](<fotos/Снимок экрана_5.png>)