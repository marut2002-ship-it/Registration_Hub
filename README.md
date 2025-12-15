<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ระบบลงทะเบียนอบรม</title>
  
  <!-- CSS Frameworks & Fonts -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
  <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600&display=swap" rel="stylesheet">
  
  <!-- SweetAlert2 -->
  <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

  <style>
    /* Custom Corporate Style */
    :root {
      --primary-color: #0d47a1; /* Navy Blue */
      --secondary-color: #6c757d;
      --bg-color: #f4f6f9;
    }
    
    body {
      font-family: 'Sarabun', sans-serif;
      background-color: var(--bg-color);
      color: #333;
    }
    
    .card {
      border: none;
      border-radius: 15px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.05);
      transition: transform 0.2s;
    }
    
    .btn-primary {
      background-color: var(--primary-color);
      border-color: var(--primary-color);
      border-radius: 8px;
      padding: 10px 20px;
    }
    
    .btn-primary:hover {
      background-color: #08337a;
    }

    .form-control, .form-select {
      border-radius: 8px;
      padding: 12px;
      border: 1px solid #e0e0e0;
    }
    
    .form-control:focus, .form-select:focus {
      box-shadow: 0 0 0 3px rgba(13, 71, 161, 0.15);
      border-color: var(--primary-color);
    }

    .hidden { display: none !important; }

    /* Loading Spinner Overlay */
    #loading-overlay {
      position: fixed;
      top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(255,255,255,0.98);
      z-index: 9999;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      transition: opacity 0.3s ease;
    }
  </style>
