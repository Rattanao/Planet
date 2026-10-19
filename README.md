# Check EDI Planet

เครื่องมือตรวจสอบเอกสารขาเข้าเรือก่อนจัดทำใบขนสินค้าขาเข้า (Thailand ocean import) —
เทียบ **MANIFEST** (รายงานจากสายเรือ, .xls/.xlsx หรือ PDF "CARGO MANIFEST") กับ **ENTER**
(ฟอร์ม AMENDMENT ของ UNITEX LOGISTICS (Planet), .pdf) ทีละ B/L ย่อย แล้วสร้างรายงาน `EDI.xlsx` แบบไฮไลต์สี
บอกจุดที่ต้องแก้ก่อนยื่นใบขน

> **ข้อจำกัด:** หน้าเว็บรองรับเฉพาะ **ENTER ฟอร์มของ Planet**
> ถ้าเจอฟอร์มของเจ้าอื่นหน้าเว็บจะแจ้งว่าไม่ใช่ฟอร์มของ Planet · ไฟล์ CNTRS ไม่นำมาเทียบ

## Check EDI Planet บนเว็บ (ให้เพื่อนใช้ได้ ไม่ต้องติดตั้ง)

https://rattanao.github.io/Planet/

1. ลากไฟล์ **MANIFEST** (.xls/.xlsx หรือ PDF) และ **ENTER ของ Planet** (.pdf ได้หลายไฟล์) มาวางพร้อมกัน
   (ตั้งชื่ออะไรก็ได้ — PDF ที่เป็น CARGO MANIFEST จะถูกจัดเป็น MANIFEST ให้เอง, PDF อื่นเป็น ENTER;
   ไฟล์ที่ชื่อมี `CNTRS` จะถูกข้าม)
2. ใส่ SHED NO. ที่แจ้ง (ถ้ามี) แล้วกด **ตรวจสอบ**
3. ดูผลบนหน้าเว็บ (ตารางสีเหมือน EDI.xlsx, ค้นหา/กรอง "แสดงเฉพาะที่ไม่ตรง") หรือกด **ดาวน์โหลด EDI.xlsx**

ไฟล์ถูกประมวลผลในเบราว์เซอร์ของผู้ใช้เท่านั้น ไม่มีการอัปโหลดไปที่ใด

โค้ดหน้าเว็บอยู่ที่ `docs/` (`index.html` + `core.js` — พอร์ตจาก `build_edi.py` layout K,
ฐานเดียวกับ [Check EDI Union World](https://rattanao.github.io/Union-world/) และ
[Check EDI Able](https://rattanao.github.io/check-edi-Able/))

## ฟอร์ม ENTER ที่รองรับ

ฟอร์ม `AMENDMENT` (FM-CSI-01) ของ UNITEX LOGISTICS — หัวตาราง
`B/L CHANGE NO. | MARKS & NUMBERS | QUANTITY | DESCRIPTION | CONSIGNEE | GROSS WEIGHT | HB/L NO` ต่อบล็อก:

```
HASLC0126000   MARKS          10 PALLET(S)   GOODS DESC   ABC (THAILAND)    1,000.000 KGS.   SHBKK00000000S
0012A          MADE IN CHINA                              CO.,LTD.               5.000 M3
               CNTR NO.:
               HALU1234567/20'GP
               STATUS: CY
```

- เลข B/L ตัดเป็น 2 บรรทัด (ฐาน + ตัวท้าย เช่น `0012A`) — หน้าเว็บต่อกลับเป็นเลขเต็มให้เอง
- คอลัมน์ MARKS มีเครื่องหมายจริง แล้วตามด้วย `CNTR NO.:` / เลขตู้ / `STATUS: ..` (ส่วนหลังไม่นับเป็น MARKS)
- ยอดรวมท้ายเอกสาร `TOTAL  n PACKAGE(S)  G.W n KGS.  n M3.` ใช้เทียบกับ MANIFEST

## ตรวจอะไรบ้าง

- CONSIGNEE, CONTAINER NO. (**B/L หลายตู้ต้องตรงครบทุกตู้**), STATUS (CY / LCL / LCL-CFS), TOTAL PACKAGE, PACKAGING
- GROSS WEIGHT, MEASUREMENT
- MARKS, DESCRIPTION — เทียบทีละคู่ ไม่นับช่องว่าง/วรรคตอน แต่คำที่เกิน/ขาดต้องเตือน
- REEFER TEMP, DG (CLASS/UN), TRANSIT/TRANSHIPMENT — พบฝั่งใดก็เตือนเสมอ
- SHED NO., ประเทศปลายทาง, CARGO MOVEMENT, TAX ID เทียบ NOTIFY — **ถูกต้องตามกฎ = แสดง `-`**
  (แสดงค่าจริงเฉพาะตอนไม่ตรง/ต้องตรวจ)
- ยอดรวมทั้งฉบับเทียบกับ MANIFEST GRAND TOTAL และ B/L ที่มีใน ENTER แต่ตกหล่นจาก MANIFEST (วิกฤต)

สี: แดง ⚠ = ไม่ตรง · ส้ม = ต้องให้คนยืนยัน · เหลือง = ไม่มี ENTER เทียบ · เขียว = ผ่าน (คอลัมน์สรุป)

## ใช้งานแบบโปรแกรม Python (ในเครื่อง)

`build_edi.py` อ่าน ENTER ได้หลายฟอร์ม (Planet, Able, MAC-NELS, Penanshin, CMS / THE M CORP, AGN, Quality,
Cargoport, DSV ฯลฯ) และ MANIFEST ได้ทั้ง .xls/.xlsx และ PDF

```bash
pip install -r requirements.txt
python build_edi.py --manifest MANIFEST.xls --enter ENTER.pdf --outdir output
```

เพิ่ม `--shed 0121` ถ้ามี SHED NO. ที่แจ้ง · ถ้า MANIFEST เป็นของทั้งลำเรือ แต่ ENTER เป็นของลูกค้ารายเดียว
ให้เพิ่ม `--only-enter`

## วิธีเปิดเว็บให้เพื่อนใช้ (GitHub Pages)

1. ขึ้น GitHub: https://github.com/Rattanao/Planet → **Settings → Pages**
2. **Build and deployment → Source:** *Deploy from a branch*
3. **Branch:** `main` และโฟลเดอร์ `/ (root)` → กด **Save**
4. รอ ~1 นาที ลิงก์จะขึ้นที่หัวหน้า Pages: `https://rattanao.github.io/Planet/`
   (หน้าแรกของ repo จะเด้งไป `docs/` เอง)
5. ส่งลิงก์ให้เพื่อนได้เลย — แก้โค้ดแล้ว `git push` หน้าเว็บอัปเดตเองภายในไม่กี่นาที

## หมายเหตุเรื่องข้อมูลลูกค้า

`input/`, `output/` และไฟล์ `.xls/.xlsx/.pdf` ถูก git-ignore ไว้ — repo นี้เก็บเฉพาะโปรแกรม
ไม่เก็บเอกสารจริงของลูกค้า

---
Project by Ms. Rattana Mudlae — Position Import DOC.
