# APM & Error Tracking ด้วย Sentry - วันที่ 3: Alerting, Release Health, การผนวก CI/CD และ Workshop ปิดท้าย

**หลักสูตรอบรมเชิงปฏิบัติการ: Application Performance Monitoring & Error Tracking ด้วย Sentry (Self-hosted)**
**จัดอบรมให้: ທະນາຄານການຄ້າຕ່າງປະເທດລາວ / Banque pour le Commerce Extérieur Lao (BCEL)**
**วันที่ 3: ตั้งระบบแจ้งเตือนที่ใช้งานได้จริง ติดตามคุณภาพ Release และผนวกเข้ากับ Jenkins + Kubernetes**
วันที่: 31 กรกฎาคม 2569 | เวลา 09:30–16:30 น. | Onsite Hands-on Workshop
ผู้สอน: อ.สามิตร โกยม | สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง

---

## 🎯 วัตถุประสงค์การเรียนรู้ประจำวัน

เมื่อจบการอบรมวันที่ 3 ผู้เรียนจะสามารถ:

1. แยกความแตกต่างระหว่าง **Issue Alerts** และ **Metric Alerts** และเลือกใช้ให้ถูกสถานการณ์ได้
2. ออกแบบ Alert ที่ **ลด Alert Fatigue** และเชื่อมโยงกับ **SLA / SLO** ของระบบงานได้
3. สร้าง **Dashboard** สำหรับแต่ละบทบาท (Developer / Ops / Management) และใช้ **Discover** สืบค้นเชิงลึกได้
4. อธิบายและใช้งาน **Release Health** (Crash Free Rate, Adoption, Regression) ได้
5. อัปโหลด **Source Maps** ของ Angular เพื่อให้ Stack Trace อ่านได้ และเชื่อม **Suspect Commits** ได้
6. เปิดใช้ **Session Replay** พร้อมตั้งค่า **Privacy / Masking** ที่เหมาะกับระบบงานธนาคาร
7. เขียน **Jenkins Pipeline** ที่สร้าง Release, อัปโหลด Source Maps และแจ้ง Deploy ไปยัง Sentry อัตโนมัติ (Workshop 3.1)
8. จัดการ **DSN และ Secrets** อย่างปลอดภัยบน Jenkins และ Kubernetes (RKE2)
9. วางระบบ Monitoring ครบวงจรให้ระบบงาน CRM/ERP และนำเสนอผลการวินิจฉัยได้ (**Capstone Project**)
10. **⭐ ติดตั้ง Sentry Self-hosted ด้วยตนเองได้สำเร็จ** ตั้งแต่รัน `install.sh` จนส่ง Error เข้า instance ของตัวเองได้ พร้อมตั้งค่าตามนโยบายธนาคาร (**Module 3.8 ปิดท้ายหลักสูตร**)

---

## 🧭 กำหนดการวันที่ 3

| เวลา | หัวข้อ |
| --- | --- |
| 09:30–10:30 | **Module 3.1** ระบบแจ้งเตือน (Alerting & Notifications) + ทบทวนวันที่ 2 |
| 10:30–11:15 | **Module 3.2** Dashboards และการวิเคราะห์ข้อมูล (Discover) |
| 11:15–12:00 | **Module 3.3** Release Health และ Suspect Commits |
| 12:00–13:00 | พักกลางวัน |
| 13:00–13:45 | **Module 3.4** Source Maps และ Source Context |
| 13:45–14:15 | **Module 3.5** Session Replay และ Profiling |
| 14:15–15:00 | **Module 3.6** ผนวก Monitoring เข้ากับ Jenkins + Kubernetes + **Workshop 3.1** |
| 15:00–15:45 | **Module 3.7 Capstone Project** จำลอง Incident และวินิจฉัย + นำเสนอ |
| 15:45–16:30 | **⭐ Module 3.8 ปิดท้ายหลักสูตร: ติดตั้ง Sentry Self-hosted ด้วยตนเอง** |

> ⭐ **ไฮไลต์ของวันนี้อยู่ที่ Module 3.8** ผู้เรียนแต่ละท่านจะได้ **เซิร์ฟเวอร์ของตัวเอง 1 เครื่อง** (8 vCPU / 32 GB) ไปลงมือรัน `./install.sh` ติดตั้ง Sentry Self-hosted ด้วยมือ ซึ่งเป็นสิ่งที่จะนำกลับไปทำจริงที่ BCEL
>
> ระหว่างที่ `install.sh` ทำงาน (8-12 นาที) เราจะทำ **Post-test และสรุปปิดหลักสูตร** ไปพร้อมกัน จึงไม่เสียเวลา

---

## 🔄 ทบทวนวันที่ 2 และตรวจความพร้อม (รวมอยู่ในช่วงต้น Module 3.1)

### เวลา 09:30–09:45 น. (15 นาทีแรกของ Module 3.1)

```bash
# 1) ระบบทั้งหมดยังทำงานอยู่
curl -s -o /dev/null -w 'sentry:  %{http_code}\n' https://sentry.io
curl -s -o /dev/null -w 'jenkins: %{http_code}\n' https://jenkins.lab.itgenius.co.th/login
curl -s -o /dev/null -w 'git:     %{http_code}\n' https://git.lab.itgenius.co.th

# 2) ฐานข้อมูลในเครื่อง + Backend + Frontend
cd bcel-crm-lite && docker compose up -d mariadb
cd backend  && mvn spring-boot:run   &
cd frontend && ng serve              &
```

**คำถามทบทวน 3 ข้อ:**

1. `sentry-trace` header ประกอบด้วยอะไรบ้าง และตัวเลขท้ายสุดหมายถึงอะไร
2. ถ้า Issue หนึ่งแตกเป็นพันรายการเพราะ id อยู่ในข้อความ ควรแก้อย่างไร (บอก 2 วิธี)
3. `traces-sample-rate` มีผลกับ Error ด้วยหรือไม่ เพราะอะไร

---

## 📚 Module 3.1: ระบบแจ้งเตือน (Alerting & Notifications)

### เวลา 09:30–10:30 น.

> 💡 **หัวใจของ Module นี้:** Alert ที่ดีที่สุดคือ Alert ที่ "ทุกครั้งที่ดัง คนที่รับต้องลุกขึ้นมาทำอะไรบางอย่าง" ถ้า Alert ดังแล้วทุกคนกดปิดโดยไม่อ่าน แปลว่าระบบ Alert ล้มเหลว และอันตรายกว่าไม่มี Alert เลย เพราะสร้างความรู้สึกปลอดภัยจอมปลอม

---

### 3.1.1 ประเภทของ Alert

| ประเภท | ทำงานอย่างไร | ตัวอย่างที่ใช้จริง |
| --- | --- | --- |
| **Issue Alert** | เมื่อ **Issue** มีเหตุการณ์ตรงเงื่อนไข | "มี Issue ใหม่ที่ยังไม่เคยเห็นบน production" |
| **Metric Alert** | เมื่อ **ตัวเลขรวม** เกินเกณฑ์ในช่วงเวลาที่กำหนด | "Error rate > 5% ต่อเนื่อง 5 นาที" |

```
Issue Alert            เน้น "เหตุการณ์"          → ตอบสนองรายกรณี
   │                    เกิด Issue ใหม่?
   │                    Issue เก่ากลับมา?
   │                    Issue กระทบผู้ใช้เกิน N คน?
   v
Metric Alert           เน้น "แนวโน้ม/สุขภาพรวม"  → ตอบสนองระดับระบบ
                        Error rate สูงผิดปกติ?
                        P95 latency เกิน SLA?
                        Apdex ตกต่ำกว่าเกณฑ์?
                        Failure rate ของ endpoint หลักพุ่ง?
```

### 3.1.2 สร้าง Issue Alert

**เส้นทาง:** Alerts → Create Alert → **Issues** → เลือก project

**โครงสร้างของ Issue Alert:**

```
WHEN   (trigger)     ─ เหตุการณ์ที่จุดชนวน
IF     (filter)      ─ เงื่อนไขกรองเพิ่มเติม
THEN   (action)      ─ สิ่งที่ต้องทำ
EVERY  (frequency)   ─ ความถี่ที่ยอมให้ดังซ้ำ
```

**ตัวอย่างที่ 1 - Issue ใหม่บน Production (Alert พื้นฐานที่ทุกโปรเจกต์ควรมี)**

```
Alert name: [PROD] Issue ใหม่บนระบบ CRM

WHEN   A new issue is created
IF     The event's environment equals production
  AND  The event's level equals error or fatal
THEN   Send a notification to #crm-alerts (Slack)
  AND  Send a notification to Suggested Assignees
EVERY  30 minutes
```

**ตัวอย่างที่ 2 - Issue ที่กระทบผู้ใช้จำนวนมาก (เร่งด่วน)**

```
Alert name: [URGENT] Issue กระทบผู้ใช้จำนวนมาก

WHEN   The issue is seen by more than 20 users in one hour
IF     The event's environment equals production
THEN   Send a notification to #crm-oncall (Slack)
  AND  Send a notification to Microsoft Teams
  AND  Send a notification via Webhook (ระบบ On-call ขององค์กร)
EVERY  15 minutes
```

**ตัวอย่างที่ 3 - Regression (เรื่องที่ควรอายเป็นพิเศษ)**

```
Alert name: [REGRESSION] ปัญหาที่เคยแก้แล้วกลับมาอีก

WHEN   The issue changes state from resolved to unresolved
IF     The event's environment equals production
THEN   Send a notification to #crm-dev
  AND  Create a Jira ticket (ถ้าเชื่อม integration ไว้)
EVERY  60 minutes
```

**ตัวอย่างที่ 4 - เจาะเฉพาะโมดูลสำคัญ**

```
Alert name: [CRITICAL] ระบบเชื่อมต่อ Core Banking มีปัญหา

WHEN   An event is seen
IF     The event's tags match module equals core-banking-client
  AND  The event's level equals fatal
THEN   Send a notification to #it-ops-critical
  AND  Send email to หัวหน้าฝ่ายปฏิบัติการ
EVERY  5 minutes
```

**ตารางเงื่อนไขที่ใช้บ่อย:**

| ประเภท | เงื่อนไข | ใช้เมื่อ |
| --- | --- | --- |
| Trigger | A new issue is created | จับปัญหาใหม่ |
| Trigger | The issue changes state from resolved to unresolved | จับ Regression ⭐ |
| Trigger | The issue is seen more than X times in Y | จับปัญหาที่ลุกลาม |
| Trigger | The issue is seen by more than X users in Y | จับปัญหาที่กระทบวงกว้าง ⭐ |
| Filter | The event's environment equals ... | แยก prod ออกจาก dev |
| Filter | The event's level equals ... | สนใจเฉพาะร้ายแรง |
| Filter | The event's tags match ... | เจาะเฉพาะโมดูล/สาขา |
| Filter | The issue is older/newer than ... | ตัด issue เก่าออก |
| Filter | The issue's release is the latest | สนใจเฉพาะ release ปัจจุบัน |

### 3.1.3 สร้าง Metric Alert

**เส้นทาง:** Alerts → Create Alert → **Number of Errors / Failure Rate / Latency / Crash Free Rate**

**ตัวอย่างที่ 1 - Error Rate**

```
Dataset:     Errors
Metric:      count()
Filter:      environment:production
Time window: 5 minutes

Critical:    count() > 100      → #crm-oncall + PagerDuty
Warning:     count() > 30       → #crm-alerts
Resolve:     count() < 10
```

**ตัวอย่างที่ 2 - Latency ผูกกับ SLA**

```
Dataset:     Transactions
Metric:      p95(transaction.duration)
Filter:      environment:production AND transaction:"GET /api/customers/{id}/summary"
Time window: 10 minutes

Critical:    p95 > 3000 ms      → #crm-oncall
Warning:     p95 > 1500 ms      → #crm-alerts
```

**ตัวอย่างที่ 3 - Failure Rate ของ API**

```
Dataset:     Transactions
Metric:      failure_rate()
Filter:      environment:production AND transaction.op:http.server
Time window: 15 minutes

Critical:    failure_rate() > 0.05    (5%)
Warning:     failure_rate() > 0.02    (2%)
```

**ตัวอย่างที่ 4 - Apdex (ความพึงพอใจโดยรวม)**

```
Dataset:     Transactions
Metric:      apdex(300)          ← ตั้ง threshold ที่ 300 ms
Filter:      environment:production
Time window: 30 minutes

Critical:    apdex < 0.7
Warning:     apdex < 0.85
```

> 💡 **Apdex คำนวณอย่างไร:** `Apdex = (จำนวน satisfied + จำนวน tolerating/2) / จำนวนทั้งหมด` โดย satisfied = เร็วกว่า T, tolerating = ระหว่าง T ถึง 4T, frustrated = ช้ากว่า 4T ค่า 1.0 คือสมบูรณ์แบบ ค่าต่ำกว่า 0.85 ถือว่าเริ่มมีปัญหา

### 3.1.4 ช่องทางการแจ้งเตือน

| ช่องทาง | บน SaaS (ห้อง Lab) | บน Self-hosted (ที่ BCEL) | เหมาะกับ |
| --- | --- | --- | --- |
| **Email** | ✅ ใช้ได้ทันที ส่งเข้าอีเมลที่สมัคร | ต้องตั้ง SMTP เองใน `sentry/config.yml` | ทุกกรณี (พื้นฐาน) |
| **Slack** | Organization Settings → Integrations → Slack | เหมือนกัน | แจ้งเตือนทีมแบบเรียลไทม์ |
| **Microsoft Teams** | Integrations → Microsoft Teams | เหมือนกัน | องค์กรที่ใช้ Microsoft 365 |
| **Webhook** | Project Settings → Legacy Integrations → WebHooks | เหมือนกัน | ต่อเข้าระบบ On-call / LINE Notify ภายใน |

> 🖥️ **ในห้อง Lab เราจะใช้ Email เป็นหลัก** เพราะ SaaS ส่งเข้าอีเมลที่ท่านสมัครไว้ได้ทันทีโดยไม่ต้องตั้งค่าอะไรเลย เปิดกล่องจดหมายของตัวเองรอไว้ได้เลย
>
> ส่วน Slack/Teams ให้ดูวิทยากรสาธิต เพราะต้องมี workspace ที่มีสิทธิ์ติดตั้งแอป

**ตั้งค่า Email SMTP (สำหรับ Self-hosted ที่ BCEL)**

Self-hosted ไม่มีบริการส่งอีเมลในตัว ต้องชี้ไป SMTP ขององค์กรเอง

```yaml
# sentry/config.yml
mail.backend: 'smtp'
mail.host: 'smtp.bcel.local'
mail.port: 587
mail.username: 'sentry-notify@bcel.com.la'
mail.password: '<password>'
mail.use-tls: true
mail.from: 'sentry@bcel.com.la'
```

