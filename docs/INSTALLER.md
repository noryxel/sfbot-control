# ตัวติดตั้ง SFKeyword (Setup)

`SFKeyword_v<version>_Setup.exe` เป็นไฟล์เดียวจบ — ดับเบิลคลิกแล้วติดตั้ง/ถอนได้
ไม่ต้องติดตั้ง Inno Setup / NSIS และ **ไม่ต้องใช้สิทธิ์ผู้ดูแลระบบ**
(ติดตั้งใต้โปรไฟล์ผู้ใช้)

---

## สิ่งที่ตัวติดตั้งทำ

| ขั้น | รายละเอียด |
|---|---|
| ติดตั้งโปรแกรม | คัดลอก EXE เป็น `<โฟลเดอร์ติดตั้ง>\SFKeyword.exe` — **ชื่อคงที่** ตามที่ updater ต้องใช้ (ไอคอนเดสก์ท็อปไม่เปลี่ยนทุกครั้งที่อัปเดต) |
| เก็บตัวถอน | คัดลอกตัวเองเป็น `<โฟลเดอร์ติดตั้ง>\SFKeywordSetup.exe` |
| ทางลัด | เดสก์ท็อป + Start Menu (`SFKeyword.lnk`) — ยกเลิกได้ |
| ถอนผ่าน Windows | เพิ่มรายการใน *แอปและฟีเจอร์* (HKCU) พร้อม `UninstallString` / `QuietUninstallString` |
| ข้อมูลผู้ใช้ | **ไม่แตะ** `%LOCALAPPDATA%\SFKeyword` (บัญชี/คุกกี้/การตั้งค่า) เว้นแต่ติ๊ก "ลบข้อมูลผู้ใช้" ตอนถอน |

ค่าเริ่มต้นโฟลเดอร์ติดตั้ง: `%LOCALAPPDATA%\Programs\SFKeyword`

> ⚠ ยังไม่ได้ code-sign ตัว Setup — Windows SmartScreen อาจเตือนครั้งแรก
> (กด "More info" → "Run anyway") จนกว่าจะมีใบรับรองลงนาม

---

## วิธี build

ตัวติดตั้งถูกสร้างอัตโนมัติท้าย `tools/build_tools/build_protected_driver.py`
(หลัง build EXE หลักเสร็จ):

```bash
venv/Scripts/python.exe -X utf8 tools/build_tools/build_protected_driver.py
# → dist/SFKeyword/SFKeyword_v<version>.exe
# → dist/SFKeyword/SFKeyword_v<version>_Setup.exe   (ขนาด ~45 MB)
```

ข้ามขั้น Setup (เช่น build เร็ว ๆ ระหว่างพัฒนา):

```bash
venv/Scripts/python.exe tools/build_tools/build_protected_driver.py --no-setup
```

สร้างเฉพาะตัวติดตั้ง (ต้องมี EXE หลักอยู่แล้ว):

```bash
venv/Scripts/python.exe -X utf8 tools/build_tools/build_installer.py
# กำหนด EXE เอง / โฟลเดอร์ปลายทาง:
venv/Scripts/python.exe tools/build_tools/build_installer.py \
  --app dist/SFKeyword/SFKeyword_v1.4.2.exe --out-dir dist/SFKeyword
```

ขั้นตอนภายใน: คัดลอก EXE เป็น payload → เขียน `meta.json` (รุ่น + ที่อยู่ payload)
→ PyInstaller `--onefile --noconsole` พร้อมไอคอน `assets/sf_logo.ico`

---

## แนบตัวติดตั้งในหน้าปล่อยรุ่น

`tools/release_publish.py` แนบ `dist\SFKeyword\SFKeyword_v<ver>_Setup.exe` เป็น asset ที่ 3
ของ release ให้อัตโนมัติเมื่อไฟล์อยู่ (รุ่นเดียวกันกับ EXE เท่านั้น — ชื่อไฟล์ต้องมีเลขรุ่นตรง)

