# APM & Error Tracking ด้วย Sentry - วันที่ 2: Error Tracking เชิงลึก, Performance Monitoring และ Distributed Tracing

**หลักสูตรอบรมเชิงปฏิบัติการ: Application Performance Monitoring & Error Tracking ด้วย Sentry (Self-hosted)**
**จัดอบรมให้: ທະນາຄານການຄ້າຕ່າງປະເທດລາວ / Banque pour le Commerce Extérieur Lao (BCEL)**
**วันที่ 2: วินิจฉัย Error อย่างเป็นระบบ และไล่ Trace ตั้งแต่หน้าจอ Angular จนถึง Query บน MariaDB**
วันที่: 30 กรกฎาคม 2569 | เวลา 09:30–16:30 น. | Onsite Hands-on Workshop
ผู้สอน: อ.สามิตร โกยม | สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง

---

## 🎯 วัตถุประสงค์การเรียนรู้ประจำวัน

เมื่อจบการอบรมวันที่ 2 ผู้เรียนจะสามารถ:

1. อธิบายกลไก **Issue Grouping / Fingerprinting** และแก้ปัญหาการจัดกลุ่มที่ผิดพลาดได้
2. อ่าน Stack Trace, จัดการสถานะ Issue (Unresolved / Resolved / Ignored / **Regression**) และตั้ง **Ownership Rules** ได้
3. ค้นหา Issue ด้วย **Search Query Syntax** ของ Sentry ได้อย่างคล่องแคล่ว
4. เพิ่มบริบทให้ Error ด้วย **User Context, Tags, Breadcrumbs, Custom Context** และจับ Error เองด้วย `captureException` / `captureMessage`
5. อธิบายแนวคิด **Transaction, Span, Trace** และเลือกกลยุทธ์ **Sampling** ที่เหมาะกับ Production ได้
6. เปิด Performance Monitoring บน Spring Boot และผูก **`sentry-jdbc`** เพื่อเปลี่ยนทุก SQL Query ให้เป็น Span
7. ค้นหา **Slow Query** และปัญหา **N+1** ของ MariaDB จากหน้า Performance ของ Sentry ได้ (Workshop 2.1)
8. ติดตั้ง **`@sentry/angular`** พร้อม ErrorHandler, TraceService และ Component Tracking
9. เชื่อม **Distributed Tracing แบบ End-to-End** ผ่าน `sentry-trace` / `baggage` header แล้วไล่ Trace 1 รายการจากหน้าจอจนถึง SQL ได้ (Workshop 2.2)

---

## 🧭 กำหนดการวันที่ 2

| เวลา | หัวข้อ |
| --- | --- |
| 09:30–09:45 | ทบทวนวันที่ 1 + ตรวจสถานะสภาพแวดล้อม |
| 09:45–10:45 | **Module 2.1** Error & Issue Tracking เชิงลึก |
| 10:45–12:00 | **Module 2.2** การเพิ่มบริบทให้กับข้อผิดพลาด (Context Enrichment) |
| 12:00–13:00 | พักกลางวัน |
| 13:00–13:45 | **Module 2.3** แนวคิด Performance Monitoring และ Distributed Tracing |
| 13:45–14:45 | **Module 2.4** Performance Tracing ใน Spring Boot และ MariaDB + **Workshop 2.1** |
| 14:45–15:30 | **Module 2.5** Instrument ระบบ Frontend ด้วย Angular |
| 15:30–16:30 | **Module 2.6** เชื่อม Distributed Tracing E2E + **Workshop 2.2** + สรุปประจำวัน |

---

## 🔄 ทบทวนวันที่ 1 และตรวจความพร้อม

### เวลา 09:30–09:45 น.

```bash
# 1) Sentry ยังทำงานอยู่ไหม
curl -s -o /dev/null -w '%{http_code}\n' https://sentry.io    # ต้องได้ 200 หรือ 30x

# 2) ฐานข้อมูลในเครื่องของท่าน (ต้องอยู่ในโฟลเดอร์ bcel-crm-lite)
cd bcel-crm-lite
docker compose exec mariadb mariadb -u crm_app -plabpass123 bcel_crm \
  -e "SELECT COUNT(*) FROM customer;"

# 3) Backend ยังส่ง event ได้ไหม
export SENTRY_DSN='<DSN ของ bcel-crm-backend>'
export SENTRY_ENVIRONMENT=development
export SENTRY_RELEASE='bcel-crm-backend@1.0.0'
cd backend && mvn spring-boot:run
```

**คำถามทบทวน 3 ข้อ (ตอบปากเปล่า):**

1. DSN กับ Auth Token ต่างกันอย่างไร และอันไหนห้ามฝังใน Angular bundle
2. ถ้า Error ไม่เข้า Sentry เราจะไล่ตรวจตามลำดับอย่างไร
3. `sentry.in-app-includes` มีผลอย่างไรกับหน้า Stack Trace

---

## 📚 Module 2.1: Error & Issue Tracking เชิงลึก

### เวลา 09:45–10:45 น.

> 💡 **หัวใจของ Module นี้:** Sentry ไม่ได้เก็บ Error เป็นรายการยาว ๆ แต่ **จัดกลุ่ม (group)** Error ที่เป็นเรื่องเดียวกันเข้าด้วยกันเป็น "Issue" ถ้าการจัดกลุ่มผิด เราจะเจอทั้งปัญหา "Issue เดียวมีหลายบั๊กปนกัน" หรือ "บั๊กเดียวแตกเป็นพันไอเทม" ซึ่งทำให้ระบบใช้งานไม่ได้จริง

---

### 2.1.1 Event กับ Issue ต่างกันอย่างไร

```
             Events (เหตุการณ์รายครั้ง)              Issue (กลุ่ม)
  ┌────────────────────────────────────────┐
  │ 10:01 NPE ที่ CustomerService:88       │  ─┐
  │ 10:03 NPE ที่ CustomerService:88       │   ├──> Issue #1042
  │ 10:07 NPE ที่ CustomerService:88       │   │    "NullPointerException
  │ 10:22 NPE ที่ CustomerService:88       │  ─┘     CustomerService.buildStatement"
  ├────────────────────────────────────────┤        events: 4 · users: 3
  │ 10:15 DataIntegrityViolation TK-0001   │  ─┐
  │ 10:16 DataIntegrityViolation TK-0002   │   ├──> Issue #1043
  └────────────────────────────────────────┘  ─┘
```

| หน่วย | คืออะไร | ใช้ทำอะไร |
| --- | --- | --- |
| **Event** | 1 ครั้งที่เกิด Error พร้อมบริบทเต็ม (stack, tags, breadcrumbs, user) | ดูรายละเอียดของเหตุการณ์จริง |
| **Issue** | กลุ่มของ Event ที่ Sentry ตัดสินว่า "เป็นบั๊กเดียวกัน" | ติดตาม, มอบหมาย, ปิดงาน |
| **Fingerprint** | ลายนิ้วมือที่ใช้ตัดสินว่า Event ใดอยู่กลุ่มเดียวกัน | ควบคุมการจัดกลุ่ม |

### 2.1.2 กลไกการจัดกลุ่มโดยปริยาย (Default Grouping)

Sentry คำนวณ fingerprint จากองค์ประกอบต่อไปนี้ตามลำดับความสำคัญ:

```
1) ถ้ามี Stack Trace   → ใช้ชนิด exception + ชุด stack frame ที่เป็น "in-app"
2) ถ้าไม่มี stack       → ใช้ชนิด exception + ข้อความ exception
3) ถ้าเป็น message      → ใช้ข้อความหลังจากถอด parameter ออก
```

> 💡 **นี่คือเหตุผลที่ `sentry.in-app-includes=la.com.bcel.crm` สำคัญมาก** ถ้าไม่ตั้ง Sentry อาจใช้ frame ของ Spring หรือ Hibernate มาคำนวณ fingerprint ทำให้บั๊กคนละตัวถูกจับรวมกันเพราะบังเอิญผ่าน `DispatcherServlet` เหมือนกัน

### 2.1.3 ปัญหาการจัดกลุ่ม 2 แบบ และวิธีแก้

**ปัญหาแบบที่ 1: แตกกลุ่มมากเกินไป (over-grouping split)**

```
❌ อาการ: Issue ใหม่โผล่ทุกครั้งที่มีลูกค้าคนใหม่เจอปัญหา
   "Customer C0001 not found"     -> Issue #1
   "Customer C0002 not found"     -> Issue #2
   "Customer C0003 not found"     -> Issue #3   ... จนถึง #5000
```

**สาเหตุ:** ข้อความ exception มีค่าที่เปลี่ยนไปเรื่อย ๆ (id, timestamp, uuid) ปนอยู่

**วิธีแก้ที่ 1 - ใช้ข้อความคงที่แล้วเก็บค่าตัวแปรใน context (แนะนำที่สุด):**

```java
// ❌ แบบเดิม: id ปนในข้อความ ทำให้แตกกลุ่ม
throw new CustomerNotFoundException("Customer " + id + " not found");

// ✅ แบบใหม่: ข้อความคงที่ ค่าตัวแปรเก็บแยก
Sentry.withScope(scope -> {
    scope.setTag("customer_id", String.valueOf(id));
    scope.setContexts("lookup", Map.of("customerId", id, "source", "crm-search"));
    Sentry.captureException(new CustomerNotFoundException("Customer not found"));
});
```

**วิธีแก้ที่ 2 - กำหนด fingerprint เอง:**

```java
Sentry.withScope(scope -> {
    // บังคับให้ทุก event ของกรณีนี้อยู่กลุ่มเดียวกัน
    scope.setFingerprint(List.of("customer-not-found"));
    Sentry.captureException(e);
});
```

**วิธีแก้ที่ 3 - ตั้ง Fingerprint Rule ใน UI** (Project Settings → Issue Grouping → Fingerprint Rules):

```
error.type:"CustomerNotFoundException"                  -> customer-not-found
error.value:"*not found*"                               -> resource-not-found
error.type:"DataIntegrityViolationException" path:"*/tickets" -> ticket-duplicate
```

**ปัญหาแบบที่ 2: รวมกลุ่มมากเกินไป (over-grouping merge)**

```
❌ อาการ: Issue เดียวชื่อ "RuntimeException" มี 40,000 events
   แต่ข้างในมีทั้งปัญหา DB timeout, ปัญหาแปลงวันที่ และปัญหา null ปนกัน
```

**สาเหตุ:** โค้ดใช้ exception กว้างเกินไป เช่น `throw new RuntimeException(e)` ทับหมด

**วิธีแก้:**

```java
// ❌ ห่อทุกอย่างด้วย RuntimeException -> Sentry แยกไม่ออก
catch (Exception e) {
    throw new RuntimeException(e);
}

// ✅ ใช้ exception ที่มีความหมายเฉพาะ
catch (SQLTimeoutException e) {
    throw new CustomerQueryTimeoutException("query customer summary timeout", e);
} catch (DateTimeParseException e) {
    throw new InvalidReportDateException("invalid report date format", e);
}
```

หรือใช้ปุ่ม **Split** ใน UI เพื่อแยก Issue ที่ปนกันออกจากกัน (Issue → เมนู ⋯ → Split)

### 2.1.4 อ่าน Stack Trace ให้เป็น

เมื่อเปิด Issue จะเห็นส่วนประกอบเหล่านี้:

```
┌─────────────────────────────────────────────────────────────────────┐
│ NullPointerException                                    [Resolve ▾] │
│ CustomerStatementService.buildStatement                             │
│ Cannot invoke "Account.getBalance()" because "account" is null      │
├─────────────────────────────────────────────────────────────────────┤
│ Events: 47   Users: 12   First seen: 3h ago   Last seen: 2m ago     │
├─────────────────────────────────────────────────────────────────────┤
│ TAGS                                                                 │
│  environment: development(100%)  release: 1.0.0(100%)               │
│  server_name: [filtered]         level: error(100%)                 │
├─────────────────────────────────────────────────────────────────────┤
│ STACK TRACE                              [Newest ▾] [Full ☐]        │
│                                                                      │
│   la.com.bcel.crm.customer.CustomerStatementService  ← in-app (เข้ม) │
│     in buildStatement at line 88                                     │
│       86 |   Account account = accountRepo.findPrimary(customerId);  │
│     ▸ 88 |   BigDecimal balance = account.getBalance();  💥          │
│       89 |   statement.setOpeningBalance(balance);                   │
│                                                                      │
│   la.com.bcel.crm.customer.CustomerController         ← in-app       │
│     in statement at line 54                                          │
│                                                                      │
│   org.springframework.web.servlet.DispatcherServlet   ← ไม่ใช่ in-app │
│     (จาง / ยุบไว้โดยอัตโนมัติ)                                       │
├─────────────────────────────────────────────────────────────────────┤
│ BREADCRUMBS (ย้อนรอยก่อนเกิด error)                                  │
│  10:22:14 http   GET /api/customers/4999          200                │
│  10:22:15 query  SELECT * FROM account WHERE ...  0 rows            │
│  10:22:15 error  ...                                                 │
└─────────────────────────────────────────────────────────────────────┘
```

| ส่วน | สิ่งที่ต้องดูเป็นอันดับแรก |
| --- | --- |
| **Culprit** (บรรทัดใต้ชื่อ) | บอกทันทีว่าเมธอดใดเป็นต้นเหตุ |
| **Events / Users** | บอกความรุนแรง: 47 events แต่กระทบ 12 users ต่างจาก 47 events กระทบ 1 user มาก |
| **First seen / Last seen** | ถ้า First seen ตรงกับเวลา Deploy = สงสัย Release ใหม่ทันที |
| **In-app frames** | ต้องอ่านบรรทัดนี้ก่อนเสมอ ไม่ใช่ frame ของ framework |
| **Breadcrumbs** | บอกลำดับเหตุการณ์ก่อนพัง มักเจอเบาะแสที่ stack trace ไม่บอก |

