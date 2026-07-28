# Lab วันที่ 1 — ฝัง Sentry เข้ากับ Spring Boot ให้เห็น Error ตัวแรก

**วันพุธที่ 29 กรกฎาคม 2569 · 09:30–16:30 น.**
เชื่อมโยงกับ `notes/Day1_note.md` Module 1.1–1.5

> ✅ **สถานะการทดสอบ** ทุกขั้นตอนในเอกสารนี้รันจริงบนเครื่องวิทยากรและยืนยันผลใน Sentry แล้ว
> ค่าเวลา ผลลัพธ์ และข้อความ error ที่แสดงคือค่าที่ได้จริง

---

## ภาพรวม Lab วันที่ 1

| Lab | หัวข้อ | เวลา | ผลลัพธ์ที่จับต้องได้ |
| --- | --- | --- | --- |
| 1.1 | ฝึกตั้งคำถามแบบ Observability | 20 นาที | ตารางคำถาม 3 เสา |
| 1.2 | วางแผนติดตั้ง Self-hosted สำหรับ BCEL | 30 นาที | เอกสารสเปกเซิร์ฟเวอร์ |
| 1.3 | ตั้งค่าโปรเจกต์และนโยบายข้อมูล | 20 นาที | Project + Environment + Scrubbing |
| **1.4** | **ฝัง SDK เข้า Spring Boot** | **60 นาที** | **Issue ตัวแรกใน Sentry** |
| **1.5** | **Data Scrubbing ปกป้อง PII** | **30 นาที** | **เห็น `[ACCOUNT_REDACTED]`** |
| **1.6** | **ทดลองกับดัก eventId** | **20 นาที** | **เข้าใจ DuplicateEventDetection** |

---

## Lab 1.1 — ฝึกตั้งคำถามแบบ Observability

**โจทย์** ระบบ CRM ของ BCEL ช้าลงตอน 14:00 น. ลูกค้าโทรมาบ่นว่า "หน้าค้นหาลูกค้าหมุนไม่จบ"

เติมตารางนี้ให้ครบ (ทำเป็นกลุ่ม 15 นาที แล้วนำเสนอ 5 นาที)

| เสาหลัก | คำถามที่ตอบได้ | เครื่องมือ |
| --- | --- | --- |
| Metrics | ตอน 14:00 CPU/latency สูงผิดปกติหรือไม่ | Prometheus/Grafana |
| Logs | ช่วงนั้นมี ERROR อะไรบ้าง | Sentry Breadcrumbs / ELK |
| Traces | request ที่ช้า เสียเวลาอยู่ที่ชั้นไหน | **Sentry Performance** |

**คำถามชวนคิด** ถ้ามีแค่ Metrics อย่างเดียว จะรู้ไหมว่า "ช้าเพราะ query ไหน"
→ ไม่รู้ นี่คือเหตุผลที่ต้องมี Traces

---

## Lab 1.2 — วางแผนติดตั้ง Self-hosted สำหรับ BCEL

**โจทย์** เขียนสเปกเซิร์ฟเวอร์และแผนติดตั้ง Sentry Self-hosted ให้ BCEL

| หัวข้อ | ขั้นต่ำ | ที่แนะนำสำหรับ BCEL |
| --- | --- | --- |
| CPU | 4 cores | 8 cores |
| RAM | 16 GB | 32 GB |
| Disk | 100 GB SSD | 500 GB SSD (event retention 90 วัน) |
| OS | Debian/Ubuntu | ⚠️ BCEL ใช้ **CentOS 8** ซึ่ง Sentry ไม่ได้ทดสอบไว้ |

**ประเด็นที่ต้องตัดสินใจ (อภิปรายในห้อง)**

