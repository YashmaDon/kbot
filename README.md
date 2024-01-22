# Стек моніторингу Grafana для демо-застосунку розгорнуто за допомогою docker-compose

## Середовище розгортання
- VDS з [Ubuntu 22.04.3](https://ubuntu.com/) LTS (GNU/Linux 5.15.0-91-generic x86_64) та доменним ім'ям [http://smart-home.dp.ua/](http://smart-home.dp.ua:3002/d/KhuTkQcSk/world-time-bot?orgId=1)  
- [Телеграм бот](https://t.me/umanetsvitalii_bot) 🌍 time_bot
- [Docker Compose](https://docs.docker.com/compose/)

## Підготовка коду, інструменталізація коду бота та налаштування інтерфейсу Grafana

Весь процес налаштування та інструменталізації кода телеграм бота описаний в лекційному матеріалі [за цим посиланням](https://github.com/vit-um/DevOps/wiki/%D0%9C%D0%BE%D0%BD%D1%96%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3#coding-session-k8s--otel)

## Процес розгортання

1. Клонуємо цей репозиторій з необхідними компонентами 
```sh
git clone -b opentelemetry https://github.com/vit-um/kbot
cd kbot
```
2. Готуємо змінну середовища з токеном: 
```sh
sudo -s
export TELE_TOKEN={token}
env
```

3. Розгортаємо заздалегідь налаштовані компоненти та описані в `docker-compose.yaml`:
```sh
git pull
docker-compose -f otel/docker-compose.yaml up
```
4. Впевнимось, що все піднялось:
```sh 
ubuntu-serv# docker ps -a

CONTAINER ID   IMAGE                                            COMMAND                  CREATED        STATUS        PORTS                                                        NAMES
e898dd49a3b0   umanetsvitaliy/kbot:v1.5.0-8d0aaa8-linux-amd64   "./kbot go"              30 hours ago   Up 30 hours                                                                otel-kbot-1
ec7ae74269ee   grafana/loki:2.8.2                               "/usr/bin/loki -conf…"   30 hours ago   Up 30 hours   0.0.0.0:3100->3100/tcp, :::3100->3100/tcp                    otel-loki-1
5008a49941d6   grafana/grafana:9.4.3                            "/run.sh"                30 hours ago   Up 30 hours   3000/tcp, 0.0.0.0:3002->3002/tcp, :::3002->3002/tcp          otel-grafana-1
0580e35886fd   prom/prometheus:latest                           "/bin/prometheus --c…"   30 hours ago   Up 30 hours   0.0.0.0:9090->9090/tcp, :::9090->9090/tcp                    otel-prometheus-1
f53db7f2aa4a   otel/opentelemetry-collector-contrib:0.78.0      "/otelcol-contrib --…"   30 hours ago   Up 30 hours   0.0.0.0:4317->4317/tcp, :::4317->4317/tcp, 55678-55679/tcp   otel-collector-1
db4a6fb3bc49   fluent/fluent-bit:latest                         "/fluent-bit/bin/flu…"   30 hours ago   Up 30 hours   2020/tcp, 0.0.0.0:3001->3001/tcp, :::3001->3001/tcp          otel-fluentbit-1
```

5. Інтерфейс системи моніторингу

![Grafana Dashboard](.img/GrafanaDashboard.png) 

## Стек моніторингу Grafana розгорнуто в кластері Kubernetes 

**Завдання:**   
Стек розгорнуто та налаштовано у dev-середовищі в Kubernetes для власного проєкту за допомогою Flux. Otel розгорнуто оператором. Fluentbit збирає та експортує логи проєкту та усіх нод кластеру. Проєкт інструментовано для експорту метрик.

Спроба виконати завдання на базі досягнень шановного [Andygol](https://github.com/Andygol) знаходяться в гілці `otel`цього репозиторію.