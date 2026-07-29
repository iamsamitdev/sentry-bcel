# Lab วันที่ 2 — Error Tracking เชิงลึก, Performance และ Distributed Tracing

**วันพฤหัสบดีที่ 30 กรกฎาคม 2569 · 09:30–16:30 น.**
เชื่อมโยงกับ `notes/Day2_note.md` Module 2.1–2.6

> ✅ **สถานะการทดสอบ** ทุกขั้นตอนรันจริงบนเครื่องวิทยากรและยืนยันผลใน Sentry แล้ว
> Trace ที่ใช้เป็นตัวอย่างคือ `35c83640eff74ec7ab944bd12a0b3e9d` (Angular → Spring Boot → MariaDB)

**เงื่อนไขก่อนเริ่ม** ต้องทำ Lab วันที่ 1 จบแล้ว (Sentry SDK ทำงาน + เห็น Issue ใน `bcel-crm-backend`)

---

## ภาพรวม Lab วันที่ 2

| Lab | หัวข้อ | เวลา | ผลลัพธ์ที่จับต้องได้ |
| --- | --- | --- | --- |
| 2.1 | Issue Grouping และ Fingerprint | 40 นาที | ยุบ 5 Issue ให้เหลือ 1 |
| 2.2 | เติมบริบท (User, Tag, Breadcrumb) | 40 นาที | รู้ว่าใครได้รับผลกระทบ |
| **2.3** | **เปิด Performance + sentry-jdbc** | **50 นาที** | **เห็นทุก Query เป็น Span** |
| **2.4** | **ล่า N+1 และ Slow Query**<br><sub>(รวมบรรยาย "N+1 Query คืออะไร" 10 นาที)</sub> | **70 นาที** | **หาต้นเหตุได้จาก Sentry** |
| **2.5** | **ฝัง @sentry/angular** | **50 นาที** | **Error ฝั่งหน้าเว็บเข้า Sentry** |
| **2.6** | **Distributed Tracing End-to-End** | **50 นาที** | **1 Trace เห็นครบ 3 ชั้น** |

---

## Lab 2.1 — Issue Grouping และ Fingerprint

### ขั้นที่ 1 — สร้างปัญหา "Issue แตกกลุ่ม"

ยิง endpoint เดียวกันด้วย id ต่างกัน 5 ครั้ง

```bash
for i in 1 2 3 4 5; do curl -s http://localhost:8080/api/_debug/sentry/split/$i; echo; done
```

ไปดูใน Sentry จะได้ **5 Issue แยกกัน** ทั้งที่เป็นบั๊กตัวเดียวกัน

**สาเหตุ** โค้ดเอา id ใส่ลงไปในข้อความ exception

```java
throw new IllegalStateException("ไม่พบข้อมูลของลูกค้ารหัส " + id);
```

Sentry คำนวณ fingerprint จาก (ชนิด exception + ข้อความ + stack trace) เมื่อข้อความต่างกัน
fingerprint ก็ต่างกัน → แตกเป็นคนละ Issue

> 💬 **ประเด็นอภิปราย** ถ้าระบบจริงมีลูกค้า 5,000 คน บั๊กตัวเดียวนี้จะสร้าง Issue ได้ถึง 5,000 รายการ
> ทีมจะไม่มีทางรู้เลยว่ามันคือปัญหาเดียวกัน และ Alert จะยิงจนไม่มีใครอ่าน

### ขั้นที่ 2 — แก้ด้วย Fingerprint

เปิด `config/SentryDebugController.java` เมธอด `splitFixed()` ปลดคอมเมนต์

```java
Sentry.withScope(scope -> {
    scope.setFingerprint(List.of("customer-data-missing"));
    scope.setTag("customer_id", String.valueOf(id));
    Sentry.captureException(new IllegalStateException("ไม่พบข้อมูลของลูกค้า"));
});
```

เพิ่ม import `java.util.List` แล้ว rebuild + รันใหม่

```bash
for i in 1 2 3 4 5; do curl -s http://localhost:8080/api/_debug/sentry/split-fixed/$i; echo; done
```

**ผลที่ต้องได้** Issue เดียว มี **5 events** และมี tag `customer_id` ให้กรองย่อยได้

### ✅ เกณฑ์ตรวจผ่าน 2.1

- ก่อนแก้: 5 Issue · หลังแก้: 1 Issue ที่มี 5 events
- คลิกที่ tag `customer_id` แล้วเห็นการกระจายตัว 1–5 อย่างละ 20%

### ขั้นที่ 3 — ฝึกใช้ Search Query

ลองพิมพ์ในช่องค้นหาของหน้า Issues

| Query | ได้อะไร |
| --- | --- |
| `is:unresolved` | ยังไม่แก้ |
| `is:unresolved level:error` | เฉพาะระดับ error |
| `environment:development` | เฉพาะ dev |
| `transaction:"GET /api/customers/{id}/statement"` | เฉพาะ endpoint นั้น |
| `release:1.0.0` | เฉพาะ release นั้น |
| `customer_id:3` | ใช้ tag ที่เราตั้งเอง |
| `firstSeen:-1h` | Issue ที่เพิ่งโผล่ชั่วโมงนี้ |

### ขั้นที่ 4 — จัดการสถานะ Issue

