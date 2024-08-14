FROM nginx:latest

# Устанавливаем необходимые пакеты
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# Добавляем конфигурационный файл для метрик
COPY nginx.conf /etc/nginx/nginx.conf

# Скачиваем nginx-prometheus-exporter
RUN curl -L https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v0.9.0/nginx-prometheus-exporter-0.9.0-linux-amd64.tar.gz | tar xz -C /usr/local/bin/

# Открываем порт 9113 для метрик
EXPOSE 80 9113

CMD ["nginx", "-g", "daemon off;"]