```bash
# ทดสอบส่งเมล: ใช้ปุ่ม "Send Test Email" ที่หน้า Admin > Mail ใน UI
# หรือตรวจจาก log ของ worker
docker compose logs -f worker | grep -i mail
```

> 💡 **เคล็ดลับตอนพัฒนา:** ถ้ายังไม่ได้สิทธิ์ SMTP ขององค์กร ให้ติดตั้ง **MailHog** เป็น SMTP จำลอง ซึ่งจะดักจับอีเมลทั้งหมดไว้ให้ดูผ่านหน้าเว็บ ไม่มีอีเมลทดสอบหลุดออกไปข้างนอก
>
> ```bash
> docker run -d --name mailhog -p 1025:1025 -p 8025:8025 mailhog/mailhog
> # แล้วตั้ง mail.host: 'mailhog', mail.port: 1025, mail.use-tls: false
> ```

**Webhook สำหรับต่อระบบภายใน (เช่น ส่งเข้า LINE OA ของทีม):**

```java
// ตัวอย่างบริการรับ webhook แล้วแปลงเป็นข้อความภาษาไทย
@RestController
@RequestMapping("/hooks/sentry")
public class SentryWebhookController {

    @PostMapping
    public ResponseEntity<Void> receive(@RequestBody SentryWebhookPayload payload,
                                        @RequestHeader("Sentry-Hook-Signature") String signature) {
        // 1) ตรวจลายเซ็นก่อนเสมอ กันคนปลอมส่งเข้ามา
        if (!signatureVerifier.isValid(payload, signature)) {
            return ResponseEntity.status(401).build();
        }

        // 2) แปลงเป็นข้อความที่ทีมอ่านเข้าใจ
        String message = """
            🚨 พบปัญหาในระบบ %s
            เรื่อง: %s
            ระดับ: %s | สภาพแวดล้อม: %s
            กระทบผู้ใช้: %d คน
            ดูรายละเอียด: %s
            """.formatted(
                payload.project(), payload.title(), payload.level(),
                payload.environment(), payload.userCount(), payload.url());

        notificationService.sendToTeam(message);
        return ResponseEntity.ok().build();
    }
}
```

### 3.1.5 ออกแบบ Alert ให้ลด Alert Fatigue

```
อาการของ Alert Fatigue:
  ┌────────────────────────────────────────────────────┐
  │ Slack channel มี alert วันละ 300 ข้อความ            │
  │      ↓                                             │
  │ ทีมตั้ง mute channel                                │
  │      ↓                                             │
  │ alert สำคัญจริงถูกกลบ ไม่มีใครเห็น                  │
  │      ↓                                             │
  │ ระบบล่ม 40 นาทีกว่าจะมีคนรู้ ทั้งที่ alert ดังตั้งแต่นาทีที่ 1 │
  └────────────────────────────────────────────────────┘
```

**หลักการ 7 ข้อในการออกแบบ Alert ที่ใช้ได้จริง:**

| # | หลักการ | วิธีปฏิบัติ |
| --- | --- | --- |
| 1 | **Alert ต้องมีการกระทำรองรับ** | ถามตัวเองว่า "ถ้าดังตอนตี 3 จะให้ใครทำอะไร" ถ้าตอบไม่ได้ อย่าตั้ง |
| 2 | **แยกระดับความเร่งด่วน** | Critical → โทร/On-call · Warning → Slack · Info → Email สรุปรายวัน |
| 3 | **กรอง environment เสมอ** | ทุก Alert ต้องมี `environment:production` มิฉะนั้นจะดังจาก dev ตลอด |
| 4 | **ตั้ง frequency ให้เหมาะ** | Critical 5–15 นาที · ทั่วไป 30–60 นาที |
| 5 | **ใช้เกณฑ์ที่กระทบผู้ใช้** | "กระทบ 20 users" มีความหมายกว่า "เกิด 100 ครั้ง" |
| 6 | **ส่งไปให้เจ้าของโค้ด** | ใช้ Ownership Rules + Suggested Assignees |
| 7 | **ทบทวนทุกเดือน** | Alert ที่ดังบ่อยแต่ไม่มีใครทำอะไร = ต้องปรับหรือลบ |

**เมทริกซ์ที่แนะนำสำหรับ BCEL:**

| ระดับ | เงื่อนไข | ช่องทาง | เวลาตอบสนอง |
| --- | --- | --- | --- |
| 🔴 **P1 Critical** | ระบบใช้งานไม่ได้ / กระทบ > 50 users / core-banking ล่ม | On-call + โทรศัพท์ + Teams | 15 นาที |
| 🟠 **P2 High** | Error rate > 5% / P95 > SLA / Regression | Slack #crm-oncall | 1 ชั่วโมง |
| 🟡 **P3 Medium** | Issue ใหม่บน prod / กระทบ < 10 users | Slack #crm-alerts | 1 วันทำการ |
| 🟢 **P4 Low** | Issue บน staging / performance เสื่อมเล็กน้อย | Email สรุปรายสัปดาห์ | ตามสะดวก |

### 3.1.6 เชื่อมโยง Alert กับ SLA / SLO

```
SLI (Service Level Indicator)  = ตัววัด
    ตัวอย่าง: สัดส่วน request ที่สำเร็จและเร็วกว่า 2 วินาที

SLO (Service Level Objective)  = เป้าหมายภายใน
    ตัวอย่าง: SLI >= 99.5% ในรอบ 30 วัน

SLA (Service Level Agreement)  = สัญญากับผู้ใช้/ธุรกิจ
    ตัวอย่าง: ระบบ CRM พร้อมใช้งาน 99.0% ในเวลาทำการ

Error Budget = 100% - SLO = 0.5%
    ในรอบ 30 วัน (เวลาทำการ 8 ชม. x 22 วัน = 10,560 นาที)
    งบความผิดพลาด = 52.8 นาที
```

**แปลง SLO เป็น Alert จริง:**

```
SLO: 99.5% ของ request ต้องสำเร็จและเร็วกว่า 2 วินาที

Metric Alert #1 (Burn rate เร็ว - ใช้ budget หมดใน 2 วัน):
   failure_rate() > 0.02 ในหน้าต่าง 1 ชั่วโมง   → P1 ปลุก On-call

Metric Alert #2 (Burn rate ช้า - ใช้ budget หมดใน 2 สัปดาห์):
   failure_rate() > 0.005 ในหน้าต่าง 6 ชั่วโมง  → P3 แจ้งใน Slack

Metric Alert #3 (Latency):
   p95(transaction.duration) > 2000 ในหน้าต่าง 30 นาที → P2
```

> ✅ **ทำไมต้องมี 2 อัตราการเผาผลาญ (burn rate):** Alert แบบเร็วจับ "ระบบล่มตอนนี้" ส่วน Alert แบบช้าจับ "ระบบค่อย ๆ เสื่อมโดยไม่มีใครสังเกต" ซึ่งอันหลังคือสิ่งที่กัดกินคุณภาพระบบ CRM/ERP ในระยะยาว

---

### 🧪 Lab 3.1 - สร้างชุด Alert สำหรับ BCEL CRM

> **เป้าหมาย:** มีชุด Alert ที่ครอบคลุมและไม่สร้าง noise

> 🖥️ **ในห้อง Lab ให้เลือก Action เป็น Email ส่งถึงตัวเอง** เพราะจะได้รับจริงทันทีโดยไม่ต้องตั้งค่าอะไร ให้เปิดกล่องจดหมายรอไว้
>
> ในตารางข้างล่างที่เขียนว่า `Slack #crm-alerts` คือรูปแบบที่จะใช้จริงที่ BCEL ให้อ่านเป็นแนวทาง แล้วแทนที่ด้วย **Send a notification to Member → ตัวท่านเอง** ในห้อง Lab

**ขั้นที่ 1** สร้าง Issue Alert ตามตารางนี้

| # | ชื่อ Alert | Trigger | Filter | Action ที่ BCEL | Action ในห้อง Lab |
| --- | --- | --- | --- | --- | --- |
| 1 | [PROD] Issue ใหม่ | A new issue is created | environment:production, level:error | Slack #crm-alerts | Email ถึงตัวเอง |
| 2 | [URGENT] กระทบผู้ใช้มาก | seen by > 20 users in 1h | environment:production | Slack #crm-oncall + Email | Email ถึงตัวเอง |
| 3 | [REGRESSION] กลับมาอีก | changes state resolved → unresolved | environment:production | Slack #crm-dev | Email ถึงตัวเอง |

**ขั้นที่ 2** สร้าง Metric Alert 2 ตัว

| # | Dataset | Metric | Filter | Critical | Warning |
| --- | --- | --- | --- | --- | --- |
| 4 | Errors | count() | environment:production | > 100 / 5 min | > 30 / 5 min |
| 5 | Transactions | p95(transaction.duration) | environment:production | > 3000 ms / 10 min | > 1500 ms / 10 min |

**ขั้นที่ 3** ทดสอบให้ Alert ดังจริง

