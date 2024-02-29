# Homework2

## Задание
1. Переписать test stage для тестирования docker-a
2. Достаточно проверить, что docker контейнер на базе нашего собранного образа в предыдущей job запускается. 

## .gitlab-ci.yml

```yaml
image: busybox:latest

stages:
  - build
  - test
  - deploy
  - stop
  - stop previous jobs

variables:
  IMAGE_TAG: $CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA
  CI_DEBUG_TRACE: "true"
include:
  local: smoke.gotlab-ci.yml

cache:
  key:
    files:
      - composer.lock
  paths:
    - vendor/

before_script:
  - echo "Before script section"
  - echo "For example, you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example, you might do some cleanup here"

build1:
  stage: build
  script:
    - echo "Do your build here"
    - echo "one" >> house.txt
    # - mkdir -p vendor/
    # - echo "build" > vendor/hello.txt

docker build:
  image: docker:latest
  stage: build
  services:
    - docker:dind 
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG

test1:
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example, run a test suite"
    - sleep 60



deploy to stage:
  stage: deploy
  variables:
    TARGET_ENV: staging
  script:
    - echo "Do you deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
  only:
    - main
  environment:
    name: staging
    on_stop: stop to staging
    auto_stop_in: 1 day

stop to staging:
  stage: stop
  variables:
    TARGET_ENV: staging
  script:
    - echo "STOP ${TARGET_ENV}"
  only:
    - main
  when: manual
  environment:
    name: staging
    action: stop

deploy to prod:
  stage: deploy
  variables:
    TARGET_ENV: prod
  script:
    - echo "Do you deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
  only:
    - main
  when: manual
  environment:
    name: prod

cancel:
  stage: stop previous jobs
  image: everpeace/curl-jq
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "main" ]
        then
          (
            echo "Cancel old pipelines from the same branch except last"
            OLD_PIPELINES=$( curl -s -H "PRIVATE-TOKEN: $RUNNER_TOKEN" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines?ref=${CI_COMMIT_REF_NAME}&status=running" \
                  | jq '.[] | .id' | tail -n +2 )
                  for pipeline in ${OLD_PIPELINES}; \
                      do echo "Killing ${pipeline}" && \
                        curl -s --request POST -H "PRIVATE-TOKEN: ${RUNNER_TOKEN}" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines/${pipeline}/cancel"; done
          ) || echo "Canceling old pipelines (${OLD_PIPELINES}) failed"
      fi


```

![alt text](<FOTOS/Снимок экрана_1.png>)

# Вариант №2 

```yaml
image: busybox:latest

stages:
  - build
  - test
  - deploy
  - stop
  - stop previous jobs

variables:
  IMAGE_TAG: $CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA
  CI_DEBUG_TRACE: "true"
include:
  local: smoke.gotlab-ci.yml

cache:
  key:
    files:
      - composer.lock
  paths:
    - vendor/

before_script:
  - echo "Before script section"
  - echo "For example, you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example, you might do some cleanup here"

build1:
  stage: build
  script:
    - echo "Do your build here"
    - echo "one" >> house.txt
    # - mkdir -p vendor/
    # - echo "build" > vendor/hello.txt

docker build:
  image: docker:latest
  stage: build
  services:
    - docker:dind 
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG

# test1:
#   stage: test
#   script:
#     - echo "Do a test here"
#     - echo "For example, run a test suite"
#     - sleep 60

test1:
  stage: test
  script:
    - echo "Тестируем Docker"
    - docker run $IMAGE_TAG
  only:
    - main


deploy to stage:
  stage: deploy
  variables:
    TARGET_ENV: staging
  script:
    - echo "Do you deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
  only:
    - main
  environment:
    name: staging
    on_stop: stop to staging
    auto_stop_in: 1 day

stop to staging:
  stage: stop
  variables:
    TARGET_ENV: staging
  script:
    - echo "STOP ${TARGET_ENV}"
  only:
    - main
  when: manual
  environment:
    name: staging
    action: stop

deploy to prod:
  stage: deploy
  variables:
    TARGET_ENV: prod
  script:
    - echo "Do you deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
  only:
    - main
  when: manual
  environment:
    name: prod

cancel:
  stage: stop previous jobs
  image: everpeace/curl-jq
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "main" ]
        then
          (
            echo "Cancel old pipelines from the same branch except last"
            OLD_PIPELINES=$( curl -s -H "PRIVATE-TOKEN: $RUNNER_TOKEN" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines?ref=${CI_COMMIT_REF_NAME}&status=running" \
                  | jq '.[] | .id' | tail -n +2 )
                  for pipeline in ${OLD_PIPELINES}; \
                      do echo "Killing ${pipeline}" && \
                        curl -s --request POST -H "PRIVATE-TOKEN: ${RUNNER_TOKEN}" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines/${pipeline}/cancel"; done
          ) || echo "Canceling old pipelines (${OLD_PIPELINES}) failed"
      fi


```
![alt text](<FOTOS/Снимок экрана_2.png>)

# Вариант 3 

```yaml
image: busybox:latest

stages:
  - build
  - test
  - deploy
  - stop
  - stop previous jobs

variables:
  IMAGE_TAG: $CI_COMMIT_BRANCH-$CI_COMMIT_SHORT_SHA
  CI_DEBUG_TRACE: "true"
include:
  local: smoke.gotlab-ci.yml

cache:
  key:
    files:
      - composer.lock
  paths:
    - vendor/

before_script:
  - echo "Before script section"
  - echo "For example, you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example, you might do some cleanup here"

build1:
  stage: build
  script:
    - echo "Do your build here"
    - echo "one" >> house.txt
    # - mkdir -p vendor/
    # - echo "build" > vendor/hello.txt

docker build:
  image: docker:latest
  stage: build
  services:
    - docker:dind 
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG

# test1:
#   stage: test
#   script:
#     - echo "Do a test here"
#     - echo "For example, run a test suite"
#     - sleep 60

test1:
  stage: test
  script:
    - echo "Тестируем Docker"
    - docker run $IMAGE_TAG
  only:
    - main


deploy to stage:
  stage: deploy
  variables:
    TARGET_ENV: staging
  script:
    - echo "Do you deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
  only:
    - homework2
  environment:
    name: staging
    on_stop: stop to staging
    auto_stop_in: 1 day

stop to staging:
  stage: stop
  variables:
    TARGET_ENV: staging
  script:
    - echo "STOP ${TARGET_ENV}"
  only:
    - homework2
  when: manual
  environment:
    name: staging
    action: stop

deploy to prod:
  stage: deploy
  variables:
    TARGET_ENV: prod
  script:
    - echo "Do you deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
  only:
    - homework2
  when: manual
  environment:
    name: prod

cancel:
  stage: stop previous jobs
  image: everpeace/curl-jq
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "main" ]
        then
          (
            echo "Cancel old pipelines from the same branch except last"
            OLD_PIPELINES=$( curl -s -H "PRIVATE-TOKEN: $RUNNER_TOKEN" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines?ref=${CI_COMMIT_REF_NAME}&status=running" \
                  | jq '.[] | .id' | tail -n +2 )
                  for pipeline in ${OLD_PIPELINES}; \
                      do echo "Killing ${pipeline}" && \
                        curl -s --request POST -H "PRIVATE-TOKEN: ${RUNNER_TOKEN}" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines/${pipeline}/cancel"; done
          ) || echo "Canceling old pipelines (${OLD_PIPELINES}) failed"
      fi

```

![alt text](<FOTOS/Снимок экрана_3.png>)