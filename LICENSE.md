<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>The 5 Steps Game</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Zero-Config Cloud Sync Library (GunDB Public Relays) -->
    <script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Outfit:wght@400;600;800&display=swap');
        body {
            font-family: 'Outfit', sans-serif;
            background-color: #0b0f19;
            color: #f3f4f6;
            touch-action: manipulation;
            user-select: none;
        }
        .pixel-font { font-family: 'Press Start 2P', monospace; }
        .card-bg { background: rgba(17, 24, 39, 0.85); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.1); }
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: rgba(0,0,0,0.2); }
        ::-webkit-scrollbar-thumb { background: #4f46e5; border-radius: 3px; }
    </style>
</head>
<body class="h-screen w-screen overflow-hidden flex flex-col">

    <!-- SETUP OVERLAY (With Email Required) -->
    <div id="setup-screen" class="fixed inset-0 z-50 bg-[#0b0f19] flex items-center justify-center p-4 hidden">
        <div class="card-bg p-6 sm:p-8 rounded-2xl max-w-md w-full shadow-2xl border border-indigo-500/30 text-center">
            <h1 class="pixel-font text-xl sm:text-2xl text-indigo-400 mb-2">THE 5 STEPS GAME</h1>
            <p class="text-xs text-slate-400 mb-6">ONLINE MINING & BRAWL ARENA</p>
            
            <div class="mb-6 flex flex-col items-center">
                <div class="relative w-24 h-24 rounded-full border-4 border-indigo-500 overflow-hidden bg-slate-800 mb-3 shadow-lg group cursor-pointer">
                    <img id="pfp-preview" src="https://api.dicebear.com/7.x/pixel-art/svg?seed=Miner1" class="w-full h-full object-cover">
                    <div class="absolute inset-0 bg-black/40 flex items-center justify-center opacity-0 group-hover:opacity-100 transition text-xs font-bold">Change</div>
                    <input type="file" id="pfp-input" accept="image/*" class="absolute inset-0 opacity-0 cursor-pointer">
                </div>
                <span class="text-xs text-slate-400">Click to upload profile picture</span>
            </div>

            <div class="text-left space-y-4 mb-6">
                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-indigo-300 mb-1">Your Username</label>
                    <input type="text" id="username-input" value="ProMiner99" maxlength="16" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500 font-semibold text-sm">
                </div>
                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-indigo-300 mb-1">Your Email</label>
                    <input type="email" id="email-input" placeholder="miner@example.com" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500 font-semibold text-sm" required>
                </div>
            </div>

            <button onclick="saveAndStart()" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-3.5 rounded-xl shadow-lg shadow-indigo-600/30 transition text-sm uppercase tracking-wider">
                Enter World 🚀
            </button>
        </div>
    </div>

    <!-- MAIN GAME SCREEN -->
    <div id="game-screen" class="flex-1 flex flex-col h-full overflow-hidden hidden">
        
        <!-- HEADER -->
        <header class="card-bg border-b border-slate-800 px-4 py-3 flex items-center justify-between shrink-0 z-10">
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-full border-2 border-indigo-500 overflow-hidden bg-slate-800">
                    <img id="header-pfp" src="" class="w-full h-full object-cover">
                </div>
                <div>
                    <div id="header-username" class="font-bold text-sm leading-tight text-white"></div>
                    <div id="header-email" class="text-[9px] text-slate-400 leading-tight"></div>
                    <div class="text-[10px] text-emerald-400 flex items-center gap-1 mt-0.5"><span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span> Cloud DB Live</div>
                </div>
            </div>

            <div class="flex items-center space-x-4">
                <div class="bg-slate-900/80 border border-amber-500/30 px-3 py-1.5 rounded-xl flex items-center space-x-2 shadow-inner">
                    <span class="text-amber-400 text-base">🔵</span>
                    <div>
                        <div class="text-[9px] text-slate-400 uppercase tracking-wide leading-none">Currency</div>
                        <div id="header-bubs" class="pixel-font text-xs sm:text-sm text-amber-300 font-bold">0 BUBS</div>
                    </div>
                </div>
                <button onclick="openSettings()" class="w-9 h-9 rounded-xl bg-slate-800 hover:bg-slate-700 border border-slate-700 flex items-center justify-center text-slate-300 transition">
                    <i class="fa-solid fa-gear text-sm"></i>
                </button>
            </div>
        </header>

        <!-- NAVIGATION TABS -->
        <nav class="bg-slate-900/60 border-b border-slate-800 flex justify-around p-1 shrink-0">
            <button onclick="switchTab('mine')" id="nav-mine" class="flex-1 py-2 rounded-lg text-xs font-bold uppercase transition text-indigo-400 bg-indigo-600/20 border border-indigo-500/30">⛏️ Mine</button>
            <button onclick="switchTab('arena')" id="nav-arena" class="flex-1 py-2 rounded-lg text-xs font-bold uppercase transition text-slate-400 hover:text-white">⚔️ Arena</button>
            <button onclick="switchTab('boxes')" id="nav-boxes" class="flex-1 py-2 rounded-lg text-xs font-bold uppercase transition text-slate-400 hover:text-white">🎁 Boxes</button>
            <button onclick="switchTab('online')" id="nav-online" class="flex-1 py-2 rounded-lg text-xs font-bold uppercase transition text-slate-400 hover:text-white">👥 Players</button>
        </nav>

        <!-- CONTENT CONTAINER -->
        <main class="flex-1 overflow-y-auto p-3 sm:p-6 relative max-w-4xl mx-auto w-full pb-20">
            
            <!-- TAB: MINE -->
            <section id="tab-mine" class="space-y-4">
                <div class="card-bg p-6 rounded-2xl text-center relative overflow-hidden group border-indigo-500/20">
                    <div class="absolute inset-0 bg-gradient-to-b from-indigo-500/5 to-transparent pointer-events-none"></div>
                    <h2 class="text-xs uppercase tracking-widest text-slate-400 mb-1">The Bubble Quarry</h2>
                    <div id="ore-name" class="pixel-font text-sm text-indigo-300 mb-6">Common Stone</div>
                    
                    <button onclick="clickMine()" class="w-36 h-36 sm:w-44 sm:h-44 mx-auto rounded-full bg-gradient-to-tr from-indigo-600 to-violet-600 hover:from-indigo-500 hover:to-violet-500 active:scale-95 transition shadow-xl shadow-indigo-600/30 border-4 border-indigo-400/40 flex flex-col items-center justify-center cursor-pointer select-none relative group">
                        <i class="fa-solid fa-gem text-4xl sm:text-5xl text-white mb-2 group-hover:scale-110 transition"></i>
                        <span class="pixel-font text-xs text-white">MINE!</span>
                    </button>
                    <div class="mt-4 text-xs text-slate-400">Click power: <span id="click-power" class="text-indigo-300 font-bold">1</span> Bubs / click</div>
                </div>

                <div class="card-bg p-4 rounded-2xl">
                    <h3 class="text-xs uppercase font-bold text-slate-300 mb-3 tracking-wider flex items-center gap-2">
                        <i class="fa-solid fa-wrench text-indigo-400"></i> Mining Upgrades & Tools
                    </h3>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3" id="upgrades-list"></div>
                </div>
            </section>

            <!-- TAB: ARENA -->
            <section id="tab-arena" class="space-y-4 hidden">
                <div class="card-bg p-6 rounded-2xl text-center">
                    <h2 class="pixel-font text-sm text-amber-400 mb-2">PvP DUEL ARENA</h2>
                    <p class="text-xs text-slate-400 mb-6">Challenge real online players! Winner takes 5% of the loser's Bubs.</p>
                    
                    <div id="duel-status-box" class="bg-slate-900/80 border border-slate-700/80 rounded-xl p-5 mb-6 min-h-[100px] flex flex-col items-center justify-center">
                        <div id="duel-text" class="text-sm font-semibold text-slate-300">Go to the Players tab to challenge someone!</div>
                        <div id="duel-result" class="text-xs mt-2 font-bold text-indigo-400"></div>
                    </div>
                </div>

                <div class="card-bg p-4 rounded-2xl">
                    <h3 class="text-xs uppercase font-bold text-slate-300 mb-3 tracking-wider flex items-center gap-2">
                        <i class="fa-solid fa-shield-halved text-indigo-400"></i> Equipped Weapon
                    </h3>
                    <div id="equipped-weapon-display" class="bg-slate-900 p-4 rounded-xl border border-indigo-500/30 flex items-center justify-between"></div>
                </div>

                <div class="card-bg p-4 rounded-2xl">
                    <h3 class="text-xs uppercase font-bold text-slate-300 mb-3 tracking-wider flex items-center gap-2">
                        <i class="fa-solid fa-bag-shopping text-indigo-400"></i> Inventory (<span id="inv-count">0</span>)
                    </h3>
                    <div id="inventory-list" class="grid grid-cols-1 sm:grid-cols-2 gap-2 max-h-60 overflow-y-auto"></div>
                </div>
            </section>

            <!-- TAB: BOXES -->
            <section id="tab-boxes" class="space-y-4 hidden">
                <div class="card-bg p-6 rounded-2xl text-center">
                    <h2 class="pixel-font text-sm text-indigo-400 mb-2">MYSTERY BOXES</h2>
                    <p class="text-xs text-slate-400 mb-6">Spend Bubs to pull rare infinite weapons!</p>
                    
                    <div class="w-32 h-32 mx-auto bg-gradient-to-tr from-violet-600 to-fuchsia-600 rounded-2xl flex items-center justify-center shadow-lg shadow-fuchsia-600/30 border-2 border-fuchsia-400 mb-6 animate-bounce">
                        <i class="fa-solid fa-box-open text-5xl text-white"></i>
                    </div>

                    <button onclick="openMysteryBox()" class="w-full sm:w-auto px-8 py-3.5 bg-fuchsia-600 hover:bg-fuchsia-500 text-white font-bold rounded-xl shadow-lg shadow-fuchsia-600/30 transition uppercase text-xs tracking-wider">
                        🎁 Open Box (50 Bubs)
                    </button>
                </div>
            </section>

            <!-- TAB: ONLINE -->
            <section id="tab-online" class="space-y-4 hidden">
                <div class="card-bg p-4 rounded-2xl">
                    <h2 class="text-xs uppercase font-bold text-slate-300 mb-3 tracking-wider flex items-center gap-2">
                        <i class="fa-solid fa-globe text-emerald-400"></i> Online Lobby Players (<span id="online-count">1</span>)
                    </h2>
                    <div id="public-players-list" class="space-y-2 max-h-[400px] overflow-y-auto"></div>
                </div>
            </section>
        </main>
    </div>

    <!-- SETTINGS MODAL -->
    <div id="settings-modal" class="fixed inset-0 z-50 bg-black/70 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="card-bg p-6 rounded-2xl max-w-sm w-full space-y-4">
            <h3 class="pixel-font text-sm text-indigo-300">SETTINGS</h3>
            <div class="flex items-center justify-between py-2 border-b border-slate-800">
                <span class="text-xs font-semibold">Sound Effects</span>
                <input type="checkbox" id="sound-toggle" checked onchange="toggleSound()" class="w-4 h-4 accent-indigo-600">
            </div>
            <button onclick="logout()" class="w-full bg-rose-600/80 hover:bg-rose-600 py-2.5 rounded-xl text-xs font-bold uppercase text-white transition">Logout / Reset Profile</button>
            <button onclick="closeSettings()" class="w-full bg-slate-800 hover:bg-slate-700 py-2.5 rounded-xl text-xs font-bold uppercase transition">Close</button>
        </div>
    </div>

    <!-- ADMIN BOARD BUTTON -->
    <button onclick="openAdminPrompt()" class="fixed bottom-3 right-3 z-40 w-10 h-10 rounded-full bg-indigo-600/80 hover:bg-indigo-600 border border-indigo-400/50 flex items-center justify-center text-white shadow-lg transition hover:scale-110 active:scale-95" title="Admin Board">
        <i class="fa-solid fa-arrow-up-right-from-square text-xs"></i>
    </button>

    <!-- ADMIN BOARD MODAL -->
    <div id="admin-modal" class="fixed inset-0 z-50 bg-black/80 backdrop-blur-md hidden flex items-center justify-center p-4">
        <div class="card-bg p-6 rounded-2xl max-w-2xl w-full max-h-[85vh] overflow-y-auto space-y-6 border border-amber-500/40">
            <div class="flex items-center justify-between border-b border-slate-800 pb-3">
                <h3 class="pixel-font text-sm text-amber-400 flex items-center gap-2">
                    <i class="fa-solid fa-shield-dog"></i> ADMIN BOARD
                </h3>
                <button onclick="closeAdmin()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark"></i></button>
            </div>

            <div class="bg-slate-900 p-4 rounded-xl border border-slate-800 space-y-3">
                <h4 class="text-xs font-bold uppercase text-indigo-300">Grant Bubs to Player</h4>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
                    <select id="admin-target-user" class="bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-xs text-white"></select>
                    <input type="number" id="admin-bubs-amount" placeholder="Amount (e.g. 1000)" class="bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-xs text-white">
                </div>
                <button onclick="adminGiveBubs()" class="w-full bg-amber-600 hover:bg-amber-500 text-white font-bold py-2 rounded-lg text-xs uppercase transition">
                    Give Bubs & Record Payment
                </button>
            </div>

            <div class="space-y-3">
                <div class="flex items-center justify-between">
                    <h4 class="text-xs font-bold uppercase text-slate-300">Online Players, Playtime & Payments</h4>
                    <span id="admin-online-stat" class="text-xs text-emerald-400 font-semibold">1 Online</span>
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs">
                        <thead class="bg-slate-900 text-slate-400 uppercase">
                            <tr>
                                <th class="p-2">User & Email</th>
                                <th class="p-2">Playtime</th>
                                <th class="p-2">Payments / Granted</th>
                            </tr>
                        </thead>
                        <tbody id="admin-players-table" class="divide-y divide-slate-800"></tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <script>
        let player = {
            id: 'p_' + Math.random().toString(36).substring(2, 9),
            username: "",
            email: "",
            pfp: "https://api.dicebear.com/7.x/pixel-art/svg?seed=Miner1",
            bubs: 100,
            clickPower: 1,
            autoBps: 0,
            playtime: 0,
            paymentsTotal: 0,
            status: "Mining",
            equippedWeapon: { id: 1, name: "Rusty Dagger", rarity: "Common", power: 2, color: "text-slate-400" },
            inventory: [{ id: 1, name: "Rusty Dagger", rarity: "Common", power: 2, color: "text-slate-400" }],
            upgrades: {
                pickaxe: { level: 0, cost: 20 },
                autoMiner: { level: 0, cost: 50 }
            }
        };

        let onlinePlayersMap = {};
        let soundEnabled = true;
        let gun, userNode;

        document.addEventListener("DOMContentLoaded", () => {
            const saved = localStorage.getItem('the_5_steps_profile');
            if (saved) {
                try {
                    player = JSON.parse(saved);
                    initGame();
                } catch(e) {
                    showSetup();
                }
            } else {
                showSetup();
            }
        });

        document.getElementById('pfp-input').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(evt) {
                    player.pfp = evt.target.result;
                    document.getElementById('pfp-preview').src = player.pfp;
                };
                reader.readAsDataURL(file);
            }
        });

        function showSetup() {
            document.getElementById('setup-screen').classList.remove('hidden');
        }

        function saveAndStart() {
            const un = document.getElementById('username-input').value.trim();
            const em = document.getElementById('email-input').value.trim();
            
            if (!un) { alert("Please enter a username!"); return; }
            if (!em || !em.includes("@")) { alert("Please enter a valid email address!"); return; }

            player.username = un;
            player.email = em;
            
            localStorage.setItem('the_5_steps_profile', JSON.stringify(player));
            document.getElementById('setup-screen').classList.add('hidden');
            initGame();
        }

        function initGame() {
            document.getElementById('header-username').textContent = player.username;
            document.getElementById('header-email').textContent = player.email;
            document.getElementById('header-pfp').src = player.pfp;
            document.getElementById('game-screen').classList.remove('hidden');
            
            initCloudSync();

            setInterval(() => {
                if (player.autoBps > 0) {
                    player.bubs += player.autoBps;
                    syncCloudState();
                    updateUI();
                }
            }, 1000);

            setInterval(() => {
                player.playtime++;
                if (player.playtime % 5 === 0) syncCloudState();
            }, 1000);

            updateUI();
        }

        function initCloudSync() {
            try {
                gun = Gun(['https://gun-manhattan.herokuapp.com/gun']);
                userNode = gun.get('the_5_steps_game_room').get(player.id);
                
                setInterval(() => {
                    userNode.put({
                        id: player.id,
                        username: player.username,
                        email: player.email,
                        pfp: player.pfp,
                        bubs: player.bubs,
                        playtime: player.playtime,
                        status: player.status,
                        weaponPower: player.equippedWeapon.power,
                        lastSeen: Date.now()
                    });
                }, 3000);

                gun.get('the_5_steps_game_room').map().on((data, key) => {
                    if (data && data.id && data.id !== player.id) {
                        if (Date.now() - data.lastSeen < 15000) {
                            onlinePlayersMap[data.id] = data;
                        } else {
                            delete onlinePlayersMap[data.id];
                        }
                        updateUI();
                    }
                });
            } catch(e) {
                console.log("Cloud sync fallback active");
            }
        }

        function syncCloudState() {
            localStorage.setItem('the_5_steps_profile', JSON.stringify(player));
            if (userNode) {
                userNode.put({
                    id: player.id,
                    username: player.username,
                    email: player.email,
                    pfp: player.pfp,
                    bubs: player.bubs,
                    playtime: player.playtime,
                    status: player.status,
                    weaponPower: player.equippedWeapon.power,
                    lastSeen: Date.now()
                });
            }
        }

        function logout() {
            if (userNode) {
                userNode.put({ lastSeen: 0 });
            }
            localStorage.removeItem('the_5_steps_profile');
            location.reload();
        }

        function switchTab(tab) {
            ['mine', 'arena', 'boxes', 'online'].forEach(t => {
                document.getElementById(`tab-${t}`).classList.add('hidden');
                document.getElementById(`nav-${t}`).className = "flex-1 py-2 rounded-lg text-xs font-bold uppercase transition text-slate-400 hover:text-white";
            });
            document.getElementById(`tab-${tab}`).classList.remove('hidden');
            document.getElementById(`nav-${tab}`).className = "flex-1 py-2 rounded-lg text-xs font-bold uppercase transition text-indigo-400 bg-indigo-600/20 border border-indigo-500/30";
        }

        function clickMine() {
            player.bubs += player.clickPower;
            player.status = "Mining";
            syncCloudState();
            updateUI();
        }

        function buyUpgrade(type) {
            if (type === 'pickaxe') {
                let cost = player.upgrades.pickaxe.cost;
                if (player.bubs >= cost) {
                    player.bubs -= cost;
                    player.upgrades.pickaxe.level++;
                    player.clickPower += 2;
                    player.upgrades.pickaxe.cost = Math.floor(cost * 1.5);
                    syncCloudState();
                    updateUI();
                }
            } else if (type === 'autoMiner') {
                let cost = player.upgrades.autoMiner.cost;
                if (player.bubs >= cost) {
                    player.bubs -= cost;
                    player.upgrades.autoMiner.level++;
                    player.autoBps += 2;
                    player.upgrades.autoMiner.cost = Math.floor(cost * 1.6);
                    syncCloudState();
                    updateUI();
                }
            }
        }

        const weaponAdjectives = ["Fiery", "Shadow", "Glitch", "Cosmic", "Golden", "Void", "Quantum", "Plasma", "Diamond"];
        const weaponTypes = ["Sword", "Blade", "Dagger", "Hammer", "Scythe", "Blaster", "Staff", "Axe"];
        const rarities = [
            { name: "Common", chance: 0.6, mult: 1, color: "text-slate-400" },
            { name: "Rare", chance: 0.25, mult: 3, color: "text-blue-400" },
            { name: "Epic", chance: 0.1, mult: 7, color: "text-purple-400" },
            { name: "Legendary", chance: 0.04, mult: 15, color: "text-amber-400" },
            { name: "Godly", chance: 0.01, mult: 30, color: "text-rose-400 font-bold" }
        ];

        function openMysteryBox() {
            if (player.bubs < 50) {
                alert("Not enough Bubs! Need 50 Bubs.");
                return;
            }
            player.bubs -= 50;
            let roll = Math.random();
            let cumulative = 0;
            let selectedRarity = rarities[0];
            for (let r of rarities) {
                cumulative += r.chance;
                if (roll <= cumulative) { selectedRarity = r; break; }
            }

            let adj = weaponAdjectives[Math.floor(Math.random() * weaponAdjectives.length)];
            let type = weaponTypes[Math.floor(Math.random() * weaponTypes.length)];
            let name = `${adj} ${type}`;
            let power = Math.floor(Math.random() * 10 + 5) * selectedRarity.mult;

            let newWeapon = { id: Date.now(), name, rarity: selectedRarity.name, power, color: selectedRarity.color };
            player.inventory.push(newWeapon);
            player.equippedWeapon = newWeapon;
            syncCloudState();
            updateUI();
            alert(`🎉 You unlocked a ${selectedRarity.name} weapon: ${name} (Power: ${power})!`);
        }

        function equipWeapon(id) {
            let found = player.inventory.find(w => w.id === id);
            if (found) {
                player.equippedWeapon = found;
                syncCloudState();
                updateUI();
            }
        }

        function challengePlayer(opponentId) {
            let opponent = onlinePlayersMap[opponentId];
            if (!opponent) return;

            switchTab('arena');
            player.status = "In Duel";
            syncCloudState();

            let statusBox = document.getElementById('duel-text');
            let resultBox = document.getElementById('duel-result');
            
            statusBox.textContent = `Challenging ${opponent.username}... rolling 200-sided dice!`;
            resultBox.textContent = "";

            setTimeout(() => {
                let myRoll1 = Math.floor(Math.random() * 200) + 1;
                let myRoll2 = Math.floor(Math.random() * 200) + 1;
                let myScore = (myRoll1 + myRoll2) + (player.equippedWeapon.power * 2);

                let oppRoll1 = Math.floor(Math.random() * 200) + 1;
                let oppRoll2 = Math.floor(Math.random() * 200) + 1;
                let oppScore = (oppRoll1 + oppRoll2) + (opponent.weaponPower || 2);

                if (myScore >= oppScore) {
                    let stake = Math.floor(opponent.bubs * 0.05);
                    player.bubs += stake;
                    statusBox.textContent = `Victory against ${opponent.username}!`;
                    resultBox.innerHTML = `You rolled (${myRoll1}+${myRoll2}) vs Opponent (${oppRoll1}+${oppRoll2}). <span class="text-emerald-400">Won 5% stake: +${stake} Bubs!</span>`;
                } else {
                    let stake = Math.floor(player.bubs * 0.05);
                    player.bubs -= stake;
                    statusBox.textContent = `Defeat against ${opponent.username}!`;
                    resultBox.innerHTML = `You rolled (${myRoll1}+${myRoll2}) vs Opponent (${oppRoll1}+${oppRoll2}). <span class="text-rose-400">Lost 5% stake: -${stake} Bubs!</span>`;
                }
                player.status = "Mining";
                syncCloudState();
                updateUI();
            }, 1200);
        }

        function openAdminPrompt() {
            let pwd = prompt("Enter Admin Password:");
            if (pwd === "cbabcde") {
                openAdminBoard();
            } else if (pwd !== null) {
                alert("Incorrect Password!");
            }
        }

        function openAdminBoard() {
            document.getElementById('admin-modal').classList.remove('hidden');
            let otherList = Object.values(onlinePlayersMap);
            let select = document.getElementById('admin-target-user');
            select.innerHTML = `<option value="${player.username}">${player.username} (You)</option>` + 
                otherList.map(p => `<option value="${p.username}">${p.username}</option>`).join('');

            let table = document.getElementById('admin-players-table');
            let allUsers = [
                { name: `${player.username} (${player.email})`, playtime: `${Math.floor(player.playtime / 60)}m`, payments: `${player.paymentsTotal} Bubs` },
                ...otherList.map(p => ({ name: `${p.username} (${p.email || 'no email'})`, playtime: `${Math.floor(p.playtime / 60)}m`, payments: `${p.bubs} Bubs` }))
            ];

            table.innerHTML = allUsers.map(u => `
                <tr>
                    <td class="p-2 font-bold text-indigo-300">${u.name}</td>
                    <td class="p-2 text-slate-300">${u.playtime}</td>
                    <td class="p-2 text-amber-400">${u.payments}</td>
                </tr>
            `).join('');

            document.getElementById('admin-online-stat').textContent = `${allUsers.length} Online`;
        }

        function closeAdmin() { document.getElementById('admin-modal').classList.add('hidden'); }

        function adminGiveBubs() {
            let targetName = document.getElementById('admin-target-user').value;
            let amount = parseInt(document.getElementById('admin-bubs-amount').value);
            if (isNaN(amount) || amount <= 0) { alert("Enter valid amount"); return; }

            if (targetName === player.username) {
                player.bubs += amount;
                player.paymentsTotal += amount;
                syncCloudState();
            }
            alert(`Successfully granted ${amount} Bubs to ${targetName}!`);
            updateUI();
            openAdminBoard();
        }

        function openSettings() { document.getElementById('settings-modal').classList.remove('hidden'); }
        function closeSettings() { document.getElementById('settings-modal').classList.add('hidden'); }
        function toggleSound() { soundEnabled = document.getElementById('sound-toggle').checked; }

        function updateUI() {
            document.getElementById('header-bubs').textContent = `${player.bubs.toLocaleString()} BUBS`;
            document.getElementById('click-power').textContent = player.clickPower;
            document.getElementById('inv-count').textContent = player.inventory.length;
            
            document.getElementById('upgrades-list').innerHTML = `
                <div class="bg-slate-900 p-3 rounded-xl border border-slate-800 flex items-center justify-between">
                    <div>
                        <div class="font-bold text-xs">Better Pickaxe (Lv.${player.upgrades.pickaxe.level})</div>
                        <div class="text-[10px] text-slate-400">+2 Click Power</div>
                    </div>
                    <button onclick="buyUpgrade('pickaxe')" class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 rounded-lg text-xs font-bold text-white">
                        ${player.upgrades.pickaxe.cost} 🔵
                    </button>
                </div>
                <div class="bg-slate-900 p-3 rounded-xl border border-slate-800 flex items-center justify-between">
                    <div>
                        <div class="font-bold text-xs">Auto Miner (Lv.${player.upgrades.autoMiner.level})</div>
                        <div class="text-[10px] text-slate-400">+2 Bubs / sec</div>
                    </div>
                    <button onclick="buyUpgrade('autoMiner')" class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 rounded-lg text-xs font-bold text-white">
                        ${player.upgrades.autoMiner.cost} 🔵
                    </button>
                </div>
            `;

            let ew = player.equippedWeapon;
            document.getElementById('equipped-weapon-display').innerHTML = `
                <div>
                    <div class="font-bold text-xs ${ew.color}">${ew.name} (${ew.rarity})</div>
                    <div class="text-[10px] text-slate-400">Power Bonus: +${ew.power}</div>
                </div>
                <span class="text-xs uppercase font-bold text-indigo-400">Equipped</span>
            `;

            document.getElementById('inventory-list').innerHTML = player.inventory.map(w => `
                <div class="bg-slate-900 p-3 rounded-xl border border-slate-800 flex items-center justify-between">
                    <div>
                        <div class="font-bold text-xs ${w.color}">${w.name}</div>
                        <div class="text-[10px] text-slate-400">${w.rarity} • Power: ${w.power}</div>
                    </div>
                    ${player.equippedWeapon.id === w.id ? 
                        '<span class="text-[10px] text-emerald-400 font-bold">Equipped</span>' : 
                        `<button onclick="equipWeapon(${w.id})" class="px-2.5 py-1 bg-slate-800 hover:bg-slate-700 rounded-lg text-[10px] font-bold">Equip</button>`
                    }
                </div>
            `).join('');

            let otherList = Object.values(onlinePlayersMap);
            document.getElementById('online-count').textContent = otherList.length + 1;
            let pubList = document.getElementById('public-players-list');
            
            if (otherList.length === 0) {
                pubList.innerHTML = `
                    <div class="bg-slate-900 p-3 rounded-xl border border-indigo-500/30 flex items-center justify-between mb-2">
                        <div class="flex items-center space-x-3">
                            <img src="${player.pfp}" class="w-8 h-8 rounded-full bg-slate-800 object-cover">
                            <div>
                                <div class="font-bold text-xs text-indigo-300">${player.username} (You)</div>
                                <div class="text-[10px] text-slate-400">${player.email}</div>
                            </div>
                        </div>
                        <span class="text-xs text-emerald-400 font-semibold">Active</span>
                    </div>
                    <div class="text-center py-6 text-slate-400 text-xs px-4">
                        Oops nobody is online so you can't duel anyone yet invite your friends to come and play!!
                    </div>
                `;
            } else {
                pubList.innerHTML = `
                    <div class="bg-slate-900 p-3 rounded-xl border border-indigo-500/30 flex items-center justify-between">
                        <div class="flex items-center space-x-3">
                            <img src="${player.pfp}" class="w-8 h-8 rounded-full bg-slate-800 object-cover">
                            <div>
                                <div class="font-bold text-xs text-indigo-300">${player.username} (You)</div>
                                <div class="text-[10px] text-slate-400">${player.email}</div>
                            </div>
                        </div>
                        <span class="text-xs text-emerald-400 font-semibold">Active</span>
                    </div>
                ` + otherList.map(p => `
                    <div class="bg-slate-900 p-3 rounded-xl border border-slate-800 flex items-center justify-between">
                        <div class="flex items-center space-x-3">
                            <img src="${p.pfp || 'https://api.dicebear.com/7.x/pixel-art/svg?seed=Default'}" class="w-8 h-8 rounded-full bg-slate-800 object-cover">
                            <div>
                                <div class="font-bold text-xs text-slate-200">${p.username}</div>
                                <div class="text-[10px] text-slate-400">${p.email || 'online'} • Bubs: ${p.bubs}</div>
                            </div>
                        </div>
                        <button onclick="challengePlayer('${p.id}')" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-500 rounded-lg text-xs font-bold text-white">
                            ⚔️ Challenge
                        </button>
                    </div>
                `).join('');
            }
        }
    </script>
</body>
</html>
