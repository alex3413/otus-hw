Для старта приложения необходимо развернуть БД:    
postgres:
    helm install postgresql-otus bitnami/postgresql -f postgres.values.yaml
Сборка и отображение метрик:
    prometheus:
        - kubectl create secret generic user-app-user-chart-config --save-config  --from-file scrape-configs.yaml
        - helm install user-app-prometheus bitnami/kube-prometheus -f prometheus.values.yaml -n {{ .Release.Namespace }}
    grafana:
        - kubectl create secret generic ds-secret-prometheus --from-file=ds-prometheus.yaml -n {{ .Release.Namespace }}
        - kubectl create configmap dashboard-micrometr-spring --from-file=dashboards/micrometer-spring-throughput_rev2.json -n {{ .Release.Namespace }}
        - kubectl create configmap dashboard-nginx --from-file=dashboards/nginx.json -n {{ .Release.Namespace }}
        - helm install otus-user-grafana bitnami/grafana -f  grafana.values.yaml -n {{ .Release.Namespace }}