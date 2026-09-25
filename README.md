<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0, user-scalable=yes">
<title>謎解きクイズラリー</title>
<style>
  /* 基本設定：スクロールなしで1画面に収める */
  body, html {
    margin: 0;
    padding: 0;
    font-family: 'Helvetica Neue', Arial, 'Hiragino Kaku Gothic ProN', 'Hiragino Sans', Meiryo, sans-serif;
    background-color: #f8f9fa;
    height: 100%;
  }

  /* ホーム画面のレイアウト */
  #home-screen {
    display: flex;
    flex-direction: column;
    height: 100vh;
    padding: 15px;
    box-sizing: border-box;
  }

  /* ヘッダー部 */
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 10px;
  }
  .header .icon-zoom {
    font-size: 24px;
  }
  .header .title {
    font-size: 1.2rem;
    font-weight: bold;
    color: #333;
  }
  .header .spacer {
    width: 24px;
  }

  /* 謎の一覧（縦1列表示） */
  .list-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  .list-item {
    flex: 1;
    background: #ffffff;
    border: 2px solid #ddd;
    border-radius: 8px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
    cursor: pointer;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
    transition: transform 0.1s;
  }
  .list-item:active {
    transform: scale(0.97);
  }
  
  /* クリア済みのマスのデザイン */
  .list-item.solved {
    background: #e8f5e9;
    border-color: #4caf50;
  }
  
  .quiz-label {
    font-size: 1rem;
    font-weight: bold;
    color: #555;
  }
  .quiz-answer {
    font-size: 1.2rem;
    font-weight: bold;
    color: #333;
  }
  
  /* 普段のクリア文字は緑色 */
  .list-item.solved .quiz-answer {
    color: #2e7d32;
  }
  
  /* 指定した特定の文字だけ目立たせる（赤色・下線） */
  .highlight-char {
    color: #e53935 !important;
    border-bottom: 2px solid #e53935;
    padding-bottom: 2px;
  }

  /* 最終問題ボタンのエリア */
  .final-btn-container {
    padding-top: 15px;
  }
  .btn-final {
    background-color: #ff9800;
    color: white;
    margin-bottom: 0; /* 下部の隙間を無くす */
  }

  /* 各種画面の共通レイアウト（初期は非表示） */
  .screen-overlay {
    display: none;
    height: 100vh;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: #ffffff;
    padding: 20px;
    box-sizing: border-box;
  }
  
  .input-container {
    width: 100%;
    max-width: 320px;
    text-align: center;
  }
  .input-container h2 {
    margin-top: 0;
    color: #333;
  }
  .input-container p {
    color: #666;
    margin-bottom: 20px;
  }
  input[type="text"] {
    width: 100%;
    padding: 12px;
    font-size: 1.2rem;
    border: 2px solid #ccc;
    border-radius: 8px;
    box-sizing: border-box;
    margin-bottom: 15px;
    text-align: center;
  }
  button {
    width: 100%;
    padding: 14px;
    font-size: 1.1rem;
    font-weight: bold;
    border: none;
    border-radius: 8px;
    margin-bottom: 10px;
    cursor: pointer;
  }
  .btn-submit {
    background-color: #007aff;
    color: white;
  }
  .btn-back {
    background-color: #e0e0e0;
    color: #333;
  }
  .error-msg {
    color: #d32f2f;
    font-size: 0.9rem;
    margin-bottom: 15px;
    display: none;
  }

  /* クリア画面専用のデザイン */
  #clear-screen {
    background-color: #fffde7;
  }
  .clear-title {
    color: #fbc02d;
    font-size: 2rem;
    margin-bottom: 15px;
  }
