<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>سیستم انبارداری بارساایونت</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            color: #333;
        }
        .modal {
            display: none; /* Hidden by default */
            position: fixed; /* Stay in place */
            z-index: 1000; /* Sit on top */
            left: 0;
            top: 0;
            width: 100%; /* Full width */
            height: 100%; /* Full height */
            overflow: auto; /* Enable scroll if needed */
            background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            width: 90%;
            max-width: 500px;
            text-align: center;
        }
        .modal-buttons button {
            @apply px-4 py-2 rounded-lg font-semibold text-white transition-colors duration-200;
        }
        .modal-buttons .ok-button {
            @apply bg-blue-600 hover:bg-blue-700;
        }
        .modal-buttons .cancel-button {
            @apply bg-gray-400 hover:bg-gray-500 mr-2;
        }
        .tab-button.active {
            @apply bg-blue-600 text-white;
        }
        .loading-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            flex-direction: column;
            gap: 1rem;
        }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            border-left-color: #2563eb;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="p-4">
    <div id="loading-overlay" class="loading-overlay">
        <div class="spinner"></div>
        <p>در حال بارگذاری برنامه...</p>
    </div>

    <div class="max-w-4xl mx-auto bg-white p-6 rounded-xl shadow-lg">
        <h1 class="text-3xl font-bold text-center text-blue-700 mb-6">سیستم انبارداری بارساایونت</h1>
        <p class="text-center text-gray-600 mb-8">به مدیریت هوشمندانه ملزومات رویدادهای خود خوش آمدید.</p>

        <div class="flex flex-wrap justify-center gap-4 mb-8">
            <button id="dashboardTab" class="tab-button px-6 py-3 rounded-lg font-semibold text-gray-700 hover:bg-blue-100 transition-colors duration-200 active">داشبورد</button>
            <button id="inventoryTab" class="tab-button px-6 py-3 rounded-lg font-semibold text-gray-700 hover:bg-blue-100 transition-colors duration-200">اقلام موجودی</button>
            <button id="eventsTab" class="tab-button px-6 py-3 rounded-lg font-semibold text-gray-700 hover:bg-blue-100 transition-colors duration-200">رویدادها</button>
            <button id="transactionsTab" class="tab-button px-6 py-3 rounded-lg font-semibold text-gray-700 hover:bg-blue-100 transition-colors duration-200">تراکنش‌ها</button>
        </div>

        <div id="dashboardContent" class="tab-content">
            <h2 class="text-2xl font-semibold text-blue-600 mb-4 text-center">داشبورد</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                <div class="bg-blue-50 p-4 rounded-lg shadow-md">
                    <h3 class="font-semibold text-lg text-blue-800">تعداد کل اقلام</h3>
                    <p id="totalItemsCount" class="text-3xl font-bold text-blue-600 mt-2">0</p>
                </div>
                <div class="bg-green-50 p-4 rounded-lg shadow-md">
                    <h3 class="font-semibold text-lg text-green-800">اقلام در دسترس</h3>
                    <p id="availableItemsCount" class="text-3xl font-bold text-green-600 mt-2">0</p>
                </div>
                <div class="bg-yellow-50 p-4 rounded-lg shadow-md">
                    <h3 class="font-semibold text-lg text-yellow-800">اقلام در رویداد</h3>
                    <p id="inEventItemsCount" class="text-3xl font-bold text-yellow-600 mt-2">0</p>
                </div>
                <div class="bg-red-50 p-4 rounded-lg shadow-md">
                    <h3 class="font-semibold text-lg text-red-800">اقلام آسیب دیده</h3>
                    <p id="damagedItemsCount" class="text-3xl font-bold text-red-600 mt-2">0</p>
                </div>
                <div class="bg-purple-50 p-4 rounded-lg shadow-md">
                    <h3 class="font-semibold text-lg text-purple-800">رویدادهای فعال</h3>
                    <p id="activeEventsCount" class="text-3xl font-bold text-purple-600 mt-2">0</p>
                </div>
            </div>

            <h3 class="text-xl font-semibold text-blue-600 mt-8 mb-4 text-center">اقلام با موجودی کم</h3>
            <div id="lowStockItemsList" class="overflow-x-auto">
                </div>
        </div>

        <div id="inventoryContent" class="tab-content hidden">
            <h2 class="text-2xl font-semibold text-blue-600 mb-4 text-center">مدیریت اقلام موجودی</h2>
            <button id="showAddItemFormBtn" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg font-semibold mb-4 w-full md:w-auto">افزودن کالای جدید</button>

            <form id="addItemForm" class="hidden bg-gray-50 p-4 rounded-lg shadow-inner mb-6">
                <h3 class="text-xl font-semibold text-gray-700 mb-4">افزودن کالای جدید</h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="item-id" class="block text-sm font-medium text-gray-700">شناسه کالا (ID):</label>
                        <input type="text" id="item-id" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                    </div>
                    <div>
                        <label for="item-name" class="block text-sm font-medium text-gray-700">نام کالا:</label>
                        <input type="text" id="item-name" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                    </div>
                    <div>
                        <label for="item-category" class="block text-sm font-medium text-gray-700">دسته بندی:</label>
                        <input type="text" id="item-category" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="item-description" class="block text-sm font-medium text-gray-700">توضیحات/مدل:</label>
                        <input type="text" id="item-description" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="item-location" class="block text-sm font-medium text-gray-700">محل نگهداری:</label>
                        <input type="text" id="item-location" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="item-min-stock" class="block text-sm font-medium text-gray-700">حد حداقل موجودی:</label>
                        <input type="number" id="item-min-stock" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" value="0">
                    </div>
                </div>
                <div class="flex justify-end gap-2 mt-6">
                    <button type="submit" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-semibold">ذخیره کالا</button>
                    <button type="button" id="cancelAddItemFormBtn" class="bg-gray-400 hover:bg-gray-500 text-white px-4 py-2 rounded-lg font-semibold">انصراف</button>
                </div>
            </form>

            <div id="inventoryList" class="overflow-x-auto">
                </div>
        </div>

        <div id="eventsContent" class="tab-content hidden">
            <h2 class="text-2xl font-semibold text-blue-600 mb-4 text-center">مدیریت رویدادها</h2>
            <button id="showAddEventFormBtn" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg font-semibold mb-4 w-full md:w-auto">افزودن رویداد جدید</button>

            <form id="addEventForm" class="hidden bg-gray-50 p-4 rounded-lg shadow-inner mb-6">
                <h3 class="text-xl font-semibold text-gray-700 mb-4">افزودن رویداد جدید</h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="event-id" class="block text-sm font-medium text-gray-700">شناسه رویداد:</label>
                        <input type="text" id="event-id" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                    </div>
                    <div>
                        <label for="event-name" class="block text-sm font-medium text-gray-700">نام رویداد:</label>
                        <input type="text" id="event-name" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                    </div>
                    <div>
                        <label for="event-start-date" class="block text-sm font-medium text-gray-700">تاریخ شروع:</label>
                        <input type="date" id="event-start-date" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="event-end-date" class="block text-sm font-medium text-gray-700">تاریخ پایان:</label>
                        <input type="date" id="event-end-date" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="event-location" class="block text-sm font-medium text-gray-700">مکان برگزاری:</label>
                        <input type="text" id="event-location" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="event-responsible" class="block text-sm font-medium text-gray-700">مسئول رویداد:</label>
                        <input type="text" id="event-responsible" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="event-status" class="block text-sm font-medium text-gray-700">وضعیت رویداد:</label>
                        <select id="event-status" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                            <option value="در حال برنامه‌ریزی">در حال برنامه‌ریزی</option>
                            <option value="در حال برگزاری">در حال برگزاری</option>
                            <option value="به اتمام رسیده">به اتمام رسیده</option>
                            <option value="لغو شده">لغو شده</option>
                        </select>
                    </div>
                </div>
                <div class="flex justify-end gap-2 mt-6">
                    <button type="submit" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-semibold">ذخیره رویداد</button>
                    <button type="button" id="cancelAddEventFormBtn" class="bg-gray-400 hover:bg-gray-500 text-white px-4 py-2 rounded-lg font-semibold">انصراف</button>
                </div>
            </form>

            <div id="eventsList" class="overflow-x-auto">
                </div>
        </div>

        <div id="transactionsContent" class="tab-content hidden">
            <h2 class="text-2xl font-semibold text-blue-600 mb-4 text-center">مدیریت تراکنش‌ها</h2>

            <div class="flex flex-wrap justify-center gap-4 mb-6">
                <button id="showWithdrawalFormBtn" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg font-semibold w-full md:w-auto">ثبت خروج کالا (رفت)</button>
                <button id="showReturnFormBtn" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-lg font-semibold w-full md:w-auto">ثبت ورود کالا (برگشت/جدید)</button>
            </div>

            <form id="withdrawalForm" class="hidden bg-gray-50 p-4 rounded-lg shadow-inner mb-6">
                <h3 class="text-xl font-semibold text-gray-700 mb-4">ثبت خروج کالا (رفت به رویداد)</h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="withdrawal-item-id" class="block text-sm font-medium text-gray-700">شناسه کالا (ID):</label>
                        <input type="text" id="withdrawal-item-id" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                    </div>
                    <div>
                        <label for="withdrawal-quantity" class="block text-sm font-medium text-gray-700">تعداد:</label>
                        <input type="number" id="withdrawal-quantity" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required min="1">
                    </div>
                    <div class="md:col-span-2">
                        <label for="withdrawal-event-id" class="block text-sm font-medium text-gray-700">شناسه رویداد مرتبط:</label>
                        <select id="withdrawal-event-id" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                            </select>
                    </div>
                    <div class="md:col-span-2">
                        <label for="withdrawal-notes" class="block text-sm font-medium text-gray-700">توضیحات/ملاحظات:</label>
                        <textarea id="withdrawal-notes" rows="2" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500"></textarea>
                    </div>
                </div>
                <div class="flex justify-end gap-2 mt-6">
                    <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg font-semibold">ثبت خروج</button>
                    <button type="button" id="cancelWithdrawalFormBtn" class="bg-gray-400 hover:bg-gray-500 text-white px-4 py-2 rounded-lg font-semibold">انصراف</button>
                </div>
            </form>

            <form id="returnForm" class="hidden bg-gray-50 p-4 rounded-lg shadow-inner mb-6">
                <h3 class="text-xl font-semibold text-gray-700 mb-4">ثبت ورود کالا (برگشت از رویداد / خرید جدید)</h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="return-item-id" class="block text-sm font-medium text-gray-700">شناسه کالا (ID):</label>
                        <input type="text" id="return-item-id" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                    </div>
                    <div>
                        <label for="return-quantity" class="block text-sm font-medium text-gray-700">تعداد:</label>
                        <input type="number" id="return-quantity" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required min="1">
                    </div>
                    <div>
                        <label for="return-event-id" class="block text-sm font-medium text-gray-700">شناسه رویداد مرتبط (اختیاری - برای برگشت):</label>
                        <select id="return-event-id" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                            <option value="">-- خرید جدید / ورود مستقیم --</option>
                            </select>
                    </div>
                    <div>
                        <label for="return-status" class="block text-sm font-medium text-gray-700">وضعیت کالا پس از ورود:</label>
                        <select id="return-status" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" required>
                            <option value="سالم">سالم</option>
                            <option value="آسیب دیده">آسیب دیده</option>
                        </select>
                    </div>
                    <div class="md:col-span-2">
                        <label for="return-notes" class="block text-sm font-medium text-gray-700">توضیحات/ملاحظات:</label>
                        <textarea id="return-notes" rows="2" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500"></textarea>
                    </div>
                </div>
                <div class="flex justify-end gap-2 mt-6">
                    <button type="submit" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-semibold">ثبت ورود</button>
                    <button type="button" id="cancelReturnFormBtn" class="bg-gray-400 hover:bg-gray-500 text-white px-4 py-2 rounded-lg font-semibold">انصراف</button>
                </div>
            </form>

            <div id="transactionsList" class="overflow-x-auto">
                </div>
        </div>
    </div>

    <div id="customModal" class="modal">
        <div class="modal-content">
            <h3 id="modalTitle" class="text-xl font-semibold mb-4"></h3>
            <p id="modalMessage" class="mb-6"></p>
            <div class="modal-buttons flex justify-center">
                <button id="modalOkBtn" class="ok-button">تایید</button>
                <button id="modalCancelBtn" class="cancel-button hidden">لغو</button>
            </div>
        </div>
    </div>

    <script type="module">
        // Firebase App (the core Firebase SDK) is always required and must be listed first
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, getDocs, doc, getDoc, updateDoc, deleteDoc, onSnapshot, query, where } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase Configuration (provided by the environment)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        let userId = null;
        let inventoryItems = [];
        let events = [];
        let transactions = [];

        // --- Utility Functions ---

        /**
         * Shows a custom modal for alerts or confirmations.
         * @param {string} title - The title of the modal.
         * @param {string} message - The message to display.
         * @param {boolean} showCancel - Whether to show a cancel button.
         * @returns {Promise<boolean>} - Resolves true if OK is clicked, false if Cancel.
         */
        function showModal(title, message, showCancel = false) {
            return new Promise((resolve) => {
                const modal = document.getElementById('customModal');
                document.getElementById('modalTitle').innerText = title;
                document.getElementById('modalMessage').innerText = message;
                const okBtn = document.getElementById('modalOkBtn');
                const cancelBtn = document.getElementById('modalCancelBtn');

                okBtn.onclick = () => {
                    modal.style.display = 'none';
                    resolve(true);
                };

                if (showCancel) {
                    cancelBtn.classList.remove('hidden');
                    cancelBtn.onclick = () => {
                        modal.style.display = 'none';
                        resolve(false);
                    };
                } else {
                    cancelBtn.classList.add('hidden');
                }

                modal.style.display = 'flex';
            });
        }

        /**
         * Shows the loading overlay.
         */
        function showLoading() {
            document.getElementById('loading-overlay').style.display = 'flex';
        }

        /**
         * Hides the loading overlay.
         */
        function hideLoading() {
            document.getElementById('loading-overlay').style.display = 'none';
        }

        // --- Firebase Authentication ---
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                userId = user.uid;
                console.log("User signed in:", userId);
                // Start listening to Firestore data once authenticated
                await setupFirestoreListeners();
                hideLoading(); // Hide loading after initial data load
            } else {
                // Sign in anonymously if no user is authenticated
                try {
                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch (error) {
                    console.error("Error signing in:", error);
                    showModal('خطا', 'خطا در ورود به برنامه. لطفاً دوباره تلاش کنید.');
                    hideLoading();
                }
            }
        });

        // --- Firestore Data Listeners ---
        async function setupFirestoreListeners() {
            if (!userId) {
                console.warn("User ID not available for Firestore listeners.");
                return;
            }

            // Inventory Items Listener
            const inventoryCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/inventory`);
            onSnapshot(inventoryCollectionRef, (snapshot) => {
                inventoryItems = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderInventoryList();
                updateDashboard();
                populateEventSelects(); // Update event selects when inventory changes (for low stock checks)
            }, (error) => {
                console.error("Error fetching inventory:", error);
                showModal('خطا', 'خطا در بارگذاری اقلام موجودی.');
            });

            // Events Listener
            const eventsCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/events`);
            onSnapshot(eventsCollectionRef, (snapshot) => {
                events = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderEventsList();
                updateDashboard();
                populateEventSelects();
            }, (error) => {
                console.error("Error fetching events:", error);
                showModal('خطا', 'خطا در بارگذاری رویدادها.');
            });

            // Transactions Listener
            const transactionsCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/transactions`);
            onSnapshot(transactionsCollectionRef, (snapshot) => {
                transactions = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderTransactionsList();
            }, (error) => {
                console.error("Error fetching transactions:", error);
                showModal('خطا', 'خطا در بارگذاری تراکنش‌ها.');
            });
        }

        // --- UI Rendering Functions ---

        function updateDashboard() {
            let totalItems = 0;
            let availableItems = 0;
            let inEventItems = 0;
            let damagedItems = 0;
            let lowStockItems = [];

            inventoryItems.forEach(item => {
                totalItems += item.totalStock || 0;
                availableItems += item.availableStock || 0;
                inEventItems += item.inEventStock || 0;
                damagedItems += item.damagedStock || 0;

                if (item.availableStock < item.minStock && item.minStock > 0) {
                    lowStockItems.push(item);
                }
            });

            const activeEvents = events.filter(event => event.status === 'در حال برگزاری').length;

            document.getElementById('totalItemsCount').innerText = totalItems;
            document.getElementById('availableItemsCount').innerText = availableItems;
            document.getElementById('inEventItemsCount').innerText = inEventItems;
            document.getElementById('damagedItemsCount').innerText = damagedItems;
            document.getElementById('activeEventsCount').innerText = activeEvents;

            renderLowStockItems(lowStockItems);
        }

        function renderLowStockItems(items) {
            const container = document.getElementById('lowStockItemsList');
            if (items.length === 0) {
                container.innerHTML = '<p class="text-center text-gray-500">هیچ کالایی با موجودی کم وجود ندارد.</p>';
                return;
            }

            let html = `
                <table class="min-w-full bg-white rounded-lg shadow-md">
                    <thead>
                        <tr class="bg-gray-100 text-gray-600 uppercase text-sm leading-normal">
                            <th class="py-3 px-6 text-right">شناسه کالا</th>
                            <th class="py-3 px-6 text-right">نام کالا</th>
                            <th class="py-3 px-6 text-right">موجودی فعلی</th>
                            <th class="py-3 px-6 text-right">حد حداقل</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 text-sm font-light">
            `;
            items.forEach(item => {
                html += `
                    <tr class="border-b border-gray-200 hover:bg-gray-50">
                        <td class="py-3 px-6 text-right whitespace-nowrap">${item.id}</td>
                        <td class="py-3 px-6 text-right">${item.name}</td>
                        <td class="py-3 px-6 text-right">${item.availableStock}</td>
                        <td class="py-3 px-6 text-right">${item.minStock}</td>
                    </tr>
                `;
            });
            html += `
                    </tbody>
                </table>
            `;
            container.innerHTML = html;
        }


        function renderInventoryList() {
            const container = document.getElementById('inventoryList');
            if (inventoryItems.length === 0) {
                container.innerHTML = '<p class="text-center text-gray-500">هیچ کالایی ثبت نشده است. لطفاً کالای جدید اضافه کنید.</p>';
                return;
            }

            let html = `
                <table class="min-w-full bg-white rounded-lg shadow-md">
                    <thead>
                        <tr class="bg-gray-100 text-gray-600 uppercase text-sm leading-normal">
                            <th class="py-3 px-6 text-right">شناسه کالا</th>
                            <th class="py-3 px-6 text-right">نام کالا</th>
                            <th class="py-3 px-6 text-right">دسته بندی</th>
                            <th class="py-3 px-6 text-right">موجودی کل</th>
                            <th class="py-3 px-6 text-right">در دسترس</th>
                            <th class="py-3 px-6 text-right">در رویداد</th>
                            <th class="py-3 px-6 text-right">آسیب دیده</th>
                            <th class="py-3 px-6 text-right">عملیات</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 text-sm font-light">
            `;
            inventoryItems.forEach(item => {
                html += `
                    <tr class="border-b border-gray-200 hover:bg-gray-50">
                        <td class="py-3 px-6 text-right whitespace-nowrap">${item.id}</td>
                        <td class="py-3 px-6 text-right">${item.name}</td>
                        <td class="py-3 px-6 text-right">${item.category || '-'}</td>
                        <td class="py-3 px-6 text-right">${item.totalStock || 0}</td>
                        <td class="py-3 px-6 text-right">${item.availableStock || 0}</td>
                        <td class="py-3 px-6 text-right">${item.inEventStock || 0}</td>
                        <td class="py-3 px-6 text-right">${item.damagedStock || 0}</td>
                        <td class="py-3 px-6 text-right">
                            <button data-id="${item.id}" class="edit-item-btn text-blue-500 hover:text-blue-700 font-semibold ml-2">ویرایش</button>
                            <button data-id="${item.id}" class="delete-item-btn text-red-500 hover:text-red-700 font-semibold">حذف</button>
                        </td>
                    </tr>
                `;
            });
            html += `
                    </tbody>
                </table>
            `;
            container.innerHTML = html;

            document.querySelectorAll('.edit-item-btn').forEach(button => {
                button.onclick = (e) => editItem(e.target.dataset.id);
            });
            document.querySelectorAll('.delete-item-btn').forEach(button => {
                button.onclick = (e) => deleteItem(e.target.dataset.id);
            });
        }

        function renderEventsList() {
            const container = document.getElementById('eventsList');
            if (events.length === 0) {
                container.innerHTML = '<p class="text-center text-gray-500">هیچ رویدادی ثبت نشده است. لطفاً رویداد جدید اضافه کنید.</p>';
                return;
            }

            let html = `
                <table class="min-w-full bg-white rounded-lg shadow-md">
                    <thead>
                        <tr class="bg-gray-100 text-gray-600 uppercase text-sm leading-normal">
                            <th class="py-3 px-6 text-right">شناسه رویداد</th>
                            <th class="py-3 px-6 text-right">نام رویداد</th>
                            <th class="py-3 px-6 text-right">تاریخ شروع</th>
                            <th class="py-3 px-6 text-right">وضعیت</th>
                            <th class="py-3 px-6 text-right">عملیات</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 text-sm font-light">
            `;
            events.forEach(event => {
                html += `
                    <tr class="border-b border-gray-200 hover:bg-gray-50">
                        <td class="py-3 px-6 text-right whitespace-nowrap">${event.id}</td>
                        <td class="py-3 px-6 text-right">${event.name}</td>
                        <td class="py-3 px-6 text-right">${event.startDate || '-'}</td>
                        <td class="py-3 px-6 text-right">${event.status}</td>
                        <td class="py-3 px-6 text-right">
                            <button data-id="${event.id}" class="edit-event-btn text-blue-500 hover:text-blue-700 font-semibold ml-2">ویرایش</button>
                            <button data-id="${event.id}" class="delete-event-btn text-red-500 hover:text-red-700 font-semibold">حذف</button>
                        </td>
                    </tr>
                `;
            });
            html += `
                    </tbody>
                </table>
            `;
            container.innerHTML = html;

            document.querySelectorAll('.edit-event-btn').forEach(button => {
                button.onclick = (e) => editEvent(e.target.dataset.id);
            });
            document.querySelectorAll('.delete-event-btn').forEach(button => {
                button.onclick = (e) => deleteEvent(e.target.dataset.id);
            });
        }

        function renderTransactionsList() {
            const container = document.getElementById('transactionsList');
            if (transactions.length === 0) {
                container.innerHTML = '<p class="text-center text-gray-500">هیچ تراکنشی ثبت نشده است.</p>';
                return;
            }

            let html = `
                <table class="min-w-full bg-white rounded-lg shadow-md">
                    <thead>
                        <tr class="bg-gray-100 text-gray-600 uppercase text-sm leading-normal">
                            <th class="py-3 px-6 text-right">تاریخ</th>
                            <th class="py-3 px-6 text-right">نوع تراکنش</th>
                            <th class="py-3 px-6 text-right">کالا</th>
                            <th class="py-3 px-6 text-right">تعداد</th>
                            <th class="py-3 px-6 text-right">رویداد</th>
                            <th class="py-3 px-6 text-right">وضعیت</th>
                            <th class="py-3 px-6 text-right">ثبت کننده</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 text-sm font-light">
            `;
            transactions.sort((a, b) => b.timestamp.toDate() - a.timestamp.toDate()).forEach(txn => { // Sort by latest
                const date = txn.timestamp ? new Date(txn.timestamp.toDate()).toLocaleDateString('fa-IR') : '-';
                const time = txn.timestamp ? new Date(txn.timestamp.toDate()).toLocaleTimeString('fa-IR') : '-';
                html += `
                    <tr class="border-b border-gray-200 hover:bg-gray-50">
                        <td class="py-3 px-6 text-right whitespace-nowrap">${date} ${time}</td>
                        <td class="py-3 px-6 text-right">${txn.type}</td>
                        <td class="py-3 px-6 text-right">${txn.itemId}</td>
                        <td class="py-3 px-6 text-right">${txn.quantity}</td>
                        <td class="py-3 px-6 text-right">${txn.eventId || '-'}</td>
                        <td class="py-3 px-6 text-right">${txn.itemStatus || '-'}</td>
                        <td class="py-3 px-6 text-right">${txn.recordedBy || '-'}</td>
                    </tr>
                `;
            });
            html += `
                    </tbody>
                </table>
            `;
            container.innerHTML = html;
        }

        function populateEventSelects() {
            const withdrawalSelect = document.getElementById('withdrawal-event-id');
            const returnSelect = document.getElementById('return-event-id');

            // Clear existing options
            withdrawalSelect.innerHTML = '';
            returnSelect.innerHTML = '<option value="">-- خرید جدید / ورود مستقیم --</option>';

            // Add options from fetched events
            events.forEach(event => {
                const option = document.createElement('option');
                option.value = event.id;
                option.innerText = `${event.name} (${event.id})`;
                withdrawalSelect.appendChild(option.cloneNode(true));
                returnSelect.appendChild(option.cloneNode(true));
            });
        }

        // --- Data Management Functions (Firestore) ---

        // Add/Update Item
        async function addOrUpdateItem(itemData) {
            showLoading();
            try {
                const itemRef = doc(db, `artifacts/${appId}/users/${userId}/inventory`, itemData.id);
                const docSnap = await getDoc(itemRef);

                if (docSnap.exists()) {
                    // Update existing item
                    await updateDoc(itemRef, {
                        name: itemData.name,
                        category: itemData.category,
                        description: itemData.description,
                        location: itemData.location,
                        minStock: itemData.minStock,
                    });
                    showModal('موفقیت', 'کالا با موفقیت ویرایش شد.');
                } else {
                    // Add new item
                    await setDoc(itemRef, {
                        name: itemData.name,
                        category: itemData.category,
                        description: itemData.description,
                        totalStock: 0,
                        availableStock: 0,
                        inEventStock: 0,
                        damagedStock: 0,
                        location: itemData.location,
                        minStock: itemData.minStock,
                        createdAt: new Date(),
                    });
                    showModal('موفقیت', 'کالا با موفقیت اضافه شد.');
                }
                document.getElementById('addItemForm').reset();
                document.getElementById('addItemForm').classList.add('hidden');
            } catch (e) {
                console.error("Error adding/updating document: ", e);
                showModal('خطا', 'خطا در ذخیره کالا: ' + e.message);
            } finally {
                hideLoading();
            }
        }

        // Delete Item
        async function deleteItem(id) {
            const confirmDelete = await showModal('تایید حذف', 'آیا از حذف این کالا مطمئن هستید؟ این عملیات قابل بازگشت نیست.', true);
            if (!confirmDelete) return;

            showLoading();
            try {
                // Check if item has any stock or is in use
                const item = inventoryItems.find(i => i.id === id);
                if (item && (item.totalStock > 0 || item.inEventStock > 0 || item.damagedStock > 0)) {
                    showModal('خطا', 'این کالا دارای موجودی است یا در رویداد/تعمیر می‌باشد و قابل حذف نیست. ابتدا موجودی را صفر کنید.');
                    return;
                }

                await deleteDoc(doc(db, `artifacts/${appId}/users/${userId}/inventory`, id));
                showModal('موفقیت', 'کالا با موفقیت حذف شد.');
            } catch (e) {
                console.error("Error deleting document: ", e);
                showModal('خطا', 'خطا در حذف کالا: ' + e.message);
            } finally {
                hideLoading();
            }
        }

        // Add/Update Event
        async function addOrUpdateEvent(eventData) {
            showLoading();
            try {
                const eventRef = doc(db, `artifacts/${appId}/users/${userId}/events`, eventData.id);
                const docSnap = await getDoc(eventRef);

                if (docSnap.exists()) {
                    // Update existing event
                    await updateDoc(eventRef, {
                        name: eventData.name,
                        startDate: eventData.startDate,
                        endDate: eventData.endDate,
                        location: eventData.location,
                        responsible: eventData.responsible,
                        status: eventData.status,
                    });
                    showModal('موفقیت', 'رویداد با موفقیت ویرایش شد.');
                } else {
                    // Add new event
                    await setDoc(eventRef, {
                        name: eventData.name,
                        startDate: eventData.startDate,
                        endDate: eventData.endDate,
                        location: eventData.location,
                        responsible: eventData.responsible,
                        status: eventData.status,
                        createdAt: new Date(),
                    });
                    showModal('موفقیت', 'رویداد با موفقیت اضافه شد.');
                }
                document.getElementById('addEventForm').reset();
                document.getElementById('addEventForm').classList.add('hidden');
            } catch (e) {
                console.error("Error adding/updating event: ", e);
                showModal('خطا', 'خطا در ذخیره رویداد: ' + e.message);
            } finally {
                hideLoading();
            }
        }

        // Delete Event
        async function deleteEvent(id) {
            const confirmDelete = await showModal('تایید حذف', 'آیا از حذف این رویداد مطمئن هستید؟ این عملیات قابل بازگشت نیست.', true);
            if (!confirmDelete) return;

            showLoading();
            try {
                // Check if any items are currently in this event
                const itemsInEvent = inventoryItems.filter(item => item.inEventStock > 0 && transactions.some(txn => txn.eventId === id && txn.type === 'خروج برای رویداد' && txn.itemId === item.id));
                if (itemsInEvent.length > 0) {
                    showModal('خطا', 'این رویداد دارای کالاهای در حال استفاده است و قابل حذف نیست. ابتدا تمام کالاها را از این رویداد بازگردانید.');
                    return;
                }

                await deleteDoc(doc(db, `artifacts/${appId}/users/${userId}/events`, id));
                showModal('موفقیت', 'رویداد با موفقیت حذف شد.');
            } catch (e) {
                console.error("Error deleting event: ", e);
                showModal('خطا', 'خطا در حذف رویداد: ' + e.message);
            } finally {
                hideLoading();
            }
        }

        // Register Withdrawal Transaction
        async function registerWithdrawalTransaction(transactionData) {
            showLoading();
            try {
                const itemRef = doc(db, `artifacts/${appId}/users/${userId}/inventory`, transactionData.itemId);
                const itemSnap = await getDoc(itemRef);

                if (!itemSnap.exists()) {
                    showModal('خطا', 'کالا با شناسه "' + transactionData.itemId + '" یافت نشد.');
                    return;
                }

                const item = itemSnap.data();
                if (item.availableStock < transactionData.quantity) {
                    showModal('خطا', 'موجودی در دسترس کافی نیست. موجودی فعلی: ' + item.availableStock + ' عدد.');
                    return;
                }

                // Update inventory stocks
                await updateDoc(itemRef, {
                    availableStock: item.availableStock - transactionData.quantity,
                    inEventStock: item.inEventStock + transactionData.quantity,
                });

                // Add transaction record
                await addDoc(collection(db, `artifacts/${appId}/users/${userId}/transactions`), {
                    timestamp: new Date(),
                    type: 'خروج برای رویداد',
                    eventId: transactionData.eventId,
                    itemId: transactionData.itemId,
                    quantity: transactionData.quantity,
                    itemStatus: 'سالم', // Assumed healthy on withdrawal
                    recordedBy: auth.currentUser.email || auth.currentUser.uid,
                    notes: transactionData.notes || '',
                });

                showModal('موفقیت', transactionData.quantity + ' عدد از "' + transactionData.itemId + '" با موفقیت برای رویداد "' + transactionData.eventId + '" خارج شد.');
                document.getElementById('withdrawalForm').reset();
                document.getElementById('withdrawalForm').classList.add('hidden');
            } catch (e) {
                console.error("Error registering withdrawal: ", e);
                showModal('خطا', 'خطا در ثبت خروج کالا: ' + e.message);
            } finally {
                hideLoading();
            }
        }

        // Register Return/New Item Transaction
        async function registerReturnTransaction(transactionData) {
            showLoading();
            try {
                const itemRef = doc(db, `artifacts/${appId}/users/${userId}/inventory`, transactionData.itemId);
                const itemSnap = await getDoc(itemRef);

                if (!itemSnap.exists()) {
                    // If item doesn't exist, it's a new item being added directly via return form
                    if (transactionData.eventId) { // Cannot return from event if item doesn't exist
                        showModal('خطا', 'کالا با شناسه "' + transactionData.itemId + '" یافت نشد. برای برگشت از رویداد، کالا باید از قبل موجود باشد.');
                        return;
                    }
                    // Add new item with initial stock
                    await setDoc(itemRef, {
                        name: transactionData.itemId, // Use ID as name for simplicity if not found
                        category: 'نامشخص',
                        description: 'اضافه شده از طریق فرم ورود',
                        totalStock: transactionData.quantity,
                        availableStock: transactionData.itemStatus === 'سالم' ? transactionData.quantity : 0,
                        inEventStock: 0,
                        damagedStock: transactionData.itemStatus === 'آسیب دیده' ? transactionData.quantity : 0,
                        location: 'نامشخص',
                        minStock: 0,
                        createdAt: new Date(),
                    });
                }

                const item = (await getDoc(itemRef)).data(); // Re-fetch item data after potential creation

                // Update inventory stocks based on transaction type and status
                let updatedAvailableStock = item.availableStock;
                let updatedInEventStock = item.inEventStock;
                let updatedDamagedStock = item.damagedStock;
                let updatedTotalStock = item.totalStock;

                if (transactionData.eventId) { // Return from event
                    if (item.inEventStock < transactionData.quantity) {
                        showModal('خطا', 'تعداد کالای ورودی از رویداد بیشتر از موجودی در رویداد است. موجودی در رویداد فعلی: ' + item.inEventStock + ' عدد.');
                        return;
                    }
                    updatedInEventStock -= transactionData.quantity;
                    if (transactionData.itemStatus === 'سالم') {
                        updatedAvailableStock += transactionData.quantity;
                    } else { // 'آسیب دیده'
                        updatedDamagedStock += transactionData.quantity;
                    }
                } else { // New purchase / direct entry
                    updatedTotalStock += transactionData.quantity;
                    if (transactionData.itemStatus === 'سالم') {
                        updatedAvailableStock += transactionData.quantity;
                    } else { // 'آسیب دیده'
                        updatedDamagedStock += transactionData.quantity;
                    }
                }

                await updateDoc(itemRef, {
                    totalStock: updatedTotalStock,
                    availableStock: updatedAvailableStock,
                    inEventStock: updatedInEventStock,
                    damagedStock: updatedDamagedStock,
                });

                // Add transaction record
                await addDoc(collection(db, `artifacts/${appId}/users/${userId}/transactions`), {
                    timestamp: new Date(),
                    type: transactionData.eventId ? 'ورود از رویداد' : 'خرید جدید/ورود مستقیم',
                    eventId: transactionData.eventId || null,
                    itemId: transactionData.itemId,
                    quantity: transactionData.quantity,
                    itemStatus: transactionData.itemStatus,
                    recordedBy: auth.currentUser.email || auth.currentUser.uid,
                    notes: transactionData.notes || '',
                });

                showModal('موفقیت', transactionData.quantity + ' عدد از "' + transactionData.itemId + '" با موفقیت وارد انبار شد. وضعیت: ' + transactionData.itemStatus);
                document.getElementById('returnForm').reset();
                document.getElementById('returnForm').classList.add('hidden');
            } catch (e) {
                console.error("Error registering return: ", e);
                showModal('خطا', 'خطا در ثبت ورود کالا: ' + e.message);
            } finally {
                hideLoading();
            }
        }

        // --- Event Handlers ---

        // Tab Switching
        document.getElementById('dashboardTab').addEventListener('click', () => switchTab('dashboardContent'));
        document.getElementById('inventoryTab').addEventListener('click', () => switchTab('inventoryContent'));
        document.getElementById('eventsTab').addEventListener('click', () => switchTab('eventsContent'));
        document.getElementById('transactionsTab').addEventListener('click', () => switchTab('transactionsContent'));

        function switchTab(activeTabId) {
            document.querySelectorAll('.tab-content').forEach(content => {
                content.classList.add('hidden');
            });
            document.getElementById(activeTabId).classList.remove('hidden');

            document.querySelectorAll('.tab-button').forEach(button => {
                button.classList.remove('active');
            });
            document.getElementById(activeTabId.replace('Content', 'Tab')).classList.add('active');
        }

        // Show/Hide Add Item Form
        document.getElementById('showAddItemFormBtn').addEventListener('click', () => {
            document.getElementById('addItemForm').classList.remove('hidden');
            document.getElementById('item-id').focus();
        });
        document.getElementById('cancelAddItemFormBtn').addEventListener('click', () => {
            document.getElementById('addItemForm').classList.add('hidden');
            document.getElementById('addItemForm').reset();
        });

        // Add Item Form Submission
        document.getElementById('addItemForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const itemId = document.getElementById('item-id').value.trim();
            const itemName = document.getElementById('item-name').value.trim();
            const itemCategory = document.getElementById('item-category').value.trim();
            const itemDescription = document.getElementById('item-description').value.trim();
            const itemLocation = document.getElementById('item-location').value.trim();
            const itemMinStock = parseInt(document.getElementById('item-min-stock').value) || 0;

            if (!itemId || !itemName) {
                showModal('خطا', 'شناسه کالا و نام کالا الزامی هستند.');
                return;
            }

            addOrUpdateItem({
                id: itemId,
                name: itemName,
                category: itemCategory,
                description: itemDescription,
                location: itemLocation,
                minStock: itemMinStock,
            });
        });

        // Edit Item (pre-fill form)
        function editItem(id) {
            const item = inventoryItems.find(i => i.id === id);
            if (item) {
                document.getElementById('item-id').value = item.id;
                document.getElementById('item-id').readOnly = true; // Prevent changing ID
                document.getElementById('item-name').value = item.name;
                document.getElementById('item-category').value = item.category || '';
                document.getElementById('item-description').value = item.description || '';
                document.getElementById('item-location').value = item.location || '';
                document.getElementById('item-min-stock').value = item.minStock || 0;
                document.getElementById('addItemForm').classList.remove('hidden');
                document.getElementById('item-name').focus();
            }
        }

        // Show/Hide Add Event Form
        document.getElementById('showAddEventFormBtn').addEventListener('click', () => {
            document.getElementById('addEventForm').classList.remove('hidden');
            document.getElementById('event-id').focus();
        });
        document.getElementById('cancelAddEventFormBtn').addEventListener('click', () => {
            document.getElementById('addEventForm').classList.add('hidden');
            document.getElementById('addEventForm').reset();
            document.getElementById('event-id').readOnly = false; // Reset readOnly for ID
        });

        // Add Event Form Submission
        document.getElementById('addEventForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const eventId = document.getElementById('event-id').value.trim();
            const eventName = document.getElementById('event-name').value.trim();
            const eventStartDate = document.getElementById('event-start-date').value;
            const eventEndDate = document.getElementById('event-end-date').value;
            const eventLocation = document.getElementById('event-location').value.trim();
            const eventResponsible = document.getElementById('event-responsible').value.trim();
            const eventStatus = document.getElementById('event-status').value;

            if (!eventId || !eventName) {
                showModal('خطا', 'شناسه رویداد و نام رویداد الزامی هستند.');
                return;
            }

            addOrUpdateEvent({
                id: eventId,
                name: eventName,
                startDate: eventStartDate,
                endDate: eventEndDate,
                location: eventLocation,
                responsible: eventResponsible,
                status: eventStatus,
            });
        });

        // Edit Event (pre-fill form)
        function editEvent(id) {
            const event = events.find(e => e.id === id);
            if (event) {
                document.getElementById('event-id').value = event.id;
                document.getElementById('event-id').readOnly = true; // Prevent changing ID
                document.getElementById('event-name').value = event.name;
                document.getElementById('event-start-date').value = event.startDate || '';
                document.getElementById('event-end-date').value = event.endDate || '';
                document.getElementById('event-location').value = event.location || '';
                document.getElementById('event-responsible').value = event.responsible || '';
                document.getElementById('event-status').value = event.status;
                document.getElementById('addEventForm').classList.remove('hidden');
                document.getElementById('event-name').focus();
            }
        }

        // Show/Hide Withdrawal Form
        document.getElementById('showWithdrawalFormBtn').addEventListener('click', () => {
            document.getElementById('withdrawalForm').classList.remove('hidden');
            document.getElementById('returnForm').classList.add('hidden'); // Hide return form if open
            document.getElementById('withdrawal-item-id').focus();
        });
        document.getElementById('cancelWithdrawalFormBtn').addEventListener('click', () => {
            document.getElementById('withdrawalForm').classList.add('hidden');
            document.getElementById('withdrawalForm').reset();
        });

        // Withdrawal Form Submission
        document.getElementById('withdrawalForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const itemId = document.getElementById('withdrawal-item-id').value.trim();
            const quantity = parseInt(document.getElementById('withdrawal-quantity').value);
            const eventId = document.getElementById('withdrawal-event-id').value;
            const notes = document.getElementById('withdrawal-notes').value.trim();

            if (!itemId || isNaN(quantity) || quantity <= 0 || !eventId) {
                showModal('خطا', 'لطفاً تمام فیلدهای الزامی را به درستی پر کنید (شناسه کالا، تعداد، و رویداد).');
                return;
            }

            registerWithdrawalTransaction({
                itemId: itemId,
                quantity: quantity,
                eventId: eventId,
                notes: notes,
            });
        });

        // Show/Hide Return Form
        document.getElementById('showReturnFormBtn').addEventListener('click', () => {
            document.getElementById('returnForm').classList.remove('hidden');
            document.getElementById('withdrawalForm').classList.add('hidden'); // Hide withdrawal form if open
            document.getElementById('return-item-id').focus();
        });
        document.getElementById('cancelReturnFormBtn').addEventListener('click', () => {
            document.getElementById('returnForm').classList.add('hidden');
            document.getElementById('returnForm').reset();
        });

        // Return Form Submission
        document.getElementById('returnForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const itemId = document.getElementById('return-item-id').value.trim();
            const quantity = parseInt(document.getElementById('return-quantity').value);
            const eventId = document.getElementById('return-event-id').value || null; // Can be null for new purchases
            const itemStatus = document.getElementById('return-status').value;
            const notes = document.getElementById('return-notes').value.trim();

            if (!itemId || isNaN(quantity) || quantity <= 0 || !itemStatus) {
                showModal('خطا', 'لطفاً تمام فیلدهای الزامی را به درستی پر کنید (شناسه کالا، تعداد، و وضعیت).');
                return;
            }

            registerReturnTransaction({
                itemId: itemId,
                quantity: quantity,
                eventId: eventId,
                itemStatus: itemStatus,
                notes: notes,
            });
        });

        // Initial tab load
        window.onload = () => {
            showLoading(); // Show loading initially
            switchTab('dashboardContent'); // Default to dashboard
        };

    </script>
</body>
</html>
