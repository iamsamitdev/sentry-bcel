# Lab วันที่ 3 — Alerting, Release Health, CI/CD และ Capstone

**วันศุกร์ที่ 31 กรกฎาคม 2569 · 09:30–16:30 น.**
เชื่อมโยงกับ `notes/Day3_note.md` Module 3.1–3.6

> ✅ **สถานะการทดสอบ** Lab 3.1–3.5 รันจริงบนเครื่องวิทยากรและยืนยันผลใน Sentry แล้ว
> รวมถึงการอัปโหลด Source Maps ซึ่งพิสูจน์แล้วว่า stack trace กลับมาเป็น TypeScript ต้นฉบับได้จริง

**เงื่อนไขก่อนเริ่ม** ทำ Lab วันที่ 1 และ 2 จบแล้ว (Error + Performance + Distributed Tracing ทำงานครบ)

---

## ⚠️ อ่านก่อน — Sentry เปลี่ยนหน้า Alerts แล้ว

เอกสาร Sentry จำนวนมาก (และ Note ของหลักสูตร) ยังอ้างถึงเมนู **Alerts** ที่แบ่งเป็น
*Issue Alerts* กับ *Metric Alerts* แต่ UI ปัจจุบันของ SaaS เปลี่ยนเป็น

> **Alerts are now Monitors & Alerts.**
> Monitors detect problems and create issues. Issues trigger Alerts to notify your team.

| แนวคิดเดิม | ที่อยู่ใหม่ |
| --- | --- |
| Metric Alert | **Monitors → Metric** (สร้าง Monitor แบบ Metric) |
| Issue Alert | **Monitors → Alerts → Create Alert** |
| Cron Monitor | **Monitors → Cron** |
| Uptime Check | **Monitors → Uptime** |

**โมเดลใหม่แยกเป็น 2 ชั้น**
1. **Monitor** = เงื่อนไขที่ตรวจจับปัญหา → เมื่อเข้าเงื่อนไขจะ **สร้าง Issue**
2. **Alert** = กติกาว่าจะแจ้งใคร ผ่านช่องทางไหน เมื่อมี Issue เกิดขึ้น

การแยกแบบนี้ดีขึ้นสำหรับองค์กร เพราะ "สิ่งที่ตรวจจับ" กับ "ใครควรรู้" ปรับแยกกันได้

---

## ภาพรวม Lab วันที่ 3

| Lab | หัวข้อ | เวลา | ผลลัพธ์ที่จับต้องได้ |
| --- | --- | --- | --- |
| **3.1** | **สร้าง Monitor และ Alert** | **50 นาที** | **Metric Monitor + Alert ทำงาน** |
| **3.2** | **Dashboard และ Discover** | **50 นาที** | **Dashboard ของทีม** |
| **3.3** | **Release Health** | **40 นาที** | **Crash Free Rate อ่านออก** |
| 3.4 | Source Maps + sentry-cli | 50 นาที | Stack trace อ่านได้ |
| **3.5** | **Session Replay** | **30 นาที** | **ดู replay ที่มี error** |
| 3.6 | Jenkins Pipeline + Kubernetes | 50 นาที | Pipeline ครบขั้นตอน |
| 3.7 | **Capstone Project** | 90 นาที | รายงานวินิจฉัย Incident |

---

## Lab 3.1 — สร้าง Monitor และ Alert ⭐

### ส่วน ก) สร้าง Metric Monitor — "Error พุ่งผิดปกติ"

1. เมนูซ้าย → **Monitors** → **All Monitors** → ปุ่ม **Create Alert** (มุมขวาบน)
   หรือเข้าตรงที่ `https://<org>.sentry.io/monitors/new/`
2. **Step 1 of 2** เลือกการ์ด **Metric** → **Next**
3. **Step 2 of 2** ตั้งค่าดังนี้

| หัวข้อ | ค่าที่ใช้ |
| --- | --- |
| 1. Project | `bcel-crm-backend` |
| 1. Environment | All Environments (ของจริงเลือก `production`) |
| 2. Choose Your Metric | **Number of Errors** |
| 3. Dataset | Errors |
| 3. Interval | **1 hour** |
| 3. Visualize | `count` |
| 3. Filter | `is:unresolved` |
| 4. Issue Detection | **Threshold** |
| 4. High priority | **Above** `5` |

4. คลิกไอคอน **ดินสอ** ที่ชื่อ *New Monitor* ด้านบน แล้วตั้งชื่อ

```
BCEL CRM Backend - Error spike (>5 errors/hr)
```

5. **Create Monitor**

**ผลที่ต้องได้** หน้า Monitor แสดงกล่อง **Detect** สรุปว่า

```
Dataset: Errors
Visualize count()  Where is unresolved
Interval: 1 hour
Threshold: Static threshold
  ● High      Above 5
  ● Resolved  Below or equal to 5
```

> 💡 **ทำไมต้อง 1 hour ไม่ใช่ 1 minute** window สั้นเกินจะทำให้ alert ยิงจากสัญญาณรบกวน
> (deploy, restart, health check ล้มชั่วคราว) นี่คือต้นตอของ **Alert Fatigue**
> เริ่มที่กว้างแล้วค่อยแคบลงเมื่อรู้ baseline ของระบบจริง

### ส่วน ข) เพิ่ม Alert เพื่อแจ้งทีม

ที่หน้า Monitor เลื่อนลงมาที่ **Connected Alerts** → **+ New Alert**

