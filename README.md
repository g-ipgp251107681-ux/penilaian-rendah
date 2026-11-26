<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>TerraNusantara: Misi Azan & Iqamat</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Plus+Jakarta+Sans:wght@500;600;700;800&display=swap" rel="stylesheet">
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
        }
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #0f172a;
            font-family: 'Plus Jakarta Sans', sans-serif;
        }
        .pixel-font {
            font-family: 'Press Start 2P', monospace;
        }
        canvas {
            display: block;
            width: 100%;
            height: 100%;
            image-rendering: pixelated;
        }
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(15, 23, 42, 0.5);
            border-radius: 8px;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(148, 163, 184, 0.4);
            border-radius: 8px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(148, 163, 184, 0.7);
        }
        .hotbar-slot {
            transition: transform 0.1s ease, border-color 0.15s ease, background-color 0.15s ease;
        }
        .hotbar-slot.active {
            transform: translateY(-4px) scale(1.06);
            border-color: #fbbf24;
            background-color: rgba(251, 191, 36, 0.25);
            box-shadow: 0 0 15px rgba(251, 191, 36, 0.45);
        }
        .glass-panel {
            background: rgba(15, 23, 42, 0.84);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            border: 2px solid rgba(148, 163, 184, 0.25);
        }
        @keyframes floatSlow {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-6px); }
        }
        .animate-float {
            animation: floatSlow 3s ease-in-out infinite;
        }
        @keyframes pulseGlow {
            0%, 100% { box-shadow: 0 0 12px rgba(56, 189, 248, 0.4); }
            50% { box-shadow: 0 0 24px rgba(56, 189, 248, 0.85); }
        }
        .portal-ready-badge {
            animation: pulseGlow 1.8s infinite;
        }
    </style>
