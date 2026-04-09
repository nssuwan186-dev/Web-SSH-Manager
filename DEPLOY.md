# 🚀 Web-SSH-Manager — คู่มือ Deploy

## 📍 โค้ดอยู่ไหน
```
~/tools/Web-SSH-Manager/
```

## 📦 GitHub Repo
```
git@github.com:nssuwan186-dev/Web-SSH-Manager.git
```

---

## วิธี 1: Deploy บน VPS (แนะนำ)

### ขั้นตอนที่ 1: ตั้งค่า GitHub Secrets
```bash
cd ~/tools/Web-SSH-Manager

gh secret set DEPLOY_HOST      # ใส่ IP VPS
gh secret set DEPLOY_USER      # ใส่ SSH username (เช่น root)
gh secret set DEPLOY_SSH_KEY   < ~/.ssh/id_ed25519
gh secret set MONGODB_URI      # MongoDB Atlas URI
gh secret set JWT_SECRET       # openssl rand -hex 32
gh secret set ENCRYPTION_KEY   # openssl rand -hex 16 (32 chars)
```

### ขั้นตอนที่ 2: Push → Auto Deploy
```bash
git add . && git commit -m "deploy: add CI/CD pipeline"
git push origin main
```

CI จะ build + test → deploy ไป VPS อัตโนมัติ

---

## วิธี 2: Deploy ง่ายผ่าน Docker Compose

```bash
cd ~/tools/Web-SSH-Manager

# สร้าง .env.local
cp .env.example .env.local
# แก้ค่าใน .env.local ให้ถูกต้อง

# รัน
docker compose up -d
```

เข้าใช้งาน: `http://localhost:3000`

---

## วิธี 3: Deploy บน Free Platforms

### Railway
```bash
# เชื่อม GitHub repo → Railway จะ detect อัตโนมัติ
# ตั้งค่า env vars ใน Railway dashboard
# Deploy อัตโนมัติทุก push
```

### Render
```bash
# สร้าง Web Service จาก GitHub repo
# ตั้งค่า env vars
# Deploy อัตโนมัติ
```

### Google Cloud Run
```bash
gcloud run deploy web-ssh-manager \
  --source . \
  --set-env-vars MONGODB_URI=xxx,JWT_SECRET=xxx,ENCRYPTION_KEY=xxx
```

---

## 🔑 สร้าง Secret Keys

```bash
# JWT Secret (32+ chars)
openssl rand -hex 32

# Encryption Key (exactly 32 chars)
openssl rand -hex 16
```

## 🗄️ MongoDB Atlas (ฟรี)
1. สมัคร: https://www.mongodb.com/cloud/atlas
2. สร้าง cluster → เลือก free tier
3. สร้าง database user
4. Whitelist IP (0.0.0.0/0 สำหรับทุกที่)
5. คัดลอก connection string → ใส่ `MONGODB_URI`

---

## 📂 โครงสร้าง CI/CD

```
.github/workflows/
├── ci.yml         → Build + Test ทุก push
└── deploy.yml     → Deploy ไป VPS เมื่อ CI ผ่าน
```

### CI (ci.yml)
- Setup Node.js 20
- Install deps → `npm ci`
- Lint → `npm run lint`
- Build → `npm run build`
- Docker build check

### Deploy (deploy.yml)
- Build Docker image
- Push ไป GitHub Container Registry (ฟรี)
- SSH เข้า VPS → pull + run container ใหม่
- Auto cleanup old images

---

## 🔍 ตรวจสอบ Deploy
```bash
# ดูสถานะ CI/CD
gh run list --repo nssuwan186-dev/Web-SSH-Manager

# ดู logs
gh run view <run-id> --log
```
