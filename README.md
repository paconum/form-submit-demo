<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>ส่งข้อความและรูปภาพ</title>
  <style>
    body { font-family: Arial, sans-serif; max-width: 600px; margin: 2rem auto; }
    form { display: flex; flex-direction: column; gap: 10px; }
    input, textarea, select, button { padding: 8px; font-size: 16px; }
    button { cursor: pointer; background-color: #007bff; color: #fff; border: none; border-radius: 6px; }
    #preview { max-width: 200px; margin-top: 10px; display: none; }
    #status { margin-top: 10px; font-weight: bold; }
  </style>
</head>
<body>
  <h2>ส่งข้อความและรูปภาพ</h2>
  <form id="myForm">
    <label>หัวข้อ:</label>
    <select name="topic" id="topic" required>
      <option value="" disabled selected>-- เลือกหัวข้อ --</option>
      <option value="ข้อเสนอแนะ">ข้อเสนอแนะ</option>
      <option value="แจ้งปัญหา">แจ้งปัญหา</option>
      <option value="สอบถามข้อมูล">สอบถามข้อมูล</option>
    </select>

    <label>ข้อความ:</label>
    <textarea name="message" id="message" rows="4" required></textarea>

    <label>เลือกรูปภาพ:</label>
    <input type="file" id="file" accept="image/*" required />
    <img id="preview" alt="ตัวอย่างรูป">

    <button type="submit">ส่งข้อมูล</button>
  </form>

  <div id="status"></div>

  <script>
    const SCRIPT_URL = "[PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE](https://script.google.com/macros/s/AKfycbyVh5sTBEy0B6E9Ysh976IOjD30bSI0OCbs_d_7B3u4XgfqG6drJPti_avRCdlgFZRp-A/exec)"; // <-- ใส่ URL ของ Google Apps Script

    const form = document.getElementById("myForm");
    const fileInput = document.getElementById("file");
    const preview = document.getElementById("preview");
    const statusEl = document.getElementById("status");

    // แสดงตัวอย่างรูป
    fileInput.addEventListener("change", () => {
      const file = fileInput.files[0];
      if (!file) { preview.style.display = 'none'; return; }
      const url = URL.createObjectURL(file);
      preview.src = url;
      preview.style.display = "block";
    });

    form.addEventListener("submit", async (e) => {
      e.preventDefault();
      statusEl.textContent = "กำลังอัปโหลด...";
      const topic = document.getElementById("topic").value;
      const message = document.getElementById("message").value;
      const file = fileInput.files[0];
      if (!file) { statusEl.textContent = "กรุณาเลือกไฟล์"; return; }

      const reader = new FileReader();
      reader.onloadend = async () => {
        const base64 = reader.result.split(",")[1]; // แปลงไฟล์เป็น base64
        const data = new URLSearchParams();
        data.append("topic", topic);
        data.append("message", message);
        data.append("filename", file.name);
        data.append("mimetype", file.type || 'image/jpeg');
        data.append("file_b64", base64);

        try {
          const res = await fetch(SCRIPT_URL, { method: "POST", body: data });
          const json = await res.json();
          if (json.ok) {
            statusEl.textContent = "ส่งข้อมูลสำเร็จ! ลิงก์ไฟล์: " + json.fileUrl;
            form.reset();
            preview.style.display = "none";
          } else {
            statusEl.textContent = "เกิดข้อผิดพลาด: " + json.message;
          }
        } catch (err) {
          statusEl.textContent = "ไม่สามารถเชื่อมต่อได้";
        }
      };
      reader.readAsDataURL(file);
    });
  </script>
</body>
</html>