| หัวข้อ | ค่าที่ใช้ในห้อง Lab | ค่าที่ควรใช้จริงที่ BCEL |
| --- | --- | --- |
| ช่องทาง | Email (ตัวเอง) | Slack/MS Teams ของทีม CRM + Email หัวหน้าทีม |
| ความถี่ | ทุกครั้ง | ทุก 30 นาที ต่อ Issue |

### ส่วน ค) ทดสอบว่า Alert ยิงจริง

ยิง error ให้เกินเกณฑ์

```bash
for i in $(seq 1 8); do curl -s -o /dev/null http://localhost:8080/api/_debug/sentry/boom; done
```

> ⏱️ Metric Monitor ที่ interval 1 ชั่วโมงจะประเมินตามรอบ ในห้อง Lab ถ้าอยากเห็นผลเร็ว
> ให้สร้าง Monitor ตัวที่ 2 แบบชั่วคราวที่ interval **1 minute** และ threshold **Above 2**
> แล้วลบทิ้งหลังสาธิตจบ

### ส่วน ง) ออกแบบชุด Alert ให้ BCEL (งานกลุ่ม 20 นาที)

เติมตารางนี้ให้ครบ แล้วนำเสนอ

| # | สิ่งที่ต้องรู้ | Monitor แบบไหน | เกณฑ์ | แจ้งใคร | ความเร่งด่วน |
| --- | --- | --- | --- | --- | --- |
| 1 | ระบบ CRM ล่ม | Metric — Number of Errors | > 50/ชม. | ทีม CRM + Ops | ทันที (โทร) |
| 2 | บั๊กใหม่จาก release ล่าสุด | Alert — Issue is new | firstSeen ใน release ปัจจุบัน | ผู้พัฒนาที่ commit | ภายใน 1 ชม. |
| 3 | API ช้าผิดปกติ | Metric — span duration p95 | > 2 วินาที | ทีม CRM | ภายใน 1 ชม. |
| 4 | งานปิดรอบสิ้นวันไม่รัน | **Cron Monitor** | ไม่ check-in ภายในเวลา | ทีม Batch | ทันที |
| 5 | หน้าเว็บ CRM เข้าไม่ได้ | **Uptime Monitor** | HTTP != 200 | Ops | ทันที |

> 🔑 **หลักคิดที่ต้องย้ำ** Alert ที่ดีต้องตอบได้ 3 ข้อ
> (1) **ใครต้องลงมือทำอะไรทันที** (2) **ถ้าไม่ทำจะเกิดอะไร** (3) **มีขั้นตอนแก้ไขที่ชัดเจนไหม**
> ถ้าตอบไม่ได้ทั้ง 3 ข้อ นั่นไม่ใช่ Alert — เป็นแค่ Dashboard widget

---

## Lab 3.2 — Dashboard และ Discover ⭐

### ส่วน ก) สร้าง Dashboard ของทีม

1. เมนูซ้าย → **Dashboards** → **All Dashboards** → **Create Dashboard**
   หรือเข้าตรงที่ `https://<org>.sentry.io/dashboards/new/`
2. คลิกช่อง **+** ตรงกลาง → เลือก **From Widget Library**
3. เลือก **Duration Distribution** (p50/p75/p95 ของ `span.duration`) → **Add Widget**
4. คลิกไอคอน **ดินสอ** ข้างชื่อ *Untitled dashboard* แล้วตั้งชื่อ

```
BCEL CRM - Health Overview
```

5. **Save and Finish**

**ผลที่ได้จริง** Dashboard บันทึกแล้วพร้อมกราฟที่มีข้อมูลจริงจาก Lab วันที่ 2

### ส่วน ข) เพิ่ม Widget ให้ครบชุด (งานผู้เรียน)

เพิ่มอีก 4 widget ให้ Dashboard ตอบคำถามของ 3 บทบาท

| Widget | Dataset | Visualize | ตอบคำถามของใคร |
| --- | --- | --- | --- |
| Error Rate ราย endpoint | Errors | `count()` group by `transaction` | Developer |
| Slowest Transactions | Spans | `p95(span.duration)` group by `transaction` | Developer |
| Top Issues | Issues | `count()` sort by events | Ops |
| Crash Free Rate | Sessions | `crash_free_rate(session)` | Management |

### ส่วน ค) มุมมองแยกตามบทบาท (อภิปราย 15 นาที)

| บทบาท | อยากเห็นอะไร | ไม่ควรเห็นอะไร |
| --- | --- | --- |
| Developer | stack trace, span, suspect commit | ตัวเลขสรุประดับผู้บริหาร |
| Ops / SRE | error rate, latency p95, uptime | รายละเอียดโค้ด |
| Management | Crash Free Rate, จำนวน incident, แนวโน้มรายเดือน | ข้อมูลดิบทั้งหมด |

> 📌 **สำหรับธนาคาร** Dashboard ที่แชร์ให้ผู้บริหารต้องไม่มีข้อมูลที่ระบุตัวลูกค้าได้
> ตรวจให้แน่ว่า widget ไม่ได้ group by tag ที่มี PII

### ส่วน ง) ฝึกใช้ Explore (Discover เดิม)

**Explore → Traces** แล้วลอง query เหล่านี้

| โจทย์ | วิธีทำ |
| --- | --- |
| หา query ที่กินเวลารวมมากที่สุด | Group By `span.description`, Visualize `sum(span.duration)` |
| หา endpoint ที่ p95 เกิน 1 วินาที | Group By `transaction`, Visualize `p95(span.duration)` |
| นับ query ต่อ transaction (หา N+1) | Group By `transaction`, Visualize `count(spans)` |
| ดูเฉพาะ trace ที่มี error | Filter `trace.status:internal_error` |

