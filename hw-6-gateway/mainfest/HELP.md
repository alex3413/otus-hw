Создать namespace:
- kubectl create namespace otus-user-app

Для старта приложения необходимо развернуть БД:    
- postgres:
    - helm install postgresql-otus bitnami/postgresql -f postgres.values.yaml -n otus-user-app
Описание приложений:
    - user-app - бизнес приложение предметной области "Управление пользователем"
    - user-auth - сервис авторизации, реализован по спецификации oauth2 openid connect
    - gateway - сервси маршрутизации и резолвинга путей, так же фильтр по аутентификации и авторизации
Деплой приложения:
- Манифесты сервисов и деплойментов всех приложений, разворачиваются одним чартом 
- Создать секрет с конфигом подключения к БД 
  - kubectl apply -f user-app-secret.yaml -n otus-user-app
- Deploy user-app, user-auth, gateway
  - helm install otus-user-app ./user-chart -n otus-user-app

Сборка и отображение метрик:
- prometheus:
  - kubectl create secret generic user-app-user-chart-config --save-config  --from-file scrape-configs.yaml -n otus-user-app
  - helm install user-app-prometheus bitnami/kube-prometheus -f prometheus.values.yaml -n otus-user-app

- grafana:
  - kubectl create secret generic ds-secret-prometheus --from-file=ds-prometheus.yaml -n otus-user-app
  - kubectl create configmap dashboard-micrometr-spring --from-file=dashboards/micrometer-spring-throughput_rev2.json -n otus-user-app
  - kubectl create configmap dashboard-nginx --from-file=dashboards/nginx.json -n otus-user-app
  - helm install otus-user-grafana bitnami/grafana -f  grafana.values.yaml -n otus-user-app
