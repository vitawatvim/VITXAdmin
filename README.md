# VITX Admin Control

หน้าเว็บผู้ดูแลและ schema สำหรับควบคุมสิทธิ์ของ VITX Launcher

## URL ที่ใช้งาน

- GitHub Pages: `https://vitawatvim.github.io/VITXAdmin/`
- Public static repository: `https://github.com/vitawatvim/VITXAdmin`
- OpenAI Sites สำรอง: `https://playpark-admin-control.vitawat2545.chatgpt.site/`

GitHub Pages ไม่ใช้ session ของ ChatGPT หน้าเว็บยังบังคับล็อกอิน Supabase Auth
และทุก Admin RPC ตรวจ `admin_users` ก่อนทำงาน repository `VITXLauncher`
ยังเป็น Private ส่วน `VITXAdmin` มีเฉพาะไฟล์ static ที่ใช้แสดงหน้าเว็บ

## รัน Web Admin ในเครื่อง

ต้องมี Python 3 จากนั้นเปิด PowerShell ที่ root ของ repository แล้วรัน:

```powershell
cd admin_portal\dist
python -m http.server 8765 --bind 127.0.0.1
```

เปิด `http://127.0.0.1:8765/` แล้วล็อกอินด้วยอีเมล/รหัสผ่าน Admin ที่สร้างใน
Supabase Authentication หยุด server ด้วย `Ctrl+C`

หากต้องการดูข้อมูลตัวอย่างโดยไม่เชื่อมฐานข้อมูล ให้เปิด
`http://127.0.0.1:8765/?demo`

## Deploy Web Admin

หน้าเว็บที่พร้อมใช้งานอยู่ใน `admin_portal/dist` ของ repository ส่วนตัวนี้
OpenAI Sites ใช้ไฟล์ชุดเดียวกัน ส่วน GitHub Pages ควรใช้ repository สาธารณะ
`VITXAdmin` ที่มีเฉพาะไฟล์ static เพื่อไม่เปิดเผยซอร์สโปรแกรม desktop หรือ
migration

เมื่อแก้หน้าเว็บ ให้ทดสอบ `dist` ในเครื่อง คัดลอกไฟล์ภายใน `dist` ไปไว้ที่
root ของ `VITXAdmin` และ push เข้า `main` GitHub Pages จะ deploy ให้อัตโนมัติ

ก่อน deploy ให้ตรวจว่า `dist/config.js` มีเฉพาะ Project URL และ Publishable key
ห้ามใส่ `service_role` key ลงใน repository หรือหน้าเว็บ

## อนุมัติสมาชิกที่สมัครจากแอป

1. เปิดหน้า **ผู้ใช้งาน** บัญชีใหม่จะแสดงสถานะ **รออนุมัติ** ด้านบน
2. กด **จัดการ** กำหนดวันเริ่มและวันหมดอายุ
3. เลือกเฉพาะกิจกรรมที่อนุญาตให้ผู้ใช้นั้นใช้งาน
4. เปิด **อนุมัติให้เข้าใช้งาน** แล้วกด **บันทึก**

ก่อนอนุมัติ ผู้สมัครจะล็อกอินไม่ได้และทุกกิจกรรมถูกปิดไว้ รวมถึงกิจกรรมที่เพิ่ม
ใหม่ในอนาคต จนกว่าผู้ดูแลจะเปิดสิทธิ์ให้เป็นรายคน

## สิ่งที่ระบบรองรับ

