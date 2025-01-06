Создать namespace:
- kubectl create namespace otus-user-app

Для старта приложения необходимо развернуть системы окружения:    
- База данных postgres:
  - helm install postgresql-otus bitnami/postgresql -f postgres.values.yaml -n otus-user-app
- Брокер сообщений rabbitMQ:
  - helm install rabbit-otus bitnamicharts/rabbitmq -n otus-user-app
Описание базовых приложений:
    - user-app - бизнес приложение предметной области "Управление пользователем"
    - user-auth - сервис авторизации, реализован по спецификации oauth2 openid connect
    - gateway - сервси маршрутизации и резолвинга путей, так же фильтр по аутентификации и авторизации
    - notification-app - микросервис работы с уведомлениями пользователя
    - audit-app - микросервис работы с аудитом действий пользователя
Описание приложений ядра:
    - project-app - микросервис работы с проектами пользователя
    - manifest-app - микросервис работы с манифестами пользователя
    - artefact-app - микросервис работы с артефакатми сгенерированными на основе проекта и манифеста

Деплой приложения:
- Манифесты сервисов и деплойментов основных приложений, разворачиваются одним чартом "user-chart"
- Манифесты приложений ядра системы: работа с проектами, сгенерированными артефактами и манифестом архитектуры,
  разворачиваются чартом "otus-project-core"
- Создать секрет с конфигом подключения к БД 
  - kubectl apply -f user-app-secret.yaml -n otus-user-app
- Создать конфигМап с конфигурациями приложений:
  - kubectl apply -f otus-app-configmap.yaml -n otus-user-app
- Развернуть приложения: user-app, user-auth, gateway
  - helm install otus-user-app ./user-chart -n otus-user-app
- Развернуть приложения ядра: project-app, manifest-app, artefact-app
  - helm install otus-core-app ./otus-project-core -n otus-user-app

Сборка и отображение метрик:
- prometheus:
  - kubectl create secret generic user-app-user-chart-config --save-config  --from-file scrape-configs.yaml -n otus-user-app
  - helm install user-app-prometheus bitnami/kube-prometheus -f prometheus.values.yaml -n otus-user-app

- grafana:
  - kubectl create secret generic ds-secret-prometheus --from-file=ds-prometheus.yaml -n otus-user-app
  - kubectl create configmap dashboard-micrometr-spring --from-file=dashboards/micrometer-spring-throughput_rev2.json -n otus-user-app
  - kubectl create configmap dashboard-nginx --from-file=dashboards/nginx.json -n otus-user-app
  - helm install otus-user-grafana bitnami/grafana -f  grafana.values.yaml -n otus-user-app
