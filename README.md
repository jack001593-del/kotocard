<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>KotoCard — 日文單字卡 &amp; Gemini 拍照自動匯入</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+JP:wght@400;500;700&family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet" />
  <style>
    body {
      font-family: 'Inter', 'Noto Sans TC', 'Noto Sans JP', system-ui, -apple-system, sans-serif;
      background-color: #f4f6fb;
      color: #0f172a;
      -webkit-tap-highlight-color: transparent;
    }
    .jp-text {
      font-family: 'Noto Sans JP', 'Inter', sans-serif;
    }
    .perspective-1000 {
      perspective: 1000px;
    }
    .card-inner {
      position: relative;
      width: 100%;
      height: 100%;
      transition: transform 0.52s cubic-bezier(0.22, 1, 0.36, 1);
      transform-style: preserve-3d;
    }
    .card-inner.is-flipped {
      transform: rotateY(180deg);
    }
    .card-face {
      position: absolute;
      inset: 0;
      backface-visibility: hidden;
      -webkit-backface-visibility: hidden;
    }
    .card-face-back {
      transform: rotateY(180deg);
    }
    @keyframes popIn {
      0% { opacity: 0; transform: translateY(8px) scale(0.97); }
      100% { opacity: 1; transform: translateY(0) scale(1); }
    }
    .animate-pop {
      animation: popIn 0.22s ease-out forwards;
    }
  </style>