```bash
# ยิง error รัว ๆ ให้ทะลุเกณฑ์ Metric Alert
for i in $(seq 1 150); do
  curl -s http://localhost:8080/api/_debug/sentry/boom > /dev/null
done
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** ได้รับอีเมลแจ้งเตือนภายใน 5-10 นาที และเมื่อหยุดยิง Alert ต้องเปลี่ยนสถานะเป็น Resolved อัตโนมัติ
>
> ⚠️ **ระวังโควตา:** การยิง 150 error รัว ๆ กินโควตาไปพอสมควร ระหว่าง Business trial ยังเหลือเฟือ แต่หลังหมด trial (Developer plan เหลือ 5,000 errors/เดือน) ให้ลดจำนวนลง

**ขั้นที่ 4** ตั้ง Ownership Rules แล้วทดสอบว่า Suggested Assignee ทำงาน

```
path:*/customer/*    #crm-dev
path:*/order/*       #erp-dev
url:*/api/reports/*  #crm-dev
```

**ขั้นที่ 5 (ทางเลือก)** ลองใช้ **Seer** ฟีเจอร์ AI debugging agent ที่มาพร้อม Business trial

เปิด Issue ตัวใดตัวหนึ่งแล้วมองหาปุ่ม **Autofix** หรือ **Seer** ในหน้า Issue ระบบจะวิเคราะห์ stack trace กับโค้ดแล้วเสนอสาเหตุและแนวทางแก้

> 💡 **ฟีเจอร์นี้ Self-hosted ไม่มี** เพราะเป็น closed source ถือเป็นโอกาสได้ลองของที่จะไม่มีให้ใช้เมื่อกลับไปที่ธนาคาร

---

## 📚 Module 3.2: Dashboards และการวิเคราะห์ข้อมูล (Discover)

### เวลา 10:30–11:15 น.

> 💡 **หัวใจของ Module นี้:** Dashboard ที่ดีต้องตอบคำถามของ **คนดู** ไม่ใช่โชว์ทุกอย่างที่มี Developer, Ops และผู้บริหารต้องการเห็นคนละชุดข้อมูล

---

### 3.2.1 Discover - เครื่องมือสืบค้นเชิงลึก

Discover ให้เราสร้าง query แบบกำหนดเองบนข้อมูล Error และ Transaction

**โครงสร้างของ Discover Query:**

| ส่วน | ความหมาย | ตัวอย่าง |
| --- | --- | --- |
| **Dataset** | ชุดข้อมูล | Errors / Transactions |
| **Columns** | คอลัมน์ที่แสดง | `transaction`, `count()`, `p95(transaction.duration)` |
| **Filter** | เงื่อนไข | `environment:production release:...` |
| **Sort** | เรียงลำดับ | `-count()` |
| **Display** | รูปแบบ | Table / Bar / Line / Area / Top 5 |

**ฟังก์ชันรวม (aggregate) ที่ใช้บ่อย:**

```
count()                          จำนวน event
count_unique(user)               จำนวนผู้ใช้ที่ไม่ซ้ำ
count_unique(issue)              จำนวน issue ที่ไม่ซ้ำ
avg(transaction.duration)        ค่าเฉลี่ย
p50/p75/p95/p99(transaction.duration)   เปอร์เซ็นไทล์
failure_rate()                   สัดส่วนที่ล้มเหลว
apdex(300)                       คะแนน Apdex
epm()                            events per minute
tpm()                            transactions per minute
```

**ตัวอย่างการใช้งานจริงในบริบท BCEL:**

```
# 1) หา endpoint ที่ทำให้ผู้ใช้เสียเวลามากที่สุดรวม (ไม่ใช่แค่ช้าที่สุด)
Dataset: Transactions
Columns: transaction, count(), p95(transaction.duration)
Filter:  environment:production transaction.op:http.server
Sort:    -count()
→ คำนวณ count × p95 เอง เพื่อจัดลำดับความสำคัญที่แท้จริง

# 2) ปัญหากระจุกที่สาขาไหน
Dataset: Errors
Columns: branch_code, count(), count_unique(user), count_unique(issue)
Filter:  environment:production
Sort:    -count()

# 3) เปรียบเทียบ 2 release
Dataset: Errors
Columns: release, count(), count_unique(user)
Filter:  environment:production release:["bcel-crm-backend@1.4.0","bcel-crm-backend@1.5.0"]

# 4) Error กระจุกที่ช่วงเวลาไหนของวัน
Dataset: Errors
Columns: count()
Filter:  environment:production
Display: Line chart, interval 1h
→ มักเห็นพีคช่วง 09:00 (เปิดสาขา) และ 16:30 (ปิดรอบ)

# 5) หา query ช้าที่สุด
Dataset: Transactions
Columns: span.description, count(), p95(span.duration)
Filter:  span.op:db.query environment:production
Sort:    -p95(span.duration)

# 6) ลูกค้ากลุ่มไหนเจอปัญหามากที่สุด
Dataset: Errors
Columns: customer_segment, count(), count_unique(user)
Filter:  environment:production
```

### 3.2.2 สร้าง Dashboard ตามบทบาท

**Dashboard 1 - สำหรับทีม Developer**

| Widget | ประเภท | Query |
| --- | --- | --- |
| Issue ใหม่ 24 ชม. | Table | `is:unresolved firstSeen:-24h environment:production` |
| Issue ที่ยังไม่มีคนรับ | Table | `is:unassigned is:unresolved environment:production` |
| Top 10 Error | Bar | `count()` group by `error.type` |
| Endpoint ที่ช้าที่สุด | Table | `p95(transaction.duration)` group by `transaction` |
| Query ที่ช้าที่สุด | Table | `p95(span.duration)` filter `span.op:db.query` |
| แนวโน้ม Error ราย release | Line | `count()` group by `release` |

**Dashboard 2 - สำหรับทีม Ops / On-call**

| Widget | ประเภท | Query |
| --- | --- | --- |
| Error rate ปัจจุบัน | Big Number | `count()` interval 5m |
| Failure rate ราย endpoint | Table | `failure_rate()` group by `transaction` |
| P95 ของ endpoint หลัก | Line | `p95(transaction.duration)` |
| Apdex รวม | Big Number | `apdex(300)` |
| จำนวนผู้ใช้ที่กระทบ | Big Number | `count_unique(user)` filter `level:error` |
| Crash Free Rate | Big Number | จาก Release Health |
| Error แยกตามสาขา | World/Bar | `count()` group by `branch_code` |

**Dashboard 3 - สำหรับผู้บริหาร**

| Widget | ประเภท | Query |
| --- | --- | --- |
| Crash Free Rate รายเดือน | Line | Release Health |
| จำนวนผู้ใช้ที่กระทบ (แนวโน้ม) | Area | `count_unique(user)` filter `level:error` |
| Apdex Score | Big Number | `apdex(300)` |
| เวลาเฉลี่ยจากเจอถึงแก้ (MTTR) | Table | คำนวณจาก Issue resolve time |
| จำนวน Issue เปิด vs ปิด | Bar | เปรียบเทียบรายสัปดาห์ |
| SLO Compliance | Big Number | `1 - failure_rate()` |

> ✅ **หลักการออกแบบ Dashboard:** ผู้บริหารสนใจ **แนวโน้มและผลกระทบต่อธุรกิจ** ไม่ใช่ชื่อ exception ให้ใช้หน่วยที่เขาเข้าใจ เช่น "จำนวนพนักงานที่ทำงานไม่ได้" แทน "จำนวน NullPointerException"

### 3.2.3 การแชร์และตั้งค่ามุมมอง

```
Dashboard → ⋯ → Share
  ├─ คัดลอกลิงก์ (ต้องล็อกอินจึงจะดูได้)
  ├─ Duplicate แล้วให้แต่ละทีมปรับเอง
  └─ ตั้ง Default Dashboard ของโปรเจกต์
```

**เคล็ดลับที่ใช้ได้จริง:**

| เทคนิค | วิธีทำ |
| --- | --- |
| ตั้ง Dashboard ขึ้นจอ TV ในห้อง Ops | ใช้ URL พร้อม query string `?statsPeriod=24h` แล้วเปิด full screen |
| ล็อกช่วงเวลาไว้ | ใส่ `?start=&end=` ใน URL |
| แชร์เฉพาะ environment | ใส่ `?environment=production` |
| ทำ Dashboard สรุปรายสัปดาห์ | ตั้ง `?statsPeriod=7d` แล้ว screenshot ส่งเข้าประชุม |

---

### 🧪 Lab 3.2 - สร้าง Dashboard และใช้ Discover

> **เป้าหมาย:** ตอบคำถามเชิงธุรกิจได้ด้วยข้อมูลจริง

**ขั้นที่ 1** ใช้ Discover ตอบคำถามต่อไปนี้ แล้วบันทึกคำตอบ

| # | คำถาม | Query ที่ใช้ | คำตอบ |
| --- | --- | --- | --- |
| 1 | Endpoint ไหนมี P95 สูงสุด | | |
| 2 | Error ชนิดใดเกิดมากที่สุด | | |
| 3 | สาขาไหนเจอปัญหามากที่สุด | | |
| 4 | ผู้ใช้กี่คนได้รับผลกระทบใน 24 ชม. | | |
| 5 | SQL ตัวไหนช้าที่สุด | | |

**ขั้นที่ 2** บันทึกทั้ง 5 query เป็น Saved Query

**ขั้นที่ 3** สร้าง Dashboard ชื่อ **"BCEL CRM - Ops View"** โดยมีอย่างน้อย 6 widget ตามตารางในหัวข้อ 3.2.2

**ขั้นที่ 4** สร้าง Dashboard ชื่อ **"BCEL CRM - Executive"** ที่ใช้ภาษาที่ผู้บริหารเข้าใจ (หลีกเลี่ยงศัพท์เทคนิค)

---

## 📚 Module 3.3: Release Health และ Suspect Commits

### เวลา 11:15–12:00 น.

> 💡 **หัวใจของ Module นี้:** Release Health ตอบคำถามที่ผู้บริหารถามบ่อยที่สุดว่า **"Deploy เมื่อคืนแล้วระบบดีขึ้นหรือแย่ลง"** ด้วยตัวเลขที่พิสูจน์ได้ ไม่ใช่ความรู้สึก

---

### 3.3.1 แนวคิด Session และ Release Health

```
Session = ช่วงเวลาที่ผู้ใช้ 1 คนใช้งานแอป 1 ครั้ง

  ┌─ Session เริ่ม (เปิดหน้าเว็บ / แอปเริ่มทำงาน)
  │
  ├─ ผู้ใช้ทำงานต่าง ๆ
  │
  └─ Session จบ พร้อมสถานะ:
       ├── healthy    ไม่มีปัญหาเลย                  ✅
       ├── errored    มี error แต่ยังใช้งานต่อได้     ⚠️
       ├── crashed    error ร้ายแรงจนใช้งานต่อไม่ได้  ❌
       └── abnormal   จบผิดปกติ (ปิดเบราว์เซอร์กลางคัน)
```

| ตัวชี้วัด | สูตร | ค่าเป้าหมายที่แนะนำ |
| --- | --- | --- |
| **Crash Free Sessions** | (sessions ทั้งหมด - crashed) / ทั้งหมด | > 99.5% |
| **Crash Free Users** | (users ทั้งหมด - users ที่ crash) / ทั้งหมด | > 99.0% |
| **Adoption** | สัดส่วน session ที่ใช้ release นี้ | ดูว่า release ใหม่ถูกใช้แพร่หลายแค่ไหน |
| **New Issues** | Issue ที่เพิ่งเกิดครั้งแรกใน release นี้ | ยิ่งน้อยยิ่งดี |

```
หน้า Releases:
┌──────────────────────────┬─────────┬───────────┬──────────┬──────────┐
│ Release                  │ Adoption│ Crash Free│ New Issue│ Sessions │
├──────────────────────────┼─────────┼───────────┼──────────┼──────────┤
│ bcel-crm-frontend@1.5.0  │  ▓▓░ 62%│   97.2% 🔴│    8     │  4,210   │
│ bcel-crm-frontend@1.4.0  │  ▓░░ 31%│   99.7% 🟢│    1     │  2,105   │
│ bcel-crm-frontend@1.3.0  │  ░░░  7%│   99.8% 🟢│    0     │    480   │
└──────────────────────────┴─────────┴───────────┴──────────┴──────────┘

อ่านได้ทันทีว่า: release 1.5.0 แย่ลงชัดเจน (97.2% เทียบ 99.7%)
                 และมี Issue ใหม่ 8 ตัว → ควรพิจารณา rollback
```

### 3.3.2 เปิด Release Health

**ฝั่ง Angular (Browser Sessions):**

```typescript
// frontend/src/main.ts
Sentry.init({
  dsn: environment.sentryDsn,
  release: environment.sentryRelease,     // ⭐ ต้องมี ไม่งั้น Release Health ไม่ทำงาน
  environment: environment.sentryEnvironment

  // ไม่ต้องตั้งค่าอะไรเพิ่ม: SDK สาย 8 ขึ้นไปเปิด integration ชื่อ "BrowserSession"
  // ไว้เป็นค่าเริ่มต้นอยู่แล้ว โดยจะสร้าง session ใหม่ทุกครั้งที่โหลดหน้า
  // และทุกครั้งที่เปลี่ยน route ใน SPA (History API)
})
```

หากต้องการ **ปิด** การเก็บ session (เช่น ในสภาพแวดล้อมที่ห้ามเก็บข้อมูลการใช้งาน) ให้กรอง integration ออก:

```typescript
Sentry.init({
  integrations: defaultIntegrations =>
    defaultIntegrations.filter(integration => integration.name !== 'BrowserSession')
})
```

**นิยามสถานะ session ฝั่งเบราว์เซอร์:**

| สถานะ | เกิดเมื่อ |
| --- | --- |
| `crashed` | มี unhandled error หรือ unhandled promise rejection หลุดถึง global handler |
| `errored` | SDK จับ event ที่มี exception (รวมถึงที่เรา `captureException` เอง) |
| `healthy` | ไม่เข้าเงื่อนไขข้างบน |

> 💡 **เงื่อนไขของ Crash Free Users:** ต้องตั้งค่า `user` ด้วย (ผ่าน `Sentry.setUser()` หรือ `initialScope`) มิฉะนั้นจะมีเฉพาะตัวเลข Crash Free Sessions แต่ไม่มี Crash Free Users

**ฝั่ง Spring Boot (Server Sessions):**

```properties
# application.properties
sentry.release=${SENTRY_RELEASE:bcel-crm-backend@1.0.0}
sentry.enable-auto-session-tracking=true
sentry.session-flush-timeout-millis=15000
```

> ⚠️ **เงื่อนไขบังคับ:** ถ้าไม่ตั้งค่า `release` ระบบ Release Health จะไม่มีข้อมูลเลย และหน้า Releases จะว่างเปล่า นี่คือเหตุผลที่เราลงทุนตั้ง Release ให้ถูกต้องตั้งแต่วันที่ 1

### 3.3.3 สร้าง Release และเชื่อม Commit

> 🖥️ **ค่าที่ใช้จริงในห้อง Lab** ตลอด Module 3.3 ถึง 3.6 เอกสารจะเขียน `SENTRY_URL='https://sentry.bcel.local'` ซึ่งเป็นชื่อของสภาพแวดล้อมจริงที่ BCEL จะใช้หลังจบหลักสูตร **ในห้อง Lab ให้ใช้ค่าเหล่านี้แทน**
>
> ```bash
> # ⭐ ไม่ต้องตั้ง SENTRY_URL เลย เพราะ sentry-cli ชี้ไป sentry.io เป็นค่าเริ่มต้นอยู่แล้ว
> export SENTRY_ORG='<org slug ของท่าน>'   # ดูได้จาก URL หรือ Settings
> export SENTRY_PROJECT='bcel-crm-backend'
> export SENTRY_AUTH_TOKEN='<token ที่สร้างจาก Settings -> Auth Tokens>'
> ```
>
> **สร้าง Auth Token บน SaaS** ไปที่ `Settings → Developer Settings → Auth Tokens → Create New Token` เลือก scope `project:releases`, `project:read`, `org:read` เหมือนกับ Self-hosted ทุกประการ

```bash
# ตั้งค่า sentry-cli
export SENTRY_URL='https://sentry.bcel.local'
export SENTRY_ORG='bcel'
export SENTRY_PROJECT='bcel-crm-backend'
export SENTRY_AUTH_TOKEN='<auth token>'      # ⚠️ ความลับ เก็บใน Jenkins Credentials

# ต่อเนื่องจากวันที่ 2: เราจบวันที่ 2 ด้วย release 1.1.0 (หลังแก้ N+1 และ Slow Query)
# วันนี้จะออก release 1.2.0 เป็นรอบถัดไป
VERSION="bcel-crm-backend@1.2.0"

# 1) สร้าง release
sentry-cli releases new "$VERSION"

# 2) ผูก commit เข้ากับ release (ทำให้ Suspect Commits ทำงาน)
sentry-cli releases set-commits "$VERSION" --auto

# 3) ปิด release (บอกว่าไม่มีอะไรจะเพิ่มแล้ว)
sentry-cli releases finalize "$VERSION"

# 4) แจ้งว่า deploy ไปยัง environment ใด
sentry-cli releases deploys "$VERSION" new \
  --env production \
  --name "deploy-$(date +%Y%m%d-%H%M)"
```

**ติดตั้ง sentry-cli:**

```bash
# วิธีที่ 1: สคริปต์ทางการ
curl -sL https://sentry.io/get-cli/ | bash

# วิธีที่ 2: npm (เหมาะกับ Jenkins agent ที่มี Node อยู่แล้ว)
npm install -g @sentry/cli

# วิธีที่ 3: Docker (ใช้บนเซิร์ฟเวอร์หรือ Jenkins agent เท่านั้น ผู้เรียนไม่ต้องใช้)
docker run --rm -v "$(pwd):/work" -w /work \
  -e SENTRY_URL -e SENTRY_ORG -e SENTRY_AUTH_TOKEN \
  getsentry/sentry-cli releases new "$VERSION"
# ⚠️ บน SaaS ไม่ต้องส่ง -e SENTRY_URL เพราะชี้ไป sentry.io เป็นค่าเริ่มต้นอยู่แล้ว

sentry-cli --version
sentry-cli info                    # ตรวจว่าเชื่อมต่อ Sentry ได้
```

> 📖 **หมายเหตุความเข้ากันได้:** sentry-cli รองรับ Sentry Self-Hosted ตั้งแต่รุ่น **24.11.1** ขึ้นไป ถ้าติดตั้ง Sentry เวอร์ชันเก่ากว่านี้ ให้อัปเกรดก่อน

### 3.3.4 Suspect Commits

เมื่อผูก commit เข้ากับ release แล้ว Sentry จะวิเคราะห์ว่า commit ใดแตะไฟล์ที่ปรากฏใน stack trace แล้วชี้ว่า **"commit นี้น่าจะเป็นต้นเหตุ"**

**ขั้นตอนการเปิดใช้:**

1. เชื่อม Git integration: Organization Settings → Integrations → GitHub / GitLab / Bitbucket (หรือใช้ `--auto` กับ sentry-cli ในกรณี self-hosted ที่ไม่ได้เชื่อม)
2. ผูก repository เข้ากับโปรเจกต์
3. รัน `sentry-cli releases set-commits` ทุกครั้งที่ deploy
4. ตั้ง `sentry.in-app-includes` ให้ถูกต้อง (ทำไปแล้ววันที่ 1)

**ผลลัพธ์ที่เห็นในหน้า Issue:**

```
┌───────────────────────────────────────────────────────────┐
│ 🔍 Suspect Commits                                        │
│                                                           │
│  a3f5c9d  แก้การคำนวณยอดคงเหลือของบัญชีร่วม               │
│           somchai@bcel.com.la · 2 ชั่วโมงที่แล้ว          │
│           แตะไฟล์: CustomerStatementService.java (บรรทัด 88)│
│                                    [ Assign to somchai ]  │
└───────────────────────────────────────────────────────────┘
```

> ✅ **คุณค่าที่วัดได้:** จากประสบการณ์การใช้งานจริง Suspect Commits ช่วยลดเวลาหาต้นเหตุจากหลักชั่วโมงเหลือหลักนาที เพราะไม่ต้องไล่ `git log` เอง

### 3.3.5 ติดตามว่า Release ใหม่ดีขึ้นหรือแย่ลง

**วิธีที่ 1 - เปรียบเทียบใน Releases page** ดู Crash Free Rate เทียบกัน (ดูภาพในหัวข้อ 3.3.1)

**วิธีที่ 2 - Discover comparison**

```
Dataset: Errors
Columns: release, count(), count_unique(user), count_unique(issue)
Filter:  environment:production
```

**วิธีที่ 3 - ค้นหา Issue ที่เกิดครั้งแรกใน release ใหม่**

```
firstRelease:bcel-crm-backend@1.2.0 is:unresolved
```

**เกณฑ์ตัดสินใจ Rollback ที่แนะนำสำหรับ BCEL:**

| สัญญาณ | เกณฑ์ | การกระทำ |
| --- | --- | --- |
| Crash Free Rate | ลดลง > 1% เทียบ release ก่อน | 🔴 พิจารณา rollback ทันที |
| Issue ใหม่ระดับ fatal | มีตั้งแต่ 1 ตัว | 🔴 rollback |
| P95 latency | เพิ่มขึ้น > 50% | 🟠 ตรวจสอบด่วน |
| จำนวนผู้ใช้ที่กระทบ | > 5% ของผู้ใช้ทั้งหมด | 🔴 rollback |
| Error rate | เพิ่มขึ้น > 2 เท่า | 🟠 ตรวจสอบด่วน |

---

## 📚 Module 3.4: Source Maps และ Source Context

### เวลา 13:00–13:45 น.

> 💡 **หัวใจของ Module นี้:** ถ้าไม่มี Source Maps, Stack Trace ของ Angular ที่ผ่าน production build จะเป็นขยะที่อ่านไม่ออก ทำให้ Error Tracking ฝั่ง Frontend แทบไร้ค่า

---

### 3.4.1 ปัญหาที่ Source Maps แก้

```
❌ ไม่มี Source Maps:
   TypeError: Cannot read properties of undefined (reading 'balance')
     at main-4f2a9b.js:1:284719
     at main-4f2a9b.js:1:284955
     at t.next (polyfills-8c3d.js:1:9284)
   → ไม่มีทางรู้เลยว่าโค้ดบรรทัดไหน

✅ มี Source Maps:
   TypeError: Cannot read properties of undefined (reading 'balance')
     at CustomerDetailComponent.calculateTotal
        (src/app/customers/customer-detail.component.ts:87:24)
     at CustomerService.getSummary
        (src/app/customers/customer.service.ts:42:18)
   → เห็นไฟล์ บรรทัด และคอลัมน์จริง พร้อมโค้ดรอบ ๆ
```

### 3.4.2 เปิดการสร้าง Source Maps ใน Angular

```json
// frontend/angular.json
{
  "projects": {
    "bcel-crm-frontend": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "sourceMap": {
                "scripts": true,
                "styles": false,
                "hidden": true,
                "vendor": false
              },
              "namedChunks": false,
              "outputHashing": "all"
            }
          }
        }
      }
    }
  }
}
```

| ตัวเลือก | ค่าที่แนะนำ | เหตุผล |
| --- | --- | --- |
| `scripts` | `true` | ต้องมี source map ของ JS |
| `styles` | `false` | ไม่จำเป็นสำหรับ error tracking |
| `hidden` | `true` ⭐ | สร้าง `.map` แต่ **ไม่ใส่ comment `sourceMappingURL`** ใน bundle ทำให้ผู้ใช้ทั่วไปเปิดดูซอร์สไม่ได้ แต่ Sentry ยังใช้ได้ผ่าน Debug ID |
| `vendor` | `false` | ลดขนาด ไม่ต้อง debug โค้ด library |

> ⚠️ **สำคัญมากสำหรับระบบธนาคาร:** ต้องตั้ง `hidden: true` เสมอ และ **ต้องลบไฟล์ `.map` ออกจาก server ก่อน deploy** เพราะถ้าปล่อยไว้ ใครก็ตามที่เข้าถึงเว็บได้จะดาวน์โหลด source code ทั้งหมดของระบบ CRM ไปดูได้

### 3.4.3 อัปโหลด Source Maps ด้วย sentry-cli

**ขั้นตอนมาตรฐาน 3 ขั้น:**

```bash
cd frontend

# --- ขั้นที่ 1: build production ---
export SENTRY_RELEASE="bcel-crm-frontend@1.2.0"
ng build --configuration production

# --- ขั้นที่ 2: ฉีด Debug ID เข้าไปในไฟล์ ---
# Debug ID คือรหัสที่ใช้จับคู่ไฟล์ที่ถูก minify กับ source map ของมัน
sentry-cli sourcemaps inject ./dist/bcel-crm-frontend/browser

# ตรวจว่า inject สำเร็จ (ท้ายไฟล์ .js ต้องมี //# debugId=...)
tail -c 200 ./dist/bcel-crm-frontend/browser/main-*.js

# --- ขั้นที่ 3: อัปโหลด ---
sentry-cli sourcemaps upload \
  --org bcel \
  --project bcel-crm-frontend \
  --release "$SENTRY_RELEASE" \
  ./dist/bcel-crm-frontend/browser

# --- ขั้นที่ 4 (สำคัญมาก): ลบไฟล์ .map ก่อน deploy ---
find ./dist/bcel-crm-frontend/browser -name '*.map' -delete
echo "ลบ source map ออกจาก artifact แล้ว"
```

**ตรวจว่าอัปโหลดสำเร็จ:**

```bash
sentry-cli sourcemaps explain <event-id>   # วิเคราะห์ว่าทำไม event นี้ยัง unminify ไม่ได้
```

หรือดูใน UI ที่ **Project Settings → Source Maps → Artifact Bundles**

### 3.4.4 การแก้ปัญหา Source Maps ที่พบบ่อย

| อาการ | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| Stack trace ยัง minify | ยังไม่ได้ inject debug ID | รัน `sourcemaps inject` ก่อน `upload` เสมอ |
| Stack trace ยัง minify | `release` ตอน upload ≠ `release` ใน `Sentry.init()` | ทำให้ตรงกันเป๊ะ ๆ (ใช้ตัวแปรเดียวกัน) |
| อัปโหลดแล้วแต่ไม่พบไฟล์ | path ผิด (Angular 17+ ใช้ `dist/<project>/browser`) | ตรวจ path จริงจาก `ls dist/` |
| บางไฟล์ unminify ได้ บางไฟล์ไม่ได้ | อัปโหลดไม่ครบ | อัปโหลดทั้งโฟลเดอร์ ไม่ใช่ทีละไฟล์ |
| ใช้ Service Worker แล้วพัง | inject ทำให้ hash ใน `ngsw.json` ไม่ตรง | รัน `ngsw-config` ใหม่หลัง inject |

**กรณีใช้ Angular Service Worker:**

```bash
sentry-cli sourcemaps inject ./dist/bcel-crm-frontend/browser
# ⭐ ต้อง regenerate ngsw.json หลัง inject
node_modules/.bin/ngsw-config ./dist/bcel-crm-frontend/browser ./ngsw-config.json
sentry-cli sourcemaps upload ...
```

### 3.4.5 Source Context สำหรับ Java

ฝั่ง Backend ก็ทำให้เห็นโค้ดจริงใน Stack Trace ได้เช่นกัน โดยใช้ `sentry-maven-plugin`:

```xml
<!-- backend/pom.xml -->
<plugin>
  <groupId>io.sentry</groupId>
  <artifactId>sentry-maven-plugin</artifactId>
  <version>0.11.0</version>
  <extensions>true</extensions>
  <configuration>
    <!-- ที่ BCEL (Self-hosted): org=bcel และต้องมี <url> -->
    <!-- ในห้อง Lab (SaaS): org=<org slug ของท่าน> และ *ลบ* บรรทัด <url> ออก -->
    <org>bcel</org>
    <project>bcel-crm-backend</project>
    <url>https://sentry.bcel.local</url>
    <!-- อ่าน auth token จาก environment variable SENTRY_AUTH_TOKEN -->
    <skipAutoInstall>true</skipAutoInstall>
  </configuration>
  <executions>
    <execution>
      <phase>generate-resources</phase>
      <goals>
        <goal>uploadSourceBundle</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

> 💡 **ผลลัพธ์:** เมื่อเปิด Issue จะเห็นโค้ด Java จริง ๆ รอบบรรทัดที่พัง ไม่ต้องเปิด IDE ควบคู่ ตรวจเวอร์ชันล่าสุดของปลั๊กอินได้ที่ [GitHub: sentry-maven-plugin](https://github.com/getsentry/sentry-maven-plugin/releases)

---

## 📚 Module 3.5: Session Replay และ Profiling

### เวลา 13:45–14:15 น.

> 💡 **หัวใจของ Module นี้:** Session Replay คือเครื่องมือที่ทรงพลังที่สุดในการปิดช่องว่าง "ผู้ใช้จำไม่ได้ว่ากดอะไร" แต่ก็เป็นเครื่องมือที่ **เสี่ยงด้านความเป็นส่วนตัวมากที่สุด** ในบริบทธนาคาร ต้องตั้งค่าอย่างระมัดระวังที่สุด

---

### 3.5.1 Session Replay คืออะไร

Session Replay **ไม่ใช่การอัดวิดีโอ** แต่เป็นการบันทึกการเปลี่ยนแปลงของ DOM แล้วนำมาเล่นกลับ (ทำให้ไฟล์เล็กมากและปกปิดข้อมูลได้)

```
สิ่งที่ Replay บันทึก:
  ├── โครงสร้าง DOM และการเปลี่ยนแปลง
  ├── การเคลื่อนไหวและคลิกเมาส์
  ├── การเลื่อนหน้าจอ
  ├── การเปลี่ยนขนาดหน้าต่าง
  ├── Console log และ Network request (ตั้งค่าได้)
  └── ไม่ได้บันทึกเป็นไฟล์วิดีโอ (จึง mask ข้อมูลได้)
```

### 3.5.2 ตั้งค่า Privacy / Masking สำหรับระบบธนาคาร

```typescript
// frontend/src/main.ts
Sentry.init({
  integrations: [
    Sentry.replayIntegration({
      // ===== การปกปิดข้อมูล (ตั้งเข้มที่สุดสำหรับ BCEL) =====

      // ปิดบังข้อความทั้งหมดเป็น *** โดยปริยาย
      maskAllText: true,

      // ปิดบังค่าใน input ทุกช่อง
      maskAllInputs: true,

      // บล็อกรูปภาพ วิดีโอ iframe ทั้งหมด
      blockAllMedia: true,

      // เลือกบล็อกทั้ง element (ไม่แสดงแม้โครงสร้าง)
      block: ['.sensitive-block', '[data-sensitive]', '#account-panel'],

      // เลือกปิดบังเฉพาะข้อความใน element เหล่านี้
      mask: ['.customer-name', '.account-no', '.balance', '.phone'],

      // ยอมให้แสดงตามปกติ (ใช้กับส่วนที่ปลอดภัยแน่นอน เช่น ปุ่ม ป้ายกำกับ)
      unmask: ['.btn-label', '.page-title', '.nav-menu'],

      // ===== การเก็บข้อมูลประกอบ =====
      networkDetailAllowUrls: [],      // ไม่เก็บ body ของ request ใด ๆ
      networkCaptureBodies: false,     // ⭐ สำคัญมาก
      networkRequestHeaders: [],       // ไม่เก็บ header
      networkResponseHeaders: []
    })
  ],

  // ===== อัตราการบันทึก =====
  replaysSessionSampleRate: 0.0,   // ไม่บันทึก session ปกติเลย
  replaysOnErrorSampleRate: 1.0    // บันทึกเฉพาะ session ที่เกิด error
})
```

**ปกปิดในระดับ HTML template (แนะนำให้ทำควบคู่):**

```html
<!-- customer-detail.component.html -->

<!-- ปิดบังข้อความ แต่ยังเห็นโครงสร้างและตำแหน่ง -->
<span data-sentry-mask class="customer-name">{{ customer.fullName }}</span>
<span data-sentry-mask class="account-no">{{ account.accountNo }}</span>

<!-- บล็อกทั้ง element (ไม่เห็นแม้ว่ามีอะไรอยู่) -->
<div data-sentry-block class="account-balance-panel">
  <h3>ยอดคงเหลือ</h3>
  <p>{{ account.balance | currency:'LAK' }}</p>
</div>

<!-- ยอมให้แสดง (ข้อความคงที่ ไม่มีข้อมูลลูกค้า) -->
<h1 data-sentry-unmask>ระบบจัดการข้อมูลลูกค้า</h1>
<button data-sentry-unmask>บันทึกข้อมูล</button>
```

**ตารางตัดสินใจว่าอะไรต้องปกปิด:**

| ประเภทข้อมูล | การตั้งค่า | เหตุผล |
| --- | --- | --- |
| ชื่อ-นามสกุลลูกค้า | `mask` | PII |
| เลขบัญชี / เลขบัตร | `block` | ข้อมูลลับสูงสุด |
| ยอดเงิน | `block` | ข้อมูลลับสูงสุด |
| เบอร์โทร / อีเมล | `mask` | PII |
| รหัสผ่าน / OTP | `block` + ห้ามบันทึก | ความลับ |
| ปุ่ม / เมนู / หัวข้อ | `unmask` | ช่วยให้ดู replay แล้วเข้าใจ |
| ข้อความ error | `unmask` | จำเป็นต่อการวินิจฉัย |

> ⛔ **กฎเหล็กของ BCEL:** ก่อนเปิด Session Replay บน production **ต้องผ่านการอนุมัติจากฝ่าย Compliance / Data Protection ขององค์กร** และควรทดสอบดู replay ในสภาพแวดล้อม staging ด้วยข้อมูลจำลองก่อนเสมอ เพื่อยืนยันด้วยตาว่าไม่มีข้อมูลใดหลุด

### 3.5.3 การใช้ Replay ในการวินิจฉัย

```
Replay Player:
┌────────────────────────────────────────────────────────────┐
│ ▶ ━━━━━━━━●━━━━━━━━━━━━━━━━━━━  00:42 / 01:35   [1x] [⛶] │
├────────────────────────────────────────────────────────────┤
│                                                            │
│    [หน้าจอที่ผู้ใช้เห็น พร้อมข้อมูลที่ถูก mask เป็น ***]    │
│                                                            │
├────────────────────────────────────────────────────────────┤
│ Breadcrumbs        │ Console      │ Network   │ Errors     │
│ 00:12 คลิก "ค้นหา" │ warn: ...    │ GET /api  │ 🔴 TypeError│
│ 00:28 พิมพ์ในช่อง   │              │ 200 145ms │            │
│ 00:41 คลิก "ดูราย.."│              │ GET /sum  │            │
│ 00:42 🔴 เกิด error │              │ 500 2.4s  │            │
└────────────────────────────────────────────────────────────┘
```

**เชื่อมโยง Replay กับ Issue และ Trace:**

- ในหน้า **Issue** จะมีแท็บ **Replays** แสดง session ที่เกิด error นี้
- ในหน้า **Replay** จะมีลิงก์ไปยัง **Issue** และ **Trace** ของช่วงเวลานั้น
- ทำให้เห็นครบวงจร: **ผู้ใช้ทำอะไร → error อะไร → backend ช้าตรงไหน → SQL ตัวไหน**

### 3.5.4 Profiling

**Profiling** ต่างจาก Tracing ตรงที่เจาะลึกถึงระดับ **ฟังก์ชันภายในโค้ด** ว่าใช้ CPU ไปกับอะไรบ้าง

| ระดับ | เครื่องมือ | ตอบคำถามว่า |
| --- | --- | --- |
| Trace | Distributed Tracing | ช้าที่ **บริการ** ไหน |
| Span | Instrumentation | ช้าที่ **การทำงาน** ไหน (query, http call) |
| **Profile** | Profiling | ช้าที่ **ฟังก์ชัน** ไหน บรรทัดไหน |

**เปิดใช้ฝั่ง Java:**

```xml
<dependency>
  <groupId>io.sentry</groupId>
  <artifactId>sentry-async-profiler</artifactId>
</dependency>
```

```properties
# application.properties
sentry.profile-session-sample-rate=1.0
sentry.profile-lifecycle=TRACE
```

**เปิดใช้ฝั่ง Browser:**

```typescript
import * as Sentry from '@sentry/angular'

Sentry.init({
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.browserProfilingIntegration()
  ],
  tracesSampleRate: 1.0,
  profilesSampleRate: 1.0     // สัดส่วนเทียบกับ transaction ที่ถูก sample
})
```

> ⚠️ **ข้อควรระวัง:** Profiling กิน CPU และแบนด์วิดท์มากกว่า Tracing พอสมควร ในสภาพแวดล้อม production ควรตั้ง `profilesSampleRate` ต่ำมาก (0.01–0.05) หรือเปิดชั่วคราวเฉพาะช่วงที่ต้องการสืบสวนปัญหา แล้วปิดกลับ

> 💡 **คำแนะนำสำหรับ BCEL:** เริ่มจาก Tracing + Session Replay ให้เชี่ยวชาญก่อน แล้วค่อยเปิด Profiling ในระยะที่สอง เพราะเครื่องมือสองอย่างแรกให้ผลตอบแทนต่อความพยายามสูงกว่ามาก

---

### 🧪 Lab 3.3 - Release Health, Source Maps และ Replay

> **เป้าหมาย:** เห็นข้อมูล Release Health จริง, Stack Trace ที่อ่านได้ และ Replay ที่ปกปิดข้อมูลถูกต้อง

**ขั้นที่ 1 - สร้าง release และ build production**

```bash
cd frontend
# ในห้อง Lab: ไม่ต้องตั้ง SENTRY_URL และใช้ org ของท่านเอง
export SENTRY_ORG='<org slug ของท่าน>'   # ดูได้จาก URL หรือ Settings
export SENTRY_PROJECT='bcel-crm-frontend'
export SENTRY_AUTH_TOKEN='<token>'
export SENTRY_RELEASE='bcel-crm-frontend@1.2.0'

sentry-cli releases new "$SENTRY_RELEASE"
ng build --configuration production
sentry-cli sourcemaps inject ./dist/bcel-crm-frontend/browser
sentry-cli sourcemaps upload --release "$SENTRY_RELEASE" ./dist/bcel-crm-frontend/browser
sentry-cli releases set-commits "$SENTRY_RELEASE" --auto
sentry-cli releases finalize "$SENTRY_RELEASE"
sentry-cli releases deploys "$SENTRY_RELEASE" new --env production
```

**ขั้นที่ 2 - เสิร์ฟไฟล์ที่ build แล้ว (ลบ .map ก่อน)**

```bash
find ./dist/bcel-crm-frontend/browser -name '*.map' -delete
npx http-server ./dist/bcel-crm-frontend/browser -p 4300
```

**ขั้นที่ 3 - เปิดหน้าเว็บแล้วสร้าง error**

เปิด `http://localhost:4300` แล้วกดปุ่มทดสอบ error พร้อมทำกิจกรรมหลาย ๆ อย่างก่อนกด (คลิกเมนู พิมพ์ค้นหา เลื่อนหน้าจอ) เพื่อให้ replay มีเนื้อหา

**ขั้นที่ 4 - ตรวจผล**

| สิ่งที่ต้องตรวจ | ตรวจที่ไหน | ผลที่คาดหวัง |
| --- | --- | --- |
| Stack trace อ่านได้ | Issue detail | เห็นชื่อไฟล์ `.ts` และบรรทัดจริง |
| Release ปรากฏ | Releases | เห็น `bcel-crm-frontend@1.2.0` |
| Crash Free Rate | Releases | มีตัวเลข |
| Suspect Commits | Issue detail | เห็นชื่อ commit และผู้เขียน |
| Replay บันทึกได้ | Issue → Replays | เล่นกลับได้ |
| **ข้อมูลถูกปกปิด** | Replay player | ชื่อและเลขบัญชีเป็น `***` ทั้งหมด ⭐ |

**ขั้นที่ 5 - ตรวจสอบด้าน Compliance**

ดู Replay ทั้งคลิปแล้วบันทึกลงตาราง (นี่คือเอกสารที่ฝ่าย Compliance ต้องการ)

| ข้อมูล | เห็นหรือไม่ | ต้องแก้ไข |
| --- | --- | --- |
| ชื่อลูกค้า | | |
| เลขบัญชี | | |
| ยอดเงิน | | |
| เบอร์โทร | | |
| ข้อความใน input | | |
| รูปภาพ/เอกสารแนบ | | |

---

## 📚 Module 3.6: ผนวก Monitoring เข้ากับ CI/CD (Jenkins) และ Kubernetes

### เวลา 14:15–15:00 น.

> 💡 **หัวใจของ Module นี้:** ถ้าการสร้าง Release และอัปโหลด Source Map ต้องทำด้วยมือ สุดท้ายจะไม่มีใครทำ ระบบ Monitoring จะค่อย ๆ เสื่อมจนไร้ประโยชน์ **การทำให้อัตโนมัติคือสิ่งเดียวที่ทำให้ระบบนี้อยู่รอดในระยะยาว**

---

### 3.6.1 เตรียม Credentials ใน Jenkins

**สร้าง Auth Token ใน Sentry:**

Settings → Auth Tokens → Create New Token พร้อม scope ต่อไปนี้:

| Scope | ใช้ทำอะไร |
| --- | --- |
| `project:releases` | สร้าง release, ผูก commit, แจ้ง deploy |
| `project:read` | อ่านข้อมูลโปรเจกต์ |
| `org:read` | อ่านข้อมูลองค์กร |

**เพิ่มเข้า Jenkins Credentials:**

```
Jenkins → Manage Jenkins → Credentials → System → Global credentials → Add

  ID: sentry-auth-token-01    Kind: Secret text   Secret: <token>     # เติมเลขของท่าน
  ID: sentry-dsn-backend      Kind: Secret text   Secret: <DSN ของ bcel-crm-backend>
  ID: sentry-dsn-frontend     Kind: Secret text   Secret: <DSN ของ bcel-crm-frontend>

  # เฉพาะกรณีจะ deploy ลง Kubernetes จริง (stage "Deploy" ใน pipeline)
  ID: rke2-kubeconfig         Kind: Secret file   File:   kubeconfig.yaml
```

> 🖥️ **ในห้อง Lab** Jenkins อยู่ที่ `https://jenkins.lab.itgenius.co.th` และเป็นเครื่องเดียวที่ทุกท่านใช้ร่วมกัน จึงให้ตั้งชื่อ credential และชื่อ Job เติมเลขประจำตัวต่อท้าย เช่น `sentry-auth-token-01`, `bcel-crm-lite-deploy-01` เพื่อไม่ให้ชนกัน

> ⛔ **ห้ามเด็ดขาด:** เขียน token ลงใน `Jenkinsfile` ตรง ๆ เพราะ `Jenkinsfile` อยู่ใน Git ที่ทุกคนในทีมอ่านได้

### 3.6.2 Jenkins Pipeline ฉบับสมบูรณ์

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        // ที่ BCEL (Self-hosted) ต้องตั้งบรรทัดนี้ · ในห้อง Lab (SaaS) ให้ลบออก
        SENTRY_URL        = 'https://sentry.bcel.local'
        SENTRY_ORG        = 'bcel'
        SENTRY_AUTH_TOKEN = credentials('sentry-auth-token-01')   // เติมเลขของท่าน

        // สร้างชื่อ release จาก build number + git sha (ไม่ซ้ำแน่นอน)
        APP_VERSION       = "1.${BUILD_NUMBER}.0"
        GIT_SHA_SHORT     = "${GIT_COMMIT.take(7)}"
        RELEASE_BACKEND   = "bcel-crm-backend@${APP_VERSION}+${GIT_SHA_SHORT}"
        RELEASE_FRONTEND  = "bcel-crm-frontend@${APP_VERSION}+${GIT_SHA_SHORT}"
        DEPLOY_ENV        = 'production'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    echo "Backend release : ${RELEASE_BACKEND}"
                    echo "Frontend release: ${RELEASE_FRONTEND}"
                }
            }
        }

        stage('ติดตั้ง sentry-cli') {
            steps {
                sh '''
                    if ! command -v sentry-cli > /dev/null 2>&1; then
                        npm install -g @sentry/cli
                    fi
                    sentry-cli --version
                    sentry-cli info
                '''
            }
        }

        // ---------- BACKEND ----------
        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn -B clean package -DskipTests=false'
                }
            }
            post {
                always {
                    junit 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('สร้าง Release ฝั่ง Backend') {
            environment { SENTRY_PROJECT = 'bcel-crm-backend' }
            steps {
                sh '''
                    sentry-cli releases new "$RELEASE_BACKEND"
                    sentry-cli releases set-commits "$RELEASE_BACKEND" --auto
                '''
            }
        }

        // ---------- FRONTEND ----------
        stage('Build Frontend') {
            environment {
                SENTRY_PROJECT = 'bcel-crm-frontend'
                SENTRY_DSN     = credentials('sentry-dsn-frontend')
            }
            steps {
                dir('frontend') {
                    sh '''
                        npm ci

                        # แทนที่ placeholder ในไฟล์ environment ด้วยค่าจริง
                        sed -i "s|__SENTRY_DSN__|${SENTRY_DSN}|g"          src/environments/environment.prod.ts
                        sed -i "s|__SENTRY_RELEASE__|${RELEASE_FRONTEND}|g" src/environments/environment.prod.ts

                        npm run build -- --configuration production
                    '''
                }
            }
        }

        stage('อัปโหลด Source Maps') {
            environment { SENTRY_PROJECT = 'bcel-crm-frontend' }
            steps {
                dir('frontend') {
                    sh '''
                        DIST=./dist/bcel-crm-frontend/browser

                        sentry-cli releases new "$RELEASE_FRONTEND"

                        # 1) ฉีด Debug ID
                        sentry-cli sourcemaps inject "$DIST"

                        # 2) ถ้าใช้ Angular Service Worker ต้อง regenerate ngsw.json
                        if [ -f ngsw-config.json ]; then
                            node_modules/.bin/ngsw-config "$DIST" ./ngsw-config.json
                        fi

                        # 3) อัปโหลด
                        sentry-cli sourcemaps upload \
                            --release "$RELEASE_FRONTEND" \
                            "$DIST"

                        # 4) ⭐ ลบ .map ออกจาก artifact ก่อน deploy (ความปลอดภัย)
                        find "$DIST" -name '*.map' -delete
                        echo "ลบ source map ออกจาก artifact เรียบร้อย"

                        sentry-cli releases set-commits "$RELEASE_FRONTEND" --auto
                    '''
                }
            }
        }

        // ---------- DEPLOY ----------
        stage('Deploy to RKE2') {
            steps {
                withCredentials([file(credentialsId: 'rke2-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl -n bcel-crm set image deployment/crm-backend \
                            crm-backend=registry.bcel.local/bcel-crm-backend:${APP_VERSION}

                        kubectl -n bcel-crm set env deployment/crm-backend \
                            SENTRY_RELEASE="${RELEASE_BACKEND}" \
                            SENTRY_ENVIRONMENT="${DEPLOY_ENV}"

                        kubectl -n bcel-crm set image deployment/crm-frontend \
                            crm-frontend=registry.bcel.local/bcel-crm-frontend:${APP_VERSION}

                        kubectl -n bcel-crm rollout status deployment/crm-backend  --timeout=300s
                        kubectl -n bcel-crm rollout status deployment/crm-frontend --timeout=300s
                    '''
                }
            }
        }

        stage('แจ้ง Sentry ว่า Deploy แล้ว') {
            steps {
                sh '''
                    # Backend
                    SENTRY_PROJECT=bcel-crm-backend \
                    sentry-cli releases finalize "$RELEASE_BACKEND"

                    SENTRY_PROJECT=bcel-crm-backend \
                    sentry-cli releases deploys "$RELEASE_BACKEND" new \
                        --env "$DEPLOY_ENV" \
                        --name "jenkins-build-${BUILD_NUMBER}"

                    # Frontend
                    SENTRY_PROJECT=bcel-crm-frontend \
                    sentry-cli releases finalize "$RELEASE_FRONTEND"

                    SENTRY_PROJECT=bcel-crm-frontend \
                    sentry-cli releases deploys "$RELEASE_FRONTEND" new \
                        --env "$DEPLOY_ENV" \
                        --name "jenkins-build-${BUILD_NUMBER}"
                '''
            }
        }

        stage('ตรวจสุขภาพหลัง Deploy') {
            steps {
                sh '''
                    echo "รอ 3 นาทีเพื่อให้ข้อมูลไหลเข้า Sentry..."
                    sleep 180

                    # ตรวจว่ามี issue ใหม่ที่ระดับ fatal หรือไม่
                    # (ในงานจริงควรเรียก Sentry API แล้วตัดสินใจ rollback อัตโนมัติ)
                    # ⚠️ ในห้อง Lab ให้ลบบรรทัดนี้ออก เพราะไม่มีระบบที่ deploy จริง
                    curl -sf https://crm-api.bcel.local/actuator/health || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deploy สำเร็จ: ${RELEASE_BACKEND} และ ${RELEASE_FRONTEND}"
        }
        failure {
            echo "❌ Pipeline ล้มเหลว ตรวจสอบที่ ${BUILD_URL}"
        }
        always {
            cleanWs()
        }
    }
}
```

### 3.6.3 จัดการ Secrets บน Kubernetes (RKE2)

**สร้าง Secret:**

```bash
kubectl -n bcel-crm create secret generic sentry-config \
  --from-literal=SENTRY_DSN='https://<key>@sentry.bcel.local/2' \
  --from-literal=SENTRY_AUTH_TOKEN='<token>'

# ตรวจสอบ (ค่าจะถูกเข้ารหัส base64)
kubectl -n bcel-crm get secret sentry-config -o yaml
```

**ใช้ใน Deployment:**

```yaml
# k8s/crm-backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crm-backend
  namespace: bcel-crm
spec:
  replicas: 3
  selector:
    matchLabels:
      app: crm-backend
  template:
    metadata:
      labels:
        app: crm-backend
    spec:
      containers:
        - name: crm-backend
          image: registry.bcel.local/bcel-crm-backend:1.2.0
          ports:
            - containerPort: 8080
          env:
            # DSN มาจาก Secret ไม่ได้อยู่ใน image
            - name: SENTRY_DSN
              valueFrom:
                secretKeyRef:
                  name: sentry-config
                  key: SENTRY_DSN

            - name: SENTRY_ENVIRONMENT
              value: "production"

            # Jenkins ตั้งค่านี้ผ่าน kubectl set env
            - name: SENTRY_RELEASE
              value: "bcel-crm-backend@1.2.0+a3f5c9d"

            - name: SENTRY_TRACES_RATE
              value: "0.1"

            # ส่งชื่อ pod/node เข้ามาเป็น env ธรรมดา
            # แล้วให้แอปอ่านไปตั้งเป็น Sentry tag เอง (ดูโค้ดใต้ไฟล์นี้)
            # ⚠️ อย่าใช้ชื่อ SENTRY_TAGS_POD_NAME โดยหวังว่าจะได้ tag ชื่อ pod_name
            #    เพราะ relaxed binding ของ Spring Boot จะแปลงเป็น sentry.tags.pod.name
            - name: K8S_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: K8S_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName

          resources:
            requests:
              memory: "1Gi"
              cpu: "500m"
            limits:
              memory: "2Gi"
              cpu: "2000m"

          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 60
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 30
```

**ตั้งค่า tag ในฝั่งแอป:**

```java
// backend/src/main/java/la/com/bcel/crm/config/K8sTagConfig.java
package la.com.bcel.crm.config;

import io.sentry.Sentry;
import jakarta.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class K8sTagConfig {

    @Value("${K8S_POD_NAME:unknown}")
    private String podName;

    @Value("${K8S_NODE_NAME:unknown}")
    private String nodeName;

    @PostConstruct
    public void registerTags() {
        Sentry.configureScope(scope -> {
            scope.setTag("pod_name", podName);
            scope.setTag("node_name", nodeName);
        });
    }
}
```

> 💡 **ประโยชน์ของ tag `pod_name` และ `node_name`:** เมื่อ Error เกิดเฉพาะบาง pod จะเห็นทันทีในแถบ Tags ว่า `pod_name: crm-backend-7f8d-x2k9` มีสัดส่วน 98% ซึ่งชี้ว่าเป็นปัญหาระดับ infrastructure (เช่น node นั้นมีปัญหา disk หรือ network) ไม่ใช่บั๊กของโค้ด

**แนวทางที่ปลอดภัยยิ่งขึ้น (แนะนำสำหรับ Production ของธนาคาร):**

| ระดับ | วิธี | ข้อดี |
| --- | --- | --- |
| พื้นฐาน | Kubernetes Secret | ง่าย แต่ base64 ไม่ใช่การเข้ารหัส |
| กลาง | Sealed Secrets / SOPS | เก็บใน Git ได้อย่างปลอดภัย |
| สูง | HashiCorp Vault + External Secrets Operator | หมุนเวียน secret อัตโนมัติ มี audit log |

### 3.6.4 Deploy Sentry เองบน RKE2 (ทางเลือก)

```bash
helm repo add sentry https://sentry-kubernetes.github.io/charts
helm repo update

# สร้าง values ที่เหมาะกับองค์กร
cat > sentry-values.yaml <<'YAML'
user:
  email: admin@bcel.com.la
  create: true

sentry:
  web:
    replicas: 2
    resources:
      requests: { memory: 2Gi, cpu: 500m }
  worker:
    replicas: 3

ingress:
  enabled: true
  hostname: sentry.bcel.local
  ingressClassName: nginx
  tls: true

postgresql:
  enabled: true
  persistence:
    size: 100Gi
clickhouse:
  persistence:
    size: 200Gi
kafka:
  persistence:
    size: 100Gi

# ปิดการติดต่อภายนอกทั้งหมด
config:
  configYml:
    beacon.record-only-if-internet-connection: false
YAML

helm install sentry sentry/sentry \
  --namespace sentry --create-namespace \
  -f sentry-values.yaml \
  --timeout 30m

kubectl -n sentry get pods -w
```

> ⚠️ **ข้อควรพิจารณาก่อนเลือกทางนี้:** Sentry บน Kubernetes ประกอบด้วย pod กว่า 30 ตัว ต้องการ persistent storage หลายร้อย GB และการดูแลที่ซับซ้อนกว่า Docker Compose มาก แนะนำให้เลือกทางนี้ **ก็ต่อเมื่อ** องค์กรมีทีม Platform ที่ดูแล Kubernetes เต็มเวลาอยู่แล้ว

### 3.6.5 เสริม Infrastructure Monitoring ด้วย Prometheus + Grafana

```
        ┌──────────────────────────────────────────────────┐
        │              ชั้นแอปพลิเคชัน                       │
        │  Sentry: Errors, Traces, Release Health, Replay  │
        │  ตอบ: "โค้ดบรรทัดไหนพัง / query ไหนช้า"           │
        └──────────────────────────────────────────────────┘
                              ▲
                              │  เชื่อมด้วย trace_id / timestamp
                              ▼
        ┌──────────────────────────────────────────────────┐
        │            ชั้นโครงสร้างพื้นฐาน                     │
        │  Prometheus + Grafana                            │
        │  ตอบ: "Node ไหน CPU เต็ม / pod ไหน OOMKilled"     │
        │  - node-exporter    (CPU, RAM, Disk, Network)    │
        │  - kube-state-metrics (สถานะ pod/deployment)      │
        │  - mysqld-exporter  (MariaDB: connections, locks)│
        │  - Spring Actuator  (JVM heap, GC, thread, pool) │
        └──────────────────────────────────────────────────┘
```

**เปิด metrics endpoint ของ Spring Boot:**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=health,info,prometheus,metrics
management.endpoint.health.probes.enabled=true
management.metrics.tags.application=bcel-crm-backend
management.metrics.tags.environment=${SENTRY_ENVIRONMENT:development}
```

> ✅ **ข้อสรุปเรื่องเครื่องมือ:** Sentry กับ Prometheus **ไม่ได้แข่งกัน แต่เสริมกัน** เมื่อ Sentry แจ้งว่า "query ช้าตอน 14:05" ให้ไปดู Grafana ว่า "ตอน 14:05 CPU ของ MariaDB เป็นอย่างไร" คำตอบที่สมบูรณ์มักต้องใช้ทั้งสองมุมมอง

---

## 🛠️ Workshop 3.1 - เพิ่มขั้นตอน Sentry ลงใน Jenkins Pipeline

### เวลา (รวมอยู่ในช่วง 14:15–15:00 น.)

> **โจทย์:** สร้าง Jenkins Pipeline ที่ทุกครั้งที่ deploy จะสร้าง Release, ผูก commit, อัปโหลด Source Maps และแจ้ง Deploy ไปยัง Sentry โดยอัตโนมัติ

### ขั้นที่ 1 - เตรียม Credentials

เพิ่ม credential 3 ตัวตามหัวข้อ 3.6.1

### ขั้นที่ 2 - สร้าง Pipeline Job

Jenkins → New Item → Pipeline → ชื่อ `bcel-crm-lite-deploy-01` (เติมเลขประจำตัวของท่าน เพราะใช้ Jenkins เครื่องเดียวกันทั้งห้อง)

วาง `Jenkinsfile` จากหัวข้อ 3.6.2 โดยปรับค่าต่อไปนี้ให้ตรงกับห้อง Lab

```groovy
environment {
    // ไม่ต้องตั้ง SENTRY_URL เพราะ sentry-cli ชี้ไป sentry.io เป็นค่าเริ่มต้น
    SENTRY_ORG        = '<org slug ของท่าน>'   // ดูได้จาก URL หรือ Settings
    SENTRY_AUTH_TOKEN = credentials('sentry-auth-token-01')   // เติมเลขของท่าน
    RELEASE_BACKEND   = "bcel-crm-backend@${APP_VERSION}+${GIT_SHA_SHORT}"
    RELEASE_FRONTEND  = "bcel-crm-frontend@${APP_VERSION}+${GIT_SHA_SHORT}"
}
```

**เรื่อง stage "Deploy to RKE2"** ห้อง Lab ใช้ **k3d cluster** ที่ชื่อ `bcel-lab` แทน RKE2 ซึ่งใช้ `kubectl` และ manifest ชุดเดียวกันทุกประการ ให้เลือกทำอย่างใดอย่างหนึ่ง

| ทางเลือก | วิธีทำ | เหมาะกับ |
| --- | --- | --- |
| **A. Deploy จริงบน k3d** (แนะนำ) | ใช้ kubeconfig ที่วิทยากรแจก แล้วแก้ stage เป็น `kubectl -n bcel-crm set image ...` | ผู้เรียนที่ทำเสร็จเร็ว |
| **B. แทนที่ด้วย echo** | เปลี่ยน stage Deploy เป็น `sh 'echo "จำลองการ deploy"'` | ถ้าเวลาไม่พอ |

> 💡 **สาระสำคัญของ Workshop นี้อยู่ที่ stage "สร้าง Release", "อัปโหลด Source Maps" และ "แจ้ง Sentry ว่า Deploy แล้ว"** ไม่ใช่ตัวการ deploy เอง ดังนั้นทางเลือก B ก็ยังบรรลุวัตถุประสงค์การเรียนรู้ครบถ้วน

### ขั้นที่ 3 - รัน Pipeline

```
Build Now → ดู Console Output
```

**ตรวจว่าแต่ละ stage สำเร็จ:**

| Stage | สิ่งที่ต้องเห็นใน log |
| --- | --- |
| ติดตั้ง sentry-cli | `sentry-cli 3.x.x` |
| สร้าง Release Backend | `Created release bcel-crm-backend@1.x.0+xxxxxxx` |
| Build Frontend | build สำเร็จ |
| อัปโหลด Source Maps | `Uploaded N files to Sentry` |
| แจ้ง Deploy | `Created new deploy for production` |

### ขั้นที่ 4 - ตรวจผลใน Sentry

| สิ่งที่ต้องเห็น | ที่ไหน |
| --- | --- |
| Release ใหม่ 2 ตัว | Releases |
| จำนวน commit ที่ผูก | Release detail → Commits |
| Artifact Bundle | Project Settings → Source Maps |
| Deploy record | Release detail → Deploys |

### ขั้นที่ 5 - พิสูจน์ว่า Source Maps ทำงาน

1. เปิดแอปที่ deploy แล้ว (production build)
2. กดปุ่มที่ทำให้เกิด error
3. เปิด Issue ใน Sentry แล้วตรวจว่า Stack Trace แสดงชื่อไฟล์ `.ts` และบรรทัดจริง

> ✅ **เกณฑ์ผ่าน Workshop 3.1:**
> - [ ] Pipeline รันผ่านทุก stage
> - [ ] Release ปรากฏใน Sentry พร้อม commit
> - [ ] Stack Trace ของ production build อ่านได้
> - [ ] ไม่มีไฟล์ `.map` เหลืออยู่ใน artifact ที่ deploy
> - [ ] ไม่มี token หรือ DSN โผล่ใน Console Output

---

## 🎓 Module 3.7: Capstone Project - จำลอง Incident และวินิจฉัย

### เวลา 15:00–15:45 น. (ลงมือ 30 นาที + นำเสนอ 15 นาที)

> **โจทย์ใหญ่:** คุณคือทีมที่รับผิดชอบระบบ BCEL CRM Lite ซึ่งกำลังจะขึ้นใช้งานจริง ผู้บริหารต้องการความมั่นใจว่า **"ถ้าระบบมีปัญหา เราจะรู้ก่อนผู้ใช้ และรู้ว่าต้องแก้ที่ไหน"** ให้วางระบบ Monitoring ให้ครบวงจร แล้วนำเสนอผลการวินิจฉัยสถานการณ์ Incident ที่วิทยากรจำลองขึ้น

### รูปแบบการทำงาน

- แบ่งเป็น 2 กลุ่ม (กลุ่มละ 2–3 คน)
- **ลงมือทำ 30 นาที (15:00–15:30) แล้วนำเสนอกลุ่มละ 7 นาที (15:30–15:45)**
- จากนั้นต่อด้วย Module 3.8 ซึ่งเป็นเนื้อหาปิดท้ายหลักสูตร

> ⏱️ **เวลาจำกัด ให้เน้นที่ส่วนที่ 2 (วินิจฉัย Incident) เป็นหลัก** ส่วนที่ 1 ใช้เป็น checklist ตรวจเร็ว ๆ ไม่เกิน 5 นาที

---

### ส่วนที่ 1 - ตรวจสอบความครบถ้วนของการติดตั้ง (5 นาที)

| # | รายการ | Backend | Frontend | หมายเหตุ |
| --- | --- | --- | --- | --- |
| 1 | SDK ติดตั้งและส่ง event ได้ | ☐ | ☐ | |
| 2 | `environment` ถูกต้อง | ☐ | ☐ | |
| 3 | `release` มีรูปแบบ `<project>@<version>+<sha>` | ☐ | ☐ | |
| 4 | Performance Monitoring เปิด | ☐ | ☐ | |
| 5 | Distributed Tracing เชื่อมกัน | ☐ | ☐ | ต้องมี trace_id เดียวกัน |
| 6 | Database spans ปรากฏ | ☐ | - | ผ่าน sentry-jdbc |
| 7 | User Context ผูกแล้ว (ไม่ใช่ PII) | ☐ | ☐ | |
| 8 | Tags ทางธุรกิจครบ | ☐ | ☐ | branch_code, module, segment |
| 9 | Data Scrubbing ทำงาน | ☐ | ☐ | ทดสอบด้วยเลขบัญชี |
| 10 | Source Maps อัปโหลดแล้ว | - | ☐ | |
| 11 | Session Replay มี masking | - | ☐ | |
| 12 | Release Health มีข้อมูล | ☐ | ☐ | |
| 13 | Alerts ตั้งครบตามเมทริกซ์ | ☐ | ☐ | |
| 14 | Dashboard พร้อมใช้ | ☐ | ☐ | |
| 15 | Ownership Rules ตั้งแล้ว | ☐ | ☐ | |

---

### ส่วนที่ 2 - จำลองสถานการณ์ Incident (25 นาที)

วิทยากรจะเปิดใช้งานปัญหาผ่าน `/api/_debug/chaos/enable?scenario=X` โดยแต่ละกลุ่มได้คนละสถานการณ์ (ไม่บอกว่าเป็นอันไหน)

| Scenario | สิ่งที่เกิดขึ้นจริง | เบาะแสที่ควรพบ |
| --- | --- | --- |
| **A** | ลบ index ของ `transaction_log` ทำให้รายงานช้ามาก | span `db.query` เดียวยาว 8+ วินาที · Performance Issue "Slow DB Query" |
| **B** | เปิดโค้ดที่มี N+1 กลับมา | span `db.query` ซ้ำ 15+ ครั้ง · Performance Issue "N+1 Query" |
| **C** | Core Banking client timeout เป็นบางครั้ง | span `http.client` ยาว · failure_rate พุ่ง · error เป็นช่วง ๆ |
| **D** | Deploy release ใหม่ที่มีบั๊ก NPE | Crash Free Rate ตก · Issue ใหม่ที่มี `firstRelease` = release ใหม่ · Suspect Commit ชี้ commit ล่าสุด |
| **E** | Connection pool ตั้งเล็กเกินไป | มี gap ก่อน span db แรก · error `HikariPool timeout` เป็นช่วง ๆ |

**สิ่งที่แต่ละกลุ่มต้องทำ:**

1. ตรวจพบปัญหาผ่าน **Alert หรือ Dashboard** (ไม่ใช่จากการบอกของวิทยากร)
2. ระบุขอบเขตผลกระทบ: กี่ผู้ใช้ กี่สาขา endpoint ใดบ้าง ช่วงเวลาไหน
3. หา **trace_id** ที่เป็นตัวอย่างของปัญหา
4. ระบุ **ชั้นที่เป็นคอขวด** พร้อมหลักฐานเป็นตัวเลข
5. ถ้าเกี่ยวกับ release ให้ระบุ **Suspect Commit**
6. เสนอ **วิธีแก้ไขเฉพาะหน้า** และ **วิธีป้องกันระยะยาว**

---

### ส่วนที่ 3 - นำเสนอ (15:30-15:45 กลุ่มละ 7 นาที)

**รูปแบบรายงาน Incident (ใช้เป็นแม่แบบในงานจริงได้):**

```markdown
# รายงาน Incident: [ชื่อสั้น ๆ]

## 1. สรุปผู้บริหาร (Executive Summary)
- เกิดอะไรขึ้น: ................................................
- ผลกระทบ: ผู้ใช้ ....... คน · ....... สาขา · ระยะเวลา ....... นาที
- สถานะปัจจุบัน: ☐ กำลังตรวจสอบ ☐ ระบุสาเหตุแล้ว ☐ แก้ไขแล้ว

## 2. ไทม์ไลน์
| เวลา | เหตุการณ์ | แหล่งข้อมูล |
| --- | --- | --- |
| 14:02 | เริ่มมี error | Sentry Issue #.... |
| 14:05 | Alert ดัง | Metric Alert "...." |
| 14:12 | ระบุสาเหตุ | Trace .... |
| 14:30 | แก้ไขเสร็จ | Release .... |

## 3. หลักฐานจาก Sentry
- Issue: ........................ (events: ...., users: ....)
- Trace ID: ....................
- Transaction ที่กระทบ: ........................
- P95 ก่อน / หลัง: ....... ms / ....... ms
- Span ที่เป็นคอขวด: ........................ (....... ms, ....... ครั้ง)
- Suspect Commit: ........................ โดย ........................

## 4. สาเหตุที่แท้จริง (Root Cause)
................................................................

## 5. การแก้ไข
- เฉพาะหน้า: ................................................
- ระยะยาว: ................................................

## 6. บทเรียนและการป้องกัน
- ทำไมเราไม่รู้เร็วกว่านี้: ................................
- Alert ที่ควรเพิ่ม: ........................................
- การทดสอบที่ควรเพิ่มใน CI: ................................
```

**เกณฑ์การประเมิน:**

| หัวข้อ | คะแนน | สิ่งที่ดู |
| --- | --- | --- |
| ตรวจพบด้วยเครื่องมือเอง | 20 | ใช้ Alert/Dashboard ไม่ใช่เดา |
| ระบุขอบเขตผลกระทบเป็นตัวเลข | 20 | มีจำนวนผู้ใช้ สาขา เวลา |
| หาสาเหตุถูกต้องพร้อมหลักฐาน | 30 | อ้าง trace, span, ตัวเลขจริง |
| วิธีแก้เหมาะสมและทำได้จริง | 20 | ทั้งเฉพาะหน้าและระยะยาว |
| การนำเสนอชัดเจน | 10 | ผู้บริหารฟังแล้วเข้าใจ |

---

### ส่วนที่ 4 - แนวทางต่อยอดสำหรับ BCEL (อ่านเพิ่มเติมหลังจบคอร์ส)

**แผน 90 วันแรกหลังจบหลักสูตร:**

| ช่วง | สิ่งที่ควรทำ | ผลลัพธ์ที่คาดหวัง |
| --- | --- | --- |
| **สัปดาห์ 1–2** | ติดตั้ง Sentry บน VM production, ตั้ง TLS + backup อัตโนมัติ, กำหนดนโยบาย PII เป็นเอกสาร | มี instance ที่ใช้งานได้จริง ผ่าน Compliance |
| **สัปดาห์ 3–4** | Instrument ระบบ CRM ตัวจริง (dev → staging), ตั้ง Environment และ Release | เห็น Error จากระบบจริง |
| **เดือนที่ 2** | เปิด Performance + JDBC, แก้ N+1 และ Slow Query ที่พบ, ตั้ง Alert ชุดแรก | P95 ของ endpoint หลักดีขึ้นวัดผลได้ |
| **เดือนที่ 2–3** | ผนวก Jenkins Pipeline, Source Maps อัตโนมัติ, Release Health | ทุก deploy มี release และวัดคุณภาพได้ |
| **เดือนที่ 3** | ขยายไปยังระบบ ERP, เปิด Session Replay (หลังผ่าน Compliance), เสริม Prometheus + Grafana | Observability ครอบคลุมทั้ง 2 ระบบ |

**แนวปฏิบัติที่ควรกำหนดเป็นมาตรฐานองค์กร:**

| หัวข้อ | มาตรฐานที่แนะนำ |
| --- | --- |
| ชื่อ Project | `<org>-<system>-<layer>` เช่น `bcel-crm-backend` |
| ชื่อ Environment | `development` / `staging` / `production` เท่านั้น |
| ชื่อ Release | `<project>@<semver>+<git-sha-7>` |
| Sampling ใน prod | traces 5–10% (ใช้ dynamic sampling สำหรับ endpoint สำคัญ) |
| การเก็บข้อมูล | 30–90 วัน ตามนโยบายองค์กร |
| ทบทวน Alert | ทุกเดือนในการประชุมทีม |
| ทบทวน Issue backlog | ทุกสัปดาห์ (triage meeting) |
| อัปเกรด Sentry | ทุกไตรมาส (ห้ามปล่อยเกิน 1 ปี) |
| Backup | ทุกวัน เก็บ 30 วัน ทดสอบ restore ทุกไตรมาส |

**สิ่งที่ Sentry ทำไม่ได้ และควรเสริม:**

| ความต้องการ | เครื่องมือที่แนะนำ |
| --- | --- |
| CPU / RAM / Disk ของ Node | Prometheus + node-exporter + Grafana |
| สถานะ Pod / Deployment บน RKE2 | kube-state-metrics + Grafana |
| Metrics ของ MariaDB (locks, connections) | mysqld-exporter + Grafana |
| รวมศูนย์ log ทั้งหมด | Loki หรือ ELK Stack |
| Synthetic / Uptime monitoring | Blackbox exporter หรือ Uptime Kuma |
| Network flow | องค์กรมีอยู่แล้ว |

---

## 🏁 Module 3.8: ปิดท้ายหลักสูตร - ติดตั้ง Sentry Self-hosted ด้วยตนเอง

### เวลา 15:45–16:30 น.

> 💡 **หัวใจของ Module นี้:** ตลอด 3 วันที่ผ่านมาเราใช้ Sentry บน SaaS เพื่อเรียนรู้การใช้งานให้ครบทุกด้าน ช่วงสุดท้ายนี้เราจะ **ลงมือติดตั้ง Sentry Self-hosted ด้วยมือของท่านเอง** ซึ่งเป็นสิ่งที่จะนำกลับไปทำจริงที่ BCEL เพราะข้อมูลของธนาคารออกนอกองค์กรไม่ได้

> 🖥️ **ท่านได้เซิร์ฟเวอร์ของตัวเอง 1 เครื่อง**
>
> | รายการ | รายละเอียด |
> | --- | --- |
> | Hostname | `install-lab-XX.itgenius.co.th` (XX = หมายเลขประจำตัวของท่าน) |
> | สเปก | 8 vCPU / 32 GB RAM / Ubuntu 22.04 LTS + swap 16 GB |
> | บัญชี | `lab-user` / รหัสผ่าน `LabUser2026!` |
> | สถานะ | ติดตั้ง Docker และดึง image ไว้ครบแล้ว **แต่ยังไม่ได้รัน `install.sh`** |
>
> ⏳ **เซิร์ฟเวอร์นี้จะถูกลบทิ้งหลังจบการอบรม** ให้ใช้ให้เต็มที่ในช่วงนี้

---

### 3.8.1 ทบทวนก่อนลงมือ

ย้อนกลับไปดู **Module 1.3** ที่เราเรียนไว้ในวันแรก เนื้อหาทั้งหมดกำลังจะถูกใช้จริงในอีกไม่กี่นาที

| หัวข้อจาก Module 1.3 (ตัดสินใจไว้วันแรก) | จะได้ลงมือตรงไหนใน Lab นี้ |
| --- | --- |
| 1.3.1 ข้อกำหนดของระบบ | **ขั้นที่ 1** ตรวจว่าเครื่องที่ได้ตรงกับที่วางแผนไว้ (8 vCPU / 32 GB) |
| 1.3.2 ข้อควรระวังบน CentOS 8 | เครื่องที่ได้เป็น Ubuntu 22.04 ตามที่ตัดสินใจไว้ |
| 1.3.3 นโยบายอัปเกรดและสำรองข้อมูล | **ขั้นที่ 6** รันคำสั่ง `sentry export` และขั้นตอนอัปเกรดจริง |
| 1.3.4 Reverse Proxy + TLS | **ขั้นที่ 7** รายการที่ต้องทำเพิ่มก่อนใช้จริงที่ธนาคาร |
| 1.3.5 การบริหารผู้ใช้และสิทธิ์ | **ขั้นที่ 4** ตั้ง 2FA, ปิด Open Membership, กำหนด Owner |
| **Lab 1.2 แบบฟอร์มขอทรัพยากร** | ใช้เป็นเช็กลิสต์ตลอด Lab นี้ ว่าทำได้ตามที่วางแผนไว้ไหม |

**คำถามทบทวน 3 ข้อ (ตอบก่อนเริ่ม)**

1. ทำไม Sentry ถึงต้องการ swap 16 GB ทั้งที่มี RAM 32 GB อยู่แล้ว
2. `iowait` เกินกี่เปอร์เซ็นต์ถือว่าเครื่องรับโหลดไม่ไหว
3. Sentry รองรับการอัปเกรดข้ามได้ไกลสุดกี่ปี

---

### ขั้นที่ 1 - เชื่อมต่อเซิร์ฟเวอร์ของท่าน

```bash
ssh lab-user@install-lab-XX.itgenius.co.th
# รหัสผ่าน: LabUser2026!
```

ตรวจว่าเครื่องพร้อมและ image ถูกดึงไว้แล้วจริง

```bash
nproc                      # ควรได้ 8
free -h                    # RAM 32 GB + swap 16 GB
df -h /                    # ว่างอย่างน้อย 20 GB
docker images | wc -l      # ควรเห็นหลายสิบรายการ (pre-pull ไว้แล้ว)
cat /etc/os-release | head -2
```

---

### ขั้นที่ 2 - รันสคริปต์ติดตั้ง

```bash
cd ~/self-hosted
git describe --tags        # ดูว่าอยู่ที่ tag เวอร์ชันไหน

./install.sh --no-report-self-hosted-issues
```

> ⚠️ **`--no-report-self-hosted-issues` คือสิ่งที่ต้องใช้เสมอที่ BCEL** เพราะเป็นการปฏิเสธไม่ให้ Sentry ส่งข้อมูล error ของตัว instance กลับไปยังทีม Sentry ซึ่งขัดกับนโยบายธนาคาร

**สิ่งที่จะเห็นระหว่างรอ 8-12 นาที** ให้สังเกตและจับคู่กับสถาปัตยกรรมที่เรียนไว้

| ข้อความใน log | กำลังทำอะไร | ส่วนประกอบที่เกี่ยวข้อง |
| --- | --- | --- |
| `Checking minimum requirements` | ตรวจ CPU / RAM / Disk | - |
| `Creating volumes for persistent data` | สร้างที่เก็บข้อมูลถาวร | ทุกฐานข้อมูล |
| `Ensuring proper PostgreSQL version` | เตรียม metadata database | PostgreSQL |
| `Bootstrapping and migrating Snuba` | เตรียมที่เก็บ event | ClickHouse |
| `Creating additional Kafka topics` | เตรียมคิวกลาง | Kafka |
| `Setting up / migrating database` | รัน migration ของ Sentry | PostgreSQL |
| `Would you like to create a user account now?` | สร้าง Superuser | - |

> ⏱️ **ระหว่างรอ เราจะทำ Post-test และสรุปปิดหลักสูตรไปพร้อมกัน** (ดูขั้นที่ 7) ไม่ต้องนั่งจ้องจอเปล่า ๆ

**สร้างบัญชี Superuser เมื่อระบบถาม**

```
Email    : <อีเมลของท่าน>
Password : <กำหนดเอง จดไว้>
```

---

### ขั้นที่ 3 - เริ่มระบบและตรวจสุขภาพ

```bash
docker compose up --wait
```

```bash
# ทุกบริการควรเป็น running / healthy
docker compose ps --format 'table {{.Service}}\t{{.Status}}'

# health endpoint ต้องตอบ 200
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:9000/_health/

# ดูว่ากิน RAM ไปเท่าไร (จะเห็นว่าใกล้ 16 GB จริงตามที่บอกไว้วันแรก)
free -h
docker stats --no-stream | head -15
```

> ⛔ **ถ้ามีบริการไหนเป็น `Exited`** ให้ไล่ตรวจตามลำดับ
>
> ```bash
> docker compose logs clickhouse --tail=50
> docker compose logs kafka --tail=50
> free -h && df -h /
> ```
>
> สาเหตุอันดับหนึ่งคือ RAM หรือ disk ไม่พอ ซึ่งบนเครื่อง 32 GB ไม่ควรเกิด

---

### ขั้นที่ 4 - เข้าใช้งานครั้งแรกและตั้งค่าตามนโยบายธนาคาร

เปิดเบราว์เซอร์ไปที่ `http://install-lab-XX.itgenius.co.th:9000` แล้วล็อกอินด้วยบัญชีที่เพิ่งสร้าง

**หน้า Welcome** ให้กรอก

- **Root URL** = `http://install-lab-XX.itgenius.co.th:9000`
- ยกเลิกเครื่องหมายถูกที่ **"Allow anonymous usage statistics"**

**ปิดการติดต่อภายนอกทั้งหมด** (นโยบายธนาคาร)

```bash
cd ~/self-hosted
cat >> sentry/sentry.conf.py <<'PY'

# ---- นโยบายความปลอดภัยของ BCEL ----
SENTRY_BEACON = False      # ไม่ติดต่อ beacon server ของ Sentry
SENTRY_AIR_GAP = True      # ไม่เรียก network ออกนอกองค์กรเลย
PY
```

**ปิดการสมัครสมาชิกเอง**

```yaml
# แก้ sentry/config.yml
auth.allow-registration: false
```

**ลดระยะเก็บ event** (ค่านี้อยู่ใน `.env` ไม่ใช่ `sentry.conf.py`)

```bash
cd ~/self-hosted
sed -i 's/^SENTRY_EVENT_RETENTION_DAYS=.*/SENTRY_EVENT_RETENTION_DAYS=30/' .env
grep SENTRY_EVENT_RETENTION_DAYS .env