1. CentOS 8 หมด EOL แล้ว → ทางเลือก: ย้าย Sentry ไป Ubuntu VM แยก หรือ deploy บน RKE2 แทน
2. Reverse proxy + TLS ภายใน (`sentry.bcel.local`) ใครเป็นผู้ออกใบรับรอง
3. นโยบาย retention ของธนาคาร — เก็บ event นานเท่าไร ใครเข้าถึงได้
4. แผนสำรองข้อมูล PostgreSQL + ClickHouse ทำสัปดาห์ละกี่ครั้ง

**ส่งงาน** เอกสาร 1 หน้า ระบุ สเปก / ผังเครือข่าย / ผู้รับผิดชอบ / แผน backup

---

## Lab 1.3 — ตั้งค่าโปรเจกต์และนโยบายข้อมูล

ทำตาม `Lab_00_Setup.md` ข้อ 0.5 ให้เสร็จก่อน แล้วทำต่อดังนี้

### ก) สร้าง Environment

Sentry สร้าง environment ให้อัตโนมัติเมื่อมี event เข้ามาครั้งแรก เราเพียงกำหนดว่าจะใช้ชื่ออะไร
**ห้องนี้ใช้ 3 ค่าเท่านั้น** — `development` / `staging` / `production`

> ❌ อย่าใช้ `dev`, `prod`, `test`, `local` ปนกัน เพราะจะกรองข้อมูลไม่ได้และ Release Health เพี้ยน

### ข) เปิด Data Scrubbing ระดับ Server (ชั้นที่ 2)

**Settings → Security & Privacy** เปิดทั้งหมดนี้

| ตัวเลือก | ค่า | เหตุผลสำหรับธนาคาร |
| --- | --- | --- |
| Data Scrubber | ✅ เปิด | ตัดฟิลด์อ่อนไหวอัตโนมัติ |
| Use Default Scrubbers | ✅ เปิด | ครอบคลุม password, token, secret |
| Additional Sensitive Fields | `account_no`, `id_card`, `citizen_id`, `phone` | ฟิลด์เฉพาะของ BCEL |
| Prevent Storing of IP Addresses | ✅ เปิด | IP = ข้อมูลส่วนบุคคล |

> 🔑 **แนวคิดสำคัญ** Data Scrubbing มี **2 ชั้น**
> ชั้นที่ 1 = `BeforeSendCallback` ในแอปเรา (ทำงานก่อนออกจากเครื่อง) ← ทำใน Lab 1.5
> ชั้นที่ 2 = Server-side scrubbing ของ Sentry (ทำงานหลังข้อมูลถึง Sentry แล้ว)
> **สำหรับข้อมูลที่ห้ามออกนอกองค์กรเด็ดขาด ต้องพึ่งชั้นที่ 1 เท่านั้น**

---

## Lab 1.4 — ฝัง Sentry SDK เข้ากับ Spring Boot ⭐

> นี่คือ Workshop หลักของวันที่ 1 · ใช้เวลา 60 นาที

### ขั้นที่ 1 — ปลดคอมเมนต์ dependency

เปิด `backend/pom.xml` หาบล็อกที่มีคอมเมนต์ `📌 วันที่ 1 Module 1.5` แล้วปลดคอมเมนต์

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

> ⚠️ **ต้องเป็น `-jakarta`** เพราะ Spring Boot 3 ใช้ `jakarta.*` ไม่ใช่ `javax.*`
> ถ้าเผลอใส่ `sentry-spring-boot-starter` (ไม่มี `-jakarta`) จะ build ผ่านแต่ **ไม่จับ error เลย**
> ซึ่งเป็นบั๊กที่หายากมากเพราะไม่มี error ให้เห็น

เวอร์ชันถูกกำหนดจาก `sentry-bom` ที่บรรทัด `<sentry.version>` อยู่แล้ว ไม่ต้องใส่ `<version>` รายตัว
เวอร์ชันที่ทดสอบแล้ว: **8.43.2**

### ขั้นที่ 2 — เติมค่าใน application.properties

เปิด `backend/src/main/resources/application.properties` ปลดคอมเมนต์บล็อก `📌 วันที่ 1 Module 1.5.3`