- ผู้ใช้โปรแกรมแบบ username/password แยกจากบัญชี VITX
- สมัครสมาชิกจากหน้าแอปได้ โดยบัญชีใหม่อยู่ในสถานะรออนุมัติและไม่มีสิทธิ์กิจกรรม
- กำหนดวันเริ่มต้น วันหมดอายุ และเปิด/ปิดบัญชี
- ห้ามล็อกอินพร้อมกันมากกว่าหนึ่งเครื่องด้วย session lease และ heartbeat
- Session ที่ค้างจากโปรแกรมปิดผิดปกติหมดอายุอัตโนมัติ (ค่าเริ่มต้น 120 วินาที)
- จำนวนผู้ใช้ทั้งหมด ออนไลน์ ใกล้หมดอายุ และหมดอายุแล้ว
- เปิด/ปิด/ซ่อนกิจกรรมจากส่วนกลาง
- กำหนดสิทธิ์กิจกรรมแยกเป็นรายผู้ใช้
- ปิดระบบและเปิด maintenance mode
- กำหนดเวอร์ชันล่าสุด เวอร์ชันขั้นต่ำ และบังคับอัปเดต
- Usage log, login attempts และ admin audit log
- เก็บรหัสผ่านด้วย bcrypt (`pgcrypto`) และเก็บเฉพาะ SHA-256 digest ของ session token
- RLS ปิดการอ่านตารางโดยตรง ทุกคำสั่งผ่าน RPC ที่ตรวจสิทธิ์

## ตั้งค่า Supabase

1. สร้าง Supabase project
2. เปิด **SQL Editor** แล้วรันไฟล์ใน `supabase/migrations/` ตามลำดับ
3. ไปที่ **Authentication → Users → Add user** และสร้างอีเมล/รหัสผ่านของ Admin
4. คัดลอก UUID ของ Admin แล้วรัน:

```sql
insert into public.admin_users(user_id, display_name)
values ('UUID-ของ-Admin', 'Owner');
```

5. ไปที่ **Project Settings → API** แล้วคัดลอก Project URL กับ Publishable key
6. ใส่ค่าใน `dist/config.js`

```js
window.VITX_CONFIG = {
  supabaseUrl: "https://YOUR_PROJECT.supabase.co",
  supabasePublishableKey: "YOUR_PUBLISHABLE_KEY"
};
```

Publishable key อยู่ใน browser ได้ตามการออกแบบของ Supabase ส่วน `service_role`
ห้ามใส่ในไฟล์นี้หรือในแอพเดสก์ท็อป

## API สำหรับแอพเดสก์ท็อป

เรียกผ่าน `POST /rest/v1/rpc/<function>` โดยส่ง `apikey` เป็น Publishable key:

1. `app_login` ส่ง `p_username`, `p_password`, `p_device_id`, `p_app_version`
2. เก็บ `token` ที่ได้ไว้ใน memory
3. เรียก `app_heartbeat` ทุก 45 วินาที
4. ใช้ `activities` และ `system` ที่ heartbeat ส่งกลับมาปรับหน้า UI
5. เรียก `app_log_usage` เมื่อเริ่ม/จบกิจกรรม
6. เรียก `app_logout` ตอนปิดโปรแกรม

หากโปรแกรมดับโดยไม่ได้ logout ผู้ใช้จะเข้าสู่ระบบใหม่ได้หลัง lease หมดอายุ
ค่า lease ปรับได้จากหน้า Admin ระหว่าง 60–900 วินาที

## การแจกอัปเดตแบบจำกัดผู้ใช้

GitHub Release สาธารณะไม่เหมาะกับไฟล์ที่ต้องจำกัดผู้ใช้ ให้เก็บ ZIP ใน
Supabase Storage bucket แบบ private แล้วใช้ Edge Function ตรวจ `app_sessions`
ก่อนสร้าง signed URL อายุสั้น หน้า Admin ในชุดนี้เก็บ version, SHA-256,
release notes และ private storage path ไว้พร้อมแล้ว

อย่าเปิด GitHub Release เดิมเป็น Public จนกว่าจุดดาวน์โหลดจะตรวจ session token
ของผู้ใช้ก่อนส่งไฟล์

## โครงสร้าง

- `dist/` หน้าเว็บ static พร้อม deploy
- `supabase/migrations/001_initial.sql` schema, RLS และ RPC ทั้งหมด
- `supabase/migrations/002_foreign_key_indexes.sql` ดัชนีสำหรับ foreign key และหน้ารายงาน
- `supabase/migrations/003_self_registration.sql` สมัครสมาชิก สถานะรออนุมัติ และ RPC ที่เกี่ยวข้อง
- `supabase/migrations/004_registered_user_default_access.sql` ปิดสิทธิ์กิจกรรมใหม่ให้ผู้สมัครโดยอัตโนมัติ
- `.openai/hosting.json` การตั้งค่า OpenAI Sites
