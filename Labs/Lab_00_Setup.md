# Lab 00 — เตรียมเครื่องก่อนเริ่มอบรม

**หลักสูตร** Application Performance Monitoring & Error Tracking ด้วย Sentry
**ผู้เข้าอบรม** ธนาคารการค้าต่างประเทศลาว มหาชน (BCEL) · 29–31 กรกฎาคม 2569
**เวลาที่ใช้** 20–30 นาที (ทำเช้าวันแรก 09:30–10:00 น.)

> เอกสารนี้ทดสอบจริงบนเครื่องวิทยากรแล้วทุกขั้นตอน ค่าที่ระบุคือค่าที่ได้จริง ไม่ใช่ค่าประมาณ

---

## 0.1 ตรวจเครื่องมือบนเครื่องผู้เรียน

รันทีละบรรทัด ผลที่ได้ต้องไม่ต่ำกว่าเวอร์ชันที่ระบุ

```bash
docker --version
```

```bash
docker compose version
```

```bash
java -version
```

```bash
mvn -v
```

```bash
node -v
```

| เครื่องมือ | เวอร์ชันขั้นต่ำ | เวอร์ชันที่ทดสอบแล้ว |
| --- | --- | --- |
| Docker | 24.x | 29.1.2 |
| Docker Compose | v2.20 | v2.40.3 |
| JDK | **21** (บังคับ) | 21.0.8 |
| Maven | 3.9 | 3.9.11 |
| Node.js | 20 LTS | 22.14.0 |

> ⚠️ **JDK ต้องเป็น 21 เท่านั้น** โปรเจกต์ตั้ง `<java.version>21</java.version>` ไว้ ถ้าใช้ 17 จะ compile ไม่ผ่าน

---

## 0.2 ⛔ ข้อควรระวังเรื่อง "ที่อยู่โฟลเดอร์" (สำคัญมาก — เจอจริงตอนทดสอบ)

**วางโปรเจกต์ไว้ในพาธที่ไม่มีช่องว่างและไม่มีอักขระพิเศษ** เช่น

```
C:\labs\bcel-crm-lite          ✅ ใช้ได้
```

```
C:\คอร์สอบรม 2026\Sentry APM & Error Tracking\bcel-crm-lite    ❌ พัง
```

**เกิดอะไรขึ้น** ตัว shim ของ npm บน Windows เป็นไฟล์ `.cmd` เครื่องหมาย `&` ในชื่อโฟลเดอร์
ถูก `cmd.exe` ตีความว่าเป็น "ตัวคั่นคำสั่ง" ผลคือรัน `npm start` แล้วได้

```
'Error' is not recognized as an internal or external command
Error: Cannot find module 'C:\คอร์สอบรม 2026\@angular\cli\bin\ng.js'
```

**วิธีแก้** ย้าย/คัดลอกโปรเจกต์ไปพาธที่สะอาด หรือถ้าย้ายไม่ได้ ให้เรียก Angular CLI ตรง ๆ แทน `npm start`

```bash
node node_modules/@angular/cli/bin/ng.js serve
```

---

## 0.3 เตรียมฐานข้อมูล MariaDB

```bash
cd bcel-crm-lite
```

```bash
docker compose up -d mariadb
```

ครั้งแรกจะดึง image และ seed ข้อมูล **ใช้เวลา 1–3 นาที** ระหว่างรอให้ทำข้อ 0.5 ไปพร้อมกัน

### ตรวจว่า seed เสร็จแล้ว

```bash
docker compose exec mariadb mariadb -u crm_app -plabpass123 bcel_crm -e "SELECT COUNT(*) AS customers FROM customer; SELECT COUNT(*) AS tx_logs FROM transaction_log;"
```

**ผลที่ต้องได้**

```
customers
5000
tx_logs
800000
```

ถ้าได้ `Table doesn't exist` แปลว่า seed ยังไม่จบ รออีกสักครู่แล้วสั่งใหม่

### 🔧 ถ้าพอร์ต 3306 ชนกัน

อาการ

```
Error response from daemon: ports are not available: exposing port TCP 0.0.0.0:3306
listen tcp 0.0.0.0:3306: bind: Only one usage of each socket address ...
```

หาว่าใครใช้พอร์ตอยู่ (บ่อยที่สุดคือ XAMPP/MySQL ที่ติดตั้งไว้เดิม)

```powershell
Get-NetTCPConnection -LocalPort 3306 -State Listen | ForEach-Object { Get-Process -Id $_.OwningProcess | Select-Object Id,ProcessName,Path }
```