> 💡 **เคล็ดลับการอ่าน:** กด **"Full"** เพื่อดู frame ทั้งหมดรวม library และกดที่ frame แต่ละอันเพื่อดู source context (โค้ดรอบ ๆ) หากตั้งค่า Source Context ไว้แล้ว (`sentry-maven-plugin` อัปโหลดซอร์ส) จะเห็นโค้ดจริงตรงนั้นเลย

### 2.1.5 สถานะของ Issue และการทำงานเป็นทีม

```
                    ┌──────────────┐
   Error เกิดใหม่ ──>│  UNRESOLVED  │<─────────────┐
                    └──────┬───────┘              │
                           │                      │ เกิดซ้ำใน release ใหม่
              ┌────────────┼────────────┐         │
              v            v            v         │
       ┌──────────┐  ┌──────────┐  ┌─────────┐   │
       │ RESOLVED │  │ IGNORED  │  │ARCHIVED │   │
       └────┬─────┘  └──────────┘  └─────────┘   │
            │                                     │
            └──────> REGRESSION ──────────────────┘
                (Sentry เปลี่ยนกลับเป็น Unresolved
                 พร้อมติดป้าย regression อัตโนมัติ)
```

| สถานะ | ความหมาย | ใช้เมื่อไร |
| --- | --- | --- |
| **Unresolved** | ยังไม่แก้ | ค่าเริ่มต้น |
| **Resolved** | แก้แล้ว | หลังแก้โค้ดเสร็จ |
| **Resolved in next release** | แก้แล้ว รอ deploy | แก้ใน branch แล้ว แต่ยังไม่ขึ้น production ✅ แนะนำมาก |
| **Ignored / Archived** | รู้แล้ว แต่ยังไม่แก้ | Error จาก bot, browser เก่า, หรือของ third-party |
| **Regression** | เคยแก้แล้วกลับมาอีก | Sentry ตั้งให้เองเมื่อเจอ event ใน release ที่ใหม่กว่าตอน resolve |

> ✅ **แนวปฏิบัติที่แนะนำสำหรับทีม BCEL:** ใช้ **"Resolved in next release"** แทน "Resolved" เปล่า ๆ เพราะจะทำให้ Sentry ตรวจ Regression ได้แม่นยำ (ต้องมี Release ตั้งถูกต้องตามที่ทำไปวันที่ 1)

**Ignore แบบมีเงื่อนไข** (คลิกลูกศรข้างปุ่ม Ignore):

| ตัวเลือก | ความหมาย |
| --- | --- |
| Ignore until it happens X times | เงียบไว้ก่อน แต่ถ้าเกินเกณฑ์ให้เตือน |
| Ignore for 24h / 7d | พักไว้ชั่วคราว |
| Ignore until it affects X users | สนใจเฉพาะเมื่อกระทบผู้ใช้จำนวนมาก |

### 2.1.6 ระดับความรุนแรง (Level) และการมอบหมาย (Ownership Rules)

**Level** ที่ Sentry รองรับ: `fatal` > `error` > `warning` > `info` > `debug`

```java
// กำหนด level เองตามความรุนแรงทางธุรกิจ
Sentry.withScope(scope -> {
    scope.setLevel(SentryLevel.FATAL);   // กระทบการทำธุรกรรม = ร้ายแรงที่สุด
    Sentry.captureException(new CoreBankingUnreachableException("core banking timeout"));
});
```

**Ownership Rules** ทำให้ Issue ถูกมอบหมายอัตโนมัติ ตั้งที่ Project Settings → Ownership Rules:

```
# รูปแบบ: matcher:pattern  #team หรือ email

# แยกตาม package ของโค้ด
path:*/customer/*        #crm-dev
path:*/order/*           #erp-dev
path:*/report/*          #crm-dev somchai@bcel.com.la

# แยกตาม URL ของ API
url:*/api/customers/*    #crm-dev
url:*/api/orders/*       #erp-dev

# แยกตาม tag
tags.module:billing      #finance-dev
```

> 💡 **ผลลัพธ์ที่ได้:** เมื่อ Error เกิดในไฟล์ที่ตรงกับ pattern Sentry จะแสดง "Suggested Assignees" และสามารถตั้งให้ Alert ส่งเข้าเฉพาะทีมนั้นได้ (จะใช้จริงในวันที่ 3)

### 2.1.7 Search Query Syntax ที่ต้องใช้ทุกวัน

```
# --- พื้นฐาน ---
is:unresolved                                  Issue ที่ยังไม่แก้
is:resolved                                    Issue ที่แก้แล้ว
is:assigned                                    มีคนรับผิดชอบแล้ว
is:unassigned is:unresolved                    ยังไม่มีใครรับ และยังไม่แก้

# --- กรองตาม environment / release ---
environment:production is:unresolved
release:bcel-crm-backend@1.4.0
firstRelease:bcel-crm-backend@1.4.0            เพิ่งเกิดครั้งแรกใน release นี้ ⭐

# --- กรองตามความรุนแรง ---
level:error
level:fatal is:unresolved

# --- กรองตามจำนวน ---
timesSeen:>100                                 เกิดมากกว่า 100 ครั้ง
userCount:>50                                  กระทบผู้ใช้มากกว่า 50 คน

# --- กรองตามเวลา ---
firstSeen:-24h                                 เพิ่งโผล่ใน 24 ชม.
lastSeen:-1h                                   ยังเกิดอยู่ในชั่วโมงล่าสุด
age:-2h                                        Issue อายุน้อยกว่า 2 ชม.

# --- กรองตาม tag ที่เราตั้งเอง ---
customer_segment:PREMIER
branch_code:HQ-001
module:billing

# --- ค้นข้อความ ---
message:"timeout"
error.type:"NullPointerException"
error.value:"*account*"

# --- ผสมกันแบบใช้งานจริง ---
is:unresolved environment:production level:error firstRelease:bcel-crm-backend@1.4.0
is:unresolved userCount:>10 lastSeen:-2h
is:unassigned level:fatal environment:production
```

> ✅ **บันทึกไว้เป็น Saved Search:** คลิกไอคอนบุ๊กมาร์กข้างช่องค้นหา แล้วบันทึกชุดค้นหาที่ใช้บ่อย เช่น "🔥 ต้องแก้ด่วนวันนี้" = `is:unresolved environment:production level:error userCount:>5`

---

### 🧪 Lab 2.1 - ฝึกจัดการ Issue และค้นหา

> **เป้าหมาย:** ใช้เครื่องมือจัดการ Issue ได้คล่อง และเข้าใจการจัดกลุ่ม

**ขั้นที่ 1 - สร้าง Error ที่แตกกลุ่มมากเกินไป**

```java
// เพิ่มใน SentryDebugController.java (ชั่วคราว)
@GetMapping("/split/{id}")
public String splitDemo(@PathVariable long id) {
    // ❌ ตั้งใจเขียนผิด: เอา id ใส่ในข้อความ
    throw new IllegalStateException("ไม่พบข้อมูลของลูกค้ารหัส " + id);
}
```

```bash
for i in 1 2 3 4 5; do curl -s "http://localhost:8080/api/_debug/sentry/split/$i" > /dev/null; done
```

> **สังเกต:** ควรได้ Issue แยกกัน 5 รายการ ทั้งที่เป็นบั๊กเดียวกัน

**ขั้นที่ 2 - แก้ด้วย Fingerprint แล้วเทียบผล**

```java
@GetMapping("/split-fixed/{id}")
public String splitFixed(@PathVariable long id) {
    Sentry.withScope(scope -> {
        scope.setFingerprint(java.util.List.of("customer-data-missing"));
        scope.setTag("customer_id", String.valueOf(id));
        Sentry.captureException(new IllegalStateException("ไม่พบข้อมูลของลูกค้า"));
    });
    return "sent";
}
```

```bash
for i in 1 2 3 4 5; do curl -s "http://localhost:8080/api/_debug/sentry/split-fixed/$i" > /dev/null; done
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** คราวนี้ได้ **Issue เดียว** ที่มี 5 events และมี tag `customer_id` ให้กรองย่อยได้

**ขั้นที่ 3 - ฝึก Merge / Split / Ignore**

1. เลือก Issue ที่แตก 5 อันจากขั้นที่ 1 → กดเลือกทั้งหมด → **Merge** → สังเกตว่ารวมเป็นอันเดียว
2. เปิด Issue ที่รวมแล้ว → เมนู ⋯ → **Split** → แยกกลับ
3. ตั้ง **Ignore until it happens 10 times in 1 hour** ให้กับ Issue ของ debug endpoint

**ขั้นที่ 4 - ฝึก Search Query**

ลองค้นหาต่อไปนี้แล้วบันทึกจำนวนผลลัพธ์:

```
is:unresolved environment:development
error.type:"NullPointerException"
timesSeen:>3
firstSeen:-1h is:unresolved
```

---

## 📚 Module 2.2: การเพิ่มบริบทให้กับข้อผิดพลาด (Context Enrichment)

### เวลา 10:45–12:00 น.

> 💡 **หัวใจของ Module นี้:** Stack Trace บอกว่า "โค้ดบรรทัดไหนพัง" แต่ไม่บอกว่า "พังกับใคร ตอนไหน ในสถานการณ์อะไร" ซึ่งเป็นข้อมูลที่ทำให้แก้บั๊กได้ใน 10 นาทีแทนที่จะเป็น 3 วัน Module นี้คือหัวใจของหลักสูตรทั้งหมด

---

### 2.2.1 ภาพรวมกลไกการเพิ่มบริบท

| กลไก | จำนวนที่เก็บได้ | ค้นหา/กรองได้ | เหมาะกับ |
| --- | --- | --- | --- |
| **User** | 1 ต่อ event | ✅ | ใครได้รับผลกระทบ |
| **Tags** | หลายคู่ key/value (ค่าสั้น) | ✅ **กรองได้** | มิติที่ใช้แบ่งกลุ่ม เช่น branch, module |
| **Breadcrumbs** | สูงสุด ~100 รายการ | ❌ | ย้อนรอยลำดับเหตุการณ์ |
| **Contexts** | โครงสร้างซ้อนกันได้ | ❌ | ข้อมูลชุดใหญ่ เช่น payload สรุป |
| **Extra** | key/value อิสระ | ❌ | ข้อมูลเสริมเบ็ดเตล็ด (ควรย้ายไปใช้ Contexts) |
| **Attachments** | ไฟล์แนบ | ❌ | log file, screenshot |

```
 Event หนึ่งตัวประกอบด้วย:
 ┌────────────────────────────────────────────────────────┐
 │ EXCEPTION   NullPointerException + stack trace         │
 │ USER        id=CUST-0042, username=teller05            │  ⟵ ใครเจอ
 │ TAGS        branch=HQ-001, module=crm, segment=PREMIER │  ⟵ กรองได้
 │ CONTEXTS    request{...}, business{orderTotal: 15000}  │  ⟵ รายละเอียด
 │ BREADCRUMBS [10:22:14] GET /api/customers/4999 -> 200  │  ⟵ ย้อนรอย
 │             [10:22:15] SELECT account ... -> 0 rows    │
 │ TRACE       trace_id=a1b2c3... span_id=d4e5f6...       │  ⟵ เชื่อม Trace
 └────────────────────────────────────────────────────────┘
```

### 2.2.2 User Context - ระบุว่าใครได้รับผลกระทบ

**วิธีที่ 1 - ทำให้อัตโนมัติทุก request ด้วย Filter (แนะนำ)**

```java
// backend/src/main/java/la/com/bcel/crm/config/SentryUserContextFilter.java
package la.com.bcel.crm.config;

import io.sentry.Sentry;
import io.sentry.protocol.User;
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.io.IOException;

/**
 * ผูกข้อมูลผู้ใช้เข้ากับทุก event ที่เกิดใน request นี้
 *
 * สำคัญ: ระบบงานธนาคาร ห้ามส่งชื่อจริง/อีเมล/เบอร์โทรของลูกค้า
 * ให้ส่งเฉพาะ "รหัสภายใน" ที่ทีมเราแปลกลับเป็นตัวตนได้เอง
 */
@Component
@Order(1)
public class SentryUserContextFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) req;

        Sentry.configureScope(scope -> {
            User user = new User();

            // ✅ ใช้รหัสพนักงาน/รหัสผู้ใช้ภายใน ไม่ใช่ชื่อจริง
            String staffId = request.getHeader("X-Staff-Id");
            String branch  = request.getHeader("X-Branch-Code");

            user.setId(staffId != null ? staffId : "anonymous");
            user.setUsername(staffId);     // ยังเป็นรหัส ไม่ใช่ชื่อ

            // ❌ ห้ามทำ: user.setEmail(...), user.setIpAddress(...)

            scope.setUser(user);

            // ผูก tag ที่ใช้กรองบ่อย
            if (branch != null) {
                scope.setTag("branch_code", branch);
            }
            scope.setTag("http_method", request.getMethod());
        });

        try {
            chain.doFilter(req, res);
        } finally {
            // เคลียร์ user ออกเมื่อจบ request กัน thread pool นำไปใช้ซ้ำผิดคน
            Sentry.configureScope(scope -> scope.setUser(null));
        }
    }
}
```

**วิธีที่ 2 - ดึงจาก Spring Security**

```java
@Component
public class SecurityUserProvider implements SentryUserProvider {