docker compose restart web worker
```

**ตั้งค่าความปลอดภัยระดับองค์กร** ไปที่ Settings → Security & Privacy แล้วเปิด

- [ ] Require Two-Factor Authentication
- [ ] Require Data Scrubber
- [ ] Require Using Default Scrubbers
- [ ] Prevent Storing of IP Addresses
- [ ] ปิด Open Membership

---

### ขั้นที่ 5 - ส่ง Error ตัวแรกเข้า instance ของตัวเอง

สร้าง Project แล้วนำ DSN มาใช้กับแอปที่รันอยู่บนโน้ตบุ๊กของท่าน

```
Projects -> Create Project -> Platform: Spring Boot -> ชื่อ bcel-crm-backend
คัดลอก DSN จาก Project Settings -> Client Keys
```

```bash
# บนโน้ตบุ๊กของท่าน (หยุดแอปเดิมก่อนแล้วรันใหม่ด้วย DSN ใหม่)
cd bcel-crm-lite/backend

export SENTRY_DSN='http://<public-key>@install-lab-XX.itgenius.co.th:9000/2'
export SENTRY_ENVIRONMENT=development
export SENTRY_RELEASE='bcel-crm-backend@1.0.0'
export DB_USER='crm_app' DB_PASSWORD='labpass123'

