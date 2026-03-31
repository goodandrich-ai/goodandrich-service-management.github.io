# ระบบใบสั่งงานช่าง — Good & Rich Powerplus

ระบบจัดการใบสั่งงานสำหรับทีมช่าง แบบ Multi-user Real-time Sync
Host บน GitHub Pages | Backend: Google Apps Script + Google Sheets

---

## ไฟล์หลัก

| ไฟล์ | หน้าที่ |
|---|---|
| `service-mng-system.html` | Frontend ทั้งหมด (Single HTML file) |
| `folklift-api.gs` | Google Apps Script API backend |

---

## Architecture

```
User Browser (GitHub Pages)
    ↕ fetch / POST (Content-Type: text/plain)
Google Apps Script Web App
    ↕ SpreadsheetApp
Google Sheets — forklift-work-order-system
    └── WorkOrders  ← ข้อมูลใบสั่งงานทั้งหมด
    └── Technicians
    └── Config
```

**Multi-user sync:**
- Auto-poll ทุก **15 วินาที** — UI อัปเดตอัตโนมัติเมื่อ user อื่นแก้ไข
- `visibilitychange` — sync ทันทีเมื่อ user กลับมา tab นี้
- Offline fallback — ถ้า API ไม่ตอบ บันทึก localStorage ชั่วคราว
- Conflict detection — แจ้งเตือนถ้า 2 user แก้ WO เดียวกันพร้อมกัน

---

## วิธี Deploy (ครั้งแรก)

### 1. ตั้งค่า Google Apps Script

1. เปิด Google Sheet → Extensions → Apps Script
2. วาง code จาก `folklift-api.gs` ใน `Code.gs`
3. แก้ `SPREADSHEET_ID` บรรทัดที่ 13:
   ```javascript
   const SPREADSHEET_ID = 'ใส่ ID ของ Sheet ที่นี่';
   ```
   > ID คือส่วนกลางของ URL: `docs.google.com/spreadsheets/d/**ID**/edit`
4. ตรวจว่า `SHEET_WORKORDERS` ตรงกับชื่อ Tab ในชีตแบบ **case-sensitive เป๊ะ**:
   ```javascript
   const SHEET_WORKORDERS = 'WorkOrders'; // ← ต้องตรงกับ Tab จริง
   ```
5. Deploy → New Deployment → Web App
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Copy Web App URL

### 2. ตั้งค่า HTML

เปิด `service-mng-system.html` บรรทัดที่ ~1426:
```javascript
const API_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
```
วาง URL ที่ copy มาแทน `''`

### 3. Push ขึ้น GitHub Pages

```bash
git add service-mng-system.html
git commit -m "update: service management system"
git push
```

---

## วิธี Update Code (ครั้งต่อไป)

### แก้ HTML
```bash
git add service-mng-system.html
git commit -m "fix: ..."
git push
# GitHub Pages อัปเดตอัตโนมัติใน 1-2 นาที
```

### แก้ GAS (folklift-api.gs)
1. เปิด Apps Script → แก้ code
2. Deploy → **Manage Deployments** → เลือก deployment เดิม → pencil icon → **New Version**
3. ไม่ต้องเปลี่ยน URL ใน HTML

---

## Features

| Feature | รายละเอียด |
|---|---|
| สร้างใบสั่งงาน | ฟอร์มเต็ม + auto WO number |
| รายการใบสั่งงาน | ค้นหา, filter สถานะ, filter วันที่ |
| แก้ไขใบงาน | Edit modal ครบทุก field |
| เปลี่ยนสถานะ | Dropdown auto-save ในหน้า detail |
| ย้ายวันนัด | บันทึก reschedule history |
| Archive | ซ่อนใบงานเก่าโดยไม่ลบ |
| แพลนประจำวัน | ดูใบงานวันนี้ + เลื่อนวัน |
| พิมพ์ | A5 print-ready |
| Export/Import | Backup เป็น JSON |
| Sync indicator | แสดงสถานะ sync มุมบนขวา |

---

## Data Schema — WorkOrders Sheet

| Column | ชนิด | หมายเหตุ |
|---|---|---|
| id | string | collision-safe: `timestamp_base36 + random` |
| no | string | display number: `WO-000001` |
| date | string | ISO date YYYY-MM-DD |
| apptTime | string | เวลานัด เช่น "09:00 น." |
| status | string | รอดำเนินการ / กำลังดำเนินการ / เสร็จแล้ว / ยกเลิก |
| jobType | string | repair / rental / newcar |
| subtype | JSON array | `["ตรวจเช็ค"]` |
| company | string | ชื่อบริษัทลูกค้า |
| contactChannels | JSON array | `["line","phone"]` |
| forkliftNos | JSON array | `["FL-001","FL-002"]` |
| rescheduleHistory | JSON array | ประวัติการย้ายวันนัด |
| techs | JSON array | ช่างที่รับงาน |
| archived | string | `"true"` / `"false"` |
| updatedAt | ISO timestamp | ใช้ตรวจ conflict ระหว่าง user |

---

## ⚠️ Known Pitfalls

**GAS:**
- `SHEET_WORKORDERS` ต้อง match ชื่อ Tab **case-sensitive เป๊ะ** — ถ้าผิดจะสร้าง Tab ใหม่ว่างเปล่า
- POST ต้องใช้ `Content-Type: text/plain` เท่านั้น (GAS CORS limitation)
- หลังแก้ GAS code ต้อง **New Version** ใน Manage Deployments — Save อย่างเดียวไม่อัปเดต

**HTML:**
- วันที่จาก Sheet อาจเป็น JS `Date.toString()` ไม่ใช่ ISO — ใช้ `normalizeDate()` ก่อน format เสมอ
- `async saveWorkOrder()` + polling = risk duplicate — ต้องตรวจ `find(w.id)` ก่อน `unshift`

---

## ติดต่อ / ดูแลระบบ

ระบบนี้สร้างโดย **ปัน ณัฐพัชร์ (@pun_nattapatch)**
สำหรับ Good & Rich Powerplus — 2026