| ปุ่ม | ใช้เมื่อไร | ผลข้างเคียงที่ต้องรู้ |
| --- | --- | --- |
| **Resolve** | แก้โค้ดแล้ว | ถ้าเกิดอีกใน release ใหม่ จะกลับมาเป็น **Regression** |
| **Resolve in next release** | แก้แล้วแต่ยังไม่ deploy | ต้องมี Release ที่ตั้งค่าถูกต้อง |
| **Archive** | รู้แล้วแต่ยังไม่แก้ | ไม่แจ้งเตือน แต่ยังนับ event |
| **Delete & Discard** | ขยะแท้ ๆ | ⚠️ event ในอนาคตจะถูกทิ้งถาวร ใช้อย่างระวัง |

---

## Lab 2.2 — เติมบริบทให้ Error

### ขั้นที่ 1 — ติดตั้ง User Context Filter

```bash
cp solutions/backend-sentry-config/SentryUserContextFilter.java backend/src/main/java/la/com/bcel/crm/config/
```

```bash
cp solutions/backend-sentry-config/BreadcrumbFilter.java backend/src/main/java/la/com/bcel/crm/config/
```

Rebuild + รันใหม่ แล้วยิง request พร้อม header ผู้ใช้

```bash
curl -s -H "X-User-Id: teller-042" -H "X-Branch-Code: BR-002" http://localhost:8080/api/customers/4999/statement
```

### ขั้นที่ 2 — ตรวจใน Sentry

Issue ต้องมี

| ส่วน | ค่าที่ต้องเห็น |
| --- | --- |
| **User** | `id: teller-042` |
| **Tags** | `branch_code: BR-002` |
| **Breadcrumbs** | ลำดับเหตุการณ์ก่อน error |
| **Users (30d)** | เพิ่มจาก 0 เป็น 1 |

> ⚠️ **ข้อควรระวังสำหรับธนาคาร** เราตั้ง `sentry.send-default-pii=false` ไว้
> ดังนั้น **ห้ามใส่ชื่อ-นามสกุล อีเมล หรือเบอร์โทรของลูกค้าลง User Context**
> ให้ใช้ **รหัสพนักงาน** หรือ **รหัสอ้างอิงภายใน** ที่ map กลับได้เฉพาะในระบบ HR เท่านั้น

### ขั้นที่ 3 — ทดลอง Breadcrumb ที่มีประโยชน์

Breadcrumb ที่ดีต้องตอบได้ว่า "ผู้ใช้ทำอะไรมาก่อนหน้านี้" ลองเปรียบเทียบ

```
❌ ไม่มีประโยชน์:  "เข้าเมธอด getStatement()"
✅ มีประโยชน์:     "ค้นหาลูกค้า customerId=4999 ผลลัพธ์ 1 รายการ"
✅ มีประโยชน์:     "เรียก Core Banking /accounts ใช้เวลา 1,240 ms"
```

---

## Lab 2.3 — เปิด Performance Monitoring และผูก sentry-jdbc ⭐

### ขั้นที่ 1 — เปิด Tracing

`backend/src/main/resources/application.properties` ปลดคอมเมนต์บล็อก `📌 วันที่ 2 Module 2.4.1`

```properties
sentry.traces-sample-rate=${SENTRY_TRACES_RATE:1.0}
sentry.trace-propagation-targets=localhost,^https://.*\\.bcel\\.local/.*$
sentry.enable-auto-session-tracking=true
```

> ⚠️ **`traces-sample-rate=1.0` ใช้ได้เฉพาะในห้อง Lab** บน production ของ BCEL
> ควรเริ่มที่ **0.1 (10%)** แล้วค่อยปรับ เพราะ 1.0 หมายถึงเก็บทุก transaction ซึ่งกิน quota เร็วมาก

### ขั้นที่ 2 — เพิ่ม sentry-jdbc

`backend/pom.xml` ปลดคอมเมนต์บล็อก `📌 วันที่ 2 Module 2.4`

```xml
<dependency>
  <groupId>io.sentry</groupId>
  <artifactId>sentry-jdbc</artifactId>
</dependency>
```

### ขั้นที่ 3 — เปลี่ยน DataSource ไปใช้ P6Spy

ใน `application.properties` **คอมเมนต์บรรทัดเดิม**

```properties
# spring.datasource.url=jdbc:mariadb://localhost:3306/bcel_crm
```

แล้วเพิ่มชุดใหม่ท้ายไฟล์

```properties
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
spring.datasource.url=jdbc:p6spy:mariadb://localhost:3306/bcel_crm
```

> 🔎 **sentry-jdbc ทำงานอย่างไร** มันไม่ได้แก้ Hibernate แต่แทรกตัวเองเป็น **JDBC driver wrapper**
> ผ่านไลบรารี P6Spy ทุก statement ที่ผ่าน driver จะกลายเป็น Span ชื่อ `db.query` โดยอัตโนมัติ
> ดังนั้นต้องเปลี่ยน **ทั้ง 2 อย่าง** — `driver-class-name` และ prefix `jdbc:p6spy:` ใน URL
> ถ้าเปลี่ยนแค่อย่างเดียวจะไม่เห็น Span ของ DB เลย

ไฟล์ `backend/src/main/resources/spy.properties` เตรียมไว้ให้แล้ว มีบรรทัดสำคัญ

```properties
modulelist=com.p6spy.engine.spy.P6SpyFactory
```

บรรทัดนี้ปิดโมดูล logging ของ P6Spy ถ้าไม่ตั้ง ไฟล์ `spy.log` จะโตจนดิสก์เต็ม