mvn spring-boot:run
```

```bash
curl -i http://localhost:8080/api/_debug/sentry/boom
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** เห็น Issue โผล่ใน instance ของท่านเองภายใน 10-30 วินาที
>
> 🎉 **ถึงจุดนี้ท่านมี Sentry ของตัวเองที่ทำงานครบวงจรแล้ว** ตั้งแต่ SDK ในแอป ผ่าน Relay เข้า Kafka ประมวลผลลง ClickHouse และแสดงผลบนหน้าเว็บ

---

### ขั้นที่ 6 - คำสั่งดูแลระบบที่ต้องใช้จริงที่ธนาคาร

ลองรันดูทีละคำสั่ง เพื่อให้คุ้นมือก่อนกลับไปใช้งานจริง

```bash
# --- ดูสถานะและ log ---
docker compose ps
docker compose logs -f relay             # ดูว่า event เข้าถึงระบบไหม
docker compose logs -f ingest-consumer   # ดูว่า event ถูกประมวลผลไหม
docker stats

# --- สำรองข้อมูล (ทำก่อนอัปเกรดทุกครั้ง) ---
docker compose run --rm -T web sentry export > ~/sentry-backup-$(date +%F).json
ls -lh ~/sentry-backup-*.json

# --- จัดการผู้ใช้ ---
docker compose run --rm web sentry createuser

# --- ตรวจ Disk I/O (ตัวชี้วัดสำคัญที่สุด) ---
sudo apt install -y sysstat
iostat -x 5 3          # %iowait ต้องต่ำกว่า 10
```