</style>
</head>
<body>

  <!-- ホーム画面 -->
  <div id="home-screen">
    <div class="header">
      <div class="icon-zoom">🔍</div>
      <div class="title">クイズラリー</div>
      <div class="spacer"></div>
    </div>
    <div class="list-container" id="quiz-list">
      <!-- ここにJavaScriptでマスの要素が生成されます -->
    </div>
    <!-- 追加：最終問題へのボタン -->
    <div class="final-btn-container">
      <button class="btn-final" onclick="openFinalScreen()">最終問題に挑戦する</button>
    </div>
  </div>

  <!-- 各謎の解答入力画面 -->
  <div id="input-screen" class="screen-overlay">
    <div class="input-container">
      <h2 id="current-quiz-title">謎 1</h2>
      <p>この謎の答えを入力してください</p>
      
      <input type="text" id="answer-input" placeholder="答えを入力" autocomplete="off">
      <div class="error-msg" id="error-message">答えが違います。もう一度考えてみよう！</div>
      
      <button class="btn-submit" onclick="checkAnswer()">解答する</button>
      <button class="btn-back" onclick="goHome()">一覧に戻る</button>
    </div>
  </div>

  <!-- 追加：最終問題の解答入力画面 -->
  <div id="final-screen" class="screen-overlay">
    <div class="input-container">
      <h2>最終問題</h2>
      <p>集めた8つの文字を順番に入力してください</p>
      
      <input type="text" id="final-answer-input" placeholder="最終キーワードを入力" autocomplete="off">
      <div class="error-msg" id="final-error-message">キーワードが違います。集めた文字を確認しよう！</div>
      
      <button class="btn-submit" onclick="checkFinalAnswer()" style="background-color: #ff9800;">解答する</button>
      <button class="btn-back" onclick="goHomeFromFinal()">一覧に戻る</button>
    </div>
  </div>

  <!-- 追加：ゲームクリア画面 -->
  <div id="clear-screen" class="screen-overlay">
    <div class="input-container">
      <h1 class="clear-title">🎉 CLEAR! 🎉</h1>
      <p>すべての謎を解き明かし、<br>最終キーワードを見つけ出しました！</p>
      <p style="font-weight: bold; margin-top: 20px;">おめでとうございます！</p>
    </div>
  </div>

  <script>
    // --- 謎の設定 ---
    const quizzes = [
      { id: 1, label: "謎 1", answer: "すなはま", highlightPos: 4, solved: false },
      { id: 2, label: "謎 2", answer: "しーらかんす",      highlightPos: 2, solved: false },
      { id: 3, label: "謎 3", answer: "とーくしょう",      highlightPos: 3, solved: false },
      { id: 4, label: "謎 4", answer: "をかしかりき",        highlightPos: 1, solved: false },
      { id: 5, label: "謎 5", answer: "つあーがいど",    highlightPos: 2, solved: false },
      { id: 6, label: "謎 6", answer: "わすれもの",      highlightPos: 1, solved: false },
      { id: 7, label: "謎 7", answer: "じょうきせん",  highlightPos: 5, solved: false },
      { id: 8, label: "謎 8", answer: "ふぁいなる",  highlightPos: 5, solved: false }
    ];

    // 最終問題の答え（設定したハイライト文字をつなげたもの）
    const FINAL_ANSWER = "としょしりょうしつ";

    let currentQuizId = null;

    // ホーム画面のリストを描画
    function renderList() {
      const list = document.getElementById('quiz-list');
      list.innerHTML = '';

      quizzes.forEach(quiz => {
        const item = document.createElement('div');
        item.className = 'list-item' + (quiz.solved ? ' solved' : '');
        item.onclick = () => openInputScreen(quiz.id);

        const label = document.createElement('div');
        label.className = 'quiz-label';
        label.textContent = quiz.label;

        const answerText = document.createElement('div');
        answerText.className = 'quiz-answer';

        if (quiz.solved) {
          answerText.innerHTML = ''; 
          for (let i = 0; i < quiz.answer.length; i++) {
            const charSpan = document.createElement('span');
            charSpan.textContent = quiz.answer[i];
            
            if (i + 1 === quiz.highlightPos) {
              charSpan.className = 'highlight-char';
            }
            answerText.appendChild(charSpan);
          }
        } else {
          answerText.textContent = '???';
        }

        item.appendChild(label);
        item.appendChild(answerText);
        list.appendChild(item);
      });
    }

    // 各謎の入力画面を開く
    function openInputScreen(id) {
      currentQuizId = id;
      const quiz = quizzes.find(q => q.id === id);

      document.getElementById('current-quiz-title').textContent = quiz.label;
      document.getElementById('answer-input').value = quiz.solved ? quiz.answer : '';
      document.getElementById('error-message').style.display = 'none';

      document.getElementById('home-screen').style.display = 'none';
      document.getElementById('input-screen').style.display = 'flex';
    }

    // 各謎の入力画面からホームに戻る
    function goHome() {
      document.getElementById('input-screen').style.display = 'none';
      document.getElementById('home-screen').style.display = 'flex';
      renderList();
    }

    // 各謎の答え合わせ
    function checkAnswer() {
      const inputElement = document.getElementById('answer-input');
      const userInput = inputElement.value.trim();
      const quiz = quizzes.find(q => q.id === currentQuizId);

      if (quiz.solved && userInput === quiz.answer) {
        goHome();
        return;
      }

      if (userInput === quiz.answer) {
        quiz.solved = true;
        alert('正解です！');
        goHome();
      } else {
        document.getElementById('error-message').style.display = 'block';
      }
    }

    // --- 最終問題用の関数 ---

    // 最終問題画面を開く
    function openFinalScreen() {
      document.getElementById('home-screen').style.display = 'none';
      document.getElementById('final-error-message').style.display = 'none';
      document.getElementById('final-answer-input').value = '';
      document.getElementById('final-screen').style.display = 'flex';
    }

    // 最終問題画面からホームに戻る
    function goHomeFromFinal() {
      document.getElementById('final-screen').style.display = 'none';
      document.getElementById('home-screen').style.display = 'flex';
    }

    // 最終問題の答え合わせ
    function checkFinalAnswer() {
      const inputElement = document.getElementById('final-answer-input');
      const userInput = inputElement.value.trim();

      if (userInput === FINAL_ANSWER) {
        // 正解したらクリア画面を表示
        document.getElementById('final-screen').style.display = 'none';
        document.getElementById('clear-screen').style.display = 'flex';
      } else {
        // 不正解
        document.getElementById('final-error-message').style.display = 'block';
      }
    }

    // 初期化
    window.onload = () => {
      renderList();
    };
  </script>
</body>
</html>