```properties
sentry.dsn=${SENTRY_DSN:}
sentry.environment=${SENTRY_ENVIRONMENT:development}
sentry.release=${SENTRY_RELEASE:bcel-crm-backend@1.0.0}

# ปิดการส่ง PII อัตโนมัติ (IP, header, cookie) บังคับสำหรับระบบงานธนาคาร
sentry.send-default-pii=false

# ระบุ package ของเราเอง เพื่อให้ Sentry แยก "โค้ดเรา" ออกจาก "โค้ด library"
sentry.in-app-includes=la.com.bcel.crm

# จับ exception ที่ถูก @ExceptionHandler จัดการไปแล้วด้วย
sentry.exception-resolver-order=-2147483647

# ไม่ต้องส่ง exception ที่เป็นเรื่องปกติของธุรกิจ
sentry.ignored-exceptions-for-type=\
  la.com.bcel.crm.common.BusinessValidationException,\
  org.springframework.web.servlet.NoHandlerFoundException

sentry.debug=false
sentry.logging.minimum-breadcrumb-level=info
sentry.logging.minimum-event-level=error
```

**อธิบายทีละบรรทัด (วิทยากรบรรยายพร้อมชี้จอ)**

| Property | ทำอะไร | ถ้าไม่ตั้งจะเป็นอย่างไร |
| --- | --- | --- |
| `sentry.dsn` | บอกว่าส่งไปที่ไหน อ่านจาก env เสมอ | SDK ปิดตัวเงียบ ๆ ไม่มี error |
| `sentry.in-app-includes` | แยกโค้ดเราออกจาก library | stack trace เต็มไปด้วยเฟรมของ Spring อ่านไม่รู้เรื่อง |
| `sentry.exception-resolver-order` | ให้ Sentry จับก่อน `@ControllerAdvice` | exception ที่มี handler จะหายไปหมด |
| `sentry.ignored-exceptions-for-type` | ไม่ส่ง error ที่เป็นเรื่องปกติทางธุรกิจ | Issue ท่วมด้วย validation error |
| `sentry.logging.minimum-event-level` | `log.error()` จะกลายเป็น event | (ดูกับดักใน Lab 1.6) |

### ขั้นที่ 3 — เพิ่ม captureException ใน 3 จุด

**ก) `common/GlobalExceptionHandler.java`**

เพิ่ม import

```java
import io.sentry.Sentry;
import io.sentry.protocol.SentryId;
```

ในเมธอด `handleDuplicate()` — ⚠️ **ต้องวางไว้ก่อน `log.error`**

```java
Sentry.captureException(e);
log.error("บันทึกข้อมูลไม่สำเร็จเพราะข้อมูลซ้ำ", e);
```

ในเมธอด `handleAll()`

```java
SentryId eventId = Sentry.captureException(e);
log.error("เกิดข้อผิดพลาดภายในระบบ", e);

String referenceId = SentryId.EMPTY_ID.equals(eventId) ? null : eventId.toString();

return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(new ApiError("INTERNAL_ERROR",
                "เกิดข้อผิดพลาด กรุณาแจ้งรหัสอ้างอิงนี้กับฝ่ายสนับสนุน", referenceId));
```

**ข) `config/SentryDebugController.java`**

เพิ่ม import `io.sentry.Sentry` และ `io.sentry.SentryLevel` แล้วปลดคอมเมนต์ 2 บรรทัด

```java
// ในเมธอด capture()
Sentry.captureException(e);

// ในเมธอด message()
Sentry.captureMessage("BCEL CRM: ทดสอบส่งข้อความเข้า Sentry", SentryLevel.INFO);
```

### ขั้นที่ 4 — Build และรัน

```bash
cd backend && mvn -B clean package -DskipTests
```

รันโดยส่ง DSN ผ่าน environment variable (**ห้าม hardcode DSN ลงไฟล์เด็ดขาด**)

