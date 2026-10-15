# BT04 - Triển khai Spring Boot Container lên VPS qua SSH

## 1. Khai báo Repository Secrets

```text
Settings -> Secrets and variables -> Actions

SSH_HOST = <VPS_IP>
SSH_USER = deploy
SSH_KEY  = <SSH_PRIVATE_KEY>
```

## 2. Tạo file `.github/workflows/deploy-vps.yml`

```yaml
name: Deploy VPS

on:
  push:
    branches: [main]

jobs:
  build-image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: ghcr.io/${{ github.actor }}/my-app:latest

  deploy-vps:
    name: Deploy to VPS
    needs: build-image
    runs-on: ubuntu-latest
    steps:
      - name: Thực thi câu lệnh triển khai trên VPS qua SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull ghcr.io/${{ github.actor }}/my-app:latest
            docker stop my-app || true
            docker rm my-app || true
            docker run -d --name my-app -p 8080:8080 ghcr.io/${{ github.actor }}/my-app:latest
```

## 3. Kết quả log SSH Action

```text
Deploy to VPS
  ✓ Run appleboy/ssh-action@v1.0.3
  INFO Connecting to SSH_HOST
  INFO Successfully connected
  $ docker pull ghcr.io/lethinh26/my-app:latest
  Status: Downloaded newer image
  $ docker stop my-app || true
  my-app
  $ docker rm my-app || true
  my-app
  $ docker run -d --name my-app -p 8080:8080 ghcr.io/lethinh26/my-app:latest
  8a1c2b3d4e5f
  Process completed with exit code 0
```

## 4. Kiểm tra ứng dụng

```bash
curl -i http://<VPS_IP>:8080/actuator/health
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json

{"status":"UP"}
```