</head>
<body>

  <!-- Loading Overlay -->
  <div id="loading-overlay">
    <div class="spinner-border text-primary mb-3" role="status" style="width: 3rem; height: 3rem;">
      <span class="visually-hidden">Loading...</span>
    </div>
    <div class="text-muted fw-bold" id="loading-text">กำลังโหลดข้อมูล...</div>
    <small class="text-secondary mt-2" id="loading-subtext" style="display:none;">ระบบกำลังเตรียมพร้อม...</small>
  </div>

  <div class="container py-5">
    
    <!-- === SCENARIO A: LOGIN VIEW === -->
    <div id="view-login" class="row justify-content-center hidden">
      <div class="col-md-5">
        <div class="text-center mb-4">
          <i class="fas fa-chalkboard-teacher fa-3x text-primary mb-3"></i>
          <h3 class="fw-bold text-primary">Instructor Portal</h3>
          <p class="text-muted">ระบบจัดการการอบรมสำหรับวิทยากร</p>
          <small class="text-secondary bg-white px-2 py-1 rounded border shadow-sm">Test Account: admin / 1234</small>
        </div>
        <div class="card p-4">
          <form id="form-login" onsubmit="handleLogin(event)">
            <div class="mb-3">
              <label class="form-label">Username</label>
              <input type="text" class="form-control" name="username" required placeholder="กรอกชื่อผู้ใช้งาน" autocomplete="username">
            </div>
            <div class="mb-4">
              <label class="form-label">Password</label>
              <input type="password" class="form-control" name="password" required placeholder="กรอกรหัสผ่าน" autocomplete="current-password">
            </div>
            <button type="submit" class="btn btn-primary w-100 fw-bold">
              <i class="fas fa-sign-in-alt me-2"></i> เข้าสู่ระบบ
            </button>
          </form>
        </div>
      </div>
    </div>

    <!-- === SCENARIO A: DASHBOARD VIEW === -->
    <div id="view-dashboard" class="hidden">
      <nav class="navbar navbar-light bg-white rounded-3 shadow-sm px-4 mb-4 d-flex justify-content-between">
        <span class="navbar-brand mb-0 h1 text-primary"><i class="fas fa-user-tie me-2"></i>สวัสดี, <span id="user-display-name">User</span></span>
        <button class="btn btn-outline-danger btn-sm" onclick="logout()">ออกจากระบบ</button>
      </nav>

      <div class="row mb-4">
        <div class="col-12 d-flex justify-content-between align-items-center">
          <h4 class="fw-bold mb-0">คลาสสอนของฉัน</h4>
          <button class="btn btn-primary" onclick="showCreateCourseModal()">
            <i class="fas fa-plus-circle me-2"></i> เปิดคลาสเรียนใหม่
          </button>
        </div>
      </div>

      <div class="card p-0 overflow-hidden">
        <div class="table-responsive">
          <table class="table table-hover mb-0 align-middle">
            <thead class="bg-light">
              <tr>
                <th class="p-3">หัวข้อ</th>
                <th>วัน-เวลา</th>
                <th>ระยะเวลา</th>
                <th class="text-end p-3">จัดการ</th>
              </tr>
            </thead>
            <tbody id="course-list-body">
              <!-- JS จะเติมข้อมูลตรงนี้ -->
            </tbody>
          </table>
        </div>
        <div id="empty-state" class="text-center py-5 hidden">
          <p class="text-muted">ยังไม่มีคลาสเรียน กดปุ่ม "เปิดคลาสเรียนใหม่" เพื่อเริ่มสร้าง</p>
        </div>
      </div>
    </div>

    <!-- === SCENARIO B: STUDENT REGISTER VIEW === -->
    <div id="view-register" class="row justify-content-center hidden">
      <div class="col-md-6">
        <div class="text-center mb-4">
          <h2 class="fw-bold text-primary">ลงทะเบียนเข้าอบรม</h2>
          <p class="text-muted">กรุณากรอกข้อมูลให้ครบถ้วน</p>
        </div>
        
        <!-- Course Info Card -->
        <div class="card mb-4 bg-primary text-white">
          <div class="card-body p-4">
            <h5 class="card-title text-white-50 mb-1">หัวข้อการอบรม</h5>
            <h3 class="card-text fw-bold mb-3" id="reg-topic">กำลังโหลด...</h3>
            <div class="d-flex justify-content-between">
              <span><i class="far fa-calendar-alt me-2"></i><span id="reg-date">-</span></span>
              <span><i class="far fa-clock me-2"></i><span id="reg-duration">-</span> ชม.</span>
            </div>
            <div class="mt-2 text-white-50 small">
              <i class="fas fa-user me-2"></i>วิทยากร: <span id="reg-instructor">-</span>
            </div>
          </div>
        </div>

        <!-- Registration Form -->
        <div class="card p-4">
          <form id="form-register" onsubmit="handleRegister(event)">
            <input type="hidden" name="courseId" id="reg-course-id">
            
            <div class="mb-3">
              <label class="form-label">ชื่อ-นามสกุล</label>
              <input type="text" class="form-control" name="studentName" required placeholder="นาย กอ ไก่">
            </div>
            <div class="mb-4">
              <label class="form-label">แผนก / ฝ่าย</label>
              <input type="text" class="form-control" name="department" required placeholder="เช่น IT, HR, Marketing">
            </div>
            
            <button type="submit" class="btn btn-primary w-100 fw-bold py-3">
              <i class="fas fa-check-circle me-2"></i> ยืนยันการลงทะเบียน
            </button>
          </form>
        </div>
      </div>
    </div>

  </div> <!-- End Container -->

  <!-- Modal: Create Course -->
  <div class="modal fade" id="createCourseModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content border-0 shadow">
        <div class="modal-header">
          <h5 class="modal-title fw-bold">สร้างคลาสเรียนใหม่</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <form id="form-create-course" onsubmit="handleCreateCourse(event)">
            <div class="mb-3">
              <label class="form-label">หัวข้อการอบรม</label>
              <select class="form-select" name="topic" id="courseTopicSelect" required>
                <option value="" selected disabled>เลือกหัวข้อการอบรม...</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label">วันและเวลา</label>
              <input type="datetime-local" class="form-control" name="dateTime" required>
            </div>
            <div class="mb-3">
              <label class="form-label">ระยะเวลา (ชั่วโมง)</label>
              <input type="number" class="form-control" name="duration" step="0.5" required>
            </div>
            <div class="d-grid">
              <button type="submit" class="btn btn-primary">บันทึกข้อมูล</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal: QR Code Result -->
  <div class="modal fade" id="qrModal" tabindex="-1">
    <div class="modal-dialog text-center">
      <div class="modal-content border-0 shadow">
        <div class="modal-body p-5">
          <h4 class="fw-bold text-success mb-3"><i class="fas fa-check-circle"></i> สร้างคลาสสำเร็จ!</h4>
          <p class="text-muted">ให้นักเรียนสแกน QR Code นี้เพื่อลงทะเบียน</p>
          
          <img id="qr-image" src="" alt="QR Code" class="img-fluid border p-2 rounded mb-3" style="max-width: 250px;">
          
          <div class="input-group mb-3">
            <input type="text" class="form-control" id="course-link" readonly>
            <button class="btn btn-outline-secondary" type="button" onclick="copyLink()">
              <i class="far fa-copy"></i>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- FIX: เพิ่ม Bootstrap JS Bundle เพื่อให้ Modal ทำงานได้ -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>

  <script>
    // --- Configuration for External Hosting (GitHub Pages / Firebase) ---
    // ***************************************************************
    // ⬇️⬇️⬇️ นำ URL ที่ได้จากการ Deploy Google Apps Script (Web App URL) มาวางตรงนี้ ⬇️⬇️⬇️
    const GAS_SCRIPT_URL = 'https://script.google.com/a/macros/srisawadpower.com/s/AKfycbwIHNohXYsqPkcDNcV4jiiOi-OMaKwX2LhKk9B8qiL8nU_QQ_BJjENqt08pQ7PzBCQZ/exec';
    // ***************************************************************

    // --- Safe Backend Wrapper for External Site ---
    var Backend = {
      run: function(funcName, arg1, arg2) {
        return new Promise(function(resolve, reject) {
          
          // ตรวจสอบว่าใส่ URL หรือยัง
          if (!GAS_SCRIPT_URL || GAS_SCRIPT_URL.includes('วางลิงก์')) {
             // ถ้ายังไม่ใส่ ให้ใช้ Mock Data ไปก่อนเพื่อไม่ให้เว็บพังตอนเทสหน้าตา
             console.warn("ยังไม่ได้ใส่ Web App URL, กำลังใช้ Mock Data...");
             runMock(funcName, arg1, arg2, resolve, reject);
             return;
          }

          // เตรียมข้อมูลส่งไปที่ Google Apps Script (doPost)
          var payload = { action: funcName };

          // Mapping Arguments (จับคู่ตัวแปรให้ตรงกับที่ doPost ใน Code.gs ต้องการ)
          if (funcName === 'login') {
            payload.username = arg1;
            payload.password = arg2;
          } 
          else if (funcName === 'getInstructorCourses') {
            payload.username = arg1;
          }
          else if (funcName === 'createCourse' || funcName === 'registerStudent') {
            // arg1 คือ object (formObj) ให้แตกออกมาใส่ payload เลย
            Object.assign(payload, arg1);
          }
          else if (funcName === 'getCourseDetails') {
            payload.courseId = arg1;
          }
          // getTopics ไม่ต้องส่ง parameter เพิ่ม

          // ยิง Request ไปที่ Google Sheet
          fetch(GAS_SCRIPT_URL, {
            method: 'POST',
            redirect: "follow", // ให้ตาม Redirect ของ Google ไป
            body: JSON.stringify(payload),
            headers: {
              "Content-Type": "text/plain;charset=utf-8", // ต้องเป็น text/plain เพื่อเลี่ยง CORS Preflight
            },
          })
          .then(function(response) { return response.json(); })
          .then(function(data) { resolve(data); })
          .catch(function(error) { 
            console.error("API Error:", error);
            reject("เชื่อมต่อ Server ไม่ได้: " + error.toString()); 
          });
        });
      },
      
      getParams: function() {
        return new Promise(function(resolve) {
           // บนเว็บภายนอก เราดึง parameter จาก URL ของ Browser ได้เลย
           const urlParams = new URLSearchParams(window.location.search);
           const params = {};
           for (const [key, value] of urlParams) {
             params[key] = value;
           }
           resolve(params);
        });
      },

      getScriptUrl: function() {
        // สำหรับเว็บนอก URL ของ Script คือ URL ของหน้าเว็บปัจจุบัน
        // ส่วน API Endpoint คือ GAS_SCRIPT_URL
        // แต่ในบริบทนี้เราต้องการลิงก์สำหรับสร้าง QR Code ซึ่งควรจะเป็นลิงก์ของหน้าเว็บนี้
        return Promise.resolve(window.location.href.split('?')[0]); 
      }
    };

    // --- Mock Data Function (Backup) ---
    function runMock(funcName, arg1, arg2, resolve, reject) {
       var MockData = {
          getScriptUrl: function() { return "https://example.com/myapp"; },
          login: function(u, p) {
            if(u==='admin' && p==='1234') return {status:'success', user:{username:'admin', name:'Admin Mock', role:'Instructor'}};
            return {status:'error', message:'Login Failed (Try: admin / 1234)'};
          },
          getInstructorCourses: function(u) {
            return { 
              status: 'success', 
              data: [
                { id: 'mock-1', topic: 'Google Sheet Basic', date: '01/01/2024 10:00', duration: 3 },
                { id: 'mock-2', topic: 'Leadership 101', date: '02/01/2024 13:00', duration: 4 }
              ] 
            };
          },
          getTopics: function() {
            return {
              status: 'success',
              data: ['Google Sheet Basic', 'Advanced Python', 'Leadership 101', 'Mock Topic']
            };
          },
          createCourse: function(f) { return { status: 'success', courseId: 'new-mock-' + Date.now() }; },
          getCourseDetails: function(id) {
            return {
              status: 'success',
              data: { topic: 'Mock Class for Student', date: '15/05/2024 13:00', duration: 4, instructor: 'Admin Mock' }
            };
          },
          registerStudent: function(f) { return { status: 'success', regId: 'REG-MOCK-999' }; }
        };

        setTimeout(function() {
          if (MockData[funcName]) {
            var res = (arg2 !== undefined) ? MockData[funcName](arg1, arg2) 
                    : (arg1 !== undefined) ? MockData[funcName](arg1) 
                    : MockData[funcName]();
            resolve(res);
          } else {
            reject("Function not found in Mock");
          }
        }, 600);
    }

    // --- 3. Global State ---
    var currentUser = null;
    var scriptUrl = '';

    // --- 4. Initialization & Safety Net ---
    
    setTimeout(function() {
      var overlay = document.getElementById('loading-overlay');
      if (overlay && !overlay.classList.contains('hidden')) {
        console.warn("Safety Timeout: Forcing UI unlock");
        toggleLoading(false);
        if (document.getElementById('view-login').classList.contains('hidden') && 
            document.getElementById('view-dashboard').classList.contains('hidden') &&
            document.getElementById('view-register').classList.contains('hidden')) {
          showView('view-login');
        }
      }
    }, 2500);

    window.addEventListener('load', function() {
      // 1. Get Current Page URL (For QR Code generation)
      Backend.getScriptUrl()
        .then(function(url) {
          scriptUrl = url;
          return Backend.getParams();
        })
        .then(function(params) {
          if (params.page === 'register' && params.classId) {
            loadStudentView(params.classId);
          } else {
            showView('view-login');
            toggleLoading(false);
          }
        })
        .catch(function(err) {
          console.error("Init Error:", err);
          showView('view-login');
          toggleLoading(false);
        });
    });

    // --- 5. Application Logic ---

    function handleLogin(e) {
      e.preventDefault();
      var btn = e.target.querySelector('button');
      setLoadingBtn(btn, true);

      var username = e.target.username.value;
      var password = e.target.password.value;

      Backend.run('login', username, password)
        .then(function(res) {
          setLoadingBtn(btn, false);
          if (res.status === 'success') {
            currentUser = res.user;
            document.getElementById('user-display-name').innerText = currentUser.name;
            loadDashboard();
          } else {
            Swal.fire('Login Failed', res.message, 'error');
          }
        })
        .catch(function(err) {
          setLoadingBtn(btn, false);
          Swal.fire('Error', err.toString(), 'error');
        });
    }

    function loadDashboard() {
      showView('view-dashboard');
      toggleLoading(true);
      
      Backend.run('getInstructorCourses', currentUser.username)
        .then(function(res) {
          toggleLoading(false);
          if (res.status === 'success') {
            renderCourseList(res.data);
          }
        })
        .catch(function(err) {
          toggleLoading(false);
          console.error(err);
        });
    }

    function renderCourseList(courses) {
      var tbody = document.getElementById('course-list-body');
      tbody.innerHTML = '';
      
      if (!courses || courses.length === 0) {
        document.getElementById('empty-state').classList.remove('hidden');
        return;
      }
      
      document.getElementById('empty-state').classList.add('hidden');
      
      courses.forEach(function(c) {
        var tr = document.createElement('tr');
        tr.innerHTML = 
          '<td class="p-3 fw-bold text-primary">' + c.topic + '</td>' +
          '<td>' + c.date + '</td>' +
          '<td>' + c.duration + ' ชม.</td>' +
          '<td class="text-end p-3">' +
             '<button class="btn btn-sm btn-outline-secondary" onclick="showQrAgain(\'' + c.id + '\')"><i class="fas fa-qrcode"></i></button>' +
          '</td>';
        tbody.appendChild(tr);
      });
    }

    function showCreateCourseModal() {
      var select = document.getElementById('courseTopicSelect');
      var modal = new bootstrap.Modal(document.getElementById('createCourseModal'));
      modal.show();
      
      select.innerHTML = '<option value="" selected disabled>กำลังโหลดรายชื่อวิชา...</option>';

      Backend.run('getTopics')
        .then(function(res) {
          if (res.status === 'success' && res.data.length > 0) {
            select.innerHTML = '<option value="" selected disabled>เลือกหัวข้อการอบรม...</option>';
            res.data.forEach(function(topic) {
              var option = document.createElement('option');
              option.value = topic;
              option.text = topic;
              select.appendChild(option);
            });
          } else {
             select.innerHTML = '<option value="" selected disabled>ไม่พบข้อมูลรายวิชา</option>';
          }
        })
        .catch(function(err) {
          console.error(err);
          select.innerHTML = '<option value="" selected disabled>โหลดข้อมูลล้มเหลว (ใส่ URL ถูกต้องไหม?)</option>';
        });
    }

    function handleCreateCourse(e) {
      e.preventDefault();
      var btn = e.target.querySelector('button');
      setLoadingBtn(btn, true);

      var formObj = {
        topic: e.target.topic.value,
        dateTime: e.target.dateTime.value,
        duration: e.target.duration.value,
        createdBy: currentUser.username
      };

      Backend.run('createCourse', formObj)
        .then(function(res) {
          setLoadingBtn(btn, false);
          
          var modalEl = document.getElementById('createCourseModal');
          var modal = bootstrap.Modal.getInstance(modalEl);
          if (modal) modal.hide();
          
          e.target.reset();

          if (res.status === 'success') {
            loadDashboard(); 
            showQrModal(res.courseId);
          } else {
            Swal.fire('Error', res.message, 'error');
          }
        })
        .catch(function(err) {
          setLoadingBtn(btn, false);
          Swal.fire('Error', err.toString(), 'error');
        });
    }

    function showQrModal(courseId) {
      // สร้างลิงก์ QR Code โดยใช้ URL ของหน้าเว็บปัจจุบัน + Parameter
      var link = scriptUrl + '?page=register&classId=' + courseId;
      var qrApi = 'https://quickchart.io/qr?text=' + encodeURIComponent(link) + '&size=300';
      
      document.getElementById('qr-image').src = qrApi;
      document.getElementById('course-link').value = link;
      
      new bootstrap.Modal(document.getElementById('qrModal')).show();
    }
    
    function showQrAgain(courseId) {
       showQrModal(courseId);
    }

    function copyLink() {
      var copyText = document.getElementById("course-link");
      copyText.select();
      
      try {
        document.execCommand("copy"); 
        var Toast = Swal.mixin({
          toast: true, position: 'top-end', showConfirmButton: false, timer: 3000
        });
        Toast.fire({ icon: 'success', title: 'คัดลอกลิงก์แล้ว' });
      } catch (err) {
        Swal.fire('Copy Error', 'Manual copy required', 'error');
      }
    }
    
    function logout() {
      currentUser = null;
      document.getElementById('form-login').reset();
      showView('view-login');
    }

    // --- Scenario B: Student Logic ---

    function loadStudentView(classId) {
      showView('view-register');
      document.getElementById('reg-course-id').value = classId;
      toggleLoading(true);

      Backend.run('getCourseDetails', classId)
        .then(function(res) {
          toggleLoading(false);
          if (res.status === 'success') {
            var d = res.data;
            document.getElementById('reg-topic').innerText = d.topic;
            document.getElementById('reg-date').innerText = d.date;
            document.getElementById('reg-duration').innerText = d.duration;
            document.getElementById('reg-instructor').innerText = d.instructor;
          } else {
            Swal.fire('ไม่พบข้อมูล', 'รหัสคลาสเรียนไม่ถูกต้อง', 'error');
            document.getElementById('form-register').querySelector('button').disabled = true;
          }
        })
        .catch(function(err) {
          toggleLoading(false);
          Swal.fire('Error', 'Connection failed', 'error');
        });
    }

    function handleRegister(e) {
      e.preventDefault();
      var btn = e.target.querySelector('button');
      setLoadingBtn(btn, true);

      var formObj = {
        courseId: document.getElementById('reg-course-id').value,
        studentName: e.target.studentName.value,
        department: e.target.department.value
      };

      Backend.run('registerStudent', formObj)
        .then(function(res) {
          setLoadingBtn(btn, false);
          if (res.status === 'success') {
            Swal.fire({
              title: 'ลงทะเบียนสำเร็จ!',
              text: 'ระบบได้บันทึกข้อมูลของคุณเรียบร้อยแล้ว',
              icon: 'success',
              confirmButtonColor: '#0d47a1'
            }).then(function() {
               e.target.reset(); 
            });
          } else {
            Swal.fire('เกิดข้อผิดพลาด', res.message, 'error');
          }
        })
        .catch(function(err) {
          setLoadingBtn(btn, false);
          Swal.fire('Error', err.toString(), 'error');
        });
    }

    // --- Helpers ---

    function showView(viewId) {
      ['view-login', 'view-dashboard', 'view-register'].forEach(function(id) {
        document.getElementById(id).classList.add('hidden');
      });
      document.getElementById(viewId).classList.remove('hidden');
    }

    function toggleLoading(show) {
      var el = document.getElementById('loading-overlay');
      if(show) {
        el.classList.remove('hidden');
      } else {
        el.style.opacity = '0';
        setTimeout(function() {
          el.classList.add('hidden');
          el.style.opacity = '1';
        }, 300);
      }
    }

    function setLoadingBtn(btn, isLoading) {
      if (isLoading) {
        btn.dataset.originalText = btn.innerHTML;
        btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Loading...';
        btn.disabled = true;
      } else {
        btn.innerHTML = btn.dataset.originalText || 'Submit';
        btn.disabled = false;
      }
    }
  </script>
</body>
</html>