```bash
SENTRY_DSN='<DSN ของ bcel-crm-backend>' mvn spring-boot:run
```

บน PowerShell

```powershell
$env:SENTRY_DSN='<DSN ของ bcel-crm-backend>'; mvn spring-boot:run
```

รอจน log ขึ้น `Started BcelCrmApplication in x seconds` แล้วตรวจ

```bash
curl -s http://localhost:8080/actuator/health
```

ต้องได้ `{"status":"UP","groups":["liveness","readiness"]}`

### ขั้นที่ 5 — ยิง event ทดสอบทั้ง 5 แบบ

```bash
curl -s http://localhost:8080/api/_debug/sentry/boom
```

```bash
curl -s http://localhost:8080/api/_debug/sentry/capture
```

```bash
curl -s http://localhost:8080/api/_debug/sentry/message
```

```bash
curl -s http://localhost:8080/api/_debug/sentry/ignored
```

```bash
curl -s http://localhost:8080/api/customers/4999/statement
```

### ขั้นที่ 6 — ดูผลใน Sentry

เปิด **Issues** เลือกโปรเจกต์ `bcel-crm-backend`

> ⚠️ **กับดักของหน้า Issues** ค่าปริยายคือมุมมอง **Feed / Recommended** ซึ่งกรองด้วย
> `is:unresolved` + จัดลำดับตาม priority บางครั้งจะ**ไม่แสดง Issue ที่เพิ่งเข้ามา**
> ถ้าหน้าจอว่างเปล่าทั้งที่ log บอกว่าส่งแล้ว ให้ **ล้างช่องค้นหาให้ว่าง** และเปลี่ยนช่วงเวลาเป็น **24H**

**ผลที่ต้องเห็น (ยืนยันจริงแล้ว 5 Issue)**

| # | Issue | Level | มาจาก | Culprit |
| --- | --- | --- | --- | --- |
| 1 | `IllegalStateException` — BCEL CRM: ทดสอบ unhandled exception ตัวแรก | Fatal · Unhandled | `/boom` | `GET /api/_debug/sentry/boom` |
| 2 | `ArithmeticException` — `/ by zero` | Error | `/capture` | `SentryDebugController` |
| 3 | `NullPointerException` — `Cannot invoke "Account.getBalance()" because "account" is null` | Fatal · Unhandled | `/customers/4999/statement` | `GET /api/customers/{id}/statement` |
| 4 | `IllegalArgumentException` — ไม่พบบัญชี **[ACCOUNT_REDACTED]** ของลูกค้ารหัส CUST-0042 | Fatal · Unhandled | `/pii` | ทำใน Lab 1.5 |
| 5 | `BCEL CRM: ทดสอบส่งข้อความเข้า Sentry` | Info | `/message` | `GET /api/_debug/sentry/message` |

**และที่สำคัญไม่แพ้กัน — สิ่งที่ต้อง _ไม่_ เห็น**

`BusinessValidationException` จาก `/ignored` **ต้องไม่ปรากฏ** เพราะถูก `sentry.ignored-exceptions-for-type` กรองออก
(endpoint ตอบ HTTP 400 พร้อม `VALIDATION_ERROR` ตามปกติ แต่ไม่มี Issue)

### ขั้นที่ 7 — อ่าน Issue ให้เป็น

คลิกที่ Issue `IllegalStateException` แล้วชี้ให้ผู้เรียนดูทีละส่วน

| ส่วน | อ่านอะไรจากมัน |
| --- | --- |
| **Highlights** | `handled`, `level`, `transaction`, `url`, **Trace ID** |
| **Stack Trace** | เฟรมของเราจะไฮไลต์เป็น "In App" เพราะตั้ง `in-app-includes` |
| **Breadcrumbs** | ลำดับเหตุการณ์ก่อนพัง เห็น log INFO และ HTTP request |
| **Tags** | `release`, `environment`, `transaction`, `url`, `server_name` |
| **Events (total)** | จำนวนครั้งที่เกิด — ยิงซ้ำ 5 ครั้งจะรวมเป็น Issue เดียว 5 events |