**ทางเลือกที่ 1 (แนะนำ)** ปิดบริการที่ครองพอร์ตอยู่ชั่วคราว แล้วสั่ง `docker compose up -d mariadb` ใหม่

**ทางเลือกที่ 2** ย้ายพอร์ตของ Lab ไปที่ 13306 โดยสร้างไฟล์ `docker-compose.override.yml`
ไว้ข้าง ๆ `docker-compose.yml` (ไฟล์นี้ compose จะอ่านให้อัตโนมัติ ไม่ต้องแก้ไฟล์เดิม)

```yaml
services:
  mariadb:
    ports: !override
      - "13306:3306"
```

> ⚠️ ต้องมี `!override` ด้วย ถ้าไม่ใส่ compose จะ **รวม** รายการพอร์ต (ได้ทั้ง 3306 และ 13306) แล้วพังเหมือนเดิม

จากนั้นตอนรัน backend ให้ทับค่า URL ด้วย environment variable ไม่ต้องแก้ `application.properties`

```bash
SPRING_DATASOURCE_URL='jdbc:mariadb://localhost:13306/bcel_crm' mvn spring-boot:run
```

---

## 0.4 เตรียม Backend และ Frontend

ดึง dependency ล่วงหน้า (ทำระหว่างรอ seed)

```bash
cd backend && mvn -B -q dependency:go-offline
```

```bash
cd frontend && npm i
```

---

## 0.5 สร้างบัญชีและโปรเจกต์บน Sentry

ห้อง Lab นี้ใช้ **Sentry SaaS** (ที่ BCEL จริงจะเป็น Self-hosted ซึ่งขั้นตอนฝั่งโค้ดเหมือนกันทุกอย่าง
ต่างแค่ค่า DSN และการเติม `SENTRY_URL` ให้ `sentry-cli`)

1. เข้า `https://itgenius-qc.sentry.io/`
2. เมนูซ้าย → **Settings** → **Projects** → **Create Project**
3. สร้าง **2 โปรเจกต์**

| # | Platform | Project slug | Alert frequency |
| --- | --- | --- | --- |
| 1 | **SPRING BOOT** | `bcel-crm-backend` | เลือก *I'll create my own alerts later* |
| 2 | **ANGULAR** | `bcel-crm-frontend` | เลือก *I'll create my own alerts later* |

> เลือก "I'll create my own alerts later" เพราะเราจะสร้าง Alert เองในวันที่ 3 ถ้าเลือกค่าปริยาย
> จะมี alert rule ค้างมาให้และทำให้ Lab 3.1 สับสน

### หา DSN ของแต่ละโปรเจกต์

**Settings → Projects → <ชื่อโปรเจกต์> → Client Keys (DSN)**

รูปแบบที่ได้

```
https://<public-key>@o<org-id>.ingest.us.sentry.io/<project-id>
```

> 📌 สังเกตส่วน `.us.` — องค์กรนี้อยู่ **region US** ไม่ใช่ `de` เอกสารบางที่จะเขียน `ingest.de.sentry.io`
> ให้ยึดค่าที่หน้า Client Keys ของท่านเองเสมอ

จดทั้งสองค่าไว้ จะใช้ตลอด 3 วัน

```
BACKEND  DSN = ______________________________________________
FRONTEND DSN = ______________________________________________
```

---

## 0.6 ✅ เกณฑ์ตรวจผ่าน Lab 00

| # | สิ่งที่ต้องได้ | ตรวจอย่างไร |
| --- | --- | --- |
| 1 | MariaDB รันอยู่ | `docker compose ps` → STATUS = `Up (healthy)` |
| 2 | ข้อมูลครบ | `customers = 5000`, `tx_logs = 800000` |
| 3 | JDK 21 | `java -version` ขึ้น `21.x` |
| 4 | มี 2 โปรเจกต์ใน Sentry | หน้า Settings → Projects เห็นทั้ง backend และ frontend |
| 5 | มี DSN ทั้ง 2 ค่า | จดไว้แล้วในตารางข้างบน |

---

## 0.7 คำสั่งที่ใช้บ่อยตลอดคอร์ส

| ต้องการ | คำสั่ง |
| --- | --- |
| รัน DB | `docker compose up -d mariadb` |
| ดู log DB | `docker compose logs -f mariadb` |
| ล้าง + seed ใหม่ | `powershell -ExecutionPolicy Bypass -File scripts\reset-db.ps1` |
| รัน Backend | `mvn spring-boot:run` (ใน `backend/`) |
| รัน Frontend | `npm start` (ใน `frontend/`) |
| หยุดทุกอย่าง | `docker compose down` |

---

**สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง** · อ.สามิตร โกยม · 02-570-8449 · www.itgenius.co.th
