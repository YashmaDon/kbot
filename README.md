# Task12 GitHub Actions 

## Автоматизуйте повний цикл CICD для свого сервісу бота.

1. Додамо новий проект на [Google Cloud](https://console.cloud.google.com/projectcreate) з назвою `vit-um` та переключаємось на нього:  

2. Дозволяємо Google Container Registry API(https://console.cloud.google.com/marketplace/product/google/containerregistry.googleapis.com?project=kbot-407613)

3. Змінюємо в `Makefile` назву репозиторію на 
```Makefile
REGESTRY := gcr.io/vit-um
```
3. Створимо імідж контейнеру нашого застосунку:
```sh
$ make image                
docker build . -t gcr.io/vit-um/kbot:v.CICD-38f501f-amd64  --build-arg TARGETARCH=amd64 

$ docker images
REPOSITORY         TAG                   IMAGE ID       CREATED             SIZE
gcr.io/vit-um/kbot v.CICD-38f501f-amd64  6643b9fb35f3   About an hour ago   11.3MB

$ docker rmi -f 6643b9fb35f3
Untagged: gcr.io/vit-um/kbot:v.CICD-38f501f-amd64
Deleted: sha256:6643b9fb35f3ffdeaac5e287bc4ee556fc8cc2981513a0f840fc767686beb805

$ git tag v1.3.0


$ git branch develop 
$ g

```
4. При спробі запушити та виправити помилки, що отримаємо:
```sh
$ make push 
docker push gcr.io/vit-um/kbot.git:v.CICD-38f501f-amd64 
The push refers to repository [gcr.io/vit-um/kbot.git]
ea325a7b7ffa: Preparing 
36bdcea58c7c: Preparing 
unauthorized: You don't have the needed permissions to perform this operation, and you may have invalid credentials. To authenticate your request, follow the steps in: https://cloud.google.com/container-registry/docs/advanced-authentication
make: *** [Makefile:67: push] Error 1

sudo usermod -a -G docker ${USER}
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-456.0.0-linux-x86_64.tar.gz

tar -xf google-cloud-cli-456.0.0-linux-x86_64.tar.gz
 ./google-cloud-sdk/install.sh

gcloud version
Google Cloud SDK 456.0.0
bq 2.0.99
bundled-python3-unix 3.11.6
core 2023.12.01
gcloud-crc32c 1.0.0
gsutil 5.27

gcloud init # та обираємо в інтерактивному меню щойно заведений проект `[3] vit-um`погоджуємось на авторизацію та проходимо її за наданим посиланням.

$ gcloud config set project vit-um
Updated property [core/project].

$ gcloud auth configure-docker
Adding credentials for all GCR repositories.
WARNING: A long list of credential helpers may cause delays running 'docker build'. We recommend passing the registry name to configure only the registry you are using.
After update, the following will be written to your Docker config file located at [/root/.docker/config.json]:
 {
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}

Do you want to continue (Y/n)?  Y

Docker configuration file updated.

make push                   
docker push gcr.io/vit-um/kbot.git:v.CICD-38f501f-amd64 
The push refers to repository [gcr.io/vit-um/kbot.git]
ea325a7b7ffa: Pushed 
36bdcea58c7c: Pushed 


3. Дозволяємо [Repositories - Settings](https://console.cloud.google.com/gcr/settings?project=kbot-407613) контейнеру стати публічним:


Робочий процес GitHub Actions описується в каталозі `.github` в корні репозиторію, в якій розташуємо каталог `workflows`, де створимо `cicd.yaml` з кодом Pipeline. Зазвичай у кожного action є окремий репозиторій версії та [документація](https://github.com/actions/checkout#checkout-v4)

```yaml
# Перша секція в файлі не обов'язкова та описує назву  Pipeline:
name: KBOT-CICD

# Визначимо подію, яка запустить виконання workflow процесу.
on: push
# блок задач який налаштовано на запуск завдання на новій ВМ з останньою версією ubuntu
jobs:
  ci:
    name: CI
    runs-on: ubuntu-latest
# перший крок задачі дозволить запускати на ранері репозиторій із скриптами для виконання дії. 
    steps:
      - name: Checkout
        uses: actions/checkout@v3
# наступним кроком запустимо команду make test, зробимо коміт та подивимось як це працює
      - name: Run test
        run: make test

```
Зайдемо на [github.com](https://github.com/vit-um/kbot/actions) в розділ Actions де знайдемо наш файл `workflow` та журнал подій по кожному кроку.

Для ускладнення виконання завдань нам потрібна буде авторизація на Docker hub за нашими контейнерами. Для цього скористуємось [документацією](https://github.com/docker/login-action#docker-hub) для опису параметрів.  
```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```
З паролями працюємо через спеціальний механізм зберігання секретних даних гітхабу [Actions secrets and variables](https://github.com/vit-um/devops/settings/secrets/actions), де додамо дві змінних описаних вище:

Токен потрібно згенерувати на сторінці [акаунта Docker Hub](https://hub.docker.com/settings/security) 

Далі йде крок збірки та пушу іміджу контейнеру. Тут треба ми використовуємо змінну середовища та повернемось до першого кроку, щоб вказати функцію `fetch-depth:0` з якою нам потрібно виконати `actions/checkout@v3` щоб витягнути з гітхабу не один коміт за замовчуванням, а всю історію гілок та тегів, бо ми на їх базі будемо збирати образ. На цьому кроці важливо щоб в налаштуваннях VSCode була дозволена функція передачі тегів: `Ctr+,` `followta`

```yaml

    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
# .......
     - name: Build&Push
        env:
          APP: "kbot"
          REGISTRY: ${{ vars.DOCKERHUB_USERNAME }}
        run: make image push
```
Додамо задачу безперервного розгортання. Ця задачу назвемо CD, перший крок буде відрізнятись від CI спеціальним налаштуванням - це призначення змінної `version` на базі значень тегу та короткого хешу коміту. Далі цю змінну передамо в спеціальне середовище зберігання змінних: `$GITHUB_ENV`. Це дозволить нам користуватись значенням цієї змінної на подальших кроках 
```yaml
  cd:
    name: CD
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0 
    - run: echo "VERSION=$(git describe --tags --abbrev=0)-$(git rev-parse --short HEAD)" >> $GITHUB_ENV
```
Наступним кроком змінимо значення `tag` у файлі `helm/values.yaml` на нове, що відповідає образу контейнера, що ми запушили у попередньому джобі CI. Для цього використаємо дію, тоб-то скористаємось [бібліотекою для зміни файлів у форматі yaml](https://github.com/mikefarah/yq#yq).

```yaml
    - uses: mikefarah/yq@master
      with:
        cmd: yq -i '.image.tag=strenv(VERSION)' helm/values.yaml
```
Останнім кроком буде налаштування глобальних параметрів git на спеціальні значення `github-actions`, та команда коміта з повідомленням і заливка до репозиторію
```yaml
    - run: |
        git config user.name github-actions
        git config user.email github-actions@github.com
```
Важливим моментом є вказання залежності виконання jobs. Опцією `needs: ci` ми дозволяємо виконувати задачу CD тільки в разі успішного виконання CI.