---

## Lab 3.3 — Release Health ⭐

### ส่วน ก) ดูข้อมูลที่มีอยู่แล้ว

**Explore → Releases** เลือกทั้ง 2 โปรเจกต์ ช่วงเวลา 24H

**ผลที่ได้จริงจากการทำ Lab วันที่ 1–2**

| Release | Project | Adoption | Crash Free Rate | Crashes | New Issues |
| --- | --- | --- | --- | --- | --- |
| `1.0.0` | bcel-crm-frontend | 100% | **50%** | 2 | 1 |
| `1.0.0` | bcel-crm-backend | 0% | — | — | 8 |

**ประเด็นที่ต้องอธิบายในห้อง**

1. **ชื่อ Release แสดงเป็น `1.0.0` ไม่ใช่ `bcel-crm-backend@1.0.0`**
   Sentry ตัดส่วน package ออกจากการแสดงผล แต่ค่าเต็มยังอยู่ ใช้ค้นด้วย `release:1.0.0` ได้

2. **Crash Free Rate ของ frontend = 50% แต่ backend เป็น `—`**
   เพราะ Crash Free Rate คำนวณจาก **Session** ซึ่งฝั่งเบราว์เซอร์ส่งมาตั้งแต่เปิดหน้าเว็บ
   ส่วนฝั่ง server ต้องเปิด `sentry.enable-auto-session-tracking=true` (ทำแล้วในวันที่ 2)
   และต้องมี request เข้ามาต่อเนื่องจึงจะมีข้อมูลพอ

3. **Adoption ของ backend = 0%** เพราะยังไม่มี session ครบตามเกณฑ์

### ส่วน ข) สร้าง Release ที่มีความหมาย

รูปแบบที่หลักสูตรนี้ใช้

```
<ชื่อ service>@<เวอร์ชัน>+<git sha 7 ตัว>
```

ตัวอย่าง

```
bcel-crm-backend@1.42.0+a3f9c21
```

**ทำไมต้องมี git sha** เพื่อให้ Sentry เชื่อม Release กับ commit ได้ ซึ่งเป็นเงื่อนไขของ
**Suspect Commits** — ฟีเจอร์ที่บอกว่า "โค้ดบรรทัดไหน ใครแก้ ทำให้เกิดบั๊กนี้"

ทดลองรัน backend ด้วย release ใหม่

```bash
SENTRY_DSN='<DSN backend>' SENTRY_RELEASE='bcel-crm-backend@1.1.0+test01' mvn spring-boot:run
```

```bash
curl -s -o /dev/null http://localhost:8080/api/_debug/sentry/boom
```

**ผลที่ต้องได้** ใน Releases เห็น `1.1.0+test01` เพิ่มขึ้นมา และ Issue ใหม่ติดป้าย release นั้น

### ส่วน ค) ทดลอง Regression

1. ที่ Issue `IllegalStateException` กด **Resolve**
2. รัน backend ด้วย release ใหม่แล้วยิง `/boom` อีกครั้ง
3. **ผลที่ต้องได้** Issue กลับมาเป็น **Unresolved** พร้อมป้าย **Regression**

> 🎯 นี่คือคุณค่าจริงของ Release Health — ตอบคำถาม "release ที่เพิ่ง deploy ดีขึ้นหรือแย่ลง"
> ได้ด้วยตัวเลข ไม่ใช่ความรู้สึก

---

## Lab 3.4 — Source Maps และ sentry-cli ⭐

> ✅ **ทดสอบจริงแล้วทั้งหมด** — อัปโหลดสำเร็จ และพิสูจน์แล้วว่า stack trace จาก bundle ที่ minify
> และลบไฟล์ `.map` ทิ้งไปแล้ว ยังกลับมาเป็นโค้ด TypeScript ต้นฉบับได้

### ขั้นที่ 1 — สร้าง Organization Auth Token

**Settings → Developer Settings → Organization Tokens → Create New Token**

> ⚠️ **ต้องใช้ Organization Token ไม่ใช่ Personal Token**
>
> | | Organization Token | Personal Token |
> | --- | --- | --- |
> | ผูกกับ | องค์กร | บัญชีบุคคล |
> | ถ้าเจ้าของบัญชีลาออก | ยังใช้ได้ | **พังทันที** |
> | ออกแบบมาเพื่อ | `sentry-cli`, bundler plugin, CI/CD | เรียก API ส่วนตัว |
>
> Jenkins Pipeline ที่ BCEL ต้องรันได้แม้คนที่สร้าง token ย้ายทีมไปแล้ว จึงต้องใช้ Organization Token

Organization Token จะได้ scope **`org:ci`** มาให้อัตโนมัติ ซึ่งครอบคลุมทั้งการสร้าง release
และอัปโหลด source map ไม่ต้องเลือก scope เอง

ค่าที่ได้จะขึ้นต้นด้วย `sntrys_...` และ **แสดงครั้งเดียวเท่านั้น**

> 🔒 **ข้อควรระวังด้านความปลอดภัย**
> - Token แสดงให้เห็นครั้งเดียวตอนสร้าง คัดลอกทันที
> - **ห้าม commit ลง git เด็ดขาด** ให้เก็บใน Jenkins Credentials หรือ Kubernetes Secret
> - ตั้งวันหมดอายุและ revoke เมื่อเลิกใช้