    @Override
    public User provideUser() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) {
            return null;
        }
        User user = new User();
        user.setId(auth.getName());        // username จาก LDAP/AD
        return user;
    }
}
```

> ⚠️ **สำคัญมากสำหรับ BCEL:** อย่าลืมเคลียร์ user ใน `finally` เพราะ Servlet container ใช้ thread pool ถ้าไม่เคลียร์ Error ของ request ถัดไปอาจถูกติดป้ายว่าเป็นของผู้ใช้คนก่อน ซึ่งเป็นข้อมูลผิดที่อันตรายมากในบริบทธนาคาร

**ผลลัพธ์ที่ได้:** หน้า Issue จะแสดง **"12 users affected"** และคลิกดูได้ว่าเป็นใครบ้าง ทำให้ตอบคำถามผู้บริหารได้ว่า "ปัญหานี้กระทบสาขาไหนบ้าง"

### 2.2.3 Tags - มิติที่ใช้กรองและวิเคราะห์

**หลักการเลือกว่าอะไรควรเป็น Tag:**

| ✅ ควรเป็น Tag | ❌ ไม่ควรเป็น Tag |
| --- | --- |
| ค่าที่มีจำนวนแบบจำกัด (low cardinality) | ค่าที่ไม่ซ้ำกันเลย เช่น uuid, timestamp |
| ใช้กรอง/แบ่งกลุ่มบ่อย | ใช้แค่ดูรายละเอียด (ให้ใช้ Contexts) |
| สั้น (< 200 ตัวอักษร) | ข้อความยาว, JSON |
| เช่น `branch_code`, `module`, `segment` | เช่น `request_body`, `sql_query` |

**กำหนด Tag ระดับทั้งแอป:**

```properties
# application.properties - tag ที่ติดกับทุก event
sentry.tags.system=bcel-crm
sentry.tags.layer=backend
sentry.tags.datacenter=vte-dc1
```

**กำหนด Tag ระดับ request/business:**

```java
// backend/src/main/java/la/com/bcel/crm/customer/CustomerService.java
@Service
public class CustomerService {

    public CustomerSummary getSummary(Long customerId) {
        Customer customer = customerRepo.findById(customerId)
                .orElseThrow(() -> new CustomerNotFoundException("customer not found"));

        // ผูก tag ทางธุรกิจ เพื่อให้กรองได้ว่าปัญหาเกิดกับลูกค้ากลุ่มไหน
        Sentry.configureScope(scope -> {
            scope.setTag("customer_segment", customer.getSegment());   // RETAIL / SME / PREMIER
            scope.setTag("customer_branch", customer.getBranchCode());
            scope.setTag("module", "crm.customer.summary");
        });

        return buildSummary(customer);
    }
}
```

**ประโยชน์ที่เห็นทันที:** เปิด Issue แล้วดูแถบ Tags จะเห็นการกระจายตัวเป็นเปอร์เซ็นต์ เช่น

```
customer_segment    PREMIER 94%   RETAIL 4%   SME 2%
customer_branch     HQ-001  88%   BR-012 8%   อื่น ๆ 4%
```

> ✅ **นี่คือเบาะแสระดับทอง:** ถ้า 94% ของ Error เกิดกับลูกค้า `PREMIER` เท่านั้น แปลว่าโค้ดเส้นทางของ PREMIER มีเงื่อนไขพิเศษบางอย่างที่ผิด ซึ่งเราคงหาไม่เจอถ้าดูแค่ Stack Trace

### 2.2.4 Breadcrumbs - ย้อนรอยเหตุการณ์ก่อนพัง

Breadcrumb คือ "รอยเท้า" ที่ SDK เก็บสะสมไว้ตลอด request แล้วแนบไปกับ event เมื่อเกิด Error

**Breadcrumb ที่ Sentry เก็บให้อัตโนมัติ:**

| แหล่ง | ตัวอย่าง |
| --- | --- |
| Logback (ระดับ >= info) | `INFO CustomerService - loading summary for 4999` |
| HTTP client (RestTemplate/WebClient) | `GET https://core-banking/api/v1/balance -> 200` |
| JDBC (ถ้าเปิด sentry-jdbc) | `SELECT * FROM account WHERE customer_id = ?` |

**เพิ่ม Breadcrumb เอง:**

```java
import io.sentry.Breadcrumb;
import io.sentry.Sentry;
import io.sentry.SentryLevel;

@Service
public class TicketService {

    public Ticket create(CreateTicketRequest req) {
        Breadcrumb start = new Breadcrumb();
        start.setCategory("ticket");
        start.setMessage("เริ่มสร้าง ticket");
        start.setLevel(SentryLevel.INFO);
        start.setData("ticketNo", req.ticketNo());
        start.setData("priority", req.priority());
        // ❌ ห้ามใส่: start.setData("customerPhone", ...)
        Sentry.addBreadcrumb(start);

        validate(req);
        Sentry.addBreadcrumb("ผ่านการตรวจสอบข้อมูลแล้ว", "ticket");

        Ticket saved = ticketRepo.save(req.toEntity());

        Breadcrumb done = new Breadcrumb();
        done.setCategory("ticket");
        done.setMessage("บันทึก ticket สำเร็จ");
        done.setData("ticketId", saved.getId());
        Sentry.addBreadcrumb(done);

        return saved;
    }
}
```

**กรอง Breadcrumb ที่มีข้อมูลอ่อนไหวออก:**

```java
package la.com.bcel.crm.config;

import io.sentry.Breadcrumb;
import io.sentry.Hint;
import io.sentry.SentryOptions;
import org.springframework.stereotype.Component;

@Component
public class BreadcrumbFilter implements SentryOptions.BeforeBreadcrumbCallback {

    @Override
    public Breadcrumb execute(Breadcrumb breadcrumb, Hint hint) {
        String message = breadcrumb.getMessage();

        // ทิ้ง breadcrumb ที่มีคำอ่อนไหว
        if (message != null && (message.contains("password")
                             || message.contains("otp")
                             || message.contains("token"))) {
            return null;   // คืน null = ไม่เก็บ breadcrumb นี้
        }

        // ลบ data field ที่อ่อนไหว
        if (breadcrumb.getData() != null) {
            breadcrumb.getData().remove("authorization");
            breadcrumb.getData().remove("card_no");
        }

        return breadcrumb;
    }
}
```

> 💡 **ทำไม Breadcrumb ถึงมีค่ามาก:** ในกรณีจริง เรามักเจอ `NullPointerException` ที่ดู stack trace แล้วงงว่า "ทำไม object นี้เป็น null" แต่พอดู breadcrumb แล้วเห็นว่า `SELECT * FROM account ... -> 0 rows` ก็เข้าใจทันทีว่าไม่มีข้อมูลในฐานข้อมูล ไม่ใช่บั๊กของโค้ด

### 2.2.5 Custom Context และ Extra Data

```java
Sentry.configureScope(scope -> {
    // Context แบบมีโครงสร้าง (แนะนำ) - แสดงเป็นกล่องแยกในหน้า Issue
    scope.setContexts("business_transaction", Map.of(
        "orderNo",     order.getOrderNo(),
        "orderStatus", order.getStatus(),
        "itemCount",   order.getItems().size(),
        "totalAmount", order.getTotalAmount(),          // ตัวเลขรวม ไม่ใช่ PII
        "currency",    "LAK"
    ));

    scope.setContexts("system_state", Map.of(
        "dbPoolActive",  hikariPool.getActiveConnections(),
        "dbPoolIdle",    hikariPool.getIdleConnections(),
        "queueDepth",    taskQueue.size()
    ));

    // Extra แบบเก่า (ใช้ได้ แต่แนะนำให้ย้ายไป Contexts)
    scope.setExtra("retryCount", "2");
});
```

> ⚠️ **ขนาดข้อมูล:** Sentry จำกัดขนาด event ประมาณ 1 MB (หลังบีบอัด) อย่าใส่ payload ทั้งก้อนลงไป ให้ใส่แค่ **สรุป** เช่น จำนวนรายการ ยอดรวม สถานะ ไม่ใช่ JSON ทั้งชุด

### 2.2.6 การจับ Error ด้วยตนเอง

```java
import io.sentry.Sentry;
import io.sentry.SentryLevel;
import io.sentry.protocol.SentryId;

// --- 1) captureException: จับ exception ที่เรา handle เอง ---
try {
    coreBankingClient.getBalance(accountNo);
} catch (RestClientException e) {
    SentryId eventId = Sentry.captureException(e);
    log.warn("core banking ไม่ตอบ eventId={}", eventId);
    return BalanceResult.unavailable();     // ระบบยังทำงานต่อได้
}

// --- 2) captureMessage: ส่งข้อความ ไม่ใช่ exception ---
Sentry.captureMessage("รอบปิดสิ้นวันใช้เวลานานผิดปกติ", SentryLevel.WARNING);

// --- 3) withScope: เพิ่มบริบทเฉพาะ event นี้ (ไม่กระทบ event อื่น) ---
Sentry.withScope(scope -> {
    scope.setLevel(SentryLevel.FATAL);
    scope.setTag("module", "eod-batch");
    scope.setTag("batch_id", batchId);
    scope.setContexts("batch", Map.of(
        "recordsProcessed", processed,
        "recordsFailed", failed,
        "durationMs", duration
    ));
    scope.setFingerprint(List.of("eod-batch-failure"));
    Sentry.captureException(e);
});

// --- 4) เชื่อม eventId กลับหาผู้ใช้ (สำคัญมากสำหรับ Support) ---
SentryId eventId = Sentry.captureException(e);
return ResponseEntity.status(500).body(new ApiError(
    "INTERNAL_ERROR",
    "เกิดข้อผิดพลาด กรุณาแจ้งรหัสอ้างอิงนี้กับฝ่ายสนับสนุน",
    eventId.toString()          // ผู้ใช้เห็นรหัส -> Support ค้นใน Sentry ได้ทันที
));
```

> ✅ **เทคนิคที่ใช้ได้ผลมากในองค์กร:** แสดง `eventId` บนหน้าจอ error ของผู้ใช้ เมื่อผู้ใช้โทรแจ้งปัญหาและอ่านรหัสให้ฟัง ทีม Support ค้นใน Sentry ด้วยรหัสนั้นแล้วเจอทันทีว่าเกิดอะไรขึ้น ประหยัดเวลาการสอบถามไปมาได้มหาศาล

---

### 🧪 Lab 2.2 - เติมบริบทให้ Error ในระบบงานจริง

> **เป้าหมาย:** ทำให้ Issue ของ `GET /api/customers/{id}/statement` มีข้อมูลครบพอที่จะแก้บั๊กได้โดยไม่ต้องถามใคร

**ขั้นที่ 1** เพิ่ม `SentryUserContextFilter` ตามหัวข้อ 2.2.2

**ขั้นที่ 2** เพิ่ม tag ทางธุรกิจใน `CustomerService` ตามหัวข้อ 2.2.3

**ขั้นที่ 3** เพิ่ม breadcrumb ใน `CustomerStatementService`:

```java
Sentry.addBreadcrumb("เริ่มดึงบัญชีหลักของลูกค้า", "statement");
Account account = accountRepo.findPrimary(customerId);
Breadcrumb b = new Breadcrumb();
b.setCategory("statement");
b.setMessage("ผลการค้นหาบัญชีหลัก");
b.setData("found", account != null);
b.setData("customerId", customerId);
Sentry.addBreadcrumb(b);
```

**ขั้นที่ 4** ยิงทดสอบพร้อม header:

```bash
curl -i http://localhost:8080/api/customers/4999/statement \
  -H 'X-Staff-Id: STF-0512' \
  -H 'X-Branch-Code: HQ-001'

curl -i http://localhost:8080/api/customers/4998/statement \
  -H 'X-Staff-Id: STF-0777' \
  -H 'X-Branch-Code: BR-012'
```

**ขั้นที่ 5** เปิด Issue ใน Sentry แล้วตรวจว่ามีครบตามตาราง:

| สิ่งที่ต้องเห็น | อยู่ตรงไหน |
| --- | --- |
| "2 users affected" | ส่วนหัวของ Issue |
| tag `branch_code` มี HQ-001 และ BR-012 | แถบ Tags ด้านขวา |
| tag `customer_segment` | แถบ Tags |
| breadcrumb "ผลการค้นหาบัญชีหลัก" พร้อม `found: false` | ส่วน Breadcrumbs |
| ไม่มีชื่อจริง อีเมล หรือเบอร์โทรของลูกค้า | ตรวจทุกส่วน |

**ขั้นที่ 6** ใช้ Search Query กรองดู:

```
customer_segment:PREMIER is:unresolved
branch_code:HQ-001
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** สามารถตอบได้ทันทีว่า "ปัญหานี้เกิดกับพนักงาน 2 คน จาก 2 สาขา และสาเหตุคือลูกค้าไม่มีบัญชีหลักในระบบ"

---

## 📚 Module 2.3: แนวคิด Performance Monitoring และ Distributed Tracing

### เวลา 13:00–13:45 น.

> 💡 **หัวใจของ Module นี้:** ก่อนจะเปิดสวิตช์ Performance Monitoring เราต้องเข้าใจโครงสร้างข้อมูลของมันก่อน มิฉะนั้นจะได้กราฟสวย ๆ ที่อ่านไม่รู้เรื่อง หรือแย่กว่านั้นคือสิ้นเปลืองทรัพยากรโดยไม่ได้ประโยชน์

---

### 2.3.1 Trace, Transaction และ Span

```
TRACE (1 การกระทำของผู้ใช้ ครอบคลุมทุกบริการ)
trace_id = 4bf92f3577b34da6a3ce929d0e0e4736
│
├── TRANSACTION [frontend] "/customers/4999"                     2,410 ms
│   │   op: navigation   project: bcel-crm-frontend
│   │
│   ├── SPAN  ui.angular.init          "CustomerDetailComponent"    35 ms
│   ├── SPAN  resource.script          "main-4f2a.js"              120 ms
│   └── SPAN  http.client              "GET /api/customers/4999/summary"  2,180 ms
│                    │
│                    │  ส่ง header: sentry-trace, baggage
│                    v
└── TRANSACTION [backend] "GET /api/customers/{id}/summary"       2,150 ms
    │   op: http.server   project: bcel-crm-backend
    │
    ├── SPAN  db.query   "SELECT * FROM customer WHERE id = ?"        8 ms
    ├── SPAN  db.query   "SELECT * FROM account WHERE customer_id=?"  6 ms
    ├── SPAN  db.query   "SELECT ... FROM transaction_log WHERE ..." 12 ms  ┐
    ├── SPAN  db.query   "SELECT ... FROM transaction_log WHERE ..." 11 ms  │
    ├── SPAN  db.query   "SELECT ... FROM transaction_log WHERE ..." 14 ms  ├ N+1 !
    │   ... (ซ้ำอีก 12 ครั้ง) ...                                          │
    └── SPAN  db.query   "SELECT ... FROM transaction_log WHERE ..." 13 ms  ┘
