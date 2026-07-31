# คู่มือติดตั้ง Sentry Self-hosted ด้วยตนเองตั้งแต่ศูนย์

**หลักสูตร** Application Performance Monitoring & Error Tracking ด้วย Sentry
**ผู้เข้าอบรม** ธนาคารการค้าต่างประเทศลาว มหาชน (BCEL) · 29–31 กรกฎาคม 2569
**ช่วงเวลาที่ใช้** 🗓️ **บ่ายวันศุกร์ที่ 31 กรกฎาคม 2569 · 13:00–16:30 น.** (Module ปิดท้ายหลักสูตร)

> 🎯 **แนวทางของเอกสารนี้**
> วิทยากรเตรียมให้แค่ **Droplet เปล่า ๆ ที่ยังไม่มีอะไรเลย** นอกจาก Ubuntu
> **ผู้เรียนติดตั้งทุกอย่างเองตั้งแต่ต้น** ตั้งแต่ swap · Docker · Sentry
> แล้วจบด้วยการ **ย้ายระบบงาน Spring Boot และ Angular จาก SaaS มาใช้ Sentry ของตัวเอง**
> รวม **32 ขั้นตอน**

> 📅 **ตำแหน่งในหลักสูตร**
>
> | วัน | ใช้ Sentry ตัวไหน |
> | --- | --- |
> | วันที่ 1–2 และเช้าวันที่ 3 | **SaaS** (`itgenius-qc.sentry.io`) — เรียนการใช้งานให้คล่องก่อน |
> | **บ่ายวันที่ 3** | **Self-hosted ที่ผู้เรียนติดตั้งเอง** ← เอกสารนี้ |
>
> 🔑 **ข้อดีของการวางไว้บ่ายวันสุดท้าย**
> ผู้เรียนคุ้นเคยกับ Sentry มาแล้ว 2 วันครึ่ง จึงเข้าใจว่ากำลังติดตั้ง "อะไร" อยู่
> และแอปทั้งสองฝั่ง**ฝัง SDK ไว้เรียบร้อยแล้ว** เหลือแค่เปลี่ยน DSN ก็ใช้ได้ทันที
> ทำให้ระยะที่ 6–8 สั้นลงมาก และจบภายในบ่ายเดียวได้จริง

> 📌 **ข้อมูลทางเทคนิคทั้งหมดตรวจจาก source code จริง** ของ `github.com/getsentry/self-hosted`
> ไฟล์ที่ตรวจ: `install/_min-requirements.sh` · `install/parse-cli.sh` ·
> `install/check-minimum-requirements.sh` · `install/update-docker-images.sh` ·
> `install/set-up-and-migrate-database.sh` · `.env` · `install.sh`
> **เวอร์ชันล่าสุด ณ วันที่จัดทำ: `26.7.2`**

---

## 📑 สารบัญ

