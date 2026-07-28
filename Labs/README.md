# Labs — Application Performance Monitoring & Error Tracking ด้วย Sentry

ชุดเอกสาร Lab ประกอบการสอน หลักสูตร 3 วัน (18 ชั่วโมง) สำหรับ
**ธนาคารการค้าต่างประเทศลาว มหาชน (BCEL)** · 29–31 กรกฎาคม 2569

---

## ไฟล์ในโฟลเดอร์นี้

| ไฟล์ | ใช้เมื่อไร | เวลาโดยประมาณ |
| --- | --- | --- |
| [Lab_00_Setup.md](Lab_00_Setup.md) | เช้าวันแรก 09:30–10:00 | 30 นาที |
| [Day1_Lab.md](Day1_Lab.md) | วันพุธที่ 29 ก.ค. | ~3 ชั่วโมง |
| [Day2_Lab.md](Day2_Lab.md) | วันพฤหัสบดีที่ 30 ก.ค. | ~5 ชั่วโมง |
| [Day3_Lab.md](Day3_Lab.md) | วันศุกร์ที่ 31 ก.ค. | ~5 ชั่วโมง |

เอกสารเหล่านี้ใช้คู่กับ

- `../outlines/Sentry_APM_CRM-ERP_หลักสูตรอบรม_3วัน.md` — โครงหลักสูตร
- `../notes/Day1_note.md` … `Day3_note.md` — เนื้อหาบรรยาย
- `../bcel-crm-lite/` — ระบบงานจำลองที่ใช้ทำ Lab

---

## สถานะการทดสอบ

เอกสารทุกไฟล์ **รันจริงบนเครื่องแล้ว** ไม่ใช่การเขียนจากเอกสารอย่างเดียว
ค่าเวลา ข้อความ error ผลลัพธ์ และเส้นทางเมนูใน UI คือสิ่งที่เกิดขึ้นจริง

| ส่วน | สถานะ |
| --- | --- |
| Lab 00 ทั้งหมด | ✅ ทดสอบแล้ว |
| Day 1 — SDK, Issue, Data Scrubbing, กับดัก eventId | ✅ ทดสอบแล้ว |
| Day 2 — Performance, sentry-jdbc, N+1, Slow Query, Angular, Distributed Tracing | ✅ ทดสอบแล้ว |
| Day 3 — Metric Monitor, Dashboard, Release Health, Session Replay | ✅ ทดสอบแล้ว (สร้างของจริงไว้ใน org แล้ว) |
| Day 3 — Source Maps + sentry-cli | ✅ ทดสอบแล้ว (stack trace กลับมาเป็น `.ts` ต้นฉบับได้จริง) |
| Day 3 — Alert notification rule + Regression | ⏳ ยังไม่ได้สร้างจริง (ขั้นตอนอิงจาก UI ที่สำรวจแล้ว) |

**ของจริงที่สร้างไว้แล้วใน org `itgenius-qc`** — ใช้เป็นตัวอย่างฉายบนจอได้ทันที

| สิ่งที่สร้าง | ที่อยู่ / ค่า |
| --- | --- |
| Project `bcel-crm-backend` | id `4511813448171520` |
| Project `bcel-crm-frontend` | id `4511813450661888` |
| Metric Monitor "BCEL CRM Backend - Error spike (>5 errors/hr)" | Monitors → Metric |
| Dashboard "BCEL CRM - Health Overview" | Dashboards |
| Trace ตัวอย่าง end-to-end (126 spans, 3 ชั้น) | `35c83640eff74ec7ab944bd12a0b3e9d` |
| Release พร้อม Source Maps + 20 commits | `bcel-crm-frontend@1.0.0+lab01` |
| Release ฝั่ง backend | `bcel-crm-backend@1.0.0+lab01` |
| Session Replay ที่มี 4 errors | `1e44b911` |

> 🔒 **หลังจบคอร์สอย่าลืม revoke Organization Auth Token** ที่ใช้ทดสอบ
> Settings → Developer Settings → Organization Tokens

---

## สภาพแวดล้อมที่ใช้ทดสอบ

| รายการ | เวอร์ชัน |
| --- | --- |
| Docker / Compose | 29.1.2 / v2.40.3 |
| JDK | 21.0.8 |
| Maven | 3.9.11 |
| Node.js / npm | 22.14.0 / 11.5.1 |
| Spring Boot | 3.4.2 |
| Angular | 20 |
| MariaDB | 12.3 |
| Sentry Java SDK | 8.43.2 |
| @sentry/angular | 10.68.0 |
| @sentry/cli | 3.6.2 |
| Sentry | SaaS · org `itgenius-qc` · region **US** |

---

## เส้นทางของโค้ดตลอด 3 วัน

repo `bcel-crm-lite` มี branch ดังนี้

| Branch | สภาพ | ใช้เมื่อไร |
| --- | --- | --- |
| `main` | **ยังไม่มี Sentry · มีบั๊กครบ** | จุดเริ่มต้นของผู้เรียน |
| `instructor-lab` | ทำ Lab วันที่ 1–2 เสร็จแล้ว | เฉลยสำหรับวิทยากร / กู้สถานะเมื่อผู้เรียนตามไม่ทัน |
| `after-fix` | แก้ N+1 · Slow Query · NPE แล้ว | เทียบผลก่อน–หลังใน Sentry |

ดูว่าแต่ละวันแก้อะไรบ้าง

```bash
git log --oneline main..instructor-lab
```

```bash
git diff main instructor-lab -- backend/src/main/resources/application.properties
```

---

## 3 เรื่องที่วิทยากรควรรู้ก่อนเข้าห้อง

**1. พาธของโปรเจกต์ห้ามมี `&` หรือช่องว่าง**
npm shim บน Windows จะพังทันที ให้ผู้เรียนวางโปรเจกต์ที่ `C:\labs\bcel-crm-lite`
รายละเอียดใน `Lab_00_Setup.md` ข้อ 0.2

**2. หน้า Issues ซ่อน Issue ใหม่ได้**
มุมมองปริยาย *Feed / Recommended* กรองด้วย `is:unresolved` + priority
ถ้าผู้เรียนบอกว่า "ไม่มีอะไรเข้าเลย" ให้สั่งล้างช่องค้นหาและเปลี่ยนช่วงเป็น 24H ก่อนเสมอ

**3. เมนู Alerts ย้ายไปอยู่ใต้ Monitors แล้ว**
UI ปัจจุบันคือ **Monitors & Alerts** ซึ่งต่างจากที่เอกสาร Sentry ส่วนใหญ่เขียนไว้
ดูหัวข้อ "อ่านก่อน" ใน `Day3_Lab.md`

---

## ข้อมูลและความปลอดภัย

> ⚠️ ข้อมูลทั้งหมดในระบบงานจำลองเป็น **ข้อมูลสังเคราะห์ 100%** สร้างด้วยสูตรคณิตศาสตร์
> **ห้ามนำข้อมูลจริงของ BCEL เข้าโปรเจกต์นี้ และห้ามส่งขึ้น sentry.io เด็ดขาด**

---

**สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง** · อ.สามิตร โกยม · 02-570-8449 · www.itgenius.co.th