```

| แนวคิด | ความหมาย | ตัวอย่าง |
| --- | --- | --- |
| **Trace** | เส้นทางทั้งหมดของ 1 การกระทำ ข้ามทุกบริการ | ผู้ใช้เปิดหน้ารายละเอียดลูกค้า |
| **Transaction** | หน่วยงานหลักภายใน 1 บริการ = root span | `GET /api/customers/{id}/summary` |
| **Span** | งานย่อยภายใน Transaction | 1 SQL query, 1 HTTP call |
| **op** | ประเภทของงาน | `http.server`, `db.query`, `navigation`, `ui.render` |
| **trace_id** | รหัสที่ผูกทุก transaction เข้าด้วยกัน | ค่า 32 hex characters |

> ✅ **สิ่งที่ต้องจำ:** ในภาพข้างบน Metrics บอกได้แค่ "P95 = 2.4 วินาที" แต่ Trace บอกได้ว่า **"2.15 วินาทีจาก 2.41 วินาที หมดไปกับ backend และในนั้นมี query ซ้ำ 15 ครั้ง"** นี่คือความต่างที่จับต้องได้

### 2.3.2 ตัวชี้วัดที่ต้องดูเป็น

| ตัวชี้วัด | ความหมาย | ค่าอ้างอิงสำหรับระบบ CRM ภายใน |
| --- | --- | --- |
| **Throughput (TPM)** | จำนวน transaction ต่อนาที | ใช้ดูว่าโหลดปกติเป็นเท่าไร |
| **P50 (median)** | ครึ่งหนึ่งของ request เร็วกว่านี้ | < 300 ms |
| **P75** | 75% ของ request เร็วกว่านี้ | < 800 ms |
| **P95** | 95% เร็วกว่านี้ ⭐ ตัวหลักที่ควรใช้ | < 2,000 ms |
| **P99** | ไล่ล่า outlier | < 5,000 ms |
| **Failure Rate** | สัดส่วน transaction ที่ status ไม่ใช่ ok | < 1% |
| **Apdex** | คะแนนความพึงพอใจ 0–1 | > 0.9 |

```
ทำไมต้องใช้ P95 ไม่ใช่ค่าเฉลี่ย:

  ค่าเฉลี่ย = 400 ms   ← "ดูดีนะ"
  แต่กระจายจริงเป็น:
    ├── 90% ของ request:  120 ms   😊
    └── 10% ของ request: 3,200 ms  😱  ⟵ กลุ่มนี้คือคนที่โทรมาบ่น
  P95 = 3,100 ms  ← ตัวเลขที่สะท้อนความจริง
```

> ⚠️ **กับดักคลาสสิก:** อย่ารายงานผู้บริหารด้วย "ค่าเฉลี่ย" เพราะ outlier ไม่กี่ตัวจะถูกกลบหายไป ให้ใช้ P95 เป็นตัวหลักเสมอ

### 2.3.3 Distributed Tracing และ Trace Propagation

Trace เชื่อมข้ามบริการได้ด้วย HTTP header 2 ตัว:

```http
GET /api/customers/4999/summary HTTP/1.1
Host: crm-api.bcel.local

sentry-trace: 4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-1
              └────────── trace_id ──────────┘ └── parent span ─┘ └ sampled

baggage: sentry-trace_id=4bf92f3577b34da6a3ce929d0e0e4736,
         sentry-public_key=a1b2c3d4e5f6,
         sentry-release=bcel-crm-frontend@1.0.0,
         sentry-environment=development,
         sentry-sample_rate=1.0,
         sentry-transaction=%2Fcustomers%2F4999
```

| Header | หน้าที่ |
| --- | --- |
| `sentry-trace` | บอก trace_id, parent span_id และการตัดสินใจ sampling |
| `baggage` | ส่งข้อมูลบริบท (release, environment, sample rate) ตาม W3C Baggage spec |

```
ขั้นตอนการเชื่อม Trace:

[Angular]                              [Spring Boot]
   │
   │ 1. สร้าง trace_id ใหม่
   │ 2. ตัดสินใจ sampling (sampled=1)
   │
   ├── HTTP GET + sentry-trace header ──>│
   │                                      │ 3. อ่าน header เจอ trace_id
   │                                      │ 4. ใช้ trace_id เดิม (ไม่สร้างใหม่)
   │                                      │ 5. เคารพการตัดสินใจ sampling ของต้นทาง
   │                                      │
   │                                      ├── SQL spans ต่าง ๆ
   │<── 200 OK ──────────────────────────┤
   │
   └─> ทั้งสอง transaction ผูกกันด้วย trace_id เดียวกันใน Sentry ✅
```

> ⚠️ **จุดที่ต้องระวังที่สุด: CORS** ถ้า Frontend กับ Backend อยู่คนละ origin เบราว์เซอร์จะบล็อก header `sentry-trace` และ `baggage` เว้นแต่ Backend จะประกาศอนุญาตผ่าน `Access-Control-Allow-Headers`
>
> **อาการเมื่อลืม:** Frontend มี transaction, Backend ก็มี transaction แต่ **trace_id คนละอัน** ทำให้ดู waterfall รวมกันไม่ได้
>
> โค้ด `CorsConfig` ฉบับสมบูรณ์ที่จะใช้จริงในโปรเจกต์อยู่ในหัวข้อ **2.6.3** ท้ายบทเรียนวันนี้

### 2.3.4 กลยุทธ์ Sampling สำหรับ Production

**ทำไมต้อง Sampling:** ถ้าเก็บทุก transaction ระบบที่มี 1,000 req/s จะสร้าง 86 ล้าน transaction ต่อวัน ซึ่งทั้งเปลืองแบนด์วิดท์ เปลืองที่เก็บ และทำให้ Sentry ทำงานหนักเกินจำเป็น

**วิธีที่ 1 - อัตราคงที่:**

```properties
# development: เก็บหมด เพื่อให้เห็นทุกอย่างตอนพัฒนา
sentry.traces-sample-rate=1.0

# staging: เก็บครึ่ง
sentry.traces-sample-rate=0.5

# production: เก็บ 10%
sentry.traces-sample-rate=0.1
```

**วิธีที่ 2 - Dynamic Sampling ด้วย `TracesSamplerCallback` (แนะนำสำหรับ Production):**

```java
// backend/src/main/java/la/com/bcel/crm/config/BcelTracesSampler.java
package la.com.bcel.crm.config;

import io.sentry.SamplingContext;
import io.sentry.SentryOptions;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.stereotype.Component;

/**
 * กลยุทธ์ Sampling ของ BCEL CRM
 *
 * หลักคิด: เก็บมากในเส้นทางที่สำคัญและมักมีปัญหา
 *          เก็บน้อยในเส้นทางที่เรียกบ่อยแต่ไม่ค่อยมีปัญหา
 *          ไม่เก็บเลยในเส้นทางที่ไม่มีคุณค่าทางการวินิจฉัย
 */
@Component
public class BcelTracesSampler implements SentryOptions.TracesSamplerCallback {

    @Override
    public Double sample(SamplingContext context) {
        Object raw = context.getCustomSamplingContext() == null
                ? null
                : context.getCustomSamplingContext().get("request");

        if (!(raw instanceof HttpServletRequest request)) {
            return null;    // คืน null = ใช้ค่า traces-sample-rate ที่ตั้งไว้
        }

        String path = request.getRequestURI();

        // 1) ไม่เก็บเลย: health check และ static resource (ไร้ประโยชน์ เปลืองที่)
        if (path.startsWith("/actuator") || path.startsWith("/static")) {
            return 0.0;
        }

        // 2) เก็บทั้งหมด: รายงานที่รู้อยู่แล้วว่าช้า และงานปิดรอบ
        if (path.startsWith("/api/reports") || path.startsWith("/api/eod")) {
            return 1.0;
        }

        // 3) เก็บมาก: เส้นทางที่กระทบผู้ใช้โดยตรง
        if (path.startsWith("/api/customers") || path.startsWith("/api/tickets")) {
            return 0.3;
        }

        // 4) ที่เหลือใช้ค่าเริ่มต้น
        return 0.05;
    }
}
```

> ⚠️ **กฎเหล็กของ Sampling ในระบบ Distributed:** การตัดสินใจ sampling ต้องเกิดที่ **จุดเริ่มต้นของ Trace** (คือ Frontend) แล้วส่งต่อผ่าน header ถ้า Backend ตัดสินใจเองอิสระ จะได้ Trace ที่ขาดเป็นท่อน ๆ (frontend เก็บ แต่ backend ไม่เก็บ) ซึ่งใช้วิเคราะห์ไม่ได้เลย

> 💡 **ข้อควรรู้:** `traces-sample-rate` **ไม่กระทบ Error** เลย Error ทุกตัวยังถูกส่งครบ 100% เสมอ การ sampling มีผลเฉพาะกับ Transaction/Span เท่านั้น

### 2.3.5 การอ่านหน้า Performance ใน Sentry

```
Performance > Transactions
┌───────────────────────────────────┬────────┬───────┬───────┬──────┬────────┐
│ Transaction                       │  TPM   │  P50  │  P95  │ Fail │ Users  │
├───────────────────────────────────┼────────┼───────┼───────┼──────┼────────┤
│ GET /api/reports/daily-summary    │   2.1  │ 6.2s  │ 9.8s  │ 0.2% │    18  │ 🔴
│ GET /api/customers/{id}/summary   │  48.5  │ 1.9s  │ 2.6s  │ 0.0% │   210  │ 🟠
│ GET /api/customers                │ 132.0  │ 110ms │ 240ms │ 0.1% │   380  │ 🟢
│ POST /api/tickets                 │  12.4  │  85ms │ 190ms │ 2.1% │    95  │ 🟡
└───────────────────────────────────┴────────┴───────┴───────┴──────┴────────┘
```

**ลำดับการวินิจฉัยที่แนะนำ:**

1. เรียงตาม **P95** จากมากไปน้อย → หา endpoint ที่ช้าที่สุด
2. แต่อย่าดูแค่ช้า ให้ดู **TPM × P95** ด้วย = "เวลารวมที่ผู้ใช้เสียไป" เพราะ endpoint ที่ช้า 10 วิแต่เรียกวันละครั้ง อาจสำคัญน้อยกว่า endpoint ที่ช้า 2 วิแต่เรียก 50,000 ครั้ง
3. คลิกเข้าไปดู **Suspect Spans** ที่ Sentry คำนวณให้ว่า span ไหนกินเวลามากสุด
4. เปิด **Sample Events** เลือก event ที่ช้าที่สุด แล้วดู waterfall

**Performance Issues ที่ Sentry ตรวจจับให้อัตโนมัติ:**

| ชนิด | ความหมาย |
| --- | --- |
| **N+1 Queries** | query แบบเดียวกันถูกเรียกซ้ำในลูป ⭐ เจอบ่อยที่สุด |
| **Slow DB Query** | query เดียวใช้เวลาเกินเกณฑ์ |
| **Consecutive DB Queries** | query ต่อกันเป็นทอด ๆ ที่ควรรวมได้ |
| **Uncompressed Asset** | ไฟล์ static ไม่ได้บีบอัด |
| **Large HTTP Payload** | response ขนาดใหญ่ผิดปกติ |
| **Render Blocking Asset** | script ที่บล็อกการแสดงผลหน้าเว็บ |

> ✅ **นี่คือเหตุผลที่หลักสูตรนี้ให้ความสำคัญกับ Performance Monitoring:** Sentry ไม่ได้แค่แสดงตัวเลข แต่ **บอกชื่อปัญหาให้เลย** พร้อมชี้ว่าอยู่บรรทัดไหน

---

## 📚 Module 2.4: Performance Tracing ใน Spring Boot และ MariaDB

### เวลา 13:45–14:45 น.

> 💡 **หัวใจของ Module นี้:** ทำให้ทุก SQL Query ที่วิ่งไปหา MariaDB กลายเป็น Span ที่มองเห็นได้ ซึ่งเป็นเครื่องมือที่ทรงพลังที่สุดในการหาคอขวดของระบบ CRM/ERP

---

### 2.4.1 เปิด Performance Monitoring

```properties
# application.properties
sentry.traces-sample-rate=1.0     # development เก็บทั้งหมดก่อน
```

เพียงเท่านี้ Sentry ก็จะสร้าง Transaction ให้ทุก HTTP request โดยอัตโนมัติผ่าน `SentryTracingFilter` โดยชื่อ Transaction จะเป็นรูปแบบ **`<HTTP method> <Spring MVC route pattern>`**

```java
@RestController
@RequestMapping("/api/customers")
public class CustomerController {

    @GetMapping("/{id}/summary")
    public CustomerSummary summary(@PathVariable Long id) {
        return customerService.getSummary(id);
    }
}
```

> ✅ **Transaction ที่ได้:** `GET /api/customers/{id}/summary` (ใช้ **route pattern** ไม่ใช่ URL จริง) นี่สำคัญมาก เพราะถ้าใช้ URL จริงจะได้ transaction ชื่อ `/api/customers/4999/summary`, `/api/customers/5000/summary` แยกกันเป็นพันชื่อ ทำให้ดูสถิติไม่ได้เลย

### 2.4.2 Instrument HTTP Client ขาออก

ถ้าระบบ CRM ต้องเรียก Core Banking ต่อ ให้สร้าง client ผ่าน builder เพื่อให้ Sentry ใส่ instrumentation ให้:

```java
// backend/src/main/java/la/com/bcel/crm/config/HttpClientConfig.java
@Configuration
public class HttpClientConfig {