---

## Lab 1.5 — Data Scrubbing ปกป้อง PII ⭐

> หัวใจของหลักสูตรสำหรับสถาบันการเงิน · 30 นาที

### ขั้นที่ 1 — ทดสอบ "ก่อน" มีตัวกรอง

```bash
curl -s http://localhost:8080/api/_debug/sentry/pii
```

Endpoint นี้จงใจโยน exception ที่มีเลขบัญชีอยู่ในข้อความ

```java
throw new IllegalArgumentException(
        "ไม่พบบัญชี 011-123456789012 ของลูกค้ารหัส CUST-0042");
```

ไปดูใน Sentry — **จะเห็นเลขบัญชีเต็ม ๆ** นี่คือสิ่งที่ต้องไม่เกิดขึ้นกับข้อมูลจริง

### ขั้นที่ 2 — ติดตั้งตัวกรองชั้นที่ 1

คัดลอกไฟล์เฉลยเข้าโปรเจกต์

```bash
cp solutions/backend-sentry-config/BcelDataScrubber.java backend/src/main/java/la/com/bcel/crm/config/
```

เนื้อหาสำคัญของไฟล์

```java
@Component
public class BcelDataScrubber implements SentryOptions.BeforeSendCallback {

    private static final String ACCOUNT_PATTERN = "\\b\\d{3}-\\d{12}\\b";
    private static final String ID_CARD_PATTERN = "\\b\\d{13}\\b";

    @Override
    public SentryEvent execute(SentryEvent event, Hint hint) {
        event.setServerName(null);                    // ไม่เปิดเผยชื่อ host ภายใน

        if (event.getMessage() != null && event.getMessage().getFormatted() != null) {
            event.getMessage().setFormatted(mask(event.getMessage().getFormatted()));
        }

        // ⭐ จุดที่คนลืมบ่อยที่สุด
        if (event.getExceptions() != null) {
            for (SentryException ex : event.getExceptions()) {
                if (ex.getValue() != null) ex.setValue(mask(ex.getValue()));
            }
        }

        event.removeTag("customer_phone");
        event.removeTag("customer_email");
        return event;
    }
}
```

> ⭐ **จุดสอนที่สำคัญที่สุดของ Lab นี้**
> `unhandled exception` **ไม่มีข้อความอยู่ใน `event.getMessage()`** (ค่านั้นเป็น `null`)
> ข้อความจริงอยู่ใน `event.getExceptions()` ต่างหาก
> ถ้าเขียน scrubber ที่กรองเฉพาะ `getMessage()` จะรู้สึกว่า "ทำแล้ว" แต่**เลขบัญชีหลุดออกไปครบทุกตัว**

### ขั้นที่ 3 — ทดสอบ "หลัง" มีตัวกรอง

```bash
cd backend && mvn -B clean package -DskipTests
```

รันใหม่แล้วยิงซ้ำ

```bash
curl -s http://localhost:8080/api/_debug/sentry/pii
```

### ✅ เกณฑ์ตรวจผ่าน

ใน Sentry ข้อความของ Issue ต้องเป็น

```
ไม่พบบัญชี [ACCOUNT_REDACTED] ของลูกค้ารหัส CUST-0042
```

ถ้ายังเห็น `011-123456789012` แปลว่า

- ยังไม่ได้ rebuild หรือยังรัน jar ตัวเก่าอยู่
- หรือคลาสไม่ได้อยู่ใน package ที่ Spring สแกนเจอ (`la.com.bcel.crm.config`)
- หรือลืม `@Component`

> 💡 Issue เดิมที่ส่งไปก่อนติดตั้ง scrubber จะยังมีเลขเต็มอยู่ **ลบทิ้งไม่ได้ย้อนหลัง**
> ประเด็นนี้ต้องย้ำกับผู้เรียน: **ต้องติดตั้ง scrubber ตั้งแต่วันแรกที่เปิดใช้ Sentry**