**ขั้นตอนอัปเกรดที่จะใช้ทุกไตรมาส**

```bash
cd ~/self-hosted
docker compose run --rm -T web sentry export > ~/backup-before-upgrade.json   # ⭐ ห้ามข้าม
git fetch --tags
git checkout <เวอร์ชันใหม่>
./install.sh --no-report-self-hosted-issues
docker compose up --wait
```

> ⚠️ **กฎเหล็ก:** Sentry รองรับการอัปเกรดข้ามได้ไม่เกิน **1 ปี** ถ้าปล่อยให้เก่ากว่านั้นต้องอัปเกรดเป็นทอด ๆ ผ่าน hard stop versions ให้ตั้งรอบอัปเกรดอย่างน้อยทุกไตรมาส

---

### ขั้นที่ 7 - สิ่งที่ยังต้องทำเพิ่มเมื่อนำไปใช้จริงที่ BCEL

ห้อง Lab นี้จบแค่ `http://...:9000` ซึ่งใช้จริงไม่ได้ ต่อไปนี้คือรายการที่ต้องทำเพิ่ม (รายละเอียดอยู่ใน Module 1.3 หัวข้อ 1.3.4 และคู่มือ `Lab_Setup_Guide.md`)

| สิ่งที่ต้องทำ | ทำไม | อ้างอิง |
| --- | --- | --- |
| วาง **Nginx + TLS** ไว้หน้า | ห้ามเปิดพอร์ต 9000 ตรง ๆ | Module 1.3.4 |
| **import CA ภายในเข้า JVM truststore** | ไม่งั้น Spring Boot จะขึ้น `PKIX path building failed` | Module 1.5.7 |
| ตั้ง `system.url-prefix` ให้ตรง URL จริง | ไม่งั้นเจอ CSRF error | Module 1.3.4 |
| ตั้ง **SMTP ขององค์กร** | Self-hosted ไม่มีบริการส่งอีเมลในตัว | Module 3.1.4 |
| ตั้ง **cron สำรองข้อมูลรายวัน** | ไม่มีใครสำรองให้ | Module 1.3.3 |
| วางแผน **monitoring ตัว Sentry เอง** | Prometheus + Grafana ดูแล CPU/RAM/Disk | Module 3.6.5 |
| กำหนด **Owner 1-2 คน + บังคับ 2FA** | ข้อกำหนดพื้นฐานของระบบธนาคาร | Module 1.3.5 |