### ขั้นที่ 2 — ติดตั้ง sentry-cli

```bash
cd frontend && npm install --save-dev @sentry/cli
```

เวอร์ชันที่ทดสอบแล้ว: **3.6.2**

ตรวจการเชื่อมต่อ

```bash
SENTRY_AUTH_TOKEN='<token>' SENTRY_ORG='itgenius-qc' node node_modules/@sentry/cli/bin/sentry-cli info
```

**ผลที่ต้องได้**

```
Sentry Server: https://sentry.io
Default Organization: itgenius-qc
Default Project: -

Authentication Info:
  Method: Auth Token
  Scopes:
    - org:ci
```

> 🏦 **ที่ BCEL (Self-hosted)** ต้องเพิ่ม `SENTRY_URL='https://sentry.bcel.local'` ทุกคำสั่ง
> ในห้อง Lab (SaaS) **ไม่ต้องใส่** ถ้าใส่จะชี้ผิดที่และ upload ไม่ขึ้น

### ขั้นที่ 3 — Build แบบ production

แทนที่ placeholder ใน `environment.prod.ts` ก่อน (ปกติ Jenkins ทำให้)

```bash
cd frontend
sed -i "s|__SENTRY_DSN__|<DSN frontend>|g" src/environments/environment.prod.ts
sed -i "s|__SENTRY_RELEASE__|bcel-crm-frontend@1.0.0+lab|g" src/environments/environment.prod.ts
npm run build -- --configuration production
```

**ผลที่ได้จริง** ไฟล์ออกที่ `dist/bcel-crm-frontend/browser/`

```
index.html                    817 B
main-LWD4JQ45.js              587,979 B
main-LWD4JQ45.js.map        5,197,456 B     ← source map
polyfills-JUTM3XWE.js          34,585 B
polyfills-JUTM3XWE.js.map     194,294 B
styles-SCZY3OSS.css             1,787 B
```

> 📌 `angular.json` ตั้ง `"sourceMap": { "scripts": true, "hidden": true }` ไว้แล้ว
> `hidden: true` หมายถึงไม่ใส่คอมเมนต์ `//# sourceMappingURL=` ท้ายไฟล์ `.js`
> ทำให้เบราว์เซอร์ทั่วไปหา map ไม่เจอ แต่ Sentry ยังจับคู่ได้ผ่าน **Debug ID**

### ขั้นที่ 4 — Inject Debug ID แล้วอัปโหลด

```bash
export SENTRY_AUTH_TOKEN='<token>'
export SENTRY_ORG='itgenius-qc'
export SENTRY_PROJECT='bcel-crm-frontend'
export RELEASE='bcel-crm-frontend@1.0.0+lab'
DIST=./dist/bcel-crm-frontend/browser
```

```bash
node node_modules/@sentry/cli/bin/sentry-cli releases new "$RELEASE"
```

```bash
node node_modules/@sentry/cli/bin/sentry-cli sourcemaps inject "$DIST"
```

```bash
node node_modules/@sentry/cli/bin/sentry-cli sourcemaps upload --release "$RELEASE" "$DIST"
```

**ผลที่ได้จริง — ขั้น `inject`**

```
> Found 4 files
> Analyzing 4 sources
> Injecting debug ids

Source Map Debug ID Injection Report
  Modified: The following source files have been modified to have debug ids
    55cbf768-6a5b-57e5-8341-d9b549246e59 - main-BMWC5DFH.js
    07f5a28d-d3c8-5867-993b-a52e0da8a6da - polyfills-JUTM3XWE.js
  Modified: The following sourcemap files have been modified to have debug ids
    55cbf768-6a5b-57e5-8341-d9b549246e59 - main-BMWC5DFH.js.map
    07f5a28d-d3c8-5867-993b-a52e0da8a6da - polyfills-JUTM3XWE.js.map
```

**ผลที่ได้จริง — ขั้น `upload`**

```
> Bundled 4 files for upload
> Bundle ID: b7a81581-b35e-5e43-9199-e8d55dc77065
> Uploaded files to Sentry
> Organization: itgenius-qc
> Projects: bcel-crm-frontend
> Release: bcel-crm-frontend@1.0.0+lab01
> Upload type: artifact bundle
```

> 🔎 **สังเกตว่า Debug ID ของไฟล์ `.js` และ `.map` เป็นค่าเดียวกัน** นี่คือกลไกที่ทำให้จับคู่ได้
> โดยไม่พึ่งชื่อไฟล์ ซึ่งสำคัญมากเพราะเราเปิด `outputHashing: all` ทำให้ชื่อไฟล์เปลี่ยนทุก build

> ⭐ **ต้องทำ `inject` ก่อน `upload` เสมอ** คำสั่ง `inject` ฝัง Debug ID ลงในทั้งไฟล์ `.js`
> และ `.map` ทำให้จับคู่กันได้แม้ชื่อไฟล์เปลี่ยนหรือมี hash ต่างกัน
> ถ้าข้ามขั้นนี้ Sentry จะจับคู่ด้วยชื่อไฟล์อย่างเดียว ซึ่งพังบ่อยมากเมื่อใช้ `outputHashing: all`

### ขั้นที่ 5 — ⭐ ลบ .map ออกก่อน deploy

```bash
find "$DIST" -name '*.map' -delete
```

