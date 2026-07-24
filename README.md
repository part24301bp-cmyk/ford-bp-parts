<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเบิกอะไหล่ Ford BP เชิงเนิน 24301</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <!-- Supabase JS Client -->
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
</head>
<body class="bg-slate-100 min-h-screen">

    <!-- Header / Navbar -->
    <nav class="bg-blue-900 text-white shadow-lg">
        <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-3">
                <i class="fa-solid fa-car-side text-blue-400 text-2xl"></i>
                <div>
                    <h1 class="text-xl font-bold leading-none">Ford BP เชิงเนิน 24301</h1>
                    <p class="text-xs text-blue-200 mt-1">ระบบค้นหาและจัดการเบิกอะไหล่</p>
                </div>
            </div>
            
            <!-- User Status Section -->
            <div class="flex items-center space-x-3">
                <div class="text-right">
                    <span class="text-xs text-slate-300 block">ผู้ใช้งานปัจจุบัน:</span>
                    <span id="displayUserName" class="font-semibold text-sm text-yellow-300">สมชาย (ช่างเทคนิค)</span>
                </div>
                <button onclick="changeUser()" class="bg-blue-800 hover:bg-blue-700 text-xs px-2.5 py-1.5 rounded transition border border-blue-600">
                    <i class="fa-solid fa-user-pen"></i> เปลี่ยนผู้ใช้
                </button>
            </div>
        </div>
    </nav>

    <main class="max-w-5xl mx-auto p-4 md:p-6">

        <!-- Search Box -->
        <section class="bg-white rounded-xl shadow-sm p-6 mb-6 text-center border border-slate-200">
            <h2 class="text-xl font-bold text-slate-800 mb-2">ค้นหาอะไหล่ตามป้ายทะเบียน</h2>
            <div class="flex max-w-md mx-auto mt-4">
                <input type="text" id="searchInput" placeholder="กรอกป้ายทะเบียน เช่น กข-1234" 
                       class="w-full px-4 py-2.5 rounded-l-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-blue-500">
                <button onclick="searchParts()" class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2.5 rounded-r-lg font-medium transition">
                    <i class="fa-solid fa-magnifying-glass"></i> ค้นหา
                </button>
            </div>
        </section>

        <!-- Admin Add/Edit Form -->
        <section class="bg-slate-800 text-white rounded-xl shadow-md p-6 mb-6">
            <h3 class="text-lg font-bold mb-4 text-blue-300 flex items-center gap-2">
                <i class="fa-solid fa-folder-plus"></i> เพิ่ม / แก้ไขรายการอะไหล่
            </h3>
            <form id="partForm" onsubmit="handleFormSubmit(event)" class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <input type="hidden" id="editId" value="">
                
                <div>
                    <label class="block text-xs font-medium text-slate-300 mb-1">ป้ายทะเบียนรถ *</label>
                    <input type="text" id="plateInput" required placeholder="เช่น 1กข-8888" 
                           class="w-full p-2 rounded bg-slate-700 border border-slate-600 text-white text-sm focus:outline-none focus:border-blue-400">
                </div>

                <div>
                    <label class="block text-xs font-medium text-slate-300 mb-1">ชื่อรายการอะไหล่ *</label>
                    <input type="text" id="partNameInput" required placeholder="เช่น ไฟหน้าขวา LED" 
                           class="w-full p-2 rounded bg-slate-700 border border-slate-600 text-white text-sm focus:outline-none focus:border-blue-400">
                </div>

                <div>
                    <label class="block text-xs font-medium text-slate-300 mb-1">อัปโหลดรูปภาพ *</label>
                    <input type="file" id="imageFileInput" accept="image/*" 
                           class="w-full text-xs text-slate-300 file:mr-2 file:py-2 file:px-3 file:rounded file:border-0 file:bg-blue-600 file:text-white hover:file:bg-blue-500 cursor-pointer">
                </div>

                <div class="md:col-span-3 flex justify-end gap-2 mt-2">
                    <button type="button" onclick="resetForm()" class="px-4 py-2 text-xs rounded bg-slate-600 hover:bg-slate-500">ยกเลิก</button>
                    <button type="submit" id="submitBtn" class="px-6 py-2 text-xs rounded bg-blue-500 hover:bg-blue-400 font-bold">บันทึกข้อมูล</button>
                </div>
            </form>
        </section>

        <!-- Display Cards Grid -->
        <section>
            <div class="flex justify-between items-center mb-4">
                <h3 id="resultTitle" class="text-lg font-bold text-slate-700">รายการอะไหล่ทั้งหมด</h3>
                <button onclick="loadParts()" class="text-xs text-blue-600 hover:underline">
                    <i class="fa-solid fa-rotate-right"></i> รีเฟรชข้อมูล
                </button>
            </div>
            
            <div id="cardsGrid" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
                <!-- Cards will render here -->
            </div>
        </section>
    </main>

    <script>
        // 1. ตั้งค่า Supabase (นำค่าจากขั้นตอนที่ 1 มาวางแทนที่ตรงนี้)
        const SUPABASE_URL = 'https://chjlpenifmdyywfkarar.supabase.co/rest/v1/';
        const SUPABASE_KEY = 'sb_publishable_0eUZN66ipqoawN-AFUucbg_JLpS1fN4';
        const supabase = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

        // State Management
        let currentUser = localStorage.getItem('ford_user') || 'สมชาย (ผู้ดูแลระบบ)';
        document.getElementById('displayUserName').innerText = currentUser;

        // 2. เปลี่ยนชื่อผู้ใช้งาน
        function changeUser() {
            const newUser = prompt('กรุณากรอกชื่อผู้ใช้งานของคุณ:', currentUser);
            if (newUser && newUser.trim() !== '') {
                currentUser = newUser.trim();
                localStorage.setItem('ford_user', currentUser);
                document.getElementById('displayUserName').innerText = currentUser;
            }
        }

        // 3. โหลดและแสดงรายการอะไหล่
        async function loadParts(searchPlate = '') {
            const grid = document.getElementById('cardsGrid');
            grid.innerHTML = '<p class="text-slate-400 col-span-full text-center py-8">กำลังโหลดข้อมูล...</p>';

            let query = supabase.from('parts').select('*').order('created_at', { ascending: false });
            
            if (searchPlate) {
                query = query.ilike('plate_number', `%${searchPlate}%`);
            }

            const { data, error } = await query;

            if (error) {
                alert('เกิดข้อผิดพลาดในการดึงข้อมูล: ' + error.message);
                return;
            }

            if (data.length === 0) {
                grid.innerHTML = '<p class="text-slate-400 col-span-full text-center py-8">ไม่พบรายการอะไหล่</p>';
                return;
            }

            grid.innerHTML = data.map(item => `
                <div class="bg-white rounded-lg shadow-sm border border-slate-200 overflow-hidden flex flex-col justify-between">
                    <div>
                        <img src="${item.image_url}" alt="${item.part_name}" class="w-full h-44 object-cover bg-slate-100">
                        <div class="p-3">
                            <span class="inline-block bg-blue-100 text-blue-800 text-xs font-bold px-2 py-0.5 rounded mb-1">
                                ${item.plate_number}
                            </span>
                            <h4 class="font-bold text-slate-800 text-sm">${item.part_name}</h4>
                            <p class="text-[11px] text-slate-400 mt-2">
                                <i class="fa-regular fa-user"></i> บันทึกโดย: ${item.created_by || 'ไม่ระบุ'}
                            </p>
                        </div>
                    </div>
                    <div class="p-3 pt-0 flex justify-end gap-3 border-t border-slate-100 mt-2">
                        <button onclick="prepareEdit('${item.id}', '${item.plate_number}', '${item.part_name}')" 
                                class="text-xs text-amber-600 hover:underline">
                            <i class="fa-solid fa-pen"></i> แก้ไข
                        </button>
                        <button onclick="deletePart('${item.id}')" class="text-xs text-red-600 hover:underline">
                            <i class="fa-solid fa-trash"></i> ลบ
                        </button>
                    </div>
                </div>
            `).join('');
        }

        // 4. บันทึก/เพิ่มข้อมูลใหม่ และอัปโหลดรูปภาพ
        async function handleFormSubmit(event) {
            event.preventDefault();
            const editId = document.getElementById('editId').value;
            const plate = document.getElementById('plateInput').value.trim();
            const partName = document.getElementById('partNameInput').value.trim();
            const fileInput = document.getElementById('imageFileInput');
            const submitBtn = document.getElementById('submitBtn');

            submitBtn.disabled = true;
            submitBtn.innerText = 'กำลังบันทึก...';

            try {
                let imageUrl = '';

                // อัปโหลดรูปภาพ (ถ้ามีการเลือกไฟล์)
                if (fileInput.files.length > 0) {
                    const file = fileInput.files[0];
                    const fileExt = file.name.split('.').pop();
                    const fileName = `${Date.now()}.${fileExt}`;

                    const { error: uploadError } = await supabase.storage
                        .from('part-images')
                        .upload(fileName, file);

                    if (uploadError) throw uploadError;

                    // ดึง Public URL ของภาพ
                    const { data: urlData } = supabase.storage
                        .from('part-images')
                        .getPublicUrl(fileName);
                    
                    imageUrl = urlData.publicUrl;
                }

                if (editId) {
                    // Update
                    const updatePayload = { plate_number: plate, part_name: partName };
                    if (imageUrl) updatePayload.image_url = imageUrl;

                    const { error } = await supabase.from('parts').update(updatePayload).eq('id', editId);
                    if (error) throw error;
                } else {
                    // Insert
                    if (!imageUrl) {
                        alert('กรุณาเลือกรูปภาพสำหรับรายการใหม่');
                        return;
                    }
                    const { error } = await supabase.from('parts').insert([{
                        plate_number: plate,
                        part_name: partName,
                        image_url: imageUrl,
                        created_by: currentUser
                    }]);
                    if (error) throw error;
                }

                resetForm();
                loadParts();
                alert('บันทึกข้อมูลเรียบร้อยแล้ว');
            } catch (err) {
                alert('เกิดข้อผิดพลาด: ' + err.message);
            } finally {
                submitBtn.disabled = false;
                submitBtn.innerText = 'บันทึกข้อมูล';
            }
        }

        // 5. ค้นหาตามทะเบียน
        function searchParts() {
            const query = document.getElementById('searchInput').value.trim();
            loadParts(query);
        }

        // 6. เตรียมแก้ไขข้อมูล
        function prepareEdit(id, plate, name) {
            document.getElementById('editId').value = id;
            document.getElementById('plateInput').value = plate;
            document.getElementById('partNameInput').value = name;
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        // 7. รีเซ็ตฟอร์ม
        function resetForm() {
            document.getElementById('editId').value = '';
            document.getElementById('partForm').reset();
        }

        // 8. ลบรายการ
        async function deletePart(id) {
            if (confirm('คุณแน่ใจหรือไม่ว่าต้องการลบรายการนี้?')) {
                const { error } = await supabase.from('parts').delete().eq('id', id);
                if (error) alert('ลบไม่สำเร็จ: ' + error.message);
                else loadParts();
            }
        }

        // โหลดข้อมูลเมื่อเปิดหน้าเว็บครั้งแรก
        loadParts();
    </script>
</body>
</html>
