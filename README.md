<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>우리들의 소중한 기록 - Mate Diary</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;700&family=Nanum+Pen+Script&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Noto Sans KR', sans-serif; transition: background 1.5s ease; min-height: 100vh; }
        .font-handwriting { font-family: 'Nanum Pen Script', cursive; }
        .diary-card { transition: all 0.3s ease; }
        .diary-card:hover { transform: translateY(-3px); box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
        .loading-spinner { border: 3px solid #f3f3f3; border-top: 3px solid #5eead4; border-radius: 50%; width: 20px; height: 20px; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        
        .blurred-content { filter: blur(10px); pointer-events: none; user-select: none; }
        .lock-overlay { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; background: rgba(255, 255, 255, 0.4); backdrop-filter: blur(4px); border-radius: 1.5rem; z-index: 10; }
        
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(4px); }
        .modal.active { display: flex; }

        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
    </style>
</head>
<body id="main-body" class="bg-slate-50 text-slate-800">

    <!-- Connection Screen -->
    <div id="connection-screen" class="fixed inset-0 bg-slate-100 z-50 flex items-center justify-center p-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl max-w-sm w-full text-center border border-slate-200">
            <h2 class="text-3xl font-bold text-teal-600 mb-6 font-handwriting">함께 일기 쓰기</h2>
            <div id="setup-step-1">
                <p class="text-slate-500 mb-8 text-sm leading-relaxed">공유 코드를 통해 소중한 사람과<br>일기장을 실시간으로 연결해 보세요.</p>
                <button id="btn-show-join" class="w-full bg-teal-500 text-white font-bold py-3.5 rounded-2xl mb-3 hover:bg-teal-600 transition shadow-lg shadow-teal-100">코드 입력하고 시작하기</button>
                <button id="btn-create-code" class="w-full bg-white border-2 border-teal-500 text-teal-500 font-bold py-3.5 rounded-2xl hover:bg-teal-50 transition">새로운 연결 코드 만들기</button>
            </div>
            <div id="setup-step-join" class="hidden">
                <p class="text-sm text-slate-500 mb-4 text-center">전달받은 고유 코드를 입력하세요</p>
                <input id="join-code-input" type="text" placeholder="6자리 코드 입력" class="w-full p-4 rounded-2xl bg-slate-50 mb-4 border-2 border-transparent focus:border-teal-300 outline-none text-center font-bold tracking-widest uppercase text-xl">
                <button id="btn-join-couple" class="w-full bg-teal-500 text-white font-bold py-3.5 rounded-2xl mb-3">일기장 입장하기</button>
                <button id="btn-back-to-step1" class="text-slate-400 text-sm underline cursor-pointer">처음으로 돌아가기</button>
            </div>
            <div id="setup-step-code" class="hidden">
                <p class="text-sm text-slate-500 mb-3">상대방에게 이 코드를 알려주세요</p>
                <div id="generated-code-display" class="text-3xl font-bold text-teal-600 tracking-widest mb-6 bg-teal-50 py-5 rounded-2xl border-2 border-dashed border-teal-200">------</div>
                <div class="flex flex-col items-center gap-3">
                    <div class="loading-spinner"></div>
                    <p class="text-xs text-teal-500 animate-pulse">상대방의 연결을 기다리는 중입니다...</p>
                    <button id="btn-cancel-code" class="mt-4 text-slate-400 text-sm underline cursor-pointer">생성 취소</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="settings-modal" class="modal p-4">
        <div class="bg-white w-full max-w-md rounded-3xl p-6 shadow-2xl">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-xl font-bold text-slate-700">공간 설정</h3>
                <button id="btn-close-settings" class="text-slate-400 hover:text-slate-600 text-2xl">&times;</button>
            </div>
            <div class="space-y-6">
                <div>
                    <label class="block text-sm font-bold text-slate-500 mb-2">우리만의 시작일 (D-Day)</label>
                    <input id="input-start-date" type="date" class="w-full p-3 rounded-xl border border-slate-200 outline-none focus:border-teal-400 transition">
                    <p class="text-[10px] text-slate-400 mt-1">* 처음 만난 날 등 기준일을 설정해 주세요.</p>
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-sm font-bold text-slate-500 mb-2">나의 별명</label>
                        <input id="input-my-name" type="text" maxlength="8" placeholder="나의 별명" class="w-full p-3 rounded-xl border border-slate-200 outline-none focus:border-teal-400 transition">
                    </div>
                    <div>
                        <label class="block text-sm font-bold text-slate-500 mb-2">상대방 별명</label>
                        <input id="input-partner-name" type="text" maxlength="8" placeholder="상대방 별명" class="w-full p-3 rounded-xl border border-slate-200 outline-none focus:border-teal-400 transition">
                    </div>
                </div>
                <button id="btn-save-settings" class="w-full bg-teal-500 text-white font-bold py-3 rounded-2xl shadow-lg hover:bg-teal-600 transition">설정 저장하기</button>
                <div class="pt-4 border-t border-slate-100">
                    <button id="btn-logout" class="w-full text-slate-300 text-[10px] underline uppercase tracking-tighter">연결 해제 및 초기화</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Main Header -->
    <header id="main-header" class="bg-white/70 backdrop-blur-md py-8 px-4 text-center border-b border-white/20">
        <div class="max-w-4xl mx-auto relative flex flex-col items-center">
            <button id="btn-open-settings" class="absolute top-0 right-0 p-2 text-slate-400 hover:text-teal-500 transition">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                </svg>
            </button>
            <div class="flex items-center gap-2 mb-1">
                <span class="text-2xl">🧸</span>
                <h1 class="text-2xl font-bold text-slate-700">우리들의 기록 공간</h1>
            </div>
            <div class="flex flex-col items-center mt-3">
                <p id="d-day-display" class="text-4xl font-bold text-teal-500 font-handwriting">연결 중...</p>
                <div class="flex items-center gap-2 text-xs text-slate-400 mt-1 uppercase tracking-widest font-bold">
                    <span id="name-me">나</span> ❤️ <span id="name-partner">너</span>
                </div>
            </div>
            
            <div id="partner-mood-status" class="md:absolute md:top-6 md:left-0 mt-4 md:mt-0 text-sm bg-white/80 border border-white/50 px-4 py-2 rounded-full hidden flex items-center gap-2 shadow-sm">
                <span class="text-slate-500 font-medium text-xs"><span id="partner-label">상대방</span>:</span> 
                <span id="partner-mood-emoji" class="text-lg">?</span>
            </div>
        </div>
    </header>

    <main class="max-w-4xl mx-auto p-6">
        <!-- Diary Creation -->
        <section id="write-section" class="bg-white/80 backdrop-blur-sm rounded-3xl p-6 shadow-sm mb-12 border border-white/50">
            <div class="flex items-center gap-2 mb-6">
                <div class="w-8 h-8 bg-teal-100 rounded-full flex items-center justify-center text-teal-600">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 5.793L3 14.172V17h2.828l8.38-8.379-2.83-2.828z" />
                    </svg>
                </div>
                <h2 class="text-lg font-bold text-slate-700">오늘의 일기 남기기</h2>
            </div>
            
            <div class="space-y-4">
                <input id="diary-title" type="text" placeholder="제목을 입력하세요" class="w-full p-4 rounded-2xl bg-white/50 border border-slate-200 outline-none focus:border-teal-300 transition-colors">
                <textarea id="diary-content" rows="4" placeholder="오늘 하루는 어땠나요?" class="w-full p-4 rounded-2xl bg-white/50 border border-slate-200 outline-none focus:border-teal-300 transition-colors resize-none"></textarea>
                
                <!-- Image Preview Area -->
                <div id="image-preview-container" class="hidden relative w-full max-w-xs h-48 mb-4 rounded-2xl overflow-hidden border-2 border-teal-200 group">
                    <img id="image-preview" src="" class="w-full h-full object-cover">
                    <button id="btn-remove-image" class="absolute top-2 right-2 bg-black/50 text-white rounded-full w-8 h-8 flex items-center justify-center transition hover:bg-black/70">&times;</button>
                </div>
                
                <div class="flex flex-wrap justify-between items-center gap-4 pt-2">
                    <div class="flex items-center gap-2">
                        <!-- Image Upload Trigger -->
                        <label class="cursor-pointer bg-slate-100 hover:bg-slate-200 p-3 rounded-2xl transition border border-slate-200" title="사진 첨부">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-slate-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                            </svg>
                            <input type="file" id="input-image" class="hidden" accept="image/*">
                        </label>
                        <div class="flex items-center gap-3 bg-white/50 px-4 py-2 rounded-2xl border border-slate-200">
                            <span class="text-xs text-slate-500 font-medium">기분</span>
                            <select id="diary-mood" class="bg-transparent text-sm text-slate-700 font-bold outline-none cursor-pointer">
                                <option value="😊">😊 평온</option>
                                <option value="🚀">🚀 의욕</option>
                                <option value="🍕">🍕 맛남</option>
                                <option value="☕">☕ 여유</option>
                                <option value="😴">😴 피곤</option>
                                <option value="✨">✨ 즐거움</option>
                            </select>
                        </div>
                    </div>
                    <button id="btn-save-diary" class="bg-teal-500 hover:bg-teal-600 text-white font-bold py-3 px-10 rounded-2xl shadow-lg transition duration-300 w-full md:w-auto">
                        공유하기
                    </button>
                </div>
            </div>
        </section>

        <!-- Records List -->
        <section>
            <div class="flex items-center justify-between mb-8">
                <h2 class="text-xl font-bold text-slate-700 flex items-center gap-2">
                    <span class="text-teal-500 italic">#</span> 우리의 기록들
                </h2>
                <span id="current-code-tag" class="text-[10px] bg-white/50 text-slate-400 px-2 py-1 rounded font-bold border border-slate-200">코드: ------</span>
            </div>
            
            <div id="unlock-msg" class="hidden mb-6 p-4 bg-teal-50 border border-teal-100 rounded-2xl text-teal-700 text-sm text-center font-medium shadow-sm">
                🎁 오늘 두 사람 모두 일기를 완성했어요! 상대방의 일기를 확인해 보세요.
            </div>

            <div id="diary-list" class="grid grid-cols-1 md:grid-cols-2 gap-6 relative pb-20">
                <div class="col-span-full py-20 text-center text-slate-400 border-2 border-dashed border-white/40 rounded-3xl">
                    첫 일기를 기다리고 있어요.
                </div>
            </div>
        </section>
    </main>

    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-app.js";
        import { getFirestore, doc, setDoc, getDoc, collection, query, onSnapshot, addDoc, serverTimestamp, updateDoc, deleteDoc } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-auth.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'mate-diary-prod-v1';

        let currentUser = null;
        let activeCode = localStorage.getItem('mate_active_code');
        let currentImgBase64 = null;
        
        const moodColors = {
            "😊": "#f0fdf4", "🚀": "#eff6ff", "🍕": "#fff7ed", 
            "☕": "#faf5ff", "😴": "#f8fafc", "✨": "#fefce8"
        };

        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                await signInWithCustomToken(auth, __initial_auth_token);
            } else {
                await signInAnonymously(auth);
            }
        };

        onAuthStateChanged(auth, (user) => {
            if (user) {
                currentUser = user;
                if (activeCode) startApp(activeCode);
            }
        });

        initAuth();

        const getTodayString = () => new Date().toLocaleDateString('ko-KR');
        const generateHexCode = () => Math.random().toString(36).substring(2, 8).toUpperCase();

        // --- Image Compression Logic (Critical for Firestore storage) ---
        const compressImage = (file, maxWidth = 800) => {
            return new Promise((resolve) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = (event) => {
                    const img = new Image();
                    img.src = event.target.result;
                    img.onload = () => {
                        const canvas = document.createElement('canvas');
                        let width = img.width;
                        let height = img.height;

                        if (width > maxWidth) {
                            height = (maxWidth / width) * height;
                            width = maxWidth;
                        }

                        canvas.width = width;
                        canvas.height = height;
                        const ctx = canvas.getContext('2d');
                        ctx.drawImage(img, 0, 0, width, height);
                        
                        // Quality 0.7 to keep size under control
                        resolve(canvas.toDataURL('image/jpeg', 0.7));
                    };
                };
            });
        };

        document.getElementById('input-image').addEventListener('change', async (e) => {
            const file = e.target.files[0];
            if (file) {
                const compressed = await compressImage(file);
                currentImgBase64 = compressed;
                document.getElementById('image-preview').src = compressed;
                document.getElementById('image-preview-container').classList.remove('hidden');
            }
        });

        document.getElementById('btn-remove-image').addEventListener('click', () => {
            currentImgBase64 = null;
            document.getElementById('image-preview-container').classList.add('hidden');
            document.getElementById('input-image').value = '';
        });

        const createShareCode = async () => {
            if (!currentUser) return;
            const newCode = generateHexCode();
            document.getElementById('generated-code-display').innerText = newCode;
            document.getElementById('setup-step-1').classList.add('hidden');
            document.getElementById('setup-step-code').classList.remove('hidden');

            const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', newCode);
            try {
                await setDoc(roomRef, {
                    hostId: currentUser.uid,
                    createdAt: serverTimestamp(),
                    status: 'waiting',
                    startDate: new Date().toISOString().split('T')[0],
                    lastMood: { [currentUser.uid]: '😊' },
                    lastWrittenDate: { [currentUser.uid]: '' },
                    nicknames: { [currentUser.uid]: '나' }
                });

                onSnapshot(roomRef, (snapshot) => {
                    const data = snapshot.data();
                    if (data && data.status === 'connected') {
                        localStorage.setItem('mate_active_code', newCode);
                        startApp(newCode);
                    }
                });
            } catch (err) { alert("연결 생성 실패"); }
        };

        const joinCouple = async () => {
            if (!currentUser) return;
            const code = document.getElementById('join-code-input').value.trim().toUpperCase();
            if (!code) return;

            try {
                const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', code);
                const roomSnap = await getDoc(roomRef);
                if (!roomSnap.exists()) return alert('유효하지 않은 코드입니다.');

                const data = roomSnap.data();
                if (data.status === 'waiting' && data.hostId !== currentUser.uid) {
                    await updateDoc(roomRef, {
                        status: 'connected',
                        guestId: currentUser.uid,
                        [`lastMood.${currentUser.uid}`]: '😊',
                        [`lastWrittenDate.${currentUser.uid}`]: '',
                        [`nicknames.${currentUser.uid}`]: '상대방'
                    });
                }
                localStorage.setItem('mate_active_code', code);
                startApp(code);
            } catch (err) { alert("연결 실패"); }
        };

        function startApp(code) {
            activeCode = code;
            document.getElementById('connection-screen').classList.add('hidden');
            document.getElementById('current-code-tag').innerText = `코드: ${code}`;
            
            const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', code);
            const diaryCol = collection(db, 'artifacts', appId, 'public', 'data', 'diaries_' + code);

            onSnapshot(roomRef, (docSnap) => {
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    
                    // D-Day
                    const start = new Date(data.startDate);
                    const today = new Date();
                    today.setHours(0,0,0,0); start.setHours(0,0,0,0);
                    const diff = Math.floor((today - start) / (1000 * 60 * 60 * 24)) + 1;
                    document.getElementById('d-day-display').innerText = `우리 함께한 지 ${diff}일`;
                    document.getElementById('input-start-date').value = data.startDate;

                    // Nicknames
                    const myNick = data.nicknames[currentUser.uid] || '나';
                    const partnerId = Object.keys(data.nicknames).find(id => id !== currentUser.uid);
                    const partnerNick = partnerId ? (data.nicknames[partnerId] || '상대방') : '상대방';
                    
                    document.getElementById('name-me').innerText = myNick;
                    document.getElementById('name-partner').innerText = partnerNick;
                    document.getElementById('partner-label').innerText = partnerNick;
                    document.getElementById('input-my-name').value = myNick;
                    if (partnerId) document.getElementById('input-partner-name').value = partnerNick;

                    // Theme
                    const moods = data.lastMood || {};
                    const myColor = moodColors[moods[currentUser.uid]] || "#f8fafc";
                    if (partnerId && moods[partnerId]) {
                        const partnerColor = moodColors[moods[partnerId]] || "#f8fafc";
                        document.getElementById('main-body').style.background = `linear-gradient(135deg, ${myColor} 0%, ${partnerColor} 100%)`;
                        document.getElementById('partner-mood-status').classList.remove('hidden');
                        document.getElementById('partner-mood-emoji').innerText = moods[partnerId];
                    } else {
                        document.getElementById('main-body').style.background = myColor;
                    }

                    renderDiaries(diaryCol, data.lastWrittenDate || {}, partnerId);
                }
            });
        }

        function renderDiaries(col, lastWrittenDates, partnerId) {
            const todayStr = getTodayString();
            const isTodayUnlocked = partnerId && (lastWrittenDates[currentUser.uid] === todayStr) && (lastWrittenDates[partnerId] === todayStr);
            
            if (isTodayUnlocked) document.getElementById('unlock-msg').classList.remove('hidden');
            else document.getElementById('unlock-msg').classList.add('hidden');

            onSnapshot(col, (snapshot) => {
                const list = document.getElementById('diary-list');
                const diaries = [];
                snapshot.forEach(doc => diaries.push(doc.data()));
                
                if (diaries.length === 0) return;

                list.innerHTML = '';
                diaries.sort((a, b) => (b.createdAt?.seconds || 0) - (a.createdAt?.seconds || 0));

                diaries.forEach(data => {
                    const isMyDiary = data.author === currentUser.uid;
                    const diaryDate = data.date;
                    const isLocked = !isMyDiary && partnerId && (lastWrittenDates[currentUser.uid] !== diaryDate || lastWrittenDates[partnerId] !== diaryDate);

                    const card = document.createElement('div');
                    card.className = `diary-card relative bg-white/95 backdrop-blur-sm p-6 rounded-3xl shadow-sm border border-white/50 flex flex-col ${isMyDiary ? 'ring-2 ring-teal-500/10' : ''}`;
                    
                    const imageHtml = data.image ? `<img src="${data.image}" class="w-full h-56 object-cover rounded-2xl mb-4">` : '';

                    const contentHtml = `
                        <div class="flex justify-between items-center mb-4">
                            <span class="text-[10px] font-bold text-slate-300 uppercase tracking-tighter">${data.date}</span>
                            <span class="text-xl">${data.mood}</span>
                        </div>
                        ${imageHtml}
                        <h3 class="font-bold text-slate-800 mb-2 truncate">${data.title}</h3>
                        <p class="text-slate-500 text-sm leading-relaxed whitespace-pre-wrap">${data.content}</p>
                        <div class="mt-4 pt-4 border-t border-slate-50 flex items-center justify-between">
                            <span class="text-[10px] text-slate-300 font-bold uppercase">${isMyDiary ? 'MINE' : 'PARTNER'}</span>
                        </div>
                    `;

                    if (isLocked) {
                        card.innerHTML = `<div class="blurred-content">${contentHtml}</div><div class="lock-overlay"><span class="text-2xl mb-1">🔒</span><p class="text-[9px] font-bold text-slate-400">둘 다 쓰면 열려요!</p></div>`;
                    } else {
                        card.innerHTML = contentHtml;
                    }
                    list.appendChild(card);
                });
            });
        }

        const saveDiary = async () => {
            const title = document.getElementById('diary-title').value.trim();
            const content = document.getElementById('diary-content').value.trim();
            if (!title || !content || !activeCode) return;

            const mood = document.getElementById('diary-mood').value;
            const todayStr = getTodayString();

            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'diaries_' + activeCode), {
                    title, content, mood, author: currentUser.uid, date: todayStr, image: currentImgBase64, createdAt: serverTimestamp()
                });
                
                await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'rooms', activeCode), { 
                    [`lastMood.${currentUser.uid}`]: mood, [`lastWrittenDate.${currentUser.uid}`]: todayStr
                });

                document.getElementById('diary-title').value = '';
                document.getElementById('diary-content').value = '';
                currentImgBase64 = null;
                document.getElementById('image-preview-container').classList.add('hidden');
                document.getElementById('input-image').value = '';
            } catch (e) { alert("저장 실패"); }
        };

        const saveSettings = async () => {
            const startDate = document.getElementById('input-start-date').value;
            const myName = document.getElementById('input-my-name').value.trim();
            const partnerName = document.getElementById('input-partner-name').value.trim();
            if (!startDate || !activeCode) return;

            try {
                const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', activeCode);
                const roomSnap = await getDoc(roomRef);
                const data = roomSnap.data();
                const partnerId = Object.keys(data.nicknames || {}).find(id => id !== currentUser.uid);

                const updates = { startDate, [`nicknames.${currentUser.uid}`]: myName || '나' };
                if (partnerId && partnerName) updates[`nicknames.${partnerId}`] = partnerName;

                await updateDoc(roomRef, updates);
                document.getElementById('settings-modal').classList.remove('active');
            } catch (e) { alert('설정 실패'); }
        };

        // UI Handlers
        document.getElementById('btn-show-join').addEventListener('click', () => {
            document.getElementById('setup-step-1').classList.add('hidden');
            document.getElementById('setup-step-join').classList.remove('hidden');
        });
        document.getElementById('btn-create-code').addEventListener('click', createShareCode);
        document.getElementById('btn-join-couple').addEventListener('click', joinCouple);
        document.getElementById('btn-save-diary').addEventListener('click', saveDiary);
        document.getElementById('btn-open-settings').addEventListener('click', () => document.getElementById('settings-modal').classList.add('active'));
        document.getElementById('btn-close-settings').addEventListener('click', () => document.getElementById('settings-modal').classList.remove('active'));
        document.getElementById('btn-save-settings').addEventListener('click', saveSettings);
        document.getElementById('btn-logout').addEventListener('click', () => { if(confirm('연결을 해제할까요?')) { localStorage.removeItem('mate_active_code'); location.reload(); }});
    </script>
</body>
</html>