### ขั้นที่ 4 — Rebuild และตรวจว่า P6Spy ทำงาน

```bash
cd backend && mvn -B clean package -DskipTests
```

```bash
SENTRY_DSN='<DSN backend>' mvn spring-boot:run
```

**ตรวจจาก log ตอน start** ต้องเห็น

```
HikariPool-1 - Added connection com.p6spy.engine.wrapper.ConnectionWrapper@...
```

ถ้ายังเป็น `org.mariadb.jdbc.Connection` แปลว่า P6Spy ยังไม่ทำงาน

### ✅ เกณฑ์ตรวจผ่าน 2.3

ยิง request สัก 2–3 ครั้ง

```bash
curl -s -o /dev/null http://localhost:8080/api/customers?page=0&size=20
```

รอประมาณ 1–2 นาที แล้วเปิด **Insights → Backend** จะเห็นการ์ด **Queries by Time Spent**
พร้อม SQL จริง เช่น

```
SELECT branch_code, COUNT(*), SUM(amount) FROM transaction_log t
WHERE tx_date >= %s AND tx_date < %s GROUP BY branch_code ORDER BY totalAmount DESC
```

> ⏱️ **ข้อมูลไม่ขึ้นทันที** span ใช้เวลาประมวลผลราว 1–2 นาที ให้ผู้เรียนยิง request แล้วพักเบรก
> อย่าตกใจว่า "ทำผิด" ถ้ากดดูภายใน 10 วินาทีแรกแล้วยังว่าง

---

## Lab 2.4 — ล่า N+1 Query และ Slow Query ⭐

> Workshop หลักของช่วงเช้า-บ่ายวันที่ 2 · 60 นาที

---

### 📖 ทำความเข้าใจก่อน: N+1 Query คืออะไร

> วิทยากรบรรยายส่วนนี้ก่อนลงมือ ใช้เวลา 10 นาที
> ผู้เรียนหลายคนเคยได้ยินคำนี้แต่ยังนึกภาพไม่ออกว่ามันหน้าตาเป็นอย่างไรในระบบจริง

#### นิยาม

**N+1 Query** คือปัญหาด้านประสิทธิภาพที่เกิดขึ้นเมื่อ

> ระบบดึงข้อมูล**หลัก** 1 ครั้ง (**1** query) แล้ว**วนลูป**ดึงข้อมูล**ย่อย**ที่เกี่ยวข้องเพิ่มอีก **N** ครั้ง (**N** queries)
> ส่งผลให้เกิดคำสั่งค้นหาฐานข้อมูล **มากเกินความจำเป็น**

ชื่อ "N+1" มาจากจำนวน query ทั้งหมด = **N + 1**

#### ตัวอย่างที่เห็นภาพที่สุด

สมมติต้องแสดงรายชื่อผู้ใช้ 100 คน พร้อมจำนวนคำสั่งซื้อของแต่ละคน

```
query ที่ 1     SELECT * FROM users LIMIT 100                    ← ดึงข้อมูลหลัก 1 ครั้ง
                ↓ วนลูป 100 รอบ
query ที่ 2     SELECT * FROM orders WHERE user_id = 1
query ที่ 3     SELECT * FROM orders WHERE user_id = 2
query ที่ 4     SELECT * FROM orders WHERE user_id = 3
   ...                          ...
query ที่ 101   SELECT * FROM orders WHERE user_id = 100

รวม = 1 + 100 = 101 queries   ทั้งที่ควรใช้แค่ 1–2 queries
```

#### สาเหตุหลัก — Lazy Loading ของ ORM

ปัญหานี้ **แทบไม่เคยเกิดจากคนตั้งใจเขียน** แต่เกิดจากพฤติกรรมของ ORM
(Object-Relational Mapping เช่น Hibernate/JPA, Entity Framework, Eloquent, ActiveRecord)

ORM ใช้กลไก **Lazy Loading** — จะยังไม่ดึงข้อมูลความสัมพันธ์ (relation) มา
จนกว่าจะมีโค้ดไปเรียกใช้จริง ซึ่ง "ตอนเรียกใช้" มักอยู่ **ในลูป** พอดี

```java
// โค้ดดูสะอาดมาก แต่ซ่อน N+1 ไว้เต็ม ๆ
List<Account> accounts = accountRepository.findByCustomerId(id);   // 1 query

for (Account acc : accounts) {
    // ⚠️ ทุกครั้งที่วนลูป Hibernate ยิง SELECT ใหม่ 1 ครั้งเงียบ ๆ
    List<TransactionLog> txs = txRepository.findByAccountId(acc.getId());
    total = total.add(sum(txs));
}
```

> 🎯 **นี่คือเหตุผลที่ N+1 อันตราย** โค้ดอ่านแล้วสวยงาม ไม่มีอะไรผิดสายตา
> ทดสอบบนเครื่อง dev ที่มีข้อมูล 10 แถวก็เร็วปกติ **แต่พังตอนขึ้น production ที่มีข้อมูลจริง**

#### ผลกระทบ

