# แค็ตตาล็อกสินค้า KMO จาก Supabase (ทาง A) — แทนที่ `CODEX_TASK_catalog_supabase_migration.md`

อัปเดต 2026-11-14 · ตัดสินโดยผู้คุม (WSTERA) หลังออดิต · เอกสารนี้แทนแผนเดิมที่วางตาราง `products` แบบแบน 195 แถว

## สถานะจริง (ตรวจด้วย anon REST แบบอ่านอย่างเดียว)

- Supabase project `ybyseaenceyswjnwdmdf` **ไม่มี** `public.products` (ตอบ `PGRST205`) → `products_setup.sql` / `products_seed.sql` ใน repo `kmorackbarcustom.github.io` **ไม่ได้ถูกรัน และเลิกใช้**
- แต่ DB มีแค็ตตาล็อกแบบ `wc_*` อยู่แล้ว: `wc_products` 14 แถว (published ทั้งหมด), `wc_product_images` 49 รูป (bucket public `wstera-catalog-images`), `wc_product_fitments` 14, `wc_brands` 7 · `anon` อ่านได้ เขียนไม่ได้
- หน้าร้านอ่านจาก `wc_products` โดยตรงด้วย PostgREST embed — **ไม่ต้องสร้างตาราง/view ใหม่ ไม่ต้องเขียน DB**

## หน้าเว็บอ่านอย่างไร (`app.js`)

`GET /rest/v1/wc_products?select=<WC_PRODUCT_SELECT>&is_published=eq.true&order=sort_order,created_at.desc` แล้ว `mapWcProduct()` แปลงเป็น shape แบนเดิม:

| ที่หน้าเว็บใช้ | มาจาก |
|---|---|
| `id` | `slug` |
| `brand` / `model` | `wc_product_fitments[0].wc_brands.name` / `model_name` (ตัวพิมพ์ใหญ่) |
| `name` | ประกอบ: ชนิด(ไทย)+ยี่ห้อ+รุ่น เช่น "แร็คท้าย Honda CRF300R" (accessory ใช้คำไทยหน้าชื่อ DB เช่น "บาร์เสริม") |
| `price` | `retail_price_override` ถ้ามี ไม่งั้น `list_price` |
| `category` | `rear-rack→rear`, `side-rack→side`, `crash-bar→crashbar`, `accessory→accessory` |
| `image_url` | รูป `sort_order` ต่ำสุด |

## ที่ปิดไว้ก่อน (DB ยังไม่มีข้อมูล)

`shopee_url`, `allow_booking`, `allow_order`, `featured` → ไม่มีใน `wc_products` จึงเป็น false: **ปุ่มนัดคิว/สั่งตรง/Shopee และส่วน "ขายดี" ไม่แสดง** หน้าเป็นโหมดดูแค็ตตาล็อกอย่างเดียวจนกว่า DB จะมีคอลัมน์เหล่านี้ (ต้อง Owner GO เพราะเป็น DDL: เพิ่มคอลัมน์ใน `wc_products` หรือสร้างตารางข้างเคียง) แล้วเพิ่ม map ใน `mapWcProduct()`

## ข้อมูลต้นทางที่ควรแก้ (ไม่ได้แก้ — เขียน DB ต้อง GO)

- `wc_products.name` เป็น slug (`kawasaki-kle500-แร็คข้าง`) ควรเป็นชื่ออ่านง่าย · ตอนนี้ฝั่งหน้าเว็บประกอบชื่อเองชั่วคราว
- `description` หลายแถวขึ้นต้นด้วย slug ซ้ำชื่อ
- `model_name` เป็นตัวพิมพ์เล็กติดกัน (`versys650`) ควรเขียนแบบอ่านได้ (`Versys 650`)
- ไม่มีสินค้ากลุ่ม `gear` / `service` ปุ่มกรองสองกลุ่มนี้จะว่าง

## เรื่อง Origin/CORS

PostgREST ของ Supabase ไม่จำกัด origin สำหรับ anon · edge function `internal-proxy` / `staff-reply` ใน `kmorackbarcustom.github.io` ล็อก origin เป็น `https://kmorackbarcustom.github.io` (ถูกต้องแล้ว) · หน้าร้านนี้ย้ายมา `https://kmorackbarcustom.github.io/kmo-landing-page/` (origin เดียวกับ production repo) ไม่ต้องแก้ CORS

## ที่เลิกจากแผนเดิม

ตาราง `products`, edge function `products-proxy`, `admin-products.html` (ตามแผนเดิม), seed 195 แถวจาก CSV · ถ้าจะทำหน้าแอดมินในอนาคต ให้ออกแบบใหม่บน `wc_*` และบริบทลูกค้า KMO ผ่านการอนุมัติ Owner ก่อน

## ตรวจสอบ (2026-11-14)

- เปิดหน้า local โหลด 14 สินค้าจาก Supabase ครบ รูปโหลดครบ 14/14 console ไม่มี error
- ระดับความพร้อม: BUILD_PASS (ยังไม่ LIVE_PROVEN จนกว่า merge แล้วตรวจหน้า production)
