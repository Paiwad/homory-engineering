# Source root

## Working mode

ก่อนเริ่มงาน source ให้ตรวจ `docs/active/CURRENT.md` หาก task ไม่ระบุโหมด ให้ใช้ `guided` เป็นค่าเริ่มต้น

- `[vibe]`: อ่าน `docs/workflows/Vibe Coding.md`
- `[cowork]`: อ่าน `docs/workflows/Co-work with AI.md`
- `[guided]`: อ่าน `docs/workflows/Guided Coding.md`

โหมดกำหนดวิธีร่วมงาน ไม่ยกเว้นกติกา source root, architecture, specification หรือ verification ด้านล่าง

เมื่อทำงานที่เกี่ยวข้องกับ source root ของ Homory:

1. อ่าน `docs/ProjectPaths.md` ก่อนเสมอ
2. ตรวจสอบว่า path ของ source root ที่ระบุเข้าถึงได้ก่อนอ่านหรือแก้ไข source
3. หาก path ไม่มีอยู่จริงหรือเข้าถึงไม่ได้ ให้หยุดงานส่วนที่เกี่ยวข้องและแจ้งผู้ใช้ทันที ห้ามเดาหรือค้นหา source root อื่นเอง

## Homory engineering references

ก่อนแก้ไข source ของ Homory:

1. อ่าน `docs/ProjectPaths.md` และตรวจสอบ source root ตามกติกาด้านบน
2. อ่าน `docs/architecture/Homory — Module Structure.md` เมื่อการเปลี่ยนแปลงเกี่ยวข้องกับ module, dependency, package, DI, data flow หรือ UI architecture
3. อ่านไฟล์ specification ที่เกี่ยวข้องใน `docs/features/` ก่อนพัฒนา feature หรือแก้ business logic, validation, error handling และ test
4. รักษา dependency direction และ module boundary ตาม Module Structure; ห้ามให้ feature/UI เรียก Room, DAO หรือ repository implementation โดยตรง
5. หากเอกสาร architecture กับ feature specification ขัดกัน หรือ specification ยังมี open decision ที่กระทบงาน ให้หยุดและขอข้อสรุปก่อนลงมือ
6. อ่าน `docs/guides/Senior Android Handbook.md` ก่อนออกแบบหรือแก้ source และใช้เป็น quality/design guardrail ตาม product risk
7. ให้ feature specification เป็นตัวกำหนด product scope; ห้ามเพิ่มความซับซ้อนหรือ feature นอก scope เพียงเพื่อครอบคลุมหัวข้อใน Senior Android Handbook
8. ก่อน implement ระบุ goal, non-goal, UI states, invariant, failure modes, data ownership และ verification ที่เหมาะกับงาน; ทดสอบ business logic, error handling และ concurrency/lifecycle behavior ที่มีความเสี่ยง