| ด้าน | เกิดอะไรขึ้น |
| --- | --- |
| **Latency สูง** | เวลาตอบสนองเพิ่มตามจำนวนข้อมูล ยิ่งข้อมูลมากยิ่งช้า |
| **เปลือง network round-trip** | แต่ละ query เสียเวลาไป-กลับ DB ต่อให้ query เร็วแค่ 1 ms ก็ตาม |
| **ฐานข้อมูลทำงานหนักเกินจำเป็น** | connection pool ถูกใช้นานขึ้น กระทบ request อื่นทั้งระบบ |
| **ไม่ scale** | ข้อมูลโต 10 เท่า → query โต 10 เท่า → ช้าลงแบบเชิงเส้น |

> 🏦 **ที่ BCEL จะหนักกว่านี้มาก** ในห้อง Lab ฐานข้อมูลอยู่ในเครื่องเดียวกัน round-trip เกือบ 0
> แต่ระบบจริงของธนาคาร DB อยู่คนละเครื่อง (บางทีคนละ data center)
> round-trip อาจ 2–5 ms ต่อครั้ง → **15 queries = เสียเวลาไป-กลับเปล่า ๆ 30–75 ms**

#### วิธีแก้

| วิธี | ทำอย่างไร | เหมาะกับ |
| --- | --- | --- |
| **Eager Loading / JOIN FETCH** | สั่ง `JOIN` ดึงข้อมูลย่อยมาพร้อมกันในคำสั่งเดียว | กรณีทั่วไป · แก้ได้ตรงจุดที่สุด |
| **Batching** | ดึง id ทั้งหมดก่อน แล้วยิง `WHERE id IN (...)` ครั้งเดียว | ข้อมูลย่อยเยอะจน JOIN แล้วแถวบานปลาย |
| **Projection / DTO Query** | เขียน query คืนเฉพาะฟิลด์ที่ต้องใช้ | หน้าจอที่ต้องการแค่ยอดรวม ไม่ต้องการ entity เต็ม |
| **`@BatchSize`** (Hibernate) | ให้ Hibernate รวม query ย่อยเป็นชุด | แก้แบบไม่ต้องแตะ query เดิม |

**ตัวอย่างการแก้ด้วย JOIN FETCH ใน Spring Data JPA**

```java
// ❌ ก่อนแก้ — 1 + N queries
List<Account> findByCustomerId(Long customerId);

// ✅ หลังแก้ — 1 query เดียวจบ
@Query("""
    SELECT DISTINCT a FROM Account a
    LEFT JOIN FETCH a.transactions
    WHERE a.customerId = :customerId
    """)
List<Account> findByCustomerIdWithTransactions(@Param("customerId") Long customerId);
```

#### ทำไมต้องใช้ Sentry หา ไม่ใช่แค่ code review

| วิธีตรวจ | ข้อจำกัด |
| --- | --- |
| อ่านโค้ดเอง | ORM ซ่อน query ไว้ มองด้วยตาแทบไม่เห็น |
| ดู log ของ Hibernate | ต้องเปิด `show-sql` ซึ่งทำใน production ไม่ได้ · log ท่วม |
| **ดู Span ใน Sentry** | ✅ เห็นทุก query **ในระบบจริง** พร้อมเวลา และเห็นว่า request ไหนเป็นคนก่อ |

