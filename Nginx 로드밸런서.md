# Nginx 로드밸런서

Nginx는 웹 서버, 리버스 프록시 및 로드밸런서 기능을 제공하는 고성능 HTTP 및 프록시 서버입니다. 로드밸런서로서 Nginx는 여러 서버에 트래픽을 분산시켜 시스템의 부하를 분산하고 가용성과 신뢰성을 향상시킵니다.

## 기본 로드밸런싱 설정

Nginx에서 로드밸런싱을 구현하는 기본적인 설정은 다음과 같습니다:

```nginx
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

## 로드밸런싱 알고리즘

Nginx는 다양한 로드밸런싱 알고리즘을 제공합니다:

### 1. 라운드 로빈 (기본값)
모든 서버에 순차적으로 요청을 분배합니다.

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
```

### 2. 가중치 기반 라운드 로빈
각 서버에 가중치를 부여하여 더 많은 트래픽을 처리할 수 있는 서버에 더 많은 요청을 할당합니다.

```nginx
upstream backend {
    server backend1.example.com weight=3;
    server backend2.example.com weight=1;
}
```

### 3. 최소 연결 (Least Connections)
활성 연결이 가장 적은 서버로 요청을 라우팅합니다.

```nginx
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

### 4. IP 해시
클라이언트 IP 주소를 기반으로 요청을 분배하여 동일한 클라이언트가 항상 동일한 서버로 라우팅되도록 합니다.

```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

### 5. 일반 해시
지정된 키(URL, 쿠키 등)에 따라 요청을 분배합니다.

```nginx
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com;
    server backend2.example.com;
}
```

## 서버 상태 확인 및 장애 처리

Nginx는 업스트림 서버의 상태를 모니터링하고 장애 발생 시 적절히 대응할 수 있습니다:

### 패시브 헬스 체크

```nginx
upstream backend {
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}
```

### 액티브 헬스 체크 (Nginx Plus 기능)

```nginx
http {
    server {
        # ... 서버 설정 ...

        location / {
            proxy_pass http://backend;
            health_check interval=5s fails=3 passes=2;
        }
    }
}
```

## 고급 로드밸런싱 설정

### 세션 유지 (스티키 세션)

```nginx
upstream backend {
    sticky cookie srv_id expires=1h domain=.example.com path=/;
    server backend1.example.com;
    server backend2.example.com;
}
```

### 백업 서버 설정

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backup1.example.com backup;
    server backup2.example.com backup;
}
```

### 슬로우 스타트

```nginx
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
}
```

### 연결 제한

```nginx
upstream backend {
    server backend1.example.com max_conns=100;
    server backend2.example.com max_conns=100;
}
```

## SSL 터미네이션

로드밸런서에서 SSL을 처리하고 백엔드 서버와는 HTTP로 통신하는 구성:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 실제 구현 사례

### 마이크로서비스 아키텍처 예제

```nginx
http {
    # API 서비스 업스트림
    upstream api_service {
        least_conn;
        server api1.example.com:8080;
        server api2.example.com:8080;
        server api3.example.com:8080;
    }

    # 웹 서비스 업스트림
    upstream web_service {
        ip_hash;
        server web1.example.com:8081;
        server web2.example.com:8081;
    }

    # 인증 서비스 업스트림
    upstream auth_service {
        server auth1.example.com:8082;
        server auth2.example.com:8082;
    }

    server {
        listen 80;
        server_name api.example.com;
        
        location / {
            proxy_pass http://api_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }

    server {
        listen 80;
        server_name www.example.com;
        
        location / {
            proxy_pass http://web_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }

    server {
        listen 80;
        server_name auth.example.com;
        
        location / {
            proxy_pass http://auth_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

Nginx 로드밸런서는 고성능, 확장성 및 구성 유연성으로 인해 많은 기업에서 널리 사용되고 있습니다. 적절한 로드밸런싱 전략을 선택하고 구현함으로써 애플리케이션의 가용성, 신뢰성 및 성능을 크게 향상시킬 수 있습니다.
