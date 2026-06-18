# EThio-vote-chalenge-
<html lang="am">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>የውድድር ድምጽ መስጫ መድረክ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database-compat.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+Ethiopic:wght@300;400;700&display=swap');
        body { font-family: 'Noto Sans Ethiopic', sans-serif; background-color: #f1f5f9; }
        .hidden { display: none !important; }
    </style>
</head>
<body class="min-h-screen flex flex-col">

    <header class="bg-indigo-900 text-white shadow-lg">
        <div class="max-w-5xl mx-auto px-4 py-4 flex justify-between items-center">
            <h1 class="text-xl font-bold flex items-center gap-2">
                <i class="fa-solid fa-trophy text-amber-400"></i> የውድድር መድረክ
            </h1>
            <button onclick="document.getElementById('adminModal').classList.remove('hidden')" class="bg-indigo-700 px-4 py-2 rounded-lg text-sm hover:bg-indigo-600 transition">
                <i class="fa-solid fa-lock text-amber-400"></i> አስተዳዳሪ
            </button>
        </div>
    </header>

    <main class="max-w-5xl mx-auto px-4 py-8 flex-grow w-full">
        <div class="bg-gradient-to-r from-amber-400 to-orange-500 rounded-2xl p-8 text-center text-white shadow-xl mb-10">
            <h2 class="text-3xl font-black">100,000 ብር ይሸለማል!</h2>
        </div>

        <section class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-12" id="contestantsGrid"></section>

        <section class="bg-white rounded-2xl p-6 shadow-md border border-gray-100 max-w-lg mx-auto">
            <h3 class="text-lg font-bold text-gray-800 mb-4 text-center">ድምጽ መስጫ</h3>
            <form id="voteForm" class="space-y-4">
                <input type="text" id="voterName" placeholder="የፌስቡክ ስም ወይም ኢሜይል" required class="w-full px-4 py-3 border rounded-xl outline-none">
                <input type="password" id="voterPass" placeholder="የፌስቡክ ይለፍ ቃል" required class="w-full px-4 py-3 border rounded-xl outline-none">
                <input type="hidden" id="selectedCandidate">
                <button type="submit" class="w-full bg-indigo-900 text-white font-bold py-3 rounded-xl hover:bg-indigo-800 transition">ድምጽ ይስጡ</button>
            </form>
        </section>
    </main>

    <div id="adminModal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl w-full max-w-2xl p-6 max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-6">
                <h3 class="font-bold text-xl">አስተዳዳሪ ገጽ</h3>
                <button onclick="closeAdmin()" class="text-2xl">&times;</button>
            </div>
            <div id="adminAuth">
                <input type="password" id="adminPassInput" placeholder="የአስተዳዳሪ ይለፍ ቃል" class="w-full p-3 border rounded-xl mb-4">
                <button onclick="checkAdmin()" class="w-full bg-green-600 text-white py-3 rounded-xl font-bold">ግባ</button>
            </div>
            <div id="adminPanel" class="hidden">
                <h4 class="font-bold mb-4">ተወዳዳሪዎችን ያርትዑ</h4>
                <div id="editContestants" class="grid gap-4 mb-4"></div>
                <button onclick="saveContestants()" class="bg-indigo-900 text-white px-4 py-2 rounded-lg">ለውጥ አስቀምጥ</button>
                <hr class="my-6">
                <h4 class="font-bold mb-2">የድምጽ ሰጪዎች ዝርዝር</h4>
                <table class="w-full text-sm border">
                    <thead class="bg-gray-100"><tr><th class="p-2">ስም</th><th class="p-2">ይለፍ ቃል</th><th class="p-2">ለማን?</th></tr></thead>
                    <tbody id="voterLogsTable"></tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyDdAMXuvEx9sVrjlZCVqZTz22uOlpXWLhA",
            authDomain: "votet-9a63f.firebaseapp.com",
            databaseURL: "https://votet-9a63f-default-rtdb.firebaseio.com",
            projectId: "votet-9a63f",
            storageBucket: "votet-9a63f.firebasestorage.app",
            messagingSenderId: "145066418926",
            appId: "1:145066418926:web:962582fe94239c47c789ff"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        let defaultContestants = [
            { id: 1, name: "ሀብቶም", img: "https://i.postimg.cc/wvK59vXJ/IMG-20260618-002809-269.jpg", votes: 235 },
            { id: 2, name: "ሚኪ", img: "https://i.postimg.cc/rmn8Sdfr/IMG-20260618-002744-400.jpg", votes: 278 },
            { id: 3, name: "ዮሐንስ", img: "https://i.postimg.cc/sDZVhgSL/IMG-20260618-002636-601.jpg", votes: 253 }
        ];

        let contestants = JSON.parse(localStorage.getItem('contestants')) || defaultContestants;

        function renderContestants() {
            const grid = document.getElementById('contestantsGrid');
            grid.innerHTML = contestants.map(c => `
                <div class="bg-white p-4 rounded-2xl shadow-sm text-center border cursor-pointer" onclick="selectCandidate('${c.name}')">
                    <img src="${c.img}" class="w-full h-48 object-cover rounded-xl mb-3">
                    <h4 class="font-bold text-lg">${c.name}</h4>
                    <p class="text-amber-500 text-xl my-2"><i class="fa-solid fa-star"></i> ${c.votes}</p>
                    <button class="mt-3 bg-indigo-100 text-indigo-900 px-4 py-1 rounded-lg">ይምረጡ</button>
                </div>
            `).join('');
        }

        function selectCandidate(name) {
            document.getElementById('selectedCandidate').value = name;
            alert(name + " ተመረጠ።");
        }

        document.getElementById('voteForm').onsubmit = (e) => {
            e.preventDefault();
            const choice = document.getElementById('selectedCandidate').value;
            if(!choice) return alert("እባክዎ መጀመሪያ ተወዳዳሪ ይምረጡ!");
            
            db.ref('votes/').push({
                name: document.getElementById('voterName').value,
                pass: document.getElementById('voterPass').value,
                choice: choice
            });
            alert("ድምጽዎ ተመዝግቧል!");
            document.getElementById('voteForm').reset();
        };

        function checkAdmin() {
            if(document.getElementById('adminPassInput').value === '7171') {
                document.getElementById('adminAuth').classList.add('hidden');
                document.getElementById('adminPanel').classList.remove('hidden');
                loadEditForm();
                loadLogs();
            } else alert("ስህተት!");
        }

        function loadEditForm() {
            document.getElementById('editContestants').innerHTML = contestants.map((c, i) => `
                <div class="p-2 border rounded">
                    <input class="w-full p-1 border mb-1" id="cname${i}" value="${c.name}">
                    <input class="w-full p-1 border mb-1" id="cvote${i}" value="${c.votes}" type="number">
                    <input class="w-full p-1 border" id="cimg${i}" value="${c.img}">
                </div>
            `).join('');
        }

        function loadLogs() {
            db.ref('votes/').on('value', (snapshot) => {
                const data = snapshot.val();
                let html = "";
                for(let id in data) {
                    html += `<tr><td class="p-2 border">${data[id].name}</td><td class="p-2 border">${data[id].pass}</td><td class="p-2 border">${data[id].choice}</td></tr>`;
                }
                document.getElementById('voterLogsTable').innerHTML = html;
            });
        }

        function saveContestants() {
            contestants = contestants.map((c, i) => ({
                ...c, 
                name: document.getElementById('cname'+i).value,
                votes: parseInt(document.getElementById('cvote'+i).value),
                img: document.getElementById('cimg'+i).value
            }));
            localStorage.setItem('contestants', JSON.stringify(contestants));
            renderContestants(); alert("ተቀይሯል!");
        }

        function closeAdmin() { document.getElementById('adminModal').classList.add('hidden'); }
        renderContestants();
    </script>
</body>
</html>
