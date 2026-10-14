# Passadu241_Supabase

การตั้งค่า GitHub Actions เพื่อรันสคริปต์อัตโนมัติ (Cron Job) ไปกระตุ้น (Ping) Supabase ทุก ๆ 3 วัน 

# ขั้นตอนที่ 1: เตรียมข้อมูล API จาก Supabase
ก่อนอื่นให้คุณเข้าไปคัดลอกข้อมูลสำคัญ 2 อย่างจากหน้า Supabase Dashboard ของโปรเจ็กต์คุณ (ไปที่ Project Settings > API):
1. Project URL (เช่น https://supabase.co)
2. anon / public API Key


# ขั้นตอนที่ 2: นำข้อมูลไปใส่ใน GitHub Secrets (เพื่อความปลอดภัย)
ห้ามนำ API Key ไปแปะลงในโค้ดตรง ๆ ให้เก็บไว้ในระบบความลับของ GitHub:
1. เข้าไปที่ Repository ของคุณบน GitHub
2. คลิกที่แท็บ Settings > เมนูด้านซ้ายเลือก Secrets and variables > Actions
3. คลิกปุ่ม New repository secret แล้วเพิ่มค่า 2 ตัวนี้:
	• ชื่อ: SUPABASE_URL -> ใส่ค่า Project URL ของคุณ

  
# ขั้นตอนที่ 3: สร้างไฟล์ Workflow ใน GitHub
สร้างโฟลเดอร์และไฟล์ตามโครงสร้างนี้ในโปรเจ็กต์ของคุณ: .github/workflows/supabase-ping.yml จากนั้นใส่โค้ดด้านล่างนี้ลงไปครับ:

name: Supabase Keep-Alive Ping

on:
  schedule:
    - cron: '0 0 */3 * *'
  workflow_dispatch:

jobs:
  ping-supabase:
    runs-on: ubuntu-latest

    steps:
    - name: Ping Supabase Rest API
      run: |
        # ตรวจสอบเบื้องต้นว่ามี Secret ผูกไว้จริงไหมเพื่อไม่ให้สคริปต์พังลึกลับ
        if [ -z "${{ secrets.SUPABASE_URL }}" ] || [ -z "${{ secrets.SUPABASE_KEY }}" ]; then
          echo "Error: Missing GitHub Secrets (SUPABASE_URL or SUPABASE_KEY)"
          exit 1
        fi

        # รันคำสั่งส่งสคริปต์ Ping 
        curl -i -X GET "${{ secrets.SUPABASE_URL }}/rest/v1" \
          -H "apikey: ${{ secrets.SUPABASE_KEY }}" \
          -H "Authorization: Bearer ${{ secrets.SUPABASE_KEY }}"


          
# 💡 อธิบายการทำงานของโค้ด:

• cron: '0 0 */3 * *': เป็นตัวกำหนดเวลาให้ระบบทำงานอัตโนมัติทุก ๆ 3 วัน ซึ่งเพียงพอต่อการกด Keep-Alive (ระบบ Supabase ฟรีจะ Pause เมื่อไม่มีการใช้งานติดต่อกัน 7 วัน)
• workflow_dispatch: บรรทัดนี้จะสร้างปุ่ม "Run workflow" บนหน้า GitHub Actions ทำให้คุณสามารถกดคลิกเพื่อทดสอบการเชื่อมต่อได้ทันทีโดยไม่ต้องรอให้ครบ 3 วัน
• curl -I: เป็นการส่งคำขอไปดึงข้อมูล Header จากหน้าแรกของ Rest API บน Supabase ซึ่งเบามาก ไม่กินโควต้า Data Transfer แต่ถือเป็นการเปิดใช้งาน (Activity) ที่ทำให้ฐานข้อมูลตื่นตัวอยู่เสมอ