```bash
venv/Scripts/python.exe tools/release_publish.py --version <ver> --apply --summary "…"   # แนบ Setup เอง
venv/Scripts/python.exe tools/release_publish.py --version <ver> --apply --summary "…" --setup D:\path\Setup.exe
venv/Scripts/python.exe tools/release_publish.py --version <ver> --apply --summary "…" --no-setup   # ไม่แนบ
```

Setup **ไม่ถูกใส่ใน `update.json` หรือ `docs/release_history.csv`** — สองที่นั้นชี้ EXE ตรง ๆ
ตาม schema ของ updater · ตัวติดตั้งเป็นไฟล์ให้ผู้ใช้ใหม่กดโหลดเองจากหน้า release

---

## ใช้งานแบบเงียบ (สำหรับ deploy/test)

รับทั้ง `/silent` และ `--silent`:

```bat
SFKeyword_v1.4.2_Setup.exe /silent /dir "D:\Apps\SFKeyword" /no-shortcuts /no-launch
SFKeyword_v1.4.2_Setup.exe /uninstall /silent
```

| ธง | ผล |
|---|---|
| `/silent` | ไม่เปิดหน้าต่าง (exit code 0 = สำเร็จ, 1 = ล้มเหลว; error เขียนที่ `%TEMP%\sfkeyword_setup.log`) |
| `/dir <path>` | เปลี่ยนโฟลเดอร์ติดตั้ง (ปกติอ่านจาก registry ถ้าติดตั้งอยู่แล้ว) |
| `/no-shortcuts` | ไม่สร้าง/ไม่ลบทางลัด |
| `/no-registry` | ไม่เขียน/ไม่ลบรายการถอนใน Windows |
| `/no-launch` | ไม่เปิดโปรแกรมหลังติดตั้ง |
| `/purge-data` | (ตอนถอน) ลบ `%LOCALAPPDATA%\SFKeyword` ด้วย — **กู้คืนไม่ได้** |

> ใน Git Bash ต้องนำหน้าด้วย `MSYS_NO_PATHCONV=1` ไม่งั้น `/silent` จะถูกแปลงเป็น path

---

## อัปเกรด / ติดตั้งทับ

- รัน Setup รุ่นใหม่แล้วเลือกโฟลเดอร์เดิม (หรือปล่อยว่างให้อ่านจาก registry) —
  เขียนทับ `SFKeyword.exe` และอัปเดตเลขรุ่นใน *แอปและฟีเจอร์*
- ถ้าโปรแกรมเปิดอยู่ ตัวติดตั้งจะลองเขียนซ้ำ ~5 วินาที แล้วแจ้งให้ปิดโปรแกรมก่อน
- ผู้ใช้เดิมที่ติดตั้งเองด้วย EXE/ ZIP ไม่มีรายการใน registry — Setup ยังติดตั้งทับโฟลเดอร์เดิมได้
  (ตัวอัปเดตในโปรแกรมยังทำงานเหมือนเดิม เพราะชื่อไฟล์เป็น `SFKeyword.exe`)

## ถอนการติดตั้ง

- ผ่าน *Settings → Apps → Installed apps → SFKeyword → Uninstall*
  หรือรัน `SFKeywordSetup.exe /uninstall`
- ลบโปรแกรม ทางลัด และรีจิสทรี; ข้อมูลผู้ใช้เก็บไว้เป็นค่าเริ่มต้น
- ตัวถอนจะลบโฟลเดอร์ตัวเองทีหลัง (2–3 วินาที) เพราะ EXE ที่กำลังรันลบตัวเองไม่ได้

---

## ทดสอบที่ใช้ตรวจรอบนี้

```bash
# ติดตั้งเงียบ → ตรวจไฟล์/รีจิสทรี → ถอนเงียบ → ตรวจว่าโฟลเดอร์และคีย์หาย
MSYS_NO_PATHCONV=1 dist/SFKeyword/SFKeyword_v1.4.2_Setup.exe \
  /silent /dir build/_setup_bin /no-shortcuts /no-launch
MSYS_NO_PATHCONV=1 build/_setup_bin/SFKeywordSetup.exe /uninstall /silent /no-shortcuts
```
