<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Финансовая Грамотность: Супер-Квест!</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts: Comic Neue / Montserrat -->
    <link href="https://fonts.googleapis.com/css2?family=Comic+Neue:wght@400;700&family=Montserrat:wght@500;700;800&display=swap" rel="stylesheet">
    
    <!-- Firebase SDK (Модульное подключение) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Глобальные переменные окружения или дефолтная конфигурация
        const firebaseConfig = typeof __firebase_config !== 'undefined' 
            ? JSON.parse(__firebase_config) 
            : {
                // Вы можете вставить свои ключи Firebase сюда, если создадите свой проект
                apiKey: "AIzaSyDemoKeyForFinancialLiteracyQuiz",
                authDomain: "fin-quiz-demo.firebaseapp.com",
                projectId: "fin-quiz-demo",
                storageBucket: "fin-quiz-demo.appspot.com",
                messagingSenderId: "123456789",
                appId: "1:123456789:web:demo"
            };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        // Инициализация авторизации и сохранение в window для вызова из обычных скриптов
        window.db = db;
        window.auth = auth;
        window.appId = typeof __app_id !== 'undefined' ? __app_id : 'fin-quiz-app';

        async function initAuth() {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (err) {
                console.warn("Ошибка авторизации Firebase, переходим в локальный режим:", err);
            }
        }

        window.saveResultToCloud = async function(resultData) {
            if (!auth.currentUser) await initAuth();
            if (!auth.currentUser) return false;

            try {
                // Путь к публичным данным согласно правилам
                const colRef = collection(db, 'artifacts', window.appId, 'public', 'data', 'quiz_results');
                await addDoc(colRef, {
                    ...resultData,
                    createdAt: serverTimestamp(),
                    userId: auth.currentUser.uid
                });
                return true;
            } catch (err) {
                console.error("Ошибка сохранения в Firebase:", err);
                return false;
            }
        };

        window.subscribeToTeacherResults = async function(callback) {
            if (!auth.currentUser) await initAuth();
            if (!auth.currentUser) return;

            try {
                const colRef = collection(db, 'artifacts', window.appId, 'public', 'data', 'quiz_results');
                onSnapshot(colRef, (snapshot) => {
                    const data = [];
                    snapshot.forEach(doc => {
                        data.push({ id: doc.id, ...doc.data() });
                    });
                    // Сортировка по времени (свежие сверху) в памяти JS
                    data.sort((a, b) => {
                        const tA = a.createdAt?.seconds || 0;
                        const t


B = b.createdAt?.seconds || 0;
                        return tB - tA;
                    });
                    callback(data);
                }, (err) => {
                    console.error("Ошибка получения данных:", err);
                });
            } catch (err) {
                console.error("Ошибка подписки:", err);
            }
        };

        initAuth();
    </script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brandYellow: '#FFD166',
                        brandGreen: '#06D6A0',
                        brandBlue: '#118AB2',
                        brandRed: '#EF476F',
                        brandPurple: '#8338EC',
                        darkText: '#2B2D42',
                    },
                    fontFamily: {
                        comic: ['"Comic Neue"', 'cursive', 'sans-serif'],
                        main: ['Montserrat', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Montserrat', sans-serif;
            background: linear-gradient(135deg, #e0f2fe 0%, #fef3c7 50%, #f3e8ff 100%);
            min-height: 100vh;
        }
        .comic-font {
            font-family: 'Comic Neue', cursive, sans-serif;
        }
        .bounce-hover:hover {
            transform: translateY(-4px);
            transition: all 0.2s ease;
        }
        .custom-shadow {
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body class="text-darkText antialiased p-3 sm:p-6 flex flex-col justify-between min-h-screen">

    <!-- Верхняя шапка -->
    <header class="max-w-4xl mx-auto w-full mb-4 text-center flex justify-between items-center px-2">
        <div class="inline-flex items-center gap-3 bg-white px-5 py-2.5 rounded-full shadow-md border-2 border-brandYellow">
            <i class="fa-solid fa-piggy-bank text-2xl sm:text-3xl text-brandRed animate-bounce"></i>
            <h1 class="text-xl sm:text-2xl font-black text-brandBlue comic-font tracking-wide">
                Юный Финансист: 1-4 классы
            </h1>
        </div>
        <!-- Переключатель режима Учитель / Ученик -->
        <button onclick="toggleTeacherModal()" class="bg-white hover:bg-slate-100 text-brandPurple font-bold px-4 py-2 rounded-2xl border-2 border-brandPurple shadow transition text-xs sm:text-sm flex items-center gap-2">
            <i class="fa-solid fa-user-tie"></i>
            <span class="hidden sm:inline">Вход для</span> Учителя
        </button>
    </header>

    <!-- Основная обертка карточки -->
    <main class="max-w-3xl mx-auto w-full bg-white rounded-3xl p-5 sm:p-8 custom-shadow border-4 border-white flex-grow flex flex-col justify-center relative overflow-hidden">
        
        <!-- ================= СТАРТОВЫЙ ЭКРАН ================= -->
        <div id="start-screen" class="space-y-6 text-center py-4">
            <div class="bg-amber-100 border-2 border-amber-300 rounded-2xl p-6 text-left relative">
                <div class="absolute -top-4 -right-4 bg-brandYellow text-darkText font-bold text-xs px-3 py-1 rounded-full shadow">
                    40 Интересных Вопросов 🚀
                </div>
                <h2 class="text-2xl font-bold text-brandPurple mb-3 comic-font flex items-center gap-2">
                    <i class="fa-solid fa-star text-brandYellow"></i> Привет, будущий миллионер!
                </h2>
                <p class="text-gray-700 text-base leading-relaxed mb-2">
                    Пройди увлекательный онлайн-квест по финансовой грамотности. Узнай, как правильно копить, разумно тратить деньги и защищаться от мошенников!
                </p>
                <p class="text-sm font-semibold text-brandBlue">
                    ⏱️ Отвечай внимательно! После теста ты получишь именной диплом знатока!
                </p>


</div>

            <form id="start-form" onsubmit="startQuiz(event)" class="space-y-4 max-w-md mx-auto text-left">
                <div>
                    <label for="student-name" class="block text-sm font-bold text-gray-700 mb-1">
                        <i class="fa-solid fa-user text-brandBlue mr-1"></i> Твое Имя и Фамилия:
                    </label>
                    <input type="text" id="student-name" required placeholder="Например: Вася Иванов" 
                           class="w-full px-4 py-3 rounded-xl border-2 border-gray-300 focus:border-brandBlue focus:outline-none text-lg transition">
                </div>
                <div>
                    <label for="student-class" class="block text-sm font-bold text-gray-700 mb-1">
                        <i class="fa-solid fa-school text-brandGreen mr-1"></i> Твой Класс:
                    </label>
                    <select id="student-class" required class="w-full px-4 py-3 rounded-xl border-2 border-gray-300 focus:border-brandBlue focus:outline-none text-lg bg-white transition">
                        <option value="" disabled selected>Выбери свой класс...</option>
                        <option value="1 класс">1 класс</option>
                        <option value="2 класс">2 класс</option>
                        <option value="3 класс">3 класс</option>
                        <option value="4 класс">4 класс</option>
                    </select>
                </div>
                <button type="submit" class="w-full bg-brandGreen hover:bg-emerald-500 text-white font-extrabold text-xl py-4 rounded-xl shadow-lg hover:shadow-xl transition transform active:scale-95 comic-font flex items-center justify-center gap-2">
                    <span>Начать тест!</span>
                    <i class="fa-solid fa-rocket"></i>
                </button>
            </form>
        </div>

        <!-- ================= ЭКРАН ТЕСТИРОВАНИЯ ================= -->
        <div id="quiz-screen" class="hidden flex-col h-full justify-between space-y-6">
            <!-- Верхняя панель прогресса -->
            <div class="space-y-2">
                <div class="flex justify-between items-center text-sm font-bold text-gray-600">
                    <span id="question-tracker" class="bg-blue-100 text-brandBlue px-3 py-1 rounded-full">
                        Вопрос 1 из 40
                    </span>
                    <span id="category-tag" class="bg-purple-100 text-brandPurple px-3 py-1 rounded-full text-xs sm:text-sm">
                        Категория
                    </span>
                </div>
                <!-- Прогресс-бар -->
                <div class="w-full bg-gray-200 h-4 rounded-full overflow-hidden p-0.5 border border-gray-300">
                    <div id="progress-bar" class="bg-gradient-to-r from-brandGreen to-brandBlue h-full rounded-full transition-all duration-300" style="width: 2.5%"></div>
                </div>
            </div>

            <!-- Текст вопроса -->
            <div class="bg-slate-50 border-2 border-slate-200 rounded-2xl p-5 sm:p-6 min-h-[120px] flex items-center shadow-inner">
                <h3 id="question-text" class="text-xl sm:text-2xl font-bold text-darkText comic-font leading-snug">
                    Загрузка вопроса...
                </h3>
            </div>

            <!-- Варианты ответов -->
            <div id="options-container" class="grid grid-cols-1 gap-3">
                <!-- Кнопки генерируются динамически JS -->
            </div>

            <!-- Нижняя панель навигации -->
            <div class="flex justify-between items-center pt-2">
                <div class="text-xs text-gray-400"> Выбери вариант ответа</div>
                <button id="next-btn" onclick="submitAnswer()" disabled 
                        class="bg-gray-300 text-gray-500 font-bold px-6 py-3 rounded-xl transition flex items-center gap-2 cursor-not-allowed">
                    <span>Ответить</span>


<i class="fa-solid fa-arrow-right"></i>
                </button>
            </div>
        </div>

        <!-- ================= ФИНАЛЬНЫЙ ЭКРАН РЕЗУЛЬТАТОВ ================= -->
        <div id="results-screen" class="hidden space-y-6 py-2">
            <div class="text-center space-y-3">
                <div id="result-badge" class="inline-block px-4 py-1 rounded-full font-bold text-sm bg-yellow-100 text-yellow-800 border border-yellow-300">
                    🎉 Тест завершен!
                </div>
                <h2 id="student-title" class="text-3xl font-black text-brandBlue comic-font">
                    Молодец, Ученик!
                </h2>
                
                <!-- Оценка и Счёт -->
                <div class="bg-gradient-to-br from-amber-50 to-orange-50 border-2 border-brandYellow rounded-2xl p-6 shadow-sm">
                    <div class="text-5xl font-black text-brandRed comic-font mb-1" id="score-display">
                        0 / 40
                    </div>
                    <div class="text-xl font-bold text-brandPurple mb-2" id="rank-display">
                        Звание
                    </div>
                    <p class="text-sm text-gray-600 max-w-md mx-auto" id="rank-desc">
                        Описание результата...
                    </p>
                </div>
            </div>

            <!-- Действия -->
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                <button onclick="copyResultsForTeacher()" class="bg-brandBlue hover:bg-blue-600 text-white font-bold py-3 px-4 rounded-xl shadow transition flex items-center justify-center gap-2 text-sm">
                    <i class="fa-solid fa-copy"></i>
                    <span>Скопировать отчёт</span>
                </button>
                <button onclick="window.print()" class="bg-brandPurple hover:bg-purple-700 text-white font-bold py-3 px-4 rounded-xl shadow transition flex items-center justify-center gap-2 text-sm">
                    <i class="fa-solid fa-print"></i>
                    <span>Распечатать / Сохранить PDF</span>
                </button>
            </div>

            <div id="save-status" class="text-center text-xs font-bold p-2 rounded-lg transition">
                <!-- Статус отправки в базу -->
            </div>

            <div id="alert-msg" class="hidden text-center text-xs font-bold text-brandGreen bg-emerald-50 p-2 rounded-lg border border-brandGreen">
                Результаты скопированы в буфер обмена! Перешлите их учителю.
            </div>

            <!-- Подробный разбор ошибок -->
            <div class="border-t-2 border-gray-100 pt-4">
        <!-- ================= ФИНАЛЬНЫЙ ЭКРАН РЕЗУЛЬТАТОВ ================= -->

        <!-- ================= ОНЛАЙН-ПАНЕЛЬ УЧИТЕЛЯ ================= -->
        <div id="teacher-screen" class="hidden space-y-6 py-2">
            <div class="flex justify-between items-center border-b pb-3 border-gray-200">
                <div>
                    <h2 class="text-2xl font-black text-brandPurple comic-font flex items-center gap-2">
                        <i class="fa-solid fa-chart-line text-brandGreen"></i> Журнал ответов (Онлайн)
                    </h2>
                    <p class="text-xs text-gray-500">Результаты учеников обновляются автоматически в режиме реального времени</p>
                </div>
                <button onclick="closeTeacherPanel()" class="bg-gray-100 hover:bg-gray-200 text-gray-700 font-bold px-3 py-1.5 rounded-xl text-xs transition">
                    ❌ Выйти
                </button>
            </div>

            <!-- Фильтр по классам -->
            <div class="flex items-center gap-3 bg-slate-50 p-3 rounded-xl border border-slate-200">
                <label for="filter-class" class="text-xs font-bold text-gray-600">Фильтр по классу:</label>
                <select id="filter-class" onchange="renderTeacherList()" class="px-3 py-1.5 rounded-lg border border-gray-300


text-xs bg-white focus:outline-none">
                    <option value="ALL">Все классы</option>
                    <option value="1 класс">1 класс</option>
                    <option value="2 класс">2 класс</option>
                    <option value="3 класс">3 класс</option>
                    <option value="4 класс">4 класс</option>
                </select>
                <div class="ml-auto text-xs font-bold text-brandBlue" id="total-count-badge">
                    Всего сдали: 0
                </div>
            </div>

            <!-- Список пройденных тестов -->
            <div id="teacher-results-list" class="space-y-3 max-h-[450px] overflow-y-auto pr-1">
                <div class="text-center py-10 text-gray-400 text-sm">
                    <i class="fa-solid fa-spinner animate-spin text-2xl mb-2 text-brandBlue"></i>
                    <p>Загрузка результатов из базы данных...</p>
                </div>
            </div>
        </div>

    </main>

    <!-- Модальное окно ввода PIN-кода учителя -->
    <div id="pin-modal" class="hidden fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl p-6 max-w-sm w-full space-y-4 shadow-2xl border-4 border-brandPurple text-center">
            <div class="w-12 h-12 bg-purple-100 text-brandPurple rounded-full flex items-center justify-center mx-auto text-xl">
                <i class="fa-solid fa-lock"></i>
            </div>
            <h3 class="text-xl font-bold text-darkText comic-font">Вход для учителя</h3>
            <p class="text-xs text-gray-500">Введите ПИН-код для доступа к онлайн-журналу ответов учеников</p>
            <input type="password" id="teacher-pin" maxlength="6" placeholder="ПИН-код (по умолч.: 1234)" 
                   class="w-full text-center text-2xl font-bold tracking-widest px-4 py-3 rounded-xl border-2 border-gray-300 focus:border-brandPurple focus:outline-none">
            <div class="flex gap-2">
                <button onclick="closePinModal()" class="w-1/2 bg-gray-100 hover:bg-gray-200 text-gray-700 font-bold py-3 rounded-xl text-sm transition">
                    Отмена
                </button>
                <button onclick="verifyPin()" class="w-1/2 bg-brandPurple hover:bg-purple-700 text-white font-bold py-3 rounded-xl text-sm transition">
                    Войти
                </button>
            </div>
        </div>
    </div>

    <footer class="max-w-4xl mx-auto w-full text-center mt-4 text-xs text-gray-500 font-medium">
        Онлайн-тест по финансовой грамотности для 1-4 классов • Подготовлено для школы
    </footer>

    <!-- JS Логика и 40 Полных Вопросов -->
    <script>
        // База из 40 полных вопросов по финансовой грамотности для младших школьников
        const questions = [
            // Категория: Деньги и история
            {
                cat: "Деньги и история",
                q: "Как назывался обмен одних товаров на другие до появления денег?",
                options: ["Бартер", "Покупка", "Кредит", "Скидка"],
                correct: 0,
                explain: "Бартер — это прямой обмен товарами (например, обменять топор на зерно) без участия денег."
            },
            {
                cat: "Деньги и история",
                q: "Почему бумажные деньги удобнее древних металлических монет?",
                options: ["Они тяжелее", "Они занимают меньше места и легкие", "Они никогда не рвутся", "Их можно нарисовать самому"],
                correct: 1,
                explain: "Бумажные купюры очень легкие и компактные по сравнению с кучей тяжелых монет."
            },
            {
                cat: "Деньги и история",
                q: "Как называется национальная валюта России?",
                options: ["Доллар", "Евро", "Рубль", "Юань"],
                correct: 2,
                explain: "Официальной денежной единицей (валютой) Российской Федерации является рубль."
            },