> 🔒 **สำคัญมากสำหรับธนาคาร** ถ้าปล่อยไฟล์ `.map` ขึ้น production server
> **ใครก็ตามที่เข้าเว็บได้จะดาวน์โหลด source code ทั้งระบบไปอ่านได้** รวมถึง logic
> การตรวจสอบสิทธิ์และ endpoint ภายในที่ไม่ได้ตั้งใจเปิดเผย

### ขั้นที่ 6 — Associate Commits

```bash
node node_modules/@sentry/cli/bin/sentry-cli releases set-commits "$RELEASE" --auto
```

```bash
node node_modules/@sentry/cli/bin/sentry-cli releases finalize "$RELEASE"
```

```bash
node node_modules/@sentry/cli/bin/sentry-cli releases deploys "$RELEASE" new --env production --name "lab-manual-01"
```

**ผลที่ได้จริง**

```
Could not determine any commits to be associated with a repo-based integration.
Proceeding to find commits from local git tree.
Could not find the previous commit. Creating a release with 20 commits.
Success! Set commits for release bcel-crm-frontend@1.0.0+lab01
Finalized release bcel-crm-frontend@1.0.0+lab01
Created new deploy lab-manual-01 for 'production'
```

> ⚠️ **บรรทัดแรกคือประเด็นสำคัญ** `set-commits --auto` จะพยายามใช้ **repo-based integration**
> (GitHub / GitLab / Bitbucket ที่เชื่อมไว้ใน Settings → Integrations) ก่อน
> ถ้ายังไม่ได้เชื่อม มันจะถอยไปอ่านจาก **local git tree** แทน ซึ่ง
> **ได้รายชื่อ commit ครบ แต่ Sentry จะลิงก์ไปดู diff บนเว็บไม่ได้ และ Suspect Commits ทำงานไม่เต็มที่**
>
> ที่ BCEL ถ้าต้องการ Suspect Commits เต็มรูปแบบ ต้องเชื่อม repository integration ก่อน
> (Self-hosted รองรับ GitHub Enterprise และ GitLab self-managed)

### ขั้นที่ 7 — พิสูจน์ว่า Source Maps ใช้ได้จริง

เสิร์ฟ bundle production ที่**ลบ `.map` ไปแล้ว** เพื่อจำลองสภาพ production จริง

```bash
cd dist/bcel-crm-frontend/browser && npx http-server -p 4300 -c-1
```

เปิด `http://localhost:4300` แล้วกดปุ่ม **ทดสอบ Error**

### ✅ เกณฑ์ตรวจผ่าน 3.4 (ยืนยันผลจริงแล้วทั้งหมด)

| # | สิ่งที่ต้องได้ | ผลที่ได้จริง |
| --- | --- | --- |
| 1 | Release ปรากฏใน Sentry | `1.0.0 (lab01)` · Semver: Yes · Package: `bcel-crm-frontend` |
| 2 | มี Artifact bundle | **Source Maps: 4 artifacts** |
| 3 | เชื่อม commit สำเร็จ | **Commits 20 · Files Changed 65** |
| 4 | เห็นผู้เขียน commit | Somchai Vongsa 25% · Bounmy Phommachanh 25% · Samit Koyom 20% · BCEL Lab 15% · Khamla Sisouk 15% |
| 5 | มี Deploy | `production` |
| 6 | **Stack trace อ่านออก** | ดูด้านล่าง ⬇️ |

**Stack trace ที่ได้จริง — จาก bundle ที่ minify และไม่มีไฟล์ `.map` บนเซิร์ฟเวอร์**

```
src/app/customers/customer-list.component.ts:90:11
in CustomerListComponent.throwTestError                                    [In App]

  88     /** ทดสอบ unhandled error ฝั่ง Frontend (วันที่ 2 Lab 2.3) */
  89     throwTestError(): void {
▶ 90       throw new Error('BCEL CRM Frontend: ทดสอบ error ตัวแรก')
  91     }
  92
  93     /** ทดสอบ error ที่เกิดใน setTimeout ซึ่งจับยากกว่า */

src/app/customers/customer-list.component.ts:27:41  in listenerFn          [In App]
Called from: …ngular/core/fesm2022/debug_node.mjs:13838:12
```

> 🎯 **นี่คือหลักฐานที่ต้องฉายบนจอ** Sentry แสดง
> **ชื่อไฟล์ `.ts` จริง · เลขบรรทัดจริง · โค้ดรอบข้าง · แม้แต่คอมเมนต์ภาษาไทย**
> ทั้งที่บนเซิร์ฟเวอร์มีเพียงไฟล์ `main-BMWC5DFH.js` ที่ถูก minify แล้ว และ **ไม่มีไฟล์ `.map` เลย**
> เพราะ Sentry เก็บ source map ไว้ฝั่งตัวเองและจับคู่ผ่าน **Debug ID**
>
> เปรียบเทียบให้ผู้เรียนเห็น: ถ้าไม่อัปโหลด source map จะได้แค่
> `main-BMWC5DFH.js:1:284729` ซึ่งวินิจฉัยอะไรไม่ได้เลย

**หมายเหตุเรื่อง Suspect Commits** repo นี้มีประวัติ commit ย้อนหลังจริงตั้งแต่ 2 มิ.ย. ถึง 31 ก.ค. 2569
จากผู้เขียน 5 คน commit ที่สำคัญที่สุดคือ

```
แก้การคำนวณยอดคงเหลือของบัญชีร่วม
Somchai Vongsa <somchai@bcel.com.la> · 20 ก.ค. 2569
ไฟล์ที่แตะ: CustomerStatementService.java   ⟵ ต้นเหตุของ NullPointerException
```