---

## Lab 1.6 — กับดัก eventId ที่ทุกทีมเจอ ⭐

> Lab สั้น ๆ 20 นาที แต่เป็นความรู้ที่หาไม่ได้จากเอกสาร ทดสอบจริงบนเครื่องแล้ว

### สถานการณ์

เราอยากให้หน้าจอแสดง "รหัสอ้างอิง" ให้ลูกค้าแจ้ง Support เวลาเจอ error
โค้ดที่เขียนไปใน Lab 1.4 คือ

```java
SentryId eventId = Sentry.captureException(e);
```

### ทดลอง

```bash
curl -s http://localhost:8080/api/_debug/sentry/boom
```

**ผลที่ได้จริง**

```json
{"code":"INTERNAL_ERROR","message":"เกิดข้อผิดพลาด กรุณาแจ้งรหัสอ้างอิงนี้กับฝ่ายสนับสนุน","eventId":null}
```

`eventId` เป็น `null` (ก่อนใส่การป้องกันคือเลข **ศูนย์ 32 ตัว** `00000000000000000000000000000000`)
ทั้งที่ Issue ขึ้นใน Sentry เรียบร้อย

### ทำไม

เปิด `sentry.debug=true` แล้วรันใหม่ จะเห็นใน log

```
DEBUG: Duplicate Exception detected. Event 50a0b5db... will be discarded.
DEBUG: Event was dropped by a processor: io.sentry.DuplicateEventDetectionEventProcessor
```

**ลำดับเหตุการณ์จริง**

1. `sentry.exception-resolver-order=-2147483647` ทำให้ `SentryExceptionResolver` ของ Spring
   ทำงาน**ก่อน** `@ExceptionHandler` ของเรา และมัน capture exception ไปแล้ว
2. พอโค้ดเราเรียก `Sentry.captureException(e)` อีกครั้ง → `DuplicateEventDetectionEventProcessor`
   ตัดทิ้ง แล้วคืนค่า `SentryId.EMPTY_ID`
3. `Sentry.getLastEventId()` ก็คืนค่าว่างเช่นกัน เพราะ resolver ทำงานคนละ scope

**สาเหตุซ้ำอีกชั้น** `sentry.logging.minimum-event-level=error` ทำให้ `log.error("...", e)`
ส่ง event เองผ่าน `sentry-logback` ด้วย ถ้าเขียน `log.error` ไว้**ก่อน** `captureException`
ก็จะโดนตัดเป็นของซ้ำเหมือนกัน — นี่คือเหตุผลที่ Lab 1.4 สั่งให้วาง `captureException` ไว้ก่อนเสมอ

### ทางเลือกที่ต้องตัดสินใจ (อภิปรายในห้อง)

| | กลยุทธ์ A — ให้ Sentry จับเอง | กลยุทธ์ B — จับเองเพื่อเอา eventId |
| --- | --- | --- |
| `sentry.exception-resolver-order` | `-2147483647` (ใช้ค่านี้) | **ไม่ตั้ง** หรือตั้งเป็นค่าสูง |
| เขียน `captureException` เองไหม | ไม่ต้อง | ต้อง และวางไว้ก่อน `log.error` |
| ได้ eventId ให้ผู้ใช้ไหม | ❌ ไม่ได้ | ✅ ได้ |
| ครอบคลุม exception นอก DispatcherServlet | ✅ ครบ | ⚠️ อาจหลุดบางกรณี |

### ทดลองกลยุทธ์ B

รัน backend ใหม่โดยทับค่า property ตอนรัน

```bash
SENTRY_DSN='<DSN>' mvn spring-boot:run -Dspring-boot.run.arguments=--sentry.exception-resolver-order=2147483647
```

```bash
curl -s http://localhost:8080/api/_debug/sentry/boom
```