    /** RestTemplate ที่สร้างจาก builder จะได้ SentrySpanRestTemplateCustomizer อัตโนมัติ */
    @Bean
    RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
                .setConnectTimeout(Duration.ofSeconds(3))
                .setReadTimeout(Duration.ofSeconds(10))
                .build();
    }

    /** RestClient (Spring 6.1+) ก็ได้รับ instrumentation เช่นกัน */
    @Bean
    RestClient restClient(RestClient.Builder builder) {
        return builder.baseUrl("https://core-banking.bcel.local").build();
    }
}
```

> ⚠️ **ถ้าสร้างด้วย `new RestTemplate()` ตรง ๆ จะไม่มี Span** เพราะ Sentry ฉีด customizer ผ่าน builder เท่านั้น นี่เป็นสาเหตุที่พบบ่อยว่า "ทำไมไม่เห็น span ของการเรียก API ภายนอก"

### 2.4.3 `@SentryTransaction` และ `@SentrySpan`

**เตรียมความพร้อม** ต้องมี `spring-boot-starter-aop` (เพิ่มไปแล้วในวันที่ 1)

```java
// backend/src/main/java/la/com/bcel/crm/report/EodBatchJob.java
package la.com.bcel.crm.report;

// ⚠️ Spring Boot 3 ต้องใช้ namespace jakarta
import io.sentry.spring.jakarta.tracing.SentrySpan;
import io.sentry.spring.jakarta.tracing.SentryTransaction;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class EodBatchJob {

    /**
     * งานที่ไม่ได้เกิดจาก HTTP request จะไม่มี Transaction ให้เอง
     * ต้องประกาศด้วย @SentryTransaction
     */
    @Scheduled(cron = "0 30 23 * * *")
    @SentryTransaction(operation = "task", name = "eod.daily-summary")
    public void runDailySummary() {
        List<Branch> branches = loadBranches();
        for (Branch branch : branches) {
            summarizeBranch(branch);
        }
        writeReportFile();
    }

    /** แต่ละเมธอดกลายเป็น Span ย่อยใน Transaction ข้างบน */
    @SentrySpan(operation = "task.branch-summary")
    void summarizeBranch(Branch branch) {
        // ...
    }

    @SentrySpan(operation = "task.write-file")
    void writeReportFile() {
        // ...
    }
}
```

**สร้าง Span ด้วยมือเมื่อต้องการควบคุมละเอียด:**

```java
import io.sentry.ISpan;
import io.sentry.Sentry;
import io.sentry.SpanStatus;

public CustomerSummary getSummary(Long customerId) {
    // ดึง transaction/span ปัจจุบันที่ SentryTracingFilter สร้างไว้
    ISpan parent = Sentry.getSpan();
    ISpan span = (parent == null)
            ? null
            : parent.startChild("business.calculate", "คำนวณสรุปยอดลูกค้า");

    try {
        CustomerSummary result = doCalculate(customerId);
        if (span != null) {
            span.setData("accountCount", result.getAccounts().size());
            span.setData("segment", result.getSegment());
            span.setStatus(SpanStatus.OK);
        }
        return result;
    } catch (Exception e) {
        if (span != null) {
            span.setThrowable(e);
            span.setStatus(SpanStatus.INTERNAL_ERROR);
        }
        throw e;
    } finally {
        // ⭐ ต้อง finish() เสมอ มิฉะนั้น span จะไม่ปรากฏใน Sentry
        if (span != null) {
            span.finish();
        }
    }
}
```

> ⚠️ **ข้อผิดพลาดที่พบบ่อยที่สุดกับ Span:** ลืมเรียก `finish()` ผลคือ span หายไปเฉย ๆ ไม่มี error แจ้ง ให้ใช้ `try-finally` เสมอ

### 2.4.4 ผูก `sentry-jdbc` เพื่อเห็นทุก Query เป็น Span

**ขั้นที่ 1 - เพิ่ม dependency**

```xml
<!-- backend/pom.xml (เวอร์ชันมาจาก sentry-bom ที่ตั้งไว้วันที่ 1) -->
<dependency>
    <groupId>io.sentry</groupId>
    <artifactId>sentry-jdbc</artifactId>
</dependency>
```

**ขั้นที่ 2 - เปลี่ยน DataSource ให้ผ่าน P6Spy**

`sentry-jdbc` ทำงานโดยการเป็น **listener ของ P6Spy** ซึ่งเป็น proxy driver ที่คั่นกลางระหว่างแอปกับ JDBC driver จริง

```properties
# application.properties

# ❌ เดิม
# spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
# spring.datasource.url=jdbc:mariadb://localhost:3306/bcel_crm

# ✅ ใหม่: ใช้ P6Spy เป็น driver แล้วใส่ prefix p6spy: ใน URL
#    MariaDB รันอยู่ในเครื่องของท่านเอง จึงใช้ localhost
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
spring.datasource.url=jdbc:p6spy:mariadb://localhost:3306/bcel_crm?sslMode=disable&timezone=Asia/Vientiane
spring.datasource.username=${DB_USER:crm_app}
spring.datasource.password=${DB_PASSWORD:labpass123}
```

> 💡 **สังเกตว่า prefix `jdbc:p6spy:` แทรกอยู่ระหว่าง `jdbc:` กับ `mariadb:`** ส่วนที่เหลือของ URL รวมทั้ง host, port, database และ query parameter ยังเหมือนเดิมทุกประการ

```
กลไกการทำงาน:

  Spring Data JPA / Hibernate
          │
          v
  P6SpyDriver  ──────> SentryJdbcEventListener  ──> สร้าง Span "db.query"
          │                                          พร้อม description = SQL
          v
  org.mariadb.jdbc.Driver  ──────> MariaDB
```

**ขั้นที่ 3 - ปิดการเขียน log file ของ P6Spy (สำคัญมาก)**

โดยปริยาย P6Spy จะเขียนทุก query ลงไฟล์ `spy.log` ซึ่งจะโตเร็วมากจนดิสก์เต็มได้ใน production

```properties
# backend/src/main/resources/spy.properties
# เปิดเฉพาะโมดูลหลัก = ไม่โหลดโมดูล logging จึงไม่เขียนไฟล์ spy.log
modulelist=com.p6spy.engine.spy.P6SpyFactory
```

> ⚠️ **ประเด็นความเป็นส่วนตัวที่ต้องพิจารณา:** span ที่ `sentry-jdbc` สร้างจะมี **ข้อความ SQL** เป็น description ซึ่งโดยปกติยังคงเครื่องหมาย `?` ไว้ (ไม่ใช่ค่าจริง) แต่ถ้ามี native query ที่ต่อสตริงค่าเข้าไปเอง ค่าจริงจะหลุดเข้า Sentry ได้ ให้ตรวจสอบว่าโค้ดทุกที่ใช้ **prepared statement / named parameter** เท่านั้น และเพิ่มกฎ Advanced Data Scrubbing ที่ระดับโปรเจกต์ไว้อีกชั้นหนึ่ง

หรือกำหนดผ่าน JVM system property:

```bash
java -Dp6spy.config.modulelist=com.p6spy.engine.spy.P6SpyFactory -jar app.jar
```

**ขั้นที่ 4 - ระวังเมื่อใช้ Docker Compose support ของ Spring Boot**

> ⚠️ **ข้อนี้สำคัญมากในห้อง Lab** เพราะเรารัน MariaDB ด้วย `docker compose` บนเครื่องตัวเอง ถ้าโปรเจกต์เผลอเปิดใช้ `spring-boot-docker-compose` ไว้ span ของ database จะหายทันที ให้ตรวจว่าไม่มี dependency ตัวนี้ใน `pom.xml`
>
> ถ้าโปรเจกต์ใช้ `spring-boot-docker-compose` ตัว extension นี้จะ **เขียนทับ** `spring.datasource.url` ทำให้ prefix `jdbc:p6spy:` หายไป และ span ของ database จะไม่ขึ้น วิธีแก้คือปิด extension นี้ในสภาพแวดล้อมที่ต้องการ tracing:
>
> ```properties
> spring.docker.compose.enabled=false
> ```

**ผลลัพธ์ที่ได้:**

```
Transaction: GET /api/customers/{id}/summary        2,150 ms
├─ db.query  SELECT c.* FROM customer c WHERE c.id = ?               8 ms
├─ db.query  SELECT a.* FROM account a WHERE a.customer_id = ?       6 ms
├─ db.query  SELECT t.* FROM transaction_log t WHERE t.account_id=? 12 ms
├─ db.query  SELECT t.* FROM transaction_log t WHERE t.account_id=? 11 ms
├─ ... (ซ้ำอีก 13 ครั้ง)
└─ db.query  SELECT t.* FROM transaction_log t WHERE t.account_id=? 13 ms

⚠️ Sentry ตรวจพบ: "N+1 Query" - repeating db.query 15 times
```

### 2.4.5 วินิจฉัยปัญหา N+1 Query

**โค้ดที่มีปัญหา (ตั้งใจใส่ไว้ใน BCEL CRM Lite):**

```java
// ❌ ต้นเหตุของ N+1
@Service
public class CustomerSummaryService {

    public CustomerSummary build(Long customerId) {
        Customer customer = customerRepo.findById(customerId).orElseThrow();

        // query ที่ 1: ดึงบัญชีทั้งหมด
        List<Account> accounts = accountRepo.findByCustomerId(customerId);

        List<AccountSummary> summaries = new ArrayList<>();
        for (Account account : accounts) {
            // 💥 query ที่ 2..N+1: วนดึงรายการเคลื่อนไหวทีละบัญชี
            List<TransactionLog> logs =
                    transactionRepo.findTop10ByAccountIdOrderByTxDateDesc(account.getId());
            summaries.add(AccountSummary.of(account, logs));
        }
        return new CustomerSummary(customer, summaries);
    }
}
```

**วิธีแก้ที่ 1 - ดึงทีเดียวด้วย `IN` clause:**

```java
// ✅ query 2 ครั้งจบ ไม่ว่าจะมีกี่บัญชี
public CustomerSummary build(Long customerId) {
    Customer customer = customerRepo.findById(customerId).orElseThrow();
    List<Account> accounts = accountRepo.findByCustomerId(customerId);

    List<Long> accountIds = accounts.stream().map(Account::getId).toList();

    // ดึงทีเดียวทั้งหมด แล้วจัดกลุ่มในหน่วยความจำ
    Map<Long, List<TransactionLog>> logsByAccount =
            transactionRepo.findRecentByAccountIds(accountIds)
                           .stream()
                           .collect(Collectors.groupingBy(TransactionLog::getAccountId));

    List<AccountSummary> summaries = accounts.stream()
            .map(a -> AccountSummary.of(a, logsByAccount.getOrDefault(a.getId(), List.of())))
            .toList();

    return new CustomerSummary(customer, summaries);
}
```

```java
// Repository
@Query("""
       SELECT t FROM TransactionLog t
       WHERE t.accountId IN :accountIds
         AND t.txDate >= :since
       ORDER BY t.txDate DESC
       """)
List<TransactionLog> findRecentByAccountIds(
        @Param("accountIds") List<Long> accountIds,
        @Param("since") LocalDateTime since);
```

**วิธีแก้ที่ 2 - ใช้ `JOIN FETCH` (กรณีเป็น JPA relation):**

```java
@Query("SELECT DISTINCT c FROM Customer c LEFT JOIN FETCH c.accounts WHERE c.id = :id")
Optional<Customer> findByIdWithAccounts(@Param("id") Long id);
```

**วิธีแก้ที่ 3 - ตั้ง batch fetch ของ Hibernate:**

```properties
spring.jpa.properties.hibernate.default_batch_fetch_size=50
```

> ✅ **ผลลัพธ์ที่วัดได้จริง:** ในระบบจำลอง ลูกค้าที่มี 15 บัญชี จะลดจาก **16 queries / 2,150 ms** เหลือ **2 queries / ~180 ms** คือเร็วขึ้นประมาณ 12 เท่า

### 2.4.6 วินิจฉัย Slow Query

**โค้ดที่มีปัญหา:**

```java
// ❌ ใช้ฟังก์ชันครอบคอลัมน์ ทำให้ MariaDB ใช้ index ไม่ได้
@Query(value = """
       SELECT branch_code, COUNT(*) AS cnt, SUM(amount) AS total
       FROM transaction_log
       WHERE DATE(tx_date) = :reportDate
       GROUP BY branch_code
       """, nativeQuery = true)
List<Object[]> summarizeByDate(@Param("reportDate") LocalDate reportDate);
```

**Span ที่เห็นใน Sentry:**

```
db.query  SELECT branch_code, COUNT(*)... WHERE DATE(tx_date) = ?     8,240 ms 🔴
```

**ยืนยันสาเหตุด้วย EXPLAIN บน MariaDB:**

```sql
EXPLAIN SELECT branch_code, COUNT(*), SUM(amount)
FROM transaction_log
WHERE DATE(tx_date) = '2026-07-29'
GROUP BY branch_code;

-- ผลลัพธ์:
-- type: ALL          <- full table scan สแกน 800,000 แถว 💥
-- key:  NULL         <- ไม่ใช้ index เลย
-- rows: 800000
```

**วิธีแก้ - ใช้ช่วงเวลาแทนการครอบฟังก์ชัน:**

```java
// ✅ ใช้ช่วง [start, end) ทำให้ index ของ tx_date ถูกใช้งาน
@Query(value = """
       SELECT branch_code, COUNT(*) AS cnt, SUM(amount) AS total
       FROM transaction_log
       WHERE tx_date >= :start AND tx_date < :end
       GROUP BY branch_code
       """, nativeQuery = true)
