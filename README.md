# task22
Конфигурация docker-compose: <br>
```
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:v3.4.2 
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:v1.9.1 
    ports:
      - "9100:9100"
    restart: unless-stopped

  nginx:
    image: nginx:1.18.0
    volumes:
      - ./nginx_logs:/var/log/nginx
    ports:
      - "80:80"

  apache:
    image: httpd:2.4.64
    volumes:
      - ./html:/usr/local/apache2/htdocs/
      - ./apache_logs:/usr/local/apache2/logs
    ports:
      - "81:80"

  promtail:
    image: grafana/promtail:latest
    ports:
      - "9080:9080"
    volumes:
      - ./promtail.yaml:/etc/promtail/config.yaml
      - ./nginx_logs:/var/log/nginx
      - ./apache_logs:/usr/local/apache2/logs
    command: ["-config.file=/etc/promtail/config.yaml"]

  grafana:
    image: grafana/grafana:12.0.2
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_SECURITY_ADMIN_USER=admin
    volumes:
      - grafanadata:/var/lib/grafana

  loki:
    image: grafana/loki:3.5.2
    ports:
      - "3100:3100"

volumes:
  grafanadata: {}
```

Конфигурация prometheus: <br>
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - 'node-exporter:9100'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'Local machine'
```

Конфигурация promtail: <br>
```
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml


clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: nginx-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log
    
  - job_name: apache-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: apache
          __path__: /var/log/apache2/*.log
          __path_eclude__: /usr/local/apache2/logs/*.pid
```


Проверка: <br>
<img width="1067" height="303" alt="image" src="https://github.com/user-attachments/assets/3861b5d6-126a-4163-9281-d005d31480c0" />
<img width="1173" height="255" alt="image" src="https://github.com/user-attachments/assets/a83e78a2-601e-41c5-8ba2-58e244db75d6" />
<img width="680" height="449" alt="image" src="https://github.com/user-attachments/assets/87fa2d8b-30fd-42f9-aca9-cde6f34563f8" />
<img width="403" height="180" alt="image" src="https://github.com/user-attachments/assets/f5716882-a934-4897-a9c4-b2f7cdebce7d" /> <br>
<img width="551" height="868" alt="image" src="https://github.com/user-attachments/assets/16c2cc5f-2366-4d07-8998-f68579d95244" />

<br>
Создание dashboards: <br>
1) CPU/RAM <br>
<img width="1508" height="826" alt="image" src="https://github.com/user-attachments/assets/586a2cc4-7ebc-4ac9-90bb-fc327b7e0f6a" />





2) Logs nginx/apache: <br>

<img width="1522" height="844" alt="image" src="https://github.com/user-attachments/assets/9216952e-e58c-4d32-aa10-d91976f066bf" />


Настроил отправку уведомлений в slack: <br>
<img width="671" height="572" alt="image" src="https://github.com/user-attachments/assets/8c61c44c-ce22-42f0-b92d-4b9198a5adfa" />
<img width="440" height="317" alt="image" src="https://github.com/user-attachments/assets/a7de06e6-deca-4a26-8272-9355f9b5bf25" />