**ผลที่ได้จริง**

```json
{"code":"INTERNAL_ERROR","message":"เกิดข้อผิดพลาด กรุณาแจ้งรหัสอ้างอิงนี้กับฝ่ายสนับสนุน","eventId":"59f6387c28f94424932aa7a46a680b04"}
```

นำ `59f6387c` ไปค้นใน Sentry จะเจอ event นั้นตรง ๆ (ดูที่ช่อง `ID:` ในหน้า Issue)

**ข้อสรุปที่ต้องจดจำ** เลือกกลยุทธ์เดียว อย่าทำทั้งสองอย่างพร้อมกัน
สำหรับ BCEL แนะนำ **กลยุทธ์ A** เพื่อความครอบคลุม แล้วให้ Support ค้นด้วย
`transaction` + ช่วงเวลา + `user.id` แทนการใช้ eventId

---

## 📋 เกณฑ์ตรวจผ่าน Lab วันที่ 1

| # | สิ่งที่ต้องพิสูจน์ได้ | วิธีตรวจ |
| --- | --- | --- |
| 1 | Backend รันได้พร้อม Sentry SDK 8.43.2 | log ตอน start ไม่มี error และ `/actuator/health` = UP |
| 2 | มี Issue อย่างน้อย 5 รายการใน `bcel-crm-backend` | หน้า Issues (ล้าง filter, ช่วง 24H) |
| 3 | Unhandled exception ถูกจับเองโดยไม่ต้องเขียนโค้ด | Issue `/boom` มี tag `Unhandled` |
| 4 | `captureException` และ `captureMessage` ทำงาน | เห็น Issue จาก `/capture` และ `/message` |
| 5 | **`BusinessValidationException` ไม่ขึ้นใน Sentry** | ค้นหา `BusinessValidation` แล้วไม่เจอ |
| 6 | **Data Scrubbing ทำงาน** | Issue จาก `/pii` แสดง `[ACCOUNT_REDACTED]` |
| 7 | `in-app-includes` ทำงาน | Stack trace ไฮไลต์เฉพาะเฟรม `la.com.bcel.crm` |
| 8 | Release และ Environment ถูกติดป้าย | Tags แสดง `release: 1.0.0`, `environment: development` |
| 9 | อธิบายกับดัก eventId ได้ | ตอบได้ว่าทำไมได้เลขศูนย์ 32 ตัว |

---

## 🔧 ปัญหาที่พบบ่อยในวันที่ 1

| อาการ | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| ไม่มี Issue เข้า Sentry เลย | หน้า Issues ใช้มุมมอง Feed/Recommended | ล้างช่องค้นหา + เปลี่ยนเป็น 24H |
| ไม่มี Issue เข้าจริง ๆ | `SENTRY_DSN` ว่าง SDK จะปิดตัวเงียบ ๆ | รันด้วย `--sentry.debug=true` ต้องเห็น `Initializing SDK with DSN:` |
| ส่งได้แต่ช้ามาก | proxy องค์กรบล็อก | ดู log ต้องมี `DEBUG: Envelope sent successfully.` |
| Build ไม่ผ่าน `package io.sentry does not exist` | ยังไม่ปลดคอมเมนต์ใน `pom.xml` | ทำขั้นที่ 1 |
| `Unable to rename ...jar to ...jar.original` | backend เก่ายังรันค้างอยู่ | หยุด process เดิมก่อน `mvn clean` |
| Build ผ่านแต่ไม่จับ error | ใช้ starter ที่ไม่มี `-jakarta` | เปลี่ยนเป็น `sentry-spring-boot-starter-jakarta` |
| `Communications link failure` | MariaDB ยังไม่พร้อม / พอร์ตผิด | ดู `Lab_00_Setup.md` ข้อ 0.3 |

---

**สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง** · อ.สามิตร โกยม · 02-570-8449 · www.itgenius.co.th
