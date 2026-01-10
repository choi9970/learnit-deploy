# LearnIT Deploy

LearnIT 서비스 운영/배포 전용 레포지토리입니다.

이 레포는 **Docker 이미지를 빌드하지 않습니다.**  
EC2 서버에서 `docker compose`로 컨테이너를 실행/관리하는 역할만 합니다.

---

## 📦 구성 요소

- **Backend**: Spring Boot (ECR 이미지)
- **Chat Agent**: FastAPI (ECR 이미지)
- **Web**: nginx
- **SSL**: certbot (Let's Encrypt)
- **Infra**: EC2 + Docker Compose

---

## 📁 디렉토리 구조

```text
learnit-deploy
├─ docker-compose.yml
├─ .env.example
├─ nginx/
│  └─ conf.d/
│     ├─ 00-http.conf
│     └─ 10-https.conf
├─ certbot/
│  ├─ conf/
│  └─ www/
└─ README.md
```

🚀 배포 방법 (EC2)
1) 레포 클론
```bash
cd /srv/learnit
git clone https://github.com/choi9970/learnit-deploy.git
cd learnit-deploy
```

2) 환경 변수 설정
```
cp .env.example .env
nano .env
```
.env는 절대 GitHub에 커밋하지 않습니다.

3) ECR 로그인 (IAM Role 사용)
```
aws ecr get-login-password --region ap-northeast-2 \
| docker login --username AWS --password-stdin \
593883982145.dkr.ecr.ap-northeast-2.amazonaws.com
```
4) 컨테이너 실행
```
docker compose pull
docker compose up -d
docker compose ps
```
🔐 HTTPS 인증서 발급 (최초 1회)
```
docker compose stop nginx

docker run --rm \
  -v "$(pwd)/certbot/conf:/etc/letsencrypt" \
  -v "$(pwd)/certbot/www:/var/www/certbot" \
  certbot/certbot certonly \
  --webroot -w /var/www/certbot \
  -d learnit24.com -d www.learnit24.com \
  --email YOUR_EMAIL@example.com \
  --agree-tos --no-eff-email

docker compose up -d
```
🔄 업데이트 방법
```
git pull
docker compose pull
docker compose up -d
```
⚠️ 주의 사항

이 레포는 운영 전용입니다.

GitHub Actions / Dockerfile / 빌드 설정은 포함하지 않습니다.

이미지 빌드는 각 서비스 레포에서 수행합니다.

🔗 관련 레포

Backend: Acorn_Project_LearnIT

Chat Agent: learnit-chat-agent