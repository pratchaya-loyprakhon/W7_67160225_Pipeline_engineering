Python Data Pipeline Engineering
Objective
ETL Pipeline สำหรับ Omnichannel Retail Dataset แบบ Incremental และ Idempotent ตาม Lab Assignment
Files
`pipeline.py` — source code
`retail_dw.db` — SQLite Star Schema หลังรันครบ batch 1-3
`quarantine.csv` — records ที่ไม่ผ่าน validation พร้อม reason_code
`pipeline_run_log.csv` — ประวัติการรัน pipeline
`Python_Data_Pipeline_Lab_Dataset.xlsx` — source dataset
Run
```bash
python pipeline.py
```
Pipeline จะรันหลักฐาน 4 รอบ:
batch_1
batch_1 ซ้ำ
batch_2
batch_3
การรัน batch เดิมซ้ำจะไม่เพิ่มจำนวน `fact_sales` เพราะใช้ `order_id` เป็น Primary Key และเปรียบเทียบ `updated_at` ก่อน upsert
Star Schema
`dim_customer`
`dim_product`
`dim_date`
`fact_sales`
Grain ของ `fact_sales`: หนึ่งรายการขายสินค้าที่ผ่านการตรวจสอบต่อ `order_id`
Data Quality
ตรวจสอบ:
datetime และ numeric type
missing values
quantity > 0
unit_price > 0
discount_pct 0-100
customer_id / product_id ต้องมีอยู่ใน Dimension
payment_method normalization
`E-Commerce` -> `Online`
duplicate `order_id` โดยเก็บรายการที่ `updated_at` ล่าสุด
คำนวณ `gross_amount` และ `net_amount`
records ที่ผิดถูกแยกไป quarantine พร้อม `reason_code`
Reflection
Availability มักสำคัญกว่า Strictness ใน Production Pipeline เพราะข้อมูลจริงมักมีความผิดพลาดบางส่วนและเข้ามาเป็น batch อย่างต่อเนื่อง หาก pipeline หยุดทั้งหมดเพราะข้อมูลเสียเพียงไม่กี่แถว จะทำให้ข้อมูลที่ถูกต้องไม่สามารถเข้าสู่ระบบได้ ดังนั้นการ quarantine เฉพาะแถวที่ผิดช่วยให้ pipeline เดินหน้าต่อได้ ขณะเดียวกันยังเก็บรายละเอียดของปัญหาไว้สำหรับตรวจสอบและแก้ไขภายหลัง แนวทางนี้ช่วยรักษาความต่อเนื่องของระบบโดยไม่ละทิ้ง Data Quality และทำให้สามารถ monitor ปัญหาได้อย่างเป็นระบบ