> 📋 **แบบฟอร์มที่กรอกไว้ใน Lab 1.2 ของวันแรก** คือสิ่งที่ท่านจะยื่นให้ทีม Infrastructure ของ BCEL เพื่อขอทรัพยากร ตอนนี้ท่านมีทั้งแผนและประสบการณ์ลงมือจริงแล้ว

---

### 🧪 กิจกรรมระหว่างรอ install.sh (ทำคู่ขนาน)

ระหว่างที่สคริปต์ทำงาน 8-12 นาที ให้ทำสองอย่างนี้ไปพร้อมกัน

**1. ทดสอบหลังเรียน (Post-test)** ตามลิงก์ที่วิทยากรแจ้ง เพื่อวัดความเข้าใจเทียบกับ Pre-test ของวันแรก

**2. เขียนแผนนำไปใช้จริงของตัวเอง** ตอบ 3 คำถามนี้

```
1. ระบบไหนของ BCEL ที่ท่านจะ instrument เป็นระบบแรก และทำไม
   ........................................................

2. ปัญหาที่ท่านสงสัยว่ามีอยู่แล้วในระบบนั้น แต่ยังไม่มีหลักฐาน
   ........................................................

3. Alert 3 ตัวแรกที่ท่านจะตั้ง
   1) ....................................................
   2) ....................................................
   3) ....................................................
```