> Sentry มีการตรวจจับรูปแบบนี้ให้อัตโนมัติในชื่อ **Performance Issue: N+1 Queries**
> โดยมองหา span ที่มี SQL รูปแบบเดียวกันซ้ำติดกันภายใน transaction เดียว
> 📎 อ้างอิง: [Sentry Docs — N+1 Queries](https://docs.sentry.io/product/issues/issue-details/performance-issues/n-one-queries/)

#### ในโปรเจกต์นี้ N+1 อยู่ที่ไหน

| รายการ | ค่า |
| --- | --- |
| Endpoint | `GET /api/customers/1/summary` |
| ไฟล์ | `customer/CustomerSummaryService.java` |
| ข้อมูลหลัก | ลูกค้า 1 คน + บัญชีของเขา = **2 queries** |
| ข้อมูลย่อย (N) | รายการเดินบัญชีของแต่ละบัญชี ~**15 บัญชี = 15 queries** |
| **รวม** | **17 queries ต่อ 1 request** |
| ตัวที่แก้แล้ว | `GET /api/customers/1/summary-fixed` |

ต่อไปนี้เราจะไปหามันด้วย Sentry จริง ๆ

---

### สถานการณ์จำลอง

> เจ้าหน้าที่สาขาแจ้งว่า "หน้าสรุปลูกค้าและหน้ารายงานประจำวันช้ามาก"
> ท่านมีเพียง Sentry เท่านั้น จงหาว่าช้าที่ตรงไหนและเพราะอะไร

### ขั้นที่ 1 — เก็บข้อมูลพื้นฐาน (Baseline)

```bash
curl -s -o /dev/null -w "N+1        time=%{time_total}s\n" http://localhost:8080/api/customers/1/summary
```

```bash
curl -s -o /dev/null -w "N+1 fixed  time=%{time_total}s\n" http://localhost:8080/api/customers/1/summary-fixed
```

```bash
curl -s -o /dev/null -w "slow query time=%{time_total}s\n" 'http://localhost:8080/api/reports/daily-summary?date=2026-07-29'
```

```bash
curl -s -o /dev/null -w "fast query time=%{time_total}s\n" 'http://localhost:8080/api/reports/daily-summary-fast?date=2026-07-29'
```

**ค่าที่วัดได้จริงบนเครื่องทดสอบ**

| Endpoint | เวลา | เทียบกับตัวที่แก้แล้ว |
| --- | --- | --- |
| `/customers/1/summary` (N+1) | **0.341 s** | ช้ากว่า 2.8 เท่า |
| `/customers/1/summary-fixed` | 0.119 s | — |
| `/reports/daily-summary` (Slow Query) | **2.021 s** | ช้ากว่า **12.5 เท่า** |
| `/reports/daily-summary-fast` | 0.162 s | — |

### ขั้นที่ 2 — วินิจฉัย N+1 จาก Sentry

**Explore → Traces** เลือกโปรเจกต์ `bcel-crm-backend` แล้วหา trace ที่มี root
`GET /api/customers/{id}/summary` คลิกเข้าไปดู waterfall

**สิ่งที่จะเห็น (ยืนยันจริงแล้ว)**

```
GET /api/customers/{id}/summary                                   59.73ms
├── db.query  select ... from customer c1_0 where c1_0.id=?         0.53ms   ← 1 query
├── db.query  select ... from account a1_0 where a1_0.customer_id=? 1.07ms   ← 1 query
├── db.query  select ... from transaction_log tl1_0 where account_id=? 1.32ms
├── db.query  select ... from transaction_log tl1_0 where account_id=? 1.08ms
├── db.query  select ... from transaction_log tl1_0 where account_id=? 1.04ms
│   ... (ซ้ำแบบเดียวกันทั้งหมด 15 ครั้ง) ...
└── db.query  select ... from transaction_log tl1_0 where account_id=? 1.03ms
```

**รวม 17 queries ต่อ 1 request** = 1 (customer) + 1 (accounts) + 15 (transaction ต่อบัญชี)

> 🎯 **ลายเซ็นของ N+1 ที่ต้องจำให้ได้**
> Span ที่มี **SQL รูปแบบเดียวกันเป๊ะ ๆ ซ้ำติดกันหลายอัน** ต่างกันแค่ parameter
> ไม่จำเป็นต้องแต่ละอันช้า — แต่ละอันแค่ 1 ms ก็จริง แต่ 15 อันบวก network round-trip
> ทำให้ช้ากว่าเดิม 3 เท่า และที่ BCEL ซึ่ง DB อยู่คนละเครื่อง จะแย่กว่านี้อีกมาก

**สาเหตุในโค้ด** `customer/CustomerSummaryService.java` วนลูปเรียก repository ทีละบัญชี
ซึ่งตรงกับกลไก **Lazy Loading ของ ORM** ที่อธิบายไว้ในหัวข้อ 📖 ด้านบนพอดี

**ให้ผู้เรียนเปิดไฟล์แล้วเทียบกับ waterfall** จะเห็นความสัมพันธ์ 1 ต่อ 1 ชัดเจน

```java
// CustomerSummaryService.build() — ต้นเหตุของ N+1
Customer customer = customerRepository.findById(customerId)...;            // ← span ที่ 1

// query ที่ 1: ดึงบัญชีทั้งหมดของลูกค้า
List<Account> accounts = accountRepository.findByCustomerId(customerId);   // ← span ที่ 2

for (Account account : accounts) {                                         // ← วน 15 รอบ
    // 💥 query ที่ 2..N+1: วนดึงรายการเคลื่อนไหวทีละบัญชี
    List<TransactionLog> logs =
            transactionRepository.findTop10ByAccountIdOrderByTxDateDesc(account.getId());
    ...                                                                    // ← span ที่ 3–17
}
```

**วิธีแก้ที่โปรเจกต์นี้ใช้** — เมธอด `buildFixed()` ดึงทีเดียวด้วย `IN` clause แล้วจัดกลุ่มใน memory

```java
List<Long> accountIds = accounts.stream().map(Account::getId).toList();

// ดึงทีเดียวทั้งหมด แล้ว groupingBy เอง
Map<Long, List<TransactionLog>> logsByAccount =
        transactionRepository.findRecentByAccountIds(accountIds, ...)
                .stream()
                .collect(Collectors.groupingBy(TransactionLog::getAccountId));
```

> 📉 **ลดจาก 17 queries เหลือ 3 queries** และที่สำคัญกว่านั้นคือ
> **จำนวน query คงที่ไม่ว่าลูกค้าจะมีกี่บัญชี** — จาก O(N) กลายเป็น O(1)

> 💬 **คำถามชวนคิดสำหรับห้อง** ถ้าลูกค้ารายนี้มี 15 บัญชีแล้วช้า 0.34 วินาที
> แล้วถ้าเป็นลูกค้าองค์กรที่มี 200 บัญชีล่ะ · และถ้าหน้าจอต้องแสดงลูกค้า 20 คนพร้อมกันล่ะ
> (คำตอบ: 1 + 1 + 200 = 202 queries ต่อคน × 20 คน = **มากกว่า 4,000 queries ต่อการโหลด 1 หน้า**)

> 📌 ข้อมูลจำลองตั้งใจให้ลูกค้ารหัส **1–100** มีบัญชีคนละ ~15 ใบ เพื่อให้ N+1 เห็นชัด
> ถ้าทดสอบกับลูกค้ารหัสอื่นจะเห็นไม่ชัด

### ขั้นที่ 3 — วินิจฉัย Slow Query

หา trace ที่มี root `GET /api/reports/daily-summary` เปรียบเทียบกับ `daily-summary-fast`

| | `daily-summary` | `daily-summary-fast` |
| --- | --- | --- |
| จำนวน query | 1 | 1 |
| เวลาของ query | ~2,000 ms | ~150 ms |
| แถวที่สแกน | 800,000 (full table scan) | เฉพาะช่วงวันที่ |

**สาเหตุ** query เอาฟังก์ชันครอบคอลัมน์ที่ต้องการใช้ index

```sql
WHERE DATE(tx_date) = ?        -- ❌ MariaDB ใช้ index ไม่ได้
```

**วิธีแก้ 2 ชั้น**

1. เขียน query เป็นช่วงแทนการครอบฟังก์ชัน

```sql
WHERE tx_date >= ? AND tx_date < ?     -- ✅ ใช้ index ได้
```

2. สร้าง index ที่ยังขาด

```bash
docker compose exec mariadb mariadb -u root -plabpass123 bcel_crm -e "CREATE INDEX idx_txlog_date_branch ON transaction_log (tx_date, branch_code);"
```

วัดผลใหม่หลังสร้าง index

```bash
curl -s -o /dev/null -w "หลังสร้าง index time=%{time_total}s\n" 'http://localhost:8080/api/reports/daily-summary-fast?date=2026-07-29'
```

> 🧪 **อยากทำซ้ำ** ให้รีเซ็ตฐานข้อมูลกลับสถานะเดิมด้วย
> `powershell -ExecutionPolicy Bypass -File scripts\reset-db.ps1`

### ขั้นที่ 4 — Chaos Mode (วิทยากรเปิด ผู้เรียนวินิจฉัยเอง)

วิทยากรเปิดสถานการณ์โดย**ไม่บอกว่าเป็นอันไหน**

```bash
curl -X POST 'http://localhost:8080/api/_debug/chaos/enable?scenario=A_SLOW_QUERY'
```

| Scenario | อาการที่ผู้เรียนต้องวินิจฉัยให้ได้ |
| --- | --- |
| `A_SLOW_QUERY` | span `db.query` เดียวกินเวลาเกือบทั้งหมด |
| `B_N_PLUS_ONE` | span `db.query` ซ้ำรูปแบบเดิมหลายสิบอัน |
| `C_HTTP_TIMEOUT` | span `http.client` ไป Core Banking ค้างนาน |
| `D_SLOW_LOGIC` | transaction ช้าแต่ **ไม่มี** span ย่อยที่ช้า → ช้าที่โค้ดเอง |
| `E_POOL_EXHAUSTED` | span แรกค้างนานมากตั้งแต่ยังไม่ query → รอ connection |

ปิดเมื่อจบ

```bash
curl -X POST http://localhost:8080/api/_debug/chaos/disable
```

**ส่งงาน** แต่ละกลุ่มรายงาน: อาการ / span ที่เป็นหลักฐาน / สมมติฐาน / วิธีแก้

---

## Lab 2.5 — ฝัง @sentry/angular ⭐

### ขั้นที่ 1 — ติดตั้ง

```bash
cd frontend && npm install --save @sentry/angular
```

เวอร์ชันที่ทดสอบแล้ว: **10.68.0**

### ขั้นที่ 2 — ใส่ DSN

`frontend/src/environments/environment.ts`

```typescript
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:8080/api',
  sentryDsn: '<DSN ของ bcel-crm-frontend>',      // ⚠️ คนละตัวกับ backend
  sentryEnvironment: 'development',
  sentryRelease: 'bcel-crm-frontend@1.0.0'
}
```

> ⚠️ **ต้องใช้ DSN ของโปรเจกต์ frontend** ถ้าเผลอใช้ของ backend ข้อมูลจะไปรวมกันมั่ว
> และ Session Replay จะไม่ทำงานเพราะโปรเจกต์ฝั่ง server ไม่รองรับ

### ขั้นที่ 3 — เรียก Sentry.init() ใน main.ts

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

  // ⭐ หัวใจของ Distributed Tracing
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

> ⚠️ **`Sentry.init()` ต้องอยู่ก่อน `bootstrapApplication()`** ถ้าเรียกทีหลัง
> error ที่เกิดตอน bootstrap จะจับไม่ได้

### ขั้นที่ 4 — เพิ่ม 3 provider ใน app.config.ts

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

    // 1) แทนที่ ErrorHandler มาตรฐานของ Angular
    ,{ provide: ErrorHandler, useValue: Sentry.createErrorHandler({ showDialog: false }) }

    // 2) TraceService: ติดตาม routing เป็น transaction
    ,{ provide: Sentry.TraceService, deps: [Router] }

    // 3) ⚠️ ถ้าลืมข้อนี้ TraceService จะไม่ถูกสร้าง
    ,provideAppInitializer(() => { inject(Sentry.TraceService) })
  ]
}
```

> ⭐ **ข้อ 3 คือจุดที่คนลืมบ่อยที่สุด** Angular DI จะสร้าง service ก็ต่อเมื่อมีคน inject
> ถ้าไม่มี `provideAppInitializer` ที่ inject `TraceService` มันจะไม่ถูกสร้างเลย
> ผลคือ **error จับได้ปกติ แต่ transaction ของ navigation ไม่เกิดขึ้นสักอัน**
> และคนมักไปโทษว่า `tracesSampleRate` ผิด ทั้งที่ปัญหาอยู่ตรงนี้

### ขั้นที่ 5 — รันและทดสอบ

```bash
cd frontend && npm start
```

> ถ้า `npm start` พังด้วย `'Error' is not recognized...` แปลว่าพาธมี `&` หรือช่องว่าง
> ดู `Lab_00_Setup.md` ข้อ 0.2 หรือใช้ `node node_modules/@angular/cli/bin/ng.js serve`

เปิด `http://localhost:4200` แล้วกดปุ่มทดสอบในหน้า **ลูกค้า**

