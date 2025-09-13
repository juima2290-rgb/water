<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>생태천 OX 스피드 퀴즈 (완성판)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Jua&family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            overflow: hidden; /* 화면 전환 시 스크롤바 방지 */
        }
        .font-jua {
            font-family: 'Jua', sans-serif;
        }
        /* 기본 카드 스타일 (글래스모피즘) */
        .card {
            background-color: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(10px);
            padding: 2rem;
            border-radius: 1.5rem; /* 더 둥글게 */
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            border: 1px solid rgba(255, 255, 255, 0.2);
            transition: all 0.5s cubic-bezier(0.25, 0.8, 0.25, 1); /* 부드러운 전환 효과 */
            width: 100%;
            max-width: 42rem;
            position: absolute; /* 애니메이션을 위해 absolute로 변경 */
            opacity: 0;
            transform: translateY(20px);
            pointer-events: none; /* 보이지 않을 때 클릭 방지 */
        }
        .card.active {
            opacity: 1;
            transform: translateY(0);
            pointer-events: auto;
        }
        .btn-choice {
            transition: all 0.2s ease-in-out;
        }
        .btn-choice:hover {
            transform: scale(1.05) translateY(-5px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
        }
        .btn-choice:active {
            transform: scale(0.98);
        }

        /* 랭킹 테이블 스타일 */
        .rank-table th, .rank-table td {
            padding: 1rem 0.75rem;
            text-align: center;
        }
        .rank-table tbody tr {
            transition: background-color 0.2s;
        }
        .rank-table tbody tr:hover {
            background-color: rgba(0,0,0,0.05);
        }
        
        /* 모달 스타일 */
        .modal {
            position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex; align-items: center; justify-content: center;
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.3s ease;
            pointer-events: none;
        }
        .modal.active {
            opacity: 1;
            pointer-events: auto;
        }
        .modal-content {
            transform: scale(0.95);
            transition: transform 0.3s ease;
        }
        .modal.active .modal-content {
             transform: scale(1);
        }

        /* 애니메이션 효과 */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
            20%, 40%, 60%, 80% { transform: translateX(5px); }
        }
        .animate-shake {
            animation: shake 0.5s ease-in-out;
        }

        @keyframes pulse {
            50% { opacity: .7; }
        }
        .animate-pulse-fast {
            animation: pulse 1s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }

        /* 정답/오답 피드백 오버레이 */
        .feedback-overlay {
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            border-radius: 1.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10rem;
            color: white;
            opacity: 0;
            transition: opacity 0.3s;
            pointer-events: none;
        }
        .feedback-overlay.correct { background-color: rgba(16, 185, 129, 0.7); }
        .feedback-overlay.incorrect { background-color: rgba(239, 68, 68, 0.7); }
        .feedback-overlay.show { opacity: 1; }

        /* 생물 애니메이션 */
        .creature-container {
            position: absolute;
            top: 0; left: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            z-index: 0; /* 카드 콘텐츠 뒤로 보내기 */
            pointer-events: none; /* 클릭 방지 */
        }
        .creature {
            position: absolute;
            left: -150px; /* 화면 밖에서 시작 */
            will-change: transform; /* 애니메이션 성능 최적화 */
        }

        /* 오리 스타일 및 애니메이션 */
        .duck {
            width: 100px;
            bottom: 25%;
            animation: float 25s linear infinite;
            animation-delay: 2s;
        }
        @keyframes float {
            0% { transform: translateX(0) translateY(0) rotate(-2deg); }
            50% { transform: translateX(calc(100vw + 100px)) translateY(-10px) rotate(2deg); }
            100% { transform: translateX(0) translateY(0) rotate(-2deg); }
        }

        /* 물고기 공통 스타일 */
        .fish {
            width: 80px;
        }
        .fish-1 {
            bottom: 10%;
            transform: scaleX(-1); /* 좌우 반전 */
            animation: swim-1 18s ease-in-out infinite;
        }
        .fish-2 {
            bottom: 18%;
            width: 60px;
            animation: swim-2 22s ease-in-out infinite;
            animation-delay: 5s;
        }
        .fish-3 {
            bottom: 5%;
            width: 90px;
            animation: swim-2 15s ease-in-out infinite;
            animation-delay: 10s;
        }

        /* 물고기 애니메이션 */
        @keyframes swim-1 {
            0% { transform: translateX(calc(100vw + 150px)) translateY(0) scaleX(-1); }
            45% { transform: translateX(50vw) translateY(20px) scaleX(-1); }
            50% { transform: translateX(45vw) translateY(20px) scaleX(1); } /* 방향 전환 */
            95% { transform: translateX(0) translateY(0) scaleX(1); }
            100% { transform: translateX(calc(-150px)) translateY(0) scaleX(1); }
        }
        @keyframes swim-2 {
            0% { transform: translateX(-150px) translateY(0); }
            100% { transform: translateX(calc(100vw + 150px)) translateY(-20px); }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-emerald-100 via-cyan-100 to-sky-200 flex items-center justify-center min-h-screen p-4">

    <div id="app-container" class="relative container mx-auto flex justify-center items-center h-full" style="height: 600px;">
        
        <div id="start-screen" class="card active text-center">
            
            <div class="creature-container">
                <svg viewBox="0 0 128 128" class="creature duck">
                    <path fill="#FFD95A" d="M107.2,62.7c0.2,1.3-0.9,4.4-1.1,4.6c-1,1.1-6,2.2-6.2,2.2c-0.2,0-3.2-1.2-5.1-2.4c-4-2.5-5.6-5.8-5.6-5.8s0.3,4.6-2.9,9.2c-2.3,3.3-6.1,6-11,7.2c-8.5,2.1-16.1-0.2-22.3-5.2c-5.7-4.6-9.5-11.2-10.4-18.4c-0.9-7.2,1-15.1,5.3-20.9c4.3-5.8,10.7-9,17.7-8.5c7,0.5,13.4,4.2,17.7,10c0.9,1.2,1.6,2.4,2.3,3.7c0,0-2.3-1.4-3.7-1.2c-1.4,0.2-2.1,1.1-2.1,1.1s1.8-0.9,2.9,0c1.1,0.9,0.7,2.1,0.7,2.1s2-2.3,3.7-1.4c1.8,0.9,1.4,3.1,1.4,3.1s2.5-2.5,4.2-1.2c1.7,1.3,1.1,3.3,1.1,3.3l4.2,3.3C104.9,59.2,107,61.4,107.2,62.7z"/>
                    <path fill="#F9A629" d="M94.9,69.5c-0.2,0-3.2-1.2-5.1-2.4c-4-2.5-5.6-5.8-5.6-5.8s1.6,1.4,4.6,3.1c3,1.8,5.8,2.7,6,2.7C95.1,66.9,94.9,69.5,94.9,69.5z"/>
                    <circle fill="#2F333A" cx="99.5" cy="59.9" r="2.5"/>
                </svg>
                <svg viewBox="0 0 128 128" class="creature fish fish-1">
                    <path fill="#84D2F6" d="M114.3,64.1c0,14.6-15.3,26.4-34.1,26.4S46,78.7,46,64.1s15.3-26.4,34.1-26.4S114.3,49.5,114.3,64.1z"/>
                    <path fill="#59A5D8" d="M103.5,64.1c0,9.2-9.6,16.7-21.4,16.7s-21.4-7.5-21.4-16.7s9.6-16.7,21.4-16.7S103.5,54.9,103.5,64.1z"/>
                    <path fill="#84D2F6" d="M57.4,80.1c-6.2,0-11.2-5-11.2-11.2c0-5.2,3.5-9.6,8.4-10.8c-1.3-1.6-2.1-3.6-2.1-5.8c0-5.1,4.1-9.2,9.2-9.2s9.2,4.1,9.2,9.2c0,0.5-0.1,1-0.1,1.5c-2.4-1-5.1-1.5-8,1.2C59.1,68.4,57.4,74,57.4,80.1z"/>
                    <circle fill="#2F333A" cx="95.2" cy="60.2" r="4"/>
                </svg>
                <svg viewBox="0 0 128 128" class="creature fish fish-2">
                     <path fill="#F3B4B4" d="M114.3,64.1c0,14.6-15.3,26.4-34.1,26.4S46,78.7,46,64.1s15.3-26.4,34.1-26.4S114.3,49.5,114.3,64.1z"/>
                    <path fill="#E86262" d="M103.5,64.1c0,9.2-9.6,16.7-21.4,16.7s-21.4-7.5-21.4-16.7s9.6-16.7,21.4-16.7S103.5,54.9,103.5,64.1z"/>
                    <path fill="#F3B4B4" d="M57.4,80.1c-6.2,0-11.2-5-11.2-11.2c0-5.2,3.5-9.6,8.4-10.8c-1.3-1.6-2.1-3.6-2.1-5.8c0-5.1,4.1-9.2,9.2-9.2s9.2,4.1,9.2,9.2c0,0.5-0.1,1-0.1,1.5c-2.4-1-5.1-1.5-8,1.2C59.1,68.4,57.4,74,57.4,80.1z"/>
                    <circle fill="#2F333A" cx="95.2" cy="60.2" r="4"/>
                </svg>
                <svg viewBox="0 0 128 128" class="creature fish fish-3">
                    <path fill="#F7D35F" d="M114.3,64.1c0,14.6-15.3,26.4-34.1,26.4S46,78.7,46,64.1s15.3-26.4,34.1-26.4S114.3,49.5,114.3,64.1z"/>
                    <path fill="#F1B522" d="M103.5,64.1c0,9.2-9.6,16.7-21.4,16.7s-21.4-7.5-21.4-16.7s9.6-16.7,21.4-16.7S103.5,54.9,103.5,64.1z"/>
                    <path fill="#F7D35F" d="M57.4,80.1c-6.2,0-11.2-5-11.2-11.2c0-5.2,3.5-9.6,8.4-10.8c-1.3-1.6-2.1-3.6-2.1-5.8c0-5.1,4.1-9.2,9.2-9.2s9.2,4.1,9.2,9.2c0,0.5-0.1,1-0.1,1.5c-2.4-1-5.1-1.5-8,1.2C59.1,68.4,57.4,74,57.4,80.1z"/>
                    <circle fill="#2F333A" cx="95.2" cy="60.2" r="4"/>
                </svg>
            </div>

            <h1 class="text-5xl font-jua text-emerald-700 mb-4 flex items-center justify-center gap-3">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M17.707 9.293a1 1 0 010 1.414l-7 7a1 1 0 01-1.414 0l-7-7A.997.997 0 012 10V5a1 1 0 011-1h14a1 1 0 011 1v4.293zM5 6a1 1 0 100 2 1 1 0 000-2zm4 0a1 1 0 100 2 1 1 0 000-2zm4 0a1 1 0 100 2 1 1 0 000-2z" clip-rule="evenodd" /></svg>
                생태천 OX 스피드 퀴즈
            </h1>
            <p class="text-gray-600 mb-8 text-lg">당신의 생태 지식과 순발력을 마음껏 뽐내보세요!</p>
            <div class="space-y-4">
                <input type="text" id="userId" placeholder="아이디 (예: 홍길동)" class="w-full p-3 border-2 border-gray-200 rounded-lg focus:outline-none focus:ring-4 focus:ring-emerald-300 transition-all duration-300">
                <input type="tel" id="userPhone" placeholder="전화번호 (예: 010-1234-5678)" class="w-full p-3 border-2 border-gray-200 rounded-lg focus:outline-none focus:ring-4 focus:ring-emerald-300 transition-all duration-300">
                <button id="start-btn" class="w-full bg-emerald-600 text-white font-bold py-4 px-6 rounded-lg hover:bg-emerald-700 transition-all duration-300 text-xl shadow-lg hover:shadow-xl transform hover:-translate-y-1">퀴즈 시작!</button>
            </div>
             <p id="start-error" class="text-red-500 mt-4 text-sm h-5"></p>
             <div class="mt-6 text-right">
                <button id="admin-mode-btn" class="text-sm text-gray-500 hover:text-gray-700 underline">관리자 모드</button>
             </div>
        </div>

        <div id="quiz-screen" class="card">
            <div class="feedback-overlay" id="feedback-overlay"></div>
            <div class="flex justify-between items-center mb-6">
                <div id="question-counter" class="text-xl font-bold text-gray-700 bg-white/50 px-4 py-2 rounded-full"></div>
                <div id="timer" class="text-3xl font-jua text-emerald-600">0.00초</div>
            </div>
            <div class="bg-white/50 p-6 rounded-lg min-h-[200px] flex items-center justify-center mb-6 shadow-inner">
                <p id="question-text" class="text-3xl font-bold text-center text-gray-800 leading-normal"></p>
            </div>
            <div class="grid grid-cols-2 gap-6">
                <button data-answer="true" class="btn-choice text-8xl font-jua bg-blue-500 text-white rounded-2xl py-12 hover:bg-blue-600 shadow-lg">O</button>
                <button data-answer="false" class="btn-choice text-8xl font-jua bg-red-500 text-white rounded-2xl py-12 hover:bg-red-600 shadow-lg">X</button>
            </div>
        </div>

        <div id="result-screen" class="card text-center">
            <h2 class="text-4xl font-jua text-emerald-700 mb-6">퀴즈 완료! 수고하셨습니다!</h2>
            <div class="flex justify-center gap-6 mb-8">
                <div class="bg-white/50 p-6 rounded-lg flex-1">
                    <p class="text-lg text-gray-600 mb-2">정답 개수</p> 
                    <span id="score" class="font-bold text-5xl text-blue-600 font-jua"></span>
                </div>
                <div class="bg-white/50 p-6 rounded-lg flex-1">
                    <p class="text-lg text-gray-600 mb-2">소요 시간</p> 
                    <span id="time-taken" class="font-bold text-5xl text-red-600 font-jua"></span>
                </div>
            </div>

            <h3 class="text-2xl font-jua text-gray-800 mb-4 flex items-center justify-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-amber-500" viewBox="0 0 20 20" fill="currentColor"><path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" /></svg>
                명예의 전당 (TOP 10)
            </h3>
            <div class="overflow-x-auto max-h-60">
                <table id="rank-table" class="rank-table w-full text-base text-left text-gray-700">
                    <thead class="text-sm text-gray-800 uppercase bg-gray-200/70 sticky top-0">
                        <tr><th>순위</th><th>아이디</th><th>정답</th><th>기록(초)</th></tr>
                    </thead>
                    <tbody></tbody>
                </table>
            </div>
            <button id="restart-btn" class="mt-8 w-full bg-emerald-600 text-white font-bold py-4 px-6 rounded-lg hover:bg-emerald-700 transition-all duration-300 text-xl shadow-lg hover:shadow-xl transform hover:-translate-y-1">처음으로 돌아가기</button>
        </div>

        <div id="admin-screen" class="card">
            <h2 class="text-3xl font-jua text-blue-700 mb-6">관리자 페이지 - 전체 기록</h2>
            <div class="overflow-y-auto max-h-[450px]">
                <table id="admin-rank-table" class="rank-table w-full text-sm text-left text-gray-600">
                    <thead class="text-xs text-gray-700 uppercase bg-blue-100/70 sticky top-0">
                        <tr><th>순위</th><th>아이디</th><th>전화번호</th><th>정답</th><th>기록(초)</th></tr>
                    </thead>
                    <tbody id="admin-rank-body"></tbody>
                </table>
            </div>
            <button id="admin-back-btn" class="mt-8 w-full bg-gray-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-gray-700 text-lg">처음으로</button>
        </div>
    </div>

    <div id="admin-login-modal" class="modal">
        <div class="modal-content bg-white p-8 rounded-lg shadow-xl w-full max-w-sm text-center">
            <h3 class="text-2xl font-jua mb-4">관리자 로그인</h3>
            <input type="password" id="admin-password" placeholder="비밀번호" class="w-full p-3 border-2 border-gray-300 rounded-lg mb-2 focus:outline-none focus:ring-4 focus:ring-blue-300">
            <p id="admin-error" class="text-red-500 text-sm h-5 mb-4"></p>
            <div class="flex gap-4">
                <button id="admin-login-btn" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg hover:bg-blue-700">로그인</button>
                <button id="admin-close-btn" class="w-full bg-gray-300 text-gray-800 font-bold py-2 rounded-lg hover:bg-gray-400">닫기</button>
            </div>
        </div>
    </div>


    <script>
        const quizData = [
            { question: "생태하천은 오염된 물을 스스로 정화하는 능력이 있다.", answer: true },
            { question: "모든 하천의 물고기는 1급수에서만 살 수 있다.", answer: false },
            { question: "수질 정화를 위해 하천 바닥을 모두 콘크리트로 덮는 것이 좋다.", answer: false },
            { question: "버드나무와 갈대는 수질 정화에 도움을 주는 대표적인 수생식물이다.", answer: true },
            { question: "생태하천 복원 사업은 동식물의 서식지를 파괴하는 주된 원인이다.", answer: false },
            { question: "빗물이 하천으로 바로 유입되면 수질 오염의 원인이 될 수 있다.", answer: true },
            { question: "하천에 사는 잠자리의 애벌레는 물 밖 풀숲에서 생활한다.", answer: false },
            { question: "생활하수를 정화 없이 하천으로 흘려보내는 것은 생태계를 풍요롭게 한다.", answer: false },
            { question: "생태하천의 돌과 자갈은 미생물이 살 공간을 제공하여 물을 깨끗하게 한다.", answer: true },
            { question: "생태하천에서는 어떠한 경우에도 낚시나 물놀이가 금지되어 있다.", answer: false }
        ];

        // --- DOM 요소 ---
        const screens = {
            start: document.getElementById('start-screen'),
            quiz: document.getElementById('quiz-screen'),
            result: document.getElementById('result-screen'),
            admin: document.getElementById('admin-screen'),
        };
        const adminLoginModal = document.getElementById('admin-login-modal');
        
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const choiceBtns = document.querySelectorAll('.btn-choice');
        const adminModeBtn = document.getElementById('admin-mode-btn');
        const adminLoginBtn = document.getElementById('admin-login-btn');
        const adminCloseBtn = document.getElementById('admin-close-btn');
        const adminBackBtn = document.getElementById('admin-back-btn');
        
        const userIdInput = document.getElementById('userId');
        const userPhoneInput = document.getElementById('userPhone');
        const startError = document.getElementById('start-error');
        const questionCounter = document.getElementById('question-counter');
        const timerDisplay = document.getElementById('timer');
        const questionText = document.getElementById('question-text');
        const scoreDisplay = document.getElementById('score');
        const timeTakenDisplay = document.getElementById('time-taken');
        const rankTableBody = document.querySelector('#rank-table tbody');
        const adminPasswordInput = document.getElementById('admin-password');
        const adminError = document.getElementById('admin-error');
        const adminRankBody = document.getElementById('admin-rank-body');
        const feedbackOverlay = document.getElementById('feedback-overlay');
        
        let currentQuestionIndex, score, timerInterval, startTime;
        let isAnswering = false; // 중복 답변 방지 플래그

        // --- 화면 전환 함수 ---
        function showScreen(screenName) {
            Object.values(screens).forEach(screen => screen.classList.remove('active'));
            screens[screenName].classList.add('active');
        }
        
        // --- 게임 로직 ---
        function startGame() {
            const userId = userIdInput.value.trim();
            const userPhone = userPhoneInput.value.trim();
            const phoneRegex = /^\d{2,3}-\d{3,4}-\d{4}$/;

            if (!userId || !userPhone) {
                startError.textContent = '아이디와 전화번호를 모두 입력해주세요.'; return;
            }
            if (!phoneRegex.test(userPhone)) {
                startError.textContent = '올바른 전화번호 형식(010-1234-5678)을 입력해주세요.'; return;
            }
            
            startError.textContent = '';
            currentQuestionIndex = 0;
            score = 0;
            isAnswering = false;

            showScreen('quiz');
            displayQuestion();
            startTimer();
        }

        function displayQuestion() {
            if (currentQuestionIndex < quizData.length) {
                const currentQuestion = quizData[currentQuestionIndex];
                questionText.textContent = currentQuestion.question;
                questionCounter.textContent = `${currentQuestionIndex + 1} / ${quizData.length}`;
                questionText.parentElement.classList.remove('animate-shake'); // 이전 애니메이션 제거
            } else {
                endGame();
            }
        }

        function selectAnswer(e) {
            if (isAnswering) return; // 답변 처리 중이면 리턴
            isAnswering = true;

            const selectedAnswer = e.target.dataset.answer === 'true';
            const correctAnswer = quizData[currentQuestionIndex].answer;

            if (selectedAnswer === correctAnswer) {
                score++;
                showFeedback(true);
            } else {
                showFeedback(false);
            }
            
            // 0.5초 후 다음 문제로
            setTimeout(() => {
                currentQuestionIndex++;
                displayQuestion();
                isAnswering = false; // 플래그 초기화
            }, 500);
        }

        function showFeedback(isCorrect) {
            feedbackOverlay.innerHTML = isCorrect ? 'O' : 'X';
            feedbackOverlay.className = 'feedback-overlay show';
            feedbackOverlay.classList.add(isCorrect ? 'correct' : 'incorrect');

            if (!isCorrect) {
                questionText.parentElement.classList.add('animate-shake');
            }

            setTimeout(() => {
                feedbackOverlay.classList.remove('show');
            }, 500);
        }

        function startTimer() {
            startTime = Date.now();
            timerDisplay.textContent = '0.00초';
            timerInterval = setInterval(() => {
                const elapsedTime = (Date.now() - startTime) / 1000;
                timerDisplay.textContent = `${elapsedTime.toFixed(2)}초`;
            }, 10);
        }

        function endGame() {
            clearInterval(timerInterval);
            const totalTime = ((Date.now() - startTime) / 1000);
            
            showScreen('result');

            // 점수와 시간 애니메이션
            animateValue(scoreDisplay, 0, score, 500, ` / ${quizData.length}`);
            animateValue(timeTakenDisplay, 0, totalTime, 500, '초', true);

            saveRecord(totalTime);
            displayRankings();
            
            // 축하 효과
            confetti({ particleCount: 150, spread: 90, origin: { y: 0.6 } });
        }

        function animateValue(obj, start, end, duration, suffix = '', isFloat = false) {
            let startTimestamp = null;
            const step = (timestamp) => {
                if (!startTimestamp) startTimestamp = timestamp;
                const progress = Math.min((timestamp - startTimestamp) / duration, 1);
                let currentValue = progress * (end - start) + start;
                obj.innerHTML = (isFloat ? currentValue.toFixed(2) : Math.floor(currentValue)) + suffix;
                if (progress < 1) {
                    window.requestAnimationFrame(step);
                }
            };
            window.requestAnimationFrame(step);
        }
        
        function saveRecord(time) {
            const rankings = JSON.parse(localStorage.getItem('ecoQuizRankings')) || [];
            const newRecord = {
                id: userIdInput.value.trim(),
                phone: userPhoneInput.value.trim(),
                score: score,
                time: parseFloat(time.toFixed(2))
            };
            
            rankings.push(newRecord);
            rankings.sort((a, b) => b.score - a.score || a.time - b.time);
            localStorage.setItem('ecoQuizRankings', JSON.stringify(rankings));
        }

        function displayRankings() {
            const rankings = JSON.parse(localStorage.getItem('ecoQuizRankings')) || [];
            rankTableBody.innerHTML = '';
            if (rankings.length === 0) {
                 rankTableBody.innerHTML = '<tr><td colspan="4">아직 등록된 기록이 없습니다.</td></tr>'; return;
            }
            rankings.slice(0, 10).forEach((rank, i) => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="font-bold text-lg">${i + 1}</td>
                    <td>${rank.id}</td>
                    <td class="font-bold text-blue-600">${rank.score}</td>
                    <td class="text-red-600">${rank.time.toFixed(2)}</td>
                `;
                rankTableBody.appendChild(tr);
            });
        }
        
        function restartGame() {
            showScreen('start');
            userIdInput.value = '';
            userPhoneInput.value = '';
        }

        // --- 관리자 기능 ---
        function showAdminLogin() {
            adminPasswordInput.value = '';
            adminError.textContent = '';
            adminLoginModal.classList.add('active');
        }

        function handleAdminLogin() {
            if (adminPasswordInput.value === '1212') {
                adminLoginModal.classList.remove('active');
                showScreen('admin');
                displayAdminRankings();
            } else {
                adminPasswordInput.parentElement.classList.add('animate-shake');
                adminError.textContent = '비밀번호가 올바르지 않습니다.';
                setTimeout(() => adminPasswordInput.parentElement.classList.remove('animate-shake'), 500);
            }
        }

        function displayAdminRankings() {
            const rankings = JSON.parse(localStorage.getItem('ecoQuizRankings')) || [];
            adminRankBody.innerHTML = '';
            if (rankings.length === 0) {
                 adminRankBody.innerHTML = '<tr><td colspan="5">등록된 기록이 없습니다.</td></tr>'; return;
            }
            rankings.forEach((rank, index) => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="font-bold">${index + 1}</td>
                    <td>${rank.id}</td>
                    <td>${rank.phone}</td>
                    <td class="font-bold text-blue-600">${rank.score}</td>
                    <td class="text-red-600">${rank.time.toFixed(2)}</td>
                `;
                adminRankBody.appendChild(tr);
            });
        }

        // --- 이벤트 리스너 ---
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', restartGame);
        choiceBtns.forEach(button => button.addEventListener('click', selectAnswer));
        adminModeBtn.addEventListener('click', showAdminLogin);
        adminCloseBtn.addEventListener('click', () => adminLoginModal.classList.remove('active'));
        adminLoginBtn.addEventListener('click', handleAdminLogin);
        adminBackBtn.addEventListener('click', restartGame);
        adminPasswordInput.addEventListener('keyup', (e) => e.key === 'Enter' && handleAdminLogin());
    </script>
</body>
</html>

















