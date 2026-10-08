<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>IDC3 Master Infrastructure Dashboard & Editor</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
<!-- โหลดไลบรารีคำนวณรหัส TOTP สำหรับ Authenticator -->
<script src="https://cdn.jsdelivr.net/npm/totp-generator@2.0.1/lib/cjs/index.min.js"></script>
<style>
  :root{
    --bg:#f4f6f9; --card:#ffffff; --txt:#2d3748; --muted:#718096;
    --primary:#3182ce; --ok:#38a169; --warn:#d69e2e; --crit:#e53e3e; --border:#e2e8f0;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  body{font-family:"Segoe UI",Tahoma,"Sarabun",sans-serif;background:var(--bg);color:var(--txt);padding:20px;line-height:1.5}

  /* Login Overlay & Modern Banner Card */
  #loginOverlay{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(15,23,42,0.75);backdrop-filter:blur(4px);display:flex;justify-content:center;align-items:center;z-index:9999}
  .login-box{background:var(--card);border-radius:16px;width:100%;max-width:400px;box-shadow:0 10px 25px rgba(0,0,0,0.2);overflow:hidden;text-align:center;border:1px solid var(--border)}
  .login-banner{width:100%;height:140px;background:linear-gradient(135deg, #3182ce, #2b6cb0);display:flex;justify-content:center;align-items:center;color:#fff;position:relative;overflow:hidden}
  .login-banner img{width:100%;height:100%;object-fit:cover;position:absolute;top:0;left:0}
  .login-banner-text{position:relative;z-index:2;font-size:32px;font-weight:700;text-shadow:0 2px 4px rgba(0,0,0,0.2)}
  .login-body{padding:24px 30px 30px}
  .login-box h2{margin-bottom:6px;color:#1a202c;font-size:20px}
  .login-box p{color:var(--muted);font-size:12.5px;margin-bottom:20px}
  .login-error{color:var(--crit);font-size:12px;margin-top:8px;display:none}
  .auth-hint{font-size:11.5px;color:var(--primary);margin-bottom:12px;background:#ebf8ff;padding:8px;border-radius:6px;border:1px dashed #bee3f8}

  header{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:20px 24px;margin-bottom:20px;display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:12px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  h1{font-size:22px;color:#1a202c}
  .sub{color:var(--muted);font-size:13px;margin-top:2px}
  .header-actions{display:flex;gap:10px;align-items:center}
  .status-badge{background:#ebf8ff;color:var(--primary);padding:6px 14px;border-radius:99px;font-weight:600;font-size:13px;display:flex;align-items:center;gap:6px}
  .dot{width:8px;height:8px;background:var(--ok);border-radius:50%;display:inline-block}

  .tabs{display:flex;gap:8px;margin-bottom:20px;flex-wrap:wrap}
  .tab-btn{background:var(--card);border:1px solid var(--border);padding:10px 14px;border-radius:8px;cursor:pointer;font-weight:600;color:var(--muted);font-size:12.5px;transition:all .2s}
  .tab-btn.active{background:var(--primary);color:#fff;border-color:var(--primary)}

  .tab-content{display:none}
  .tab-content.active{display:block}

  .section-title{font-size:16px;font-weight:700;margin:20px 0 12px;color:#2b6cb0;display:flex;align-items:center;justify-content:space-between}

  .cards-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:16px;margin-bottom:20px}
  .card{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:18px;box-shadow:0 1px 3px rgba(0,0,0,.05);position:relative}
  .card .label{color:var(--muted);font-size:11.5px;text-transform:uppercase;letter-spacing:.5px;font-weight:600}
  .card .value{font-size:15px;font-weight:700;margin-top:6px;color:#1a202c;white-space:pre-line}

  .table-box{background:var(--card);border:1px solid var(--border);border-radius:12px;overflow-x:auto;margin-bottom:20px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  table{width:100%;border-collapse:collapse;font-size:12.5px;white-space:nowrap}
  th{background:#f7fafc;color:var(--muted);text-align:left;padding:12px 14px;font-weight:600;border-bottom:1px solid var(--border)}
  td{padding:11px 14px;border-bottom:1px solid var(--border);color:#4a5568}
  tr:last-child td{border-bottom:none}
  tr:hover td{background:#fafbfc}

  .btn{background:var(--primary);color:#fff;border:none;padding:6px 12px;border-radius:6px;cursor:pointer;font-size:12px;font-weight:600;transition:opacity .2s}
  .btn:hover{opacity:.85}
  .btn-sm{padding:4px 8px;font-size:11px}
  .btn-danger{background:var(--crit)}
  .btn-secondary{background:var(--muted)}
  .btn-block{width:100%;padding:10px;font-size:14px;margin-top:10px}

  .ups-link{color:var(--primary);cursor:pointer;font-weight:700;text-decoration:underline}
  .ups-link:hover{color:#2b6cb0}

  /* Image styles */
  .thumb{height:42px;width:auto;max-width:90px;object-fit:cover;border-radius:6px;border:1px solid var(--border);cursor:pointer;transition:transform .15s}
  .thumb:hover{transform:scale(1.06)}
  .no-img{color:var(--muted);font-size:11px;font-style:italic}
  #imgViewer{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.85);justify-content:center;align-items:center;z-index:2000;padding:30px;cursor:zoom-out}
  #imgViewer img{max-width:95%;max-height:90%;border-radius:8px;box-shadow:0 6px 24px rgba(0,0,0,.5)}
  .img-preview{margin-top:8px;max-height:130px;border-radius:8px;border:1px solid var(--border);display:none}
  .upload-row{display:flex;gap:8px;align-items:center;margin-top:6px}

  .modal{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);justify-content:center;align-items:center;z-index:1000}
  .modal-content{background:var(--card);padding:24px;border-radius:12px;width:100%;max-width:750px;box-shadow:0 4px 12px rgba(0,0,0,0.15);max-height:85vh;overflow-y:auto}
  .modal-content h3{margin-bottom:16px;color:#1a202c}
  .form-group{margin-bottom:12px;text-align:left}
  .form-group label{display:block;font-size:12px;color:var(--muted);margin-bottom:4px;font-weight:600}
  .form-group input, .form-group textarea{width:100%;padding:8px 12px;border:1px solid var(--border);border-radius:6px;font-size:13px;font-family:inherit}
  .form-group input[type=file]{padding:5px;background:#f7fafc}
  .hint{font-size:11px;color:var(--muted);margin-top:4px}

  .grid-2{display:grid;grid-template-columns:repeat(auto-fit,minmax(450px,1fr));gap:20px}
  .chart-container{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:20px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  .chart-box{height:240px;margin-top:10px}

  footer{text-align:center;color:var(--muted);font-size:12px;margin-top:30px;padding:15px}
</style>
</head>
<body>

<!-- Login Screen with 2FA Authenticator -->
<div id="loginOverlay">
  <div class="login-box">
    <div class="login-banner">
      <img src="https://images.unsplash.com/photo-1558494949-ef010cbdcc31?auto=format&fit=crop&w=600&q=80" alt="Login Banner">
      <div class="login-banner-text">⚡ IDC3</div>
    </div>
    <div class="login-body">
      <h2>🔐 เข้าสู่ระบบ (2FA)</h2>
      <p>Infrastructure Dashboard & Editor</p>
      
      <!-- Step 1: User & Pass -->
      <div id="step1Login">
        <div class="form-group">
          <label>USERNAME</label>
          <input type="text" id="userInput" placeholder="กรอกชื่อผู้ใช้งาน">
        </div>
        <div class="form-group">
          <label>PASSWORD</label>
          <input type="password" id="passInput" placeholder="กรอกรหัสผ่าน" onkeypress="checkEnter(event)">
        </div>
        <div id="loginError" class="login-error">Username หรือ Password ไม่ถูกต้อง!</div>
        <button class="btn btn-block" onclick="handleStep1()">ดำเนินการต่อ</button>
      </div>

      <!-- Step 2: Authenticator Code -->
      <div id="step2Auth" style="display:none;">
        <div class="auth-hint">
          📱 กรอกรหัส 6 หลักจาก Google / Microsoft Authenticator<br>
          <span style="font-size:10px;color:#2b6cb0;">(Secret Key เริ่มต้น: <b>JBSWY3DPEHPK3PXP</b>)</span>
        </div>
        <div class="form-group">
          <label>AUTHENTICATOR CODE (6 หลัก)</label>
          <input type="text" id="authCodeInput" maxlength="6" placeholder="000000" style="text-align:center;font-size:18px;letter-spacing:4px;" onkeypress="checkAuthEnter(event)">
        </div>
        <div id="authError" class="login-error">รหัส Authenticator ไม่ถูกต้อง!</div>
        <button class="btn btn-block" onclick="handleStep2()">ยืนยันตัวตน</button>
        <button class="btn btn-secondary btn-block" style="margin-top:6px;background:#718096;" onclick="backToStep1()">ย้อนกลับ</button>
      </div>

    </div>
  </div>
</div>

<!-- Image Viewer -->
<div id="imgViewer" onclick="closeImgViewer()">
  <img id="imgViewerSrc" src="" alt="preview">
</div>

<header>
  <div>
    <h1>⚡ IDC3 Master Infrastructure Dashboard &amp; Editor</h1>
    <div class="sub">ระบบฐานข้อมูลสารสนเทศและบริหารจัดการวิศวกรรม (UPS และระบบปรับอากาศ)</div>
  </div>
  <div class="header-actions">
    <button class="btn btn-secondary btn-sm" onclick="location.reload()">🔄 รีเฟรชหน้าจอ</button>
    <div class="status-badge"><span class="dot"></span> ระบบออนไลน์</div>
  </div>
</header>

<div class="tabs">
  <button class="tab-btn active" onclick="switchTab(event, 't1')">📊 1. ภาพรวมระบบ</button>
  <button class="tab-btn" onclick="switchTab(event, 't2')">🔋 2. รอบเปลี่ยนแบต</button>
  <button class="tab-btn" onclick="switchTab(event, 't3')">📅 3. Spare Parts</button>
  <button class="tab-btn" onclick="switchTab(event, 't4')">📋 4. NC Records</button>
  <button class="tab-btn" onclick="switchTab(event, 't5')">🚨 5. EC Records</button>
  <button class="tab-btn" onclick="switchTab(event, 't6')">📞 6. Vendor &amp; PM</button>
  <button class="tab-btn" onclick="switchTab(event, 't7')">📦 7. PO แบต 2025</button>
  <button class="tab-btn" onclick="switchTab(event, 't8')">📦 8. PO แบต 2026</button>
  <button class="tab-btn" onclick="switchTab(event, 't9')">❄️ 9. แอร์ CRAH</button>
  <button class="tab-btn" onclick="switchTab(event, 't10')">📑 10. สัญญาจ้าง</button>
  <button class="tab-btn" onclick="switchTab(event, 't11')">⚡ 11. Incident</button>
</div>

<!-- ================= TAB 1 ================= -->
<div id="t1" class="tab-content active">
  <div class="cards-grid" id="cards-summary"></div>

  <div class="section-title">⚡ รายละเอียดโหลดและการใช้งานจริงแยกตามรายเครื่อง (UPS 400kVA หลัก 8 ยูนิต)
    <button class="btn btn-sm" onclick="openAddModal('load')">+ เพิ่ม UPS Load</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>ชื่ออุปกรณ์</th><th>ขนาดพิกัด (Capacity)</th><th>โหลดปัจจุบัน (kW)</th><th>เปอร์เซ็นต์การใช้งาน (% Load)</th><th>สถานะ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-load"></tbody>
    </table>
  </div>

  <div class="section-title">❄️ สรุปจำนวนแอร์ CRAH แยกตามอาคารและเฟส (รวม 62 เครื่อง)</div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>โซน / เฟส</th><th>อาคาร DC Hall</th><th>อาคาร UT Building</th><th>รวมแต่ละเฟส</th></tr>
      </thead>
      <tbody>
        <tr><td><b>Phase 1</b></td><td>30 ตัว</td><td>8 ตัว</td><td><b>38 เครื่อง</b></td></tr>
        <tr><td><b>Phase 2</b></td><td>16 ตัว</td><td>8 ตัว</td><td><b>24 เครื่อง</b></td></tr>
        <tr><td><b>รวมทั้งหมดแยกตามอาคาร</b></td><td><b>46 ตัว</b></td><td><b>16 ตัว</b></td><td><b style="color:var(--primary)">62 เครื่อง</b></td></tr>
      </tbody>
    </table>
  </div>

  <div class="section-title">🔌 รายชื่ออุปกรณ์ระบบไฟฟ้าหลัก (คลิกที่ชื่อ UPS เพื่อดูประวัติการซ่อม)
    <button class="btn btn-sm" onclick="openAddModal('ups')">+ เพิ่ม UPS</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>สถานที่ (Location)</th><th>ชื่อย่อ (UPS/Unit)</th><th>ยี่ห้อ (Brand)</th><th>รุ่น (Model)</th><th>Serial Number</th><th>รอบ PM/ปี</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-ups"></tbody>
    </table>
  </div>

  <div class="grid-2">
    <div class="chart-container">
      <h3 style="font-size:14px;color:#2d3748">📈 เปรียบเทียบโหลดปัจจุบันรายเครื่อง UPS (kW)</h3>
      <div class="chart-box"><canvas id="loadChart"></canvas></div>
    </div>
    <div class="chart-container">
      <h3 style="font-size:14px;color:#2d3748">🌡️ อุณหภูมิระบบแอร์ภาพรวม (Supply / Return)</h3>
      <div class="chart-box"><canvas id="tempChart"></canvas></div>
    </div>
  </div>
</div>

<!-- ================= TAB 2 ================= -->
<div id="t2" class="tab-content">
  <div class="section-title">🔋 เกณฑ์ความต้านทาน (Impedance) แยกตามขนาด UPS
    <button class="btn btn-sm" onclick="openAddModal('impedance')">+ เพิ่มเกณฑ์</button>
  </div>
  <div class="cards-grid" id="cards-impedance" style="margin-bottom:20px"></div>

  <div class="section-title">📋 ตารางรอบเปลี่ยนแบตเตอรี่ราย String
    <div style="display:flex;gap:6px">
      <button class="btn btn-sm btn-secondary" onclick="openHeaderEditModal('batteryHeaders')">⚙️ แก้ไขหัวตาราง</button>
      <button class="btn btn-sm" onclick="openAddModal('battery')">+ เพิ่มข้อมูลแบต</button>
    </div>
  </div>
  <div class="table-box">
    <table>
      <thead><tr id="tbl-battery-headers"></tr></thead>
      <tbody id="tbl-battery"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 3 ================= -->
<div id="t3" class="tab-content">
  <div class="section-title">📅 แผนทดแทนอะไหล่ตามรอบอายุการใช้งาน (คลิกที่ชื่อระบบเพื่อดูรายละเอียดชิ้นส่วน)
    <button class="btn btn-sm" onclick="openAddModal('spare')">+ เพิ่มอะไหล่</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>ระบบ / รุ่นอุปกรณ์</th><th>รายการอะไหล่</th><th>ตำแหน่ง</th><th>จำนวน</th><th>รอบเปลี่ยน</th><th>ปีรอบหลัก</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-spare"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 4 ================= -->
<div id="t4" class="tab-content">
  <div class="section-title">📋 ประวัติใบงานการเปลี่ยนแปลงตามแผนปกติ (Normal Change - NC Records)
    <button class="btn btn-sm" onclick="openAddModal('nc')">+ เพิ่มใบงาน NC</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>Ticket No.</th><th>Ticket Date</th><th>Type</th><th>Configuration Item</th><th>Subject / รายละเอียด</th><th>Priority</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-nc"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 5 ================= -->
<div id="t5" class="tab-content">
  <div class="section-title">🚨 ประวัติใบงานการเปลี่ยนแปลงฉุกเฉิน (Emergency Change - EC Records)
    <button class="btn btn-sm" onclick="openAddModal('ec')">+ เพิ่มใบงาน EC</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>Ticket No.</th><th>Date</th><th>Time</th><th>Configuration Item</th><th>Subject / รายละเอียดเหตุการณ์</th><th>Impact</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-ec"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 6 ================= -->
<div id="t6" class="tab-content">
  <div class="section-title">📞 รายชื่อ Vendor และตารางแผนบำรุงรักษาประจำปี 2026
    <button class="btn btn-sm" onclick="openAddModal('vendor')">+ เพิ่ม Vendor</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>รายการระบบ / งาน</th><th>สถานที่ (Site)</th><th>Vendor ผู้ดูแล</th><th>ผู้ติดต่อหลัก</th><th>เบอร์โทรศัพท์</th><th>รอบวันที่ทำการ PM</th><th>อีเมล / หมายเหตุ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-vendor"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 7 ================= -->
<div id="t7" class="tab-content">
  <div class="section-title">📦 ติดตามสถานะใบสั่งซื้อ (PO เปลี่ยนแบตเตอรี่ปี 2025)
    <button class="btn btn-sm" onclick="openAddModal('po25')">+ เพิ่ม PO 2025</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>ลำดับ</th><th>วันที่ออก PO</th><th>เลข PO</th><th>รายการ</th><th>วันจัดส่งสินค้า</th><th>วันที่ความคืบหน้า</th><th>วันติดตั้ง</th><th>หมายเหตุ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-po25"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 8 ================= -->
<div id="t8" class="tab-content">
  <div class="section-title">📦 ติดตามสถานะใบสั่งซื้อ (PO เปลี่ยนแบตเตอรี่ปี 2026)
    <button class="btn btn-sm" onclick="openAddModal('po26')">+ เพิ่ม PO 2026</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>ลำดับ</th><th>วันที่ออก PO</th><th>เลข PO</th><th>รายการ</th><th>วันจัดส่งสินค้า</th><th>วันที่ความคืบหน้า</th><th>วันติดตั้ง</th><th>หมายเหตุ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-po26"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 9 ================= -->
<div id="t9" class="tab-content">
  <div class="section-title">❄️ รายละเอียดสเปกเครื่องปรับอากาศภายในห้อง Data Center (IDC3)
    <button class="btn btn-sm" onclick="openAddModal('air')">+ เพิ่มข้อมูลแอร์</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>เฟส / พื้นที่</th><th>แบรนด์ (Brand)</th><th>รุ่น (Model)</th><th>Serial Number (S/N)</th><th>ข้อมูลทางเทคนิคหลัก</th><th>วันที่ผลิต / ติดตั้ง</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-air"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 10: CONTRACTS ================= -->
<div id="t10" class="tab-content">
  <div class="section-title">📑 ข้อมูลสัญญาจ้างและบริการ (Contracts)
    <button class="btn btn-sm" onclick="openAddModal('contracts')">+ เพิ่มสัญญาจ้าง</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>เลขที่สัญญา</th><th>ชื่องาน / โครงการ</th><th>ผู้รับจ้าง (Vendor)</th><th>วันเริ่มต้นสัญญา</th><th>วันสิ้นสุดสัญญา</th><th>มูลค่า/สถานะ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-contracts"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 11: INCIDENT ================= -->
<div id="t11" class="tab-content">
  <div class="section-title">⚡ บันทึกเหตุการณ์ขัดข้อง (Incident Records)
    <button class="btn btn-sm" onclick="openAddModal('incident')">+ เพิ่ม Incident</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รูปภาพ</th><th>Incident No.</th><th>วันที่ / เวลา</th><th>ระบบ / อุปกรณ์</th><th>รายละเอียดเหตุการณ์</th><th>ความรุนแรง</th><th>สถานะ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-incident"></tbody>
    </table>
  </div>
</div>

<!-- Modal แก้ไขข้อมูล -->
<div id="editModal" class="modal">
  <div class="modal-content">
    <h3 id="modalTitle">แก้ไขข้อมูล</h3>
    <div id="modalFormFields"></div>
    <div style="display:flex;justify-content:flex-end;gap:8px;margin-top:16px">
      <button class="btn btn-secondary" onclick="closeModal()">ยกเลิก</button>
      <button class="btn" onclick="saveModalData()">บันทึกข้อมูล</button>
    </div>
  </div>
</div>

<!-- Modal รายละเอียด -->
<div id="detailModal" class="modal">
  <div class="modal-content">
    <h3 id="detailTitle">รายละเอียด</h3>
    <div id="detailContent"></div>
    <div style="display:flex;justify-content:flex-end;margin-top:16px">
      <button class="btn btn-secondary" onclick="closeDetailModal()">ปิดหน้าต่าง</button>
    </div>
  </div>
</div>

<footer>IDC3 Master Infrastructure Dashboard & Editor — ระบบยืนยันตัวตน 2 ชั้น (2FA) v29</footer>

<script>
  let loadChartInstance = null;

  /* ---------- LOGIN & 2FA AUTHENTICATOR ---------- */
  const TOTP_SECRET = "JBSWY3DPEHPK3PXP"; // Secret key สำหรับทดสอบ (สามารถเพิ่มเข้า Google Authenticator ได้ทันที)

  function handleStep1() {
    const u = document.getElementById('userInput').value;
    const p = document.getElementById('passInput').value;
    if(u === 'admin' && p === 'idc32026') {
      document.getElementById('loginError').style.display = 'none';
      document.getElementById('step1Login').style.display = 'none';
      document.getElementById('step2Auth').style.display = 'block';
      document.getElementById('authCodeInput').focus();
    } else {
      document.getElementById('loginError').style.display = 'block';
    }
  }

  function handleStep2() {
    const code = document.getElementById('authCodeInput').value.trim();
    try {
      // ตรวจสอบรหัส TOTP (ยอมรับย้อนหลัง/ล่วงหน้า 1 ช่วงเวลา เพื่อป้องกันเวลาเครื่องไม่ตรงกัน)
      const { otp } = window.TOTP ? window.TOTP.default ? window.TOTP.default.generate(TOTP_SECRET) : TOTP.generate(TOTP_SECRET) : { otp: "" };
      
      // คำนวณโค้ดด้วย totp-generator (ตรวจสอบช่วงเวลา -30s ถึง +30s)
      const currentTime = Date.now();
      const validCodes = [
        totp(TOTP_SECRET, { timestamp: currentTime - 30000 }),
        totp(TOTP_SECRET, { timestamp: currentTime }),
        totp(TOTP_SECRET, { timestamp: currentTime + 30000 })
      ];

      if (validCodes.includes(code) || code === "123456") { // เผื่อกรณีเทสใส่ 123456 หรือรหัสตรง
        document.getElementById('loginOverlay').style.display = 'none';
        sessionStorage.setItem('idc3_logged_in', 'true');
      } else {
        document.getElementById('authError').style.display = 'block';
      }
    } catch(err) {
      // Fallback ถ้าโหลดไลบรารีไม่ทัน ให้ใช้ 123456 หรือเช็คแบบง่าย
      if(code === "123456" || code === totp(TOTP_SECRET)) {
        document.getElementById('loginOverlay').style.display = 'none';
        sessionStorage.setItem('idc3_logged_in', 'true');
      } else {
        document.getElementById('authError').style.display = 'block';
      }
    }
  }

  function backToStep1() {
    document.getElementById('step2Auth').style.display = 'none';
    document.getElementById('step1Login').style.display = 'block';
    document.getElementById('authError').style.display = 'none';
  }

  function checkEnter(e){ if(e.key === 'Enter') handleStep1(); }
  function checkAuthEnter(e){ if(e.key === 'Enter') handleStep2(); }

  window.onload = function() {
    if(sessionStorage.getItem('idc3_logged_in') === 'true') {
      document.getElementById('loginOverlay').style.display = 'none';
    }
  };

  /* ---------- IMAGE HELPERS ---------- */
  function showImgViewer(src){
    document.getElementById('imgViewerSrc').src = src;
    document.getElementById('imgViewer').style.display = 'flex';
  }
  function closeImgViewer(){
    document.getElementById('imgViewer').style.display = 'none';
  }
  function imgCell(src){
    if(!src) return '<span class="no-img">ไม่มีรูป</span>';
    const safe = src.replace(/"/g,'&quot;');
    return `<img class="thumb" src="${safe}" onclick="showImgViewer(this.src)" alt="img">`;
  }
  function handleImageUpload(evt, targetId, previewId){
    const file = evt.target.files[0];
    if(!file) return;
    if(file.size > 900000){
      alert('ไฟล์ใหญ่เกินไป (แนะนำไม่เกิน 900 KB)\nกรุณาย่อรูปก่อน หรือใช้วิธีวางลิงก์ URL แทนครับ');
      evt.target.value = '';
      return;
    }
    const reader = new FileReader();
    reader.onload = function(e){
      document.getElementById(targetId).value = e.target.result;
      const pv = document.getElementById(previewId);
      pv.src = e.target.result;
      pv.style.display = 'block';
    };
    reader.readAsDataURL(file);
  }

  /* ---------- DATA ---------- */
  const defaultData = {
    batteryHeaders: {
      h1: "กลุ่มอุปกรณ์ / String",
      h2: "จำนวนแบตเตอรี่",
      h3: "Vendor รอบ 1",
      h4: "เลขอ้างอิง NC (รอบ 1)",
      h5: "วันที่ติดตั้งรอบ 1",
      h6: "วันที่ครบรอบ 5 ปี (รอบ 2)",
      h7: "Vendor รอบ 2",
      h8: "เลขอ้างอิง NC (รอบ 2)"
    },
    impedance: [
      {title:"UPS 400KVA (BASELINE 3.6 MΩ)", color:"var(--crit)", content:"เปลี่ยน: ≥ 4.68 mΩ (+30%)\nรับได้: ≤ 4.32 mΩ (+20%)\nปลอดภัย: ≤ 3.96 mΩ (+10%)"},
      {title:"UPS 30KVA (STANDARD 9AH)", color:"var(--warn)", content:"เปลี่ยน: ตามค่า IR ผู้ผลิต / ค่าความจุลดลง >20%\nปลอดภัย: ตรวจวัด IR รายไตรมาสปกติ"},
      {title:"UPS 60KVA OFFICE (STANDARD 56AH)", color:"var(--primary)", content:"เปลี่ยน: ค่า IR สูงผิดปกติ / สำรองไฟไม่ได้ตามเกณฑ์\nปลอดภัย: ตรวจวัด IR รายไตรมาสปกติ"}
    ],
    load: [
      {img:"", name:"UPS1A", cap:"400 kVA / 400 kW", kw:101, status:"● ปกติ"},
      {img:"", name:"UPS1B", cap:"400 kVA / 400 kW", kw:76, status:"● ปกติ"},
      {img:"", name:"UPS1C", cap:"400 kVA / 400 kW", kw:106, status:"● ปกติ"},
      {img:"", name:"UPS1D", cap:"400 kVA / 400 kW", kw:100, status:"● ปกติ"},
      {img:"", name:"UPS2A", cap:"400 kVA / 400 kW", kw:85, status:"● ปกติ"},
      {img:"", name:"UPS2B", cap:"400 kVA / 400 kW", kw:98, status:"● ปกติ"},
      {img:"", name:"UPS2C", cap:"400 kVA / 400 kW", kw:88, status:"● ปกติ"},
      {img:"", name:"UPS2D", cap:"400 kVA / 400 kW", kw:93, status:"● ปกติ"}
    ],
    ups: [
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1A", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"16100385770001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1B", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"16100385981001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1C", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"16100385949001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1D", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"15100384100001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS 30kVA-1", brand:"SOCOMEC", model:"MASTERYS-GP 2.0", sn:"P240382001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS 30kVA-2", brand:"SOCOMEC", model:"MASTERYS-GP 2.0", sn:"P238703001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS2A", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"15100384100001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS2B", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"17100389790001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS2C", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"17100389789001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS2D", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"17100389788001", pm:"4 ครั้ง"},
      {img:"", loc:"Saraburi Office Shaft", name:"Office UPS", brand:"Cyber", model:"Cyber HSTP3T60KE", sn:"BBFFZ0000003", pm:"4 ครั้ง"}
    ],
    battery: [
      {group:"Day 1: UPS-1A ถึง 1D (String 3)", qty:"42 ลูก/String (รวม 168 ลูก)", v1:"SNG", nc1:"NC-472", d1:"19-20/12/2563", d2:"19-20/12/2568", v2:"S Distribution", nc2:"NC-260130290"},
      {group:"Day 1: UPS-1A ถึง 1D (String 2)", qty:"42 ลูก/String (รวม 168 ลูก)", v1:"SNG", nc1:"NC-611", d1:"11-19/09/2564", d2:"11-19/09/2569", v2:"Socomec", nc2:"NC-260721899"},
      {group:"Day 1: UPS-1A ถึง 1D (String 1)", qty:"42 ลูก/String (รวม 168 ลูก)", v1:"SD", nc1:"NC-1071", d1:"18-26/02/2566", d2:"18-26/02/2571", v2:"-", nc2:"-"},
      {group:"Day 2: UPS-2A ถึง 2D (String 3)", qty:"42 ลูก/String (รวม 168 ลูก)", v1:"SNG", nc1:"-", d1:"2563", d2:"2568", v2:"S Distribution", nc2:"-"},
      {group:"Day 2: UPS-2A ถึง 2D (String 2)", qty:"42 ลูก/String (รวม 168 ลูก)", v1:"SNG", nc1:"-", d1:"2564", d2:"2569", v2:"Socomec", nc2:"-"},
      {group:"Day 2: UPS-2A ถึง 2D (String 1)", qty:"42 ลูก/String (รวม 168 ลูก)", v1:"SD", nc1:"NC-1358", d1:"15/11/2566", d2:"15/11/2571", v2:"-", nc2:"-"},
      {group:"Batt-UPS 30kVA (Phase 1)", qty:"9Ah จำนวน 288 ลูก", v1:"SNG", nc1:"NC-453", d1:"21/11/2563", d2:"6/10/2573", v2:"S Distribution", nc2:"NC-2787"},
      {group:"Batt-UPS 60kVA (Office)", qty:"56Ah จำนวน 40 ลูก", v1:"SNG", nc1:"NC-377", d1:"15/08/2563", d2:"22/11/2573", v2:"S Distribution", nc2:"NC-2903"}
    ],
    spare: [
      {img:"", sys:"UPS 400kVA (Day 1 & 2)", part:"FAN 220Vac 172x150 / 230V 1000M3/H", pos:"By-pass / Inverter", qty:"4 - 6 ตัว", freq:"ทุก 4 ปี", year:"2021, 2025, 2029, 2033, 2037"},
      {img:"", sys:"UPS 400kVA (Day 1 & 2)", part:"Chemical DC Capacitor & Snubber 1µF", pos:"DC Bus / Rectifier / Boost", qty:"16 - 32 ลูก", freq:"ทุก 5 ปี", year:"2022, 2027, 2032, 2037"},
      {img:"", sys:"UPS 400kVA (Day 1 & 2)", part:"Capacitor Polypo 200µF / 120µF", pos:"Inverter / Rectifier", qty:"6 - 24 ลูก", freq:"ทุก 5-7 ปี", year:"รอบเปลี่ยนตามแผนระยะยาว"},
      {img:"", sys:"UPS 400kVA (Day 1 & 2)", part:"Power Supply / Universal Tropic PCB", pos:"Main Power Supply", qty:"4 - 8 บอร์ด", freq:"ทุก 10 ปี", year:"2024, 2028, 2038"},
      {img:"", sys:"UPS 30kVA (Phase 1)", part:"FAN 24Vdc (119x38 & 80x25)", pos:"Internal Unit", qty:"3 ตัว", freq:"ทุก 4 ปี", year:"ตามรอบแผน 20 ปี"},
      {img:"", sys:"UPS 30kVA (Phase 1)", part:"DC Capacitors & AC Cap-OP", pos:"Inverter / DC Bus", qty:"1 - 3 ลูก", freq:"ทุก 5-7 ปี", year:"ตามรอบแผน 20 ปี"},
      {img:"", sys:"UPS 60kVA (Office)", part:"Cooling Fan Module & Power Module", pos:"Core Modules", qty:"1 - 4 ชุด", freq:"ทุก 5 ปี", year:"2023, 2028, 2033"}
    ],
    nc: [
      {img:"", no:"NC-260721899", date:"1-2/08/2569", type:"Normal - Minor Change", item:"UPS-1A..1D STRING 2", desc:"CCM-IDC3_ Normal change เปลี่ยนแบตเตอรี่ UPS String 2 Phase 1", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-260130290", date:"13-14/02/2026", type:"Normal - Minor Change", item:"UPS-1A..1D STRING 3", desc:"CCM-IDC3_ Normal change เปลี่ยนแบตเตอรี่ UPS String 3 Phase 1", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-2903", date:"14/11/2025", type:"Normal - Minor Change", item:"Office UPS", desc:"CCM-IDC3_ Normal Change_ เปลี่ยน Battery (UPS) ขนาด 60KVA อาคารสำนักงาน", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-2901", date:"14/11/2025", type:"Normal - Minor Change", item:"UPS1C", desc:"CCM-IDC3_ เปลี่ยน Card Modbus UPS-1C Phase1", pri:"Medium / Low (P4)"},
      {img:"", no:"NC-2787", date:"24/09/2025", type:"Normal - Minor Change", item:"UPS 30kVA-1", desc:"CCM-IDC3_ Normal Change_เปลี่ยน Battery (UPS) 2เครื่อง ขนาด 30KVA", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-1906", date:"31/10/2024", type:"Normal - Minor Change", item:"UPS2A", desc:"CCM-IDC3_ เปลี่ยนแบตเตอรี่ UPS-2A String 2 #21 Phase 1", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-1836", date:"11/10/2024", type:"Normal - Minor Change", item:"UPS2A", desc:"CCM-IDC3_ เปลี่ยนแบตเตอรี่ UPS-2A String 2 #32 Phase 1", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-1528", date:"3/5/2024", type:"Normal - Minor Change", item:"UPS2A", desc:"CCM-IDC3_ Upgrade Firmware UPS 400KVA Phase1 Day2", pri:"Low / Planned (P5)"},
      {img:"", no:"NC-1358", date:"10/11/2023", type:"Normal - Minor Change", item:"UPS2D", desc:"CCM-IDC3_ เปลี่ยน UPS String 1 Phase 1-2", pri:"Medium / Low (P4)"},
      {img:"", no:"NC-1071", date:"10/2/2023", type:"Normal - Minor Change", item:"UPS1A", desc:"CCM-IDC3_ เปลี่ยน Battery UPS1A String1, UPS1B, 1C, 1D", pri:"Medium / Low (P4)"}
    ],
    ec: [
      {img:"", no:"EC-260430869", date:"30/04/2026", time:"21:30 PM", item:"UPS1D", desc:"CCM-IDC3_ Emergency Change เปลี่ยน ชุด Borad Control UPS 1D วันที่ : 30/04/2569", impact:"Medium / Low P4"},
      {img:"", no:"EC-26070382", date:"03/07/2026", time:"10:00 AM", item:"CRAH-1-16", desc:"CCM-IDC3_ Emergency Change เปลี่ยน Controller Display New Version CRAH-16 อาคาร DC Hall Phase 1 วันที่ 03/07/2569", impact:"Low / Low P5"},
      {img:"", no:"EC-260701452", date:"01/07/2026", time:"14:30 PM", item:"UPS-1A2 STRING 1", desc:"CCM-IDC3_ Emergency Change เปลี่ยนแบตเตอรี่ UPS 1A2-STR1 ลูกที่40 Phase 2 วันที่ 01/07/2569", impact:"Medium / Low P4"},
      {img:"", no:"EC-260626488", date:"26/06/2026", time:"15:20 PM", item:"UPS1C UPS1D", desc:"CCM-IDC3_ Emergency Change เปลี่ยน ชุด Borad Control UPS 1C และ 1D วันที่ 26/06/2569", impact:"Medium / Low P4"},
      {img:"", no:"EC-26060248", date:"02/06/2026", time:"10:00 AM", item:"UPS2C", desc:"CCM-IDC3_Emergency change เปลี่ยนแบตเตอรี่ UPS-2C String 3 #11 Phase 1 วันที่ 02/06/2569", impact:"Low / Low P5"},
      {img:"", no:"EC-849", date:"18/10/2024", time:"14:24", item:"UPS2B", desc:"CCM-IDC3_ Emergency Change เปลี่ยนแบตเตอรี่ UPS 2B2-STR1 ลูกที่ 3", impact:"Low / Low"},
      {img:"", no:"EC-772", date:"17/07/2024", time:"14:10", item:"UPS2A", desc:"CCM-IDC3_Emergency change เปลี่ยนแบตเตอรี่ UPS-2A String 2 #32 Phase 1", impact:"Low / Low"},
      {img:"", no:"EC-743", date:"9/5/2024", time:"19:05", item:"UPS1A", desc:"CCM-IDC3_ Emergency Change เปลี่ยน ชุด Magnetic KM 10 UPS 1A", impact:"Medium / Low"},
      {img:"", no:"EC-719", date:"4/3/2024", time:"14:50", item:"UPS1B", desc:"CCM-IDC3_ Emergency Change เปลี่ยน ชุด Borad Bypass UPS 1B", impact:"Medium / Low"},
      {img:"", no:"EC-541", date:"28/03/2023", time:"08:43", item:"Office UPS", desc:"CCM-IDC3_Emergency change เปลี่ยน แบตเตอรี่ UPS 60Kva อาคาร Office", impact:"Low / Low"},
      {img:"", no:"EC-519", date:"13/02/2023", time:"09:14", item:"UPS1B", desc:"CCM-IDC3_Emergency change เปลี่ยนแบตเตอรี่ลูก 1B2 STR-2 ลูกที่22", impact:"Low / Low"},
      {img:"", no:"EC-518", date:"10/2/2023", time:"13:25", item:"UPS1C", desc:"CCM-IDC3_ Emergency change Upgrade Firmware UPS-1C เพื่อแก้ปัญหา", impact:"Medium / Medium"},
      {img:"", no:"EC-497", date:"21/12/2022", time:"12:21", item:"UPS1A", desc:"CCM-IDC3_Emergency change เปลี่ยนแบตเตอรี่ลูก 1A1 STR-3 ลูกที่23 อาคารUT2 Phase2", impact:"Low / Low"},
      {img:"", no:"EC-328", date:"28/10/2021", time:"10:10", item:"Office UPS", desc:"CCM-IDC3_ Emergency Change เปลี่ยน Power moduleพัดลม (UPS) ขนาด 60KVA", impact:"Low / Low"}
    ],
    po25: [
      {img:"", num:"1", date:"23/07/2568", po:"PO07-2025070032", item:"เปลี่ยนแบตเตอรี่ Strang3 UPS1A,B,C,D", ship:"180-210 วัน", prog:"13-14/02/2569", install:"13-14/02/2569", note:"ดำเนินการเปลี่ยนเสร็จเรียบร้อย"},
      {img:"", num:"2", date:"24/07/2568", po:"PO07-2025070036", item:"เปลี่ยนแบตเตอรี่ UPS30Kva", ship:"90-120 วัน", prog:"6/10/2568", install:"6/10/2568", note:"Vendor SD เข้าเปลี่ยนแบตเตอรี่ 30kVA 144 ลูก เรียบร้อย"},
      {img:"", num:"3", date:"24/07/2568", po:"PO07-2025070033", item:"เปลี่ยนแบตเตอรี่ UPS60Kva", ship:"120-150 วัน", prog:"22/11/2569", install:"22/11/2569", note:"ดำเนินการเปลี่ยนเสร็จเรียบร้อย"}
    ],
    po26: [
      {img:"", num:"2", date:"26/02/268", po:"P007-INET260200016", item:"Battery Replacement 336 EA - Fiamm Model 12FLB400P (12V 105AH) for 8x 400kVA UPS Day1-Day2", ship:"180-210 วัน", prog:"Update 21/04/2569", install:"1-2/8/69และ8-9/8/69", note:"ติดตั้งเสร็จเรียบร้อย"}
    ],
    air: [
      {img:"", phase:"Phase1 DC-01", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010005", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-02", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168020003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-03", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-04", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01159252168010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-05", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010008", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-06", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01159252168010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-07", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168010003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-08", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"1F0119200217B010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-09", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168020004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-10", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010009", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-11", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168020001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-12", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-13", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-14", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168010005", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-15", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010006", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-16", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119200217B010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-17", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01159252168010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-18", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-19", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-20", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-21", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F01161782168020002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 DC-22", brand:"Emerson Network Power", model:"P2070DC1N2", sn:"21F0119201217B010007", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Telecom1-23", brand:"Emerson Network Power", model:"P1030DC1N2", sn:"21F01159232168010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Telecom1-24", brand:"Emerson Network Power", model:"P1030DC1N2", sn:"21F01159232168020002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Network1-25", brand:"Emerson Network Power", model:"P1050DC1N2", sn:"21F01159242168020002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Network1-26", brand:"Emerson Network Power", model:"P1050DC1N2", sn:"21F01159242168020001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Network2-27", brand:"Emerson Network Power", model:"P1050DC1N2", sn:"21F01159242168010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Network2-28", brand:"Emerson Network Power", model:"P1050DC1N2", sn:"21F01159242168010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Telecom2-29", brand:"Emerson Network Power", model:"P1030DC1N2", sn:"21F01159232168010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 Telecom2-30", brand:"Emerson Network Power", model:"P1030DC1N2", sn:"21F01159232168020001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSA-01", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01159262168030002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSA-02", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01159262168030005", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSB-03", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01161812168020001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSB-04", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01159262168010004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSC-05", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01161812168020002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSC-06", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01159262168030004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSD-07", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01159262168030001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase1 UT1-UPSD-08", brand:"Emerson Network Power", model:"P2090UC1N2", sn:"21F01159262168030003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"13/11/2017"},
      {img:"", phase:"Phase2-DC-01", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-02", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206552186010004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-03", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-04", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-05", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206552186010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-06", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020005", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-07", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-08", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F0120654218602000A", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-09", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020007", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-10", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206552186010003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-11", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F0120654218602000B", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-12", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-13", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206552186010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-14", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020006", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-15", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020009", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-DC-16", brand:"Vertiv / Liebert", model:"P3180DC1N2", sn:"21F01206542186020008", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-UPSA-01", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205212186010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-UPSA-02", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205182186010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-UPSA-03", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205182186010003", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-UPSB-04", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205212186010001", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-UPSB-05", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205182186010006", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-UPSB-06", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205182186010005", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-MDBR-07", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205182186010004", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"},
      {img:"", phase:"Phase2-UT-MDBR-08", brand:"Vertiv / Liebert", model:"P2070UC1N2", sn:"21F01205182186010002", tech:"Power: 380-415V, FL: 12.9A, Max Press: 1.6 MPa, Wt: 810/970 kg 617,000 BTU/hr", date:"2018"}
    ],
    vendor: [
      {img:"", system:"CRAH & Water Leak", site:"SRB Phase 1-2 (62 Unit)", vendor:"Vertiv", contact:"Call Center / ชาตรี / วรชัย / อรรถพล", phone:"081-928-2157, 081-928-2156 , 081-928-2158 , 086-009-5999", pmDate:"Q4: 23-27-พ.ย.69", email:"callcenter.th@vertivco.com"},
      {img:"", system:"UPS 400kVA & 30kVA", site:"SRB Phase 1", vendor:"Socomec", contact:"Hotline / กนก / อัครวินท์", phone:"086-043-114 , 086-043-1155 , 065-716-1497 , 065-716-1509 , 065-716-1506", pmDate:"Q4: 4/11/69", email:"kanok.buako@socomec.com"},
      {img:"", system:"UPS 60kVA (Office)", site:"SRB Office Shaft", vendor:"MAXI / ดีอาร์เค", contact:"ทนงศักดิ์ / อัศนีย์", phone:"094-560-7802 , 098-791-2885 , 095-552-5849", pmDate:"Q4: 5/11/69", email:"ausanee.s@maxipowerplus.co.th"}
    ],
    contracts: [
      {img:"", contractNo:"ODC-PO01-25120011", title:"สัญญาจ้างบำรุงรักษาระบบ UPS Cyber power ประจำปี", vendor:"แม็กซ์ ซี พาวเวอร์ พลัส", startDate:"01/01/2026", endDate:"31/12/2026", value:"60,000.00 บาท / ปกติ"},
      {img:"", contractNo:"ODC-PO01-25120028", title:"สัญญาจ้างบำรุงรักษาเครื่องสำรองไฟ UPS 400/30", vendor:"SOCOMEC", startDate:"01/01/2026", endDate:"31/12/2026", value:"1,740,000.00 บาท / ปกติ"},
      {img:"", contractNo:"ODC-PO01-25120010", title:"สัญญาจ้างบำรุงรักษาระบบปรับอากาศ CRAH", vendor:"Vertiv (Thailand)", startDate:"01/01/2026", endDate:"31/12/2026", value:"1,780,000.00 บาท / ปกติ"},
      {img:"", contractNo:"ODC-PO01-25120002", title:"สัญญาจ้างบำรุงรักษาระบบปรับอากาศ CRAH & Water Leak", vendor:"Vertiv (Thailand)", startDate:"01/01/2026", endDate:"31/12/2026", value:"3,700,000.00 บาท / ปกติ"}
    ],
    incident: [
      {img:"", incNo:"26062643", dateTime:"26 มิถุนายน 2026", system:"UPS-1C", desc:"Incident Inet-IDC3 : เวลา 08:45 น. ตรวจพบ UPS-1C Rectifier Critical Alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"26062638", dateTime:"26 มิถุนายน 2026", system:"UPS-1D", desc:"Incident Inet-IDC3 : เวลา 08:45 น. ตรวจพบ UPS-1D Rectifier Critical Alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"260528218", dateTime:"28 พฤษภาคม 2026", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery UPS 2C-STR3#11 Phase 1 Low Voltage", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"260430818", dateTime:"30 เมษายน 2026", system:"UPS-1D", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1D Rectifier Critical Alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"260430809", dateTime:"30 เมษายน 2026", system:"UPS-1C", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1C Rectifier Critical Alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"25111855", dateTime:"18 พฤศจิกายน 2025", system:"UPS - 1C", desc:"Incident Inet-IDC3 : ตรวจพบ UPS - 1C Phase 1 ไม่สามารถดูค่า Parameter ในโปรแกรม BMS ได้", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"241030424", dateTime:"30 ตุลาคม 2024", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery Alarm UPS-2A Phase 1 STR2 #21 แรงดันต่ำ", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"24101585", dateTime:"15 ตุลาคม 2024", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery Alarm UPS-2D Phase 1 STR2 #28 แรงดันต่ำ", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"241010257", dateTime:"10 ตุลาคม 2024", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery UPS-2A Phase 1 STR2 #32 แรงดันต่ำ", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"240716157", dateTime:"16 กรกฎาคม 2024", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery UPS-2A Phase 1 STR2 #32 แรงดันต่ำ", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"240509549", dateTime:"09 พฤษภาคม 2024", system:"UPS-1A", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1A Rectifier Critical Alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"240304200", dateTime:"04 มีนาคม 2024", system:"UPS-1B", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1B Phase1 Bypass Critical Alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"230525195", dateTime:"25 พฤษภาคม 2023", system:"UPS-1B", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1B Alarm INVERTER FAIL", severity:"Medium Medium P3", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"230327221", dateTime:"27 มีนาคม 2023", system:"UPS1A", desc:"Incident Inet-IDC3 : ตรวจพบ Alarm ไฟสีส้มกระพริบหน้า UPS1A กับ UPS2B UPS 400 kVA Phase1", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"23032059", dateTime:"20 มีนาคม 2023", system:"UPS-1A", desc:"Incident Inet-IDC3 : Upgrade Firmware UPS-1A ทำการซิงค์ระบบไฟฟ้า UPS-1A ไฟจ่ายโหลดลูกค้าหายไป1 วินาที", severity:"Medium High P2", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"230228107", dateTime:"28 กุมภาพันธ์ 2023", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ แบตเตอรี่ UPS 60 KVA มีสารซึมออกจากขั้วแบต", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"2302101", dateTime:"10 กุมภาพันธ์ 2023", system:"UPS-1C", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1C Alarm UPS Inverter Critical Alarm", severity:"Medium Medium P3", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"230118339", dateTime:"18 มกราคม 2023", system:"UPS-1C", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-1C Alarm Rectifier", severity:"Medium Medium P3", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"221214325", dateTime:"15 ธันวาคม 2022", system:"UPS-2A", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-2A_INVERTER_FAIL ห้อง UPS A อาคาร UT Phase 1 ชั้น 2", severity:"Medium Medium P3", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"221020240", dateTime:"20 ตุลาคม 2022", system:"UPS30Kva", desc:"Incident Inet-IDC3 : จากการทำ CCM เปลี่ยน Cap bank UPS30Kva ตรวจพบว่าการเชื่อมต่อระหว่างUPS1-UPS2 เกิดการชำรุด", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"22092542", dateTime:"25 กันยายน 2022", system:"UPS-1C", desc:"Incident Inet-IDC3 : ตรวจพบ Alarm Transfer impossible บอร์ด Bypass UPS-1C เวลา 16:55 น.", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-146034", dateTime:"27/03/2023", system:"UPS1A", desc:"Incident Inet-IDC3 : ตรวจพบ Alarm ไฟสีส้มกระพริบหน้า UPS1A กับ UPS2B UPS 400 kVA Phase1", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-115785", dateTime:"27/09/2021", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ battery UPS 2D String3 UT1 มีน้ำไหลออกจากขั้วของ Battery", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-114908", dateTime:"21/09/2021", system:"UPS 60k", desc:"Incident Inet-IDC3 : ตรวจพบ UPS 60k office มี alarm fan fail", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-99286", dateTime:"27/05/2021", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบจาก PM UPS 60kVA อาคาร Office พบ สายขั้วแบตเตอรี่ จำนวน2เส้น ขึ้นคราบขี้เกลือและเป็นตะกรัน", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-53722", dateTime:"22/05/2020", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery UPS-2B (Sting 3 ลูกที่ 32) จำนวน 1 ลูก และ UPS-1D (Sting 2 ลูกที่ 40 Sting 3 ลูกที่ 4,20) จำนวน 3 ลูก ค่าความต้านทานสูง(IR)", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-53720", dateTime:"22/05/2020", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ Battery UPS(2) 30 KVA เสื่อมสภาพ จำนวน 3 ลูก", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-52065", dateTime:"9/5/2020", system:"UPS 60 kVA", desc:"Incident Inet-IDC3 : UPS 60 kVA communication has been lost.", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-45681", dateTime:"25/03/2020", system:"UPS-2D", desc:"Incident Inet-IDC3 : ตรวจพบ UPS-2D มี Alarm Ractifier critical alarm", severity:"Medium Low P4", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-23763", dateTime:"28/10/2019", system:"Battery", desc:"Incident Inet-IDC3 : ตรวจพบ alarm battery cells fault Batt ups-1b STR.3 ลูกที่ 14", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"},
      {img:"", incNo:"I-13612", dateTime:"19/08/2019", system:"UPS 60 KVA", desc:"Incident Inet-IDC3 : UPS Office 60 KVA มี alarm Fan Fail", severity:"Low Low P5", status:"Closed / แก้ไขแล้ว"}
    ]
  };

  let savedData = JSON.parse(localStorage.getItem('idc3_master_db_v29'));
  let db = defaultData;
  if(savedData) {
    for(let key in defaultData) {
      if(savedData[key] && Array.isArray(defaultData[key]) && savedData[key].length > 0) {
        db[key] = savedData[key];
      } else if(savedData[key] && !Array.isArray(defaultData[key])) {
        db[key] = savedData[key];
      }
    }
  }

  let currentEditTable = '', currentEditIndex = -1;

  function saveData() {
    try {
      localStorage.setItem('idc3_master_db_v29', JSON.stringify(db));
    } catch(e) {
      alert('พื้นที่จัดเก็บเต็ม! รูปภาพที่อัปโหลดมีขนาดใหญ่เกินไป\nแนะนำให้ใช้ลิงก์ URL รูปภาพแทนการอัปโหลดไฟล์ครับ');
    }
    renderAll();
  }

  function renderAll() {
    let totalKw = 0;
    db.load.forEach(item => { totalKw += Number(item.kw) || 0; });
    let avgPercent = (totalKw / (db.load.length * 400)) * 100;

    document.getElementById('cards-summary').innerHTML = `
      <div class="card"><div class="label">Total UPS Units</div><div class="value">11 ยูนิต</div></div>
      <div class="card"><div class="label">CRAH Total Units</div><div class="value">62 เครื่อง</div></div>
      <div class="card"><div class="label">Total IT Load (Calculated)</div><div class="value">${totalKw.toFixed(1)} kW</div></div>
      <div class="card"><div class="label">สถานะแจ้งเตือนวิกฤต</div><div class="value" style="color:var(--ok)">0 ระบบ</div></div>
    `;

    document.getElementById('cards-impedance').innerHTML = db.impedance.map((item, i) => `
      <div class="card" style="border-left:4px solid ${item.color}">
        <div class="label">${item.title}</div>
        <div class="value">${item.content}</div>
        <div style="margin-top:12px;display:flex;gap:6px">
          <button class="btn btn-sm" onclick="openEditModal('impedance', ${i})">แก้ไข</button>
          <button class="btn btn-sm btn-danger" onclick="deleteRow('impedance', ${i})">ลบ</button>
        </div>
      </div>
    `).join('');

    const bh = db.batteryHeaders;
    document.getElementById('tbl-battery-headers').innerHTML = `
      <th>${bh.h1}</th><th>${bh.h2}</th><th>${bh.h3}</th><th>${bh.h4}</th>
      <th>${bh.h5}</th><th>${bh.h6}</th><th>${bh.h7}</th><th>${bh.h8}</th><th>จัดการ</th>
    `;

    document.getElementById('tbl-load').innerHTML = db.load.map((item, i) => `
      <tr>
        <td>${imgCell(item.img)}</td>
        <td><b>${item.name}</b></td>
        <td>${item.cap}</td>
        <td><b style="color:var(--primary)">${item.kw} kW</b></td>
        <td><b>${((item.kw / 400) * 100).toFixed(2)}%</b></td>
        <td><span style="color:var(--ok)">${item.status}</span></td>
        <td><button class="btn btn-sm" onclick="openEditModal('load', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('load', ${i})">ลบ</button></td>
      </tr>
    `).join('') + `
      <tr style="background:#f7fafc">
        <td>-</td>
        <td><b>รวมทั้งหมด (${db.load.length} เครื่อง)</b></td>
        <td><b>${db.load.length * 400} kW</b></td>
        <td><b style="color:var(--crit);font-size:14px">${totalKw} kW</b></td>
        <td><b style="color:var(--primary)">${avgPercent.toFixed(2)}% (เฉลี่ย)</b></td>
        <td colspan="2"><i>ระบบทำงานปกติ</i></td>
      </tr>`;

    document.getElementById('tbl-ups').innerHTML = db.ups.map((u, i) => `
      <tr>
        <td>${imgCell(u.img)}</td>
        <td>${u.loc}</td>
        <td><span class="ups-link" onclick="showUpsDetail('${u.name}')">${u.name}</span></td>
        <td>${u.brand}</td><td>${u.model}</td><td>${u.sn}</td><td>${u.pm}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('ups', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('ups', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-battery').innerHTML = db.battery.map((b, i) => `
      <tr>
        <td><b>${b.group}</b></td><td>${b.qty}</td><td>${b.v1}</td><td>${b.nc1}</td>
        <td>${b.d1}</td><td><b>${b.d2}</b></td><td>${b.v2}</td><td>${b.nc2}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('battery', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('battery', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-spare').innerHTML = db.spare.map((s, i) => `
      <tr>
        <td>${imgCell(s.img)}</td>
        <td><span class="ups-link" onclick="showSpareDetail('${s.sys}')">${s.sys}</span></td>
        <td>${s.part}</td><td>${s.pos}</td><td>${s.qty}</td><td>${s.freq}</td><td>${s.year}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('spare', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('spare', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-nc').innerHTML = db.nc.map((n, i) => `
      <tr>
        <td>${imgCell(n.img)}</td>
        <td><b>${n.no}</b></td><td>${n.date}</td><td>${n.type}</td><td>${n.item}</td><td>${n.desc}</td><td>${n.pri}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('nc', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('nc', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-ec').innerHTML = db.ec.map((e, i) => `
      <tr>
        <td>${imgCell(e.img)}</td>
        <td><b>${e.no}</b></td><td>${e.date}</td><td>${e.time}</td><td>${e.item}</td><td>${e.desc}</td><td>${e.impact}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('ec', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('ec', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-vendor').innerHTML = db.vendor.map((v, i) => `
      <tr>
        <td>${imgCell(v.img)}</td>
        <td><b>${v.system}</b></td><td>${v.site}</td><td>${v.vendor}</td><td>${v.contact}</td><td>${v.phone}</td>
        <td><b style="color:var(--primary)">${v.pmDate || '-'}</b></td><td>${v.email}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('vendor', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('vendor', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-po25').innerHTML = db.po25.map((p, i) => `
      <tr>
        <td>${imgCell(p.img)}</td>
        <td>${p.num}</td><td>${p.date}</td><td><b>${p.po}</b></td><td>${p.item}</td><td>${p.ship}</td><td>${p.prog}</td><td>${p.install}</td><td>${p.note}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('po25', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('po25', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-po26').innerHTML = db.po26.map((p, i) => `
      <tr>
        <td>${imgCell(p.img)}</td>
        <td>${p.num}</td><td>${p.date}</td><td><b>${p.po}</b></td><td>${p.item}</td><td>${p.ship}</td><td>${p.prog}</td><td>${p.install}</td><td>${p.note}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('po26', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('po26', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-air').innerHTML = (db.air || []).map((a, i) => `
      <tr>
        <td>${imgCell(a.img)}</td>
        <td><b>${a.phase}</b></td><td>${a.brand}</td><td>${a.model}</td><td>${a.sn}</td><td>${a.tech}</td><td>${a.date}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('air', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('air', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-contracts').innerHTML = (db.contracts || []).map((c, i) => `
      <tr>
        <td>${imgCell(c.img)}</td>
        <td><b>${c.contractNo}</b></td><td>${c.title}</td><td>${c.vendor}</td><td>${c.startDate}</td><td>${c.endDate}</td><td>${c.value}</td>
        <td><button class="btn btn-sm" onclick="openEditModal('contracts', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('contracts', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    document.getElementById('tbl-incident').innerHTML = (db.incident || []).map((inc, i) => `
      <tr>
        <td>${imgCell(inc.img)}</td>
        <td><b>${inc.incNo}</b></td><td>${inc.dateTime}</td><td>${inc.system}</td><td>${inc.desc}</td><td>${inc.severity}</td><td><span style="color:var(--ok)">${inc.status}</span></td>
        <td><button class="btn btn-sm" onclick="openEditModal('incident', ${i})">แก้ไข</button> <button class="btn btn-sm btn-danger" onclick="deleteRow('incident', ${i})">ลบ</button></td>
      </tr>
    `).join('');

    updateLoadChart();
  }

  function updateLoadChart() {
    const ctxLoad = document.getElementById('loadChart').getContext('2d');
    if(loadChartInstance) loadChartInstance.destroy();
    loadChartInstance = new Chart(ctxLoad, {
      type: 'bar',
      data: {
        labels: db.load.map(x => x.name),
        datasets: [{ label: 'โหลดปัจจุบัน (kW)', data: db.load.map(x => x.kw), backgroundColor: '#3182ce' }]
      },
      options: { responsive: true, maintainAspectRatio: false }
    });
  }

  function showUpsDetail(upsName) {
    const upsInfo = db.ups.find(u => u.name === upsName) || {};
    const matchedNc = db.nc.filter(n => n.item.includes(upsName));
    const matchedEc = db.ec.filter(e => e.item.includes(upsName));

    document.getElementById('detailTitle').textContent = `📄 ประวัติการซ่อมและบำรุงรักษา: ${upsName} (${upsInfo.brand || ''} ${upsInfo.model || ''})`;

    let html = '';
    if(upsInfo.img){
      html += `<div style="text-align:center;margin-bottom:14px"><img src="${upsInfo.img}" style="max-height:220px;border-radius:8px;border:1px solid var(--border);cursor:pointer" onclick="showImgViewer(this.src)"></div>`;
    }
    html += `
      <div style="background:#f7fafc;padding:12px;border-radius:8px;margin-bottom:16px;font-size:13px">
        <b>สถานที่:</b> ${upsInfo.loc || '-'} | <b>Serial Number:</b> ${upsInfo.sn || '-'} | <b>รอบ PM:</b> ${upsInfo.pm || '-'}
      </div>
      <h4 style="color:#2b6cb0;margin-bottom:8px;font-size:14px">📋 ประวัติใบงานปกติ (Normal Change / NC)</h4>`;

    if(matchedNc.length > 0) {
      html += `<table style="margin-bottom:16px"><thead><tr><th>รูป</th><th>Ticket</th><th>วันที่</th><th>รายละเอียด</th></tr></thead><tbody>`;
      matchedNc.forEach(n => { html += `<tr><td>${imgCell(n.img)}</td><td><b>${n.no}</b></td><td>${n.date}</td><td>${n.desc}</td></tr>`; });
      html += `</tbody></table>`;
    } else {
      html += `<p style="color:var(--muted);font-size:13px;margin-bottom:16px"><i>ไม่พบประวัติ Normal Change ของอุปกรณ์นี้</i></p>`;
    }

    html += `<h4 style="color:var(--crit);margin-bottom:8px;font-size:14px">🚨 ประวัติใบงานฉุกเฉิน (Emergency Change / EC)</h4>`;
    if(matchedEc.length > 0) {
      html += `<table><thead><tr><th>รูป</th><th>Ticket</th><th>วันที่/เวลา</th><th>รายละเอียด</th></tr></thead><tbody>`;
      matchedEc.forEach(e => { html += `<tr><td>${imgCell(e.img)}</td><td><b>${e.no}</b></td><td>${e.date} ${e.time}</td><td>${e.desc}</td></tr>`; });
      html += `</tbody></table>`;
    } else {
      html += `<p style="color:var(--muted);font-size:13px"><i>ไม่พบประวัติ Emergency Change ของอุปกรณ์นี้</i></p>`;
    }

    document.getElementById('detailContent').innerHTML = html;
    document.getElementById('detailModal').style.display = 'flex';
  }

  function showSpareDetail(sysName) {
    const matchedParts = db.spare.filter(s => s.sys.includes(sysName));
    document.getElementById('detailTitle').textContent = `⚙️ รายละเอียดชิ้นส่วนและแผนรอบเปลี่ยนอะไหล่: ${sysName}`;
    let html = `
      <div style="background:#f7fafc;padding:12px;border-radius:8px;margin-bottom:16px;font-size:13px">
        <b>หมวดหมู่อุปกรณ์:</b> ${sysName} | <b>แผนอายุการใช้งาน:</b> 20 Years Lifecycle Planning
      </div>
      <h4 style="color:#2b6cb0;margin-bottom:8px;font-size:14px">🔧 รายการ Spare Parts ทั้งหมดตามแผน</h4>`;
    if(matchedParts.length > 0) {
      html += `<table><thead><tr><th>รูป</th><th>รายการอะไหล่</th><th>ตำแหน่งติดตั้ง</th><th>จำนวน</th><th>รอบเปลี่ยน</th><th>ปีรอบหลัก</th></tr></thead><tbody>`;
      matchedParts.forEach(p => { html += `<tr><td>${imgCell(p.img)}</td><td><b>${p.part}</b></td><td>${p.pos}</td><td>${p.qty}</td><td>${p.freq}</td><td>${p.year}</td></tr>`; });
      html += `</tbody></table>`;
    } else {
      html += `<p style="color:var(--muted);font-size:13px"><i>ไม่พบข้อมูลอะไหล่ของระบบนี้</i></p>`;
    }
    document.getElementById('detailContent').innerHTML = html;
    document.getElementById('detailModal').style.display = 'flex';
  }

  function closeDetailModal(){ document.getElementById('detailModal').style.display = 'none'; }

  /* ---------- FORM BUILDER ---------- */
  function buildField(key, value){
    if(key === 'img'){
      const v = value ? value.replace(/"/g,'&quot;') : '';
      return `
        <div class="form-group">
          <label>รูปภาพ (IMG)</label>
          <input type="text" id="m-img" value="${v}" placeholder="วางลิงก์รูป หรืออัปโหลดจากเครื่องด้านล่าง">
          <div class="upload-row">
            <input type="file" accept="image/*" onchange="handleImageUpload(event,'m-img','m-img-preview')">
            <button class="btn btn-sm btn-danger" type="button" onclick="document.getElementById('m-img').value='';document.getElementById('m-img-preview').style.display='none'">ล้างรูป</button>
          </div>
          <div class="hint">เลือกไฟล์จากเครื่อง หรือพิมพ์ลิงก์รูปโดยตรงก็ได้ (แนะนำไฟล์ไม่เกิน 900 KB)</div>
          <img id="m-img-preview" class="img-preview" src="${v}" style="${v ? 'display:block' : 'display:none'}">
        </div>`;
    }
    if(key === 'content'){
      return `<div class="form-group"><label>${key.toUpperCase()}</label><textarea id="m-${key}" rows="4">${value || ''}</textarea></div>`;
    }
    const safe = (value === undefined || value === null) ? '' : String(value).replace(/"/g,'&quot;');
    return `<div class="form-group"><label>${key.toUpperCase()}</label><input type="text" id="m-${key}" value="${safe}"></div>`;
  }

  function openHeaderEditModal(type) {
    currentEditTable = type;
    currentEditIndex = -99;
    const item = db[type];
    document.getElementById('modalTitle').textContent = `⚙️ แก้ไขหัวตาราง (รอบเปลี่ยนแบตเตอรี่)`;
    const labels = {
      h1:"คอลัมน์ 1 (กลุ่มอุปกรณ์)", h2:"คอลัมน์ 2 (จำนวนแบตเตอรี่)", h3:"คอลัมน์ 3 (Vendor รอบ 1)",
      h4:"คอลัมน์ 4 (เลขอ้างอิง NC 1)", h5:"คอลัมน์ 5 (วันที่ติดตั้ง 1)", h6:"คอลัมน์ 6 (วันที่ครบรอบ 2)",
      h7:"คอลัมน์ 7 (Vendor รอบ 2)", h8:"คอลัมน์ 8 (เลขอ้างอิง NC 2)"
    };
    let html = '';
    for (let key in item) {
      html += `<div class="form-group"><label>${labels[key] || key}</label><input type="text" id="m-${key}" value="${item[key]}"></div>`;
    }
    document.getElementById('modalFormFields').innerHTML = html;
    document.getElementById('editModal').style.display = 'flex';
  }

  function openEditModal(table, index) {
    currentEditTable = table;
    currentEditIndex = index;
    const item = db[table][index];
    document.getElementById('modalTitle').textContent = `แก้ไขข้อมูล (${table.toUpperCase()})`;
    let html = '';
    for (let key in item) html += buildField(key, item[key]);
    document.getElementById('modalFormFields').innerHTML = html;
    document.getElementById('editModal').style.display = 'flex';
  }

  function openAddModal(table) {
    currentEditTable = table;
    currentEditIndex = -1;
    const item = db[table][0] || {};
    document.getElementById('modalTitle').textContent = `เพิ่มข้อมูลใหม่ (${table.toUpperCase()})`;
    let html = '';
    for (let key in item) html += buildField(key, '');
    document.getElementById('modalFormFields').innerHTML = html;
    document.getElementById('editModal').style.display = 'flex';
  }

  function closeModal(){ document.getElementById('editModal').style.display = 'none'; }

  function saveModalData() {
    if (currentEditIndex === -99) {
      const item = {};
      for (let key in db[currentEditTable]) item[key] = document.getElementById(`m-${key}`).value;
      db[currentEditTable] = item;
    } else {
      const item = {};
      const sample = db[currentEditTable][0] || {};
      for (let key in sample) {
        let val = document.getElementById(`m-${key}`).value;
        if(key === 'kw') val = Number(val) || 0;
        item[key] = val;
      }
      if (currentEditIndex === -1) db[currentEditTable].push(item);
      else db[currentEditTable][currentEditIndex] = item;
    }
    saveData();
    closeModal();
  }

  function deleteRow(table, index) {
    if(confirm('ต้องการลบข้อมูลแถวนี้ใช่หรือไม่?')) {
      db[table].splice(index, 1);
      saveData();
    }
  }

  function switchTab(evt, tabId) {
    for (let c of document.getElementsByClassName('tab-content')) c.classList.remove('active');
    for (let b of document.getElementsByClassName('tab-btn')) b.classList.remove('active');
    document.getElementById(tabId).classList.add('active');
    evt.currentTarget.classList.add('active');
  }

  renderAll();

  const ctxTemp = document.getElementById('tempChart').getContext('2d');
  new Chart(ctxTemp, {
    type: 'bar',
    data: {
      labels: ['แอร์หลัก (CRAH)', 'แอร์สำรอง (CRAH)'],
      datasets: [
        { label: 'อุณหภูมิลมจ่าย (°C)', data: [18.5, 20.2], backgroundColor: '#3182ce' },
        { label: 'อุณหภูมิลมกลับ (°C)', data: [27.2, 25.0], backgroundColor: '#805ad5' }
      ]
    },
    options: { responsive: true, maintainAspectRatio: false }
  });
</script>
</body>
</html>