</head>
<body class="min-h-screen flex flex-col">
  <!-- Top Header -->
  <header class="bg-white border-b border-slate-200 sticky top-0 z-30">
    <div class="max-w-4xl mx-auto px-3 sm:px-6 h-14 flex items-center justify-between gap-2">
      <div class="flex items-center gap-2 min-w-0">
        <div class="w-8 h-8 rounded-lg bg-indigo-600 text-white flex items-center justify-center font-bold text-sm shrink-0 shadow-sm">
          言
        </div>
        <div class="min-w-0">
          <h1 class="font-bold text-slate-900 text-sm sm:text-base truncate">KotoCard 日文單字</h1>
        </div>
      </div>

      <div class="flex items-center gap-1.5 sm:gap-2 shrink-0">
        <select id="deckSelect" aria-label="選擇單字本" class="text-xs sm:text-sm bg-slate-100 hover:bg-slate-200 text-slate-800 font-medium rounded-lg px-2.5 py-1.5 border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 max-w-[130px] sm:max-w-[200px] truncate">
        </select>
        <button id="openImportBtn" type="button" class="inline-flex items-center gap-1 bg-indigo-600 hover:bg-indigo-700 active:scale-[0.98] text-white text-xs sm:text-sm font-medium px-2.5 sm:px-3 py-1.5 rounded-lg transition shadow-sm">
          <span>＋</span>
          <span class="hidden xs:inline sm:inline">匯入</span>
        </button>
        <button id="openGemBtn" type="button" class="inline-flex items-center gap-1 bg-amber-50 hover:bg-amber-100 text-amber-800 border border-amber-200 text-xs sm:text-sm font-medium px-2.5 py-1.5 rounded-lg transition">
          <span>✨</span>
          <span class="hidden sm:inline">Gemini 設定</span>
        </button>
      </div>
    </div>

    <!-- Mode Tabs -->
    <nav class="max-w-4xl mx-auto px-3 sm:px-6 flex items-center gap-1 border-t border-slate-100 overflow-x-auto">
      <button data-mode="flashcards" class="mode-tab flex-1 sm:flex-none px-3 sm:px-4 py-2.5 text-xs sm:text-sm font-semibold border-b-2 transition whitespace-nowrap">
        單字卡
      </button>
      <button data-mode="quiz" class="mode-tab flex-1 sm:flex-none px-3 sm:px-4 py-2.5 text-xs sm:text-sm font-semibold border-b-2 transition whitespace-nowrap">
        測驗
      </button>
      <button data-mode="match" class="mode-tab flex-1 sm:flex-none px-3 sm:px-4 py-2.5 text-xs sm:text-sm font-semibold border-b-2 transition whitespace-nowrap">
        配對
      </button>
      <button data-mode="list" class="mode-tab flex-1 sm:flex-none px-3 sm:px-4 py-2.5 text-xs sm:text-sm font-semibold border-b-2 transition whitespace-nowrap">
        單字庫 <span id="wordCountBadge" class="ml-1 text-xs px-1.5 py-0.5 rounded-full bg-slate-100 text-slate-600">0</span>
      </button>
    </nav>
  </header>

  <!-- Main Content -->
  <main class="flex-1 max-w-4xl w-full mx-auto px-3 sm:px-6 py-5 sm:py-7">
    <!-- Progress Summary Strip -->
    <div id="statsBar" class="mb-5 bg-white rounded-xl border border-slate-200 p-3 sm:p-4 flex flex-wrap items-center justify-between gap-3 shadow-sm">
      <div class="flex items-center gap-4 text-xs sm:text-sm">
        <div>
          <span class="text-slate-500">總計</span>
          <span id="statTotal" class="font-bold text-slate-900 ml-1">0</span>
        </div>
        <div>
          <span class="text-emerald-600 font-medium">已掌握</span>
          <span id="statMastered" class="font-bold text-emerald-700 ml-1">0</span>
        </div>
        <div>
          <span class="text-amber-600 font-medium">學習中</span>
          <span id="statLearning" class="font-bold text-amber-700 ml-1">0</span>
        </div>
      </div>
      <div class="flex-1 min-w-[140px] max-w-xs flex items-center gap-2">
        <div class="w-full bg-slate-100 h-2 rounded-full overflow-hidden">
          <div id="statProgressBar" class="bg-emerald-500 h-full transition-all duration-300" style="width: 0%"></div>
        </div>
        <span id="statPercent" class="text-xs font-semibold text-slate-600 w-9 text-right">0%</span>
      </div>
    </div>

    <!-- VIEW: FLASHCARDS -->
    <section id="view-flashcards" class="space-y-4">
      <div class="flex flex-wrap items-center justify-between gap-2">
        <div class="flex items-center gap-1.5">
          <button id="fcFilterAll" type="button" class="fc-filter-btn px-2.5 py-1 rounded-lg text-xs font-medium border transition">全部</button>
          <button id="fcFilterLearning" type="button" class="fc-filter-btn px-2.5 py-1 rounded-lg text-xs font-medium border transition">只看學習中</button>
        </div>
        <div class="flex items-center gap-2">
          <button id="fcShuffleBtn" type="button" class="px-2.5 py-1 rounded-lg text-xs font-medium bg-white border border-slate-200 hover:bg-slate-50 text-slate-700 transition">
            🔀 隨機排序
          </button>
          <span id="fcCounter" class="text-xs sm:text-sm font-semibold text-slate-600">0 / 0</span>
        </div>
      </div>

      <!-- Flashcard Container -->
      <div id="fcEmptyState" class="hidden bg-white rounded-2xl border border-slate-200 p-10 text-center space-y-3">
        <p class="text-slate-600 font-medium">目前篩選條件下沒有單字</p>
        <button id="fcResetFilterBtn" type="button" class="px-4 py-2 bg-indigo-600 text-white text-sm font-medium rounded-lg hover:bg-indigo-700 transition">顯示全部單字</button>
      </div>

      <div id="fcStage" class="perspective-1000 h-72 sm:h-80 cursor-pointer select-none">
        <div id="fcCardInner" class="card-inner">
          <!-- Front -->
          <div class="card-face bg-white rounded-2xl border border-slate-200 shadow-md p-6 flex flex-col justify-between">
            <div class="flex items-center justify-between">
              <div class="flex items-center gap-1.5">
                <span id="fcLevelBadge" class="text-xs font-semibold px-2 py-0.5 rounded-md bg-indigo-50 text-indigo-700">N3</span>
                <span id="fcPosBadge" class="text-xs font-medium px-2 py-0.5 rounded-md bg-slate-100 text-slate-600">名詞</span>
              </div>
              <button id="fcSpeakFront" type="button" aria-label="朗讀日文發音" class="w-9 h-9 rounded-full bg-slate-100 hover:bg-indigo-100 text-slate-700 hover:text-indigo-700 flex items-center justify-center transition">
                🔊
              </button>
            </div>

            <div class="text-center my-auto space-y-2">
              <div id="fcWord" class="jp-text text-4xl sm:text-5xl font-bold text-slate-900 tracking-wide"></div>
              <div id="fcReadingHint" class="jp-text text-base sm:text-lg text-slate-400"></div>
            </div>

            <div class="flex items-center justify-between text-xs text-slate-400">
              <span>點擊卡片翻面</span>
              <span id="fcStatusFront" class="font-medium"></span>
            </div>
          </div>

          <!-- Back -->
          <div class="card-face card-face-back bg-indigo-950 text-white rounded-2xl border border-indigo-900 shadow-md p-6 flex flex-col justify-between">
            <div class="flex items-center justify-between">
              <span id="fcWordBack" class="jp-text text-lg font-bold text-indigo-200"></span>
              <button id="fcSpeakBack" type="button" aria-label="朗讀日文發音" class="w-9 h-9 rounded-full bg-indigo-800 hover:bg-indigo-700 text-white flex items-center justify-center transition">
                🔊
              </button>
            </div>

            <div class="text-center my-auto space-y-3 px-2">
              <div id="fcReading" class="jp-text text-base sm:text-lg text-indigo-300 font-medium"></div>
              <div class="flex items-center justify-center gap-2 flex-wrap">
                <span id="fcPosBackBadge" class="hidden text-xs sm:text-sm font-bold px-2 py-0.5 rounded-md bg-indigo-800 text-indigo-100 border border-indigo-600"></span>
                <span id="fcMeaning" class="text-2xl sm:text-3xl font-bold text-white"></span>
              </div>
              <div id="fcExample" class="jp-text text-xs sm:text-sm text-indigo-200 bg-indigo-900/60 rounded-xl px-3.5 py-2.5 leading-relaxed"></div>
            </div>

            <div class="text-center text-xs text-indigo-300/70">點擊翻回正面</div>
          </div>
        </div>
      </div>

      <!-- Flashcard Controls -->
      <div id="fcControls" class="flex items-center justify-between gap-2 pt-1">
        <button id="fcPrevBtn" type="button" class="px-4 py-2.5 rounded-xl bg-white border border-slate-200 hover:bg-slate-50 font-semibold text-sm text-slate-700 shadow-sm transition">
          ← 上一個
        </button>

        <button id="fcToggleMasteredBtn" type="button" class="flex-1 max-w-xs px-4 py-2.5 rounded-xl font-semibold text-sm border shadow-sm transition">
          標記為已掌握
        </button>

        <button id="fcNextBtn" type="button" class="px-4 py-2.5 rounded-xl bg-indigo-600 hover:bg-indigo-700 font-semibold text-sm text-white shadow-sm transition">
          下一個 →
        </button>
      </div>
    </section>

    <!-- VIEW: QUIZ -->
    <section id="view-quiz" class="hidden space-y-4">
      <div id="quizContainer" class="bg-white rounded-2xl border border-slate-200 shadow-sm p-5 sm:p-7 space-y-5">
        <div class="flex items-center justify-between">
          <span id="quizProgress" class="text-xs sm:text-sm font-semibold text-indigo-600">第 1 / 5 題</span>
          <span id="quizScoreBadge" class="text-xs sm:text-sm font-semibold text-slate-600">得分：0</span>
        </div>

        <div class="py-3 text-center space-y-2 border-y border-slate-100">
          <div class="flex items-center justify-center gap-2">
            <span id="quizWord" class="jp-text text-3xl sm:text-4xl font-bold text-slate-900"></span>
            <button id="quizSpeakBtn" type="button" aria-label="朗讀題目" class="w-8 h-8 rounded-full bg-slate-100 hover:bg-indigo-100 text-slate-700 flex items-center justify-center text-sm transition">🔊</button>
          </div>
          <div id="quizReading" class="jp-text text-sm text-slate-500"></div>
        </div>

        <div id="quizOptions" class="grid grid-cols-1 sm:grid-cols-2 gap-2.5"></div>

        <div id="quizFeedback" class="hidden rounded-xl p-3.5 text-sm flex items-center justify-between gap-3">
          <span id="quizFeedbackText" class="font-medium"></span>
          <button id="quizNextBtn" type="button" class="px-4 py-1.5 rounded-lg bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-xs sm:text-sm shrink-0 transition">
            下一題 →
          </button>
        </div>
      </div>

      <div id="quizResult" class="hidden bg-white rounded-2xl border border-slate-200 shadow-sm p-7 text-center space-y-4">
        <div class="text-4xl">🎯</div>
        <h2 class="text-xl font-bold text-slate-900">測驗完成！</h2>
        <p id="quizResultSummary" class="text-slate-600 text-sm sm:text-base"></p>
        <div class="flex flex-wrap items-center justify-center gap-2 pt-2">
          <button id="quizRestartBtn" type="button" class="px-5 py-2.5 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-sm transition">
            再測一次
          </button>
          <button id="quizWrongOnlyBtn" type="button" class="hidden px-5 py-2.5 rounded-xl bg-amber-100 hover:bg-amber-200 text-amber-900 font-semibold text-sm transition">
            只複習錯題
          </button>
        </div>
      </div>
    </section>

    <!-- VIEW: MATCH GAME -->
    <section id="view-match" class="hidden space-y-4">
      <div class="bg-white rounded-xl border border-slate-200 p-3.5 flex items-center justify-between">
        <div class="text-xs sm:text-sm text-slate-600">
          點選配對「日文單字」與「中文意思」
        </div>
        <div class="flex items-center gap-3">
          <span id="matchTimer" class="font-mono font-bold text-indigo-600 text-sm sm:text-base">0.0s</span>
          <button id="matchRestartBtn" type="button" class="px-3 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-800 text-xs font-semibold transition">
            重新開始
          </button>
        </div>
      </div>

      <div id="matchGrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-2.5 sm:gap-3"></div>

      <div id="matchDone" class="hidden bg-white rounded-2xl border border-slate-200 p-7 text-center space-y-3">
        <div class="text-4xl">⚡</div>
        <h2 class="text-xl font-bold text-slate-900">全部配對完成！</h2>
        <p id="matchDoneText" class="text-slate-600 text-sm"></p>
        <button id="matchPlayAgainBtn" type="button" class="px-5 py-2.5 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-sm transition">
          再玩一局
        </button>
      </div>
    </section>

    <!-- VIEW: WORD LIST -->
    <section id="view-list" class="hidden space-y-4">
      <div class="bg-white rounded-xl border border-slate-200 p-3 sm:p-4 flex flex-wrap items-center justify-between gap-2.5">
        <input id="listSearchInput" type="search" placeholder="搜尋日文、假名或中文意思…" class="flex-1 min-w-[180px] text-sm px-3 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500" />
        <div class="flex items-center gap-2">
          <select id="listFilterSelect" class="text-xs sm:text-sm px-2.5 py-2 rounded-lg border border-slate-200 bg-slate-50 font-medium">
            <option value="all">全部狀態</option>
            <option value="learning">學習中</option>
            <option value="mastered">已掌握</option>
          </select>
          <button id="exportJsonBtn" type="button" class="px-3 py-2 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-semibold transition">
            匯出 JSON
          </button>
        </div>
      </div>

      <div id="wordListContainer" class="space-y-2.5"></div>
    </section>
  </main>

  <!-- MODAL: IMPORT / ADD WORDS -->
  <div id="importModal" class="hidden fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-3 sm:p-4">
    <div class="bg-white rounded-2xl max-w-lg w-full max-h-[90vh] overflow-y-auto shadow-xl border border-slate-200 p-5 sm:p-6 space-y-4 animate-pop">
      <div class="flex items-center justify-between">
        <h2 class="text-lg font-bold text-slate-900">匯入 / 新增日文單字</h2>
        <button id="closeImportModal" type="button" class="w-8 h-8 rounded-full hover:bg-slate-100 text-slate-500 flex items-center justify-center">✕</button>
      </div>

      <div class="flex flex-wrap gap-1.5 border-b border-slate-100 pb-2">
        <button id="tabImageImport" type="button" class="px-3 py-1.5 rounded-lg text-xs sm:text-sm font-semibold bg-indigo-50 text-indigo-700">📸 截圖辨識</button>
        <button id="tabBatchImport" type="button" class="px-3 py-1.5 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100">批次匯入 (Gemini 輸出)</button>
        <button id="tabSingleAdd" type="button" class="px-3 py-1.5 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100">手動新增單筆</button>
        <button id="tabNewDeck" type="button" class="px-3 py-1.5 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100">建立新單字本</button>
      </div>

      <!-- Image OCR Import Panel -->
      <div id="panelImageImport" class="space-y-3">
        <p class="text-xs text-slate-600 leading-relaxed">
          直接上傳單字書照片或截圖，系統會透過 Gemini Vision 自動辨識有標記的單字並附上 <strong>【名】【動】【副】</strong> 詞性匯入（或直接在手機 Gemini App 傳圖更省設定）。
        </p>
        <div>
          <label class="block text-xs font-semibold text-slate-700 mb-1">1. 選擇照片或截圖 *</label>
          <input id="imageUploadInput" type="file" accept="image/*" class="w-full text-xs text-slate-600 file:mr-3 file:py-1.5 file:px-3 file:rounded-lg file:border-0 file:text-xs file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100" />
          <img id="imagePreviewEl" alt="預覽圖片" class="hidden mt-2 max-h-40 rounded-lg border border-slate-200 object-contain mx-auto" />
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-700 mb-1">2. 擷取條件</label>
          <input id="imageCriteriaInput" type="text" value="只擷取旁邊有紅點、畫線、螢光筆或打勾標記的單字；若整張圖都沒有標記，則擷取全部單字" class="w-full text-xs px-3 py-2 rounded-lg border border-slate-200" />
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-700 mb-1">3. Gemini API Key（僅存於本機瀏覽器）</label>
          <input id="geminiApiKeyInput" type="password" placeholder="AIza..." class="w-full text-xs px-3 py-2 rounded-lg border border-slate-200 font-mono" />
        </div>
        <div id="imageImportStatus" class="hidden text-xs rounded-lg p-2.5"></div>
        <div class="flex justify-end">
          <button id="runImageImportBtn" type="button" class="px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold transition">
            開始辨識並匯入
          </button>
        </div>
      </div>

      <!-- Batch Import Panel -->
      <div id="panelBatchImport" class="space-y-3">
        <p class="text-xs text-slate-600 leading-relaxed">
          貼上 Gemini 辨識照片後輸出的 <strong>JSON 陣列</strong>，或每行一筆的簡易格式（<code>單字|讀音|中文|例句</code>）：
        </p>
        <textarea id="batchImportTextarea" rows="6" placeholder='[{"word":"把握","reading":"はあく","meaning":"掌握、理解","pos":"名詞・スル","level":"N2","example":"状況を正確に把握する。"}]' class="w-full text-xs sm:text-sm font-mono p-3 rounded-xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500"></textarea>
        <div class="flex justify-end gap-2">
          <button id="runBatchImportBtn" type="button" class="px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold transition">
            匯入至目前單字本
          </button>
        </div>
      </div>

      <!-- Single Add Panel -->
      <form id="panelSingleAdd" class="hidden space-y-3">
        <div class="grid grid-cols-2 gap-2.5">
          <div>
            <label class="block text-xs font-semibold text-slate-600 mb-1">日文單字 *</label>
            <input id="addWord" required type="text" placeholder="例：見直す" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
          </div>
          <div>
            <label class="block text-xs font-semibold text-slate-600 mb-1">讀音 (假名)</label>
            <input id="addReading" type="text" placeholder="例：みなおす" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
          </div>
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-600 mb-1">中文意思 *</label>
          <input id="addMeaning" required type="text" placeholder="例：重新審視、改善" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
        </div>
        <div class="grid grid-cols-2 gap-2.5">
          <div>
            <label class="block text-xs font-semibold text-slate-600 mb-1">詞性</label>
            <input id="addPos" type="text" placeholder="例：動詞" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
          </div>
          <div>
            <label class="block text-xs font-semibold text-slate-600 mb-1">級別 / 標籤</label>
            <input id="addLevel" type="text" placeholder="例：N2" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
          </div>
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-600 mb-1">例句</label>
          <input id="addExample" type="text" placeholder="例：計画をもう一度見直す。" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
        </div>
        <div class="flex justify-end">
          <button type="submit" class="px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold transition">
            加入單字本
          </button>
        </div>
      </form>

      <!-- New Deck Panel -->
      <form id="panelNewDeck" class="hidden space-y-3">
        <div>
          <label class="block text-xs font-semibold text-slate-600 mb-1">新單字本名稱 *</label>
          <input id="newDeckName" required type="text" placeholder="例：日劇生詞 / 商業日文" class="w-full text-sm px-3 py-2 rounded-lg border border-slate-200" />
        </div>
        <div class="flex justify-end">
          <button type="submit" class="px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold transition">
            建立並切換
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- MODAL: GEMINI GEM SETUP -->
  <div id="gemModal" class="hidden fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-3 sm:p-4">
    <div class="bg-white rounded-2xl max-w-xl w-full max-h-[90vh] overflow-y-auto shadow-xl border border-slate-200 p-5 sm:p-6 space-y-4 animate-pop">
      <div class="flex items-center justify-between">
        <h2 class="text-lg font-bold text-slate-900">✨ 手機 Gemini 一鍵啟動 &amp; 拍照自動匯入設定</h2>
        <button id="closeGemModal" type="button" class="w-8 h-8 rounded-full hover:bg-slate-100 text-slate-500 flex items-center justify-center">✕</button>
      </div>

      <div class="bg-amber-50 border border-amber-200 rounded-xl p-3.5 text-xs text-amber-900 space-y-1.5 leading-relaxed">
        <p class="font-semibold">運作方式（3 步驟完成）：</p>
        <ol class="list-decimal list-inside space-y-1">
          <li>將此單一 HTML 檔免費部署到 GitHub Pages、Netlify Drop 或 Vercel，取得固定網址。</li>
          <li>在 Gemini 建立一個 <strong>Gem</strong>，貼上下方產生的系統指令。</li>
          <li>以後在手機打開 Gemini 拍照，Gem 會篩選符合條件的日文單字並產生<strong>「一鍵啟動並匯入」連結</strong>，點擊即可直接打開本網頁並自動存入單字卡！</li>
        </ol>
      </div>

      <div class="space-y-2.5">
        <div>
          <label class="block text-xs font-semibold text-slate-700 mb-1">你的網頁部署網址（App URL）</label>
          <input id="gemAppUrlInput" type="url" placeholder="https://yourname.github.io/kotocard/" class="w-full text-xs sm:text-sm px-3 py-2 rounded-lg border border-slate-200" />
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-700 mb-1">單字篩選條件（符合才匯入）</label>
          <input id="gemCriteriaInput" type="text" value="只挑選照片中有紅點、畫線、螢光筆或打勾標記的單字；若無標記則挑選值得學習的實詞" class="w-full text-xs sm:text-sm px-3 py-2 rounded-lg border border-slate-200" />
        </div>
      </div>

      <div class="space-y-1.5">
        <div class="flex items-center justify-between">
          <label class="text-xs font-semibold text-slate-700">Gem 系統指令（點擊複製）</label>
          <button id="copyGemPromptBtn" type="button" class="px-3 py-1 rounded-lg bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-semibold transition">
            複製指令
          </button>
        </div>
        <pre id="gemPromptPreview" class="text-xs bg-slate-900 text-slate-100 p-3.5 rounded-xl overflow-x-auto whitespace-pre-wrap leading-relaxed font-mono max-h-52"></pre>
      </div>
    </div>
  </div>

  <!-- TOAST NOTIFICATION -->
  <div id="toast" class="hidden fixed bottom-5 left-1/2 -translate-x-1/2 z-50 bg-slate-900 text-white text-xs sm:text-sm font-medium px-4 py-2.5 rounded-full shadow-lg transition">
  </div>

  <script>
    (function () {
      const STORAGE_KEY = 'kotocard_decks_v1';

      const DEFAULT_DECKS = [
        {
          id: 'deck_default',
          name: '精選日文核心單字',
          words: [
            { id: 'w1', word: '把握', reading: 'はあく', meaning: '掌握、確實理解', pos: '名詞・スル', level: 'N2', example: '全体の状況を正確に把握する。', mastered: false },
            { id: 'w2', word: '見直す', reading: 'みなおす', meaning: '重新審視、改善', pos: '動詞', level: 'N2', example: '今の生活習慣を見直す必要がある。', mastered: false },
            { id: 'w3', word: '曖昧', reading: 'あいまい', meaning: '模糊不清、曖昧', pos: 'な形容詞', level: 'N2', example: '曖昧な返事は避けたほうがいい。', mastered: false },
            { id: 'w4', word: '手配', reading: 'てはい', meaning: '安排、籌備、預訂', pos: '名詞・スル', level: 'N2', example: '出張の航空券とホテルを手配した。', mastered: true },
            { id: 'w5', word: 'きっかけ', reading: 'きっかけ', meaning: '契機、起點、導火線', pos: '名詞', level: 'N3', example: '日本旅行が日本語を学ぶきっかけになった。', mastered: false },
            { id: 'w6', word: '相変わらず', reading: 'あいかわらず', meaning: '依然、照舊', pos: '副詞', level: 'N3', example: '彼は相変わらず毎日忙しそうだ。', mastered: false },
            { id: 'w7', word: '検討', reading: 'けんとう', meaning: '研討、考量、評估', pos: '名詞・スル', level: 'N3', example: '新しい提案について社内で検討します。', mastered: false },
            { id: 'w8', word: '充実', reading: 'じゅうじつ', meaning: '充實、豐富', pos: '名詞・スル', level: 'N3', example: 'とても充実した週末を過ごせた。', mastered: true }
          ]
        }
      ];

      let state = {
        decks: [],
        activeDeckId: 'deck_default',
        mode: 'flashcards',
        fcFilter: 'all',
        fcIndex: 0,
        fcFlipped: false,
        fcOrder: [],
        quiz: {
          questions: [],
          idx: 0,
          score: 0,
          answered: false,
          wrongIds: []
        },
        match: {
          cards: [],
          selected: [],
          matchedIds: new Set(),
          startTime: 0,
          timerId: null,
          finished: false
        }
      };

      // --- Storage Helpers ---
      function loadState() {
        try {
          const raw = localStorage.getItem(STORAGE_KEY);
          if (raw) {
            const parsed = JSON.parse(raw);
            if (Array.isArray(parsed.decks) && parsed.decks.length > 0) {
              state.decks = parsed.decks;
              state.activeDeckId = parsed.activeDeckId || parsed.decks[0].id;
              return;
            }
          }
        } catch (e) {}
        state.decks = JSON.parse(JSON.stringify(DEFAULT_DECKS));
        state.activeDeckId = state.decks[0].id;
      }

      function saveState() {
        try {
          localStorage.setItem(STORAGE_KEY, JSON.stringify({
            decks: state.decks,
            activeDeckId: state.activeDeckId
          }));
        } catch (e) {}
      }

      function formatPosShort(pos) {
        if (!pos) return '';
        const p = String(pos).trim();
        if (p.includes('名')) return '名';
        if (p.includes('動')) return '動';
        if (p.includes('副')) return '副';
        if (p.includes('な形') || p.includes('形容動')) return 'な形';
        if (p.includes('い形') || p.includes('形')) return '形';
        if (p.includes('接')) return '接';
        if (p.includes('助')) return '助';
        return p.slice(0, 3);
      }

      function getActiveDeck() {
        return state.decks.find(d => d.id === state.activeDeckId) || state.decks[0];
      }

      function showToast(msg) {
        const el = document.getElementById('toast');
        el.textContent = msg;
        el.classList.remove('hidden');
        clearTimeout(el._tid);
        el._tid = setTimeout(() => el.classList.add('hidden'), 3200);
      }

      function speakJapanese(text) {
        if (!text || !('speechSynthesis' in window)) {
          showToast('此瀏覽器不支援語音朗讀');
          return;
        }
        try {
          window.speechSynthesis.cancel();
          const utter = new SpeechSynthesisUtterance(text);
          utter.lang = 'ja-JP';
          utter.rate = 0.92;
          window.speechSynthesis.speak(utter);
        } catch (e) {
          showToast('無法播放語音');
        }
      }

      // --- Import Parsing & URL Auto-Import ---
      function parseImportText(rawText) {
        const trimmed = (rawText || '').trim();
        if (!trimmed) return [];
        // Try JSON first
        if (trimmed.startsWith('[') || trimmed.startsWith('{')) {
          try {
            const parsed = JSON.parse(trimmed);
            const arr = Array.isArray(parsed) ? parsed : [parsed];
            return arr
              .filter(item => item && (item.word || item.kanji || item.term))
              .map(item => ({
                word: String(item.word || item.kanji || item.term).trim(),
                reading: String(item.reading || item.kana || item.furigana || '').trim(),
                meaning: String(item.meaning || item.zh || item.definition || '').trim() || '（未填寫解釋）',
                pos: String(item.pos || item.type || '').trim(),
                level: String(item.level || item.tag || '匯入').trim(),
                example: String(item.example || item.sentence || '').trim()
              }));
          } catch (e) {}
        }
        // Fallback: line-by-line pipe/comma/tab separated
        const lines = trimmed.split(/\r?\n/).map(l => l.trim()).filter(Boolean);
        const items = [];
        for (const line of lines) {
          const parts = line.includes('|') ? line.split('|') : line.includes('\t') ? line.split('\t') : line.split(',');
          if (parts.length >= 2) {
            items.push({
              word: parts[0].trim(),
              reading: (parts[1] || '').trim(),
              meaning: (parts[2] || parts[1] || '').trim(),
              pos: (parts[3] || '').trim(),
              level: (parts[4] || '匯入').trim(),
              example: (parts[5] || '').trim()
            });
          }
        }
        return items;
      }

      function addWordsToDeck(items) {
        if (!items || items.length === 0) return 0;
        const deck = getActiveDeck();
        const existingSet = new Set(deck.words.map(w => w.word));
        let added = 0;
        for (const item of items) {
          if (!item.word || existingSet.has(item.word)) continue;
          existingSet.add(item.word);
          deck.words.unshift({
            id: 'w_' + Date.now() + '_' + Math.random().toString(36).slice(2, 7),
            word: item.word,
            reading: item.reading || '',
            meaning: item.meaning || '（未填寫）',
            pos: item.pos || '',
            level: item.level || '匯入',
            example: item.example || '',
            mastered: false
          });
          added++;
        }
        if (added > 0) {
          saveState();
          rebuildFlashcardOrder();
        }
        return added;
      }

      function decodeBase64Utf8(str) {
        const bin = atob(str.replace(/-/g, '+').replace(/_/g, '/'));
        const bytes = Uint8Array.from(bin, c => c.charCodeAt(0));
        return new TextDecoder().decode(bytes);
      }

      function checkUrlAutoImport() {
        try {
          const params = new URLSearchParams(window.location.search);
          let payload = params.get('import') || params.get('words');
          if (!payload && window.location.hash.startsWith('#import=')) {
            payload = decodeURIComponent(window.location.hash.slice('#import='.length));
          }
          if (!payload) return;

          let decoded = payload;
          if (!payload.trim().startsWith('[') && !payload.includes('|')) {
            try {
              decoded = decodeBase64Utf8(payload);
            } catch (e) {}
          }
          const items = parseImportText(decoded);
          const count = addWordsToDeck(items);
          if (count > 0) {
            showToast('已從 Gemini 自動匯入 ' + count + ' 個新單字！');
          } else if (items.length > 0) {
            showToast('單字已存在於目前單字本中');
          }
          if (window.history && window.history.replaceState) {
            const cleanUrl = window.location.pathname + window.location.hash.replace(/^#import=.*/, '');
            window.history.replaceState({}, document.title, cleanUrl);
          }
        } catch (e) {}
      }

      // --- Render Stats & Deck Selector ---
      function renderHeaderAndStats() {
        const deckSelect = document.getElementById('deckSelect');
        deckSelect.innerHTML = '';
        state.decks.forEach(d => {
          const opt = document.createElement('option');
          opt.value = d.id;
          opt.textContent = d.name + ' (' + d.words.length + ')';
          if (d.id === state.activeDeckId) opt.selected = true;
          deckSelect.appendChild(opt);
        });

        const deck = getActiveDeck();
        const total = deck.words.length;
        const mastered = deck.words.filter(w => w.mastered).length;
        const learning = total - mastered;
        const pct = total > 0 ? Math.round((mastered / total) * 100) : 0;

        document.getElementById('wordCountBadge').textContent = total;
        document.getElementById('statTotal').textContent = total;
        document.getElementById('statMastered').textContent = mastered;
        document.getElementById('statLearning').textContent = learning;
        document.getElementById('statProgressBar').style.width = pct + '%';
        document.getElementById('statPercent').textContent = pct + '%';
      }

      // --- Flashcards ---
      function getFilteredWords() {
        const deck = getActiveDeck();
        if (state.fcFilter === 'learning') {
          return deck.words.filter(w => !w.mastered);
        }
        return deck.words;
      }

      function rebuildFlashcardOrder() {
        const words = getFilteredWords();
        state.fcOrder = words.map(w => w.id);
        state.fcIndex = 0;
        state.fcFlipped = false;
      }

      function renderFlashcards() {
        const deck = getActiveDeck();
        const words = getFilteredWords();
        // Keep fcOrder synced with valid IDs
        const validIds = new Set(words.map(w => w.id));
        state.fcOrder = state.fcOrder.filter(id => validIds.has(id));
        words.forEach(w => {
          if (!state.fcOrder.includes(w.id)) state.fcOrder.push(w.id);
        });

        const allBtn = document.getElementById('fcFilterAll');
        const learnBtn = document.getElementById('fcFilterLearning');
        allBtn.className = 'fc-filter-btn px-2.5 py-1 rounded-lg text-xs font-medium border transition ' +
          (state.fcFilter === 'all' ? 'bg-indigo-600 text-white border-indigo-600' : 'bg-white text-slate-700 border-slate-200');
        learnBtn.className = 'fc-filter-btn px-2.5 py-1 rounded-lg text-xs font-medium border transition ' +
          (state.fcFilter === 'learning' ? 'bg-amber-500 text-white border-amber-500' : 'bg-white text-slate-700 border-slate-200');

        const emptyEl = document.getElementById('fcEmptyState');
        const stageEl = document.getElementById('fcStage');
        const ctrlEl = document.getElementById('fcControls');

        if (state.fcOrder.length === 0) {
          emptyEl.classList.remove('hidden');
          stageEl.classList.add('hidden');
          ctrlEl.classList.add('hidden');
          document.getElementById('fcCounter').textContent = '0 / 0';
          return;
        }

        emptyEl.classList.add('hidden');
        stageEl.classList.remove('hidden');
        ctrlEl.classList.remove('hidden');

        if (state.fcIndex >= state.fcOrder.length) state.fcIndex = 0;
        if (state.fcIndex < 0) state.fcIndex = state.fcOrder.length - 1;

        const currentId = state.fcOrder[state.fcIndex];
        const card = deck.words.find(w => w.id === currentId);

        document.getElementById('fcCounter').textContent = (state.fcIndex + 1) + ' / ' + state.fcOrder.length;
        document.getElementById('fcCardInner').classList.toggle('is-flipped', state.fcFlipped);

        document.getElementById('fcLevelBadge').textContent = card.level || '日文';
        document.getElementById('fcPosBadge').textContent = card.pos || '單字';
        document.getElementById('fcWord').textContent = card.word;
        document.getElementById('fcReadingHint').textContent = card.reading && card.reading !== card.word ? card.reading : '';
        document.getElementById('fcStatusFront').textContent = card.mastered ? '✅ 已掌握' : '📖 學習中';

        document.getElementById('fcWordBack').textContent = card.word;
        document.getElementById('fcReading').textContent = card.reading || '';
        const shortPos = formatPosShort(card.pos);
        const posBackEl = document.getElementById('fcPosBackBadge');
        if (shortPos) {
          posBackEl.textContent = '【' + shortPos + '】';
          posBackEl.classList.remove('hidden');
        } else {
          posBackEl.classList.add('hidden');
        }
        document.getElementById('fcMeaning').textContent = card.meaning;
        const exEl = document.getElementById('fcExample');
        if (card.example) {
          exEl.textContent = card.example;
          exEl.classList.remove('hidden');
        } else {
          exEl.classList.add('hidden');
        }

        const masterBtn = document.getElementById('fcToggleMasteredBtn');
        if (card.mastered) {
          masterBtn.textContent = '✅ 已掌握（點擊取消）';
          masterBtn.className = 'flex-1 max-w-xs px-4 py-2.5 rounded-xl font-semibold text-sm border bg-emerald-50 border-emerald-300 text-emerald-800 shadow-sm transition';
        } else {
          masterBtn.textContent = '標記為已掌握';
          masterBtn.className = 'flex-1 max-w-xs px-4 py-2.5 rounded-xl font-semibold text-sm border bg-white border-slate-200 hover:bg-emerald-50 text-slate-700 shadow-sm transition';
        }
      }

      // --- Quiz ---
      function startQuiz(onlyIds) {
        const deck = getActiveDeck();
        const pool = onlyIds && onlyIds.length > 0
          ? deck.words.filter(w => onlyIds.includes(w.id))
          : deck.words.slice();

        if (pool.length < 2) {
          showToast('單字本至少需要 2 個單字才能進行測驗');
          switchMode('list');
          return;
        }

        const shuffled = pool.sort(() => Math.random() - 0.5).slice(0, 10);
        const questions = shuffled.map(target => {
          const fmtOpt = w => {
            const p = formatPosShort(w.pos);
            return (p ? '【' + p + '】' : '') + w.meaning;
          };
          const correctText = fmtOpt(target);
          const distractors = deck.words
            .filter(w => w.id !== target.id)
            .sort(() => Math.random() - 0.5)
            .slice(0, 3)
            .map(fmtOpt);
          const options = [correctText, ...distractors].sort(() => Math.random() - 0.5);
          return { target, correctText, options };
        });

        state.quiz = { questions, idx: 0, score: 0, answered: false, wrongIds: [] };
        document.getElementById('quizContainer').classList.remove('hidden');
        document.getElementById('quizResult').classList.add('hidden');
        renderQuizQuestion();
      }

      function renderQuizQuestion() {
        const qState = state.quiz;
        if (qState.idx >= qState.questions.length) {
          document.getElementById('quizContainer').classList.add('hidden');
          document.getElementById('quizResult').classList.remove('hidden');
          document.getElementById('quizResultSummary').textContent =
            '共 ' + qState.questions.length + ' 題，答對 ' + qState.score + ' 題（正確率 ' +
            Math.round((qState.score / qState.questions.length) * 100) + '%）';
          const wrongBtn = document.getElementById('quizWrongOnlyBtn');
          wrongBtn.classList.toggle('hidden', qState.wrongIds.length === 0);
          return;
        }

        const q = qState.questions[qState.idx];
        document.getElementById('quizProgress').textContent = '第 ' + (qState.idx + 1) + ' / ' + qState.questions.length + ' 題';
        document.getElementById('quizScoreBadge').textContent = '得分：' + qState.score;
        document.getElementById('quizWord').textContent = q.target.word;
        document.getElementById('quizReading').textContent = q.target.reading || '';
        document.getElementById('quizFeedback').classList.add('hidden');

        const optBox = document.getElementById('quizOptions');
        optBox.innerHTML = '';
        qState.answered = false;

        q.options.forEach(optText => {
          const btn = document.createElement('button');
          btn.type = 'button';
          btn.textContent = optText;
          btn.className = 'text-left px-4 py-3.5 rounded-xl border border-slate-200 bg-slate-50 hover:bg-indigo-50 hover:border-indigo-300 font-medium text-sm text-slate-800 transition';
          btn.addEventListener('click', () => handleQuizAnswer(btn, optText, q));
          optBox.appendChild(btn);
        });
      }

      function handleQuizAnswer(clickedBtn, chosen, q) {
        if (state.quiz.answered) return;
        state.quiz.answered = true;
        const isCorrect = chosen === q.correctText;

        const buttons = document.getElementById('quizOptions').querySelectorAll('button');
        buttons.forEach(b => {
          b.disabled = true;
          if (b.textContent === q.correctText) {
            b.className = 'text-left px-4 py-3.5 rounded-xl border border-emerald-500 bg-emerald-50 font-semibold text-sm text-emerald-900';
          } else if (b === clickedBtn && !isCorrect) {
            b.className = 'text-left px-4 py-3.5 rounded-xl border border-rose-400 bg-rose-50 font-semibold text-sm text-rose-900';
          } else {
            b.className = 'text-left px-4 py-3.5 rounded-xl border border-slate-200 bg-slate-50 opacity-60 text-sm text-slate-600';
          }
        });

        const fb = document.getElementById('quizFeedback');
        const fbText = document.getElementById('quizFeedbackText');
        fb.classList.remove('hidden');
        if (isCorrect) {
          state.quiz.score++;
          fb.className = 'rounded-xl p-3.5 text-sm flex items-center justify-between gap-3 bg-emerald-50 text-emerald-900 border border-emerald-200';
          fbText.textContent = '答對了！';
        } else {
          state.quiz.wrongIds.push(q.target.id);
          fb.className = 'rounded-xl p-3.5 text-sm flex items-center justify-between gap-3 bg-rose-50 text-rose-900 border border-rose-200';
          fbText.textContent = '正確答案：' + q.correctText;
        }
        document.getElementById('quizScoreBadge').textContent = '得分：' + state.quiz.score;
      }

      // --- Match Game ---
      function startMatchGame() {
        clearInterval(state.match.timerId);
        const deck = getActiveDeck();
        if (deck.words.length < 3) {
          showToast('單字本至少需要 3 個單字才能玩配對遊戲');
          switchMode('list');
          return;
        }

        const sample = deck.words.slice().sort(() => Math.random() - 0.5).slice(0, 6);
        const cards = [];
        sample.forEach(w => {
          const p = formatPosShort(w.pos);
          cards.push({ uid: w.id + '_jp', pairId: w.id, text: w.word, sub: w.reading, kind: 'jp' });
          cards.push({ uid: w.id + '_zh', pairId: w.id, text: (p ? '【' + p + '】' : '') + w.meaning, sub: '', kind: 'zh' });
        });
        cards.sort(() => Math.random() - 0.5);

        state.match = {
          cards,
          selected: [],
          matchedIds: new Set(),
          startTime: Date.now(),
          timerId: null,
          finished: false
        };

        document.getElementById('matchDone').classList.add('hidden');
        document.getElementById('matchGrid').classList.remove('hidden');
        document.getElementById('matchTimer').textContent = '0.0s';

        state.match.timerId = setInterval(() => {
          if (state.match.finished) return;
          const sec = ((Date.now() - state.match.startTime) / 1000).toFixed(1);
          document.getElementById('matchTimer').textContent = sec + 's';
        }, 100);

        renderMatchGrid();
      }

      function renderMatchGrid() {
        const grid = document.getElementById('matchGrid');
        grid.innerHTML = '';
        state.match.cards.forEach(card => {
          const isMatched = state.match.matchedIds.has(card.pairId);
          const isSelected = state.match.selected.some(s => s.uid === card.uid);

          const btn = document.createElement('button');
          btn.type = 'button';
          btn.disabled = isMatched;

          let cls = 'min-h-[84px] p-3 rounded-xl border text-center flex flex-col items-center justify-center gap-1 transition select-none ';
          if (isMatched) {
            cls += 'bg-emerald-50 border-emerald-200 text-emerald-700 opacity-40 cursor-default';
          } else if (isSelected) {
            cls += 'bg-indigo-600 border-indigo-600 text-white shadow-md scale-[1.02]';
          } else {
            cls += 'bg-white border-slate-200 hover:border-indigo-300 hover:bg-indigo-50/40 text-slate-800 shadow-xs';
          }
          btn.className = cls;

          const mainSpan = document.createElement('span');
          mainSpan.className = (card.kind === 'jp' ? 'jp-text text-base sm:text-lg font-bold' : 'text-xs sm:text-sm font-medium');
          mainSpan.textContent = card.text;
          btn.appendChild(mainSpan);

          if (card.sub && card.sub !== card.text) {
            const subSpan = document.createElement('span');
            subSpan.className = 'jp-text text-[11px] ' + (isSelected ? 'text-indigo-200' : 'text-slate-400');
            subSpan.textContent = card.sub;
            btn.appendChild(subSpan);
          }

          btn.addEventListener('click', () => handleMatchClick(card));
          grid.appendChild(btn);
        });
      }

      function handleMatchClick(card) {
        if (state.match.finished || state.match.matchedIds.has(card.pairId)) return;
        if (state.match.selected.some(s => s.uid === card.uid)) {
          state.match.selected = [];
          renderMatchGrid();
          return;
        }
        state.match.selected.push(card);
        if (state.match.selected.length === 2) {
          const [a, b] = state.match.selected;
          if (a.pairId === b.pairId && a.uid !== b.uid) {
            state.match.matchedIds.add(a.pairId);
            state.match.selected = [];
            renderMatchGrid();
            if (state.match.matchedIds.size === state.match.cards.length / 2) {
              state.match.finished = true;
              clearInterval(state.match.timerId);
              const elapsed = ((Date.now() - state.match.startTime) / 1000).toFixed(1);
              document.getElementById('matchTimer').textContent = elapsed + 's';
              document.getElementById('matchGrid').classList.add('hidden');
              document.getElementById('matchDone').classList.remove('hidden');
              document.getElementById('matchDoneText').textContent = '耗時 ' + elapsed + ' 秒完成所有配對！';
            }
          } else {
            renderMatchGrid();
            setTimeout(() => {
              state.match.selected = [];
              renderMatchGrid();
            }, 380);
          }
        } else {
          renderMatchGrid();
        }
      }

      // --- Word List ---
      function renderWordList() {
        const deck = getActiveDeck();
        const q = document.getElementById('listSearchInput').value.trim().toLowerCase();
        const filter = document.getElementById('listFilterSelect').value;

        const filtered = deck.words.filter(w => {
          if (filter === 'mastered' && !w.mastered) return false;
          if (filter === 'learning' && w.mastered) return false;
          if (!q) return true;
          return (
            w.word.toLowerCase().includes(q) ||
            (w.reading || '').toLowerCase().includes(q) ||
            w.meaning.toLowerCase().includes(q)
          );
        });

        const container = document.getElementById('wordListContainer');
        container.innerHTML = '';

        if (filtered.length === 0) {
          const empty = document.createElement('div');
          empty.className = 'bg-white rounded-xl border border-slate-200 p-8 text-center text-sm text-slate-500';
          empty.textContent = '找不到符合條件的單字。';
          container.appendChild(empty);
          return;
        }

        filtered.forEach(w => {
          const card = document.createElement('div');
          card.className = 'bg-white rounded-xl border border-slate-200 p-3.5 sm:p-4 flex items-start justify-between gap-3 shadow-xs';

          const left = document.createElement('div');
          left.className = 'space-y-1 min-w-0 flex-1';

          const topRow = document.createElement('div');
          topRow.className = 'flex flex-wrap items-baseline gap-2';

          const wordEl = document.createElement('span');
          wordEl.className = 'jp-text text-lg font-bold text-slate-900';
          wordEl.textContent = w.word;
          topRow.appendChild(wordEl);

          if (w.reading) {
            const readEl = document.createElement('span');
            readEl.className = 'jp-text text-xs sm:text-sm text-slate-500';
            readEl.textContent = w.reading;
            topRow.appendChild(readEl);
          }

          if (w.level) {
            const lvl = document.createElement('span');
            lvl.className = 'text-[11px] font-semibold px-1.5 py-0.5 rounded bg-indigo-50 text-indigo-700';
            lvl.textContent = w.level;
            topRow.appendChild(lvl);
          }

          left.appendChild(topRow);

          const meaningEl = document.createElement('div');
          meaningEl.className = 'text-sm text-slate-800 font-medium flex items-center gap-1.5 flex-wrap';
          const shortP = formatPosShort(w.pos);
          if (shortP) {
            const pBadge = document.createElement('span');
            pBadge.className = 'text-[11px] font-bold px-1.5 py-0.5 rounded bg-slate-700 text-white';
            pBadge.textContent = shortP;
            meaningEl.appendChild(pBadge);
          }
          const mSpan = document.createElement('span');
          mSpan.textContent = w.meaning;
          meaningEl.appendChild(mSpan);
          left.appendChild(meaningEl);

          if (w.example) {
            const exEl = document.createElement('div');
            exEl.className = 'jp-text text-xs text-slate-500';
            exEl.textContent = w.example;
            left.appendChild(exEl);
          }

          const right = document.createElement('div');
          right.className = 'flex items-center gap-1.5 shrink-0';

          const speakBtn = document.createElement('button');
          speakBtn.type = 'button';
          speakBtn.textContent = '🔊';
          speakBtn.title = '朗讀';
          speakBtn.className = 'w-8 h-8 rounded-lg bg-slate-100 hover:bg-indigo-100 text-xs flex items-center justify-center transition';
          speakBtn.addEventListener('click', () => speakJapanese(w.word));

          const toggleBtn = document.createElement('button');
          toggleBtn.type = 'button';
          toggleBtn.textContent = w.mastered ? '已掌握' : '學習中';
          toggleBtn.className = w.mastered
            ? 'px-2.5 py-1.5 rounded-lg text-xs font-semibold bg-emerald-50 text-emerald-700 border border-emerald-200'
            : 'px-2.5 py-1.5 rounded-lg text-xs font-semibold bg-amber-50 text-amber-700 border border-amber-200';
          toggleBtn.addEventListener('click', () => {
            w.mastered = !w.mastered;
            saveState();
            renderHeaderAndStats();
            renderWordList();
          });

          const delBtn = document.createElement('button');
          delBtn.type = 'button';
          delBtn.textContent = '🗑';
          delBtn.title = '刪除';
          delBtn.className = 'w-8 h-8 rounded-lg hover:bg-rose-50 text-slate-400 hover:text-rose-600 text-xs flex items-center justify-center transition';
          delBtn.addEventListener('click', () => {
            deck.words = deck.words.filter(item => item.id !== w.id);
            saveState();
            rebuildFlashcardOrder();
            renderHeaderAndStats();
            renderWordList();
          });

          right.append(speakBtn, toggleBtn, delBtn);
          card.append(left, right);
          container.appendChild(card);
        });
      }

      // --- Mode Switching ---
      function switchMode(mode) {
        state.mode = mode;
        clearInterval(state.match.timerId);

        document.querySelectorAll('.mode-tab').forEach(tab => {
          const active = tab.dataset.mode === mode;
          tab.className = 'mode-tab flex-1 sm:flex-none px-3 sm:px-4 py-2.5 text-xs sm:text-sm font-semibold border-b-2 transition whitespace-nowrap ' +
            (active ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-slate-500 hover:text-slate-800');
        });

        ['flashcards', 'quiz', 'match', 'list'].forEach(m => {
          document.getElementById('view-' + m).classList.toggle('hidden', m !== mode);
        });

        if (mode === 'flashcards') renderFlashcards();
        if (mode === 'quiz') startQuiz();
        if (mode === 'match') startMatchGame();
        if (mode === 'list') renderWordList();
      }

      // --- Gemini Gem Prompt Builder ---
      function updateGemPromptPreview() {
        const urlInput = document.getElementById('gemAppUrlInput').value.trim() || 'https://yourname.github.io/kotocard/';
        const criteria = document.getElementById('gemCriteriaInput').value.trim() || '挑選照片中值得學習的日文單字';
        const prompt = [
          '你是我的「日文單字卡自動匯入助手」。',
          '當我上傳日文文章、課本、菜單或螢幕截圖照片時，請執行以下步驟：',
          '',
          '1. 【篩選單字】：根據我的條件「' + criteria + '」，從圖片中擷取符合要求的日文單字。',
          '2. 【整理資訊】：為每個單字提供：word（原形/漢字）、reading（平假名讀音）、meaning（繁體中文意思）、pos（簡短詞性如：名、動、副、形、な形）、level（JLPT 級別）、example（簡短日文例句）。',
          '3. 【產生 JSON】：整理成純 JSON 陣列（格式如下）：',
          '[{"word":"把握","reading":"はあく","meaning":"掌握","pos":"名","level":"N2","example":"状況を把握する。"}]',
          '4. 【產生一鍵啟動連結】：將上述 JSON 進行 URL Encode 後，接在以下網址後方，輸出成一個可直接點擊的超連結：',
          urlInput + '?import=<URL_ENCODED_JSON>',
          '',
          '請先列出整理好的單字表格，並在最後附上「👉 點此一鍵匯入 KotoCard 開始背單字」的超連結與原始 JSON（方便備用貼上）。'
        ].join('\n');
        document.getElementById('gemPromptPreview').textContent = prompt;
      }

      // --- Event Bindings ---
      document.querySelectorAll('.mode-tab').forEach(tab => {
        tab.addEventListener('click', () => switchMode(tab.dataset.mode));
      });

      document.getElementById('deckSelect').addEventListener('change', e => {
        state.activeDeckId = e.target.value;
        saveState();
        rebuildFlashcardOrder();
        renderHeaderAndStats();
        switchMode(state.mode);
      });

      // Flashcard events
      document.getElementById('fcStage').addEventListener('click', e => {
        if (e.target.closest('button')) return;
        state.fcFlipped = !state.fcFlipped;
        document.getElementById('fcCardInner').classList.toggle('is-flipped', state.fcFlipped);
      });

      document.getElementById('fcSpeakFront').addEventListener('click', e => {
        e.stopPropagation();
        speakJapanese(document.getElementById('fcWord').textContent);
      });
      document.getElementById('fcSpeakBack').addEventListener('click', e => {
        e.stopPropagation();
        speakJapanese(document.getElementById('fcWordBack').textContent);
      });

      document.getElementById('fcPrevBtn').addEventListener('click', () => {
        state.fcIndex--;
        state.fcFlipped = false;
        renderFlashcards();
      });
      document.getElementById('fcNextBtn').addEventListener('click', () => {
        state.fcIndex++;
        state.fcFlipped = false;
        renderFlashcards();
      });

      document.getElementById('fcToggleMasteredBtn').addEventListener('click', () => {
        const deck = getActiveDeck();
        const currentId = state.fcOrder[state.fcIndex];
        const card = deck.words.find(w => w.id === currentId);
        if (!card) return;
        card.mastered = !card.mastered;
        saveState();
        renderHeaderAndStats();
        renderFlashcards();
      });

      document.getElementById('fcFilterAll').addEventListener('click', () => {
        state.fcFilter = 'all';
        rebuildFlashcardOrder();
        renderFlashcards();
      });
      document.getElementById('fcFilterLearning').addEventListener('click', () => {
        state.fcFilter = 'learning';
        rebuildFlashcardOrder();
        renderFlashcards();
      });
      document.getElementById('fcResetFilterBtn').addEventListener('click', () => {
        state.fcFilter = 'all';
        rebuildFlashcardOrder();
        renderFlashcards();
      });
      document.getElementById('fcShuffleBtn').addEventListener('click', () => {
        state.fcOrder.sort(() => Math.random() - 0.5);
        state.fcIndex = 0;
        state.fcFlipped = false;
        renderFlashcards();
        showToast('已隨機排序單字卡');
      });

      // Keyboard navigation for flashcards
      document.addEventListener('keydown', e => {
        if (state.mode !== 'flashcards') return;
        if (document.querySelector('input:focus, textarea:focus')) return;
        if (e.code === 'Space') {
          e.preventDefault();
          state.fcFlipped = !state.fcFlipped;
          document.getElementById('fcCardInner').classList.toggle('is-flipped', state.fcFlipped);
        } else if (e.code === 'ArrowRight') {
          state.fcIndex++;
          state.fcFlipped = false;
          renderFlashcards();
        } else if (e.code === 'ArrowLeft') {
          state.fcIndex--;
          state.fcFlipped = false;
          renderFlashcards();
        }
      });

      // Quiz events
      document.getElementById('quizSpeakBtn').addEventListener('click', () => {
        speakJapanese(document.getElementById('quizWord').textContent);
      });
      document.getElementById('quizNextBtn').addEventListener('click', () => {
        state.quiz.idx++;
        renderQuizQuestion();
      });
      document.getElementById('quizRestartBtn').addEventListener('click', () => startQuiz());
      document.getElementById('quizWrongOnlyBtn').addEventListener('click', () => startQuiz(state.quiz.wrongIds));

      // Match events
      document.getElementById('matchRestartBtn').addEventListener('click', startMatchGame);
      document.getElementById('matchPlayAgainBtn').addEventListener('click', startMatchGame);

      // List events
      document.getElementById('listSearchInput').addEventListener('input', renderWordList);
      document.getElementById('listFilterSelect').addEventListener('change', renderWordList);
      document.getElementById('exportJsonBtn').addEventListener('click', () => {
        const deck = getActiveDeck();
        const json = JSON.stringify(deck.words, null, 2);
        const copyFallback = () => {
          const ta = document.createElement('textarea');
          ta.value = json;
          document.body.appendChild(ta);
          ta.select();
          document.execCommand('copy');
          ta.remove();
          showToast('已複製單字本 JSON 至剪貼簿');
        };
        if (navigator.clipboard && navigator.clipboard.writeText) {
          navigator.clipboard.writeText(json).then(() => showToast('已複製單字本 JSON 至剪貼簿')).catch(copyFallback);
        } else {
          copyFallback();
        }
      });

      // Import Modal events
      const importModal = document.getElementById('importModal');
      function switchImportTab(tabId) {
        const tabs = ['tabImageImport', 'tabBatchImport', 'tabSingleAdd', 'tabNewDeck'];
        const panels = ['panelImageImport', 'panelBatchImport', 'panelSingleAdd', 'panelNewDeck'];
        tabs.forEach((t, i) => {
          const active = t === tabId;
          document.getElementById(t).className = active
            ? 'px-3 py-1.5 rounded-lg text-xs sm:text-sm font-semibold bg-indigo-50 text-indigo-700'
            : 'px-3 py-1.5 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100';
          document.getElementById(panels[i]).classList.toggle('hidden', !active);
        });
      }
      document.getElementById('openImportBtn').addEventListener('click', () => {
        try {
          const savedKey = localStorage.getItem('kotocard_gemini_key');
          if (savedKey) document.getElementById('geminiApiKeyInput').value = savedKey;
        } catch (e) {}
        switchImportTab('tabImageImport');
        importModal.classList.remove('hidden');
      });
      document.getElementById('closeImportModal').addEventListener('click', () => importModal.classList.add('hidden'));
      document.getElementById('tabImageImport').addEventListener('click', () => switchImportTab('tabImageImport'));
      document.getElementById('tabBatchImport').addEventListener('click', () => switchImportTab('tabBatchImport'));
      document.getElementById('tabSingleAdd').addEventListener('click', () => switchImportTab('tabSingleAdd'));
      document.getElementById('tabNewDeck').addEventListener('click', () => switchImportTab('tabNewDeck'));

      let uploadedImageBase64 = '';
      let uploadedImageMime = 'image/jpeg';
      document.getElementById('imageUploadInput').addEventListener('change', e => {
        const file = e.target.files && e.target.files[0];
        const preview = document.getElementById('imagePreviewEl');
        if (!file) {
          uploadedImageBase64 = '';
          preview.classList.add('hidden');
          return;
        }
        uploadedImageMime = file.type || 'image/jpeg';
        const reader = new FileReader();
        reader.onload = ev => {
          const dataUrl = String(ev.target.result || '');
          preview.src = dataUrl;
          preview.classList.remove('hidden');
          uploadedImageBase64 = dataUrl.split(',')[1] || '';
        };
        reader.readAsDataURL(file);
      });

      document.getElementById('runImageImportBtn').addEventListener('click', async () => {
        const statusEl = document.getElementById('imageImportStatus');
        const apiKey = document.getElementById('geminiApiKeyInput').value.trim();
        const criteria = document.getElementById('imageCriteriaInput').value.trim();

        const setStatus = (msg, isError) => {
          statusEl.textContent = msg;
          statusEl.className = 'text-xs rounded-lg p-2.5 ' + (isError
            ? 'bg-rose-50 text-rose-800 border border-rose-200'
            : 'bg-indigo-50 text-indigo-800 border border-indigo-200');
          statusEl.classList.remove('hidden');
        };

        if (!uploadedImageBase64) {
          setStatus('請先選擇一張單字書照片或截圖。', true);
          return;
        }
        if (!apiKey) {
          setStatus('請填入 Gemini API Key（或改用手機 Gemini App 傳圖產生匯入連結）。', true);
          return;
        }

        try { localStorage.setItem('kotocard_gemini_key', apiKey); } catch (e) {}

        const btn = document.getElementById('runImageImportBtn');
        btn.disabled = true;
        setStatus('正在透過 Gemini Vision 辨識圖片中的日文單字…', false);

        try {
          const promptText = [
            '請辨識這張圖片中的日文單字，並遵守以下條件：' + criteria,
            '請只輸出純 JSON 陣列（不要加 markdown 標記），每個物件包含：',
            'word（日文單字）、reading（平假名讀音，若與 word 相同可留空或填平假名）、meaning（繁體中文意思）、pos（簡短詞性：名、動、副、形、な形）、level（級別如 N5/N4/N3，若不確定填 日文）、example（簡短日文例句）。'
          ].join('\n');

          const res = await fetch('https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=' + encodeURIComponent(apiKey), {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              contents: [{
                parts: [
                  { text: promptText },
                  { inline_data: { mime_type: uploadedImageMime, data: uploadedImageBase64 } }
                ]
              }]
            })
          });
          if (!res.ok) {
            throw new Error('API 回應錯誤 (' + res.status + ')');
          }
          const data = await res.json();
          const rawText = (data.candidates && data.candidates[0] && data.candidates[0].content && data.candidates[0].content.parts && data.candidates[0].content.parts[0] && data.candidates[0].content.parts[0].text) || '';
          const cleaned = rawText.replace(/^```json\s*|```$/gim, '').trim();
          const items = parseImportText(cleaned);
          if (items.length === 0) {
            setStatus('未辨識到符合條件的單字，請換一張清晰一點的照片或調整條件。', true);
          } else {
            const added = addWordsToDeck(items);
            importModal.classList.add('hidden');
            statusEl.classList.add('hidden');
            renderHeaderAndStats();
            switchMode(state.mode);
            showToast('圖片辨識完成！已匯入 ' + added + ' 個新單字');
          }
        } catch (err) {
          setStatus('辨識失敗：' + (err && err.message ? err.message : '網路或 API Key 錯誤'), true);
        } finally {
          btn.disabled = false;
        }
      });

      document.getElementById('runBatchImportBtn').addEventListener('click', () => {
        const raw = document.getElementById('batchImportTextarea').value;
        const items = parseImportText(raw);
        if (items.length === 0) {
          showToast('無法解析格式，請確認為 JSON 或 單字|讀音|中文');
          return;
        }
        const added = addWordsToDeck(items);
        document.getElementById('batchImportTextarea').value = '';
        importModal.classList.add('hidden');
        renderHeaderAndStats();
        switchMode(state.mode);
        showToast('成功匯入 ' + added + ' 個新單字！');
      });

      document.getElementById('panelSingleAdd').addEventListener('submit', e => {
        e.preventDefault();
        const word = document.getElementById('addWord').value.trim();
        const reading = document.getElementById('addReading').value.trim();
        const meaning = document.getElementById('addMeaning').value.trim();
        const pos = document.getElementById('addPos').value.trim();
        const level = document.getElementById('addLevel').value.trim();
        const example = document.getElementById('addExample').value.trim();
        const added = addWordsToDeck([{ word, reading, meaning, pos, level, example }]);
        e.target.reset();
        importModal.classList.add('hidden');
        renderHeaderAndStats();
        switchMode(state.mode);
        showToast(added > 0 ? '已新增單字「' + word + '」' : '此單字已存在於單字本中');
      });

      document.getElementById('panelNewDeck').addEventListener('submit', e => {
        e.preventDefault();
        const name = document.getElementById('newDeckName').value.trim();
        if (!name) return;
        const newDeck = { id: 'deck_' + Date.now(), name, words: [] };
        state.decks.push(newDeck);
        state.activeDeckId = newDeck.id;
        saveState();
        e.target.reset();
        importModal.classList.add('hidden');
        rebuildFlashcardOrder();
        renderHeaderAndStats();
        switchMode('list');
        showToast('已建立單字本「' + name + '」');
      });

      // Gem Modal events
      const gemModal = document.getElementById('gemModal');
      document.getElementById('openGemBtn').addEventListener('click', () => {
        const cur = window.location.href.split('?')[0].split('#')[0];
        if (cur.startsWith('http') && !document.getElementById('gemAppUrlInput').value) {
          document.getElementById('gemAppUrlInput').value = cur;
        }
        updateGemPromptPreview();
        gemModal.classList.remove('hidden');
      });
      document.getElementById('closeGemModal').addEventListener('click', () => gemModal.classList.add('hidden'));
      document.getElementById('gemAppUrlInput').addEventListener('input', updateGemPromptPreview);
      document.getElementById('gemCriteriaInput').addEventListener('input', updateGemPromptPreview);
      document.getElementById('copyGemPromptBtn').addEventListener('click', () => {
        const text = document.getElementById('gemPromptPreview').textContent;
        const fallback = () => {
          const ta = document.createElement('textarea');
          ta.value = text;
          document.body.appendChild(ta);
          ta.select();
          document.execCommand('copy');
          ta.remove();
          showToast('已複製 Gem 指令！');
        };
        if (navigator.clipboard && navigator.clipboard.writeText) {
          navigator.clipboard.writeText(text).then(() => showToast('已複製 Gem 指令！')).catch(fallback);
        } else {
          fallback();
        }
      });

      // Init
      loadState();
      checkUrlAutoImport();
      rebuildFlashcardOrder();
      renderHeaderAndStats();
      switchMode('flashcards');
    })();
  </script>
</body>
</html>
