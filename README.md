# nodejs-cicd-selfhosted


บันทึกรูปหน้า Runners โดยคัดลอกให้เห็น Account ของ GitHub และ Repository
<img width="2932" height="1492" alt="image" src="https://github.com/user-attachments/assets/f6c80ccd-bf22-443f-ae9a-8bb9e58df7a2" />
<img width="2928" height="1360" alt="image" src="https://github.com/user-attachments/assets/002365c0-b4f2-4ac1-b089-be610162dc7f" />


บันทึกผลการรันคำสั่ง docker logs nodejs-selfhosted-app
<img width="2338" height="1090" alt="image" src="https://github.com/user-attachments/assets/2c6d1c92-1594-4700-86ba-fc2c902f7df8" />

บันทึกผลการรัน monitor.sh
<img width="2338" height="1090" alt="image" src="https://github.com/user-attachments/assets/a2854073-86b5-4dd1-a221-8df3731c3fae" />

คำถามท้ายบท

Pull-based Model ของ Self-Hosted Runner คืออะไร มีข้อดีอย่างไร
คือรูปแบบที่ Runner เป็นฝ่ายดึงงาน (pull job) จากระบบ CI/CD เอง ไม่ใช่ให้ระบบ CI ส่งงานเข้ามา

ข้อดีหลัก
- ปลอดภัย ไม่ต้องเปิดพอร์ตจากภายนอก
- ตั้งค่าง่าย ใช้ได้หลัง Firewall / NAT
- ประหยัดค่าใช้จ่าย ใช้เครื่องของตนเอง
- ควบคุมทรัพยากรได้ เลือกสภาพแวดล้อมให้เหมาะกับงาน
- เหมาะกับระบบภายใน/Production

ทำไม Pull-based ปลอดภัยกว่า Push-based
- ไม่ต้องเปิดพอร์ต (No Inbound Access)
Pull-based ให้ Runner เชื่อมต่อออกไปเอง จึงไม่ต้องเปิดพอร์ตให้คนนอกเข้ามา
- ลดความเสี่ยงถูกโจมตีจากภายนอก
Push-based ต้องให้ CI Server เชื่อมเข้ามา → เสี่ยงถูกสแกนพอร์ต/โจมตี
- ใช้ Token แทนการเปิดระบบรับคำสั่ง
Runner ดึงงานด้วย Token ที่กำหนดสิทธิ์ได้ จำกัดการเข้าถึง
- ทำงานได้หลัง Firewall / NAT
เครื่องภายในไม่ถูก expose สู่ Internet


ทำไมต้องใช้ npm ci แทน npm install ใน production
- ได้ dependency ตรงตามที่ล็อกไว้ 100%
npm ci ใช้ package-lock.json เท่านั้น → เวอร์ชันไม่เปลี่ยน
- เร็วกว่าและเสถียรกว่า
ติดตั้งแบบ clean install ลบ node_modules เดิมก่อนเสมอ
- ป้องกันความเสี่ยงจากการอัปเดตโดยไม่ตั้งใจ
npm install อาจดึงเวอร์ชันใหม่ที่เข้ากันไม่ได้
- ผลลัพธ์เหมือนกันทุกเครื่อง (reproducible build)
เหมาะกับ CI/CD และ production
- ไม่แก้ไขไฟล์ lock
ลดโอกาสเกิดปัญหา config ต่างจากที่ทดสอบ


ทำไมห้ามใช้ Self-Hosted Runner กับ Public Repository
- เสี่ยงรันโค้ดอันตราย
ใครก็ได้สามารถแก้ไขโค้ดหรือเปิด Pull Request ใน public repo → โค้ดนั้นอาจถูกนำไปรันบน Runner ของคุณทันที
- ข้อมูลลับรั่ว (Secrets Leak)
Self-Hosted Runner มักมี
token / SSH key
access ระบบภายใน
database / server production
โค้ดอันตรายสามารถขโมยข้อมูลเหล่านี้ได้
- ควบคุมสภาพแวดล้อมไม่ได้
โค้ดจากคนนอกอาจ:
ลบไฟล์
ฝัง backdoor
ใช้เครื่องคุณขุด crypto
โจมตีระบบภายใน
- ไม่มี isolation เท่า Hosted Runner
Hosted Runner ถูกออกแบบให้เป็น sandbox และถูกลบทิ้งหลังจบงาน
แต่ Self-Hosted Runner คือ “เครื่องจริงของคุณ”


nginx คืออะไร และการทำ Revers Proxy ใน nginx มีความสำคัญอย่างไร
Nginx คือเว็บเซิร์ฟเวอร์ที่มีประสิทธิภาพสูง และทำหน้าที่เป็น Reverse Proxy โดยรับคำขอจากผู้ใช้แล้วส่งต่อไปยังเซิร์ฟเวอร์ด้านหลัง ทำให้ผู้ใช้ไม่สามารถเข้าถึงระบบภายในได้โดยตรง การทำ Reverse Proxy มีความสำคัญเพราะช่วยเพิ่มความปลอดภัย ลดภาระของแบ็กเอนด์ และทำให้ระบบทำงานได้รวดเร็วและเสถียรมากขึ้น เหมาะกับการใช้งานในระบบ Production

