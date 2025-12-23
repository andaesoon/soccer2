<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>스마트 풋살 팀 편성</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800 p-4 min-h-screen flex flex-col items-center">
    
    <div class="container mx-auto p-6 bg-white rounded-xl shadow-lg w-full max-w-2xl">
        <h1 class="text-3xl font-bold text-center mb-6">스마트 풋살 팀 편성</h1>

        <!-- API 키 설정 섹션 (보안을 위해 UI에서 입력) -->
        <div class="mb-6 p-4 bg-yellow-50 border border-yellow-200 rounded-lg">
            <h2 class="text-sm font-bold text-yellow-800 mb-2 flex items-center">
                <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="Vector"></path></svg>
                API 설정 (보안 유지)
            </h2>
            <div class="flex space-x-2">
                <input type="password" id="apiKeyInput" placeholder="Gemini API 키를 입력하세요" class="flex-1 p-2 text-sm border border-yellow-300 rounded focus:outline-none focus:ring-1 focus:ring-yellow-500">
                <button onclick="saveApiKey()" class="px-4 py-2 bg-yellow-600 text-white text-sm font-semibold rounded hover:bg-yellow-700 transition">저장</button>
            </div>
            <p class="text-[10px] text-yellow-700 mt-1">* 키는 브라우저에만 저장되며 코드에 노출되지 않습니다.</p>
        </div>

        <!-- 1. 참가 인원 파악 섹션 -->
        <div class="mb-8 p-6 bg-gray-50 rounded-lg">
            <h2 class="text-xl font-semibold mb-4 text-blue-800">1. 참가 인원 파악</h2>
            <div class="flex items-center space-x-4 mb-4">
                <input type="file" id="imageUpload" accept="image/*" class="flex-1 text-sm text-gray-500
                    file:mr-4 file:py-2 file:px-4
                    file:rounded-full file:border-0
                    file:text-sm file:font-semibold
                    file:bg-blue-50 file:text-blue-700
                    hover:file:bg-blue-100 cursor-pointer">
                <button onclick="uploadImage()" class="px-6 py-2 bg-blue-600 text-white font-semibold rounded-full shadow hover:bg-blue-700 transition duration-300">사진 분석</button>
            </div>
            
            <div class="flex items-center space-x-2 border-t pt-4 border-gray-200">
                <input type="text" id="manualPlayerInput" placeholder="이름 직접 입력" class="flex-1 p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500">
                <button onclick="addPlayerManually()" class="px-6 py-2 bg-purple-600 text-white font-semibold rounded-full shadow hover:bg-purple-700 transition duration-300">추가</button>
            </div>
            <p id="statusMessage" class="mt-4 text-sm text-gray-600 font-medium"></p>
            <div id="playerList" class="mt-4 p-4 bg-white border border-gray-200 rounded-lg min-h-[5rem] flex flex-wrap gap-2">
                <p class="text-gray-400 w-full text-center py-4">명단이 비어 있습니다.</p>
            </div>
        </div>

        <!-- 2. 팀 설정 및 그룹 지정 섹션 -->
        <div class="mb-8 p-6 bg-gray-50 rounded-lg">
            <h2 class="text-xl font-semibold mb-4 text-green-800">2. 팀 설정 및 그룹 지정</h2>
            <div class="mb-4">
                <label for="teamCount" class="block text-sm font-medium text-gray-700 mb-1">나눌 팀 개수:</label>
                <select id="teamCount" class="block w-full p-2 border border-gray-300 rounded-md shadow-sm">
                    <option value="2">2개 팀</option>
                    <option value="3">3개 팀</option>
                    <option value="4">4개 팀</option>
                </select>
            </div>
            <div class="mb-4">
                <h3 class="font-medium text-lg mb-2">지인 그룹 지정 (한 팀 배정)</h3>
                <p class="text-xs text-gray-500 mb-2">아래 명단에서 이름을 클릭하여 선택한 뒤 그룹명을 입력하세요.</p>
                <div class="flex space-x-2">
                    <input type="text" id="groupName" placeholder="그룹 이름 (예: 친구들)" class="flex-1 p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500">
                    <button onclick="addGroup()" class="px-6 py-2 bg-green-600 text-white font-semibold rounded-full shadow hover:bg-green-700 transition duration-300">그룹 묶기</button>
                </div>
            </div>
            <div id="groupList" class="mt-4 p-4 bg-white border border-gray-200 rounded-lg min-h-[4rem]">
                <p class="text-gray-400">지정된 그룹이 없습니다.</p>
            </div>
        </div>

        <!-- 3. 팀 편성 및 결과 섹션 -->
        <div class="mb-8 p-6 bg-gray-50 rounded-lg">
            <h2 class="text-xl font-semibold mb-4 text-indigo-800">3. 결과 확인</h2>
            <button onclick="generateTeams()" class="w-full py-3 bg-indigo-600 text-white font-bold text-lg rounded-full shadow hover:bg-indigo-700 transition duration-300 mb-4">팀 편성 시작!</button>
            <div id="teamResults" class="space-y-3">
                <p class="text-gray-400 text-center py-8">팀 편성 결과가 여기에 표시됩니다.</p>
            </div>
        </div>
    </div>

    <script>
        // State management
        let allPlayers = [];
        let selectedPlayers = new Set();
        let groups = {};

        // Load API Key from localStorage on load
        window.onload = () => {
            const savedKey = localStorage.getItem('gemini_futsal_api_key');
            if (savedKey) {
                document.getElementById('apiKeyInput').value = savedKey;
            }
        };

        function saveApiKey() {
            const key = document.getElementById('apiKeyInput').value.trim();
            if (key) {
                localStorage.setItem('gemini_futsal_api_key', key);
                alert('API 키가 브라우저에 저장되었습니다.');
            } else {
                alert('키를 입력해주세요.');
            }
        }

        const fileToBase64 = (file) => {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result.split(',')[1]);
                reader.onerror = error => reject(error);
            });
        };

        async function uploadImage() {
            const apiKey = document.getElementById('apiKeyInput').value.trim();
            if (!apiKey) {
                alert('먼저 API 키를 입력하고 저장해주세요.');
                return;
            }

            const fileInput = document.getElementById('imageUpload');
            const file = fileInput.files[0];
            const status = document.getElementById('statusMessage');
            
            if (!file) {
                status.textContent = '이미지 파일을 선택해주세요.';
                return;
            }

            status.textContent = '이미지 분석 중... 잠시만 기다려 주세요.';
            const playerList = document.getElementById('playerList');
            playerList.innerHTML = '<p class="text-gray-400 w-full text-center py-4">분석 중입니다...</p>';

            try {
                const base64ImageData = await fileToBase64(file);
                const prompt = "Please identify and list only the names of the people (voters/participants) from the image. Respond with a comma-separated list of names only (e.g., 'Anderson, 국주, 김남혁'). Remove any prefixes or numbers. Do not add explanations.";
                
                const payload = {
                    contents: [{
                        role: "user",
                        parts: [
                            { text: prompt },
                            { inlineData: { mimeType: file.type, data: base64ImageData } }
                        ]
                    }],
                };

                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                const result = await response.json();
                
                if (result.error) {
                    throw new Error(result.error.message);
                }

                let text = '';
                if (result.candidates?.[0]?.content?.parts?.[0]?.text) {
                    text = result.candidates[0].content.parts[0].text;
                }
                
                if (text) {
                    let cleanedText = text.replace(/["`*]/g, '').trim(); 
                    const names = cleanedText.split(/[\n,]/) 
                                             .map(name => name.trim())
                                             .filter(name => name.length > 0);
                    
                    const newPlayers = [...new Set(names)];
                    allPlayers = [...new Set([...allPlayers, ...newPlayers])];
                    
                    status.textContent = `${newPlayers.length}명의 인원을 새로 파악했습니다.`;
                    renderPlayers();
                } else {
                    status.textContent = '이름을 찾을 수 없습니다.';
                    renderPlayers();
                }

            } catch (error) {
                status.textContent = `오류 발생: ${error.message}`;
                renderPlayers();
                console.error('API Error:', error);
            }
        }

        function renderPlayers() {
            const playerList = document.getElementById('playerList');
            playerList.innerHTML = '';
            
            if (allPlayers.length === 0) {
                playerList.innerHTML = '<p class="text-gray-400 w-full text-center py-4">명단이 비어 있습니다.</p>';
                return;
            }

            allPlayers.forEach(player => {
                const isSelected = selectedPlayers.has(player);
                const div = document.createElement('div');
                div.className = `flex items-center space-x-2 px-3 py-1.5 rounded-full border transition-all duration-200 cursor-pointer ${
                    isSelected ? 'bg-blue-600 text-white border-blue-700 shadow-md' : 'bg-white text-gray-700 border-gray-300 hover:border-blue-400'
                }`;
                
                div.onclick = () => togglePlayerSelection(player);
                
                const nameSpan = document.createElement('span');
                nameSpan.textContent = player;
                nameSpan.className = "text-sm font-medium";
                div.appendChild(nameSpan);

                const delBtn = document.createElement('button');
                delBtn.innerHTML = "&times;";
                delBtn.className = `ml-2 font-bold hover:text-red-500 ${isSelected ? 'text-blue-200' : 'text-gray-400'}`;
                delBtn.onclick = (e) => {
                    e.stopPropagation();
                    deletePlayer(player);
                };
                div.appendChild(delBtn);

                playerList.appendChild(div);
            });
        }

        function togglePlayerSelection(player) {
            if (selectedPlayers.has(player)) {
                selectedPlayers.delete(player);
            } else {
                selectedPlayers.add(player);
            }
            renderPlayers();
        }

        function deletePlayer(player) {
            allPlayers = allPlayers.filter(p => p !== player);
            selectedPlayers.delete(player);
            // Remove from groups if exists
            for (let group in groups) {
                groups[group] = groups[group].filter(p => p !== player);
                if (groups[group].length === 0) delete groups[group];
            }
            renderPlayers();
            renderGroups();
        }

        function addPlayerManually() {
            const input = document.getElementById('manualPlayerInput');
            const name = input.value.trim();
            if (name) {
                if (!allPlayers.includes(name)) {
                    allPlayers.push(name);
                    input.value = '';
                    renderPlayers();
                    document.getElementById('statusMessage').textContent = `"${name}" 추가됨.`;
                } else {
                    alert('이미 명단에 있는 이름입니다.');
                }
            }
        }

        function addGroup() {
            if (selectedPlayers.size < 2) {
                alert('그룹으로 묶으려면 최소 2명 이상 선택해야 합니다.');
                return;
            }
            const nameInput = document.getElementById('groupName');
            const name = nameInput.value.trim() || `그룹 ${Object.keys(groups).length + 1}`;
            
            groups[name] = Array.from(selectedPlayers);
            selectedPlayers.clear();
            nameInput.value = '';
            renderPlayers();
            renderGroups();
        }

        function renderGroups() {
            const list = document.getElementById('groupList');
            list.innerHTML = '';
            const names = Object.keys(groups);
            
            if (names.length === 0) {
                list.innerHTML = '<p class="text-gray-400">지정된 그룹이 없습니다.</p>';
                return;
            }

            names.forEach(name => {
                const div = document.createElement('div');
                div.className = "flex justify-between items-center p-2 mb-2 bg-green-50 border border-green-100 rounded text-sm";
                div.innerHTML = `<div><span class="font-bold text-green-800">[${name}]</span> ${groups[name].join(', ')}</div>`;
                
                const del = document.createElement('button');
                del.textContent = "해제";
                del.className = "text-red-500 hover:underline ml-2 text-xs";
                del.onclick = () => {
                    delete groups[name];
                    renderGroups();
                };
                div.appendChild(del);
                list.appendChild(div);
            });
        }

        function generateTeams() {
            if (allPlayers.length === 0) {
                alert('참가 인원이 없습니다.');
                return;
            }

            const teamCount = parseInt(document.getElementById('teamCount').value);
            const teams = Array.from({ length: teamCount }, () => []);
            
            // 인원 분배 로직
            let remaining = [...allPlayers];
            
            // 1. 그룹 우선 배정 (현재 가장 인원이 적은 팀에 배정)
            const sortedGroups = Object.values(groups).sort((a, b) => b.length - a.length);
            sortedGroups.forEach(groupMembers => {
                const targetTeam = teams.reduce((min, t) => t.length < min.length ? t : min, teams[0]);
                groupMembers.forEach(p => {
                    targetTeam.push(p);
                    remaining = remaining.filter(rp => rp !== p);
                });
            });

            // 2. 남은 인원 랜덤 배정
            remaining.sort(() => Math.random() - 0.5);
            remaining.forEach(p => {
                const targetTeam = teams.reduce((min, t) => t.length < min.length ? t : min, teams[0]);
                targetTeam.push(p);
            });

            renderResults(teams);
        }

        function renderResults(teams) {
            const container = document.getElementById('teamResults');
            container.innerHTML = '';
            
            teams.forEach((team, idx) => {
                const div = document.createElement('div');
                div.className = "p-4 bg-white border border-gray-200 rounded-lg shadow-sm";
                div.innerHTML = `
                    <div class="flex justify-between items-center mb-2">
                        <span class="font-bold text-indigo-700 text-lg">TEAM ${idx + 1}</span>
                        <span class="bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded text-xs font-bold">${team.length}명</span>
                    </div>
                    <p class="text-gray-700 leading-relaxed">${team.join(', ')}</p>
                `;
                container.appendChild(div);
            });
        }
    </script>
</body>
</html>