| ปุ่ม | ทดสอบอะไร |
| --- | --- |
| **ทดสอบ Error** | error แบบ synchronous ผ่าน Angular ErrorHandler |
| **ทดสอบ Error แบบ async** | error ใน `setTimeout` (นอก Zone) |
| **เรียก API ที่พัง (ลูกค้า 4999)** | HTTP 500 จาก backend |
| **เรียก API ที่ช้า (N+1)** | transaction ที่ช้า |

### ✅ เกณฑ์ตรวจผ่าน 2.5

เปิด Issues เลือกโปรเจกต์ `bcel-crm-frontend` ต้องเห็น

```
Error: BCEL CRM Frontend: ทดสอบ error ตัวแรก
Error: BCEL CRM Frontend: ทดสอบ error แบบ async
HTTP error 500 http://localhost:8080/api/customers/4999/statement
```

---

## Lab 2.6 — Distributed Tracing End-to-End ⭐⭐

> ไฮไลต์ของทั้งหลักสูตร · 50 นาที

### แนวคิดใน 30 วินาที

1. Angular สร้าง `trace_id` ตอนเริ่ม pageload
2. ทุก HTTP request ที่ปลายทางตรงกับ `tracePropagationTargets` จะถูกแนบ header
   `sentry-trace` และ `baggage`
