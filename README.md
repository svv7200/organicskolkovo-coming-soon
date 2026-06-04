# Organic Skolkovo — Coming Soon Page

> Сайт-заглушка для команды [Organic Skolkovo](https://organicskolkovo.su) — ресторанной группы с 10+ летним опытом и 8 ресторанами в Москве.

![preview](preview.png)

## 🌐 Live

**https://organicskolkovo.su**

---

## Стек

- Чистый HTML/CSS/JS — без фреймворков и зависимостей
- Canvas API — анимация плавающих частиц
- Google Fonts (Nunito / Nunito Sans)
- Адаптивная вёрстка (mobile-first)

## Особенности

- Логотип воссоздан в CSS с точными цветами бренда
- Плавные анимации появления (staggered letter-by-letter)
- Фоновое фото с медленным zoom-эффектом и виньеткой
- Анимированные частицы в цветах палитры бренда
- Фиксированный контактный футер

---

## Инфраструктура — как это работает на сервере

### Проблема

На том же VPS уже работал **VPN (VLESS+TLS)** на порту `443` через Xray/3x-ui.  
Стандартный способ поднять HTTPS-сайт — повесить nginx на 443 — невозможен: порт занят.

### Решение: Xray Fallback

Xray умеет различать входящий трафик по первому байту соединения:

```
Клиент → :443 (Xray, TLS)
              ├── VLESS-клиент (VPN)  → Xray обрабатывает сам
              └── Браузер (HTTP/1.1)  → fallback → nginx :8080 → сайт
```

Xray терминирует TLS с сертификатом Let's Encrypt, а «не-VPN» трафик  
проксирует на `127.0.0.1:8080`, где nginx отдаёт статику сайта по plain HTTP.

### Конфигурация Xray (inbound на 443)

```json
{
  "port": 443,
  "protocol": "vless",
  "settings": {
    "clients": [ "...65 пользователей..." ],
    "fallbacks": [
      { "dest": 8080, "xver": 0 }
    ]
  },
  "streamSettings": {
    "network": "tcp",
    "security": "tls",
    "tlsSettings": {
      "alpn": ["http/1.1"],
      "certificates": [{
        "certificateFile": "/root/cert/organicskolkovo.su/fullchain.pem",
        "keyFile": "/root/cert/organicskolkovo.su/privkey.pem"
      }]
    }
  }
}
```

### Конфигурация Nginx

```nginx
# HTTP → HTTPS редирект
server {
    listen 80;
    server_name organicskolkovo.su www.organicskolkovo.su;
    return 301 https://$host$request_uri;
}

# Принимает трафик от Xray fallback (plain HTTP, TLS уже снят)
server {
    listen 127.0.0.1:8080;
    server_name organicskolkovo.su www.organicskolkovo.su;

    root /var/www/organicskolkovo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(jpg|jpeg|png|gif|svg|webp)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    gzip on;
    gzip_types text/html text/css application/javascript image/svg+xml;
}
```

### SSL-сертификат

Сертификат Let's Encrypt выпущен через certbot, автопродление настроено:

```bash
certbot --nginx -d organicskolkovo.su -d www.organicskolkovo.su \
  --non-interactive --agree-tos --email admin@example.com
```

> Поскольку TLS терминирует Xray, а не nginx — сертификат прописывается  
> в конфиге Xray (`tlsSettings.certificates`), а не в nginx.

### Итоговая карта портов на сервере

| Порт | Сервис | Описание |
|------|--------|----------|
| `443` | Xray (VLESS+TLS) | VPN + fallback для сайта |
| `80` | nginx | Редирект на HTTPS |
| `127.0.0.1:8080` | nginx | Сайт (принимает от Xray fallback) |
| `50970` | 3x-ui | Панель управления VPN |
| `2096` | 3x-ui | Сервер подписок VPN |

---

## Структура файлов

```
/var/www/organicskolkovo/
├── index.html   — единственный файл сайта
└── bg.jpg       — фоновое фото (оптимизировано, ~130kb)
```

---

## Цвета бренда

| Цвет | HEX | Использование |
|------|-----|---------------|
| Teal (основной) | `#1a6068` | O, N — буквы логотипа, SKOLKOVO |
| Terracotta | `#c8624a` | R — буква логотипа |
| Sand | `#c9a46a` | G, C — буквы логотипа, акценты |
| Silver | `#a8b8bc` | A, I — буквы логотипа |

---

## Контакты

- it@organicskolkovo.com  
- marketing@organicskolkovo.com