Sentry จะชี้ commit นี้ให้ได้เมื่อ **เชื่อม repository integration แล้วเท่านั้น**
ถ้ายังไม่ได้เชื่อม จะเห็นแค่รายชื่อ commit ในหน้า Release แต่ไม่มีกล่อง Suspect Commits ใน Issue

---

## Lab 3.5 — Session Replay ⭐

### ขั้นที่ 1 — ตรวจการตั้งค่า

เราตั้งไว้แล้วใน `main.ts` ตั้งแต่วันที่ 2

```typescript
Sentry.replayIntegration({
  maskAllText: true,      // ⭐ ปิดข้อความทั้งหมด
  maskAllInputs: true,    // ⭐ ปิดค่าที่พิมพ์ในฟอร์มทั้งหมด
  blockAllMedia: true     // ⭐ ไม่บันทึกรูป/วิดีโอ
})

replaysSessionSampleRate: 0.0,    // ไม่บันทึกเซสชันปกติ
replaysOnErrorSampleRate: 1.0     // บันทึกเฉพาะเซสชันที่เกิด error
```

> 🏦 **นี่คือการตั้งค่าที่เหมาะกับธนาคาร** — บันทึก "เฉพาะตอนมีปัญหา" และ "ปิดข้อมูลทุกอย่าง"
> ถ้าใช้ค่าปริยายของ Sentry (`maskAllText: false`) **หน้าจอที่มีเลขบัญชีและยอดเงินจะถูกบันทึกไว้ทั้งหมด**
> ซึ่งผิดนโยบายข้อมูลของสถาบันการเงินทันที

### ขั้นที่ 2 — สร้าง Replay

ที่ `http://localhost:4200` ทำ 3 อย่างต่อกัน

1. ค้นหาลูกค้าสักคน
2. กด **เรียก API ที่พัง (ลูกค้า 4999)**
3. กด **ทดสอบ Error**

### ขั้นที่ 3 — ดูผล

**Explore → Replays** เลือกโปรเจกต์ `bcel-crm-frontend`

**ผลที่ได้จริง**

| Replay | User | Duration | Dead clicks | Rage clicks | Errors |
| --- | --- | --- | --- | --- | --- |
| `1e44b911` | Anonymous User | 01:17 | 0 | 0 | **4** |

คลิกเข้าไปดู จะเห็นวิดีโอย้อนหลังพร้อม timeline ของ Console / Network / Errors

### ✅ เกณฑ์ตรวจผ่าน 3.5

| # | สิ่งที่ต้องพิสูจน์ | หมายเหตุ |
| --- | --- | --- |
| 1 | มี Replay อย่างน้อย 1 รายการ | ต้องมี error เกิดขึ้นถึงจะบันทึก |
| 2 | Replay ผูกกับ Issue | เปิด Issue ของ frontend เห็นแท็บ **Replays** |
| 3 | **ข้อความถูกปิดหมด** | ในวิดีโอต้องเห็นเป็นบล็อกสีเทา ไม่ใช่ตัวอักษรจริง |
| 4 | อธิบายเหตุผลของ sample rate ได้ | ทำไม session=0.0 แต่ onError=1.0 |

> 🧪 **การทดลองที่คุ้มค่า** ลองตั้ง `maskAllText: false` ชั่วคราวแล้วดู replay ใหม่
> ผู้เรียนจะเห็นชื่อลูกค้าและตัวเลขทั้งหมดชัดเจน — เป็นภาพที่ทำให้เข้าใจความเสี่ยงทันที
> **อย่าลืมเปลี่ยนกลับ**

---

## Lab 3.6 — Jenkins Pipeline และ Kubernetes

### ส่วน ก) อ่านและแก้ Jenkinsfile

เปิด `Jenkinsfile` ที่รากโปรเจกต์ แก้ 2 จุด

```groovy
environment {
    // ที่ BCEL (Self-hosted) ต้องเปิดบรรทัดนี้ · ในห้อง Lab (SaaS) ให้ลบทิ้ง
    // SENTRY_URL     = 'https://sentry.bcel.local'

    SENTRY_ORG        = 'itgenius-qc'                          // ← แก้เป็น org ของท่าน
    SENTRY_AUTH_TOKEN = credentials('sentry-auth-token-01')    // ← ชื่อ credential ใน Jenkins
    ...
}
```

### ส่วน ข) ไล่ทำความเข้าใจแต่ละ stage

| Stage | ทำอะไร | ถ้าไม่มีจะเป็นอย่างไร |
| --- | --- | --- |
| ติดตั้ง sentry-cli | เตรียมเครื่องมือ | คำสั่ง release ทั้งหมดล้มเหลว |
| สร้าง Release ฝั่ง Backend | `releases new` + `set-commits --auto` | ไม่มี Suspect Commits |
| Build Frontend | แทนที่ `__SENTRY_DSN__` และ `__SENTRY_RELEASE__` | prod build ไม่มี DSN → ไม่ส่งข้อมูล |
| อัปโหลด Source Maps | `inject` → `upload` → **ลบ .map** | stack trace อ่านไม่ออก / source code รั่ว |
| แจ้ง Sentry ว่า Deploy แล้ว | `finalize` + `deploys new` | ไม่รู้ว่า release ไหนขึ้น production เมื่อไร |

### ส่วน ค) จัดการ Secret อย่างปลอดภัย (อภิปราย 20 นาที)