3. `SentryTracingFilter` ฝั่ง Spring Boot อ่าน header นั้นแล้ว **ต่อ trace เดิม** แทนที่จะสร้างใหม่
4. `sentry-jdbc` เติม span ของทุก query เข้าไปใน trace เดียวกัน
5. Sentry ประกอบทั้งหมดเป็น waterfall เดียว

### ขั้นที่ 1 — ตรวจการตั้งค่าทั้งสองฝั่ง

| ฝั่ง | ค่า | ต้องครอบคลุมอะไร |
| --- | --- | --- |
| Angular | `tracePropagationTargets: ['localhost', /^\/api/]` | **origin ของ API จริง** ไม่ใช่แค่ path |
| Spring Boot | `sentry.trace-propagation-targets=localhost,^https://.*\.bcel\.local/.*$` | ปลายทางที่ backend เรียกต่อ |

> ⚠️ ในห้อง Lab หน้าเว็บอยู่พอร์ต **4200** แต่ API อยู่พอร์ต **8080** ซึ่งเป็นคนละ origin
> จึงต้องมี `'localhost'` อยู่ในลิสต์ ถ้าใส่แค่ `/^\/api/` อย่างเดียว header จะไม่ถูกแนบ
> และ trace จะขาดเป็นสองท่อน

### ขั้นที่ 2 — พิสูจน์ว่า header ถูกแนบจริง

เปิด DevTools → Network → คลิก request ไปที่ `/api/customers` → แท็บ **Headers**
ต้องเห็นใน Request Headers

```
sentry-trace: 35c83640eff74ec7ab944bd12a0b3e9d-<span-id>-1
baggage: sentry-environment=development,sentry-release=bcel-crm-frontend@1.0.0,sentry-trace_id=35c83640...
```

> ถ้าไม่เห็น 2 header นี้ → `tracePropagationTargets` ผิด กลับไปขั้นที่ 1

### ขั้นที่ 3 — สร้าง trace จริงจากหน้าจอ

ที่ `http://localhost:4200` ทำตามลำดับนี้ในเซสชันเดียว

1. เปิดหน้า **ลูกค้า** (จะเกิด pageload transaction)
2. กด **เรียก API ที่พัง (ลูกค้า 4999)**
3. กด **เรียก API ที่ช้า (N+1)**
4. ไปหน้า **รายงาน** แล้วกด **เรียกเวอร์ชันช้า**

### ขั้นที่ 4 — ดู Trace แบบ End-to-End

**Explore → Traces** เลือก **ทั้งสองโปรเจกต์** พร้อมกัน (backend + frontend)
แล้วเปิดแท็บ **Trace Samples** หา trace ที่ **Trace Root เป็นไอคอน Angular** และมี span จำนวนมาก

**ผลที่ได้จริง — trace `35c83640eff74ec7ab944bd12a0b3e9d`**

```
Trace — 35c83640eff74ec7ab944bd12a0b3e9d              Spans 126 · Root Duration 614.20ms
│
├── 🅰️ pageload — /customers/                                    614.20ms
│     └── 🟢 http.server — GET /api/customers                     10.94ms
│
├── 🟢 http.server — GET /api/customers/{id}/statement            44.33ms  🔴
├── 🟢 http.server — GET /api/customers/{id}/summary              59.73ms
│
├── 🅰️ Error — BCEL CRM Frontend: ทดสอบ error ตัวแรก
├── 🅰️ Error — BCEL CRM Frontend: ทดสอบ error แบบ async
│
├── 🟢 http.server — GET /api/customers/{id}/statement            17.80ms  🔴
└── 🟢 http.server — GET /api/customers/{id}/summary              61.52ms
```

พร้อม Web Vitals ที่มุมขวาบน: **LCP 544ms · FCP 544ms · TTFB 16ms**

