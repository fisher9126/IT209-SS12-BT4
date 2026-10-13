# Bài 4 - Session 12: Tự động triển khai Spring Boot (container) lên VPS qua SSH

## Luồng hoạt động

Workflow `.github/workflows/deploy-vps.yml` chạy khi push vào nhánh `main`, gồm 2 job nối tiếp:

1. `build-image`: build Docker image từ `Dockerfile` (multi-stage) rồi đẩy lên GHCR với tag `ghcr.io/<owner>/app:latest`.
2. `deploy-vps` (có `needs: build-image`): dùng `appleboy/ssh-action@v1.0.3` SSH vào VPS, đăng nhập GHCR, pull image mới, dừng và xóa container `app` cũ (nếu có), chạy lại container `app` ở cổng 8080, rồi gọi thử `/actuator/health`.

Image dùng `github.repository_owner` làm tên người dùng. GHCR yêu cầu tên viết thường, tên tài khoản `fisher9126` đã đúng.

## Chuẩn bị SSH key (làm một lần)

Tạo cặp khóa riêng cho deploy trên máy cá nhân:

```
ssh-keygen -t ed25519 -f ~/.ssh/deploy_key -N "" -C "github-actions-deploy"
```

Đưa khóa công khai lên VPS:

```
ssh-copy-id -i ~/.ssh/deploy_key.pub root@<VPS_IP>
```

Kiểm tra đăng nhập được bằng khóa mới:

```
ssh -i ~/.ssh/deploy_key root@<VPS_IP> "docker --version"
```

In nội dung khóa riêng để dán vào secret `SSH_KEY` (copy toàn bộ, gồm cả dòng BEGIN và END):

```
cat ~/.ssh/deploy_key
```

## Khai báo Secrets

Repo trên GitHub: Settings, Secrets and variables, Actions, New repository secret:

| Secret | Giá trị |
| --- | --- |
| `SSH_HOST` | Địa chỉ IP của VPS |
| `SSH_USER` | Tên người dùng SSH, ví dụ `root` |
| `SSH_KEY` | Nội dung khóa riêng `deploy_key` |

Không dán khóa riêng hay mật khẩu VPS vào chat hay commit lên repo.

## Mở cổng 8080 trên VPS

```
ufw allow 8080/tcp
```

Nếu nhà cung cấp VPS có firewall riêng trên trang quản trị, cũng cần mở cổng 8080 ở đó.

## Đẩy code

```
git init
git add .
git commit -m "Add deploy to VPS workflow"
git branch -M main
git remote add origin https://github.com/fisher9126/IT209-SS12-BT4.git
git push -u origin main
```

## Kiểm tra

- Tab Actions: cả `build-image` và `deploy-vps` tích xanh. Mở log `deploy-vps` để chụp ảnh log SSH Deploy, thấy dòng `{"status":"UP"}`.
- Trình duyệt: `http://<VPS_IP>:8080/actuator/health` trả về HTTP 200 với `{"status":"UP"}`.
- Trên VPS: `docker ps` thấy container `app` ở trạng thái Up.

## Sản phẩm nộp

- `.github/workflows/deploy-vps.yml`
- Ảnh chụp log SSH Deploy