| Secret | เก็บที่ไหน | ห้ามทำอะไร |
| --- | --- | --- |
| `SENTRY_AUTH_TOKEN` | Jenkins Credentials | ห้าม echo ลง console log |
| `SENTRY_DSN` (frontend) | Jenkins Credentials → แทนที่ตอน build | DSN ฝั่ง browser เห็นได้อยู่แล้วในหน้าเว็บ แต่ไม่ควร commit |
| `SENTRY_DSN` (backend) | **Kubernetes Secret** | ห้ามฝังใน image |

ดู `k8s/crm-backend-deployment.yaml` เป็นตัวอย่าง

```yaml
env:
  - name: SENTRY_DSN
    valueFrom:
      secretKeyRef:
        name: sentry-config
        key: SENTRY_DSN
  - name: SENTRY_ENVIRONMENT
    value: "production"
  - name: SENTRY_TRACES_RATE
    value: "0.1"          # ⚠️ production ใช้ 10% ไม่ใช่ 100%
```

### ส่วน ง) ⚠️ กับดัก Relaxed Binding ของ Spring Boot

manifest มีคอมเมนต์เตือนไว้ และเป็นเรื่องที่ควรสาธิตบนจอ

```yaml
# ⚠️ อย่าใช้ชื่อ SENTRY_TAGS_POD_NAME โดยหวังว่าจะได้ tag ชื่อ pod_name
#    เพราะ relaxed binding ของ Spring Boot จะแปลงเป็น sentry.tags.pod.name
- name: K8S_POD_NAME
  valueFrom:
    fieldRef: { fieldPath: metadata.name }
```

วิธีที่ถูกต้องคือรับ env ธรรมดาแล้วตั้ง tag เองในโค้ด

```bash
cp solutions/backend-sentry-config/K8sTagConfig.java backend/src/main/java/la/com/bcel/crm/config/
```

```java
@PostConstruct
public void registerTags() {
    Sentry.configureScope(scope -> {
        scope.setTag("pod_name", podName);
        scope.setTag("node_name", nodeName);
    });
}
```

> 🎯 **ประโยชน์จริง** เมื่อ error เกิดขึ้น ดู tag `pod_name` แล้วรู้ทันทีว่า
> เกิดเฉพาะ pod เดียว (ปัญหา infrastructure — node นั้นมีปัญหา) หรือเกิดทุก pod (บั๊กของโค้ด)

### ส่วน จ) กลยุทธ์ Sampling สำหรับ Production

```bash
cp solutions/backend-sentry-config/BcelTracesSampler.java backend/src/main/java/la/com/bcel/crm/config/
```

| เส้นทาง | Sample rate | เหตุผล |
| --- | --- | --- |
| `/actuator`, `/static` | **0.0** | ไม่มีคุณค่าทางการวินิจฉัย และเรียกถี่มาก |
| `/api/reports`, `/api/eod` | **1.0** | รู้อยู่แล้วว่าช้า ต้องเก็บครบเพื่อวิเคราะห์ |
| `/api/customers`, `/api/tickets` | **0.3** | กระทบผู้ใช้โดยตรง เก็บมากหน่อย |
| อื่น ๆ | **0.05** | ประหยัด quota |

> 💰 นี่คือวิธีคุม quota ที่ได้ผลจริง แทนการตั้งค่าเดียวทั้งระบบ

---

## Lab 3.7 — Capstone Project 🏆

> 90 นาที · แบ่งกลุ่ม · ปิดท้ายด้วยการนำเสนอกลุ่มละ 10 นาที

### โจทย์

> **08:40 น. วันจันทร์** เจ้าหน้าที่สาขา BR-002 โทรแจ้ง Help Desk ว่า
> *"ระบบ CRM ช้ามากตั้งแต่เช้า บางครั้งกดแล้วขึ้นข้อความสีแดง ทำงานไม่ได้เลย"*
>
> ท่านคือทีมที่ต้องวินิจฉัย **โดยใช้ Sentry เป็นหลักฐานเท่านั้น** ห้ามเดา

### ขั้นตอน

**ขั้นที่ 1 — วิทยากรเปิดสถานการณ์ (ผู้เรียนไม่รู้ว่าเป็นอันไหน)**

```bash
curl -X POST 'http://localhost:8080/api/_debug/chaos/enable?scenario=<A-E>'
```

**ขั้นที่ 2 — ผู้เรียนสร้าง traffic ผ่านหน้าเว็บจริง** (ไม่ใช่ curl เพื่อให้ได้ trace ครบ 3 ชั้น)

**ขั้นที่ 3 — วินิจฉัยและเขียนรายงาน**

### แบบฟอร์มรายงาน (ส่งงาน)

| หัวข้อ | เนื้อหา |
| --- | --- |
| **1. อาการที่ผู้ใช้เจอ** | อธิบายจากมุมผู้ใช้ |
| **2. หลักฐานจาก Sentry** | Trace ID / Issue ID / ภาพ waterfall |
| **3. ชั้นที่เป็นปัญหา** | Frontend / Network / Backend logic / Database |
| **4. Span ที่เป็นตัวชี้ขาด** | ชื่อ span + duration + จำนวนครั้ง |
| **5. สมมติฐานต้นเหตุ** | อธิบายเชิงเทคนิค |
| **6. วิธีแก้ระยะสั้น** | ทำได้ภายในวันนี้ |
| **7. วิธีแก้ระยะยาว** | แก้ที่ต้นเหตุ |
| **8. Monitor/Alert ที่ควรมี** | เพื่อไม่ให้เกิดซ้ำโดยไม่มีใครรู้ |
| **9. ผู้ใช้กี่คนได้รับผลกระทบ** | จากตัวเลข Users ของ Issue |