> 🎯 **นี่คือภาพที่ตอบโจทย์ของหลักสูตร** จากคลิกเดียวของผู้ใช้ เราไล่ได้ตั้งแต่
> หน้าจอ Angular → HTTP → Controller ของ Spring Boot → SQL ที่ยิงเข้า MariaDB
> และเห็นว่า **error ฝั่ง frontend เกิดขึ้นในช่วงเวลาไหนของ trace เดียวกัน**

### ขั้นที่ 5 — ตอบคำถามวินิจฉัยจาก Trace เดียว

ให้ผู้เรียนตอบจาก waterfall ที่เปิดอยู่

1. request ไหนใช้เวลานานที่สุด และเวลาส่วนใหญ่หมดไปกับอะไร (network / server / DB)
2. `/summary` มี span `db.query` กี่อัน เป็นรูปแบบซ้ำหรือไม่
3. ถ้าจะลดเวลาให้ผู้ใช้รู้สึกได้ ควรแก้ชั้นไหนก่อน
4. Web Vitals บอกอะไรเกี่ยวกับประสบการณ์ผู้ใช้

### ⚠️ สิ่งที่พบระหว่างทดสอบและควรอธิบายในห้อง

**1. CORS preflight สร้าง trace แยก**
จะเห็น trace สั้น ๆ ชื่อ `OPTIONS /api/customers/{id}/summary` (1 span, ~1 ms) แยกออกมา
เพราะเบราว์เซอร์ยิง preflight เองโดยไม่แนบ trace header
→ **เป็นเรื่องปกติ ไม่ใช่บั๊ก** แต่ถ้ารำคาญให้กรองด้วย `!transaction:OPTIONS*`

**2. request จาก `curl` จะเป็น trace แยกเสมอ**
เพราะไม่มี header `sentry-trace` การไล่ trace แบบ end-to-end **ต้องทำผ่านเบราว์เซอร์เท่านั้น**

**3. ข้อมูลมาช้า 1–2 นาที**
span ต้องผ่านการประมวลผลก่อน อย่ารีเฟรชรัว ๆ แล้วสรุปว่าไม่ทำงาน

---

## 📋 เกณฑ์ตรวจผ่าน Lab วันที่ 2

| # | สิ่งที่ต้องพิสูจน์ได้ | วิธีตรวจ |
| --- | --- | --- |
| 1 | ยุบ Issue ที่แตกกลุ่มได้ | `/split` = 5 Issue · `/split-fixed` = 1 Issue 5 events |
| 2 | Issue มี User + Tag + Breadcrumb | Issue แสดง `teller-042` และ `branch_code` |
| 3 | P6Spy ทำงาน | log ตอน start มี `p6spy.engine.wrapper.ConnectionWrapper` |
| 4 | เห็น Query เป็น Span | Insights → Backend → Queries by Time Spent มี SQL จริง |
| 5 | **หา N+1 เจอจาก Sentry** | เห็น `db.query` รูปแบบเดิม 15 อันใน 1 transaction |
| 6 | **หา Slow Query เจอ** | เทียบ `daily-summary` 2.0s กับ `-fast` 0.16s ได้ |
| 7 | Angular ส่ง Error เข้า Sentry | Issues ของ `bcel-crm-frontend` มี 3 รายการ |
| 8 | Trace header ถูกแนบ | DevTools เห็น `sentry-trace` และ `baggage` |
| 9 | **Trace เดียวเห็นครบ 3 ชั้น** | waterfall มี pageload → http.server → db.query |
| 10 | อธิบาย Chaos scenario ได้ | รายงานกลุ่มระบุ span ที่เป็นหลักฐาน |

---

## 🔧 ปัญหาที่พบบ่อยในวันที่ 2

| อาการ | สาเหตุ | วิธีแก้ |
| --- | --- | --- |
| ไม่เห็น span ของ DB เลย | เปลี่ยนแค่ URL ไม่ได้เปลี่ยน `driver-class-name` | ต้องแก้ทั้ง 2 บรรทัด |
| ไม่เห็น transaction เลย | ลืม `sentry.traces-sample-rate` | ตั้งเป็น 1.0 ในห้อง Lab |
| Angular ไม่มี transaction ของ navigation | ลืม `provideAppInitializer` | ดู Lab 2.5 ขั้นที่ 4 ข้อ 3 |
| Trace ขาดเป็น 2 ท่อน | `tracePropagationTargets` ไม่ครอบคลุม origin ของ API | เพิ่ม `'localhost'` |
| Trace ไม่ต่อกันเมื่อใช้ curl | curl ไม่แนบ trace header | ทดสอบผ่านเบราว์เซอร์ |
| ข้อมูลไม่ขึ้นในหน้า Performance | ยังไม่ถึงเวลาประมวลผล | รอ 1–2 นาที |
| `spy.log` โตจนดิสก์เต็ม | ลบ/แก้ `spy.properties` | คืนค่า `modulelist=com.p6spy.engine.spy.P6SpyFactory` |
| Frontend ช้ามากตอน build | โปรแกรมป้องกันไวรัสสแกนโฟลเดอร์ | ยกเว้นโฟลเดอร์โปรเจกต์ |

---

**สถาบันไอทีจีเนียส เอ็นจิเนียริ่ง** · อ.สามิตร โกยม · 02-570-8449 · www.itgenius.co.th