</head>
<body class="relative text-slate-100">

    <!-- Main Game Canvas -->
    <canvas id="gameCanvas"></canvas>

    <!-- Top-Left HUD: Nyawa (Health), Tenaga (Stamina) & Status -->
    <div id="hudTopLeft" class="fixed top-3 left-3 z-20 flex flex-col gap-2 pointer-events-none">
        <div class="glass-panel rounded-2xl px-3.5 py-2.5 shadow-xl flex flex-col gap-1.5 min-w-[195px]">
            <div class="flex items-center justify-between gap-2">
                <span class="text-xs font-extrabold uppercase tracking-wider text-rose-400 flex items-center gap-1">
                    ❤️ Nyawa
                </span>
                <span id="hpText" class="text-xs font-bold text-slate-300">100/100</span>
            </div>
            <div id="heartsContainer" class="flex items-center gap-1"></div>

            <div class="flex items-center justify-between gap-2 mt-0.5">
                <span class="text-[11px] font-bold uppercase tracking-wider text-amber-300">⚡ Tenaga</span>
                <span id="depthText" class="text-[11px] font-semibold text-sky-300">Arena Pertempuran</span>
            </div>
            <div class="w-full h-2 bg-slate-800 rounded-full overflow-hidden border border-slate-700">
                <div id="staminaBar" class="h-full bg-gradient-to-r from-amber-500 to-yellow-300 transition-all duration-150" style="width: 100%;"></div>
            </div>
        </div>

        <div class="glass-panel rounded-xl px-3 py-1.5 shadow-lg flex items-center justify-between gap-3 text-xs font-bold">
            <span id="levelBadge" class="text-emerald-300">🚩 Level 1 / 10</span>
            <span id="scoreBadge" class="text-amber-300">⭐ 0</span>
        </div>
    </div>

    <!-- Top-Center HUD: Bar Kemajuan Objektif Level -->
    <div id="hudObjective" class="fixed top-3 left-1/2 -translate-x-1/2 z-20 pointer-events-none w-[92%] max-w-md hidden sm:block">
        <div id="objectivePanel" class="glass-panel rounded-2xl px-4 py-2.5 shadow-xl border-amber-400/35">
            <div class="flex items-center justify-between gap-2 text-xs">
                <span id="levelTitleText" class="font-extrabold text-amber-300 truncate">Level 1: Raja Lendir Hijau</span>
                <span id="objectiveProgressText" class="font-extrabold text-sky-300 shrink-0">0 / 1 Raja Lendir</span>
            </div>
            <p id="objectiveDescText" class="text-[11px] text-slate-200 mt-0.5 truncate">Tewaskan Boss Raja Lendir Hijau untuk membuka Portal Cahaya!</p>
            <div class="w-full h-2.5 bg-slate-800/90 rounded-full overflow-hidden border border-slate-700 mt-1.5">
                <div id="objectiveBar" class="h-full bg-gradient-to-r from-emerald-500 via-teal-400 to-sky-400 transition-all duration-200" style="width: 0%;"></div>
            </div>
        </div>
    </div>

    <!-- Mobile Objective Bar -->
    <div id="hudObjectiveMobile" class="fixed top-28 left-3 right-3 z-20 pointer-events-none sm:hidden">
        <div class="glass-panel rounded-xl px-3 py-1.5 shadow-lg border-amber-400/30 flex items-center justify-between gap-2 text-[11px]">
            <span id="objectiveDescMobile" class="font-bold text-slate-100 truncate">Boss Level:</span>
            <span id="objectiveProgressMobile" class="font-extrabold text-amber-300 shrink-0">0/1</span>
        </div>
    </div>

    <!-- Top-Right HUD: Action Buttons -->
    <div class="fixed top-3 right-3 z-20 flex items-center gap-1.5 sm:gap-2">
        <button id="btnPosterRef" title="Nota Poster Azan & Iqamat" class="glass-panel hover:bg-emerald-950/80 active:scale-95 transition px-3 py-2.5 rounded-xl font-bold text-xs sm:text-sm flex items-center gap-1.5 shadow-lg border-emerald-500/40 text-emerald-300 cursor-pointer">
            <span>🕌</span>
            <span class="hidden lg:inline">Nota Poster</span>
        </button>

        <button id="btnCraftToggle" title="Menu Tukangan (E / C)" class="glass-panel hover:bg-slate-800/90 active:scale-95 transition px-3 py-2.5 rounded-xl font-bold text-xs sm:text-sm flex items-center gap-1.5 shadow-lg border-amber-500/40 text-amber-300 cursor-pointer">
            <span>🧪</span>
            <span class="hidden lg:inline">Ramuan</span>
            <kbd class="hidden xl:inline-block text-[10px] bg-slate-800 px-1.5 py-0.5 rounded border border-slate-600 text-slate-300">E</kbd>
        </button>

        <button id="btnHelp" title="Bantuan & Kawalan" class="glass-panel hover:bg-slate-800/90 active:scale-95 transition p-2.5 rounded-xl font-bold text-sm shadow-lg text-sky-300 cursor-pointer">
            ❓
        </button>

        <button id="btnAudio" title="Bunyi Hidup/Mati" class="glass-panel hover:bg-slate-800/90 active:scale-95 transition p-2.5 rounded-xl font-bold text-sm shadow-lg text-emerald-300 cursor-pointer">
            🔊
        </button>

        <button id="btnRestartLevel" title="Mula Semula Level Ini" class="glass-panel hover:bg-rose-950/80 active:scale-95 transition px-2.5 py-2.5 rounded-xl font-bold text-xs sm:text-sm shadow-lg text-rose-300 border-rose-500/30 cursor-pointer">
            🔄 <span class="hidden lg:inline">Ulang Level</span>
        </button>

        <button id="btnMainMenu" title="Kembali ke Menu Utama" class="glass-panel hover:bg-indigo-950/80 active:scale-95 transition px-2.5 py-2.5 rounded-xl font-bold text-xs sm:text-sm shadow-lg text-indigo-300 border-indigo-500/30 cursor-pointer">
            🏠 <span class="hidden lg:inline">Menu Utama</span>
        </button>
    </div>

    <!-- Selected Item Tooltip above Hotbar -->
    <div id="selectedItemLabel" class="fixed bottom-24 left-1/2 -translate-x-1/2 z-20 pointer-events-none px-3.5 py-1 rounded-full bg-slate-900/80 border border-slate-700/70 text-xs font-bold text-amber-200 shadow-md tracking-wide">
        Pedang Kayu
    </div>

    <!-- Bottom Hotbar (Slots 1-6) -->
    <div id="hotbarContainer" class="fixed bottom-3 left-1/2 -translate-x-1/2 z-20 glass-panel p-2 rounded-2xl shadow-2xl flex items-center gap-1.5 sm:gap-2"></div>

    <!-- Mobile Touch Controls -->
    <div id="mobileControls" class="fixed bottom-24 left-3 right-3 z-20 flex justify-between items-end pointer-events-none md:hidden">
        <div class="flex gap-2 pointer-events-auto">
            <button id="touchLeft" class="w-12 h-12 rounded-2xl glass-panel active:bg-amber-500/40 flex items-center justify-center text-xl font-bold shadow-lg">◀</button>
            <button id="touchRight" class="w-12 h-12 rounded-2xl glass-panel active:bg-amber-500/40 flex items-center justify-center text-xl font-bold shadow-lg">▶</button>
        </div>
        <div class="flex gap-2 pointer-events-auto">
            <button id="touchPotionBtn" class="px-3 h-12 rounded-2xl glass-panel text-xs font-bold text-rose-300 flex items-center justify-center shadow-lg">🧪 Nyawa</button>
            <button id="touchAttackBtn" class="px-3.5 h-12 rounded-2xl glass-panel active:bg-amber-500/40 text-xs font-bold text-amber-300 flex items-center justify-center shadow-lg">⚔️ Serang</button>
            <button id="touchJump" class="w-14 h-12 rounded-2xl glass-panel active:bg-emerald-500/40 flex items-center justify-center text-lg font-bold shadow-lg">▲</button>
        </div>
    </div>

    <!-- Modal Nota Ringkas Poster 'Maksud Azan dan Iqamat' -->
    <div id="posterModal" class="fixed inset-0 z-40 hidden items-center justify-center bg-slate-950/75 backdrop-blur-sm p-4">
        <div class="glass-panel w-full max-w-2xl rounded-3xl p-5 sm:p-6 shadow-2xl border-2 border-emerald-400/50 flex flex-col max-h-[88vh]">
            <div class="flex items-center justify-between pb-3 border-b border-slate-700">
                <div class="flex items-center gap-2.5">
                    <span class="text-2xl">🕌</span>
                    <div>
                        <h2 class="text-lg sm:text-xl font-extrabold text-emerald-300">Nota Poster: Maksud Azan dan Iqamat</h2>
                        <p class="text-xs text-slate-400">Rujukan pantas untuk cabaran kuiz setiap akhir level</p>
                    </div>
                </div>
                <button id="btnClosePoster" class="w-9 h-9 rounded-xl bg-slate-800 hover:bg-rose-600 text-slate-300 hover:text-white font-bold flex items-center justify-center transition cursor-pointer">✕</button>
            </div>

            <div class="mt-4 overflow-y-auto pr-1 space-y-3 text-xs sm:text-sm">
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div class="bg-emerald-950/60 border border-emerald-500/40 rounded-2xl p-3.5">
                        <span class="inline-block px-2.5 py-0.5 rounded-full bg-emerald-600 text-white font-extrabold text-xs mb-1.5">AZAN</span>
                        <p class="font-bold text-amber-300 text-xs mb-1">Lafaz Permulaan: "Allahu Akbar, Allahu Akbar..."</p>
                        <p class="text-slate-200 leading-relaxed"><strong>Azan</strong> ialah seruan untuk memberitahu umat Islam bahawa <strong>waktu solat telah masuk</strong>. Azan juga mengajak umat Islam menunaikan solat.</p>
                    </div>
                    <div class="bg-pink-950/50 border border-pink-500/40 rounded-2xl p-3.5">
                        <span class="inline-block px-2.5 py-0.5 rounded-full bg-pink-600 text-white font-extrabold text-xs mb-1.5">IQAMAT</span>
                        <p class="font-bold text-amber-300 text-xs mb-1">Lafaz: "Qad Qamat as-Solah, Qad Qamat as-Solah..."</p>
                        <p class="text-slate-200 leading-relaxed"><strong>Iqamat</strong> ialah seruan yang menandakan <strong>solat berjemaah akan dimulakan tidak lama lagi</strong>.</p>
                    </div>
                </div>

                <div class="bg-slate-800/80 border border-slate-700 rounded-2xl p-3.5">
                    <h3 class="font-extrabold text-amber-300 mb-2">📌 Perbezaan Asas</h3>
                    <ul class="space-y-1.5 text-slate-200 text-xs">
                        <li><strong class="text-emerald-400">1. Waktu Azan:</strong> Azan dilaungkan <strong>apabila masuk waktu solat</strong> (tanda waktu solat telah masuk).</li>
                        <li><strong class="text-pink-400">2. Waktu Iqamat:</strong> Iqamat dilaungkan <strong>sebelum solat berjemaah dimulakan</strong> (tanda solat akan bermula tidak lama lagi).</li>
                        <li><strong class="text-sky-400">3. Fungsi Utama:</strong> Azan menyeru umat Islam supaya <strong>bersedia menunaikan solat</strong>, manakala iqamat menandakan <strong>solat akan bermula</strong>.</li>
                    </ul>
                </div>

                <div class="flex flex-wrap items-center justify-between gap-2 bg-amber-500/15 border border-amber-400/35 rounded-2xl px-4 py-2.5 text-xs">
                    <span class="text-amber-200 font-bold">👣 3 Langkah: <strong class="text-white">Dengar, Bersedia, Solat!</strong></span>
                    <span class="text-emerald-300 font-bold">💖 Slogan: <strong class="text-white">Solat Membina Insan yang Lebih Baik</strong></span>
                </div>
            </div>

            <button id="btnGotItPoster" class="mt-4 w-full py-2.5 rounded-2xl bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-400 hover:to-teal-500 font-bold text-slate-950 shadow-lg transition cursor-pointer">
                Tutup Nota & Sambung Misi
            </button>
        </div>
    </div>

    <!-- Cabaran Akhir Level (Modal Kuiz Poster) -->
    <div id="quizModal" class="fixed inset-0 z-50 hidden items-center justify-center bg-slate-950/80 backdrop-blur-md p-4">
        <div class="glass-panel w-full max-w-xl rounded-3xl p-5 sm:p-7 shadow-2xl border-2 border-sky-400/50 flex flex-col max-h-[92vh] overflow-y-auto">
            <div class="flex items-center justify-between gap-2">
                <span id="quizLevelTag" class="px-3 py-1 rounded-full bg-sky-500/20 border border-sky-400/40 text-sky-300 text-xs font-extrabold uppercase tracking-wider">
                    🌀 Portal Level 1 Dibuka!
                </span>
                <span class="text-xs font-bold text-amber-300">Cabaran Poster Azan & Iqamat</span>
            </div>

            <h2 id="quizQuestionText" class="mt-4 text-base sm:text-lg font-extrabold text-white leading-snug">
                Apakah maksud azan berdasarkan gambar rajah tersebut?
            </h2>

            <div id="quizOptionsContainer" class="mt-4 space-y-2.5"></div>

            <div id="quizFeedbackBox" class="hidden mt-4 p-3.5 rounded-2xl border text-xs sm:text-sm leading-relaxed">
                <p id="quizFeedbackTitle" class="font-extrabold mb-1"></p>
                <p id="quizFeedbackDesc" class="text-slate-200"></p>
            </div>

            <div class="mt-5 flex items-center justify-end gap-2.5">
                <button id="btnQuizRetry" class="hidden px-4 py-2.5 rounded-xl bg-amber-500/20 hover:bg-amber-500/30 border border-amber-400/50 text-amber-300 font-bold text-xs sm:text-sm transition cursor-pointer">
                    🔄 Cuba Jawab Semula
                </button>
                <button id="btnQuizNextLevel" class="hidden w-full sm:w-auto px-6 py-3 rounded-2xl bg-gradient-to-r from-emerald-500 via-teal-500 to-sky-500 hover:from-emerald-400 hover:to-sky-400 text-slate-950 font-extrabold text-sm shadow-xl transition cursor-pointer">
                    Teruskan ke Level Seterusnya ➡️
                </button>
            </div>
        </div>
    </div>

    <!-- Crafting Modal (Menu Ramuan & Naik Taraf) -->
    <div id="craftingModal" class="fixed inset-0 z-30 hidden items-center justify-center bg-slate-950/65 backdrop-blur-sm p-4">
        <div class="glass-panel w-full max-w-2xl rounded-3xl p-5 sm:p-6 shadow-2xl border-2 border-amber-500/40 flex flex-col max-h-[85vh]">
            <div class="flex items-center justify-between pb-3 border-b border-slate-700/80">
                <div class="flex items-center gap-2.5">
                    <span class="text-2xl">🧪</span>
                    <div>
                        <h2 class="text-lg sm:text-xl font-extrabold text-amber-300 tracking-wide">Menu Ramuan & Tukangan</h2>
                        <p class="text-xs text-slate-400">Gunakan Gel Lendir yang dikumpul untuk membuat Ramuan Nyawa</p>
                    </div>
                </div>
                <button id="btnCloseCraft" class="w-9 h-9 rounded-xl bg-slate-800 hover:bg-rose-600 text-slate-300 hover:text-white font-bold flex items-center justify-center transition cursor-pointer">✕</button>
            </div>

            <div id="recipeList" class="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-3 overflow-y-auto pr-1"></div>
        </div>
    </div>

    <!-- Help / Controls Modal -->
    <div id="helpModal" class="fixed inset-0 z-40 hidden items-center justify-center bg-slate-950/70 backdrop-blur-sm p-4">
        <div class="glass-panel w-full max-w-lg rounded-3xl p-6 shadow-2xl border-2 border-sky-500/40">
            <div class="flex items-center justify-between pb-3 border-b border-slate-700">
                <h2 class="text-lg font-extrabold text-sky-300 flex items-center gap-2">🧭 Panduan & Kawalan</h2>
                <button id="btnCloseHelp" class="w-8 h-8 rounded-xl bg-slate-800 hover:bg-rose-600 text-slate-300 hover:text-white font-bold flex items-center justify-center transition cursor-pointer">✕</button>
            </div>
            <div class="mt-4 space-y-3 text-sm text-slate-200">
                <div class="flex items-start gap-3 bg-slate-800/60 p-3 rounded-2xl">
                    <span class="text-xl">👹</span>
                    <div>
                        <p class="font-bold text-emerald-300">10 Boss Berbeza & Portal Cahaya</p>
                        <p class="text-xs text-slate-300 mt-0.5">Setiap level mempunyai <strong>Boss unik</strong> (Raja Lendir, Serigala, Kelawar, Rangka, Labah-labah, Golem, Roh Bayang, Kesatria Magma, Naga Api & Maharaja Kegelapan). Tewaskan Boss untuk membuka <strong>Portal Cahaya</strong> dan menjawab kuiz poster!</p>
                    </div>
                </div>
                <div class="flex items-start gap-3 bg-slate-800/60 p-3 rounded-2xl">
                    <span class="text-xl">🏃</span>
                    <div>
                        <p class="font-bold text-amber-300">Pergerakan & Lompat</p>
                        <p class="text-xs text-slate-300 mt-0.5">Kekunci <kbd class="px-1.5 py-0.5 bg-slate-700 rounded">A</kbd> / <kbd class="px-1.5 py-0.5 bg-slate-700 rounded">D</kbd> atau <kbd class="px-1.5 py-0.5 bg-slate-700 rounded">◀</kbd> <kbd class="px-1.5 py-0.5 bg-slate-700 rounded">▶</kbd> untuk berjalan, <kbd class="px-1.5 py-0.5 bg-slate-700 rounded">W</kbd> / <kbd class="px-1.5 py-0.5 bg-slate-700 rounded">Space</kbd> untuk melompat.</p>
                    </div>
                </div>
                <div class="flex items-start gap-3 bg-slate-800/60 p-3 rounded-2xl">
                    <span class="text-xl">⚔️</span>
                    <div>
                        <p class="font-bold text-sky-300">Menyerang & Ramuan Nyawa</p>
                        <p class="text-xs text-slate-300 mt-0.5"><strong>Klik Kiri</strong> untuk melibas pedang ke arah Boss. <strong>Klik Kanan</strong> atau klik slot <strong>Ramuan Nyawa</strong> untuk memulihkan nyawa.</p>
                    </div>
                </div>
            </div>
            <button id="btnStartFromHelp" class="mt-5 w-full py-3 rounded-2xl bg-gradient-to-r from-sky-500 to-indigo-600 hover:from-sky-400 hover:to-indigo-500 font-bold text-white shadow-lg transition cursor-pointer">
                Faham, Teruskan!
            </button>
        </div>
    </div>

    <!-- Title Screen Overlay (Menu Utama) -->
    <div id="titleOverlay" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-950/55 backdrop-blur-[3px] p-4">
        <div class="glass-panel max-w-lg w-full rounded-3xl p-7 text-center shadow-2xl border-2 border-emerald-400/45 animate-float">
            <div class="inline-block px-3 py-1 rounded-full bg-emerald-500/20 border border-emerald-400/40 text-emerald-300 text-xs font-bold tracking-wider uppercase mb-3">
                10 Level Boss Unik • Kuiz Poster Pendidikan Islam
            </div>
            <h1 class="pixel-font text-xl sm:text-2xl text-transparent bg-clip-text bg-gradient-to-b from-amber-200 via-amber-400 to-orange-500 leading-snug drop-shadow">
                TERRANUSANTARA
            </h1>
            <p class="text-base sm:text-lg font-extrabold text-emerald-300 mt-1">Misi Azan & Iqamat</p>
            <p class="text-xs sm:text-sm text-slate-200 mt-3 leading-relaxed">
                Tewaskan <strong>10 Boss berbeza</strong> dari Level 1 hingga Level 10, buka <strong>Portal Cahaya</strong> pada penghujung setiap level, dan jawab soalan daripada poster <em>Maksud Azan dan Iqamat</em>!
            </p>
            <div class="mt-6 flex flex-col sm:flex-row gap-2.5">
                <button id="btnPlayGame" class="flex-1 py-3.5 px-6 rounded-2xl bg-gradient-to-r from-emerald-500 via-teal-500 to-cyan-600 hover:from-emerald-400 hover:to-cyan-500 text-slate-950 font-extrabold text-base shadow-xl shadow-emerald-500/25 active:scale-95 transition cursor-pointer flex items-center justify-center gap-2">
                    <span>⚔️</span>
                    <span>Mula Misi Level 1</span>
                </button>
                <button id="btnTitlePoster" class="py-3.5 px-4 rounded-2xl bg-slate-800/90 hover:bg-slate-700 border border-emerald-400/40 text-emerald-300 font-bold text-sm transition cursor-pointer flex items-center justify-center gap-1.5">
                    <span>🕌</span>
                    <span>Baca Poster</span>
                </button>
            </div>
            <div class="mt-4 flex justify-center gap-3 text-[11px] text-slate-400">
                <span>WASD: Gerak</span>
                <span>•</span>
                <span>Klik Kiri: Serang Pedang</span>
                <span>•</span>
                <span>Klik Kanan: Guna Ramuan</span>
            </div>
        </div>
    </div>

    <!-- Victory Screen (Tamat 10 Level - Sijil Wira Solat) -->
    <div id="victoryOverlay" class="fixed inset-0 z-50 hidden items-center justify-center bg-slate-950/85 backdrop-blur-md p-4">
        <div class="glass-panel max-w-md w-full rounded-3xl p-7 text-center shadow-2xl border-2 border-amber-400/60">
            <div class="text-5xl mb-2">🏆</div>
            <div class="inline-block px-3 py-1 rounded-full bg-amber-500/20 border border-amber-400/40 text-amber-300 text-[11px] font-extrabold uppercase tracking-wider mb-1">
                Sijil Penghargaan Wira Solat
            </div>
            <div id="victoryStars" class="text-3xl tracking-widest my-2">⭐⭐⭐</div>
            <h2 class="pixel-font text-lg sm:text-xl text-amber-300 leading-relaxed">TAHNIAH, WIRA SOLAT!</h2>
            <p class="text-xs sm:text-sm text-slate-200 mt-2 leading-relaxed">
                Anda telah menewaskan kesemua <strong>10 Boss</strong> dan menguasai ilmu <em>Maksud Azan dan Iqamat</em>!
            </p>
            <div class="mt-4 bg-slate-900/90 rounded-2xl p-4 border border-slate-700/80 grid grid-cols-2 gap-3 text-xs">
                <div>
                    <span class="block text-slate-400">Jumlah Markah</span>
                    <strong id="victoryScore" class="text-xl text-amber-400">0</strong>
                </div>
                <div>
                    <span class="block text-slate-400">Jawapan Tepat Kali Pertama</span>
                    <strong id="victoryFirstTry" class="text-xl text-emerald-400">10 / 10</strong>
                </div>
            </div>
            <p class="mt-3 text-xs italic text-emerald-300">"Solat Membina Insan yang Lebih Baik — Dengar, Bersedia, Solat!"</p>
            <div class="mt-5 flex flex-col sm:flex-row gap-2.5">
                <button id="btnPlayAgainAll" class="flex-1 py-3.5 rounded-2xl bg-gradient-to-r from-amber-400 via-orange-500 to-rose-500 hover:from-amber-300 hover:to-rose-400 text-slate-950 font-extrabold text-sm shadow-xl transition cursor-pointer">
                    🔄 Main Semula
                </button>
                <button id="btnVictoryMainMenu" class="py-3.5 px-5 rounded-2xl bg-slate-800/90 hover:bg-slate-700 border border-slate-600 text-slate-200 font-bold text-sm transition cursor-pointer">
                    🏠 Menu Utama
                </button>
            </div>
        </div>
    </div>

    <!-- Game Over / Respawn Overlay -->
    <div id="gameOverOverlay" class="fixed inset-0 z-50 hidden items-center justify-center bg-rose-950/75 backdrop-blur-md p-4">
        <div class="glass-panel max-w-sm w-full rounded-3xl p-7 text-center shadow-2xl border-2 border-rose-500/50">
            <div class="text-5xl mb-3">💀</div>
            <h2 class="pixel-font text-xl text-rose-400 leading-relaxed">ANDA TELAH TEWAS!</h2>
            <p class="text-xs text-slate-300 mt-2">Jangan berputus asa! Bangkit semula untuk meneruskan misi level ini.</p>
            <div class="mt-4 bg-slate-900/80 rounded-2xl p-3 text-xs text-slate-300 flex justify-around">
                <div>
                    <span class="block text-slate-400">Level Semasa</span>
                    <strong id="failLevelText" class="text-base text-sky-400">Level 1</strong>
                </div>
                <div>
                    <span class="block text-slate-400">Skor Semasa</span>
                    <strong id="failScoreText" class="text-base text-amber-400">0</strong>
                </div>
            </div>
            <div class="mt-5 flex flex-col gap-2.5">
                <button id="btnRespawn" class="w-full py-3 rounded-2xl bg-gradient-to-r from-rose-500 to-orange-500 hover:from-rose-400 hover:to-orange-400 font-extrabold text-white shadow-lg transition cursor-pointer">
                    🔄 Bangkit Semula (Ulang Level)
                </button>
                <button id="btnGameOverMainMenu" class="w-full py-2.5 rounded-2xl bg-slate-800/90 hover:bg-slate-700 border border-slate-600 text-slate-200 font-bold text-xs sm:text-sm transition cursor-pointer">
                    🏠 Kembali ke Menu Utama
                </button>
            </div>
        </div>
    </div>

    <!-- Toast Notification -->
    <div id="toastNotification" class="fixed top-24 left-1/2 -translate-x-1/2 z-40 pointer-events-none opacity-0 transition-opacity duration-300 px-4 py-2 rounded-full bg-slate-900/90 border border-amber-400/50 text-xs font-bold text-amber-200 shadow-xl">
        Notifikasi
    </div>

    <script>
        /**
         * ============================================================================
         * TERRANUSANTARA: MISI AZAN & IQAMAT (10 LEVEL BOSS UNIK + KUIZ POSTER)
         * Enjin Permainan Pengembaraan 2D (Terraria-inspired)
         * ============================================================================
         */

        // --- 1. SISTEM AUDIO PROSEDURAL (WEB AUDIO API) ---
        class SoundEngine {
            constructor() {
                this.ctx = null;
                this.muted = false;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    if (AudioCtx) {
                        this.ctx = new AudioCtx();
                    }
                }
                if (this.ctx && this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            toggleMute() {
                this.muted = !this.muted;
                return this.muted;
            }

            playTone(freq, endFreq, duration, type = 'sine', vol = 0.12) {
                if (this.muted) return;
                this.init();
                if (!this.ctx) return;

                try {
                    const now = this.ctx.currentTime;
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = type;
                    osc.frequency.setValueAtTime(freq, now);
                    if (endFreq !== freq) {
                        osc.frequency.exponentialRampToValueAtTime(Math.max(20, endFreq), now + duration);
                    }
                    gain.gain.setValueAtTime(vol, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + duration);

                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now);
                    osc.stop(now + duration);
                } catch (e) {}
            }

            playJump() {
                this.playTone(180, 420, 0.16, 'sine', 0.1);
            }

            playSwing() {
                this.playTone(340, 110, 0.12, 'triangle', 0.09);
            }

            playHitEnemy() {
                this.playTone(150, 60, 0.15, 'sawtooth', 0.15);
            }

            playPlayerHurt() {
                this.playTone(190, 55, 0.28, 'sawtooth', 0.18);
            }

            playCraft() {
                this.playTone(523.25, 659.25, 0.1, 'triangle', 0.14);
                setTimeout(() => this.playTone(659.25, 783.99, 0.18, 'triangle', 0.14), 90);
            }

            playPickup() {
                this.playTone(600, 900, 0.07, 'sine', 0.08);
            }

            playPortalOpen() {
                this.playTone(440, 660, 0.15, 'sine', 0.14);
                setTimeout(() => this.playTone(660, 880, 0.25, 'triangle', 0.15), 120);
            }

            playCorrect() {
                this.playTone(523.25, 523.25, 0.1, 'triangle', 0.16);
                setTimeout(() => this.playTone(659.25, 659.25, 0.1, 'triangle', 0.16), 95);
                setTimeout(() => this.playTone(783.99, 1046.5, 0.25, 'triangle', 0.18), 190);
            }

            playWrong() {
                this.playTone(210, 140, 0.22, 'sawtooth', 0.14);
            }
        }

        const audio = new SoundEngine();

        // --- 2. DEFINISI 10 LEVEL BOSS UNIK & 10 SOALAN POSTER ---
        const LEVELS = [
            {
                level: 1,
                title: "Level 1: Raja Lendir Hijau",
                desc: "Tewaskan Boss Raja Lendir Hijau untuk membuka Portal Cahaya!",
                biome: "grassland",
                timeOfDay: 0.25,
                bossConfig: { count: 1, types: ['slime_king'] },
                objective: { type: "kills", target: 1, label: "Raja Lendir" },
                question: {
                    text: "Apakah maksud azan berdasarkan gambar rajah tersebut?",
                    options: [
                        "Seruan untuk memberitahu umat Islam bahawa waktu solat telah masuk",
                        "Seruan yang menandakan solat berjemaah telah selesai ditunaikan",
                        "Bacaan doa selepas keluar dari masjid atau surau",
                        "Seruan untuk mengajak orang ramai berbuka puasa sahaja"
                    ],
                    correctIndex: 0,
                    explanation: "Tepat! Berdasarkan poster, Azan ialah seruan untuk memberitahu umat Islam bahawa waktu solat telah masuk serta mengajak umat Islam menunaikan solat."
                }
            },
            {
                level: 2,
                title: "Level 2: Serigala Rimba",
                desc: "Tewaskan Boss Serigala Rimba yang pantas berlari & menerkam!",
                biome: "grassland",
                timeOfDay: 0.26,
                bossConfig: { count: 2, types: ['wolf'] },
                objective: { type: "kills", target: 2, label: "Serigala Rimba" },
                question: {
                    text: "Apakah tujuan iqamat dilaungkan?",
                    options: [
                        "Memberitahu bahawa waktu solat subuh telah tamat",
                        "Menandakan solat berjemaah akan dimulakan tidak lama lagi",
                        "Mengajak penduduk kampung berkumpul untuk gotong-royong",
                        "Menandakan masuknya waktu azan zohor"
                    ],
                    correctIndex: 1,
                    explanation: "Syabas! Iqamat ialah seruan yang menandakan solat berjemaah akan dimulakan tidak lama lagi."
                }
            },
            {
                level: 3,
                title: "Level 3: Kelawar Gua Raksasa",
                desc: "Tewaskan Boss Kelawar Gua yang terbang menyambar dari udara!",
                biome: "night_forest",
                timeOfDay: 0.42,
                bossConfig: { count: 2, types: ['bat'] },
                objective: { type: "kills", target: 2, label: "Kelawar Gua" },
                question: {
                    text: "Apakah lafaz permulaan bagi azan seperti yang ditunjukkan dalam gambar rajah?",
                    options: [
                        "Subhanallah, Alhamdulillah",
                        "Qad Qamat as-Solah, Qad Qamat as-Solah",
                        "Allahu Akbar, Allahu Akbar",
                        "Assalamualaikum Warahmatullah"
                    ],
                    correctIndex: 2,
                    explanation: "Betul! Watak muazin pada bahagian Azan melaungkan permulaan 'Allahu Akbar, Allahu Akbar'."
                }
            },
            {
                level: 4,
                title: "Level 4: Pahlawan Rangka",
                desc: "Gunakan Pedang Besi untuk menewaskan Boss Pahlawan Rangka!",
                biome: "grassland",
                timeOfDay: 0.30,
                bossConfig: { count: 2, types: ['skeleton'] },
                objective: { type: "kills", target: 2, label: "Pahlawan Rangka" },
                question: {
                    text: "Antara berikut, yang manakah lafaz yang terdapat dalam iqamat berdasarkan gambar?",
                    options: [
                        "Allahu Akbar, Allahu Akbar",
                        "Qad Qamat as-Solah, Qad Qamat as-Solah",
                        "Alhamdulillahi Rabbil Alamin",
                        "Bismillahirrahmanirrahim"
                    ],
                    correctIndex: 1,
                    explanation: "Cemerlang! Lafaz khusus yang ditunjukkan dalam kotak Iqamat ialah 'Qad Qamat as-Solah, Qad Qamat as-Solah'."
                }
            },
            {
                level: 5,
                title: "Level 5: Labah-labah Kristal",
                desc: "Tewaskan Boss Labah-labah Kristal yang melompat tinggi!",
                biome: "crystal_abyss",
                timeOfDay: 0.25,
                bossConfig: { count: 2, types: ['spider'] },
                objective: { type: "kills", target: 2, label: "Labah-labah Kristal" },
                question: {
                    text: "Bilakah waktu yang tepat untuk melaungkan azan?",
                    options: [
                        "Selepas selesai menunaikan solat fardu",
                        "Apabila masuk waktu solat",
                        "Tepat pada tengah malam setiap hari",
                        "Semasa imam sedang membaca khutbah"
                    ],
                    correctIndex: 1,
                    explanation: "Tepat sekali! Poin 1 Perbezaan Asas menyatakan Azan dilaungkan apabila masuk waktu solat."
                }
            },
            {
                level: 6,
                title: "Level 6: Golem Batu Purba",
                desc: "Tewaskan Boss Golem Batu Purba yang bertubuh besar dan kebal!",
                biome: "grassland",
                timeOfDay: 0.25,
                bossConfig: { count: 1, types: ['golem'] },
                objective: { type: "kills", target: 1, label: "Golem Batu" },
                question: {
                    text: "Bilakah iqamat dilaungkan?",
                    options: [
                        "Sebelum solat berjemaah dimulakan",
                        "Satu jam sebelum masuk waktu solat",
                        "Selepas memberi salam di akhir solat",
                        "Ketika matahari mula terbit"
                    ],
                    correctIndex: 0,
                    explanation: "Betul! Poin 2 Perbezaan Asas menerangkan bahawa Iqamat dilaungkan sebelum solat berjemaah dimulakan."
                }
            },
            {
                level: 7,
                title: "Level 7: Roh Bayang Malam",
                desc: "Tewaskan Boss Roh Bayang yang melayang pantas di udara!",
                biome: "night_forest",
                timeOfDay: 0.48,
                bossConfig: { count: 2, types: ['ghost'] },
                objective: { type: "kills", target: 2, label: "Roh Bayang" },
                question: {
                    text: "Apakah perbezaan utama fungsi antara azan dan iqamat berdasarkan poin ke-3 dalam gambar rajah?",
                    options: [
                        "Azan dilaungkan di dalam rumah, manakala iqamat dilaungkan di padang",
                        "Azan menyeru supaya bersedia menunaikan solat, manakala iqamat menandakan solat akan bermula",
                        "Azan lebih pendek lafaznya berbanding iqamat",
                        "Tiada sebarang perbezaan fungsi antara azan dan iqamat"
                    ],
                    correctIndex: 1,
                    explanation: "Hebat! Berdasarkan poin ke-3, Azan menyeru umat Islam supaya bersedia menunaikan solat, manakala iqamat menandakan solat akan bermula."
                }
            },
            {
                level: 8,
                title: "Level 8: Kesatria Magma",
                desc: "Gunakan Pedang Kristal untuk menewaskan Boss Kesatria Magma!",
                biome: "magma_core",
                timeOfDay: 0.30,
                bossConfig: { count: 2, types: ['magma_knight'] },
                objective: { type: "kills", target: 2, label: "Kesatria Magma" },
                question: {
                    text: "Ahmad mendengar seruan iqamat di surau berhampiran rumahnya. Apakah tindakan yang paling sesuai dilakukan oleh Ahmad pada masa itu?",
                    options: [
                        "Menyambung permainan videonya sehingga tamat",
                        "Segera berdiri dan meluruskan saf untuk memulakan solat berjemaah",
                        "Baru mula bersiap-siap untuk mandi di rumah",
                        "Tidur semula kerana waktu solat masih lama"
                    ],
                    correctIndex: 1,
                    explanation: "Bijak! Iqamat menandakan solat berjemaah akan bermula tidak lama lagi, jadi jemaah hendaklah segera berdiri dan meluruskan saf."
                }
            },
            {
                level: 9,
                title: "Level 9: Naga Api Merah",
                desc: "Tewaskan Boss Naga Api yang terbang di ruang angkasa!",
                biome: "magma_core",
                timeOfDay: 0.35,
                bossConfig: { count: 1, types: ['dragon'] },
                objective: { type: "kills", target: 1, label: "Naga Api" },
                question: {
                    text: "Apakah slogan yang tertera pada bahagian paling bawah infografik tersebut?",
                    options: [
                        "Solat Membina Insan yang Lebih Baik",
                        "Kebersihan Sebahagian Daripada Iman",
                        "Ilmu Pelita Hidup Kita",
                        "Masa Itu Emas"
                    ],
                    correctIndex: 0,
                    explanation: "Tepat sekali! Pada sepanduk kuning di bahagian paling bawah poster tertera slogan 'Solat Membina Insan yang Lebih Baik'."
                }
            },
            {
                level: 10,
                title: "Level 10: Maharaja Kegelapan",
                desc: "Pertempuran Akhir! Tewaskan Boss Maharaja Kegelapan untuk menamatkan misi!",
                biome: "crystal_abyss",
                timeOfDay: 0.45,
                bossConfig: { count: 1, types: ['dark_lord'] },
                objective: { type: "kills", target: 1, label: "Maharaja Kegelapan" },
                question: {
                    text: "Susunan tiga langkah yang dinyatakan oleh watak kanak-kanak pada bahagian kanan bawah gambar ialah...",
                    options: [
                        "Lihat, Fikir, Jawab!",
                        "Dengar, Bersedia, Solat!",
                        "Bangun, Mandi, Makan!",
                        "Baca, Hafal, Amal!"
                    ],
                    correctIndex: 1,
                    explanation: "Alhamdulillah & Tahniah! Watak kanak-kanak di bahagian kanan bawah poster menyatakan tiga langkah utama: 'Dengar, Bersedia, Solat!'"
                }
            }
        ];

        // --- 3. DEFINISI BLOK & ITEM ---
        const TILE_SIZE = 32;
        const WORLD_WIDTH = 130;
        const WORLD_HEIGHT = 95;

        const TILES = {
            AIR: 0,
            GRASS: 1,
            DIRT: 2,
            STONE: 3,
            WOOD: 4,
            LEAVES: 5,
            PLANK: 6,
            BEDROCK: 14
        };

        const TILE_PROPS = {
            [TILES.AIR]:     { name: 'Udara', solid: false },
            [TILES.GRASS]:   { name: 'Tanah Berumput', solid: true },
            [TILES.DIRT]:    { name: 'Tanah', solid: true },
            [TILES.STONE]:   { name: 'Batu', solid: true },
            [TILES.WOOD]:    { name: 'Kayu Balak', solid: false },
            [TILES.LEAVES]:  { name: 'Dedaun', solid: false },
            [TILES.PLANK]:   { name: 'Papan Kayu', solid: true },
            [TILES.BEDROCK]: { name: 'Batu Teras', solid: true }
        };

        const ITEMS = {
            wood_sword:    { id: 'wood_sword',    name: 'Pedang Kayu',    type: 'weapon', damage: 24, color: '#d97706', icon: '🗡️' },
            iron_sword:    { id: 'iron_sword',    name: 'Pedang Besi',    type: 'weapon', damage: 42, color: '#f1f5f9', icon: '⚔️' },
            crystal_sword: { id: 'crystal_sword', name: 'Pedang Kristal', type: 'weapon', damage: 65, color: '#22d3ee', icon: '🗡️' },
            gel:           { id: 'gel',           name: 'Titisan Tenaga', type: 'material', color: '#4ade80', icon: '💧' },
            potion:        { id: 'potion',        name: 'Ramuan Nyawa (+50)', type: 'consumable', heal: 50, color: '#f43f5e', icon: '🧪' }
        };

        const RECIPES = [
            {
                result: 'potion', count: 1,
                desc: 'Memulihkan 50 mata Nyawa serta-merta',
                ingredients: [{ id: 'gel', count: 1 }]
            }
        ];

        // --- 4. PEMBOLEHUBAH KEADAAN PERMAINAN ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let world = [];
        let lightMap = [];
        let surfaceHeights = [];

        let gameState = {
            started: false,
            gameOver: false,
            quizOpen: false,
            craftingOpen: false,
            helpOpen: false,
            posterOpen: false,
            currentLevelIndex: 0,
            score: 0,
            firstTryCorrectCount: 0,
            quizHadWrongAttempt: false,
            timeOfDay: 0.25,
            daySpeed: 0.0012,
            levelKills: 0,
            objectiveCompleted: false
        };

        const portal = {
            x: -9999,
            y: -9999,
            w: 42,
            h: 60,
            active: false,
            pulse: 0
        };

        const player = {
            x: (WORLD_WIDTH / 2) * TILE_SIZE,
            y: 15 * TILE_SIZE,
            w: 22,
            h: 42,
            vx: 0,
            vy: 0,
            speed: 255,
            jumpForce: -485,
            grounded: false,
            facing: 1,
            hp: 100,
            maxHp: 100,
            stamina: 100,
            maxStamina: 100,
            invulnTimer: 0,
            animTimer: 0,
            swingTimer: 0,
            swingDuration: 0.2,
            highestY: 0,
            inventory: [],
            selectedSlot: 0
        };

        const camera = { x: 0, y: 0 };
        const keys = {};
        const mouse = { x: 0, y: 0, worldX: 0, worldY: 0, leftDown: false, rightDown: false };

        let enemies = [];
        let particles = [];
        let floatingTexts = [];
        let droppedItems = [];
        let clouds = [];
        let stars = [];

        // --- 5. PENJANAAN TEKSTUR PIKSEL PROSEDURAL ---
        const tileTextures = {};

        function generateTileTextures() {
            const createTex = (drawFn) => {
                const c = document.createElement('canvas');
                c.width = TILE_SIZE;
                c.height = TILE_SIZE;
                const g = c.getContext('2d');
                drawFn(g, TILE_SIZE);
                return c;
            };

            tileTextures[TILES.DIRT] = createTex((g, s) => {
                g.fillStyle = '#7c3f1d';
                g.fillRect(0, 0, s, s);
                g.fillStyle = '#633014';
                g.fillRect(4, 6, 6, 4);
                g.fillRect(18, 14, 8, 5);
                g.fillRect(8, 22, 5, 5);
                g.fillStyle = '#924c25';
                g.fillRect(14, 4, 5, 4);
                g.fillRect(22, 24, 6, 4);
            });

            tileTextures[TILES.GRASS] = createTex((g, s) => {
                g.drawImage(tileTextures[TILES.DIRT], 0, 0);
                g.fillStyle = '#22c55e';
                g.fillRect(0, 0, s, 8);
                g.fillStyle = '#16a34a';
                g.fillRect(0, 6, s, 4);
                g.fillRect(2, 9, 4, 4);
                g.fillRect(10, 9, 5, 5);
                g.fillRect(20, 9, 4, 3);
                g.fillRect(27, 9, 4, 5);
                g.fillStyle = '#4ade80';
                g.fillRect(2, 1, 8, 2);
                g.fillRect(16, 2, 10, 2);
            });

            tileTextures[TILES.STONE] = createTex((g, s) => {
                g.fillStyle = '#64748b';
                g.fillRect(0, 0, s, s);
                g.fillStyle = '#475569';
                g.fillRect(2, 4, 12, 8);
                g.fillRect(16, 16, 14, 10);
                g.fillStyle = '#94a3b8';
                g.fillRect(16, 4, 12, 6);
                g.fillRect(4, 18, 10, 8);
            });

            tileTextures[TILES.WOOD] = createTex((g, s) => {
                g.fillStyle = '#78350f';
                g.fillRect(2, 0, s - 4, s);
                g.fillStyle = '#92400e';
                g.fillRect(6, 0, 5, s);
                g.fillRect(17, 0, 6, s);
                g.fillStyle = '#451a03';
                g.fillRect(12, 0, 3, s);
                g.fillRect(25, 0, 2, s);
            });

            tileTextures[TILES.LEAVES] = createTex((g, s) => {
                g.fillStyle = '#15803d';
                g.fillRect(2, 2, s - 4, s - 4);
                g.fillStyle = '#22c55e';
                g.fillRect(4, 4, 10, 10);
                g.fillRect(16, 14, 10, 10);
                g.fillStyle = '#166534';
                g.fillRect(16, 4, 10, 8);
                g.fillRect(4, 18, 10, 8);
            });

            tileTextures[TILES.PLANK] = createTex((g, s) => {
                g.fillStyle = '#d97706';
                g.fillRect(0, 0, s, s);
                g.fillStyle = '#b45309';
                g.fillRect(0, 7, s, 2);
                g.fillRect(0, 15, s, 2);
                g.fillRect(0, 23, s, 2);
                g.fillRect(0, 30, s, 2);
                g.fillStyle = '#f59e0b';
                g.fillRect(2, 2, 12, 3);
                g.fillRect(16, 10, 12, 3);
            });

            tileTextures[TILES.BEDROCK] = createTex((g, s) => {
                g.fillStyle = '#0f172a';
                g.fillRect(0, 0, s, s);
                g.fillStyle = '#1e293b';
                g.fillRect(3, 3, 10, 10);
                g.fillRect(18, 16, 11, 11);
            });
        }

        // --- 6. PENJANAAN ARENA PERTEMPURAN MENGIKUT LEVEL ---
        function loadLevel(levelIndex) {
            gameState.currentLevelIndex = levelIndex;
            const cfg = LEVELS[levelIndex];

            world = Array.from({ length: WORLD_HEIGHT }, () => Array(WORLD_WIDTH).fill(TILES.AIR));
            lightMap = Array.from({ length: WORLD_HEIGHT }, () => Array(WORLD_WIDTH).fill(0));
            surfaceHeights = [];

            const seed = (levelIndex + 1) * 137.5;
            const baseSurface = 24;

            for (let x = 0; x < WORLD_WIDTH; x++) {
                const hill = Math.sin((x + seed) * 0.05) * 1.4;
                const surfY = Math.floor(baseSurface + hill);
                surfaceHeights[x] = surfY;

                for (let y = surfY; y < WORLD_HEIGHT; y++) {
                    if (y >= WORLD_HEIGHT - 2) {
                        world[y][x] = TILES.BEDROCK;
                    } else if (y === surfY) {
                        world[y][x] = cfg.biome === 'magma_core' ? TILES.STONE : TILES.GRASS;
                    } else if (y < surfY + 4) {
                        world[y][x] = TILES.DIRT;
                    } else {
                        world[y][x] = TILES.STONE;
                    }
                }
            }

            // Pokok latar belakang di permukaan
            for (let x = 6; x < WORLD_WIDTH - 6; x++) {
                if (Math.abs(x - Math.floor(WORLD_WIDTH / 2)) < 5) continue;
                if (Math.random() < 0.16) {
                    const sy = surfaceHeights[x];
                    if (world[sy][x] === TILES.GRASS || world[sy][x] === TILES.STONE) {
                        const trunkHeight = 4;
                        for (let ty = 1; ty <= trunkHeight; ty++) {
                            if (sy - ty > 2) world[sy - ty][x] = TILES.WOOD;
                        }
                        const topY = sy - trunkHeight;
                        for (let ly = topY - 2; ly <= topY + 1; ly++) {
                            for (let lx = x - 2; lx <= x + 2; lx++) {
                                if (ly > 1 && lx > 1 && lx < WORLD_WIDTH - 1) {
                                    if (Math.abs(lx - x) === 2 && ly === topY - 2) continue;
                                    if (world[ly][lx] === TILES.AIR) {
                                        world[ly][lx] = TILES.LEAVES;
                                    }
                                }
                            }
                        }
                        x += 4;
                    }
                }
            }

            const spawnX = Math.floor(WORLD_WIDTH / 2);
            const spawnY = surfaceHeights[spawnX] - 3;
            player.x = spawnX * TILE_SIZE + 4;
            player.y = spawnY * TILE_SIZE;
            player.vx = 0;
            player.vy = 0;
            player.hp = player.maxHp;
            player.stamina = player.maxStamina;
            player.highestY = player.y;

            portal.x = -9999;
            portal.y = -9999;
            portal.active = false;
            portal.pulse = 0;

            let startSword = 'wood_sword';
            if (levelIndex >= 7) {
                startSword = 'crystal_sword';
            } else if (levelIndex >= 3) {
                startSword = 'iron_sword';
            }

            player.inventory = [
                { id: startSword, count: 1 },
                { id: 'potion', count: 5 },
                null,
                null,
                null,
                null
            ];
            player.selectedSlot = 0;

            enemies = [];
            particles = [];
            floatingTexts = [];
            droppedItems = [];
            gameState.levelKills = 0;
            gameState.objectiveCompleted = false;
            gameState.timeOfDay = cfg.timeOfDay;
            gameState.gameOver = false;
            gameState.quizOpen = false;

            clouds = Array.from({ length: 14 }, () => ({
                x: Math.random() * WORLD_WIDTH * TILE_SIZE,
                y: (3 + Math.random() * 12) * TILE_SIZE,
                w: 80 + Math.random() * 100,
                speed: 8 + Math.random() * 14
            }));

            stars = Array.from({ length: 85 }, () => ({
                x: Math.random(),
                y: Math.random() * 0.55,
                size: Math.random() > 0.75 ? 2 : 1
            }));

            const totalMonsters = cfg.bossConfig.count;
            for (let i = 0; i < totalMonsters; i++) {
                const dir = i % 2 === 0 ? 1 : -1;
                const distTiles = 6 + Math.floor(i / 2) * 5 + Math.random() * 2;
                const forcedType = cfg.bossConfig.types[i % cfg.bossConfig.types.length];
                spawnEnemy(true, dir, distTiles, forcedType);
            }

            updateLightMap();
            renderHotbar();
            updateHUD();
        }

        // --- 7. KEMAJUAN OBJEKTIF LEVEL & PORTAL ---
        function getObjectiveProgress() {
            const obj = LEVELS[gameState.currentLevelIndex].objective;
            return { current: Math.min(obj.target, gameState.levelKills), target: obj.target, label: obj.label };
        }

        function spawnPortalNearPlayer() {
            const pxTile = Math.max(8, Math.min(WORLD_WIDTH - 9, Math.floor((player.x + player.w / 2) / TILE_SIZE)));
            const pyTile = Math.max(6, Math.min(WORLD_HEIGHT - 6, Math.floor((player.y + player.h) / TILE_SIZE)));

            const dir = pxTile + 6 < WORLD_WIDTH - 6 ? 1 : -1;
            const targetX = pxTile + dir * 6;
            const targetY = pyTile;

            for (let y = targetY - 3; y <= targetY - 1; y++) {
                for (let x = targetX - 2; x <= targetX + 2; x++) {
                    if (y > 1 && y < WORLD_HEIGHT - 2) {
                        world[y][x] = TILES.AIR;
                    }
                }
            }
            for (let x = targetX - 2; x <= targetX + 2; x++) {
                if (targetY < WORLD_HEIGHT - 2) {
                    world[targetY][x] = TILES.PLANK;
                }
            }

            portal.x = targetX * TILE_SIZE - 5;
            portal.y = targetY * TILE_SIZE - portal.h;
            portal.active = true;
            portal.pulse = 0;

            spawnParticles(portal.x + portal.w / 2, portal.y + portal.h / 2, '#38bdf8', 24);
        }

        function checkObjectiveCompletion() {
            if (gameState.objectiveCompleted) return;
            const prog = getObjectiveProgress();
            if (prog.current >= prog.target) {
                gameState.objectiveCompleted = true;
                spawnPortalNearPlayer();
                audio.playPortalOpen();
                showToast('🌀 Tahniah! Portal Cahaya telah muncul berhampiran anda!');
                spawnFloatingText(player.x + player.w / 2, player.y - 26, '🌀 PORTAL MUNCUL!', '#38bdf8');
            }
        }

        function openLevelQuiz() {
            if (gameState.quizOpen) return;
            gameState.quizOpen = true;
            gameState.quizHadWrongAttempt = false;
            mouse.leftDown = false;
            mouse.rightDown = false;

            const cfg = LEVELS[gameState.currentLevelIndex];
            const q = cfg.question;

            document.getElementById('quizLevelTag').textContent = `🌀 Portal Level ${cfg.level} / 10 Dibuka!`;
            document.getElementById('quizQuestionText').textContent = q.text;

            const optsCont = document.getElementById('quizOptionsContainer');
            optsCont.innerHTML = '';

            const feedbackBox = document.getElementById('quizFeedbackBox');
            feedbackBox.classList.add('hidden');
            document.getElementById('btnQuizRetry').classList.add('hidden');
            document.getElementById('btnQuizNextLevel').classList.add('hidden');

            const letters = ['A', 'B', 'C', 'D'];
            q.options.forEach((optText, idx) => {
                const btn = document.createElement('button');
                btn.className = 'quiz-opt-btn w-full text-left p-3.5 rounded-2xl bg-slate-800/85 hover:bg-slate-700/90 border-2 border-slate-700 hover:border-sky-400 transition flex items-start gap-3 cursor-pointer';
                btn.innerHTML = `
                    <span class="w-7 h-7 rounded-lg bg-slate-900 border border-slate-600 flex items-center justify-center font-extrabold text-xs text-sky-300 shrink-0 mt-0.5">${letters[idx]}</span>
                    <span class="text-xs sm:text-sm font-semibold text-slate-100 leading-snug">${optText}</span>
                `;
                btn.addEventListener('click', () => handleQuizAnswer(idx, btn));
                optsCont.appendChild(btn);
            });

            const modal = document.getElementById('quizModal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function handleQuizAnswer(selectedIdx, clickedBtn) {
            const cfg = LEVELS[gameState.currentLevelIndex];
            const q = cfg.question;
            const allBtns = document.querySelectorAll('.quiz-opt-btn');
            const feedbackBox = document.getElementById('quizFeedbackBox');
            const fbTitle = document.getElementById('quizFeedbackTitle');
            const fbDesc = document.getElementById('quizFeedbackDesc');

            allBtns.forEach(b => b.disabled = true);

            if (selectedIdx === q.correctIndex) {
                clickedBtn.classList.remove('border-slate-700', 'bg-slate-800/85');
                clickedBtn.classList.add('border-emerald-400', 'bg-emerald-950/80');

                audio.playCorrect();
                const pointsEarned = gameState.quizHadWrongAttempt ? 100 : 200;
                gameState.score += pointsEarned;
                if (!gameState.quizHadWrongAttempt) {
                    gameState.firstTryCorrectCount++;
                }

                feedbackBox.className = 'mt-4 p-3.5 rounded-2xl border border-emerald-400/50 bg-emerald-950/60 text-xs sm:text-sm leading-relaxed';
                fbTitle.textContent = `✅ Jawapan Tepat! (+${pointsEarned} Markah)`;
                fbTitle.className = 'font-extrabold mb-1 text-emerald-300';
                fbDesc.textContent = q.explanation;

                const nextBtn = document.getElementById('btnQuizNextLevel');
                nextBtn.textContent = gameState.currentLevelIndex === LEVELS.length - 1
                    ? '🏆 Lihat Keputusan Akhir!'
                    : `Teruskan ke Level ${cfg.level + 1} ➡️`;
                nextBtn.classList.remove('hidden');
                updateHUD();
            } else {
                clickedBtn.classList.remove('border-slate-700', 'bg-slate-800/85');
                clickedBtn.classList.add('border-rose-500', 'bg-rose-950/75');

                audio.playWrong();
                gameState.quizHadWrongAttempt = true;
                gameState.score = Math.max(0, gameState.score - 25);

                feedbackBox.className = 'mt-4 p-3.5 rounded-2xl border border-rose-500/50 bg-rose-950/60 text-xs sm:text-sm leading-relaxed';
                fbTitle.textContent = '❌ Kurang Tepat! (-25 Markah)';
                fbTitle.className = 'font-extrabold mb-1 text-rose-300';
                fbDesc.textContent = 'Sila semak semula maksud dan perbezaan Azan dan Iqamat pada poster, kemudian cuba pilih jawapan yang betul.';

                document.getElementById('btnQuizRetry').classList.remove('hidden');
                updateHUD();
            }
        }

        function triggerVictoryScreen() {
            gameState.quizOpen = false;
            document.getElementById('quizModal').classList.add('hidden');
            document.getElementById('quizModal').classList.remove('flex');

            document.getElementById('victoryScore').textContent = gameState.score;
            document.getElementById('victoryFirstTry').textContent = `${gameState.firstTryCorrectCount} / 10`;

            const starsEl = document.getElementById('victoryStars');
            if (gameState.firstTryCorrectCount >= 8) starsEl.textContent = '⭐⭐⭐';
            else if (gameState.firstTryCorrectCount >= 5) starsEl.textContent = '⭐⭐';
            else starsEl.textContent = '⭐';

            const vModal = document.getElementById('victoryOverlay');
            vModal.classList.remove('hidden');
            vModal.classList.add('flex');
            audio.playCorrect();
        }

        // --- 8. SISTEM PENCAHAYAAN DINAMIK ---
        function getSunlightLevel() {
            const angle = (gameState.timeOfDay - 0.25) * Math.PI * 2;
            const raw = Math.cos(angle);
            return Math.max(0.25, Math.min(1.0, (raw + 1) * 0.5 * 0.92 + 0.22));
        }

        function updateLightMap() {
            const sun = getSunlightLevel();
            const camStartX = Math.max(0, Math.floor(camera.x / TILE_SIZE) - 6);
            const camEndX = Math.min(WORLD_WIDTH - 1, Math.ceil((camera.x + canvas.width) / TILE_SIZE) + 6);
            const camStartY = Math.max(0, Math.floor(camera.y / TILE_SIZE) - 6);
            const camEndY = Math.min(WORLD_HEIGHT - 1, Math.ceil((camera.y + canvas.height) / TILE_SIZE) + 6);

            for (let x = camStartX; x <= camEndX; x++) {
                const surfY = surfaceHeights[x] || 24;
                for (let y = camStartY; y <= camEndY; y++) {
                    if (y < surfY) {
                        lightMap[y][x] = sun;
                    } else if (y === surfY) {
                        lightMap[y][x] = sun * 0.9;
                    } else {
                        const depth = y - surfY;
                        lightMap[y][x] = Math.max(0.1, sun * Math.pow(0.75, depth));
                    }
                }
            }
        }

        // --- 9. PENGURUSAN INVENTORI & RAMUAN ---
        function getSelectedItem() {
            const slot = player.inventory[player.selectedSlot];
            return slot ? ITEMS[slot.id] : null;
        }

        function addItemToInventory(itemId, count = 1) {
            for (let i = 0; i < player.inventory.length; i++) {
                if (player.inventory[i] && player.inventory[i].id === itemId) {
                    player.inventory[i].count += count;
                    renderHotbar();
                    return true;
                }
            }
            for (let i = 0; i < player.inventory.length; i++) {
                if (!player.inventory[i]) {
                    player.inventory[i] = { id: itemId, count };
                    renderHotbar();
                    return true;
                }
            }
            return false;
        }

        function countItemInInventory(itemId) {
            let total = 0;
            for (const slot of player.inventory) {
                if (slot && slot.id === itemId) total += slot.count;
            }
            return total;
        }

        function consumeItems(ingredients) {
            for (const ing of ingredients) {
                let rem = ing.count;
                for (let i = 0; i < player.inventory.length; i++) {
                    const slot = player.inventory[i];
                    if (slot && slot.id === ing.id) {
                        const take = Math.min(slot.count, rem);
                        slot.count -= take;
                        rem -= take;
                        if (slot.count <= 0) player.inventory[i] = null;
                        if (rem <= 0) break;
                    }
                }
            }
            renderHotbar();
        }

        function renderItemGraphic(item) {
            if (!item) return '';
            return `<span class="text-lg sm:text-xl leading-none drop-shadow">${item.icon || '📦'}</span>`;
        }

        function renderHotbar() {
            const container = document.getElementById('hotbarContainer');
            container.innerHTML = '';

            for (let i = 0; i < 6; i++) {
                const slotData = player.inventory[i];
                const item = slotData ? ITEMS[slotData.id] : null;
                const isActive = i === player.selectedSlot;

                const btn = document.createElement('button');
                btn.className = `hotbar-slot relative w-11 h-11 sm:w-12 sm:h-12 rounded-xl border-2 ${
                    isActive ? 'active' : 'border-slate-700/80 bg-slate-900/75 hover:border-slate-500'
                } flex items-center justify-center cursor-pointer`;

                btn.innerHTML = `
                    <span class="absolute top-0.5 left-1.5 text-[9px] font-bold text-slate-400">${i + 1}</span>
                    ${item ? renderItemGraphic(item) : ''}
                    ${slotData && slotData.count > 1 ? `<span class="absolute bottom-0.5 right-1.5 text-[10px] font-extrabold text-white drop-shadow">${slotData.count}</span>` : ''}
                `;

                btn.addEventListener('click', () => {
                    player.selectedSlot = i;
                    useConsumableIfClicked(i);
                    renderHotbar();
                });

                container.appendChild(btn);
            }

            const selected = getSelectedItem();
            document.getElementById('selectedItemLabel').textContent = selected ? selected.name : 'Tangan Kosong';
        }

        function useConsumableIfClicked(slotIndex) {
            const slot = player.inventory[slotIndex];
            if (!slot) return;
            const item = ITEMS[slot.id];
            if (item && item.type === 'consumable') {
                if (player.hp < player.maxHp) {
                    player.hp = Math.min(player.maxHp, player.hp + item.heal);
                    slot.count--;
                    if (slot.count <= 0) player.inventory[slotIndex] = null;
                    audio.playCraft();
                    spawnFloatingText(player.x + player.w / 2, player.y - 10, `+${item.heal} Nyawa!`, '#4ade80');
                    updateHUD();
                    renderHotbar();
                } else {
                    showToast('Nyawa anda sudah penuh!');
                }
            }
        }

        function renderCraftingMenu() {
            const list = document.getElementById('recipeList');
            list.innerHTML = '';

            RECIPES.forEach((recipe) => {
                const resItem = ITEMS[recipe.result];
                const canCraft = recipe.ingredients.every(ing => countItemInInventory(ing.id) >= ing.count);

                const ingHTML = recipe.ingredients.map(ing => {
                    const ingItem = ITEMS[ing.id];
                    const owned = countItemInInventory(ing.id);
                    const ok = owned >= ing.count;
                    return `<span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-lg text-[11px] font-semibold ${
                        ok ? 'bg-emerald-950/80 text-emerald-300 border border-emerald-700/50' : 'bg-rose-950/70 text-rose-300 border border-rose-800/50'
                    }">${ingItem.name}: ${owned}/${ing.count}</span>`;
                }).join(' ');

                const card = document.createElement('div');
                card.className = `p-3.5 rounded-2xl border transition flex flex-col justify-between gap-2.5 ${
                    canCraft ? 'bg-slate-800/85 border-amber-500/40 hover:border-amber-400' : 'bg-slate-900/60 border-slate-800 opacity-75'
                }`;

                card.innerHTML = `
                    <div class="flex items-start gap-3">
                        <div class="w-11 h-11 rounded-xl bg-slate-900 border border-slate-700 flex items-center justify-center shrink-0">
                            ${renderItemGraphic(resItem)}
                        </div>
                        <div class="flex-1 min-w-0">
                            <h3 class="font-bold text-sm text-slate-100 truncate">${resItem.name} ${recipe.count > 1 ? `×${recipe.count}` : ''}</h3>
                            <p class="text-[11px] text-slate-400 mt-0.5 line-clamp-1">${recipe.desc}</p>
                        </div>
                    </div>
                    <div class="flex flex-wrap gap-1.5">${ingHTML}</div>
                    <button class="craft-btn w-full py-2 rounded-xl font-bold text-xs transition ${
                        canCraft
                            ? 'bg-gradient-to-r from-amber-500 to-orange-500 hover:from-amber-400 hover:to-orange-400 text-slate-950 shadow cursor-pointer'
                            : 'bg-slate-800 text-slate-500 cursor-not-allowed'
                    }" ${canCraft ? '' : 'disabled'}>
                        ${canCraft ? '🧪 Bancuh Ramuan' : 'Bahan Tidak Mencukupi'}
                    </button>
                `;

                const btn = card.querySelector('.craft-btn');
                if (canCraft) {
                    btn.addEventListener('click', () => {
                        consumeItems(recipe.ingredients);
                        addItemToInventory(recipe.result, recipe.count);
                        audio.playCraft();
                        showToast(`Berjaya membancuh ${resItem.name}!`);
                        renderCraftingMenu();
                    });
                }

                list.appendChild(card);
            });
        }

        // --- 10. BOSS & RAKSASA UNIK SETIAP LEVEL ---
        function spawnEnemy(spawnClose = false, forcedDir = null, forcedDist = null, forcedType = null) {
            const dir = forcedDir !== null ? forcedDir : (Math.random() < 0.5 ? -1 : 1);
            const offsetTiles = forcedDist !== null
                ? forcedDist
                : (spawnClose ? (6 + Math.random() * 7) : (10 + Math.random() * 8));
            const pxTile = Math.floor(player.x / TILE_SIZE);
            const spawnTileX = Math.max(8, Math.min(WORLD_WIDTH - 9, Math.floor(pxTile + dir * offsetTiles)));

            const cfg = LEVELS[gameState.currentLevelIndex];
            const type = forcedType || cfg.bossConfig.types[0] || 'slime_king';

            const enemyCatalog = {
                slime_king:   { name: 'Raja Lendir Hijau',   w: 44, h: 36, hp: 45,  maxHp: 45,  dmg: 8,  color: '#22c55e', border: '#fde047', jumpForce: -360, speed: 100, ai: 'hop',   flying: false },
                wolf:         { name: 'Serigala Rimba',      w: 44, h: 28, hp: 36,  maxHp: 36,  dmg: 10, color: '#64748b', border: '#cbd5e1', jumpForce: -330, speed: 165, ai: 'run',   flying: false },
                bat:          { name: 'Kelawar Gua',         w: 38, h: 24, hp: 32,  maxHp: 32,  dmg: 10, color: '#7e22ce', border: '#d8b4fe', jumpForce: 0,    speed: 145, ai: 'fly',   flying: true  },
                skeleton:     { name: 'Pahlawan Rangka',     w: 28, h: 44, hp: 55,  maxHp: 55,  dmg: 12, color: '#e2e8f0', border: '#94a3b8', jumpForce: -380, speed: 125, ai: 'march', flying: false },
                spider:       { name: 'Labah-labah Kristal', w: 42, h: 26, hp: 50,  maxHp: 50,  dmg: 12, color: '#0284c7', border: '#7dd3fc', jumpForce: -430, speed: 155, ai: 'hop',   flying: false },
                golem:        { name: 'Golem Batu Purba',    w: 48, h: 54, hp: 110, maxHp: 110, dmg: 16, color: '#57534e', border: '#fbbf24', jumpForce: -310, speed: 85,  ai: 'march', flying: false },
                ghost:        { name: 'Roh Bayang Malam',    w: 34, h: 42, hp: 58,  maxHp: 58,  dmg: 14, color: '#38bdf8', border: '#bae6fd', jumpForce: 0,    speed: 150, ai: 'fly',   flying: true  },
                magma_knight: { name: 'Kesatria Magma',      w: 32, h: 46, hp: 85,  maxHp: 85,  dmg: 16, color: '#dc2626', border: '#facc15', jumpForce: -390, speed: 145, ai: 'march', flying: false },
                dragon:       { name: 'Naga Api Merah',      w: 58, h: 42, hp: 160, maxHp: 160, dmg: 18, color: '#ea580c', border: '#fef08a', jumpForce: 0,    speed: 155, ai: 'fly',   flying: true  },
                dark_lord:    { name: 'Maharaja Kegelapan',  w: 42, h: 56, hp: 210, maxHp: 210, dmg: 20, color: '#312e81', border: '#f43f5e', jumpForce: -420, speed: 160, ai: 'boss',  flying: false }
            };

            const stats = enemyCatalog[type] || enemyCatalog.slime_king;
            const groundY = ((surfaceHeights[spawnTileX] || 24) * TILE_SIZE) - stats.h;
            const spawnY = stats.flying ? groundY - 55 : groundY;

            enemies.push({
                x: spawnTileX * TILE_SIZE,
                y: spawnY,
                vx: 0,
                vy: 0,
                facing: -dir,
                grounded: false,
                jumpCooldown: 0.4 + Math.random() * 0.6,
                hurtTimer: 0,
                animPhase: Math.random() * Math.PI * 2,
                ...stats,
                type
            });
        }

        function damagePlayer(amount, knockDir = 0) {
            if (player.invulnTimer > 0 || gameState.gameOver || gameState.quizOpen) return;
            player.hp = Math.max(0, player.hp - amount);
            player.invulnTimer = 0.85;
            player.vx = knockDir * 240;
            player.vy = -220;
            audio.playPlayerHurt();
            spawnFloatingText(player.x + player.w / 2, player.y - 8, `-${amount}`, '#f43f5e');
            spawnParticles(player.x + player.w / 2, player.y + player.h / 2, '#f43f5e', 10);
            updateHUD();

            if (player.hp <= 0) {
                triggerGameOver();
            }
        }

        function triggerGameOver() {
            gameState.gameOver = true;
            document.getElementById('failLevelText').textContent = `Level ${gameState.currentLevelIndex + 1} / 10`;
            document.getElementById('failScoreText').textContent = gameState.score;
            const over = document.getElementById('gameOverOverlay');
            over.classList.remove('hidden');
            over.classList.add('flex');
        }

        // --- 11. FIZIK & PERLANGGARAN ---
        function isTileSolid(tx, ty) {
            if (tx < 0 || tx >= WORLD_WIDTH || ty < 0 || ty >= WORLD_HEIGHT) return true;
            const t = world[ty][tx];
            return TILE_PROPS[t] ? TILE_PROPS[t].solid : false;
        }

        function moveEntityWithCollisions(ent, dt) {
            const gravity = 1180;
            ent.vy = Math.min(820, ent.vy + gravity * dt);

            ent.x += ent.vx * dt;
            const leftTile = Math.floor(ent.x / TILE_SIZE);
            const rightTile = Math.floor((ent.x + ent.w - 0.01) / TILE_SIZE);
            const topTile = Math.floor(ent.y / TILE_SIZE);
            const bottomTile = Math.floor((ent.y + ent.h - 0.01) / TILE_SIZE);

            if (ent.vx > 0) {
                for (let ty = topTile; ty <= bottomTile; ty++) {
                    if (isTileSolid(rightTile, ty)) {
                        ent.x = rightTile * TILE_SIZE - ent.w;
                        ent.vx = 0;
                        break;
                    }
                }
            } else if (ent.vx < 0) {
                for (let ty = topTile; ty <= bottomTile; ty++) {
                    if (isTileSolid(leftTile, ty)) {
                        ent.x = (leftTile + 1) * TILE_SIZE;
                        ent.vx = 0;
                        break;
                    }
                }
            }

            ent.y += ent.vy * dt;
            ent.grounded = false;
            const leftTileY = Math.floor(ent.x / TILE_SIZE);
            const rightTileY = Math.floor((ent.x + ent.w - 0.01) / TILE_SIZE);
            const topTileY = Math.floor(ent.y / TILE_SIZE);
            const bottomTileY = Math.floor((ent.y + ent.h - 0.01) / TILE_SIZE);

            if (ent.vy > 0) {
                for (let tx = leftTileY; tx <= rightTileY; tx++) {
                    if (isTileSolid(tx, bottomTileY)) {
                        ent.y = bottomTileY * TILE_SIZE - ent.h;
                        ent.vy = 0;
                        ent.grounded = true;
                        break;
                    }
                }
            } else if (ent.vy < 0) {
                for (let tx = leftTileY; tx <= rightTileY; tx++) {
                    if (isTileSolid(tx, topTileY)) {
                        ent.y = (topTileY + 1) * TILE_SIZE;
                        ent.vy = 0;
                        break;
                    }
                }
            }
        }

        // --- 12. KESAN ZARAH & TEKS TERAPUNG ---
        function spawnParticles(x, y, color, count = 6) {
            for (let i = 0; i < count; i++) {
                particles.push({
                    x, y,
                    vx: (Math.random() - 0.5) * 190,
                    vy: (Math.random() - 0.8) * 200,
                    size: 3 + Math.random() * 4,
                    color,
                    life: 0.35 + Math.random() * 0.35,
                    maxLife: 0.7
                });
            }
        }

        function spawnFloatingText(x, y, text, color = '#ffffff') {
            floatingTexts.push({ x, y, text, color, vy: -45, life: 0.95 });
        }

        function showToast(msg) {
            const el = document.getElementById('toastNotification');
            el.textContent = msg;
            el.classList.remove('opacity-0');
            clearTimeout(el._timer);
            el._timer = setTimeout(() => el.classList.add('opacity-0'), 2600);
        }

        // --- 13. KEMAS KINI LOGIK UTAMA ---
        function update(dt) {
            for (const c of clouds) {
                c.x += c.speed * dt;
                if (c.x > WORLD_WIDTH * TILE_SIZE + 100) c.x = -c.w;
            }
            if (portal.active) portal.pulse += dt * 4;

            if (!gameState.started || gameState.gameOver || gameState.quizOpen) return;

            if (
                portal.active &&
                player.x < portal.x + portal.w &&
                player.x + player.w > portal.x &&
                player.y < portal.y + portal.h &&
                player.y + player.h > portal.y
            ) {
                openLevelQuiz();
                return;
            }

            gameState.timeOfDay = (gameState.timeOfDay + gameState.daySpeed * dt) % 1.0;

            if (player.invulnTimer > 0) player.invulnTimer -= dt;
            if (player.swingTimer > 0) player.swingTimer -= dt;

            player.stamina = Math.min(player.maxStamina, player.stamina + 16 * dt);
            if (player.hp < player.maxHp && player.invulnTimer <= 0) {
                player.hp = Math.min(player.maxHp, player.hp + 0.8 * dt);
            }

            let moveDir = 0;
            if (keys['KeyA'] || keys['ArrowLeft']) moveDir -= 1;
            if (keys['KeyD'] || keys['ArrowRight']) moveDir += 1;

            if (moveDir !== 0) {
                player.vx = moveDir * player.speed;
                player.facing = moveDir;
                player.animTimer += dt * 10;
            } else {
                player.vx *= 0.72;
                if (Math.abs(player.vx) < 5) player.vx = 0;
                player.animTimer = 0;
            }

            if ((keys['KeyW'] || keys['ArrowUp'] || keys['Space']) && player.grounded) {
                player.vy = player.jumpForce;
                player.grounded = false;
                audio.playJump();
                spawnParticles(player.x + player.w / 2, player.y + player.h, '#cbd5e1', 5);
            }

            moveEntityWithCollisions(player, dt);

            const targetCamX = player.x + player.w / 2 - canvas.width / 2;
            const targetCamY = player.y + player.h / 2 - canvas.height / 2;
            camera.x += (targetCamX - camera.x) * Math.min(1, dt * 10);
            camera.y += (targetCamY - camera.y) * Math.min(1, dt * 10);
            camera.x = Math.max(0, Math.min(WORLD_WIDTH * TILE_SIZE - canvas.width, camera.x));
            camera.y = Math.max(0, Math.min(WORLD_HEIGHT * TILE_SIZE - canvas.height, camera.y));

            mouse.worldX = mouse.x + camera.x;
            mouse.worldY = mouse.y + camera.y;

            const anyModalOpen = gameState.craftingOpen || gameState.helpOpen || gameState.posterOpen;

            if (mouse.leftDown && !anyModalOpen) {
                if (player.swingTimer <= 0) {
                    player.swingTimer = player.swingDuration;
                    audio.playSwing();
                    if (mouse.x !== 0 || mouse.y !== 0) {
                        player.facing = mouse.worldX >= player.x + player.w / 2 ? 1 : -1;
                    }
                    checkMeleeAttack();
                }
            }

            if (mouse.rightDown && !anyModalOpen) {
                const potionIdx = player.inventory.findIndex(s => s && s.id === 'potion');
                if (potionIdx !== -1) {
                    useConsumableIfClicked(potionIdx);
                }
                mouse.rightDown = false;
            }

            const prog = getObjectiveProgress();
            const remainingNeeded = prog.target - prog.current;
            if (!gameState.objectiveCompleted && enemies.length < remainingNeeded) {
                spawnEnemy(true);
            }

            for (let i = enemies.length - 1; i >= 0; i--) {
                const e = enemies[i];
                if (e.hurtTimer > 0) e.hurtTimer -= dt;
                e.animPhase += dt * 7;

                const distToPlayer = Math.hypot(player.x - e.x, player.y - e.y);
                if (distToPlayer > 26 * TILE_SIZE) {
                    const respawnDir = Math.random() < 0.5 ? 1 : -1;
                    const newTileX = Math.max(8, Math.min(WORLD_WIDTH - 9, Math.floor(player.x / TILE_SIZE) + respawnDir * 10));
                    e.x = newTileX * TILE_SIZE;
                    const gY = ((surfaceHeights[newTileX] || 24) * TILE_SIZE) - e.h;
                    e.y = e.flying ? gY - 55 : gY;
                    e.vx = 0;
                    e.vy = 0;
                }

                const dir = (player.x + player.w / 2) >= (e.x + e.w / 2) ? 1 : -1;
                e.facing = dir;

                if (e.flying) {
                    const targetX = player.x + player.w / 2 - e.w / 2;
                    const targetY = player.y - 6 + Math.sin(e.animPhase) * 24;
                    const dx = targetX - e.x;
                    const dy = targetY - e.y;
                    const d = Math.max(1, Math.hypot(dx, dy));
                    e.vx = (dx / d) * e.speed;
                    e.vy = (dy / d) * (e.speed * 0.85);
                    e.x += e.vx * dt;
                    e.y += e.vy * dt;
                } else {
                    if (e.ai === 'hop') {
                        if (e.grounded) {
                            e.vx *= 0.8;
                            e.jumpCooldown -= dt;
                            if (e.jumpCooldown <= 0) {
                                e.vx = dir * e.speed;
                                e.vy = e.jumpForce;
                                e.grounded = false;
                                e.jumpCooldown = 0.85 + Math.random() * 0.7;
                            }
                        }
                    } else {
                        e.vx = dir * e.speed;
                        e.jumpCooldown -= dt;
                        if (e.grounded && (e.jumpCooldown <= 0 || Math.abs(e.vx) < 10)) {
                            e.vy = e.jumpForce;
                            e.grounded = false;
                            e.jumpCooldown = (e.ai === 'run' || e.ai === 'boss') ? 1.1 : 1.7;
                        }
                    }
                    moveEntityWithCollisions(e, dt);
                }

                if (
                    player.x < e.x + e.w &&
                    player.x + player.w > e.x &&
                    player.y < e.y + e.h &&
                    player.y + player.h > e.y
                ) {
                    const knockDir = player.x + player.w / 2 > e.x + e.w / 2 ? 1 : -1;
                    damagePlayer(e.dmg, knockDir);
                }
            }

            for (let i = droppedItems.length - 1; i >= 0; i--) {
                const d = droppedItems[i];
                d.bob += dt * 5;
                const px = player.x + player.w / 2;
                const py = player.y + player.h / 2;
                const dist = Math.hypot(px - d.x, py - d.y);

                if (dist < 85) {
                    d.x += ((px - d.x) / dist) * 280 * dt;
                    d.y += ((py - d.y) / dist) * 280 * dt;
                }

                if (dist < 24) {
                    if (addItemToInventory(d.id, d.count)) {
                        audio.playPickup();
                        const itemObj = ITEMS[d.id];
                        gameState.score += 10 * d.count;
                        spawnFloatingText(px, player.y - 6, `+${d.count} ${itemObj.name}`, '#fde047');
                        droppedItems.splice(i, 1);
                    }
                }
            }

            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.vy += 450 * dt;
                p.x += p.vx * dt;
                p.y += p.vy * dt;
                p.life -= dt;
                if (p.life <= 0) particles.splice(i, 1);
            }

            for (let i = floatingTexts.length - 1; i >= 0; i--) {
                const ft = floatingTexts[i];
                ft.y += ft.vy * dt;
                ft.life -= dt;
                if (ft.life <= 0) floatingTexts.splice(i, 1);
            }

            updateLightMap();
            updateHUD();
        }

        function checkMeleeAttack() {
            const held = getSelectedItem();
            const dmg = held ? (held.damage || 24) : 24;
            const px = player.x + player.w / 2;
            const py = player.y + player.h / 2;

            for (let i = enemies.length - 1; i >= 0; i--) {
                const e = enemies[i];
                const ex = e.x + e.w / 2;
                const ey = e.y + e.h / 2;
                const dist = Math.hypot(ex - px, ey - py);
                const inFacingDir = (ex - px) * player.facing >= -24;

                if (dist <= 95 && inFacingDir && e.hurtTimer <= 0) {
                    e.hp -= dmg;
                    e.hurtTimer = 0.22;
                    e.vx = player.facing * 270;
                    e.vy = -210;
                    audio.playHitEnemy();
                    spawnFloatingText(ex, e.y - 6, `-${dmg}`, '#fbbf24');
                    spawnParticles(ex, ey, e.color, 8);

                    if (e.hp <= 0) {
                        gameState.levelKills++;
                        gameState.score += 50;
                        spawnParticles(ex, ey, e.color, 18);
                        droppedItems.push({
                            id: 'gel',
                            count: 1,
                            x: ex,
                            y: ey,
                            bob: 0
                        });
                        enemies.splice(i, 1);
                        checkObjectiveCompletion();
                    }
                }
            }
        }

        // --- 14. KEMAS KINI UI HUD ---
        function updateHUD() {
            const hpVal = Math.ceil(player.hp);
            document.getElementById('hpText').textContent = `${hpVal}/${player.maxHp}`;

            const heartsCont = document.getElementById('heartsContainer');
            let heartsHTML = '';
            for (let i = 0; i < 5; i++) {
                const threshold = (i + 1) * 20;
                if (hpVal >= threshold) heartsHTML += `<span class="text-sm drop-shadow">❤️</span>`;
                else if (hpVal > i * 20) heartsHTML += `<span class="text-sm opacity-75">💔</span>`;
                else heartsHTML += `<span class="text-sm opacity-25">🖤</span>`;
            }
            heartsCont.innerHTML = heartsHTML;

            document.getElementById('staminaBar').style.width = `${(player.stamina / player.maxStamina) * 100}%`;

            const cfg = LEVELS[gameState.currentLevelIndex];
            document.getElementById('levelBadge').textContent = `🚩 Level ${cfg.level} / 10`;
            document.getElementById('scoreBadge').textContent = `⭐ ${gameState.score}`;

            const prog = getObjectiveProgress();
            const pct = Math.min(100, (prog.current / prog.target) * 100);
            const objPanel = document.getElementById('objectivePanel');

            document.getElementById('levelTitleText').textContent = cfg.title;
            document.getElementById('objectiveBar').style.width = `${pct}%`;

            if (gameState.objectiveCompleted) {
                document.getElementById('objectiveProgressText').textContent = 'SELESAI ✓';
                document.getElementById('objectiveDescText').textContent = '🌀 Portal Cahaya dibuka! Ikuti penunjuk anak panah biru ke Portal!';
                document.getElementById('objectiveDescMobile').textContent = '🌀 Pergi ke Portal Cahaya!';
                document.getElementById('objectiveProgressMobile').textContent = 'BUKA';
                objPanel.classList.add('portal-ready-badge', 'border-sky-400');
            } else {
                document.getElementById('objectiveProgressText').textContent = `${prog.current} / ${prog.target} ${prog.label}`;
                document.getElementById('objectiveDescText').textContent = cfg.desc;
                document.getElementById('objectiveDescMobile').textContent = `${prog.label}:`;
                document.getElementById('objectiveProgressMobile').textContent = `${prog.current}/${prog.target}`;
                objPanel.classList.remove('portal-ready-badge', 'border-sky-400');
            }
        }

        // --- 15. PENJANAAN GRAFIK (RENDERING) ---
        function drawSkyAndBackdrop() {
            const sun = getSunlightLevel();
            const skyGrad = ctx.createLinearGradient(0, 0, 0, canvas.height);

            if (sun > 0.65) {
                skyGrad.addColorStop(0, '#0284c7');
                skyGrad.addColorStop(0.6, '#38bdf8');
                skyGrad.addColorStop(1, '#bae6fd');
            } else if (sun > 0.42) {
                skyGrad.addColorStop(0, '#1e1b4b');
                skyGrad.addColorStop(0.5, '#be185d');
                skyGrad.addColorStop(1, '#f97316');
            } else {
                skyGrad.addColorStop(0, '#020617');
                skyGrad.addColorStop(0.7, '#0f172a');
                skyGrad.addColorStop(1, '#1e293b');
            }
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            if (sun < 0.6) {
                const starAlpha = (0.6 - sun) * 1.8;
                ctx.fillStyle = `rgba(255, 255, 255, ${Math.min(0.95, starAlpha)})`;
                for (const s of stars) {
                    ctx.fillRect(s.x * canvas.width, s.y * canvas.height, s.size, s.size);
                }
            }

            ctx.fillStyle = `rgba(255, 255, 255, ${0.15 + sun * 0.55})`;
            for (const c of clouds) {
                const screenX = c.x - camera.x * 0.35;
                const screenY = c.y - camera.y * 0.2;
                if (screenX + c.w > 0 && screenX < canvas.width && screenY + 40 > 0 && screenY < canvas.height) {
                    ctx.beginPath();
                    ctx.roundRect(screenX, screenY, c.w, 22, 11);
                    ctx.roundRect(screenX + 14, screenY - 10, c.w * 0.55, 20, 10);
                    ctx.fill();
                }
            }
        }

        function drawWorld() {
            const startX = Math.max(0, Math.floor(camera.x / TILE_SIZE));
            const endX = Math.min(WORLD_WIDTH - 1, Math.ceil((camera.x + canvas.width) / TILE_SIZE));
            const startY = Math.max(0, Math.floor(camera.y / TILE_SIZE));
            const endY = Math.min(WORLD_HEIGHT - 1, Math.ceil((camera.y + canvas.height) / TILE_SIZE));

            for (let y = startY; y <= endY; y++) {
                for (let x = startX; x <= endX; x++) {
                    const tile = world[y][x];
                    const drawX = Math.floor(x * TILE_SIZE - camera.x);
                    const drawY = Math.floor(y * TILE_SIZE - camera.y);
                    const light = lightMap[y][x] || 0;

                    if (tile !== TILES.AIR && tileTextures[tile]) {
                        ctx.drawImage(tileTextures[tile], drawX, drawY);
                    }

                    if (y >= (surfaceHeights[x] || 24) || tile !== TILES.AIR) {
                        const darkness = Math.max(0, Math.min(0.85, 1 - light));
                        if (darkness > 0.02) {
                            ctx.fillStyle = `rgba(2, 6, 23, ${darkness})`;
                            ctx.fillRect(drawX, drawY, TILE_SIZE, TILE_SIZE);
                        }
                    }
                }
            }
        }

        function drawPortalAndCompass() {
            if (!portal.active) return;

            const px = Math.floor(portal.x - camera.x);
            const py = Math.floor(portal.y - camera.y);
            const cx = px + portal.w / 2;
            const cy = py + portal.h / 2;

            ctx.fillStyle = '#1e293b';
            ctx.beginPath();
            ctx.roundRect(px - 4, py - 4, portal.w + 8, portal.h + 4, [24, 24, 4, 4]);
            ctx.fill();
            ctx.strokeStyle = '#38bdf8';
            ctx.lineWidth = 3;
            ctx.stroke();

            const pulseR = Math.sin(portal.pulse) * 4;
            const grad = ctx.createRadialGradient(cx, cy, 4, cx, cy, 28 + pulseR);
            grad.addColorStop(0, '#ffffff');
            grad.addColorStop(0.4, '#38bdf8');
            grad.addColorStop(0.8, '#10b981');
            grad.addColorStop(1, 'rgba(16, 185, 129, 0)');
            ctx.fillStyle = grad;
            ctx.beginPath();
            ctx.ellipse(cx, cy, portal.w * 0.45, portal.h * 0.45, 0, 0, Math.PI * 2);
            ctx.fill();

            ctx.font = 'bold 12px "Plus Jakarta Sans", sans-serif';
            ctx.textAlign = 'center';
            ctx.fillStyle = '#7dd3fc';
            ctx.fillText('🌀 MASUK PORTAL', cx, py - 10);

            if (gameState.started && !gameState.quizOpen) {
                const playerCX = player.x + player.w / 2 - camera.x;
                const playerCY = player.y + player.h / 2 - camera.y;
                const dx = cx - playerCX;
                const dy = cy - playerCY;
                const dist = Math.hypot(dx, dy);

                if (dist > 110) {
                    const angle = Math.atan2(dy, dx);
                    const indDist = Math.min(dist - 35, Math.min(canvas.width, canvas.height) * 0.34);
                    const ix = playerCX + Math.cos(angle) * indDist;
                    const iy = playerCY + Math.sin(angle) * indDist;

                    ctx.save();
                    ctx.translate(ix, iy);
                    ctx.rotate(angle);
                    ctx.fillStyle = '#38bdf8';
                    ctx.beginPath();
                    ctx.moveTo(15, 0);
                    ctx.lineTo(-8, -9);
                    ctx.lineTo(-4, 0);
                    ctx.lineTo(-8, 9);
                    ctx.closePath();
                    ctx.fill();
                    ctx.restore();
                }
            }
        }

        function drawPlayer() {
            const px = Math.floor(player.x - camera.x);
            const py = Math.floor(player.y - camera.y);

            if (player.invulnTimer > 0 && Math.floor(player.invulnTimer * 18) % 2 === 0) {
                ctx.globalAlpha = 0.45;
            }

            const legOffset = player.grounded ? Math.sin(player.animTimer) * 4 : 3;

            ctx.fillStyle = '#1e3a8a';
            ctx.fillRect(px + 4, py + 28 + legOffset, 6, 14 - Math.max(0, legOffset));
            ctx.fillRect(px + 12, py + 28 - legOffset, 6, 14 - Math.max(0, -legOffset));

            ctx.fillStyle = '#059669';
            ctx.fillRect(px + 3, py + 14, 16, 15);
            ctx.fillStyle = '#fbbf24';
            ctx.fillRect(px + 3, py + 25, 16, 3);

            ctx.fillStyle = '#fed7aa';
            ctx.fillRect(px + 4, py + 2, 14, 13);

            ctx.fillStyle = '#f8fafc';
            ctx.fillRect(px + 4, py - 1, 14, 5);
            ctx.fillStyle = '#cbd5e1';
            ctx.fillRect(px + 4, py + 3, 14, 1.5);

            ctx.fillStyle = '#0f172a';
            const eyeX = player.facing === 1 ? px + 13 : px + 6;
            ctx.fillRect(eyeX, py + 7, 3, 3);

            const held = getSelectedItem();
            if (held) {
                ctx.save();
                ctx.translate(px + player.w / 2, py + 20);
                let swingAngle = 0.25;
                if (player.swingTimer > 0) {
                    const t = 1 - player.swingTimer / player.swingDuration;
                    swingAngle = -0.9 + t * 2.2;
                }
                ctx.scale(player.facing, 1);
                ctx.rotate(swingAngle);

                if (held.type === 'weapon') {
                    ctx.fillStyle = '#fbbf24';
                    ctx.fillRect(0, -3, 5, 6);
                    ctx.fillStyle = held.color;
                    ctx.fillRect(5, -2, 22, 4);
                } else if (held.type === 'consumable') {
                    ctx.fillStyle = '#f43f5e';
                    ctx.beginPath();
                    ctx.arc(10, 0, 5, 0, Math.PI * 2);
                    ctx.fill();
                }
                ctx.restore();
            }

            ctx.globalAlpha = 1.0;
        }

        function drawEnemies() {
            for (const e of enemies) {
                const ex = Math.floor(e.x - camera.x);
                const ey = Math.floor(e.y - camera.y);
                if (ex + e.w < -60 || ex > canvas.width + 60 || ey + e.h < -60 || ey > canvas.height + 60) continue;

                const isHurt = e.hurtTimer > 0;
                const mainColor = isHurt ? '#ffffff' : e.color;
                const flap = Math.sin(e.animPhase * 2) * 8;
                const walkBob = e.grounded ? Math.sin(e.animPhase * 1.5) * 3 : 0;

                ctx.save();

                if (e.type === 'slime_king') {
                    const squash = e.grounded ? 0 : -4;
                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.roundRect(ex - squash / 2, ey + squash, e.w + squash, e.h - squash, [18, 18, 6, 6]);
                    ctx.fill();
                    ctx.strokeStyle = e.border;
                    ctx.lineWidth = 2.5;
                    ctx.stroke();

                    ctx.fillStyle = '#facc15';
                    ctx.beginPath();
                    ctx.moveTo(ex + 12, ey + squash);
                    ctx.lineTo(ex + 15, ey - 10 + squash);
                    ctx.lineTo(ex + 22, ey - 3 + squash);
                    ctx.lineTo(ex + 29, ey - 10 + squash);
                    ctx.lineTo(ex + 32, ey + squash);
                    ctx.closePath();
                    ctx.fill();

                    ctx.fillStyle = '#0f172a';
                    ctx.fillRect(ex + 12, ey + 14, 4, 5);
                    ctx.fillRect(ex + 27, ey + 14, 4, 5);

                } else if (e.type === 'wolf') {
                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.roundRect(ex + 4, ey + 8 + walkBob, e.w - 8, e.h - 12, 6);
                    ctx.fill();
                    ctx.strokeStyle = e.border;
                    ctx.lineWidth = 2;
                    ctx.stroke();

                    ctx.fillStyle = '#334155';
                    ctx.fillRect(ex + 8, ey + e.h - 6, 5, 6);
                    ctx.fillRect(ex + e.w - 13, ey + e.h - 6, 5, 6);

                    const headX = e.facing === 1 ? ex + e.w - 16 : ex;
                    ctx.fillStyle = mainColor;
                    ctx.fillRect(headX, ey + 2 + walkBob, 16, 14);
                    ctx.beginPath();
                    ctx.moveTo(headX + 2, ey + 2 + walkBob);
                    ctx.lineTo(headX + 5, ey - 6 + walkBob);
                    ctx.lineTo(headX + 9, ey + 2 + walkBob);
                    ctx.fill();
                    ctx.fillStyle = '#f43f5e';
                    ctx.fillRect(e.facing === 1 ? headX + 10 : headX + 3, ey + 6 + walkBob, 3, 3);

                } else if (e.type === 'bat') {
                    ctx.fillStyle = '#581c87';
                    ctx.beginPath();
                    ctx.moveTo(ex + e.w / 2, ey + 10);
                    ctx.lineTo(ex - 10, ey + flap);
                    ctx.lineTo(ex + 4, ey + 18);
                    ctx.moveTo(ex + e.w / 2, ey + 10);
                    ctx.lineTo(ex + e.w + 10, ey + flap);
                    ctx.lineTo(ex + e.w - 4, ey + 18);
                    ctx.fill();

                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.arc(ex + e.w / 2, ey + 12, 11, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.fillStyle = '#facc15';
                    ctx.fillRect(ex + e.w / 2 - 5, ey + 9, 3, 3);
                    ctx.fillRect(ex + e.w / 2 + 2, ey + 9, 3, 3);

                } else if (e.type === 'spider') {
                    ctx.strokeStyle = '#38bdf8';
                    ctx.lineWidth = 2.5;
                    for (let leg = 0; leg < 3; leg++) {
                        const lx = ex + 8 + leg * 12;
                        ctx.beginPath();
                        ctx.moveTo(lx, ey + 14);
                        ctx.lineTo(lx - 6, ey + e.h + (leg % 2 === 0 ? walkBob : -walkBob));
                        ctx.moveTo(lx, ey + 14);
                        ctx.lineTo(lx + 6, ey + e.h - (leg % 2 === 0 ? walkBob : -walkBob));
                        ctx.stroke();
                    }
                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.ellipse(ex + e.w / 2, ey + 12, 18, 10, 0, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.fillStyle = '#f43f5e';
                    ctx.fillRect(ex + 12, ey + 10, 3, 3);
                    ctx.fillRect(ex + 19, ey + 10, 3, 3);
                    ctx.fillRect(ex + 26, ey + 10, 3, 3);

                } else if (e.type === 'ghost') {
                    ctx.globalAlpha = 0.85;
                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.roundRect(ex + 2, ey, e.w - 4, e.h - 6, [16, 16, 4, 4]);
                    ctx.fill();
                    ctx.fillStyle = '#0f172a';
                    ctx.fillRect(ex + 9, ey + 12, 4, 5);
                    ctx.fillRect(ex + 20, ey + 12, 4, 5);
                    ctx.globalAlpha = 1.0;

                } else if (e.type === 'dragon') {
                    ctx.fillStyle = '#9a3412';
                    ctx.beginPath();
                    ctx.moveTo(ex + e.w / 2, ey + 16);
                    ctx.lineTo(ex - 12, ey - 6 + flap);
                    ctx.lineTo(ex + 10, ey + 26);
                    ctx.moveTo(ex + e.w / 2, ey + 16);
                    ctx.lineTo(ex + e.w + 12, ey - 6 + flap);
                    ctx.lineTo(ex + e.w - 10, ey + 26);
                    ctx.fill();

                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.roundRect(ex + 6, ey + 6, e.w - 12, e.h - 10, 12);
                    ctx.fill();
                    ctx.strokeStyle = '#facc15';
                    ctx.lineWidth = 2.5;
                    ctx.stroke();

                    ctx.fillStyle = '#fef08a';
                    ctx.fillRect(ex + 14, ey - 2, 4, 8);
                    ctx.fillRect(ex + e.w - 18, ey - 2, 4, 8);
                    ctx.fillRect(ex + 16, ey + 14, 5, 4);
                    ctx.fillRect(ex + e.w - 21, ey + 14, 5, 4);

                } else {
                    ctx.fillStyle = mainColor;
                    ctx.beginPath();
                    ctx.roundRect(ex + 4, ey + 12 + walkBob, e.w - 8, e.h - 16, 6);
                    ctx.fill();
                    ctx.strokeStyle = e.border;
                    ctx.lineWidth = 2.5;
                    ctx.stroke();

                    ctx.fillStyle = isHurt ? '#ffffff' : e.border;
                    ctx.fillRect(ex + 7, ey + walkBob, e.w - 14, 14);

                    ctx.fillStyle = e.type === 'skeleton' ? '#0f172a' : '#facc15';
                    const lookOffset = e.facing === 1 ? 4 : -2;
                    ctx.fillRect(ex + e.w / 2 - 6 + lookOffset, ey + 5 + walkBob, 4, 4);
                    ctx.fillRect(ex + e.w / 2 + 2 + lookOffset, ey + 5 + walkBob, 4, 4);

                    ctx.fillStyle = '#facc15';
                    const weaponX = e.facing === 1 ? ex + e.w - 2 : ex - 6;
                    ctx.fillRect(weaponX, ey + 14 + walkBob, 8, 22);
                }

                const barW = Math.max(44, e.w + 8);
                const barX = ex + e.w / 2 - barW / 2;
                ctx.font = 'bold 10px "Plus Jakarta Sans", sans-serif';
                ctx.textAlign = 'center';
                ctx.fillStyle = '#fef08a';
                ctx.fillText(e.name, ex + e.w / 2, ey - 16);

                ctx.fillStyle = 'rgba(15, 23, 42, 0.85)';
                ctx.fillRect(barX, ey - 12, barW, 5);
                ctx.fillStyle = '#f43f5e';
                ctx.fillRect(barX, ey - 12, barW * Math.max(0, e.hp / e.maxHp), 5);

                ctx.restore();
            }
        }

        function drawDroppedItems() {
            for (const d of droppedItems) {
                const item = ITEMS[d.id];
                if (!item) continue;
                const dx = Math.floor(d.x - camera.x);
                const dy = Math.floor(d.y - camera.y + Math.sin(d.bob) * 4);

                ctx.fillStyle = item.color || '#fbbf24';
                ctx.beginPath();
                ctx.arc(dx, dy, 6, 0, Math.PI * 2);
                ctx.fill();
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 1.5;
                ctx.stroke();
            }
        }

        function drawParticlesAndText() {
            for (const p of particles) {
                ctx.fillStyle = p.color;
                ctx.globalAlpha = Math.max(0, p.life / p.maxLife);
                ctx.fillRect(p.x - camera.x, p.y - camera.y, p.size, p.size);
            }
            ctx.globalAlpha = 1.0;

            ctx.font = 'bold 13px "Plus Jakarta Sans", sans-serif';
            ctx.textAlign = 'center';
            for (const ft of floatingTexts) {
                ctx.globalAlpha = Math.min(1, ft.life * 1.4);
                ctx.fillStyle = '#0f172a';
                ctx.fillText(ft.text, ft.x - camera.x + 1, ft.y - camera.y + 1);
                ctx.fillStyle = ft.color;
                ctx.fillText(ft.text, ft.x - camera.x, ft.y - camera.y);
            }
            ctx.globalAlpha = 1.0;
        }

        function render() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawSkyAndBackdrop();
            drawWorld();
            drawPortalAndCompass();
            drawDroppedItems();
            drawEnemies();
            drawPlayer();
            drawParticlesAndText();
        }

        // --- 16. GELUNG PERMAINAN UTAMA ---
        let lastTime = 0;

        function gameLoop(timestamp) {
            if (!lastTime) lastTime = timestamp;
            const dt = Math.min(0.05, (timestamp - lastTime) / 1000);
            lastTime = timestamp;

            update(dt);
            render();

            requestAnimationFrame(gameLoop);
        }

        // --- 17. INPUT & PENGENDALI BUTANG UI ---
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }

        window.addEventListener('resize', resizeCanvas);

        window.addEventListener('keydown', (e) => {
            keys[e.code] = true;

            if (e.code >= 'Digit1' && e.code <= 'Digit6') {
                player.selectedSlot = parseInt(e.code.replace('Digit', ''), 10) - 1;
                renderHotbar();
            }

            if ((e.code === 'KeyE' || e.code === 'KeyC') && gameState.started && !gameState.gameOver && !gameState.quizOpen) {
                toggleCraftingModal();
            }

            if (e.code === 'Escape') {
                if (gameState.craftingOpen) toggleCraftingModal(false);
                if (gameState.helpOpen) toggleHelpModal(false);
                if (gameState.posterOpen) togglePosterModal(false);
            }
        });

        window.addEventListener('keyup', (e) => {
            keys[e.code] = false;
        });

        canvas.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            mouse.x = e.clientX - rect.left;
            mouse.y = e.clientY - rect.top;
        });

        canvas.addEventListener('mousedown', (e) => {
            if (!gameState.started || gameState.quizOpen) return;
            audio.init();
            if (e.button === 0) mouse.leftDown = true;
            if (e.button === 2) mouse.rightDown = true;
        });

        window.addEventListener('mouseup', (e) => {
            if (e.button === 0) mouse.leftDown = false;
            if (e.button === 2) mouse.rightDown = false;
        });

        canvas.addEventListener('contextmenu', (e) => e.preventDefault());

        window.addEventListener('wheel', (e) => {
            if (!gameState.started || gameState.craftingOpen || gameState.quizOpen) return;
            if (e.deltaY > 0) player.selectedSlot = (player.selectedSlot + 1) % 6;
            else player.selectedSlot = (player.selectedSlot + 5) % 6;
            renderHotbar();
        });

        // Kawalan Sentuh Mudah Alih
        const bindTouchBtn = (id, keyCode) => {
            const el = document.getElementById(id);
            el.addEventListener('touchstart', (e) => { e.preventDefault(); keys[keyCode] = true; });
            el.addEventListener('touchend', (e) => { e.preventDefault(); keys[keyCode] = false; });
        };
        bindTouchBtn('touchLeft', 'ArrowLeft');
        bindTouchBtn('touchRight', 'ArrowRight');
        bindTouchBtn('touchJump', 'Space');

        document.getElementById('touchAttackBtn').addEventListener('touchstart', (e) => {
            e.preventDefault();
            if (!gameState.started || gameState.quizOpen) return;
            audio.init();
            if (player.swingTimer <= 0) {
                player.swingTimer = player.swingDuration;
                audio.playSwing();
                checkMeleeAttack();
            }
        });

        document.getElementById('touchPotionBtn').addEventListener('click', () => {
            const potionIdx = player.inventory.findIndex(s => s && s.id === 'potion');
            if (potionIdx !== -1) {
                useConsumableIfClicked(potionIdx);
            } else {
                showToast('Tiada Ramuan Nyawa!');
            }
        });

        function toggleCraftingModal(forceState) {
            gameState.craftingOpen = forceState !== undefined ? forceState : !gameState.craftingOpen;
            const modal = document.getElementById('craftingModal');
            if (gameState.craftingOpen) {
                renderCraftingMenu();
                modal.classList.remove('hidden');
                modal.classList.add('flex');
            } else {
                modal.classList.add('hidden');
                modal.classList.remove('flex');
            }
        }

        function togglePosterModal(forceState) {
            gameState.posterOpen = forceState !== undefined ? forceState : !gameState.posterOpen;
            const modal = document.getElementById('posterModal');
            if (gameState.posterOpen) {
                modal.classList.remove('hidden');
                modal.classList.add('flex');
            } else {
                modal.classList.add('hidden');
                modal.classList.remove('flex');
            }
        }

        function toggleHelpModal(forceState) {
            gameState.helpOpen = forceState !== undefined ? forceState : !gameState.helpOpen;
            const modal = document.getElementById('helpModal');
            if (gameState.helpOpen) {
                modal.classList.remove('hidden');
                modal.classList.add('flex');
            } else {
                modal.classList.add('hidden');
                modal.classList.remove('flex');
            }
        }

        document.getElementById('btnPosterRef').addEventListener('click', () => togglePosterModal());
        document.getElementById('btnTitlePoster').addEventListener('click', () => togglePosterModal(true));
        document.getElementById('btnClosePoster').addEventListener('click', () => togglePosterModal(false));
        document.getElementById('btnGotItPoster').addEventListener('click', () => togglePosterModal(false));

        document.getElementById('btnCraftToggle').addEventListener('click', () => toggleCraftingModal());
        document.getElementById('btnCloseCraft').addEventListener('click', () => toggleCraftingModal(false));

        document.getElementById('btnHelp').addEventListener('click', () => toggleHelpModal());
        document.getElementById('btnCloseHelp').addEventListener('click', () => toggleHelpModal(false));
        document.getElementById('btnStartFromHelp').addEventListener('click', () => toggleHelpModal(false));

        document.getElementById('btnAudio').addEventListener('click', (e) => {
            const muted = audio.toggleMute();
            e.currentTarget.textContent = muted ? '🔇' : '🔊';
            showToast(muted ? 'Bunyi dimatikan' : 'Bunyi dihidupkan');
        });

        document.getElementById('btnRestartLevel').addEventListener('click', () => {
            if (!gameState.started) return;
            loadLevel(gameState.currentLevelIndex);
            showToast(`🔄 Mengulang Level ${gameState.currentLevelIndex + 1}...`);
        });

        function returnToMainMenu() {
            gameState.started = false;
            gameState.gameOver = false;
            gameState.quizOpen = false;
            gameState.craftingOpen = false;
            gameState.helpOpen = false;
            gameState.posterOpen = false;
            mouse.leftDown = false;
            mouse.rightDown = false;

            ['quizModal', 'craftingModal', 'helpModal', 'posterModal', 'gameOverOverlay', 'victoryOverlay'].forEach(id => {
                const el = document.getElementById(id);
                if (el) {
                    el.classList.add('hidden');
                    el.classList.remove('flex');
                }
            });

            gameState.score = 0;
            gameState.firstTryCorrectCount = 0;
            loadLevel(0);
            document.getElementById('titleOverlay').classList.remove('hidden');
        }

        document.getElementById('btnMainMenu').addEventListener('click', returnToMainMenu);
        document.getElementById('btnVictoryMainMenu').addEventListener('click', returnToMainMenu);
        document.getElementById('btnGameOverMainMenu').addEventListener('click', returnToMainMenu);

        document.getElementById('btnRespawn').addEventListener('click', () => {
            document.getElementById('gameOverOverlay').classList.add('hidden');
            document.getElementById('gameOverOverlay').classList.remove('flex');
            loadLevel(gameState.currentLevelIndex);
            showToast(`🔄 Bangkit semula di Level ${gameState.currentLevelIndex + 1}!`);
        });

        // Pengendali Butang Kuiz Akhir Level
        document.getElementById('btnQuizRetry').addEventListener('click', () => {
            const allBtns = document.querySelectorAll('.quiz-opt-btn');
            allBtns.forEach(b => {
                b.disabled = false;
                b.classList.remove('border-rose-500', 'bg-rose-950/75');
                b.classList.add('border-slate-700', 'bg-slate-800/85');
            });
            document.getElementById('quizFeedbackBox').classList.add('hidden');
            document.getElementById('btnQuizRetry').classList.add('hidden');
        });

        document.getElementById('btnQuizNextLevel').addEventListener('click', () => {
            const nextIdx = gameState.currentLevelIndex + 1;
            if (nextIdx >= LEVELS.length) {
                triggerVictoryScreen();
            } else {
                gameState.quizOpen = false;
                document.getElementById('quizModal').classList.add('hidden');
                document.getElementById('quizModal').classList.remove('flex');
                loadLevel(nextIdx);
                showToast(`🚩 Selamat datang ke Level ${nextIdx + 1}: ${LEVELS[nextIdx].title}!`);
            }
        });

        // Butang Mula Permainan & Main Semula Semua
        document.getElementById('btnPlayGame').addEventListener('click', () => {
            audio.init();
            document.getElementById('titleOverlay').classList.add('hidden');
            gameState.started = true;
            gameState.score = 0;
            gameState.firstTryCorrectCount = 0;
            loadLevel(0);
            showToast('⚔️ Misi bermula! Tewaskan Boss untuk membuka Portal Cahaya!');
        });

        document.getElementById('btnPlayAgainAll').addEventListener('click', () => {
            document.getElementById('victoryOverlay').classList.add('hidden');
            document.getElementById('victoryOverlay').classList.remove('flex');
            gameState.started = true;
            gameState.score = 0;
            gameState.firstTryCorrectCount = 0;
            loadLevel(0);
        });

        // --- 18. PERMULAAN APLIKASI ---
        window.onload = function () {
            resizeCanvas();
            generateTileTextures();
            loadLevel(0);
            requestAnimationFrame(gameLoop);
        };
    </script>
</body>
</html>