### เกณฑ์การให้คะแนน

| ด้าน | คะแนน | ดูจาก |
| --- | --- | --- |
| ใช้ Sentry เป็นหลักฐานได้จริง | 30 | อ้าง Trace ID / Span จริง ไม่ใช่เดา |
| ระบุชั้นที่เป็นปัญหาถูกต้อง | 25 | ตรงกับ scenario ที่เปิด |
| วิธีแก้สมเหตุสมผล | 20 | แยกสั้น/ยาวได้ชัด |
| ออกแบบ Monitor/Alert ได้ดี | 15 | ตอบ 3 คำถามของ Alert ที่ดีได้ |
| นำเสนอชัดเจน | 10 | เล่าให้คนนอกทีมเข้าใจได้ |

### ส่วนต่อยอด (ถ้าเหลือเวลา)

**สิ่งที่ Sentry ทำได้ดี vs สิ่งที่ต้องเสริม**

| ต้องการรู้ | Sentry | ต้องเสริมด้วย |
| --- | --- | --- |
| Error / Trace / Release ของแอป | ✅ ครบถ้วน | — |
| Session Replay / Web Vitals | ✅ ครบถ้วน | — |
| CPU / RAM / Disk ของ node | ❌ | **Prometheus + Grafana** |
| สถานะ pod / RKE2 cluster | ❌ | **Prometheus + kube-state-metrics** |
| Query plan ของ MariaDB | ⚠️ เห็นแค่ SQL และเวลา | slow query log + `EXPLAIN` |
| Log รวมศูนย์ทั้งองค์กร | ⚠️ เห็นเฉพาะ Breadcrumb | ELK / Loki |

> **ข้อสรุปสำหรับ BCEL** Sentry ตอบ *"แอปพลิเคชันมีปัญหาอะไร และเกิดที่โค้ดบรรทัดไหน"*
> ส่วน Prometheus/Grafana ตอบ *"เครื่องและ cluster สุขภาพเป็นอย่างไร"*
> องค์กรควรมีทั้งสองอย่าง และเชื่อมโยงกันด้วย tag `pod_name` / `node_name`

---

## 📋 เกณฑ์ตรวจผ่าน Lab วันที่ 3

| # | สิ่งที่ต้องพิสูจน์ได้ | วิธีตรวจ |
| --- | --- | --- |
| 1 | สร้าง Metric Monitor ได้ | Monitors → Metric เห็น monitor ของตัวเอง |
| 2 | เข้าใจโมเดล Monitor → Issue → Alert | อธิบายความต่างได้ |
| 3 | สร้าง Dashboard พร้อม widget | Dashboards เห็น `BCEL CRM - Health Overview` |
| 4 | อ่าน Release Health เป็น | อธิบาย Crash Free Rate 50% ของ frontend ได้ |
| 5 | ทำ Regression ให้เกิดได้ | Issue กลับมาพร้อมป้าย Regression |
| 6 | อัปโหลด Source Maps สำเร็จ | Release มี Artifacts และ stack trace อ่านออก |
| 7 | **รู้ว่าต้องลบ .map ก่อน deploy** | อธิบายความเสี่ยงได้ |
| 8 | มี Session Replay ที่ปิดข้อมูล | Replay แสดงข้อความเป็นบล็อกทึบ |
| 9 | อ่าน Jenkinsfile ได้ทุก stage | อธิบายว่าถ้าตัด stage ไหนออกจะเสียอะไร |
| 10 | จบ Capstone พร้อมรายงาน | ส่งแบบฟอร์ม 9 หัวข้อ |

---

## 🔧 ปัญหาที่พบบ่อยในวันที่ 3

| อาการ | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| หาเมนู Alerts ไม่เจอ | UI เปลี่ยนเป็น Monitors & Alerts | ดูหัวข้อ "อ่านก่อน" ด้านบน |
| Alert ไม่ยิงสักที | interval 1 ชั่วโมงยังไม่ครบรอบ | สร้าง monitor ชั่วคราว interval 1 นาที |
| `sentry-cli` บอก `Authentication failed` | token ผิด scope หรือหมดอายุ | สร้างใหม่ให้มี `project:releases` |
| `sourcemaps upload` สำเร็จแต่ stack trace ยังอ่านไม่ออก | ลืม `sourcemaps inject` | ต้อง inject ก่อน upload เสมอ |
| release ใน Sentry กับใน bundle ไม่ตรงกัน | ค่า `__SENTRY_RELEASE__` ไม่ถูกแทนที่ | ตรวจ `environment.prod.ts` หลัง `sed` |
| ไม่มี Suspect Commits | ยังไม่ได้ `set-commits --auto` หรือไม่ได้เชื่อม repo | เชื่อม GitHub/GitLab integration ก่อน |
| ไม่มี Replay เลย | `replaysSessionSampleRate: 0.0` และยังไม่มี error | ต้องทำให้เกิด error ก่อน |
| Crash Free Rate เป็น `—` | ยังไม่มี session พอ | เปิด `enable-auto-session-tracking` และมี traffic ต่อเนื่อง |

---

**สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง** · อ.สามิตร โกยม · 02-570-8449 · www.itgenius.co.th