| ส่วน | หัวข้อ | ใครทำ | เวลา |
| --- | --- | --- | --- |
| [0](#0-แผนใหม่-และผลกระทบต่อตารางเวลา) | แผนใหม่ และผลกระทบต่อตารางเวลา | วิทยากรอ่านก่อน | 15 นาที |
| [1](#1-ข้อกำหนดจริงของ-sentry-self-hosted) | ข้อกำหนดจริงของ Sentry Self-hosted | วิทยากรอ่านก่อน | 10 นาที |
| [2](#2-งานเดียวที่วิทยากรต้องทำ-สร้าง-droplet-เปล่า) | งานเดียวที่วิทยากรต้องทำ | วิทยากร | 20 นาที |
| **[3](#3-ระยะที่-1-เตรียมระบบปฏิบัติการ-ขั้นที่-16)** | **ระยะที่ 1 เตรียมระบบปฏิบัติการ** | **ผู้เรียน** | 30 นาที |
| **[4](#4-ระยะที่-2-ติดตั้ง-docker-ขั้นที่-710)** | **ระยะที่ 2 ติดตั้ง Docker** | **ผู้เรียน** | 20 นาที |
| **[5](#5-ระยะที่-3-เตรียม-sentry-ขั้นที่-1114)** | **ระยะที่ 3 เตรียม Sentry** | **ผู้เรียน** | 15 + รอ 10 นาที |
| **[6](#6-ระยะที่-4-ติดตั้ง-sentry-ขั้นที่-1518)** | **ระยะที่ 4 ติดตั้ง Sentry** | **ผู้เรียน** | 40 นาที |
| **[7](#7-ระยะที่-5-ตั้งค่าหลังติดตั้ง-ขั้นที่-1922)** | **ระยะที่ 5 ตั้งค่าหลังติดตั้ง** | **ผู้เรียน** | 25 นาที |
| **[8](#8-ระยะที่-6-เชื่อม-spring-boot-ขั้นที่-2326)** | **ระยะที่ 6 เชื่อม Spring Boot** | **ผู้เรียน** | 30 นาที |
| **[9](#9-ระยะที่-7-เชื่อม-angular-ขั้นที่-2730)** | **ระยะที่ 7 เชื่อม Angular** | **ผู้เรียน** | 30 นาที |
| **[10](#10-ระยะที่-8-พิสูจน์-end-to-end-ขั้นที่-3132)** | **ระยะที่ 8 พิสูจน์ End-to-End** | **ผู้เรียน** | 20 นาที |
| [11](#11-เกณฑ์ตรวจผ่านทั้งหมด) | เกณฑ์ตรวจผ่านทั้งหมด | ทั้งคู่ | 15 นาที |
| [12](#12-ปัญหาที่พบบ่อยและวิธีแก้) | ปัญหาที่พบบ่อยและวิธีแก้ | อ้างอิง | — |
| [13](#13-การดูแลระยะยาว-สำหรับ-bcel-ของจริง) | การดูแลระยะยาว (สำหรับ BCEL ของจริง) | อ้างอิง | — |
| [14](#14-ลบทิ้งและสรุปค่าใช้จ่าย) | ลบทิ้งและสรุปค่าใช้จ่าย | วิทยากร | 5 นาที |

---

## 0. แผนใหม่ และผลกระทบต่อตารางเวลา

### 0.1 เปรียบเทียบสามแผน

| | แผน A (SaaS) | แผน B (snapshot สำเร็จรูป) | **แผน C (ผู้เรียนทำเองทั้งหมด)** |
| --- | --- | --- | --- |
| วิทยากรเตรียม | สมัคร org | droplet + Docker + pre-pull image | **droplet เปล่าอย่างเดียว** |
| ผู้เรียนได้เรียนรู้ | ใช้งาน Sentry | รัน `install.sh` | **ทุกอย่างตั้งแต่ศูนย์** |
| ตรงกับงานจริงที่ BCEL | ⚠️ ไม่ตรง | ⚠️ ตรงบางส่วน | ✅ **ตรงที่สุด** |
| เวลาที่ต้องใช้ | 15 นาที | 75 นาที | **~3 ชั่วโมง** |
| ความเสี่ยง | ต่ำ | ปานกลาง | ⚠️ **สูง** |
| ค่าใช้จ่าย | 0 | ~$52 | ~$52 |

> ✅ **แผน C คือแผนที่เลือกใช้** — ให้คุณค่าการเรียนรู้สูงสุดและตรงกับสิ่งที่ทีม BCEL
> ต้องทำจริงที่ธนาคาร แต่**ต้องยอมรับว่ากินเวลาเกือบครึ่งวันแรก**

### 0.2 ⏱️ ตารางเวลาวันที่ 1 ที่ปรับใหม่ (สำคัญมาก)

**กุญแจสำคัญคือใช้ช่วง "รอ" ให้เป็นประโยชน์** — ระหว่างที่ Docker ดึง image
และ `install.sh` ทำงาน วิทยากรบรรยายทฤษฎีไปพร้อมกันได้

```
13:00–13:15  🎬 เปิดหัวข้อ   วิทยากรอธิบายว่าบ่ายนี้จะทำอะไร
                            แจกใบ IP · ทุกคน SSH เข้าเครื่องตัวเองให้ได้

13:15–13:40  ระยะที่ 1     เตรียมระบบปฏิบัติการ (ขั้น 1–6)
                            ตรวจสเปก · อัปเดตระบบ · สร้าง swap 16 GB

13:40–14:00  ระยะที่ 2     ติดตั้ง Docker ด้วยตนเอง (ขั้น 7–10)

14:00–14:15  ระยะที่ 3     clone repo + checkout tag + เปิด tmux
                            + สั่ง docker compose pull (ขั้น 11–14)
                            ⬇ ปล่อยให้ดึง image ทำงานเบื้องหลัง 5–15 นาที

14:15–14:35  ⏳ ระหว่างรอ   บรรยาย "สถาปัตยกรรมภายในของ Sentry"
                            PostgreSQL · ClickHouse · Kafka · Snuba · Relay
                            ทำไมต้องมีถึง 80+ container

14:35–15:15  ระยะที่ 4     รัน install.sh (ขั้น 15–18)
                            ⏳ ระหว่างรอ 20–30 นาที → อภิปราย
                              "SaaS กับ Self-hosted ต่างกันอย่างไร"
                              + วางแผนติดตั้งจริงที่ BCEL

── พักเบรก 15:15–15:25 ──

15:25–15:45  ระยะที่ 5     ตั้งค่าหลังติดตั้ง + สร้าง Org/Project/DSN (ขั้น 19–22)

15:45–16:05  ระยะที่ 6–7   ⭐ ย้าย Spring Boot และ Angular
                            จาก SaaS มาใช้ Sentry ของตัวเอง (ขั้น 23–30)
                            เปลี่ยนแค่ DSN เพราะฝัง SDK ไว้แล้วตั้งแต่วันที่ 1–2

16:05–16:20  ระยะที่ 8     พิสูจน์ End-to-End (ขั้น 31–32)

16:20–16:30  🏁 ปิดหลักสูตร  สรุป · ตอบคำถาม · วิทยากรลบ droplet ทั้งหมด
```

> 🔑 **หลักการออกแบบตารางนี้** ทุกช่วงที่ต้องรอเครื่องทำงาน จะมีเนื้อหาบรรยายคู่ขนานเสมอ
> ผู้เรียนจึงไม่ได้นั่งดูแถบ progress เฉย ๆ

> ⏱️ **ถ้าเวลาไม่พอจริง ๆ** ให้ตัดตามลำดับนี้
> 1. ระยะที่ 8 (พิสูจน์ trace 3 ชั้น) — ผู้เรียนเห็นมาแล้วในวันที่ 2
> 2. ระยะที่ 7 (Angular) — เอาแค่ Spring Boot ก็พิสูจน์แนวคิดได้แล้ว
> 3. ขั้นที่ 20 (ปิด public registration) — อธิบายด้วยปากแทนการลงมือ
> **ห้ามตัด** ขั้นที่ 5 (swap) และขั้นที่ 19 (`system.url-prefix`) เด็ดขาด

### 0.3 🌐 เส้นทางของข้อมูล — อะไรวิ่งผ่านเน็ตสถาบันบ้าง

**ประเด็นที่เข้าใจผิดกันบ่อย** และเป็นประเด็นที่ดีสำหรับสอนเรื่องสถาปัตยกรรมด้วย

```
┌─ เน็ตสถาบัน (ช่องแคบ) ────────────────────────────────────┐
│                                                            │
│  โน้ตบุ๊กผู้เรียน                                            │
│      │  SSH terminal            ระดับ KB/s   เบามาก        │
│      │  เบราว์เซอร์ → พอร์ต 9000   ไม่กี่ MB ต่อหน้า          │
│      │  แอปส่ง event → :9000     ระดับ KB ต่อ event         │
└──────┼─────────────────────────────────────────────────────┘
       │
       ▼
┌─ DigitalOcean sgp1 ────────────────────────────────────────┐
│  Droplet ของผู้เรียน                                        │
│      │                                                     │
│      │  ⭐ docker compose pull  8–10 GB                     │
│      ▼     วิ่งบน backbone ของ DigitalOcean                 │
│  ghcr.io + Docker Hub          ไม่แตะเน็ตสถาบันเลย           │
└────────────────────────────────────────────────────────────┘
```

> ✅ **การดึง image 8–10 GB เกิดขึ้นบน droplet ทั้งหมด** ไม่ผ่านเน็ตสถาบันแม้แต่ไบต์เดียว
> ความเร็วจึงขึ้นกับ backbone ของ DigitalOcean ซึ่งเร็วมาก — **ปกติ 5–15 นาที**
> และการที่ผู้เรียน 5 คนดึงพร้อมกัน **ไม่กระทบกันเลย** เพราะเป็นคนละเครื่อง คนละ public IP

> 💬 **ใช้เป็นประเด็นสอนได้** ถามผู้เรียนก่อนเริ่มว่า *"เดี๋ยวเราจะโหลด 8–10 GB
> เน็ตห้องนี้จะรับไหวไหม"* แล้วให้ช่วยกันคิดว่าจริง ๆ แล้วข้อมูลวิ่งเส้นทางไหน
> เป็นการฝึกคิดเรื่อง network topology ที่จะได้ใช้ตอนวางระบบจริงที่ธนาคาร

### 0.4 ⚠️ ความเสี่ยงที่ต้องเตรียมรับมือ

| ความเสี่ยง | โอกาสเกิด | แผนรับมือ |
| --- | --- | --- |
| **SSH หลุดระหว่างงานยาว** | **สูง** | ⭐ **ใช้ `tmux` เสมอ** (ขั้นที่ 13) — WiFi ห้องอบรมไม่นิ่ง |
| RAM ไม่พอตอน migrate | ปานกลาง | ต้องสร้าง swap ตั้งแต่ขั้นที่ 5 |
| ผู้เรียน 1–2 คนติดตั้งไม่ผ่าน | ปานกลาง | จับคู่กับเพื่อน แล้วซ่อมตอนพักเที่ยง |
| ผู้เรียนพิมพ์คำสั่งผิดแล้วไล่หาไม่เจอ | ปานกลาง | วิทยากรมี SSH key เข้าได้ทุกเครื่อง เข้าไปช่วยได้ |
| Cloud Firewall ตั้ง IP สถาบันผิด | ปานกลาง | ตรวจด้วย `curl ifconfig.me` จากโน้ตบุ๊กก่อนเริ่ม |
| ติดตั้งไม่ผ่านเกินครึ่งห้อง | ต่ำ | **สลับไปใช้ SaaS ทันที** |
| ดึง image ช้าผิดปกติ | **ต่ำ** | เป็นปัญหาฝั่ง DigitalOcean/ghcr.io ไม่ใช่เน็ตสถาบัน · รอหรือ retry |

> 💡 **แนะนำอย่างยิ่ง** ให้วิทยากรคง org บน SaaS (`itgenius-qc`) ไว้เป็น **แผนสำรอง**
> ต้นทุนเป็นศูนย์ และช่วยชีวิตได้ถ้าวันจริงมีปัญหา

---

## 1. ข้อกำหนดจริงของ Sentry Self-hosted

### 1.1 ตัวเลขที่ `install.sh` ตรวจจริง

อ่านจาก `install/_min-requirements.sh` โดยตรง — **ถ้าไม่ผ่าน สคริปต์หยุดทันที**

```bash
MIN_DOCKER_VERSION='19.03.6'
MIN_COMPOSE_VERSION='2.32.2'
MIN_BASH_VERSION='4.4.0'

if [[ "$COMPOSE_PROFILES" == "errors-only" ]]; then
  MIN_RAM_HARD=7000    # MB
  MIN_CPU_HARD=2
else
  MIN_RAM_HARD=14000   # MB
  MIN_CPU_HARD=4
fi
```

| เงื่อนไข | `feature-complete` (ค่าปริยาย) | `errors-only` |
| --- | --- | --- |
| RAM ที่ Docker มองเห็น | **≥ 14,000 MB** | ≥ 7,000 MB |
| CPU cores | **≥ 4** | ≥ 2 |
| Docker | ≥ 19.03.6 | ≥ 19.03.6 |
| Docker Compose | ≥ 2.32.2 | ≥ 2.32.2 |
| Bash | ≥ 4.4.0 | ≥ 4.4.0 |
| Disk ว่าง | ≥ 20 GB | ≥ 20 GB |
| CPU instruction set | **SSE 4.2** (ClickHouse บังคับ) | SSE 4.2 |

> ⛔ **หลักสูตรนี้ต้องใช้ `feature-complete` เท่านั้น**
> เพราะ `errors-only` **ปิด Performance Monitoring และ Distributed Tracing**
> ซึ่งเป็นเนื้อหาหลักของวันที่ 2 ทั้งวัน
> ผลคือ **RAM 16 GB เป็นขั้นต่ำจริง** — droplet 8 GB ใช้ไม่ได้

### 1.2 ระบบปฏิบัติการ

| OS | สถานะ |
| --- | --- |
| **Ubuntu / Debian** | ✅ แนะนำ — ใช้ในเอกสารนี้ (Ubuntu 24.04 LTS) |
| RHEL / CentOS / Rocky / Alma | ⚠️ มีปัญหาที่ทราบแล้ว ต้องแก้เอง |
| Alpine | ❌ ไม่รองรับ |
| Windows (Git Bash / MSYS2) | ❌ `install.sh` ปฏิเสธตั้งแต่บรรทัดแรก ต้องใช้ WSL |

> 🏦 **ประเด็นสำหรับ BCEL** สภาพแวดล้อมจริงคือ **CentOS 8** ซึ่งอยู่ในกลุ่ม
> "มีปัญหาที่ทราบแล้ว" **และหมด EOL ไปแล้ว** — อภิปรายทางเลือกจริงในหัวข้อ 13.4

### 1.3 ขนาดของระบบที่กำลังจะติดตั้ง

`feature-complete` รัน container ประมาณ **80+ ตัว**

| กลุ่ม | บริการ |
| --- | --- |
| **Data store** | PostgreSQL · pgbouncer · ClickHouse · Redis · Kafka · SeaweedFS · Memcached |
| **Sentry core** | web · nginx · relay · smtp |
| **Snuba** (query layer) | snuba-api + consumer/subscription อีก ~30 ตัว |
| **Task processing** | taskbroker · taskworker · taskscheduler · sentry-cleanup |
| **Ingest** | events · attachments · transactions · metrics · replays · monitors · feedback · spans |
| **เสริม** | symbolicator (แปล stack trace) · vroom (profiling) · uptime-checker · launchpad |

---

## 2. งานเดียวที่วิทยากรต้องทำ สร้าง Droplet เปล่า

> 🖱️ **เอกสารนี้ใช้หน้าเว็บ `digitalocean.com` ไม่ใช้ `doctl`**
> ทำครั้งเดียวได้ทั้ง 5 เครื่องด้วยช่อง Quantity — ใช้เวลารวมประมาณ 15 นาที

### 2.1 ⏰ ทำเมื่อไร

| เวลา | ทำอะไร |
| --- | --- |
| **เช้าวันที่ 3 ประมาณ 08:30 น.** | สร้าง droplet ทั้ง 5 เครื่อง + Firewall |
| 13:00 น. | แจกใบ IP ให้ผู้เรียน |
| **16:30 น. ทันทีที่จบ** | ⚠️ **ลบทิ้งทั้งหมด** |

> 💡 **อย่าสร้างล่วงหน้าหลายวัน** เพราะ DigitalOcean คิดเงินตั้งแต่วินาทีที่สร้าง
> สร้างเช้าวันที่ใช้ก็เพียงพอ ใช้เวลาแค่ 2–3 นาทีต่อชุด

### 2.2 ⭐ ตารางค่าที่ต้องกรอกในหน้า Create Droplet

เข้า **digitalocean.com → Manage → Droplets → Create Droplet**
แล้วกรอกตามนี้ทีละหัวข้อ

| # | หัวข้อบนหน้าเว็บ | ค่าที่เลือก | เหตุผล |
| --- | --- | --- | --- |
| 1 | **Choose Region** | **Singapore · SGP1** | ใกล้ไทย/ลาวที่สุด latency 30–50 ms |
| 2 | **Datacenter** | ปล่อยค่าปริยาย | ไม่มีผลกับ Lab |
| 3 | **Choose an image** | **Ubuntu · 24.04 (LTS) x64** | ✅ OS เดียวที่ Sentry รองรับเต็มที่ |
| 4 | **Choose Size → ประเภท** | **Basic** | ถูกที่สุดที่ผ่านเกณฑ์ |
| 5 | **CPU options** | **Regular · SSD** | ไม่ต้องใช้ Premium |
| 6 | **ขนาด** | ⭐ **$96/mo · 8 vCPU · 16 GB · 320 GB SSD** | **RAM 16 GB คือขั้นต่ำจริง** |
| 7 | **Choose Authentication Method** | **Password** | ผู้เรียนใช้รหัสผ่าน SSH ได้เลย |
| 8 | **Create root password** | `BcelSentry2026!` | จดไว้แจกผู้เรียน |
| 9 | **Advanced Options** | ไม่ต้องเปิด | ไม่ต้องใช้ cloud-init เพราะเลือก Password แล้ว |
| 10 | **Quantity** | ⭐ **5 Droplets** | สร้างพร้อมกันทีเดียว |
| 11 | **Hostname** | ดูตารางชื่อข้างล่าง | |
| 12 | **Tags** | `sentry-lab` · `bcel-2026` | ⭐ ใช้กรองตอนลบทีเดียว |
| 13 | **Project** | สร้างใหม่ชื่อ `BCEL-Sentry-Training` | แยกออกจากงานอื่นในบัญชี |
| 14 | **Backups** | ❌ **ไม่ติ๊ก** | เสียเงินเพิ่ม 20% โดยไม่จำเป็น |

> 🔑 **ข้อ 7 สำคัญ** ถ้าเลือก **SSH Key** ผู้เรียนจะเข้าเครื่องไม่ได้เพราะไม่มี private key
> การเลือก **Password** ทำให้ DigitalOcean เปิด `PasswordAuthentication` ให้เองตั้งแต่บูตแรก
> **ไม่ต้องยุ่งกับ cloud-init หรือแก้ `sshd_config` เลย**

### 2.3 ชื่อเครื่อง (Hostname)

ช่อง Hostname รองรับหลายบรรทัด — พิมพ์ 5 บรรทัดนี้ลงไป

```
sentry-lab-01
sentry-lab-02
sentry-lab-03
sentry-lab-04
sentry-lab-05
```

> 📛 **หลักการตั้งชื่อ** ใช้รูปแบบ `<งาน>-<บทบาท>-<เลขลำดับ 2 หลัก>`
> - มีเลข 2 หลัก (`01` ไม่ใช่ `1`) เพื่อให้เรียงลำดับถูกต้อง
> - ไม่ใส่ชื่อผู้เรียนลงไป เพราะถ้ามีคนขาดจะสับสน — ใช้เลขแล้วจับคู่ในใบแจกแทน
> - ตั้งชื่อให้ค้นหาง่ายด้วย `grep sentry-lab` ตอนลบ

### 2.4 สรุปสเปกที่เลือกและทางเลือกอื่น

| แผน | vCPU | RAM | SSD | ราคา/ชม. | ราคา/เดือน | ใช้ได้ไหม |
| --- | --- | --- | --- | --- | --- | --- |
| Basic $48 | 4 | 8 GB | 160 GB | $0.07143 | $48 | ❌ **ไม่ผ่าน** RAM < 14 GB |
| ⭐ **Basic $96** | **8** | **16 GB** | **320 GB** | **$0.14286** | **$96** | ✅ **ใช้ตัวนี้** |
| General Purpose $126 | 4 | 16 GB | 50 GB | $0.18750 | $126 | ✅ ผ่าน แต่แพงกว่าและ disk เล็ก |
| Basic $192 | 8 | 32 GB | 640 GB | $0.28571 | $192 | ✅ สบายที่สุด แต่เกินจำเป็น |

> ⚠️ **ห้ามประหยัดด้วยการเลือก $48** — `install.sh` จะหยุดทันทีพร้อมข้อความ
> `FAIL: Required minimum RAM available to Docker is 14000 MB`
> เพราะโปรไฟล์ `feature-complete` บังคับ RAM ≥ 14,000 MB

**ค่าใช้จ่ายจริงของบ่ายวันเดียว**

| รายการ | คำนวณ | ค่าใช้จ่าย |
| --- | --- | --- |
| 5 เครื่อง × 8 ชั่วโมง (08:30–16:30) | $0.14286 × 8 × 5 | **$5.71** |

> 💰 **ถูกกว่าที่คิดมาก** เพราะใช้แค่บ่ายเดียว ไม่ใช่ 3 วัน — ประมาณ **210 บาท**

### 2.5 หา IP ทั้ง 5 เครื่อง

หลังกด Create รอประมาณ 1–2 นาที แล้วดูที่ **Manage → Droplets**

| Name | IP Address | ผู้เรียน |
| --- | --- | --- |
| sentry-lab-01 | ____________ | ____________ |
| sentry-lab-02 | ____________ | ____________ |
| sentry-lab-03 | ____________ | ____________ |
| sentry-lab-04 | ____________ | ____________ |
| sentry-lab-05 | ____________ | ____________ |

> 💡 คลิกที่ IP บนหน้าเว็บจะคัดลอกได้ทันที

### 2.6 🔒 สร้าง Cloud Firewall

**ขั้นแรก หา IP ขาออกของสถาบัน** — เปิดเบราว์เซอร์บนเครื่องที่ใช้ในห้องอบรม แล้วเข้า

```
https://ifconfig.me
```

> ⚠️ **ต้องเช็คจากเครือข่ายของห้องอบรมจริง** ไม่ใช่จากบ้านหรือมือถือ

**สร้าง Firewall** — ไปที่ **Networking → Firewalls → Create Firewall**

| หัวข้อ | ค่าที่กรอก |
| --- | --- |
| **Name** | `sentry-lab-fw` |

**Inbound Rules** — ลบกฎปริยายออกก่อน แล้วเพิ่ม 2 กฎนี้

| Type | Protocol | Port Range | Sources |
| --- | --- | --- | --- |
| SSH | TCP | `22` | `<IP สถาบัน>/32` |
| Custom | TCP | `9000` | `<IP สถาบัน>/32` |

**Outbound Rules** — ปล่อยค่าปริยายไว้ (ต้องให้ droplet ดึง Docker image ได้)

| Type | Protocol | Port Range | Destinations |
| --- | --- | --- | --- |
| All TCP | TCP | All ports | `0.0.0.0/0`, `::/0` |
| All UDP | UDP | All ports | `0.0.0.0/0`, `::/0` |

**Apply to Droplets** — พิมพ์ tag `sentry-lab` ในช่องค้นหา จะเลือกครบทั้ง 5 เครื่องทีเดียว

> 🔒 **ห้ามเปิด `0.0.0.0/0` ในกฎ Inbound เด็ดขาด**
> Sentry ที่เพิ่งติดตั้งยังไม่มี TLS · หน้า login เปิดให้สมัครได้ · และไม่มี rate limit
> ถ้าเปิดสู่อินเทอร์เน็ต จะถูกสแกนภายในไม่กี่นาที

> 💬 **ประเด็นสอนที่ดี** ให้ผู้เรียนดูหน้า Firewall นี้แล้วถามว่า
> *"ถ้าเป็นระบบจริงที่ธนาคาร กฎชุดนี้ควรเป็นอย่างไร"*
> (คำตอบ: ไม่เปิดสู่อินเทอร์เน็ตเลย · อยู่หลัง VPN หรือ internal network เท่านั้น)

### 2.7 ✅ Checklist ก่อนเริ่มสอน

- [ ] สร้าง droplet ครบ 5 เครื่อง ขนาด **16 GB** ทุกเครื่อง
- [ ] ทุกเครื่องขึ้นสถานะ **Active** สีเขียว
- [ ] ติด tag `sentry-lab` ครบทุกเครื่อง
- [ ] สร้าง Firewall `sentry-lab-fw` และ apply ด้วย tag แล้ว
- [ ] **ทดสอบ `ssh root@<IP เครื่องแรก>` จากเครือข่ายห้องอบรมได้จริง**
- [ ] กรอกตาราง IP ในข้อ 2.5 ครบ
- [ ] พิมพ์ใบแจกผู้เรียน 5 ใบ
- [ ] ⏰ **ตั้งเตือนในปฏิทิน 16:30 น. ว่าต้องลบ droplet**

### 2.8 ใบแจกผู้เรียน (พิมพ์ 5 ใบ)

```
╔══════════════════════════════════════════════════════════╗
║  Sentry Self-hosted Lab · บ่ายวันที่ 3                     ║
║                                                          ║
║  เครื่องของท่าน : sentry-lab-01                           ║
║  IP address     : xxx.xxx.xxx.xxx                        ║
║  SSH user       : root                                   ║
║  SSH password   : BcelSentry2026!                        ║
║                                                          ║
║  คำสั่งเข้าเครื่อง                                          ║
║      ssh root@xxx.xxx.xxx.xxx                            ║
║                                                          ║
║  ⚠️ เครื่องนี้มีแค่ Ubuntu 24.04 เปล่า ๆ                    ║
║     ยังไม่มี Docker · ไม่มี swap · ไม่มีอะไรเลย             ║
║     ท่านจะติดตั้งทุกอย่างเองตั้งแต่ต้น                       ║
║                                                          ║
║  เมื่อติดตั้งเสร็จ จะเข้าใช้งานที่                            ║
║      http://xxx.xxx.xxx.xxx:9000                         ║
║                                                          ║
║  ⏰ เครื่องนี้จะถูกลบเวลา 16:30 น. วันนี้                    ║
║     กรุณาบันทึกสิ่งที่ต้องการเก็บก่อนหมดเวลา                 ║
╚══════════════════════════════════════════════════════════╝
```

**บน Windows ใช้อะไร SSH**

| เครื่องมือ | คำสั่ง |
| --- | --- |
| **PowerShell** (มีมาให้แล้วใน Windows 10/11) | `ssh root@<IP>` |
| Windows Terminal | `ssh root@<IP>` |
| PuTTY | Host Name = `<IP>` · Port = 22 |
| Git Bash | `ssh root@<IP>` |

> 💡 **แนะนำให้ใช้ PowerShell** เพราะไม่ต้องติดตั้งอะไรเพิ่ม
> ครั้งแรกจะถามว่า `Are you sure you want to continue connecting (yes/no)?` ให้พิมพ์ `yes`

> ✅ **จบงานของวิทยากรเพียงเท่านี้** ที่เหลือเป็นของผู้เรียนทั้งหมด

---

## 3. ระยะที่ 1 เตรียมระบบปฏิบัติการ (ขั้นที่ 1–6)

> ⏱️ 30 นาที · ผู้เรียนพิมพ์เองทุกคำสั่ง **ห้ามคัดลอกวาง**

---

### ขั้นที่ 1 — เข้าเครื่องของตัวเอง

```bash
ssh root@<IP ของท่าน>
```

รหัสผ่าน: `BcelSentry2026!`

**ตรวจว่าเข้าถูกเครื่อง**

```bash
hostname && curl -s ifconfig.me && echo
```

> ✅ ต้องได้ชื่อเครื่องและ IP ตรงกับใบแจก

**ดูว่าเครื่องนี้มีอะไรอยู่บ้าง (คำตอบคือ แทบไม่มีอะไรเลย)**

```bash
which docker git curl
```

> 💬 **ประเด็นที่ควรพูด** `docker` และ `git` ยังไม่มี — เราจะติดตั้งเองทั้งหมด
> นี่คือสภาพเดียวกับ VM เปล่าที่ทีม Infra ของธนาคารจะส่งมอบให้ท่าน

---

### ขั้นที่ 2 — ตรวจสเปกเครื่องว่าผ่านเกณฑ์

**ก่อนติดตั้งอะไรก็ตาม ต้องตรวจก่อนเสมอ** นี่คือนิสัยที่ต้องติดตัวไปใช้งานจริง

```bash
nproc
```

```bash
free -h
```

```bash
df -h /
```

```bash
lsb_release -a
```

**เติมค่าที่ได้ลงตาราง**

| ตรวจอะไร | ต้องได้ | เครื่องท่านได้เท่าไร |
| --- | --- | --- |
| CPU cores | ≥ 4 | ______ |
| RAM | ≥ 14 GB | ______ |
| Swap | (ยังเป็น 0 — เราจะสร้างเอง) | ______ |
| Disk ว่าง | ≥ 20 GB | ______ |
| OS | Ubuntu 22.04 / 24.04 | ______ |

**ตรวจ SSE 4.2 ที่ ClickHouse บังคับ**

```bash
grep -c sse4_2 /proc/cpuinfo
```

> ✅ ต้องได้ตัวเลข **มากกว่า 0** (ปกติเท่ากับจำนวน core)
> ถ้าได้ `0` แปลว่า CPU ไม่รองรับ ต้องเปลี่ยนเครื่อง
> มี flag `--skip-sse42-requirements` ให้ข้ามได้ แต่ **ไม่แนะนำ** เพราะ ClickHouse อาจ crash ตอนใช้งานจริง

---

### ขั้นที่ 3 — อัปเดตระบบและติดตั้งเครื่องมือพื้นฐาน

```bash
apt update && apt upgrade -y
```

> ⏱️ ใช้เวลา 2–5 นาที · ถ้าขึ้นหน้าจอสีม่วงถามเรื่อง service ให้กด **Enter** เลือกค่าปริยาย

```bash
apt install -y git curl ca-certificates gnupg tmux htop sysstat
```

**ตรวจว่าติดตั้งสำเร็จ**

```bash
git --version && curl --version | head -1 && tmux -V
```

| เครื่องมือ | ใช้ทำอะไรในหลักสูตรนี้ |
| --- | --- |
| `git` | โคลน repository ของ Sentry |
| `curl` | ดาวน์โหลดสคริปต์ · ทดสอบ health endpoint |
| **`tmux`** | ⭐ **สำคัญมาก** — ทำให้งานยาวไม่ตายเมื่อ SSH หลุด |
| `htop` · `sysstat` | เฝ้าดู CPU/RAM ระหว่างติดตั้ง |

---

### ขั้นที่ 4 — สร้างผู้ใช้ที่ไม่ใช่ root

**ห้ามรัน Docker และ Sentry ด้วย root** เป็นหลักปฏิบัติพื้นฐานด้านความปลอดภัย

```bash
adduser --gecos "" sentry-admin
```

> ระบบจะถามรหัสผ่าน ให้ตั้งเป็น `BcelSentry2026!` (หรือรหัสของท่านเอง จดไว้ให้ดี)

```bash
usermod -aG sudo sentry-admin
```

```bash
id sentry-admin
```

> ✅ ต้องเห็น `sudo` อยู่ในรายการกลุ่ม
> (ยังไม่มีกลุ่ม `docker` เพราะยังไม่ได้ติดตั้ง Docker — จะเพิ่มในขั้นที่ 9)

---

### ขั้นที่ 5 — ⭐ สร้าง Swap 16 GB

Sentry กำหนดว่าต้องมี **RAM 16 GB + swap 16 GB** — droplet ของ DigitalOcean **ไม่มี swap มาให้**

```bash
free -h
```

> สังเกตบรรทัด `Swap:` ตอนนี้เป็น `0B`

```bash
fallocate -l 16G /swapfile
```

```bash
chmod 600 /swapfile
```

```bash
mkswap /swapfile
```

```bash
swapon /swapfile
```

**ทำให้ swap กลับมาเองหลัง reboot**

```bash
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

**ตรวจผล**

```bash
free -h
```

> ✅ บรรทัด `Swap:` ต้องเป็น `16Gi`

**ปรับ swappiness ให้ใช้ swap เฉพาะตอนจำเป็นจริง**

```bash
sysctl vm.swappiness=10 && echo 'vm.swappiness=10' >> /etc/sysctl.conf
```

> 💬 **ทำไมต้องมี swap ทั้งที่ RAM 16 GB แล้ว**
> ขั้นตอน database migration ของ Sentry มีช่วงที่ใช้ RAM สูงมากชั่วขณะ
> ถ้าไม่มี swap รองรับ kernel จะสั่ง OOM-kill process แล้ว `install.sh` จะตายกลางคัน
> **นี่คือสาเหตุอันดับหนึ่งที่คนติดตั้งไม่สำเร็จ**

---

### ขั้นที่ 6 — ตรวจความพร้อมของระบบปฏิบัติการ

```bash
echo "CPU: $(nproc) cores" && free -h | grep -E 'Mem|Swap' && df -h / | tail -1 && echo "SSE4.2: $(grep -c sse4_2 /proc/cpuinfo)"
```

**✅ เกณฑ์ผ่านระยะที่ 1**

| # | ต้องได้ |
| --- | --- |
| 1 | CPU ≥ 4 cores |
| 2 | RAM ≥ 14 GB |
| 3 | **Swap = 16 GB** |
| 4 | Disk ว่าง ≥ 20 GB |
| 5 | SSE4.2 > 0 |
| 6 | มี `git` `curl` `tmux` |
| 7 | มีผู้ใช้ `sentry-admin` อยู่ในกลุ่ม `sudo` |

---

## 4. ระยะที่ 2 ติดตั้ง Docker (ขั้นที่ 7–10)

> ⏱️ 20 นาที

---

### ขั้นที่ 7 — เลือกวิธีติดตั้ง Docker (ประเด็นความปลอดภัยที่ควรอภิปราย)

Docker มีวิธีติดตั้งหลัก 2 แบบ

| วิธี | คำสั่ง | ข้อดี | ข้อเสีย |
| --- | --- | --- | --- |
| **A. Convenience script** | `curl -fsSL https://get.docker.com \| sh` | เร็ว · 1 คำสั่ง | ⚠️ **ดาวน์โหลดสคริปต์มารันด้วยสิทธิ์ root ทันที** |
| **B. APT repository** | เพิ่ม GPG key + repo แล้ว `apt install` | ✅ ตรวจสอบลายเซ็นได้ · อัปเดตผ่าน apt ปกติ | ยาวกว่า 5 คำสั่ง |

> 🏦 **ประเด็นอภิปรายในห้อง (5 นาที)**
> *"ถ้าท่านเป็นผู้ดูแลระบบของธนาคาร ท่านจะยอมให้ทีมรัน `curl … | sh` บนเซิร์ฟเวอร์ production ไหม"*
> คำตอบที่ถูกต้องคือ **ไม่** เพราะเป็นการรันโค้ดที่ยังไม่ได้ตรวจสอบด้วยสิทธิ์สูงสุด
> **หลักสูตรนี้จึงใช้วิธี B** เพื่อให้ตรงกับมาตรฐานที่ธนาคารควรใช้

---

### ขั้นที่ 8 — ติดตั้ง Docker ด้วย APT repository (วิธีที่แนะนำ)

**8.1 เพิ่ม GPG key ทางการของ Docker**

```bash
install -m 0755 -d /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

```bash
chmod a+r /etc/apt/keyrings/docker.asc
```

**8.2 เพิ่ม repository**

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" > /etc/apt/sources.list.d/docker.list
```

```bash
cat /etc/apt/sources.list.d/docker.list
```

> ✅ ต้องเห็นบรรทัดที่มี `signed-by=` และชื่อ codename เช่น `noble` (Ubuntu 24.04)

**8.3 ติดตั้ง**

```bash
apt update
```

```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> ⏱️ ใช้เวลา 2–4 นาที · ดาวน์โหลดประมาณ 120 MB

| package | หน้าที่ |
| --- | --- |
| `docker-ce` | Docker daemon |
| `docker-ce-cli` | คำสั่ง `docker` |
| `containerd.io` | container runtime ชั้นล่าง |
| `docker-buildx-plugin` | ระบบ build รุ่นใหม่ |
| **`docker-compose-plugin`** | ⭐ คำสั่ง `docker compose` — **ขาดไม่ได้** |

---

### ขั้นที่ 9 — ตรวจ Docker และเพิ่มผู้ใช้เข้ากลุ่ม

```bash
systemctl is-active docker
```

> ✅ ต้องได้ `active`

```bash
systemctl is-enabled docker
```

> ✅ ต้องได้ `enabled` (จะเริ่มเองเมื่อ reboot)

**ตรวจเวอร์ชันเทียบกับเกณฑ์ของ `install.sh`**

```bash
docker version --format '{{.Server.Version}}'
```

```bash
docker compose version
```

```bash
echo $BASH_VERSION
```

| เครื่องมือ | ขั้นต่ำที่ `install.sh` บังคับ | เครื่องท่านได้เท่าไร |
| --- | --- | --- |
| Docker | 19.03.6 | ______ |
| Docker Compose | **2.32.2** | ______ |
| Bash | 4.4.0 | ______ |

> ⚠️ **Docker Compose ต้อง ≥ 2.32.2** ถ้า Ubuntu ให้เวอร์ชันเก่ากว่านี้
> ต้องอัปเดตด้วย `apt update && apt upgrade docker-compose-plugin`

**เพิ่ม `sentry-admin` เข้ากลุ่ม docker**

```bash
usermod -aG docker sentry-admin
```

```bash
id sentry-admin
```

> ✅ ต้องเห็นทั้ง `sudo` และ `docker`

---

### ขั้นที่ 10 — ทดสอบ Docker ในฐานะผู้ใช้ธรรมดา

**ออกจาก root แล้วเข้าใหม่ในฐานะ `sentry-admin`**

```bash
exit
```

```bash
ssh sentry-admin@<IP ของท่าน>
```

> 💬 **ทำไมต้อง logout แล้ว login ใหม่** เพราะการเพิ่มกลุ่มมีผลเมื่อเริ่ม session ใหม่เท่านั้น
> ถ้าใช้ `su - sentry-admin` เฉย ๆ จะยังไม่เห็นกลุ่ม `docker`

```bash
groups
```

> ✅ ต้องเห็น `docker` ในรายการ

```bash
docker run --rm hello-world
```

> ✅ ต้องเห็นข้อความ `Hello from Docker!` **โดยไม่ต้องใช้ `sudo`**

```bash
docker compose version
```

**✅ เกณฑ์ผ่านระยะที่ 2**

| # | ต้องได้ |
| --- | --- |
| 1 | `docker` service `active` และ `enabled` |
| 2 | Docker ≥ 19.03.6 |
| 3 | Docker Compose ≥ 2.32.2 |
| 4 | รัน `docker run hello-world` ได้โดยไม่ต้อง sudo |

---

## 5. ระยะที่ 3 เตรียม Sentry (ขั้นที่ 11–14)

> ⏱️ ลงมือ 15 นาที + **รอ 5–15 นาที** (ใช้ช่วงรอบรรยาย "สถาปัตยกรรมภายในของ Sentry")

---

### ขั้นที่ 11 — โคลน repository

```bash
cd ~
```

```bash
git clone https://github.com/getsentry/self-hosted.git
```

```bash
cd self-hosted && ls
```

> 💬 ให้ผู้เรียนดูว่ามีอะไรบ้าง — `install.sh` · `docker-compose.yml` · `.env` · โฟลเดอร์ `install/` และ `sentry/`

---

### ขั้นที่ 12 — ⭐ เลือกเวอร์ชันให้ถูก (ห้ามใช้ master)

```bash
git branch --show-current
```

> ⚠️ ตอนนี้อยู่บน `master` ซึ่งเป็น **nightly build ที่ยังไม่ผ่านการทดสอบ**

**หาเวอร์ชัน release ล่าสุด**

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/getsentry/self-hosted/releases/latest); VERSION=${VERSION##*/}; echo "เวอร์ชันล่าสุดคือ: $VERSION"
```

> ณ กรกฎาคม 2569 จะได้ **`26.7.2`**

```bash
git checkout "$VERSION"
```

```bash
git describe --tags
```

> ✅ ต้องได้เลขเวอร์ชัน ไม่ใช่ชื่อ branch

> 🏦 **หลักปฏิบัติที่ต้องนำกลับไปใช้ที่ธนาคาร**
> ระบบ production **ต้อง pin เป็น tag เสมอ** ห้ามใช้ `master` หรือ `latest`
> เพราะจะทำให้ระบบเปลี่ยนแปลงโดยไม่ตั้งใจ และย้อนกลับไม่ได้เวลาเกิดปัญหา

**ดูว่ากำลังจะติดตั้งอะไรบ้าง**

```bash
docker compose config --services | wc -l
```

> 💬 ตัวเลขที่ได้คือจำนวน container ที่กำลังจะรัน — ประมาณ **80+ ตัว**
> ให้ผู้เรียนหยุดคิดสักครู่ว่าระบบนี้ใหญ่แค่ไหน และใครจะเป็นคนดูแลที่ธนาคาร

---

### ขั้นที่ 13 — ⭐⭐ เปิด tmux ก่อนสั่งงานยาว (ห้ามข้าม)

**นี่คือขั้นตอนที่ช่วยชีวิตเมื่อ WiFi ห้องอบรมหลุด**

```bash
tmux new -s sentry
```

> จะเห็นแถบสีเขียวด้านล่างจอ แปลว่าอยู่ใน tmux session ชื่อ `sentry` แล้ว

**คำสั่ง tmux ที่ต้องจำ**

| ต้องการ | กดปุ่ม |
| --- | --- |
| ออกจาก tmux แต่ปล่อยงานทำงานต่อ | `Ctrl+b` แล้วปล่อย แล้วกด `d` |
| กลับเข้า session เดิม | `tmux attach -t sentry` |
| ดูว่ามี session อะไรอยู่บ้าง | `tmux ls` |

> 🔑 **ถ้า SSH หลุดโดยไม่ได้ใช้ tmux** งานที่กำลังรันจะถูก kill ทันที
> และ `install.sh` ที่ตายกลางคันระหว่าง database migration จะกู้ยากมาก

---

### ขั้นที่ 14 — ดึง Docker image ล่วงหน้า

```bash
docker compose pull
```

> ⏱️ **ใช้เวลา 5–15 นาที** ดาวน์โหลดประมาณ **8–10 GB**

> 🌐 **ข้อมูล 8–10 GB นี้ไม่ผ่านเน็ตของสถาบันเลย** — ดึงจาก ghcr.io และ Docker Hub
> เข้า droplet ที่ DigitalOcean Singapore โดยตรง วิ่งบน backbone ของ DigitalOcean
> ความเร็วจึงสูงมาก และผู้เรียน 5 คนดึงพร้อมกันก็ไม่กระทบกัน เพราะเป็นคนละเครื่อง คนละ public IP
> (ดูแผนผังเส้นทางข้อมูลในหัวข้อ 0.3)

**ระหว่างนี้**

1. กด `Ctrl+b` แล้ว `d` เพื่อออกจาก tmux (งานยังทำงานต่อ)
2. ฟังวิทยากรบรรยาย **สถาปัตยกรรมภายในของ Sentry**
3. เป็นระยะให้กลับเข้าไปดูด้วย `tmux attach -t sentry`

**เมื่อดึงเสร็จ ตรวจผล**

```bash
docker images | wc -l
```

```bash
docker system df
```

> ✅ ควรเห็น image 25–35 รายการ ขนาดรวม 8–10 GB

> 💬 **ทำไมต้องดึงแยกก่อน ทั้งที่ `install.sh` ดึงเองได้**
> จริงอยู่ว่า `install.sh` มีขั้นตอน `update-docker-images.sh` ที่ดึงให้อยู่แล้ว
> แต่การแยกออกมาทำให้ **เห็น progress ชัดเจน** และ **ใช้ช่วงรอไปบรรยายได้**
> ถ้าปล่อยให้ `install.sh` ดึงเอง ผู้เรียนจะเห็นแค่หน้าจอค้างนาน 45 นาทีรวดโดยไม่รู้ว่าเกิดอะไรขึ้น

**✅ เกณฑ์ผ่านระยะที่ 3**

| # | ต้องได้ |
| --- | --- |
| 1 | `git describe --tags` ได้เลขเวอร์ชัน ไม่ใช่ branch |
| 2 | อยู่ใน tmux session |
| 3 | `docker images` มี 25–35 รายการ |

---

## 6. ระยะที่ 4 ติดตั้ง Sentry (ขั้นที่ 15–18)

> ⏱️ 40 นาที (ลงมือ 10 + รอ 30)

---

### ขั้นที่ 15 — ทำความเข้าใจไฟล์ `.env` ก่อนติดตั้ง

```bash
grep -vE '^\s*#|^\s*$' .env | head -20
```

| ตัวแปร | ค่าปริยาย | ความหมาย |
| --- | --- | --- |
| `COMPOSE_PROFILES` | `feature-complete` | ⛔ **ห้ามเปลี่ยนเป็น `errors-only`** จะไม่มี Performance/Tracing |
| `SENTRY_EVENT_RETENTION_DAYS` | `90` | เก็บ event กี่วัน |
| `SENTRY_BIND` | `9000` | พอร์ตที่เปิดให้เข้าใช้งาน |
| `SENTRY_MAIL_HOST` | (ปิดอยู่) | โดเมนสำหรับส่งอีเมล |
| `SENTRY_TASKWORKER_CONCURRENCY` | `4` | จำนวน process ประมวลผลงานเบื้องหลัง |

**ปรับ retention ให้เหมาะกับห้อง Lab**

```bash
sed -i 's/^SENTRY_EVENT_RETENTION_DAYS=.*/SENTRY_EVENT_RETENTION_DAYS=30/' .env && grep RETENTION .env
```

> 🏦 **ที่ BCEL จริง** ค่านี้ต้องกำหนดตามนโยบายเก็บรักษาข้อมูลของธนาคาร
> จดคำถามนี้ไว้ถามฝ่าย Compliance: *"event ที่มีข้อมูลระบบงานเก็บได้กี่วัน"*

---

### ขั้นที่ 16 — ⭐ รัน `install.sh`

**ดูตัวเลือกที่มีก่อนเสมอ**

```bash
./install.sh --help
```

| Flag | ใช้เมื่อไร |
| --- | --- |
| `--skip-user-creation` | ติดตั้งแบบไม่ถามอะไร (สำหรับ automation) |
| `--no-report-self-hosted-issues` | **ไม่ส่ง** error/telemetry ของระบบเรากลับไปให้ Sentry |
| `--skip-commit-check` | ข้ามการตรวจ commit ล่าสุด (ใช้ตอนอยู่บน master) |
| `--minimize-downtime` | สำหรับ upgrade ระบบที่รันอยู่ ไม่ใช่ติดตั้งใหม่ |
| `--skip-sse42-requirements` | ข้ามการตรวจ SSE 4.2 ⚠️ ไม่แนะนำ |
| `--container-engine-podman` | ใช้ podman แทน docker |

**ตรวจว่ายังอยู่ใน tmux**

```bash
echo $TMUX
```

> ✅ ต้องได้ค่าที่ไม่ว่าง — ถ้าว่างให้ `tmux attach -t sentry` ก่อน

**คำสั่งที่หลักสูตรนี้ใช้**

```bash
./install.sh --no-report-self-hosted-issues
```

> 🏦 **ทำไมต้องใส่ `--no-report-self-hosted-issues`**
> ค่าปริยายจะส่ง error และ performance data ของ Sentry เครื่องเรากลับไปยัง sentry.io
> **สำหรับระบบภายในธนาคารต้องปิดเสมอ** เพราะข้อมูลไม่ควรออกนอกองค์กร
> นี่เป็นตัวอย่างที่ดีของ *"ค่าปริยายที่ปลอดภัยสำหรับคนทั่วไป แต่ไม่ปลอดภัยสำหรับธนาคาร"*

**สังเกตแต่ละ group ที่ผ่านไป**

```
Parsing command line ...
Detected platform ...
Detecting Docker Compose version ...
Checking minimum requirements ...          ← ตรวจ RAM / CPU / SSE 4.2 ตรงนี้
Creating volumes for persistent data ...
Ensuring files from examples ...           ← สร้าง config.yml, sentry.conf.py จาก .example
Generating secret key ...
Fetching and updating docker images ...    ← เร็วมาก เพราะดึงไว้แล้วในขั้นที่ 14
Building and tagging Docker images ...
Bootstrapping Snuba ...
Setting up / migrating database ...        ← ⏱️ ขั้นที่นานที่สุด 4–8 นาที
Setting up GeoIP integration ...
```

> ⏱️ **รวมประมาณ 20–30 นาที** ใช้ช่วงนี้บรรยาย Module 1.3
> ถ้าค้างที่ `Setting up / migrating database` เกิน 15 นาที ให้เปิด terminal อีกหน้าต่าง
> (`Ctrl+b` แล้ว `c` เพื่อเปิด window ใหม่ใน tmux) แล้วดู `docker stats` และ `free -h`

---

### ขั้นที่ 17 — สร้างบัญชีผู้ดูแลระบบ

`install.sh` จะถามตอนท้าย

```
Would you like to create a user account now? [Y/n]: y
Email: admin@bcel.com.la
Password: ********
Repeat for confirmation: ********
Should this user be a superuser? [y/N]: y
```

**ถ้าเผลอข้ามไป** สร้างทีหลังได้ด้วย

```bash
docker compose run --rm web createuser
```

> 🔑 **จดอีเมลและรหัสผ่านไว้ให้ดี** ห้อง Lab ไม่มีระบบส่งอีเมลจริง
> ถ้าลืมรหัสผ่านจะกู้ผ่านหน้าเว็บไม่ได้ ต้องสร้าง user ใหม่จาก command line

---

### ขั้นที่ 18 — เปิดระบบและตรวจสุขภาพ

เมื่อ `install.sh` จบจะขึ้นข้อความนี้

```
-----------------------------------------------------------------

You're all done! Run the following command to get Sentry running:

  docker compose up --wait

-----------------------------------------------------------------
```

```bash
docker compose up --wait
```

> ⏱️ ครั้งแรกใช้เวลา 3–6 นาที เพราะรอทุก service ผ่าน healthcheck

**ตรวจสถานะ**

```bash
docker compose ps --format "table {{.Service}}\t{{.Status}}" | head -30
```

```bash
docker compose ps | grep -c "healthy\|running"
```

**ตรวจ health endpoint**

```bash
curl -s http://localhost:9000/_health/
```

> ✅ ต้องได้คำว่า `ok`

**ดูการใช้ทรัพยากร**

```bash
free -h
```

```bash
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | sort -k2 -h -r | head -10
```

> 💬 **ให้ผู้เรียนสังเกต** RAM ถูกใช้ไปเท่าไรจาก 16 GB และตัวไหนกินมากที่สุด
> (ปกติคือ ClickHouse · Kafka · PostgreSQL) นี่คือข้อมูลที่ต้องใช้ตอนวางแผน capacity จริง

**✅ เกณฑ์ผ่านระยะที่ 4**

| # | ต้องได้ |
| --- | --- |
| 1 | เห็นข้อความ `You're all done!` |
| 2 | `docker compose ps` ไม่มี `exited` หรือ `unhealthy` |
| 3 | `curl localhost:9000/_health/` ได้ `ok` |
| 4 | สร้างบัญชี superuser แล้ว |

---

## 7. ระยะที่ 5 ตั้งค่าหลังติดตั้ง (ขั้นที่ 19–22)

> ⏱️ 25 นาที

---

### ขั้นที่ 19 — ⭐ ตั้ง Root URL ให้ถูก (สำคัญที่สุดของทั้งเอกสาร)

```bash
grep -n "system.url-prefix" sentry/config.yml
```

> สังเกตว่าค่าปริยายคือ `http://localhost:9000`

```bash
sed -i "s|^system.url-prefix:.*|system.url-prefix: 'http://$(curl -s ifconfig.me):9000'|" sentry/config.yml && grep -n "system.url-prefix" sentry/config.yml
```

> 🔑 **ถ้าไม่ตั้งค่านี้จะเกิดอะไร**
> ลิงก์ในอีเมลแจ้งเตือน · ลิงก์ที่ Sentry สร้างให้ · และ **DSN ที่แสดงในหน้า Client Keys**
> จะชี้ไปที่ `localhost` ซึ่งเครื่องผู้เรียนเข้าไม่ถึง
> **นี่เป็นปัญหาอันดับหนึ่งของคนติดตั้ง Sentry self-hosted ครั้งแรก**

---

### ขั้นที่ 20 — ปิดการสมัครสมาชิกจากภายนอก

```bash
grep -n "auth:register" sentry/sentry.conf.py
```

แก้ให้เป็น `False`

```bash
sed -i "s/^SENTRY_FEATURES\['auth:register'\].*/SENTRY_FEATURES['auth:register'] = False/" sentry/sentry.conf.py && grep -n "auth:register" sentry/sentry.conf.py
```

> 🔒 **บังคับสำหรับระบบธนาคาร** ค่าปริยายเปิดให้ใครก็ได้ที่เข้าถึงหน้าเว็บสมัครบัญชีเองได้
> ในองค์กรต้องปิดแล้วให้ผู้ดูแลเชิญเข้าทีมแทน

---

### ขั้นที่ 21 — นำการเปลี่ยนแปลงไปใช้

> ⚠️ **การแก้ไฟล์ config แล้ว restart เฉย ๆ จะไม่มีผล ต้องรัน `install.sh` ใหม่**

```bash
./install.sh --no-report-self-hosted-issues --skip-user-creation
```

> ⏱️ รอบที่สองเร็วกว่ามาก (3–5 นาที) เพราะฐานข้อมูล migrate ไปแล้ว

```bash
docker compose up --wait
```

```bash
curl -s http://localhost:9000/_health/
```

---

### ขั้นที่ 22 — เข้าใช้งานและสร้าง Organization กับ Project

**เปิดเบราว์เซอร์บนโน้ตบุ๊กของท่าน**

```
http://<IP ของท่าน>:9000
```

Login ด้วยอีเมลและรหัสผ่านที่สร้างในขั้นที่ 17

**หน้าจอตั้งค่าครั้งแรก**

| ช่อง | ค่าที่ใส่ |
| --- | --- |
| Root URL | `http://<IP ของท่าน>:9000` |
| Admin Email | อีเมลที่สร้างไว้ |
| Outbound email | ปล่อยว่างไว้ (ห้อง Lab ไม่ส่งอีเมลจริง) |
| Anonymous usage statistics | **ไม่ติ๊ก** (นโยบายธนาคาร) |

**สร้าง Organization**

**Settings → Organizations → Create New Organization**

| ช่อง | ค่า |
| --- | --- |
| Organization Name | `BCEL` |
| Slug | `bcel` |

**สร้าง 2 Project**

| # | Platform | Project slug | Alert frequency |
| --- | --- | --- | --- |
| 1 | **SPRING BOOT** | `bcel-crm-backend` | *I'll create my own alerts later* |
| 2 | **ANGULAR** | `bcel-crm-frontend` | *I'll create my own alerts later* |

**หา DSN ของทั้งสอง**

**Settings → Projects → `<ชื่อโปรเจกต์>` → Client Keys (DSN)**

```
BACKEND  DSN = ______________________________________________
FRONTEND DSN = ______________________________________________
```

**รูปแบบ DSN ต่างจาก SaaS อย่างชัดเจน**

```
SaaS         https://<key>@o4510877819207680.ingest.us.sentry.io/4511813448171520
Self-hosted  http://<key>@<IP ของท่าน>:9000/<project-id>
```

| จุดที่ต่าง | SaaS | Self-hosted |
| --- | --- | --- |
| Protocol | `https` | `http` (ยังไม่มี TLS ในห้อง Lab) |
| Host | `o<org-id>.ingest.us.sentry.io` | **IP เครื่องของท่าน** |
| Port | 443 (ไม่ระบุ) | **9000** |
| Project ID | ตัวเลขยาว 16 หลัก | ตัวเลขสั้น เริ่มจาก 1, 2, 3... |

> ⚠️ **ถ้า DSN ขึ้นเป็น `http://<key>@localhost:9000/1`**
> แปลว่ายังไม่ได้ตั้ง `system.url-prefix` — กลับไปทำขั้นที่ 19

**✅ เกณฑ์ผ่านระยะที่ 5**

| # | ต้องได้ |
| --- | --- |
| 1 | เข้าหน้าเว็บจากโน้ตบุ๊กได้ |
| 2 | **DSN แสดง IP จริง ไม่ใช่ `localhost`** |
| 3 | ปิด public registration แล้ว |
| 4 | มี 2 project และจด DSN ครบ |

---

## 8. ระยะที่ 6 ย้าย Spring Boot มาใช้ Sentry ของตัวเอง (ขั้นที่ 23–26)

> ⏱️ **10 นาที** · ทำบน **โน้ตบุ๊กของผู้เรียน** ไม่ใช่บน droplet

> ✅ **ข่าวดี** ผู้เรียนฝัง Sentry SDK ไว้ตั้งแต่ Lab วันที่ 1 แล้ว
> ขั้นตอนนี้จึงเหลือแค่ **เปลี่ยนปลายทางจาก SaaS มาเป็น Sentry ของตัวเอง**
> ซึ่งเป็นการพิสูจน์ประเด็นสำคัญว่า **โค้ดแอปพลิเคชันไม่ต้องแก้อะไรเลยแม้แต่บรรทัดเดียว**

---

### ขั้นที่ 23 — ตรวจว่าโปรเจกต์พร้อม

```bash
cd bcel-crm-lite
```

```bash
docker compose up -d mariadb
```

```bash
docker compose exec mariadb mariadb -u crm_app -plabpass123 bcel_crm -e "SELECT COUNT(*) AS customers FROM customer;"
```

> ✅ ต้องได้ `customers = 5000`

---

### ขั้นที่ 24 — ตรวจว่า SDK ฝังไว้แล้วจาก Lab วันที่ 1

**ถ้าทำ Lab วันที่ 1 มาแล้ว ให้แค่ตรวจว่ามีครบ 2 อย่างนี้**

```bash
grep -c "sentry-spring-boot-starter-jakarta" backend/pom.xml
```

> ✅ ต้องได้ `1` (ไม่ใช่ `0`) และต้องไม่ถูกคอมเมนต์ไว้

```bash
grep -c "^sentry.dsn" backend/src/main/resources/application.properties
```

> ✅ ต้องได้ `1`

<details>
<summary>📦 <b>ถ้ายังไม่ได้ฝัง SDK (เช่น เอาไปทำที่ BCEL ในอนาคต) กดดูขั้นตอนเต็ม</b></summary>

เปิด `backend/pom.xml` เพิ่ม

```xml
<dependency>
  <groupId>io.sentry</groupId>
  <artifactId>sentry-spring-boot-starter-jakarta</artifactId>
</dependency>
<dependency>
  <groupId>io.sentry</groupId>
  <artifactId>sentry-logback</artifactId>
</dependency>
```

เปิด `backend/src/main/resources/application.properties` เพิ่ม

```properties
sentry.dsn=${SENTRY_DSN:}
sentry.environment=${SENTRY_ENVIRONMENT:development}
sentry.release=${SENTRY_RELEASE:bcel-crm-backend@1.0.0}
sentry.send-default-pii=false
sentry.in-app-includes=la.com.bcel.crm
sentry.exception-resolver-order=-2147483647
sentry.ignored-exceptions-for-type=\
  la.com.bcel.crm.common.BusinessValidationException,\
  org.springframework.web.servlet.NoHandlerFoundException
sentry.debug=false
sentry.logging.minimum-breadcrumb-level=info
sentry.logging.minimum-event-level=error
```

📖 รายละเอียดของแต่ละ property และคำอธิบายเลข `-2147483647`
อยู่ใน `Labs/Day1_Lab.md` หัวข้อ Lab 1.4 ขั้นที่ 2

</details>

> 🎯 **ประเด็นสำคัญที่ต้องย้ำในห้อง**
> `sentry.dsn=${SENTRY_DSN:}` อ่านค่าจาก environment variable
> ทำให้เราสลับระหว่าง SaaS กับ Self-hosted ได้โดย **ไม่ต้องแก้โค้ดหรือ build ใหม่**
> นี่คือเหตุผลที่ Lab วันที่ 1 บังคับให้อ่าน DSN จาก env ไม่ให้ hardcode

---

### ขั้นที่ 25 — รัน Backend ด้วย DSN ของ Self-hosted

```bash
cd backend && mvn -B clean package -DskipTests
```

**รันโดยชี้ไปที่ Sentry ของตัวเอง**

```bash
SENTRY_DSN='http://<key>@<IP ของท่าน>:9000/1' mvn spring-boot:run
```

บน PowerShell

```powershell
$env:SENTRY_DSN='http://<key>@<IP ของท่าน>:9000/1'; mvn spring-boot:run
```

**เทียบ DSN สองตัวให้เห็นชัด ๆ**

```
วันที่ 1–2 (SaaS)
  https://47aa76b9...@o4510877819207680.ingest.us.sentry.io/4511813448171520

บ่ายวันที่ 3 (Self-hosted ของท่านเอง)
  http://<key>@<IP ของท่าน>:9000/1
   ↑                ↑          ↑    ↑
   http ไม่ใช่ https  IP เครื่อง  พอร์ต  project id สั้น
```

> 🎯 **ประเด็นที่ต้องย้ำ** เราเปลี่ยนแค่ **environment variable ตัวเดียว**
> ไม่ได้แก้โค้ด ไม่ได้ build ใหม่ ไม่ได้แตะ `pom.xml`
> ข้อมูลทั้งหมดเปลี่ยนเส้นทางจากเซิร์ฟเวอร์ที่สหรัฐฯ มาที่เครื่องของท่านทันที
> **นี่คือคุณค่าของการออกแบบให้อ่าน config จาก environment**

**ตรวจว่าแอปขึ้น**

```bash
curl -s http://localhost:8080/actuator/health
```

---

### ขั้นที่ 26 — ⭐ ทดสอบว่า event เดินทางถึง Sentry ของตัวเองจริง

**เปิด debug ของ SDK เพื่อดูเส้นทางของข้อมูล**

```bash
SENTRY_DSN='http://<key>@<IP ของท่าน>:9000/1' mvn spring-boot:run -Dspring-boot.run.arguments=--sentry.debug=true
```

**ยิง error ทดสอบ**

```bash
curl -s http://localhost:8080/api/_debug/sentry/boom
```

**ดูที่ log ของ backend ต้องเห็นบรรทัดเหล่านี้**

```
INFO: Initializing SDK with DSN: 'http://<key>@<IP>:9000/1'
DEBUG: Envelope sent successfully.
DEBUG: Envelope flushed
```

> 🔑 **`Envelope sent successfully` คือหลักฐานชี้ขาด** ว่าข้อมูลออกจากเครื่องผู้เรียน
> ไปถึง Sentry ของตัวเองบน DigitalOcean เรียบร้อยแล้ว
> ถ้าไม่เห็นบรรทัดนี้ ให้ไปดูหัวข้อ 12.3

**ตรวจใน Sentry**

เปิด `http://<IP ของท่าน>:9000` → **Issues** → เลือก project `bcel-crm-backend`

> ⚠️ ถ้าหน้าจอว่าง ให้ **ล้างช่องค้นหาให้ว่าง** และเปลี่ยนช่วงเวลาเป็น **24H**
> เพราะมุมมองปริยาย Feed/Recommended กรอง issue ออกได้

**ผลที่ต้องเห็น**

```
IllegalStateException
BCEL CRM: ทดสอบ unhandled exception ตัวแรก
Unhandled · GET /api/_debug/sentry/boom
```

**✅ เกณฑ์ผ่านระยะที่ 6**

| # | ต้องได้ |
| --- | --- |
| 1 | log มี `Initializing SDK with DSN` ที่ชี้ไป IP ของตัวเอง |
| 2 | log มี `Envelope sent successfully` |
| 3 | เห็น Issue ใน Sentry ของตัวเอง |

---

## 9. ระยะที่ 7 ย้าย Angular มาใช้ Sentry ของตัวเอง (ขั้นที่ 27–30)

> ⏱️ **10 นาที** (ถ้าทำ Lab วันที่ 2 มาแล้ว)

> ✅ เช่นเดียวกับฝั่ง backend — SDK ฝังไว้แล้วตั้งแต่ Lab วันที่ 2
> เหลือแค่เปลี่ยน DSN ในไฟล์ `environment.ts`

---

### ขั้นที่ 27 — ตรวจว่า SDK ติดตั้งไว้แล้ว

```bash
cd frontend && node -e "console.log(require('./node_modules/@sentry/angular/package.json').version)"
```

> ✅ ต้องได้เลขเวอร์ชัน เช่น **10.68.0**
> ถ้าขึ้น error ว่าไม่พบโมดูล ให้ติดตั้งก่อนด้วย

```bash
npm install --save @sentry/angular
```

---

### ขั้นที่ 28 — ⭐ เปลี่ยน DSN เป็นของ Self-hosted

เปิด `frontend/src/environments/environment.ts` แล้วเปลี่ยนบรรทัด `sentryDsn`

```typescript
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:8080/api',

  // เดิม (SaaS)      : 'https://a6797c26...@o4510877819207680.ingest.us.sentry.io/4511813450661888'
  // ใหม่ (Self-hosted) ⬇  ⚠️ DSN ของ bcel-crm-frontend คนละตัวกับ backend
  sentryDsn: 'http://<key ของ frontend>@<IP ของท่าน>:9000/2',

  sentryEnvironment: 'development',
  sentryRelease: 'bcel-crm-frontend@1.0.0'
}
```

> ⚠️ **สังเกต project id เป็น `2` ไม่ใช่ `1`** เพราะเป็นโปรเจกต์ที่สองที่สร้าง
> ถ้าใช้ DSN ของ backend ข้อมูลจะไปรวมกันมั่ว และ Session Replay จะไม่ทำงาน

> 💬 **ประเด็นชวนคิด** ทำไมฝั่ง Angular ต้องแก้ไฟล์ แต่ฝั่ง Spring Boot แค่เปลี่ยน env var
> (คำตอบ: โค้ด frontend ถูก compile ลง bundle ที่ส่งไปรันบนเบราว์เซอร์
> จึงไม่มี environment variable ให้อ่านตอน runtime — ต้องฝังค่าตอน build
> นี่คือเหตุผลที่ Jenkins ต้องใช้ `sed` แทนที่ `__SENTRY_DSN__` ใน `environment.prod.ts`)

---

### ขั้นที่ 29 — ตรวจว่า `Sentry.init()` และ provider ครบ

<details>
<summary>📦 <b>ถ้ายังไม่ได้ทำ Lab วันที่ 2 กดดูขั้นตอนเต็ม</b></summary>

`frontend/src/main.ts` ปลดคอมเมนต์บล็อก `📌 วันที่ 2 Module 2.5.2`

```typescript
import * as Sentry from '@sentry/angular'
import { environment } from './environments/environment'

Sentry.init({
  dsn: environment.sentryDsn,
  environment: environment.sentryEnvironment,
  release: environment.sentryRelease,
  sendDefaultPii: false,

  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration({
      maskAllText: true,
      maskAllInputs: true,
      blockAllMedia: true
    })
  ],

  tracesSampleRate: environment.production ? 0.1 : 1.0,
  tracePropagationTargets: ['localhost', /^\/api/],

  replaysSessionSampleRate: 0.0,
  replaysOnErrorSampleRate: 1.0,

  beforeSend(event) {
    if (event.request?.url) event.request.url = event.request.url.split('?')[0]
    if (event.request?.cookies) delete event.request.cookies
    return event
  },

  ignoreErrors: [
    'ResizeObserver loop limit exceeded',
    'Non-Error promise rejection captured',
    /extension\//,
    /^chrome-extension:\/\//
  ]
})

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err))
```

`frontend/src/app/app.config.ts` ปลดคอมเมนต์ทั้ง 3 บล็อก

```typescript
import { ErrorHandler, inject, provideAppInitializer } from '@angular/core'
import { Router } from '@angular/router'
import * as Sentry from '@sentry/angular'

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withInterceptors([apiInterceptor]))

    ,{ provide: ErrorHandler, useValue: Sentry.createErrorHandler({ showDialog: false }) }
    ,{ provide: Sentry.TraceService, deps: [Router] }
    ,provideAppInitializer(() => { inject(Sentry.TraceService) })
  ]
}
```

⭐ **provider ตัวที่ 3 คือจุดที่คนลืมบ่อยที่สุด** ถ้าไม่มี `provideAppInitializer`
ที่ inject `TraceService` มันจะไม่ถูกสร้างเลย ผลคือ error จับได้ปกติ
**แต่ transaction ของ navigation ไม่เกิดขึ้นสักอัน**

</details>

**ถ้าทำ Lab วันที่ 2 มาแล้ว ให้แค่ตรวจว่ามีครบ**

```bash
grep -c "Sentry.init" src/main.ts
```

```bash
grep -c "provideAppInitializer" src/app/app.config.ts
```

> ✅ ทั้งสองคำสั่งต้องได้ `1` และต้องไม่ถูกคอมเมนต์ไว้

---

### ขั้นที่ 30 — รันและทดสอบฝั่ง Frontend

```bash
npm start
```

> ถ้า `npm start` พังด้วย `'Error' is not recognized...` แปลว่าพาธมี `&` หรือช่องว่าง
> ให้ใช้ `node node_modules/@angular/cli/bin/ng.js serve` แทน (ดู `Labs/Lab_00_Setup.md` ข้อ 0.2)

เปิด `http://localhost:4200` แล้วกดปุ่มทดสอบในหน้า **ลูกค้า**

| ปุ่ม | ทดสอบอะไร |
| --- | --- |
| **ทดสอบ Error** | error แบบ synchronous ผ่าน Angular ErrorHandler |
| **ทดสอบ Error แบบ async** | error ใน `setTimeout` (นอก Zone) |
| **เรียก API ที่พัง (ลูกค้า 4999)** | HTTP 500 จาก backend |

**พิสูจน์ว่า trace header ถูกแนบ**

เปิด DevTools → Network → คลิก request ไปที่ `/api/customers` → แท็บ **Headers**

```
sentry-trace: <trace-id>-<span-id>-1
baggage: sentry-environment=development,sentry-release=bcel-crm-frontend@1.0.0,...
```

**ตรวจใน Sentry**

`http://<IP ของท่าน>:9000` → **Issues** → project `bcel-crm-frontend`

```
Error: BCEL CRM Frontend: ทดสอบ error ตัวแรก
Error: BCEL CRM Frontend: ทดสอบ error แบบ async
HTTP error 500 http://localhost:8080/api/customers/4999/statement
```

**✅ เกณฑ์ผ่านระยะที่ 7**

| # | ต้องได้ |
| --- | --- |
| 1 | Issues ของ `bcel-crm-frontend` มี 3 รายการ |
| 2 | DevTools เห็น header `sentry-trace` และ `baggage` |

---

## 10. ระยะที่ 8 พิสูจน์ End-to-End (ขั้นที่ 31–32)

> ⏱️ 20 นาที · นี่คือบทพิสูจน์ว่าระบบที่ติดตั้งเองใช้งานได้เต็มรูปแบบจริง

---

### ขั้นที่ 31 — เปิด Performance Monitoring และ Database Tracing

**บน backend** — `backend/pom.xml` ปลดคอมเมนต์

```xml
<dependency>
  <groupId>io.sentry</groupId>
  <artifactId>sentry-jdbc</artifactId>
</dependency>
```

`application.properties` ปลดคอมเมนต์บล็อก `📌 วันที่ 2 Module 2.4.1`

```properties
sentry.traces-sample-rate=${SENTRY_TRACES_RATE:1.0}
sentry.trace-propagation-targets=localhost,^https://.*\\.bcel\\.local/.*$
sentry.enable-auto-session-tracking=true
```

คอมเมนต์บรรทัดเดิม แล้วเปลี่ยนไปใช้ P6Spy

```properties
# spring.datasource.url=jdbc:mariadb://localhost:3306/bcel_crm
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
spring.datasource.url=jdbc:p6spy:mariadb://localhost:3306/bcel_crm
```

```bash
cd backend && mvn -B clean package -DskipTests
```

```bash
SENTRY_DSN='http://<key>@<IP ของท่าน>:9000/1' mvn spring-boot:run
```

**ตรวจว่า P6Spy ทำงาน** — log ตอน start ต้องเห็น

```
HikariPool-1 - Added connection com.p6spy.engine.wrapper.ConnectionWrapper@...
```

---

### ขั้นที่ 32 — ⭐⭐ สร้างและอ่าน Trace แบบ 3 ชั้น

**ที่ `http://localhost:4200` ทำตามลำดับนี้ในเซสชันเดียว**

1. เปิดหน้า **ลูกค้า** (เกิด pageload transaction)
2. กด **เรียก API ที่พัง (ลูกค้า 4999)**
3. กด **เรียก API ที่ช้า (N+1)**
4. ไปหน้า **รายงาน** แล้วกด **เรียกเวอร์ชันช้า**

**ดูผลใน Sentry ของตัวเอง**

`http://<IP ของท่าน>:9000` → **Explore → Traces** → เลือก **ทั้งสองโปรเจกต์** → แท็บ **Trace Samples**

หา trace ที่ **Trace Root เป็นไอคอน Angular** และมี span จำนวนมาก แล้วคลิกเข้าไป

**ผลที่ต้องเห็น**

```
Trace — <trace-id>                              Spans 120+ · Root Duration ~600ms
│
├── 🅰️ pageload — /customers/
│     └── 🟢 http.server — GET /api/customers
│           └── 🔵 db.query — select ... from customer ...
│
├── 🟢 http.server — GET /api/customers/{id}/statement       🔴 error
├── 🟢 http.server — GET /api/customers/{id}/summary
│     ├── 🔵 db.query — select ... from account ...
│     └── 🔵 db.query — select ... from transaction_log ...  (ซ้ำ 15 ครั้ง = N+1)
│
└── 🅰️ Error — BCEL CRM Frontend: ทดสอบ error ตัวแรก
```

> 🎯 **นี่คือบทพิสูจน์สุดท้าย** ผู้เรียนติดตั้งระบบ observability ทั้งระบบด้วยตัวเอง
> ตั้งแต่ OS เปล่า ๆ จนถึงเห็น trace เดียวที่ไล่จากคลิกของผู้ใช้บนหน้าจอ Angular
> ผ่าน HTTP ไปยัง Spring Boot แล้วลงไปถึง SQL ที่ยิงเข้า MariaDB
> **โดยข้อมูลทั้งหมดไม่เคยออกนอกเซิร์ฟเวอร์ของตัวเองเลย**

**คำถามปิดท้าย**

1. ข้อมูล trace นี้เดินทางผ่านเส้นทางใดบ้าง ตั้งแต่เบราว์เซอร์จนถึง ClickHouse
2. ถ้า Kafka บนเครื่องท่านล่ม จะเกิดอะไรขึ้นกับ event ที่ส่งมาระหว่างนั้น
3. เทียบกับ SaaS แล้ว ท่านได้อะไรเพิ่ม และเสียอะไรไป

---

## 11. เกณฑ์ตรวจผ่านทั้งหมด

| ระยะ | # | สิ่งที่ต้องพิสูจน์ได้ | วิธีตรวจ |
| --- | --- | --- | --- |
| **1** | 1 | เครื่องผ่านเกณฑ์ขั้นต่ำ | `nproc` ≥ 4 · RAM ≥ 14 GB · SSE4.2 > 0 |
| **1** | 2 | **สร้าง swap 16 GB เอง** | `free -h` แสดง `Swap: 16Gi` |
| **1** | 3 | มีผู้ใช้ที่ไม่ใช่ root | `id sentry-admin` เห็นกลุ่ม `sudo` |
| **2** | 4 | ติดตั้ง Docker ผ่าน APT repo เอง | `docker version` ≥ 19.03.6 |
| **2** | 5 | Docker Compose ผ่านเกณฑ์ | `docker compose version` ≥ 2.32.2 |
| **2** | 6 | รัน docker ได้โดยไม่ใช้ sudo | `docker run --rm hello-world` |
| **3** | 7 | ใช้ tag ไม่ใช่ master | `git describe --tags` ได้เลขเวอร์ชัน |
| **3** | 8 | ใช้ tmux ป้องกัน SSH หลุด | `echo $TMUX` ไม่ว่าง |
| **3** | 9 | ดึง image ครบ | `docker images \| wc -l` ได้ 25–35 |
| **4** | 10 | `install.sh` จบไม่มี error | เห็น `You're all done!` |
| **4** | 11 | ทุก container สุขภาพดี | `docker compose ps` ไม่มี `exited`/`unhealthy` |
| **4** | 12 | Health check ผ่าน | `curl localhost:9000/_health/` ได้ `ok` |
| **5** | 13 | **ตั้ง `system.url-prefix` ถูก** | DSN แสดง IP จริง ไม่ใช่ `localhost` |
| **5** | 14 | ปิด public registration | `auth:register` = `False` |
| **5** | 15 | สร้าง 2 project ได้ DSN ครบ | จดไว้ทั้ง backend และ frontend |
| **6** | 16 | **Spring Boot ส่ง event ถึง** | log มี `Envelope sent successfully` |
| **6** | 17 | เห็น Issue ฝั่ง backend | Issues ของ `bcel-crm-backend` |
| **7** | 18 | **Angular ส่ง event ถึง** | Issues ของ `bcel-crm-frontend` มี 3 รายการ |
| **7** | 19 | trace header ถูกแนบ | DevTools เห็น `sentry-trace` + `baggage` |
| **8** | 20 | P6Spy ทำงาน | log มี `p6spy.engine.wrapper.ConnectionWrapper` |
| **8** | 21 | **Trace เดียวเห็นครบ 3 ชั้น** | pageload → http.server → db.query |
| **8** | 22 | อธิบายสิ่งที่ติดตั้งได้ | ตอบได้ว่า 80+ container ทำอะไรบ้าง |

---

## 12. ปัญหาที่พบบ่อยและวิธีแก้

### 12.1 ระยะเตรียมระบบและ Docker

| อาการ | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| `fallocate: fallocate failed: Text file busy` | มี swapfile อยู่แล้ว | `swapoff /swapfile && rm /swapfile` แล้วทำใหม่ |
| `apt update` ขึ้น GPG error หลังเพิ่ม docker repo | key ผิดสิทธิ์ | `chmod a+r /etc/apt/keyrings/docker.asc` |
| `docker: permission denied` | ยังไม่อยู่ในกลุ่ม docker หรือยังไม่ได้ login ใหม่ | `exit` แล้ว `ssh` เข้าใหม่ |
| `docker compose` ไม่มีคำสั่ง | ลืมติดตั้ง `docker-compose-plugin` | `apt install -y docker-compose-plugin` |
| Docker Compose < 2.32.2 | package เก่า | `apt update && apt upgrade docker-compose-plugin` |

### 12.2 ระยะติดตั้ง Sentry

| ข้อความที่เห็น | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| `FAIL: Required minimum RAM available to Docker is 14000 MB` | droplet เล็กเกินไป | ต้องใช้ 16 GB ขึ้นไป |
| `FAIL: Required minimum CPU cores available to Docker is 4` | vCPU น้อยเกินไป | ใช้แผน ≥ 4 vCPU |
| `FAIL: Expected minimum docker version to be 19.03.6` | Docker เก่า | ทำขั้นที่ 8 ใหม่ |
| `FAIL: ... does not support the SSE 4.2 instruction set` | CPU host ไม่รองรับ | เปลี่ยน droplet type |
| ค้างที่ `Setting up / migrating database` เกิน 15 นาที | RAM ไม่พอ กำลัง swap หนัก | เปิด tmux window ใหม่ ดู `free -h` · ถ้าไม่มี swap ให้ทำขั้นที่ 5 |
| `install.sh` ตายกลางคัน | ปกติกู้ได้ | **รัน `./install.sh` ซ้ำได้เลย** สคริปต์ออกแบบมาให้รันซ้ำได้ |
| SSH หลุดแล้วงานหาย | ไม่ได้ใช้ tmux | `tmux attach -t sentry` · ถ้าไม่มีจริงต้องเริ่มใหม่ |
| ดึง image ช้ามาก | 5 เครื่องดึงพร้อมกัน | เริ่มให้เร็วที่สุด · ถ้าไม่ทันให้ต่อตอนพักเที่ยง |

### 12.3 ระยะเชื่อมแอปพลิเคชัน

| อาการ | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| เปิด `http://<IP>:9000` ไม่ขึ้น | Cloud Firewall ยังไม่เปิดพอร์ต | ตรวจข้อ 2.4 · ลอง `curl localhost:9000/_health/` บน droplet ก่อน |
| DSN ขึ้นเป็น `localhost` | ไม่ได้ตั้ง `system.url-prefix` | ทำขั้นที่ 19 แล้วรัน `install.sh` ใหม่ |
| ไม่เห็น `Envelope sent successfully` | DSN ผิด หรือ firewall บล็อก | ตรวจ DSN · ทดสอบ `curl http://<IP>:9000/_health/` จากโน้ตบุ๊ก |
| log ขึ้น `Connection refused` | Sentry ไม่ได้รัน | บน droplet: `docker compose ps` |
| Issue ไม่ขึ้นทั้งที่ส่งสำเร็จ | มุมมอง Feed/Recommended กรองอยู่ | ล้างช่องค้นหา + เปลี่ยนเป็น 24H |
| Angular ไม่มี transaction ของ navigation | ลืม `provideAppInitializer` | ทำขั้นที่ 29 ให้ครบทั้ง 3 provider |
| Trace ขาดเป็น 2 ท่อน | `tracePropagationTargets` ไม่ครอบคลุม | ต้องมี `'localhost'` ในลิสต์ |
| ไม่เห็น span ของ DB | เปลี่ยนแค่ URL ไม่ได้เปลี่ยน driver | ต้องแก้ทั้ง `driver-class-name` และ prefix `jdbc:p6spy:` |
| `sentry-cli` ขึ้น auth error | ลืมตั้ง `SENTRY_URL` | `export SENTRY_URL='http://<IP>:9000'` |

### 12.4 คำสั่งวินิจฉัยที่ควรจำ

```bash
docker compose ps
```

```bash
docker compose logs web --tail 100
```

```bash
docker compose logs --tail 50 | grep -i error
```

```bash
docker stats --no-stream
```

```bash
df -h && free -h
```

```bash
docker compose restart web
```

---

## 13. การดูแลระยะยาว (สำหรับ BCEL ของจริง)

> ส่วนนี้ไม่ได้ทำในห้อง Lab แต่**ต้องอภิปราย** เพราะเป็นสิ่งที่ทีมต้องรับผิดชอบหลังจบหลักสูตร

### 13.1 การอัปเกรด

Sentry ออก release ทุกเดือน และ **ข้ามเวอร์ชันไม่ได้**

```bash
cd ~/self-hosted && git fetch --tags && git tag | tail -10
```

```bash
git checkout <เวอร์ชันถัดไปทีละขั้น> && ./install.sh --minimize-downtime && docker compose up --wait
```

> ⚠️ **ต้องอัปเกรดไล่ทีละเวอร์ชัน** เช่น 26.5 → 26.6 → 26.7
> ข้ามจาก 26.5 ไป 26.7 เลยจะทำให้ database migration พัง
> **ทีมต้องมีแผนอัปเกรดทุกเดือน** ไม่งั้นจะตกรุ่นจนอัปไม่ได้

### 13.2 การสำรองข้อมูล

| ต้อง backup อะไร | คำสั่ง |
| --- | --- |
| PostgreSQL (ข้อมูลหลัก) | `docker compose exec postgres pg_dump -U postgres postgres > backup.sql` |
| ไฟล์ config | `tar czf config.tgz sentry/ .env` |
| Docker volume | `docker run --rm -v sentry-postgres:/data -v $(pwd):/backup alpine tar czf /backup/pg.tgz /data` |
| ClickHouse (event) | ปกติไม่ backup เพราะใหญ่มากและมี retention อยู่แล้ว |

### 13.3 สิ่งที่ห้อง Lab ยังไม่ได้ทำ แต่ production ต้องมี

| # | สิ่งที่ขาด | ทำไมจำเป็น |
| --- | --- | --- |
| 1 | **TLS / HTTPS** | ตอนนี้ DSN เป็น `http://` ข้อมูลวิ่งแบบไม่เข้ารหัส |
| 2 | **Reverse proxy + โดเมนภายใน** | `sentry.bcel.local` แทนการใช้ IP |
| 3 | **SMTP จริง** | แจ้งเตือนทางอีเมลยังใช้ไม่ได้ |
| 4 | **SSO / LDAP** | เชื่อมกับ Active Directory ของธนาคาร |
| 5 | **Backup อัตโนมัติ** | ตอนนี้ไม่มีเลย |
| 6 | **Monitoring ตัว Sentry เอง** | ใครเฝ้าคนเฝ้า — ต้องมี Prometheus ดู disk/RAM |
| 7 | **High Availability** | ตอนนี้เครื่องเดียว ล่มคือจบ |
| 8 | **นโยบาย retention ตาม Compliance** | ต้องคุยกับฝ่ายกำกับ |
| 9 | **จำกัดสิทธิ์ SSH** | ห้อง Lab ใช้รหัสผ่าน · production ต้องใช้ key เท่านั้น |

### 13.4 เรื่อง CentOS 8 ของ BCEL

| ทางเลือก | ข้อดี | ข้อเสีย |
| --- | --- | --- |
| ติดตั้งบน CentOS 8 ตรง ๆ | ตรงกับมาตรฐานเดิมขององค์กร | ⚠️ Sentry ระบุว่ามีปัญหาที่ทราบแล้ว · CentOS 8 หมด EOL |
| **VM Ubuntu LTS แยกเครื่อง** | ✅ ตรงกับที่ Sentry รองรับและทดสอบ | ต้องขออนุมัติ OS ใหม่ |
| Deploy บน RKE2 ที่มีอยู่ | ใช้ของเดิม · ทำ HA ได้ | ⚠️ Sentry ไม่มี Helm chart ทางการ ต้องดูแลเอง |

> 💬 **ข้อเสนอแนะของหลักสูตร** ใช้ **VM Ubuntu LTS แยกเครื่อง** สำหรับ Sentry
> เพราะเป็นทางที่ Sentry ทดสอบมาแล้ว และแยกความเสี่ยงออกจากระบบงานหลักของธนาคาร

---

## 14. ลบทิ้งและสรุปค่าใช้จ่าย

### 14.1 ⚠️ ลบทันทีเวลา 16:30 น. ของวันที่ 3

> ทำผ่านหน้าเว็บ `digitalocean.com` เหมือนตอนสร้าง

**วิธีที่เร็วที่สุด — ลบทีเดียวด้วย tag**

1. เข้า **Manage → Droplets**
2. ช่องค้นหาด้านบน พิมพ์ **`sentry-lab`**
3. ติ๊ก **checkbox หัวตาราง** เพื่อเลือกทั้ง 5 เครื่อง
4. กดปุ่ม **Actions → Destroy → Destroy Droplets**
5. พิมพ์ยืนยันตามที่ระบบขอ

**ลบ Firewall ที่ไม่ใช้แล้ว**

1. เข้า **Networking → Firewalls**
2. คลิกที่ `sentry-lab-fw` → **Settings → Destroy**

> 💡 **ถ้าสร้าง Project แยกไว้** (ข้อ 2.2 ข้อ 13) ให้เข้าไปที่ Project `BCEL-Sentry-Training`
> แล้วดูว่าไม่มี resource เหลืออยู่ จากนั้นลบ Project ทิ้งได้เลย

### 14.2 ✅ Checklist ปิดงาน (ทำทันทีก่อนออกจากห้อง)

- [ ] **Manage → Droplets** ค้นหา `sentry-lab` แล้ว**ไม่พบอะไรเลย**
- [ ] **Networking → Firewalls** ไม่มี `sentry-lab-fw`
- [ ] **Manage → Snapshots** ไม่มี snapshot ค้าง (ถ้าเผลอสร้างไว้)
- [ ] **Manage → Volumes** ไม่มี volume ค้าง
- [ ] **Billing → Current Usage** ตรวจว่าไม่มี droplet เดินอยู่
- [ ] เก็บบันทึกที่ผู้เรียนจดไว้ (DSN · ค่า config) ไปใช้เทียบตอนกลับไปทำที่ทำงาน

> ⚠️ **DigitalOcean คิดเงิน droplet ที่ปิดเครื่อง (Powered off) ด้วย**
> เพราะ CPU/RAM/disk ยังถูกจองไว้ให้ — **ต้อง Destroy เท่านั้น ไม่ใช่แค่ Power off**

### 14.3 สรุปค่าใช้จ่ายจริง

| รายการ | คำนวณ | ค่าใช้จ่าย |
| --- | --- | --- |
| 5 เครื่อง × 8 ชม. (08:30–16:30 วันที่ 3) | $0.14286 × 8 × 5 | **$5.71** |
| **รวมทั้งหลักสูตร** | | **≈ $6 (≈ 210 บาท)** |

> 💰 **ถูกลงมากเพราะย้ายมาทำบ่ายวันเดียว** เดิมถ้าใช้ตลอด 3 วันจะเป็น $51
> การวาง Lab ไว้บ่ายวันสุดท้ายจึงประหยัดกว่าเกือบ 9 เท่า

**ตารางเทียบให้เห็นภาพ**

| ใช้นานเท่าไร | ค่าใช้จ่าย 5 เครื่อง |
| --- | --- |
| 8 ชั่วโมง (บ่ายวันเดียว) ← **แผนของเรา** | **$5.71** |
| 3 วัน (72 ชม.) | $51.43 |
| 1 สัปดาห์ | $120 |
| **1 เดือน (ลืมลบ)** | ⚠️ **$480 (≈ 17,500 บาท)** |

> ⏰ **ตั้งเตือนในปฏิทินทันทีที่สร้าง droplet เสร็จ** ว่า *"16:30 น. ลบ droplet DigitalOcean"*

---

## ภาคผนวก ก — สรุป 32 ขั้นตอนในหน้าเดียว (แจกผู้เรียนได้)

| ระยะ | ขั้น | ทำอะไร | เวลาสะสม |
| --- | --- | --- | --- |
| **1. เตรียม OS**<br>13:15–13:40 | 1 | SSH เข้าเครื่อง | |
| | 2 | ตรวจสเปก (CPU · RAM · disk · SSE4.2) | |
| | 3 | `apt update && upgrade` + ติดตั้ง git curl tmux | |
| | 4 | สร้างผู้ใช้ `sentry-admin` | |
| | 5 | ⭐ **สร้าง swap 16 GB** (ห้ามข้าม) | |
| | 6 | ตรวจความพร้อมทั้งหมด | 13:40 |
| **2. Docker**<br>13:40–14:00 | 7 | เลือกวิธีติดตั้ง (อภิปรายความปลอดภัย) | |
| | 8 | ติดตั้งผ่าน APT repository | |
| | 9 | ตรวจเวอร์ชัน + เพิ่มเข้ากลุ่ม docker | |
| | 10 | ทดสอบด้วย `hello-world` แบบไม่ใช้ sudo | 14:00 |
| **3. เตรียม Sentry**<br>14:00–14:15 | 11 | `git clone` | |
| | 12 | ⭐ `git checkout <tag>` ห้ามใช้ master | |
| | 13 | ⭐⭐ **เปิด tmux** (กัน SSH หลุด) | |
| | 14 | `docker compose pull` (5–15 นาที · ไม่ผ่านเน็ตสถาบัน) | 14:15 |
| ⏳ **รอ + บรรยาย** | | สถาปัตยกรรมภายในของ Sentry | 14:35 |
| **4. ติดตั้ง**<br>14:35–15:15 | 15 | อ่านและปรับ `.env` | |
| | 16 | ⭐ `./install.sh --no-report-self-hosted-issues` | |
| | 17 | สร้างบัญชี superuser | |
| | 18 | `docker compose up --wait` + ตรวจ health | 15:15 |
| ☕ **พักเบรก** | | | 15:25 |
| **5. ตั้งค่า**<br>15:25–15:45 | 19 | ⭐ **ตั้ง `system.url-prefix`** (ห้ามข้าม) | |
| | 20 | ปิด public registration | |
| | 21 | รัน `install.sh` ใหม่ให้ config มีผล | |
| | 22 | สร้าง Org + 2 Project + จด DSN | 15:45 |
| **6. Spring Boot**<br>15:45–15:55 | 23 | ตรวจว่า MariaDB บนโน้ตบุ๊กพร้อม | |
| | 24 | ตรวจว่า SDK ฝังไว้แล้วจากวันที่ 1 | |
| | 25 | ⭐ **เปลี่ยนแค่ `SENTRY_DSN`** แล้วรันใหม่ | |
| | 26 | ⭐ ยืนยัน `Envelope sent successfully` | 15:55 |
| **7. Angular**<br>15:55–16:05 | 27 | ตรวจว่า `@sentry/angular` ติดตั้งแล้ว | |
| | 28 | ⭐ **เปลี่ยน `sentryDsn` ใน `environment.ts`** | |
| | 29 | ตรวจว่า `Sentry.init()` + 3 provider ครบ | |
| | 30 | ทดสอบ + ตรวจ trace header | 16:05 |
| **8. พิสูจน์**<br>16:05–16:20 | 31 | เปิด tracing + P6Spy | |
| | 32 | ⭐⭐ อ่าน Trace 3 ชั้น End-to-End | 16:20 |
| 🏁 **ปิดหลักสูตร** | | สรุป · ถาม-ตอบ · **วิทยากรลบ droplet** | 16:30 |

> 🎯 **4 ขั้นตอนที่ห้ามข้ามเด็ดขาด**
> ขั้นที่ **5** (swap) · **12** (checkout tag) · **13** (tmux) · **19** (`system.url-prefix`)
> ทั้งสี่ข้อนี้คือสาเหตุของปัญหา 90% ที่คนติดตั้ง Sentry self-hosted เจอ

---

## ภาคผนวก ข — ตารางเทียบ SaaS กับ Self-hosted

| หัวข้อ | SaaS | Self-hosted |
| --- | --- | --- |
| ข้อมูลอยู่ที่ไหน | เซิร์ฟเวอร์ของ Sentry (US/EU) | **เซิร์ฟเวอร์ของธนาคาร** |
| Data Sovereignty | ⚠️ ไม่ผ่านข้อกำหนดธนาคารบางข้อ | ✅ ผ่าน |
| เวลาที่ใช้เริ่มต้น | 5 นาที | **~3 ชั่วโมง** |
| ค่าใช้จ่าย | จ่ายตามปริมาณ event | ค่าเซิร์ฟเวอร์ + **ค่าคนดูแล** |
| ใครดูแล upgrade | Sentry | **ทีมของท่าน ทุกเดือน** |
| ใครดูแล backup | Sentry | **ทีมของท่าน** |
| ระบบล่มตอนตี 2 | Sentry รับผิดชอบ | **ทีมของท่านรับผิดชอบ** |
| DSN | `https://...ingest.us.sentry.io/...` | `http://<host>:9000/<id>` |
| `sentry-cli` | ไม่ต้องตั้ง `SENTRY_URL` | **ต้องตั้ง `SENTRY_URL` ทุกคำสั่ง** |
| ฟีเจอร์ AI (Seer) | ✅ มี | ❌ ไม่มี |
| Session Replay · Tracing · Profiling | ✅ มี | ✅ มี (ต้องใช้ `feature-complete`) |

> 💬 **ประโยคปิดหลักสูตรที่ใช้พูดได้เลย**
> *"วันนี้ท่านติดตั้ง Sentry ทั้งระบบด้วยมือตัวเอง ตั้งแต่ Ubuntu เปล่า ๆ
> จนถึงเห็น trace ที่ไล่จากหน้าจอผู้ใช้ลงไปถึง SQL ในฐานข้อมูล
> Self-hosted ไม่ได้แปลว่าฟรี — มันแปลว่าท่านย้ายต้นทุนจากค่าบริการรายเดือน
> ไปเป็นค่าเซิร์ฟเวอร์บวกเวลาของทีม สำหรับธนาคารที่มีข้อกำหนดเรื่อง Data Sovereignty
> นี่เป็นต้นทุนที่คุ้มค่า แต่ตอนนี้ท่านรู้แล้วว่ากำลังรับอะไรมาดูแล"*

---

**สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง** · อ.สามิตร โกยม · 02-570-8449 · www.itgenius.co.th