List<Object[]> summarizeByRange(@Param("start") LocalDateTime start,
                                @Param("end") LocalDateTime end);
```

```sql
-- และสร้าง index รองรับ
CREATE INDEX idx_txlog_date_branch ON transaction_log (tx_date, branch_code);

-- ตรวจซ้ำ
EXPLAIN SELECT ... WHERE tx_date >= '2026-07-29 00:00:00' AND tx_date < '2026-07-30 00:00:00' ...;
-- type: range       <- ใช้ index แล้ว ✅
-- key:  idx_txlog_date_branch
-- rows: 3200
```

> ✅ **ผลลัพธ์:** 8,240 ms → ประมาณ 120 ms

**ตารางสรุปสาเหตุ Slow Query ที่พบบ่อยในระบบ CRM/ERP:**

| อาการใน Span | สาเหตุที่พบบ่อย | วิธีแก้ |
| --- | --- | --- |
| query เดียวช้ามาก | ไม่มี index / ใช้ฟังก์ชันครอบคอลัมน์ | สร้าง index, เขียน where ใหม่ |
| query สั้นแต่ซ้ำเยอะ | N+1 | JOIN FETCH / IN clause / batch fetch |
| query ช้าเป็นบางช่วง | Lock contention / โหลดสูง | ตรวจ `SHOW ENGINE INNODB STATUS` |
| query ช้าตอนข้อมูลโต | ไม่ได้แบ่ง partition | partition ตามเดือน |
| `SELECT *` กับตารางกว้าง | ดึงคอลัมน์เกินจำเป็น | ระบุคอลัมน์ / ใช้ projection |

---

## 🛠️ Workshop 2.1 - ล่าหา Slow Query และ N+1 จากระบบงานจำลอง

### เวลา (รวมอยู่ในช่วง 13:45–14:45 น.)

> **โจทย์:** ใช้ Sentry Performance หาสาเหตุที่หน้าจอ "รายละเอียดลูกค้า" และ "รายงานสรุปประจำวัน" ช้า แล้วแก้ให้เร็วขึ้น พร้อมพิสูจน์ผลด้วยตัวเลขก่อน-หลัง

### ขั้นที่ 1 - เปิด tracing และ JDBC instrumentation

ทำตามหัวข้อ 2.4.1 และ 2.4.4 แล้วรีสตาร์ทแอป

```bash
mvn clean spring-boot:run
```

### ขั้นที่ 2 - สร้างโหลดจำลอง

```bash
# กดหน้ารายละเอียดลูกค้า 30 ครั้ง (ลูกค้าที่มีหลายบัญชี)
for i in $(seq 1 30); do
  curl -s "http://localhost:8080/api/customers/$((RANDOM % 100 + 1))/summary" \
    -H 'X-Staff-Id: STF-0512' -H 'X-Branch-Code: HQ-001' > /dev/null
done

# เรียกรายงานสรุปประจำวัน 5 ครั้ง
for d in 25 26 27 28 29; do
  curl -s "http://localhost:8080/api/reports/daily-summary?date=2026-07-$d" > /dev/null
done

# เรียก endpoint ปกติเพื่อใช้เปรียบเทียบ
for i in $(seq 1 50); do
  curl -s "http://localhost:8080/api/customers?page=0&size=20" > /dev/null
done
```

### ขั้นที่ 3 - บันทึกค่าตั้งต้น (Baseline)

เปิด **Performance → Transactions** แล้วบันทึกลงตาราง:

| Transaction | TPM | P50 | P95 | จำนวน span db.query สูงสุด |
| --- | --- | --- | --- | --- |
| `GET /api/customers/{id}/summary` | | | | |
| `GET /api/reports/daily-summary` | | | | |
| `GET /api/customers` | | | | |

### ขั้นที่ 4 - เจาะดู Waterfall

1. คลิก `GET /api/customers/{id}/summary`
2. ดูส่วน **Suspect Spans** ว่า Sentry ชี้ไปที่ span ไหน
3. เลื่อนลงไปที่ **Sample Events** เลือก event ที่ P95
4. ดู waterfall แล้วตอบ:
   - มี span `db.query` ทั้งหมดกี่อัน
   - span ที่ซ้ำมี SQL ว่าอะไร
   - เวลารวมของ span ที่ซ้ำคิดเป็นกี่ % ของ transaction

5. ไปที่ **Issues** แล้วกรอง `issue.category:performance` ดูว่า Sentry สร้าง Performance Issue ให้ไหม

### ขั้นที่ 5 - ยืนยันด้วย EXPLAIN บนฐานข้อมูลจริง

เชื่อมเข้าฐานข้อมูลที่รันอยู่ในเครื่องของท่าน

```bash
cd bcel-crm-lite
docker compose exec mariadb mariadb -u crm_app -plabpass123 bcel_crm
```

> 💡 **ถ้าถนัด GUI มากกว่า** ใช้ **DBeaver** (ฟรี ข้ามแพลตฟอร์ม) หรือปลั๊กอิน **Database Client** ของ VS Code เชื่อมไปที่ `localhost:3306` ได้เหมือนกัน

```sql
-- ตรวจ query ของรายงาน
EXPLAIN SELECT branch_code, COUNT(*), SUM(amount)
FROM transaction_log
WHERE DATE(tx_date) = '2026-07-29'
GROUP BY branch_code;

-- ดู index ที่มีอยู่
SHOW INDEX FROM transaction_log;

-- ดู query ที่กำลังทำงานอยู่
SHOW PROCESSLIST;

-- ดูการตั้งค่า slow query log
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
```

> 💡 **MariaDB รันอยู่ในเครื่องของท่านเอง** จึงมีสิทธิ์เต็มทุกอย่าง ทดลอง `CREATE INDEX`, `DROP INDEX`, แก้ config ได้อิสระโดยไม่กระทบใคร ถ้าอยากเริ่มใหม่ทั้งหมดสั่ง `docker compose down -v && docker compose up -d mariadb` (จะ seed ข้อมูลใหม่ใช้เวลา 3-5 นาที)

### ขั้นที่ 6 - ลงมือแก้

**แก้ที่ 1 - N+1 ใน `CustomerSummaryService`** ตามหัวข้อ 2.4.5

**แก้ที่ 2 - Slow query ในรายงาน** ตามหัวข้อ 2.4.6 พร้อมสร้าง index:

```sql
CREATE INDEX idx_txlog_date_branch ON transaction_log (tx_date, branch_code);
CREATE INDEX idx_txlog_account_date ON transaction_log (account_id, tx_date);
```

### ขั้นที่ 7 - วัดผลใหม่และเปรียบเทียบ

```bash
# ตั้ง release ใหม่เพื่อแยกข้อมูลก่อน-หลังให้ชัด
export SENTRY_RELEASE='bcel-crm-backend@1.1.0'
mvn clean spring-boot:run
```

รันโหลดจำลองซ้ำตามขั้นที่ 2 แล้วเปรียบเทียบใน Sentry ด้วยตัวกรอง `release:bcel-crm-backend@1.1.0`

| Transaction | P95 ก่อน | P95 หลัง | จำนวน query ก่อน | หลัง | เร็วขึ้น (เท่า) |
| --- | --- | --- | --- | --- | --- |
| `/api/customers/{id}/summary` | | | | | |
| `/api/reports/daily-summary` | | | | | |

> ✅ **เกณฑ์ผ่าน Workshop 2.1:**
> - [ ] เห็น span `db.query` ในหน้า Trace ได้จริง
> - [ ] ระบุได้ว่า N+1 เกิดที่เมธอดใด บรรทัดใด
> - [ ] แก้แล้ว P95 ของ `/summary` ลดลงอย่างน้อย 50%
> - [ ] แก้แล้ว P95 ของ `/daily-summary` ลดลงอย่างน้อย 80%
> - [ ] อธิบายได้ว่าทำไม `DATE(tx_date)` ทำให้ index ใช้ไม่ได้

---

## 📚 Module 2.5: Instrument ระบบ Frontend ด้วย Angular

### เวลา 14:45–15:30 น.

> 💡 **หัวใจของ Module นี้:** ฝั่ง Frontend คือจุดที่ผู้ใช้สัมผัสจริง ถ้าเรามองไม่เห็นฝั่งนี้ เราจะไม่มีวันรู้ว่าปัญหาที่ผู้ใช้บ่นนั้นเกิดที่ browser, ที่เครือข่าย หรือที่ backend

---

### 2.5.1 ติดตั้ง `@sentry/angular`

```bash
cd frontend
npm install @sentry/angular --save

# ตรวจเวอร์ชันที่ติดตั้งได้
npm ls @sentry/angular
```

> 📖 **ความเข้ากันได้ของเวอร์ชัน:** `@sentry/angular` รุ่นปัจจุบัน (สาย 10.x) รองรับ **Angular 14 ขึ้นไป** โครงการ BCEL CRM Lite ใช้ Angular 20 จึงใช้รุ่นล่าสุดได้เลย ถ้าโครงการอื่นในองค์กรยังใช้ Angular 12–13 ต้องใช้ `@sentry/angular-ivy@^7` แทน

### 2.5.2 ตั้งค่า `Sentry.init()` ใน `main.ts`

```typescript
// frontend/src/main.ts
import { bootstrapApplication } from '@angular/platform-browser'
import * as Sentry from '@sentry/angular'

import { appConfig } from './app/app.config'
import { AppComponent } from './app/app.component'
import { environment } from './environments/environment'

