<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>書道鑑賞支援アプリ『言の葉』</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/@panzoom/panzoom/dist/panzoom.min.js"></script>
    
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@700&family=Zen+Maru+Gothic:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Zen Maru Gothic', sans-serif;
            background-color: #F0F9FF;
        }
        h1 {
            font-family: 'Noto Serif JP', serif;
        }
        .tab-button {
            transition: all 0.3s ease;
            border-bottom: 2px solid transparent;
        }
        .tab-button.active {
            border-bottom-color: #4A5568;
            color: #2D3748;
            font-weight: 700;
        }
        .word-button {
            transition: all 0.2s ease;
            border: 1px solid #CBD5E0;
        }
        .word-button:hover:not(.selected) {
            background-color: #E2E8F0;
        }
        .screen { display: none; }
        .screen.active { display: block; }
        .word-cloud span {
            display: inline-block; padding: 2px 5px;
            margin: 2px; color: #4A5568;
        }
        #paste-area.drag-over {
            border-color: #4A5568; background-color: #F7FAFC;
        }
        .word-button.selected { transform: scale(1.05); }
        .word-button.intuitive.selected {
            background-color: #ea580c; border-color: #ea580c; color: white;
        }
        .word-button.analytical_tengaku.selected {
            background-color: #0891b2; border-color: #0891b2; color: white;
        }
        .word-button.analytical_jikei.selected {
            background-color: #0d9488; border-color: #0d9488; color: white;
        }
        /* ▼② 章法用の色 */
        .word-button.analytical_layout.selected {
            background-color: #6d28d9; border-color: #6d28d9; color: white;
        }
        .word-tag {
            padding: 2px 10px; border-radius: 9999px;
            font-size: 0.75rem; font-weight: 500; line-height: 1.2;
        }
        .intuitive-tag { background-color: #fff7ed; color: #c2410c; }
        .analytical_tengaku-tag { background-color: #ecfeff; color: #0e7490; }
        .analytical_jikei-tag { background-color: #f0fdfa; color: #134e4a; }
        /* ▼② 章法用のタグ色 */
        .analytical_layout-tag { background-color: #f5f3ff; color: #5b21b6; }
        .like-btn.liked svg { fill: #ef4444; color: #ef4444; }
    </style>
</head>
<body class="text-gray-800">

    <div id="teacher-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden z-50">
        <div class="bg-white rounded-lg shadow-xl p-6 w-full max-w-md">
            <h2 class="text-2xl font-bold mb-4">作品設定</h2>
            <div class="space-y-4">
                <div>
                    <label for="new-artwork-title" class="block text-sm font-medium text-gray-700">作品名</label>
                    <input type="text" id="new-artwork-title" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-gray-500 focus:border-gray-500">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700">作品画像</label>
                    <div id="paste-area" class="mt-1 flex justify-center items-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md h-48 cursor-pointer text-center">
                        <div id="paste-instruction">
                            <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48" aria-hidden="true">
                                <path d="M28 8H12a4 4
 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                            </svg>
                            <p class="text-sm text-gray-600">画像をここにペーストしてください</p>
                            <p class="text-xs text-gray-500">(またはクリックしてファイルを選択)</p>
                        </div>
                        <img id="image-preview" src="" class="max-h-full mx-auto hidden"/>
                    </div>
                    <input type="file" id="file-input" class="hidden" accept="image/*">
                </div>
            </div>
            <div class="mt-6 flex justify-end space-x-3">
                <button id="cancel-teacher-settings" class="bg-gray-200 text-gray-700 px-4 py-2 rounded-md hover:bg-gray-300">キャンセル</button>
                <button id="save-teacher-settings" class="bg-gray-700 text-white px-4 py-2 rounded-md hover:bg-gray-800">保存する</button>
            </div>
        </div>
    </div>

    <div id="app" class="max-w-4xl mx-auto p-4 md:p-8">
        <header class="text-center mb-6 pb-4 border-b-2 border-gray-200 relative">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-700">書道鑑賞支援アプリ『言の葉』</h1>
            <p id="artwork-title" class="text-lg text-gray-500 mt-2">作品名：蘭亭序（部分）</p>
            <button id="teacher-settings-btn" class="absolute top-0 right-0 text-sm text-gray-500 hover:underline p-2">作品を変更</button>
        </header>

        <div id="appreciation-screen" class="screen active">
            <div class="bg-white rounded-lg shadow-lg p-4 mb-6 overflow-hidden">
                <img id="artwork-image" src="https://placehold.co/800x400/f0f0f0/666666?text=%E9%91%91%E8%B3%9E%E4%BD%9C%E5%93%81%E3%81%AE%E7%94%BB%E5%83%8F" alt="鑑賞作品" class="w-full h-auto rounded-md object-cover cursor-grab">
            </div>

            <div class="flex justify-center border-b mb-6">
                <button class="tab-button active text-lg py-2 px-6 text-gray-600" data-tab="intuitive">直感的鑑賞</button>
                <button class="tab-button text-lg py-2 px-6 text-gray-600" data-tab="analytical">分析的鑑賞</button>
            </div>
           
            <div id="tab-content">
                <div id="intuitive-content">
                    <h3 class="text-xl font-bold mb-1 text-center text-gray-600">作品全体から受ける印象は？</h3>
                    <p class="text-sm text-gray-500 text-center mb-4">当てはまるものをタップしてください（複数可）</p>
                    <div id="intuitive-words" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
                    <input type="text" id="free-text-intuitive" class="mt-4 w-full border border-gray-300 rounded-md p-2 text-sm" placeholder="その他の印象（自由記述）">
                </div>

                <div id="analytical-content" class="hidden">
                    <div class="mb-8">
                        <h3 class="text-xl font-bold mb-1 text-center text-gray-600">点画や用筆の特徴は？</h3>
                        <p class="text-sm text-gray-500 text-center mb-4">当てはまるものをタップしてください（複数可）</p>
                        <div id="analytical-tengaku-words" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
                        <input type="text" id="free-text-tengaku" class="mt-4 w-full border border-gray-300 rounded-md p-2 text-sm" placeholder="その他の点画の特徴（自由記述）">
                    </div>
                    <div class="mb-8">
                        <h3 class="text-xl font-bold mb-1 text-center text-gray-600">字形の特徴は？</h3>
                        <p class="text-sm text-gray-500 text-center mb-4">当てはまるものをタップしてください（複数可）</p>
                        <div id="analytical-jikei-words" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
                        <input type="text" id="free-text-jikei" class="mt-4 w-full border border-gray-300 rounded-md p-2 text-sm" placeholder="その他の字形の特徴（自由記述）">
                    </div>
                    <div>
                        <h3 class="text-xl font-bold mb-1 text-center text-gray-600">章法（全体構成）の特徴は？</h3>
                        <p class="text-sm text-gray-500 text-center mb-4">当てはまるものをタップしてください（複数可）</p>
                        <div id="analytical-layout-words" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
                        <input type="text" id="free-text-layout" class="mt-4 w-full border border-gray-300 rounded-md p-2 text-sm" placeholder="その他の章法の特徴（自由記述）">
                    </div>
                </div>
            </div>

            <div class="text-center mt-8">
                <button id="go-to-summary" class="bg-gray-700 hover:bg-gray-800 text-white font-bold py-3 px-8 rounded-full shadow-lg transition-transform transform hover:scale-105">
                    鑑賞をまとめる
                </button>
            </div>
        </div>

        <div id="summary-screen" class="screen">
            <h2 class="text-2xl font-bold mb-6 text-center">鑑賞を言葉にする</h2>
            <div class="grid md:grid-cols-3 gap-6">
                <div class="md:col-span-1 bg-white p-4 rounded-lg shadow">
                    <h3 class="font-bold border-b pb-2 mb-3">選んだ言葉</h3>
                    <ul id="selected-words-list" class="space-y-2 text-sm"></ul>
                </div>
                <div class="md:col-span-2 bg-white p-4 rounded-lg shadow">
                    <p class="text-sm mb-2 text-gray-600">下の文章を参考に、自分の言葉で自由に鑑賞文を作成してみましょう。</p>
                    <textarea id="summary-textarea" class="w-full h-64 p-3 border rounded-md focus:ring-2 focus:ring-gray-400 focus:outline-none" placeholder="ここに鑑賞文を入力..."></textarea>
                    <div class="flex justify-between items-center mt-4">
                        <button id="back-to-appreciation" class="text-gray-600 hover:underline">← 選び直す</button>
                        <button id="go-to-gallery" class="bg-gray-700 hover:bg-gray-800 text-white font-bold py-2 px-6 rounded-full shadow-lg transition-transform transform hover:scale-105">
                            提出して共有する
                        </button>
                    </div>
                </div>
            </div>
        </div>
       
        <div id="gallery-screen" class="screen">
            <h2 class="text-2xl font-bold mb-2 text-center">みんなの鑑賞</h2>
            <p class="text-center text-gray-600 mb-6">他の人の見方から、新しい発見があるかもしれません。</p>

            <div class="bg-white p-4 rounded-lg shadow mb-8">
                    <h3 class="font-bold text-center mb-2">この作品で選ばれた言葉たち</h3>
                    <div id="word-cloud" class="word-cloud text-center leading-relaxed"></div>
            </div>

            <div id="gallery-posts" class="space-y-6">
                </div>

            <div class="text-center mt-8">
                <button id="restart-app" class="text-gray-600 hover:underline">最初の画面に戻る</button>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {

            // --- ▼▼▼ 【修正点】▼▼▼ ---
            // Supabaseの初期化コードを <head> からここに「移動」しました。
            // これで、これ以降のコードが `sbClient` を正しく認識できます。
            const SUPABASE_URL = 'https://noekuogzxgyrfmnojelx.supabase.co'; // 貼り替えてください
            const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im5vZWt1b2d6eGd5cmZtbm9qZWx4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjE1NjUzNDAsImV4cCI6MjA3NzE0MTM0MH0.gycsf6Kcl-RWq6Syr-CHEg3Cf5Ww7CJGasfkUND6ipY'; // 貼り替えてください
            
            // 'supabase' (グローバルオブジェクト) を使って 'sbClient' (私たちの道具) を作成
            const sbClient = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
            // --- ▲▲▲ 【修正点】▲▲▲ ---


            let tempImageDataUrl = null;

            const appState = {
                currentScreen: 'appreciation-screen',
                selectedWords: {},
                myPost: null, // 自分が投稿した内容を一時的に保持
                freeTexts: {}, // ▼①③
                summaryText: ''  // ▼①
            };

            const screens = {
                appreciation: document.getElementById('appreciation-screen'),
                summary: document.getElementById('summary-screen'),
                gallery: document.getElementById('gallery-screen'),
            };

            const wordData = {
                intuitive: [
                    ["温かい", "冷たい"], ["柔らかい", "硬い"], ["優しい", "厳しい"], ["穏やか", "鋭い"],
                    ["力強い", "柔和な"], ["たくましい", "しなやかな"], ["重厚", "軽快"], ["素朴", "洗練"],
                    ["安定", "変化"], ["均整の取れた", "自由な"], ["伸びやか", "端正"], ["おおらか", "緻密"],
                    ["豪快", "優雅"], ["大胆", "繊細"]
                ],
                analytical_tengaku: [
                    ["太い", "細い"], ["長い", "短い"], ["抑揚のある", "一定した"], ["直線的", "曲線的"],
                    ["急な", "緩やかな"], ["とがった", "丸みのある"], ["露鋒", "蔵鋒"], ["開放的", "緊密な"]
                ],
                analytical_jikei: [
                    ["背勢", "向勢"], ["縦長", "横長"]
                ],
                // ▼②章法のデータを追加
                analytical_layout: [
                    ["行が揺れている", "行がまっすぐ"], ["余白が生きている", "余白が窮屈"],
                    ["墨色に変化がある", "墨色が単調"], ["統一感がある", "変化に富む"]
                ]
            };
           
            const teacherModal = document.getElementById('teacher-modal');
            const teacherSettingsBtn = document.getElementById('teacher-settings-btn');
            const cancelTeacherSettingsBtn = document.getElementById('cancel-teacher-settings');
            const saveTeacherSettingsBtn = document.getElementById('save-teacher-settings');
            const artworkTitleEl = document.getElementById('artwork-title');
            const artworkImgEl = document.getElementById('artwork-image');
            const pasteArea = document.getElementById('paste-area');
            const imagePreview = document.getElementById('image-preview');
            const fileInput = document.getElementById('file-input');
            const pasteInstruction = document.getElementById('paste-instruction');

            let panzoomInstance = null; // ▼④

            function createWordButtons(words, category) {
                const container = document.createElement('div');
                const colors = { intuitive: 'bg-orange-50', analytical_tengaku: 'bg-cyan-50', analytical_jikei: 'bg-teal-50', analytical_layout: 'bg-violet-50' };
                container.className = `flex justify-center items-center space-x-2 ${colors[category] || 'bg-gray-50'} p-3 rounded-md`;
                const [word1, word2] = words;
                const pairId = words.join('-');
                const btn1 = document.createElement('button');
                btn1.textContent = word1;
                btn1.className = `word-button w-1/2 py-2 px-3 rounded-md text-sm md:text-base ${category}`;
                btn1.dataset.word = word1; btn1.dataset.category = category; btn1.dataset.pairId = pairId;
                const btn2 = document.createElement('button');
                btn2.textContent = word2;
                btn2.className = `word-button w-1/2 py-2 px-3 rounded-md text-sm md:text-base ${category}`;
                btn2.dataset.word = word2; btn2.dataset.category = category; btn2.dataset.pairId = pairId;
                const separator = document.createElement('span');
                separator.textContent = 'ー';
                separator.className = 'text-gray-400';
                container.appendChild(btn1); container.appendChild(separator); container.appendChild(btn2);
                return container;
            }

            function initializeWordSelection() {
                const intuitiveContainer = document.getElementById('intuitive-words');
                const tengakuContainer = document.getElementById('analytical-tengaku-words');
                const jikeiContainer = document.getElementById('analytical-jikei-words');
                const layoutContainer = document.getElementById('analytical-layout-words'); // ▼②
                intuitiveContainer.innerHTML = ''; tengakuContainer.innerHTML = ''; jikeiContainer.innerHTML = ''; layoutContainer.innerHTML = ''; // ▼②
                wordData.intuitive.forEach(pair => intuitiveContainer.appendChild(createWordButtons(pair, 'intuitive')));
                wordData.analytical_tengaku.forEach(pair => tengakuContainer.appendChild(createWordButtons(pair, 'analytical_tengaku')));
                wordData.analytical_jikei.forEach(pair => jikeiContainer.appendChild(createWordButtons(pair, 'analytical_jikei')));
                wordData.analytical_layout.forEach(pair => layoutContainer.appendChild(createWordButtons(pair, 'analytical_layout'))); // ▼②
            }

            // ▼① 状態をlocalStorageに保存する関数
            function saveAppState() {
                appState.freeTexts['intuitive'] = document.getElementById('free-text-intuitive').value;
                appState.freeTexts['tengaku'] = document.getElementById('free-text-tengaku').value;
                appState.freeTexts['jikei'] = document.getElementById('free-text-jikei').value;
                appState.freeTexts['layout'] = document.getElementById('free-text-layout').value;
                appState.summaryText = document.getElementById('summary-textarea').value;

                const dataToSave = {
                    selectedWords: appState.selectedWords,
                    freeTexts: appState.freeTexts,
                    summaryText: appState.summaryText
                };
                localStorage.setItem('appreciationState', JSON.stringify(dataToSave));
            }

            // ▼① ページ読み込み時に状態を復元する関数
            function loadAppState() {
                const savedState = localStorage.getItem('appreciationState');
                if (savedState) {
                    const data = JSON.parse(savedState);
                    appState.selectedWords = data.selectedWords || {};
                    appState.freeTexts = data.freeTexts || {};
                    appState.summaryText = data.summaryText || '';

                    document.querySelectorAll('.word-button').forEach(btn => {
                        if (appState.selectedWords[btn.dataset.pairId] && appState.selectedWords[btn.dataset.pairId].word === btn.dataset.word) {
                            btn.classList.add('selected');
                        }
                    });

                    if (appState.freeTexts.intuitive) document.getElementById('free-text-intuitive').value = appState.freeTexts.intuitive;
                    if (appState.freeTexts.tengaku) document.getElementById('free-text-tengaku').value = appState.freeTexts.tengaku;
                    if (appState.freeTexts.jikei) document.getElementById('free-text-jikei').value = appState.freeTexts.jikei;
                    if (appState.freeTexts.layout) document.getElementById('free-text-layout').value = appState.freeTexts.layout;
                    document.getElementById('summary-textarea').value = appState.summaryText;
                }
            }

            function handleWordClick(e) {
                if (!e.target.classList.contains('word-button')) return;
                const clickedButton = e.target;
                const { word, category, pairId } = clickedButton.dataset;
                const pairContainer = clickedButton.parentElement;
                const siblingButton = Array.from(pairContainer.children).find(child => child.classList.contains('word-button') && child !== clickedButton);
                if (clickedButton.classList.contains('selected')) {
                    clickedButton.classList.remove('selected');
                    delete appState.selectedWords[pairId];
                } else {
                    clickedButton.classList.add('selected');
                    if (siblingButton) siblingButton.classList.remove('selected');
                    appState.selectedWords[pairId] = { word: word, type: category };
                }
                saveAppState(); // ▼①
            }
           
            function navigateTo(screenId) {
                appState.currentScreen = screenId;
                Object.values(screens).forEach(screen => screen.classList.remove('active'));
                screens[screenId.replace('-screen', '')].classList.add('active');
                window.scrollTo(0, 0);
            }
           
            function updateSummaryView() {
                const listEl = document.getElementById('selected-words-list');
                const textarea = document.getElementById('summary-textarea');
                listEl.innerHTML = '';
                const selectedEntries = Object.values(appState.selectedWords);

                if (selectedEntries.length === 0) {
                    listEl.innerHTML = '<li class="text-gray-500">まだ言葉が選ばれていません。</li>';
                    textarea.value = appState.summaryText || ''; 
                    return;
                }

                selectedEntries.forEach(({ word, type }) => {
                    const li = document.createElement('li');
                    const colors = { 
                        intuitive: 'bg-orange-100 text-orange-800', 
                        analytical_tengaku: 'bg-cyan-100 text-cyan-800', 
                        analytical_jikei: 'bg-teal-100 text-teal-800',
                        analytical_layout: 'bg-violet-100 text-violet-800'
                    };
                    li.className = `px-2 py-1 rounded ${colors[type] || 'bg-gray-100'}`;
                    li.textContent = word;
                    listEl.appendChild(li);
                });
               
                const intuitiveWords = selectedEntries.filter(e => e.type === 'intuitive').map(e => e.word);
                const tengakuWords = selectedEntries.filter(e => e.type === 'analytical_tengaku').map(e => e.word);
                const jikeiWords = selectedEntries.filter(e => e.type === 'analytical_jikei').map(e => e.word);
                const layoutWords = selectedEntries.filter(e => e.type === 'analytical_layout').map(e => e.word); // ▼②
       
                let sentence = '';
                if (intuitiveWords.length > 0) sentence += `この作品からは、【${intuitiveWords.join('】【')}】ような印象を受けました。\n`;
                if (tengakuWords.length > 0 || jikeiWords.length > 0 || layoutWords.length > 0) {
                    sentence += `細部を見ると、`;
                    let details = [];
                    if (tengakuWords.length > 0) details.push(`線は【${tengakuWords.join('】【')}】特徴があり`);
                    if (jikeiWords.length > 0) details.push(`字形は【${jikeiWords.join('】【')}】で`);
                    if (layoutWords.length > 0) details.push(`章法は【${layoutWords.join('】【')}】です`); // ▼②
                    sentence += details.join('、').replace(/、$/, '') + '。\n\n';
                }
                sentence += '（ここに自分の言葉で続けてみましょう）';
                textarea.value = appState.summaryText || sentence;
            }

            function getWordType(word) {
                for (const pair of wordData.intuitive) { if (pair.includes(word)) return 'intuitive'; }
                for (const pair of wordData.analytical_tengaku) { if (pair.includes(word)) return 'analytical_tengaku'; }
                for (const pair of wordData.analytical_jikei) { if (pair.includes(word)) return 'analytical_jikei'; }
                for (const pair of wordData.analytical_layout) { if (pair.includes(word)) return 'analytical_layout'; } // ▼②
                for (const entry of Object.values(appState.selectedWords)) {
                    if (entry.word === word && entry.type.startsWith('analytical')) return entry.type;
                    if (entry.word === word && entry.type === 'intuitive') return 'intuitive';
                }
                return 'unknown';
            }
           
            // ▼⑤ Supabaseから投稿を読み込む (async関数に変更)
            async function updateGalleryView() {
                const gallery = document.getElementById('gallery-posts');
                gallery.innerHTML = '<p class="text-center text-gray-500">鑑賞文を読み込んでいます...</p>';
                
                let allPosts = [];
                try {
                    // "posts"テーブルから、created_atで降順（新しい順）にデータを取得
                    const { data, error } = await sbClient // 'sbClient' を使用
                        .from('posts')
                        .select('*')
                        .order('created_at', { ascending: false });
                    
                    if (error) throw error; // エラーがあればキャッチに投げる

                    data.forEach(post => {
                        // 自分の投稿IDと一致したら「あなた」と表示
                        const isMyPost = appState.myPost && post.id === appState.myPost.id;
                        
                        allPosts.push({
                            ...post, // DBからのデータ
                            author: isMyPost ? 'あなた' : post.author,
                            liked: false // 「いいね」状態は簡略化
                        });
                    });

                    // 自分の投稿がリストに含まれていない場合（DB反映前の即時表示）
                    if (appState.myPost && !allPosts.some(p => p.id === appState.myPost.id)) {
                        allPosts.unshift(appState.myPost);
                    }
                    
                } catch (e) {
                    console.error("Error getting documents: ", e);
                    gallery.innerHTML = '<p class="text-center text-red-500">鑑賞文の読み込みに失敗しました。</p>';
                    return;
                }

                gallery.innerHTML = ''; // ローディング表示を消す
                if (allPosts.length === 0) {
                     gallery.innerHTML = '<p class="text-center text-gray-500">まだ投稿がありません。</p>';
                }

                allPosts.forEach(post => {
                    const postEl = document.createElement('div');
                    postEl.className = 'bg-white p-5 rounded-lg shadow-md';
                   
                    const wordTags = (post.words || []).map(word => {
                        const type = getWordType(word);
                        const tagClassMapping = {
                            'intuitive': 'intuitive-tag',
                            'analytical_tengaku': 'analytical_tengaku-tag',
                            'analytical_jikei': 'analytical_jikei-tag',
                            'analytical_layout': 'analytical_layout-tag'
                        };
                        const tagClass = tagClassMapping[type] || 'intuitive-tag';
                        return `<span class="word-tag ${tagClass}">${word}</span>`;
                    }).join('');

                    postEl.innerHTML = `
                        <div class="flex justify-between items-start mb-3">
                            <h4 class="font-bold text-lg">${post.author}</h4>
                            <button data-post-id="${post.id}" class="like-btn flex items-center space-x-2 text-gray-400 hover:text-red-500 transition-colors">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M3.172 5.172a4 4 0 015.656 0L10 6.343l1.172-1.171a4 4 0 115.656 5.656L10 17.657l-6.828-6.829a4 4 0 010-5.656z" clip-rule="evenodd" /></svg>
                                <span class="like-count font-medium text-sm">${post.likes}</span>
                            </button>
                        </div>
                        <p class="text-gray-700 leading-relaxed mb-4">${post.text}</p>
                        <div class="border-t pt-3 flex flex-wrap gap-2">
                            ${wordTags}
                        </div>
                    `;
                    gallery.appendChild(postEl);
                });
                updateWordCloud(allPosts);
            }

            function updateWordCloud(posts) {
                const cloudContainer = document.getElementById('word-cloud');
                cloudContainer.innerHTML = '';
                const wordCounts = {};
                posts.forEach(post => { (post.words || []).forEach(word => { wordCounts[word] = (wordCounts[word] || 0) + 1; }); });
                const maxCount = Math.max(...Object.values(wordCounts), 1);
                const fontSizes = [1, 1.1, 1.25, 1.4, 1.6];
                const sortedWords = Object.entries(wordCounts).sort((a, b) => b[1] - a[1]);
                sortedWords.forEach(([word, count]) => {
                    const span = document.createElement('span');
                    span.textContent = word;
                    const sizeIndex = Math.floor(((count - 1) / maxCount) * (fontSizes.length - 1));
                    span.style.fontSize = `${fontSizes[sizeIndex]}rem`;
                    span.style.fontWeight = count > maxCount / 2 ? '700' : '400';
                    span.style.opacity = 0.6 + (count / maxCount) * 0.4;
                    cloudContainer.appendChild(span);
                });
            }

            function restartApp() {
                appState.selectedWords = {}; appState.myPost = null; appState.freeTexts = {}; appState.summaryText = '';
                localStorage.removeItem('appreciationState'); // ▼① 
                document.querySelectorAll('.word-button.selected').forEach(btn => btn.classList.remove('selected'));
                document.getElementById('summary-textarea').value = '';
                ['free-text-intuitive', 'free-text-tengaku', 'free-text-jikei', 'free-text-layout'].forEach(id => {
                    document.getElementById(id).value = '';
                });
                navigateTo('appreciation-screen');
            }
           
            function loadArtworkFromStorage() {
                const savedData = localStorage.getItem('artworkData');
                if (savedData) {
                    const { title, url } = JSON.parse(savedData);
                    artworkTitleEl.textContent = `作品名：${title}`; 
                    artworkImgEl.src = url;
                    if (panzoomInstance) panzoomInstance.destroy(); // ▼④
                    panzoomInstance = Panzoom(artworkImgEl, { maxScale: 5, canvas: true }); // ▼④
                }
            }
           
            function handleImageFile(file) {
                if (!file || !file.type.startsWith('image/')) { alert('画像ファイルを選択またはペーストしてください。'); return; }
                const reader = new FileReader();
                reader.onload = (e) => {
                    tempImageDataUrl = e.target.result;
                    imagePreview.src = tempImageDataUrl;
                    imagePreview.classList.remove('hidden'); pasteInstruction.classList.add('hidden');
                };
                reader.readAsDataURL(file);
            }

            // --- イベントリスナーの登録 ---

            document.querySelectorAll('.tab-button').forEach(button => {
                button.addEventListener('click', () => {
                    document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('active'));
                    button.classList.add('active');
                    const tabId = button.dataset.tab;
                    document.getElementById('intuitive-content').classList.toggle('hidden', tabId !== 'intuitive');
                    document.getElementById('analytical-content').classList.toggle('hidden', tabId !== 'analytical');
                });
            });

            document.getElementById('tab-content').addEventListener('click', handleWordClick);
            ['free-text-intuitive', 'free-text-tengaku', 'free-text-jikei', 'free-text-layout'].forEach(id => {
                document.getElementById(id).addEventListener('input', saveAppState);
            });
            document.getElementById('summary-textarea').addEventListener('input', saveAppState);
            document.getElementById('back-to-appreciation').addEventListener('click', () => navigateTo('appreciation-screen'));
            document.getElementById('restart-app').addEventListener('click', restartApp);

            document.getElementById('go-to-summary').addEventListener('click', () => {
                const freeTexts = [
                    { id: 'free-text-intuitive', type: 'intuitive' },
                    { id: 'free-text-tengaku', type: 'analytical_tengaku' },
                    { id: 'free-text-jikei', type: 'analytical_jikei' },
                    { id: 'free-text-layout', type: 'analytical_layout' }
                ];
                freeTexts.forEach(ft => {
                    const input = document.getElementById(ft.id);
                    const word = input.value.trim();
                    if (word) {
                        appState.selectedWords[ft.id] = { word: word, type: ft.type };
                    } else {
                        delete appState.selectedWords[ft.id];
                    }
                });
                saveAppState(); 
                updateSummaryView(); 
                navigateTo('summary-screen'); 
            });

            // ▼⑤ Supabaseに投稿を保存 (async関数に変更)
            document.getElementById('go-to-gallery').addEventListener('click', async () => {
                const text = document.getElementById('summary-textarea').value;
                if (!text || text.includes('（ここに自分の言葉で続けてみましょう）') || text.trim().length === 0) { 
                    alert('鑑賞文を完成させてください。'); return; 
                }

                // Supabaseの `posts` テーブルに保存するデータ
                const newPost = {
                    author: '（匿名の投稿）', // 認証機能がないため
                    text: text, 
                    words: Object.values(appState.selectedWords).map(e => e.word),
                };

                try {
                    // "posts"というテーブルに新しいドキュメントを挿入
                    const { data, error } = await sbClient // 'sbClient' を使用
                        .from('posts')
                        .insert(newPost)
                        .select(); // 挿入したデータを返してもらう

                    if (error) throw error; // エラーがあればキャッチに投げる
                    
                    // 投稿者自身の投稿として保持（IDをDBから取得）
                    appState.myPost = { ...newPost, id: data[0].id, author: 'あなた', likes: 0 }; 
                    
                    localStorage.removeItem('appreciationState'); // ▼①
                    appState.selectedWords = {}; appState.freeTexts = {}; appState.summaryText = '';

                    updateGalleryView(); // ギャラリーを更新
                    navigateTo('gallery-screen');

                } catch (e) {
                    console.error("Error adding document: ", e);
                    alert("投稿に失敗しました。");
                }
            });
           
            // ▼⑤ Supabaseの「いいね」を更新 (async関数に変更)
            document.getElementById('gallery-posts').addEventListener('click', async (e) => {
                const likeBtn = e.target.closest('.like-btn');
                if (!likeBtn) return;

                if (likeBtn.disabled) return; // 連打防止
                likeBtn.disabled = true;

                const postId = likeBtn.dataset.postId;
                
                try {
                    // SupabaseのRPC（データベース関数）を呼び出す
                    const { error } = await sbClient.rpc('increment_likes', { // 'sbClient' を使用
                        post_id_to_inc: postId 
                    });

                    if (error) throw error; // エラーがあればキャッチへ

                    // 画面を再読み込みして最新の状態にする
                    updateGalleryView();

                } catch (error) {
                    console.error("Like failed: ", error);
                    alert("「いいね」に失敗しました。");
                } finally {
                    setTimeout(() => {
                        const newBtn = document.querySelector(`.like-btn[data-post-id="${postId}"]`);
                        if (newBtn) newBtn.disabled = false;
                    }, 500);
                }
            });

            // --- 作品設定モーダルのロジック ---
            teacherSettingsBtn.addEventListener('click', () => {
                document.getElementById('new-artwork-title').value = artworkTitleEl.textContent.replace('作品名：', '');
                tempImageDataUrl = artworkImgEl.src;
                imagePreview.src = tempImageDataUrl;
                imagePreview.classList.remove('hidden'); pasteInstruction.classList.add('hidden');
                teacherModal.classList.remove('hidden');
            });
            cancelTeacherSettingsBtn.addEventListener('click', () => {
                teacherModal.classList.add('hidden');
                tempImageDataUrl = null; imagePreview.src = '';
                imagePreview.classList.add('hidden'); pasteInstruction.classList.remove('hidden');
            });
            saveTeacherSettingsBtn.addEventListener('click', () => {
                const newTitle = document.getElementById('new-artwork-title').value;
                if (newTitle && tempImageDataUrl) {
                    artworkTitleEl.textContent = `作品名：${newTitle}`;
                    artworkImgEl.src = tempImageDataUrl;
                    localStorage.setItem('artworkData', JSON.stringify({ title: newTitle, url: tempImageDataUrl }));
                    
                    if (panzoomInstance) panzoomInstance.destroy(); // ▼④
                    panzoomInstance = Panzoom(artworkImgEl, { maxScale: 5, canvas: true }); // ▼④

                    teacherModal.classList.add('hidden');
                    restartApp(); // ▼①
                } else { alert('作品名と画像を設定してください。'); }
            });
           
            // --- ファイルドロップ・ペーストのロジック ---
            pasteArea.addEventListener('click', () => fileInput.click());
            fileInput.addEventListener('change', (e) => handleImageFile(e.target.files[0]));
            pasteArea.addEventListener('paste', (e) => {
                e.preventDefault(); const items = e.clipboardData.items;
                for (const item of items) { if (item.type.indexOf('image') !== -1) { handleImageFile(item.getAsFile()); break; } }
            });
            ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
                pasteArea.addEventListener(eventName, e => e.preventDefault(), false);
            });
            ['dragenter', 'dragover'].forEach(eventName => { pasteArea.addEventListener(eventName, () => pasteArea.classList.add('drag-over')); });
            ['dragleave', 'drop'].forEach(eventName => { pasteArea.addEventListener(eventName, () => pasteArea.classList.remove('drag-over')); });
            pasteArea.addEventListener('drop', (e) => handleImageFile(e.dataTransfer.files[0]));

            // --- 初期化処理 ---
            initializeWordSelection();
            loadArtworkFromStorage();
            loadAppState(); // ▼① 状態の復元
            
            panzoomInstance = Panzoom(artworkImgEl, { maxScale: 5, canvas: true }); // ▼④
            artworkImgEl.parentElement.addEventListener('wheel', function (event) {
                if (!event.shiftKey) return; // Shiftキー + ホイールでズーム
                panzoomInstance.zoomWithWheel(event);
            });
        });
    </script>
</body>
</html>