> ✅ **เกณฑ์ผ่าน Module 3.8**
> - [ ] `docker compose ps` ทุกบริการเป็น `Up (healthy)`
> - [ ] `/_health/` ตอบ 200
> - [ ] ล็อกอินเข้าหน้าเว็บของ instance ตัวเองได้
> - [ ] ตั้ง `SENTRY_BEACON = False` และ `auth.allow-registration: false` แล้ว
> - [ ] ส่ง Error จากแอปบนโน้ตบุ๊กเข้า instance ตัวเองได้สำเร็จ
> - [ ] รันคำสั่งสำรองข้อมูลได้และเห็นไฟล์ `.json`
> - [ ] อธิบายได้ว่ายังต้องทำอะไรเพิ่มก่อนใช้จริงที่ธนาคาร

---


## 📌 สรุปหลักสูตรทั้ง 3 วัน

### เส้นทางการเรียนรู้ที่ผ่านมา

```
วันที่ 1: มองเห็น
  Observability → สถาปัตยกรรม Sentry → SaaS vs Self-hosted
  → วางแผนติดตั้ง → Project/DSN/Release/PII → Error เข้าระบบ
       │
       v
วันที่ 2: วินิจฉัย
  Issue Grouping → Context (User/Tags/Breadcrumbs) → Performance
  → JDBC Spans → แก้ N+1 & Slow Query → Angular → Distributed Tracing
       │
       v
วันที่ 3: ทำงานได้จริงในองค์กร
  Alerts (ไม่ล้า) → Dashboards → Release Health → Source Maps
  → Session Replay → Jenkins + K8s → Capstone
       │
       v
  🏁 ปิดท้าย: ติดตั้ง Sentry Self-hosted ด้วยมือตัวเอง
     พร้อมนำกลับไปวางที่ BCEL
```

### ความสามารถที่ได้รับ

| ก่อนอบรม | หลังอบรม |
| --- | --- |
| ผู้ใช้แจ้งปัญหา แล้วค่อยรู้ | Alert ดังก่อนผู้ใช้แจ้ง |
| "ระบบช้า" แต่ไม่รู้ว่าที่ไหน | เห็น waterfall ว่าช้าที่ span ไหน กี่ ms |
| ไล่ log หลายเครื่อง | เปิด Trace เดียวเห็นทั้งเส้นทาง |
| ไม่รู้ว่า deploy ดีขึ้นหรือแย่ลง | Crash Free Rate เทียบราย release |
| ทำซ้ำปัญหาไม่ได้ | Session Replay ย้อนดูสิ่งที่ผู้ใช้ทำ |
| Stack trace ของ Angular อ่านไม่ออก | Source Maps ทำให้เห็นบรรทัดจริง |
| ไม่รู้ว่า commit ไหนทำพัง | Suspect Commits ชี้ให้ |

### ✅ Checklist ปิดหลักสูตร

- [ ] **ติดตั้ง Sentry Self-hosted ด้วยมือตัวเองสำเร็จ** และส่ง Error เข้า instance ของตัวเองได้ (Module 3.8)
- [ ] มีคู่มือ TLS / backup / อัปเกรด และแบบฟอร์มขอทรัพยากร กลับไปทำต่อที่ธนาคาร
- [ ] ระบบงาน CRM/ERP ส่ง Error + Trace ครบทั้ง Frontend และ Backend
- [ ] Distributed Tracing เชื่อมกันตั้งแต่หน้าจอถึง SQL
- [ ] แก้ปัญหา N+1 และ Slow Query ได้จริง พร้อมตัวเลขพิสูจน์
- [ ] Alert ครบทุกระดับ P1–P4 และไม่สร้าง noise
- [ ] Dashboard ครบ 3 บทบาท
- [ ] Release Health + Source Maps + Suspect Commits ทำงาน
- [ ] Jenkins Pipeline สร้าง Release อัตโนมัติ
- [ ] Session Replay ปกปิดข้อมูลถูกต้องตามมาตรฐานธนาคาร
- [ ] ไม่มี PII หลุดเข้า Sentry (ตรวจซ้ำครั้งสุดท้าย)
- [ ] มีแผน 90 วันสำหรับนำไปใช้จริง

### 📖 เอกสารอ้างอิงรวมทั้งหลักสูตร

**เอกสารหลัก**

- Sentry Self-hosted: https://develop.sentry.dev/self-hosted/
- การอัปเกรด Self-hosted: https://develop.sentry.dev/self-hosted/releases/
- Sentry สำหรับ Spring Boot: https://docs.sentry.io/platforms/java/guides/spring-boot/
- Sentry สำหรับ Angular: https://docs.sentry.io/platforms/javascript/guides/angular/
- Sentry CLI: https://docs.sentry.io/cli/

**หัวข้อเฉพาะ**

- JDBC Instrumentation: https://docs.sentry.io/platforms/java/guides/spring-boot/tracing/instrumentation/jdbc/
- Custom Instrumentation: https://docs.sentry.io/platforms/java/guides/spring-boot/tracing/instrumentation/custom-instrumentation/
- Source Maps (Angular + CLI): https://docs.sentry.io/platforms/javascript/guides/angular/sourcemaps/uploading/cli/
- Session Replay Privacy: https://docs.sentry.io/platforms/javascript/session-replay/privacy/
- Alerts: https://docs.sentry.io/product/alerts/
- Releases & Health: https://docs.sentry.io/product/releases/
- Discover & Dashboards: https://docs.sentry.io/product/explore/discover-queries/
- Search Query Syntax: https://docs.sentry.io/concepts/search/
- Data Scrubbing: https://docs.sentry.io/security-legal-pii/scrubbing/

**การติดตามเวอร์ชัน**

- Sentry Self-hosted Releases: https://github.com/getsentry/self-hosted/releases
- sentry-java Releases: https://github.com/getsentry/sentry-java/releases
- sentry-javascript Releases: https://github.com/getsentry/sentry-javascript/releases
- sentry-cli Releases: https://github.com/getsentry/sentry-cli/releases
- Helm chart สำหรับ Kubernetes: https://github.com/sentry-kubernetes/charts

---

## 🙏 ปิดหลักสูตร

ขอบคุณผู้เข้าอบรมทุกท่านจาก **ທະນາຄານການຄ້າຕ່າງປະເທດລາວ (BCEL)** ที่ร่วมเรียนรู้และลงมือปฏิบัติตลอด 3 วัน

สิ่งที่สำคัญที่สุดที่อยากฝากไว้คือ **เครื่องมือ Monitoring ไม่ได้มีคุณค่าจากการติดตั้ง แต่มีคุณค่าจากการที่ทีมใช้มันจริงทุกวัน** ขอให้เริ่มจากสิ่งเล็ก ๆ ที่ทำได้จริง เช่น ตั้ง Alert 3 ตัวที่มีความหมาย แล้วค่อย ๆ ขยายไปตามแผน 90 วัน จะได้ผลดีกว่าการพยายามทำทุกอย่างพร้อมกันในสัปดาห์แรก

**ติดต่อวิทยากรและสถาบัน**

- อาจารย์สามิตร โกยม
- บริษัท ไอทีจีเนียส เอ็นจิเนียริ่ง จำกัด
- โทร. 02-570-8449 | มือถือ 088-807-9770, 092-841-7931
- Line ID: @itgenius | เว็บไซต์: www.itgenius.co.th | อีเมล: contact@itgenius.co.th
