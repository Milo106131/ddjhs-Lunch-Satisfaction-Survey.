<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>午餐滿意度</title>
    <!-- 引入 Tailwind CSS 進行現代化美化 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入 FontAwesome 圖標 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@500;650;700&family=Noto+Sans+TC:wght@400;500;700&display=swap');
        body {
            font-family: 'Quicksand', 'Noto Sans TC', sans-serif;
            background: linear-gradient(135deg, #fff0f5 0%, #e6f7ff 100%);
        }
        /* 可愛風格的卡片與陰影 */
        .document-paper {
            background-color: #ffffff;
            box-shadow: 0 10px 30px rgba(255, 182, 193, 0.25);
            border: 2px dashed #ffb6c1;
        }
    </style>
</head>
<body class="min-h-screen text-slate-700 p-4 sm:p-8 flex justify-center items-start">

    <div class="w-full max-w-6xl document-paper rounded-3xl overflow-hidden self-start">
        
        <!-- Header 標題區：午餐滿意度 (可愛粉嫩風) -->
        <div class="bg-gradient-to-r from-pink-400 via-purple-400 to-indigo-400 px-6 py-4 text-white flex flex-row justify-between items-center gap-4">
            <div class="text-xl font-bold flex items-center gap-2 drop-shadow-sm">
                <i class="fa-solid fa-utensils text-yellow-200"></i> 午餐滿意度
            </div>
            <div class="flex items-center gap-3 flex-wrap">
                <!-- 鎖定狀態指示與解鎖按鈕 -->
                <div id="authStatusContainer" class="flex items-center gap-2 bg-white/20 px-3 py-1.5 rounded-2xl border border-white/30 text-sm whitespace-nowrap backdrop-blur-sm">
                    <i id="lockIcon" class="fa-solid fa-lock text-yellow-200"></i>
                    <span id="lockStatusText" class="font-bold text-white">已上鎖</span>
                    <button onclick="openUnlockModal()" id="unlockBtn" class="ml-1 bg-white text-pink-600 hover:bg-pink-50 px-3 py-1 rounded-xl text-xs font-bold transition shadow-sm whitespace-nowrap">
                        解鎖操作
                    </button>
                    <button onclick="lockSystem()" id="lockBtn" class="hidden bg-rose-400 hover:bg-rose-500 text-white px-3 py-1 rounded-xl text-xs font-bold transition shadow-sm whitespace-nowrap">
                        重新上鎖
                    </button>
                </div>
                
                <button onclick="addNewRow()" id="addRowBtn" disabled class="bg-white/20 opacity-50 cursor-not-allowed border border-white/30 text-white px-4 py-1.5 rounded-2xl text-sm font-bold transition flex items-center gap-2 backdrop-blur-sm whitespace-nowrap shadow-sm">
                    <i class="fa-solid fa-plus"></i> 新增項目
                </button>
            </div>
        </div>

        <div class="p-6">
            <div class="mb-4 flex flex-col sm:flex-row justify-between items-start sm:items-center bg-pink-50 border border-pink-200 p-3 rounded-2xl text-sm text-pink-700 gap-2">
                <span id="rowCount" class="font-bold whitespace-nowrap ml-auto">總計: 0 項</span>
            </div>

            <!-- 清單列表容器 -->
            <div id="itemList" class="space-y-4">
                <!-- 項目會透過 JavaScript 動態載入 -->
            </div>
        </div>

        <div class="bg-pink-50/50 px-6 py-3 border-t border-pink-100 flex flex-col sm:flex-row justify-between items-center text-xs text-pink-500 gap-2">
            <div>✨ 點擊「按這裡」可直接查看表單連結</div>
            <div id="statusMessage" class="text-pink-600 font-bold"></div>
        </div>
    </div>

    <!-- 密碼解鎖 Modal -->
    <div id="unlockModal" class="fixed inset-0 bg-pink-900/30 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl max-w-md w-full p-6 shadow-2xl border-2 border-pink-200 flex flex-col">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-bold text-pink-600 flex items-center gap-2">
                    <i class="fa-solid fa-key text-yellow-400"></i> 請輸入管理密碼
                </h3>
                <button onclick="closeUnlockModal()" class="text-pink-300 hover:text-pink-500 text-xl font-bold">&times;</button>
            </div>
            <p class="text-xs text-slate-400 mb-4">預設密碼為：<code class="bg-pink-50 px-1.5 py-0.5 rounded text-pink-600 font-bold">1234</code></p>
            
            <div class="space-y-3">
                <input type="password" id="passwordInput" placeholder="請輸入密碼..." 
                    class="w-full bg-pink-50/50 border border-pink-200 text-slate-700 text-sm rounded-2xl px-4 py-3 focus:ring-2 focus:ring-pink-400 focus:outline-none"
                    onkeypress="if(event.key === 'Enter') verifyPassword()">
                <div id="passwordError" class="text-rose-400 text-xs hidden font-medium"></div>
            </div>

            <div class="mt-6 flex justify-end gap-3">
                <button onclick="closeUnlockModal()" class="bg-slate-100 hover:bg-slate-200 text-slate-600 px-4 py-2 rounded-2xl text-sm font-bold transition">
                    取消
                </button>
                <button onclick="verifyPassword()" class="bg-gradient-to-r from-pink-400 to-purple-400 hover:opacity-90 text-white px-5 py-2 rounded-2xl text-sm font-bold transition shadow-md flex items-center gap-2">
                    <i class="fa-solid fa-unlock"></i> 確認解鎖
                </button>
            </div>
        </div>
    </div>

    <script>
        // 預設解鎖密碼
        const CORRECT_PASSWORD = "1234";
        // 系統解鎖狀態變數
        let isUnlocked = false;

        // 從 LocalStorage 載入資料，若無則使用預設示範資料
        let itemsData = JSON.parse(localStorage.getItem('lunch_satisfaction_data')) || [
            { id: 1, semester: "115-1", week: 4, startDate: "2026-09-21", endDate: "2026-09-25", linkUrl: "", note: "" }
        ];

        // 儲存資料到 LocalStorage
        function saveToLocalStorage() {
            localStorage.setItem('lunch_satisfaction_data', JSON.stringify(itemsData));
        }

        // 產生從 115學年度第一學期 到 150學年度第二學期 的完整選項清單
        function generateSemesterOptions(selectedVal) {
            let optionsHTML = '';
            for (let year = 115; year <= 150; year++) {
                for (let term = 1; term <= 2; term++) {
                    const val = `${year}-${term}`;
                    const termName = (term === 1) ? '第一學期' : '第二學期';
                    const text = `${year}學年度${termName}`;
                    const selected = (val === selectedVal) ? 'selected' : '';
                    optionsHTML += `<option value="${val}" ${selected}>${text}</option>`;
                }
            }
            return optionsHTML;
        }

        // 產生從第 1 週 到 第 22 週 的選項清單
        function generateWeekOptions(selectedWeek) {
            let optionsHTML = '';
            for (let w = 1; w <= 22; w++) {
                const selected = (w == selectedWeek) ? 'selected' : '';
                optionsHTML += `<option value="${w}" ${selected}>第${w}週</option>`;
            }
            return optionsHTML;
        }

        // 渲染清單畫面
        function renderList() {
            const listContainer = document.getElementById('itemList');
            document.getElementById('rowCount').textContent = `總計: ${itemsData.length} 項`;
            
            const disabledAttr = isUnlocked ? '' : 'disabled';
            const readonlyAttr = isUnlocked ? '' : 'readonly';
            const bgClass = isUnlocked ? 'bg-white' : 'bg-pink-50/30 opacity-80 cursor-not-allowed';

            if (itemsData.length === 0) {
                listContainer.innerHTML = `
                    <div class="text-center py-12 text-pink-300 bg-pink-50/20 rounded-2xl border border-dashed border-pink-200">
                        <i class="fa-solid fa-face-smile text-3xl mb-2"></i>
                        <p class="font-medium">目前尚無任何項目。</p>
                    </div>
                `;
                return;
            }

            let html = '';
            itemsData.forEach((item, index) => {
                let linkActionHTML = '';
                const trimmedUrl = item.linkUrl ? item.linkUrl.trim() : '';
                
                if (trimmedUrl !== '') {
                    linkActionHTML = `<a href="${trimmedUrl}" target="_blank" class="bg-pink-100 hover:bg-pink-200 text-pink-600 px-3 py-1.5 rounded-xl font-bold transition shadow-sm text-xs flex items-center gap-1 cursor-pointer whitespace-nowrap"><i class="fa-solid fa-link"></i> 按這裡</a>`;
                } else {
                    linkActionHTML = `<span onclick="alertNoUrl()" class="bg-slate-100 hover:bg-slate-200 text-slate-400 px-3 py-1.5 rounded-xl font-medium transition text-xs flex items-center gap-1 cursor-pointer whitespace-nowrap" title="請先解鎖並填入網址">按這裡</span>`;
                }

                // 網址輸入框只有在解鎖時才會顯示
                let urlInputContainer = '';
                if (isUnlocked) {
                    urlInputContainer = `
                        <div class="flex items-center gap-1 bg-white border border-pink-200 rounded-xl px-2 py-1.5 focus-within:ring-2 focus-within:ring-pink-400 shadow-sm">
                            <i class="fa-solid fa-globe text-pink-400 text-xs"></i>
                            <input type="url" value="${item.linkUrl || ''}" oninput="updateItem(${index}, 'linkUrl', this.value)" 
                                class="bg-transparent text-xs text-slate-700 focus:outline-none w-36 sm:w-44"
                                placeholder="貼上網址 (https://...)">
                        </div>
                    `;
                }

                html += `
                    <div class="flex flex-col xl:flex-row items-start xl:items-center justify-between gap-3 bg-pink-50/20 p-4 rounded-2xl border border-pink-100 hover:border-pink-300 hover:shadow-md transition group">
                        
                        <!-- 左側：學期與週次選單 -->
                        <div class="flex flex-wrap items-center gap-2 w-full xl:w-auto">
                            <div class="relative">
                                <select onchange="updateItem(${index}, 'semester', this.value)" ${disabledAttr}
                                    class="${bgClass} border border-pink-200 text-slate-700 text-sm font-bold rounded-xl px-3 py-2 pr-8 focus:ring-2 focus:ring-pink-400 focus:outline-none cursor-pointer appearance-none shadow-sm">
                                    ${generateSemesterOptions(item.semester)}
                                </select>
                                <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-pink-400">
                                    <i class="fa-solid fa-chevron-down text-xs"></i>
                                </div>
                            </div>

                            <span class="text-sm font-bold text-pink-300">-</span>

                            <div class="relative">
                                <select onchange="updateItem(${index}, 'week', this.value)" ${disabledAttr}
                                    class="${bgClass} border border-pink-200 text-slate-700 text-sm font-bold rounded-xl px-3 py-2 pr-8 focus:ring-2 focus:ring-pink-400 focus:outline-none cursor-pointer appearance-none shadow-sm">
                                    ${generateWeekOptions(item.week)}
                                </select>
                                <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-pink-400">
                                    <i class="fa-solid fa-chevron-down text-xs"></i>
                                </div>
                            </div>
                        </div>

                        <!-- 中間：起訖日期選擇器 -->
                        <div class="flex items-center gap-2 w-full xl:w-auto">
                            <span class="text-xs text-pink-400 font-bold whitespace-nowrap">日期：</span>
                            <input type="date" value="${item.startDate}" onchange="updateItem(${index}, 'startDate', this.value)" ${disabledAttr}
                                class="${bgClass} border border-pink-200 text-slate-600 text-sm rounded-xl px-2.5 py-1.5 focus:ring-2 focus:ring-pink-400 focus:outline-none shadow-sm font-medium">
                            <span class="text-pink-300">~</span>
                            <input type="date" value="${item.endDate}" onchange="updateItem(${index}, 'endDate', this.value)" ${disabledAttr}
                                class="${bgClass} border border-pink-200 text-slate-600 text-sm rounded-xl px-2.5 py-1.5 focus:ring-2 focus:ring-pink-400 focus:outline-none shadow-sm font-medium">
                        </div>

                        <!-- 右側：「按這裡」連結、網址輸入框(登入才顯現)、備註、刪除 -->
                        <div class="flex flex-wrap items-center gap-2 w-full xl:w-auto justify-between xl:justify-end">
                            ${linkActionHTML}
                            ${urlInputContainer}

                            <input type="text" value="${item.note}" oninput="updateItem(${index}, 'note', this.value)" ${readonlyAttr}
                                class="${bgClass} border border-pink-200 text-slate-500 text-xs rounded-xl px-3 py-2 w-24 focus:w-32 transition-all focus:outline-none focus:border-pink-400 shadow-sm"
                                placeholder="備註...">
                            
                            <button onclick="deleteItem(${index})" ${disabledAttr} 
                                class="text-pink-300 ${isUnlocked ? 'hover:text-rose-500 hover:bg-rose-50 cursor-pointer' : 'opacity-40 cursor-not-allowed'} p-2 rounded-xl transition" title="刪除此項">
                                <i class="fa-solid fa-trash-can"></i>
                            </button>
                        </div>
                    </div>
                `;
            });
            listContainer.innerHTML = html;
        }

        // 提示尚未填寫網址
        function alertNoUrl() {
            showStatus("✨ 提示：請先解鎖並在右側填入網址後，「按這裡」就會變成可用連結唷！");
        }

        // 開啟密碼解鎖對話框
        function openUnlockModal() {
            document.getElementById('unlockModal').classList.remove('hidden');
            document.getElementById('passwordInput').value = '';
            document.getElementById('passwordError').classList.add('hidden');
            setTimeout(() => document.getElementById('passwordInput').focus(), 100);
        }

        // 關閉密碼解鎖對話框
        function closeUnlockModal() {
            document.getElementById('unlockModal').classList.add('hidden');
        }

        // 驗證密碼
        function verifyPassword() {
            const pwd = document.getElementById('passwordInput').value;
            const errDiv = document.getElementById('passwordError');

            if (pwd === CORRECT_PASSWORD) {
                isUnlocked = true;
                closeUnlockModal();
                updateUIState();
                renderList();
                showStatus("🌸 系統已成功解鎖！");
            } else {
                errDiv.textContent = "密碼錯誤唷（預設密碼為 1234）";
                errDiv.classList.remove('hidden');
            }
        }

        // 重新上鎖
        function lockSystem() {
            isUnlocked = false;
            updateUIState();
            renderList();
            showStatus("🔒 系統已重新上鎖");
        }

        // 更新頂部按鈕與狀態
        function updateUIState() {
            const lockIcon = document.getElementById('lockIcon');
            const lockStatusText = document.getElementById('lockStatusText');
            const unlockBtn = document.getElementById('unlockBtn');
            const lockBtn = document.getElementById('lockBtn');
            const addRowBtn = document.getElementById('addRowBtn');

            if (isUnlocked) {
                lockIcon.className = "fa-solid fa-lock-open text-yellow-200";
                lockStatusText.textContent = "已解鎖";
                unlockBtn.classList.add('hidden');
                lockBtn.classList.remove('hidden');
                
                addRowBtn.removeAttribute('disabled');
                addRowBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                addRowBtn.classList.add('hover:bg-white/30', 'cursor-pointer');
            } else {
                lockIcon.className = "fa-solid fa-lock text-yellow-200";
                lockStatusText.textContent = "已上鎖";
                unlockBtn.classList.remove('hidden');
                lockBtn.classList.add('hidden');
                
                addRowBtn.setAttribute('disabled', 'true');
                addRowBtn.classList.add('opacity-50', 'cursor-not-allowed');
                addRowBtn.classList.remove('hover:bg-white/30', 'cursor-pointer');
            }
        }

        // 更新資料狀態
        function updateItem(index, field, value) {
            if (!isUnlocked) return;
            itemsData[index][field] = value;
            saveToLocalStorage(); // 每次變更自動儲存
            showStatus("💾 已自動儲存");
            if (field === 'linkUrl') {
                renderList();
            }
        }

        // 新增一行
        function addNewRow() {
            if (!isUnlocked) return;
            const newWeekNum = (itemsData.length % 22) + 1;
            itemsData.push({
                id: Date.now(),
                semester: "115-1",
                week: newWeekNum,
                startDate: new Date().toISOString().split('T')[0],
                endDate: new Date().toISOString().split('T')[0],
                linkUrl: "",
                note: ""
            });
            saveToLocalStorage(); // 新增時自動儲存
            renderList();
            showStatus("✨ 已新增項目");
        }

        // 刪除項目
        function deleteItem(index) {
            if (!isUnlocked) return;
            itemsData.splice(index, 1);
            saveToLocalStorage(); // 刪除時自動儲存
            renderList();
            showStatus("🗑️ 已刪除項目");
        }

        // 顯示狀態提示
        function showStatus(text) {
            const msg = document.getElementById('statusMessage');
            if (!msg) return;
            msg.textContent = text;
            setTimeout(() => { msg.textContent = ""; }, 4000);
        }

        // 初始化載入
        window.onload = function() {
            renderList();
        }
    </script>
</body>
</html>