Sentry.init({
  dsn: environment.sentryDsn,

  // ต้องตรงกับฝั่ง Backend เพื่อให้ Trace เชื่อมกันได้อย่างมีความหมาย
  environment: environment.sentryEnvironment,
  release: environment.sentryRelease,

  // ⚠️ ระบบงานธนาคาร: ปิดการเก็บ PII อัตโนมัติ (IP address, header)
  sendDefaultPii: false,

  integrations: [
    // ติดตาม page load, navigation, http request อัตโนมัติ
    Sentry.browserTracingIntegration(),

    // Session Replay (จะเรียนละเอียดวันที่ 3) - เปิดพร้อม masking เต็มรูปแบบ
    Sentry.replayIntegration({
      maskAllText: true,
      maskAllInputs: true,
      blockAllMedia: true
    })
  ],

  // สัดส่วนการเก็บ transaction
  tracesSampleRate: environment.production ? 0.1 : 1.0,

  // ⭐ หัวใจของ Distributed Tracing: ระบุ URL ที่ให้แนบ trace header ไปด้วย
  tracePropagationTargets: [
    'localhost',
    /^https:\/\/crm-api\.bcel\.local\/api/,
    /^\/api/
  ],

  // Session Replay sampling
  replaysSessionSampleRate: 0.0,      // ไม่บันทึก session ปกติ
  replaysOnErrorSampleRate: 1.0,      // บันทึกเฉพาะ session ที่มี error

  // กรองข้อมูลก่อนส่งออกจากเบราว์เซอร์
  beforeSend(event) {
    // ตัด query string ที่อาจมีข้อมูลอ่อนไหวออกจาก URL
    if (event.request?.url) {
      event.request.url = event.request.url.split('?')[0]
    }
    // ไม่ส่ง cookie ออกไป
    if (event.request?.cookies) {
      delete event.request.cookies
    }
    return event
  },

  // ไม่ต้องรายงาน error ที่เกิดจาก extension หรือ browser เก่า
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

**ไฟล์ environment:**

```typescript
// frontend/src/environments/environment.ts (development)
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:8080/api',
  // ⚠️ ในห้อง Lab: ใช้ DSN ของ project bcel-crm-frontend ใน org ของท่านเอง
  //    หาได้ที่ Project Settings -> Client Keys (DSN)
  //    รูปแบบ https://<key>@o<org-id>.ingest.de.sentry.io/<project-id>
  sentryDsn: 'https://<public-key>@sentry.bcel.local/3',
  sentryEnvironment: 'development',
  sentryRelease: 'bcel-crm-frontend@1.0.0'
}
```

> 🖥️ **หมายเหตุห้อง Lab:** ตลอดวันที่ 2 เอกสารจะเขียน DSN แบบ Self-hosted (`@sentry.bcel.local/3`) ซึ่งเป็นรูปแบบที่ BCEL จะใช้จริง **ในห้อง Lab ให้แทนที่ด้วย DSN ของ organization ท่านบน sentry.io ทุกครั้ง** ส่วนอื่นของโค้ดไม่ต้องแก้เลย

```typescript
// frontend/src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiBaseUrl: 'https://crm-api.bcel.local/api',
  // ค่าเหล่านี้จะถูก Jenkins แทนที่ตอน build (เรียนวันที่ 3)
  sentryDsn: '__SENTRY_DSN__',
  sentryEnvironment: 'production',
  sentryRelease: '__SENTRY_RELEASE__'
}
```

### 2.5.3 ลงทะเบียน Provider ใน `app.config.ts`

```typescript
// frontend/src/app/app.config.ts
import {
  ApplicationConfig,
  ErrorHandler,
  inject,
  provideAppInitializer,
  provideZoneChangeDetection
} from '@angular/core'
import { provideRouter, Router } from '@angular/router'
import { provideHttpClient, withInterceptors } from '@angular/common/http'
import * as Sentry from '@sentry/angular'

import { routes } from './app.routes'
import { apiInterceptor } from './core/api.interceptor'

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withInterceptors([apiInterceptor])),

    // 1) แทนที่ ErrorHandler มาตรฐานของ Angular ด้วยของ Sentry
    {
      provide: ErrorHandler,
      useValue: Sentry.createErrorHandler({
        // แสดง dialog ให้ผู้ใช้กรอกคำอธิบายเมื่อเกิด error (ปิดไว้ก่อน)
        showDialog: false
      })
    },

    // 2) TraceService: ติดตามการเปลี่ยนเส้นทาง (routing) เป็น transaction
    {
      provide: Sentry.TraceService,
      deps: [Router]
    },

    // 3) บังคับให้ TraceService ถูกสร้างขึ้นตอนแอปเริ่ม
    provideAppInitializer(() => {
      inject(Sentry.TraceService)
    })
  ]
}
```

> ⚠️ **ถ้าลืม `provideAppInitializer`** ตัว `TraceService` จะไม่ถูก instantiate ผลคือ transaction ของ navigation จะไม่เกิดขึ้นเลย นี่เป็นปัญหาที่พบบ่อยมาก

**สำหรับโปรเจกต์แบบ NgModule (โค้ดเก่าในองค์กร):**

```typescript
// app.module.ts
@NgModule({
  providers: [
    { provide: ErrorHandler, useValue: Sentry.createErrorHandler() },
    { provide: Sentry.TraceService, deps: [Router] }
  ]
})
export class AppModule {
  // การ inject ผ่าน constructor ทำหน้าที่แทน provideAppInitializer
  constructor(trace: Sentry.TraceService) {}
}
```

### 2.5.4 Component Tracking

```typescript
// frontend/src/app/customers/customer-detail.component.ts
import { Component, inject, OnInit } from '@angular/core'
import { ActivatedRoute } from '@angular/router'
import * as Sentry from '@sentry/angular'
import { TraceClass, TraceMethod } from '@sentry/angular'

import { CustomerService } from './customer.service'
import { CustomerSummary } from './customer.model'

@Component({
  selector: 'app-customer-detail',
  standalone: true,
  templateUrl: './customer-detail.component.html'
})
// วัดเวลา lifecycle hook ทั้งหมดของ component นี้
@TraceClass({ name: 'CustomerDetailComponent' })
export class CustomerDetailComponent implements OnInit {
  private readonly route = inject(ActivatedRoute)
  private readonly customerService = inject(CustomerService)

  summary: CustomerSummary | null = null
  loading = false

  ngOnInit(): void {
    const id = Number(this.route.snapshot.paramMap.get('id'))
    this.loadSummary(id)
  }

  // วัดเวลาเฉพาะเมธอดนี้ เป็น span ย่อย
  @TraceMethod({ name: 'loadCustomerSummary' })
  loadSummary(id: number): void {
    this.loading = true
    this.customerService.getSummary(id).subscribe({
      next: data => {
        this.summary = data
        this.loading = false
      },
      error: err => {
        this.loading = false
        // ส่งเข้า Sentry อย่างชัดเจน พร้อมบริบท
        Sentry.withScope(scope => {
          scope.setTag('module', 'crm.customer.detail')
          scope.setContext('request', { customerId: id })
          Sentry.captureException(err)
        })
      }
    })
  }
}
```

**สร้าง Span เองรอบงานที่สนใจ:**

```typescript
import * as Sentry from '@sentry/angular'

exportToExcel(): void {
  Sentry.startSpan(
    { op: 'ui.export', name: 'ส่งออกรายชื่อลูกค้าเป็น Excel' },
    span => {
      const rows = this.buildRows()
      span?.setAttribute('rowCount', rows.length)
      this.writeWorkbook(rows)
    }
  )
}
```

### 2.5.5 ติดตาม Routing และ Page Load

`browserTracingIntegration()` สร้าง transaction ให้อัตโนมัติ 2 ประเภท:

| ประเภท | op | เกิดเมื่อ | Span ที่มีให้ |
| --- | --- | --- | --- |
| **Page Load** | `pageload` | เปิดหน้าเว็บครั้งแรก | DNS, TCP, TTFB, resource, FCP, LCP |
| **Navigation** | `navigation` | เปลี่ยน route ใน SPA | resolver, component init, HTTP call |

**ตั้งชื่อ transaction ให้เป็น route pattern (สำคัญมาก):**

```typescript
// frontend/src/app/app.routes.ts
export const routes: Routes = [
  { path: 'customers', component: CustomerListComponent },
  // Sentry อ่าน path pattern นี้ ทำให้ transaction ชื่อ "/customers/:id"
  // ไม่ใช่ "/customers/4999", "/customers/5000" แยกกันเป็นพันชื่อ
  { path: 'customers/:id', component: CustomerDetailComponent },
  { path: 'tickets', component: TicketListComponent },
  { path: 'reports', component: ReportComponent }
]
```

### 2.5.6 การจับ Error ฝั่ง Frontend

**ประเภท Error ที่ Sentry จับให้อัตโนมัติ:**

| ประเภท | ตัวอย่าง | จับโดย |
| --- | --- | --- |
| Uncaught exception | `TypeError: Cannot read properties of null` | global handler |
| Unhandled promise rejection | `Promise.reject()` ที่ไม่มี catch | global handler |
| Angular error | error ที่หลุดจาก component/service | `Sentry.createErrorHandler()` |
| HTTP error | 500 จาก API | ต้องเขียนเองใน interceptor |

**Interceptor สำหรับ HTTP Error:**

```typescript
// frontend/src/app/core/api.interceptor.ts
import { HttpErrorResponse, HttpInterceptorFn } from '@angular/common/http'
import { catchError, throwError } from 'rxjs'
import * as Sentry from '@sentry/angular'

export const apiInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      // 4xx ที่เป็นความผิดของผู้ใช้ ไม่ต้องรบกวน Sentry
      const isClientFault = error.status >= 400 && error.status < 500
        && error.status !== 408 && error.status !== 429

      if (!isClientFault) {
        Sentry.withScope(scope => {
          scope.setTag('http_status', String(error.status))
          scope.setTag('module', 'api')
          scope.setContext('http_request', {
            method: req.method,
            // ตัด query string ออกกัน PII หลุด
            url: req.urlWithParams.split('?')[0],
            status: error.status,
            statusText: error.statusText
          })
          scope.setFingerprint([
            'http-error',
            String(error.status),
            req.url.split('?')[0]
          ])
          Sentry.captureException(error)
        })
      }

      return throwError(() => error)
    })
  )
}
```

**ผูกข้อมูลผู้ใช้ฝั่ง Frontend:**

```typescript
// frontend/src/app/core/auth.service.ts
import * as Sentry from '@sentry/angular'

onLoginSuccess(session: StaffSession): void {
  Sentry.setUser({
    id: session.staffId          // ✅ รหัสพนักงานเท่านั้น
    // ❌ ห้าม: email, username ที่เป็นชื่อจริง, ip_address
  })
  Sentry.setTag('branch_code', session.branchCode)
  Sentry.setTag('staff_role', session.role)
}

onLogout(): void {
  Sentry.setUser(null)           // เคลียร์เสมอเมื่อออกจากระบบ
}
```

---

### 🧪 Lab 2.3 - ทดสอบ Frontend SDK

> **เป้าหมาย:** ยืนยันว่า Angular ส่ง Error และ Transaction เข้าโปรเจกต์ `bcel-crm-frontend` ได้

**ขั้นที่ 1** เพิ่มปุ่มทดสอบใน component ใดก็ได้:

```typescript
// customer-list.component.ts
throwTestError(): void {
  throw new Error('BCEL CRM Frontend: ทดสอบ error ตัวแรก')
}

throwInsideSpan(): void {
  Sentry.startSpan({ op: 'test', name: 'ทดสอบ span พร้อม error' }, () => {
    setTimeout(() => {
      throw new Error('BCEL CRM Frontend: error ภายใน span')
    }, 100)
  })
}

callBrokenApi(): void {
  // เรียก endpoint ที่รู้ว่าพัง เพื่อทดสอบ interceptor
  this.http.get('/api/customers/4999/statement').subscribe()
}
```

```html
<button (click)="throwTestError()">ทดสอบ Error</button>
<button (click)="throwInsideSpan()">ทดสอบ Error ใน Span</button>
<button (click)="callBrokenApi()">ทดสอบ API ที่พัง</button>
```

**ขั้นที่ 2** รันแอปแล้วกดปุ่มทั้งสาม

```bash
cd frontend && ng serve
```

**ขั้นที่ 3** ตรวจใน Sentry โปรเจกต์ `bcel-crm-frontend`:

| สิ่งที่ต้องเห็น | หน้า |
| --- | --- |
| Issue จากปุ่มทั้ง 3 | Issues |
| Transaction `pageload` และ `navigation` | Performance |
| Transaction ชื่อ `/customers/:id` ไม่ใช่ `/customers/4999` | Performance |
| tag `branch_code` | Issue detail |

> ⛔ **ถ้าไม่เห็น transaction ของ navigation:** ตรวจว่าใส่ `provideAppInitializer(() => { inject(Sentry.TraceService) })` แล้วหรือยัง

---

## 📚 Module 2.6: เชื่อม Distributed Tracing ระหว่าง Frontend และ Backend

### เวลา 15:30–16:30 น.

> 💡 **หัวใจของ Module นี้:** นี่คือช่วงที่ทุกอย่างมาบรรจบกัน เมื่อทำสำเร็จ เราจะเห็นภาพเดียวที่บอกทุกอย่างตั้งแต่ "ผู้ใช้กดปุ่ม" ไปจนถึง "SQL ตัวไหนช้า"

---

### 2.6.1 กลไกการส่งต่อ Trace

```
[1] ผู้ใช้คลิกลิงก์ "ดูรายละเอียดลูกค้า"
     │
     v
[2] Angular Router เปลี่ยนเส้นทาง
     └─> browserTracingIntegration สร้าง transaction "/customers/:id"
         trace_id = 4bf92f3577b34da6a3ce929d0e0e4736
         sampled  = true
     │
     v
[3] HttpClient เรียก GET /api/customers/4999/summary
     └─> Sentry ตรวจว่า URL ตรงกับ tracePropagationTargets ไหม
         ✅ ตรง -> แนบ header:
            sentry-trace: 4bf92f...4736-00f067aa0ba902b7-1
            baggage: sentry-trace_id=4bf92f...,sentry-environment=development,...
     │
     v
[4] เบราว์เซอร์ส่ง preflight OPTIONS (เพราะเป็น custom header ข้าม origin)
     └─> Backend ต้องตอบ Access-Control-Allow-Headers ที่ครอบคลุม ⚠️
     │
     v
[5] SentryTracingFilter ฝั่ง Spring Boot อ่าน header
     └─> ใช้ trace_id เดิม ไม่สร้างใหม่
         สร้าง transaction "GET /api/customers/{id}/summary" เป็น child
     │
     v
[6] SentryJdbcEventListener สร้าง span "db.query" ทุก query
     │
     v
[7] ทั้งสองฝั่งส่งข้อมูลเข้า Sentry แยกโปรเจกต์กัน
     └─> Sentry ประกอบร่างเป็น Trace เดียวด้วย trace_id ✅
```

### 2.6.2 ตั้งค่า `tracePropagationTargets` ให้ถูกต้อง

```typescript
// frontend/src/main.ts
Sentry.init({
  tracePropagationTargets: [
    'localhost',                              // dev
    /^\/api/,                                 // relative path (แนะนำถ้าใช้ proxy)
    /^https:\/\/crm-api\.bcel\.local\/api/,   // production API
    /^https:\/\/erp-api\.bcel\.local\/api/    // ระบบ ERP
  ]
})
```

| ค่าที่ตั้ง | จะแนบ header ไปที่ | ผลลัพธ์ |
| --- | --- | --- |
| ไม่ตั้งเลย | เฉพาะ same-origin เท่านั้น | Trace ขาดถ้า API อยู่คนละโดเมน ❌ |
| `[/.*/ ]` (ทุก URL) | ทุกที่รวมถึง Google Analytics, CDN | ⛔ **อันตราย** ข้อมูล trace รั่วออกนอกองค์กร |
| ระบุเฉพาะโดเมนของเรา | เฉพาะ API ขององค์กร | ✅ ถูกต้องและปลอดภัย |

> ⛔ **ห้ามเด็ดขาดในระบบธนาคาร:** อย่าใช้ค่า catch-all เพราะ `sentry-trace` และ `baggage` จะถูกส่งไปยังเซิร์ฟเวอร์ภายนอกทุกแห่งที่แอปเรียก ซึ่งเปิดเผยโครงสร้างระบบภายในโดยไม่จำเป็น

### 2.6.3 ตั้งค่า CORS ฝั่ง Spring Boot ให้รองรับ

```java
// backend/src/main/java/la/com/bcel/crm/config/CorsConfig.java
package la.com.bcel.crm.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins(
                        "http://localhost:4200",
                        "https://crm.bcel.local")
                .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                // ⭐ ต้องอนุญาต header ของ Sentry ให้ส่งเข้ามาได้
                .allowedHeaders(
                        "Content-Type", "Authorization",
                        "X-Staff-Id", "X-Branch-Code",
                        "sentry-trace", "baggage")
                .exposedHeaders("sentry-trace", "baggage")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

**ตั้ง `trace-propagation-targets` ฝั่ง Backend ด้วย** (สำหรับกรณีที่ Backend เรียกบริการอื่นต่อ):

```properties
# application.properties
sentry.trace-propagation-targets=localhost,^https://core-banking\\.bcel\\.local/.*$,^https://erp-api\\.bcel\\.local/.*$
```

### 2.6.4 การอ่าน Trace View แบบ End-to-End

```
Trace 4bf92f3577b34da6a3ce929d0e0e4736                    รวม 2,410 ms
──────────────────────────────────────────────────────────────────────────
 [FE] navigation  /customers/:id                          ████████████ 2410ms
 ├─ ui.angular.init  CustomerDetailComponent               █             35ms
 ├─ resource.script  main-4f2a.js                          ██           120ms
 └─ http.client      GET /api/customers/4999/summary        ██████████ 2180ms
    │
    └─ [BE] http.server  GET /api/customers/{id}/summary    █████████  2150ms
       ├─ db.query  SELECT c.* FROM customer WHERE id=?     ▏             8ms
       ├─ db.query  SELECT a.* FROM account WHERE cust=?    ▏             6ms
       ├─ db.query  SELECT t.* FROM transaction_log ...     ▎            12ms  ┐
       ├─ db.query  SELECT t.* FROM transaction_log ...     ▎            11ms  │ 15 ครั้ง
       ├─ ... (ซ้ำ 13 ครั้ง)                                              │ = 1,950 ms
       └─ db.query  SELECT t.* FROM transaction_log ...     ▎            13ms  ┘  (81%)
──────────────────────────────────────────────────────────────────────────
```

**สิ่งที่ตอบได้ทันทีจากภาพเดียวนี้:**

| คำถาม | คำตอบ |
| --- | --- |
| ผู้ใช้รอทั้งหมดกี่วินาที | 2.41 วินาที |
| เวลาส่วนใหญ่หมดไปที่ไหน | Backend (2.15s = 89%) |
| ปัญหาอยู่ที่เครือข่ายไหม | ไม่ใช่ (2410 - 2180 = 230 ms เท่านั้น) |
| ปัญหาอยู่ที่ Frontend rendering ไหม | ไม่ใช่ (35 ms) |
| แล้วอยู่ที่ไหน | Query ซ้ำ 15 ครั้ง = 1,950 ms = 81% ของทั้งหมด |
| แก้ยังไง | รวมเป็น query เดียวด้วย IN clause |

> ✅ **นี่คือคุณค่าที่แท้จริงของ Distributed Tracing:** จากเดิมที่ต้องประชุมกัน 3 ทีม (Frontend / Backend / DBA) กว่าจะสรุปได้ว่าใครผิด ตอนนี้เห็นในภาพเดียวภายใน 30 วินาที

### 2.6.5 การวินิจฉัยว่าความช้าเกิดที่ชั้นใด

```
กฎการอ่าน Trace แบบเร็ว:

┌─────────────────────────────────────────────────────────────────┐
│ ถ้า [FE] navigation ยาว แต่ http.client สั้น                    │
│    -> ปัญหาที่ Frontend (rendering, JS หนัก, bundle ใหญ่)       │
├─────────────────────────────────────────────────────────────────┤
│ ถ้า http.client ยาว แต่ [BE] http.server สั้น                   │
│    -> ปัญหาที่เครือข่าย หรือ payload ใหญ่เกินไป                 │
├─────────────────────────────────────────────────────────────────┤
│ ถ้า [BE] http.server ยาว แต่ db.query รวมกันแล้วสั้น            │
│    -> ปัญหาที่ business logic ในโค้ด (loop, การคำนวณหนัก)       │
├─────────────────────────────────────────────────────────────────┤
│ ถ้า db.query อันเดียวยาวมาก                                     │
│    -> Slow query: ขาด index หรือเขียน where ผิดหลัก             │
├─────────────────────────────────────────────────────────────────┤
│ ถ้า db.query สั้นแต่มีเยอะมาก                                   │
│    -> N+1: ต้องรวม query                                        │
├─────────────────────────────────────────────────────────────────┤
│ ถ้ามีช่องว่าง (gap) ระหว่าง span                                │
│    -> รอ lock, รอ connection pool, GC pause หรือขาด instrument  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Workshop 2.2 - ไล่ Trace หนึ่งรายการจากหน้าจอ CRM จนถึง MariaDB

### เวลา (รวมอยู่ในช่วง 15:30–16:30 น.)

> **โจทย์:** เริ่มจากการคลิกปุ่มบนหน้าจอ Angular แล้วไล่ตามรอยไปจนถึง SQL ที่ทำงานบน MariaDB ให้ได้ในภาพเดียว จากนั้นระบุคอขวดและเสนอวิธีแก้

### ขั้นที่ 1 - ตรวจ Checklist ก่อนเริ่ม

- [ ] Backend เปิด `sentry.traces-sample-rate=1.0` และมี `sentry-jdbc` แล้ว
- [ ] Frontend ตั้ง `tracePropagationTargets` ครอบคลุม API
- [ ] Backend ตั้ง CORS ให้อนุญาต `sentry-trace` และ `baggage`
- [ ] ทั้งสองฝั่งใช้ `environment` เดียวกัน

### ขั้นที่ 2 - พิสูจน์ว่า header ถูกส่งจริง (ก่อนดู Sentry)

เปิด DevTools ของเบราว์เซอร์ → แท็บ Network → คลิกเข้าหน้ารายละเอียดลูกค้า → เลือก request `summary` → ดูส่วน **Request Headers**

```http
GET /api/customers/4999/summary HTTP/1.1
sentry-trace: 4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-1
baggage: sentry-environment=development,sentry-release=bcel-crm-frontend%401.0.0,...
```

> ⛔ **ถ้าไม่เห็น header ทั้งสองตัว** ให้ไล่ตรวจตามตารางนี้:
>
> | อาการ | สาเหตุ | วิธีแก้ |
> | --- | --- | --- |
> | ไม่มี header เลย | URL ไม่ตรง `tracePropagationTargets` | แก้ regex ให้ครอบคลุม |
> | มี header แต่ preflight fail | CORS ไม่อนุญาต | เพิ่มใน `allowedHeaders` |
> | มี header แต่ trace_id ไม่ตรงกัน | Backend สร้าง trace ใหม่ | ตรวจว่า `SentryTracingFilter` ทำงาน (ลำดับ filter) |
> | ค่าลงท้าย `-0` | Frontend ตัดสินใจไม่ sample | เพิ่ม `tracesSampleRate` |

### ขั้นที่ 3 - จำลองการใช้งานจริง

บนหน้าจอ Angular ให้ทำตามลำดับนี้ (จำลองพฤติกรรมพนักงานสาขา):

1. เปิดหน้า `/customers`
2. พิมพ์คำค้นหา แล้วกดค้นหา
3. คลิกลูกค้าคนหนึ่งเพื่อเข้าหน้ารายละเอียด (`/customers/:id`)
4. คลิกแท็บ "รายการเคลื่อนไหว"
5. คลิกปุ่ม "ดูรายงานสรุป"

### ขั้นที่ 4 - เปิด Trace View

ไปที่ **Explore → Traces** (หรือ **Performance** → คลิก transaction → Sample Events → **View Full Trace**)

บันทึกคำตอบลงตาราง:

| คำถาม | คำตอบ |
| --- | --- |
| trace_id คืออะไร | |
| มี transaction กี่ตัวใน trace นี้ | |
| มาจากกี่โปรเจกต์ | |
| เวลารวมของ trace | |
| Transaction ที่ใช้เวลามากสุด | |
| Span ที่ใช้เวลามากสุด | |
| จำนวน span `db.query` ทั้งหมด | |
| Span ที่ซ้ำมี SQL ว่าอะไร | |
| คอขวดอยู่ที่ชั้นไหน (FE / Network / BE logic / DB) | |

### ขั้นที่ 5 - โจทย์วินิจฉัยจริง

วิทยากรจะกดปุ่มเปิด "โหมดปัญหา" ใน backend (`/api/_debug/chaos/enable`) ซึ่งจะทำให้เกิดสถานการณ์อย่างใดอย่างหนึ่งต่อไปนี้แบบสุ่ม ให้ผู้เรียนใช้ Trace View วินิจฉัยว่าเกิดอะไรขึ้น

| # | สถานการณ์ | เบาะแสที่ควรเห็นใน Trace |
| --- | --- | --- |
| A | Query ขาด index | span `db.query` อันเดียวยาวผิดปกติ |
| B | N+1 | span `db.query` ซ้ำหลายสิบครั้ง |
| C | Core Banking timeout | span `http.client` ยาว 10 วินาทีแล้ว status = internal_error |
| D | Business logic ช้า | span `http.server` ยาว แต่ db.query รวมกันสั้น มีช่องว่างใหญ่ |
| E | Connection pool หมด | มี gap ก่อน span `db.query` แรก |

**สิ่งที่ต้องส่ง:** ผู้เรียนแต่ละคนเขียนรายงานสั้น 5 บรรทัด

```
1. อาการที่ผู้ใช้เจอ:
2. trace_id ที่ใช้วิเคราะห์:
3. ชั้นที่เป็นคอขวด:
4. หลักฐานจาก Trace (span ไหน กี่ ms กี่ครั้ง):
5. แนวทางแก้ไขที่เสนอ:
```

### ขั้นที่ 6 - เชื่อม Error เข้ากับ Trace

1. บนหน้าจอ Angular กดปุ่มที่ทำให้ backend พัง (เช่น ดูลูกค้า id 4999)
2. เปิด Issue ที่เกิดขึ้นใน `bcel-crm-frontend`
3. มองหาลิงก์ **"View Full Trace"** หรือส่วน **Trace Details** ในหน้า Issue
4. คลิกแล้วดูว่าเห็น transaction ของฝั่ง backend และ span `db.query` ที่คืน 0 rows หรือไม่

> ✅ **เกณฑ์ผ่าน Workshop 2.2:**
> - [ ] เห็น Trace เดียวที่มี transaction จากทั้ง 2 โปรเจกต์
> - [ ] ระบุคอขวดได้ถูกต้องพร้อมหลักฐานเป็นตัวเลข
> - [ ] คลิกจาก Issue ไป Trace แล้วกลับได้
> - [ ] อธิบายบทบาทของ `sentry-trace` และ `baggage` ได้

---

## 📌 สรุปประจำวันที่ 2

### สิ่งที่ทำได้แล้ววันนี้

| หัวข้อ | ผลลัพธ์ที่จับต้องได้ |
| --- | --- |
| Issue Grouping | แก้ปัญหา Issue แตกกลุ่ม/รวมกลุ่มผิดได้ |
| Context Enrichment | Issue มี User, Tags, Breadcrumbs ครบ วินิจฉัยได้เอง |
| Search Query | ค้นหา Issue ตามเงื่อนไขซับซ้อนได้ |
| Performance Monitoring | เห็น P50/P95 ของทุก endpoint |
| sentry-jdbc | เห็นทุก SQL query เป็น span |
| N+1 & Slow Query | หาเจอ แก้ได้ และพิสูจน์ผลด้วยตัวเลข |
| Angular SDK | Error + Transaction ฝั่ง Frontend เข้าระบบ |
| Distributed Tracing | ไล่ trace จากปุ่มถึง SQL ได้ในภาพเดียว |

### ตารางอ้างอิงด่วน (Cheat Sheet)

```properties
# ===== Backend: Spring Boot =====
sentry.traces-sample-rate=1.0
sentry.trace-propagation-targets=localhost,^https://.*\.bcel\.local/.*$
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
spring.datasource.url=jdbc:p6spy:mariadb://localhost:3306/bcel_crm
```

```typescript
// ===== Frontend: Angular =====
Sentry.init({
  integrations: [Sentry.browserTracingIntegration()],
  tracesSampleRate: 1.0,
  tracePropagationTargets: ['localhost', /^\/api/]
})
```

```java
// ===== เพิ่มบริบทให้ Error =====
Sentry.withScope(scope -> {
    scope.setTag("module", "crm.customer");
    scope.setContexts("business", Map.of("customerId", id));
    scope.setFingerprint(List.of("customer-not-found"));
    Sentry.captureException(e);
});
```

### ✅ Checklist ก่อนจบวัน

- [ ] Issue ทุกตัวมี User Context และ Tag ทางธุรกิจ
- [ ] แก้ปัญหาการจัดกลุ่มด้วย Fingerprint ได้
- [ ] Performance page แสดง transaction ครบทุก endpoint
- [ ] เห็น span `db.query` ของ MariaDB
- [ ] แก้ N+1 และ Slow Query แล้ว P95 ลดลงตามเกณฑ์
- [ ] Angular ส่ง Error และ Transaction เข้า Sentry
- [ ] Trace เชื่อม Frontend กับ Backend ด้วย trace_id เดียวกัน
- [ ] ไม่มี PII หลุดเข้า Sentry (ตรวจซ้ำทั้ง 2 โปรเจกต์)

### 🔮 พรุ่งนี้เราจะทำอะไร (วันที่ 3)

พรุ่งนี้เราจะเปลี่ยนจาก "มองเห็น" เป็น "ทำงานได้จริงในองค์กร":

- **เช้า:** ตั้ง Alert ที่มีประโยชน์จริงและไม่สร้าง Alert Fatigue, สร้าง Dashboard สำหรับแต่ละบทบาท, ใช้ Discover วิเคราะห์เชิงลึก, Release Health และ Suspect Commits
- **บ่าย:** Source Maps, Session Replay พร้อม masking ระดับธนาคาร, ผนวก Jenkins Pipeline และ Kubernetes, **Capstone Project** จำลอง Incident และวินิจฉัย
- **⭐ ปิดท้ายหลักสูตร (Module 3.8):** ทุกท่านจะได้ **เซิร์ฟเวอร์ของตัวเองคนละเครื่อง** ไปลงมือ **ติดตั้ง Sentry Self-hosted ด้วยมือตัวเอง** ซึ่งเป็นสิ่งที่จะนำกลับไปทำจริงที่ BCEL

### 📖 เอกสารอ่านเพิ่มเติม

- Issue Grouping และ Fingerprint: https://docs.sentry.io/product/issues/grouping-and-fingerprints/
- Enriching Events (Java): https://docs.sentry.io/platforms/java/guides/spring-boot/enriching-events/
- Tracing (Java): https://docs.sentry.io/platforms/java/guides/spring-boot/tracing/
- Custom Instrumentation: https://docs.sentry.io/platforms/java/guides/spring-boot/tracing/instrumentation/custom-instrumentation/
- JDBC Instrumentation: https://docs.sentry.io/platforms/java/guides/spring-boot/tracing/instrumentation/jdbc/
- Sentry สำหรับ Angular (Manual Setup): https://docs.sentry.io/platforms/javascript/guides/angular/manual-setup/
- Angular Features (TraceClass / TraceMethod): https://docs.sentry.io/platforms/javascript/guides/angular/features/
- Search Query Syntax: https://docs.sentry.io/concepts/search/
