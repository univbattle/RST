<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>수강신청 반응속도 테스트</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    @font-face {
      font-family: 'Ownglyph_ParkDaHyun';
      src: url('https://fastly.jsdelivr.net/gh/projectnoonnu/2411-3@1.0/Ownglyph_ParkDaHyun.woff2') format('woff2');
    }

    body {
      font-family: 'Ownglyph_ParkDaHyun', sans-serif;
      text-align: center;
      background: #f9f9f9;
      margin: 0;
      padding: 0;
    }

    h1 {
      margin-top: 20px;
    }

    #speed {
      font-weight: bold;
    }

    #spawnArea {
      margin: 20px auto;
      width: 600px;
      height: 400px;
      border: 3px dashed #888;
      border-radius: 12px;
      position: relative;
    }

    .checkbox-item {
      position: absolute;
      display: none;
      background: white;
      padding: 8px 12px;
      border-radius: 8px;
      box-shadow: 0 0 5px rgba(0,0,0,0.2);
      font-size: 16px;
    }

    button {
      font-size: 18px;
      padding: 10px 20px;
      margin-top: 10px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }

    #startBtn {
      background: #007bff;
      color: white;
    }

    #kakaoShareBtn {
      background-color: #fee500;
      color: #3c1e1e;
      margin-top: 10px;
      display: none;
    }

    #resultPopup {
      display: none;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 300px;
      background: #ffff66;
      padding: 20px;
      border: 2px solid black;
      border-radius: 10px;
      z-index: 10;
    }

    #resultPopup button {
      margin-top: 10px;
    }

    #closeBtn {
      background: crimson;
      color: white;
    }

    #restartBtn {
      background: #007bff;
      color: white;
    }

    @media (max-width: 768px) {
      #spawnArea {
        width: 90%;
        height: 500px;
      }

      .checkbox-item {
        font-size: 18px;
      }
    }
  </style>
</head>
<body>
  <h1>🎓 수강신청 반응속도 테스트</h1>
  <p>📶 인터넷 속도: <span id="speed">확인 중...</span> Mbps (평균)</p>
  <button id="startBtn">테스트 시작</button>
  <div id="spawnArea">
    <div id="resultPopup">
      <div id="resultContent"></div>
      <button id="restartBtn">🔄 다시하기</button>
      <button id="closeBtn">❌ 닫기</button><br>
      <button id="kakaoShareBtn">📢 카카오톡으로 공유</button>
    </div>
  </div>

  <script src="https://developers.kakao.com/sdk/js/kakao.min.js"></script>
  <script>
    Kakao.init('2027b405148a706519d8960d4db5505d');

    let totalReactionTime = 0, currentIndex = 0;
    const maxBoxes = 3, delayOptions = [500, 1000, 1500, 2000], checkboxes = [];
    const spawnArea = document.getElementById("spawnArea");
    const resultPopup = document.getElementById("resultPopup");
    const resultContent = document.getElementById("resultContent");
    const shareBtn = document.getElementById("kakaoShareBtn");
    const closeBtn = document.getElementById("closeBtn");
    const restartBtn = document.getElementById("restartBtn");
    const startBtn = document.getElementById("startBtn");
    const speedDisplay = document.getElementById("speed");

    async function measureSpeed() {
      const url = "https://upload.wikimedia.org/wikipedia/commons/3/3f/Fronalpstock_big.jpg";
      const sizeMB = 2.2;
      let total = 0;

      for (let i = 0; i < 3; i++) {
        const start = performance.now();
        await fetch(url + "?t=" + Date.now(), { cache: "no-store" });
        const end = performance.now();
        total += (sizeMB * 8 / ((end - start) / 1000));
      }

      speedDisplay.textContent = (total / 3).toFixed(2);
    }

    measureSpeed();

    function resetGameUI() {
      resultPopup.style.display = "none";
      resultContent.innerHTML = "";
      shareBtn.style.display = "none";
      spawnArea.innerHTML = "";
      spawnArea.appendChild(resultPopup);
      startBtn.disabled = false;
      startBtn.textContent = "테스트 시작";
    }

    startBtn.onclick = () => {
      startBtn.disabled = true;
      resetGameUI();
      totalReactionTime = 0;
      currentIndex = 0;
      checkboxes.length = 0;
      countdown(3);
    };

    restartBtn.onclick = () => {
      startBtn.click(); // 다시 시작
    };

    closeBtn.onclick = () => {
      resultPopup.style.display = "none";
    };

    function countdown(sec) {
      if (sec > 0) {
        startBtn.textContent = `${sec}초 후 시작`;
        setTimeout(() => countdown(sec - 1), 1000);
      } else {
        startBtn.textContent = "진행 중...";
        startGame();
      }
    }

    function startGame() {
      for (let i = 0; i < maxBoxes; i++) {
        const el = document.createElement("div");
        el.className = "checkbox-item";
        el.innerHTML = `<label><input type="checkbox"> 전공 ${i+1}</label>`;
        spawnArea.appendChild(el);
        checkboxes.push({ el, shownAt: 0 });
      }
      showNext();
    }

    function showNext() {
      if (currentIndex >= maxBoxes) return showResult();

      const delay = delayOptions[Math.floor(Math.random() * delayOptions.length)];
      setTimeout(() => {
        const item = checkboxes[currentIndex];
        const { x, y } = randomPos();
        item.el.style.left = x + "px";
        item.el.style.top = y + "px";
        item.el.style.display = "block";
        item.shownAt = performance.now();

        const input = item.el.querySelector("input");
        input.addEventListener("click", () => {
          if (input.checked && !input.disabled) {
            totalReactionTime += performance.now() - item.shownAt;
            input.disabled = true;
            currentIndex++;
            showNext();
          }
        }, { once: true });
      }, delay);
    }

    function randomPos() {
      const w = spawnArea.clientWidth, h = spawnArea.clientHeight;
      return {
        x: Math.floor(Math.random() * (w - 120)),
        y: Math.floor(Math.random() * (h - 40))
      };
    }

    function getGrade(sec) {
      if (sec <= 8) return { label: "1등급 🎉", image: "https://ifh.cc/g/mKzKh5.png" };
      else if (sec <= 10) return { label: "2등급 👍", image: "https://ifh.cc/g/mKzKh5.png" };
      else if (sec <= 13) return { label: "3등급 😀", image: "https://ifh.cc/g/mKzKh5.png" };
      else if (sec <= 18) return { label: "4등급 😅", image: "https://ifh.cc/g/mKzKh5.png" };
      else return { label: "5등급 😢", image: "https://ifh.cc/g/mKzKh5.png" };
    }

    function showResult() {
      const sec = (totalReactionTime / 1000).toFixed(2);
      const grade = getGrade(sec);

      resultContent.innerHTML = `
        <p>⏱ 총 반응 시간: ${sec}초</p>
        <p>📊 수강신청 등급: <strong>${grade.label}</strong></p>
        <img src="${grade.image}" alt="${grade.label}" width="200" />
      `;

      resultPopup.style.display = "block";
      shareBtn.style.display = "inline-block";
    }

    shareBtn.onclick = () => {
      Kakao.Share.sendDefault({
        objectType: 'feed',
        content: {
          title: '내 수강신청 반응속도 등급은?',
          description: '당신도 도전해보세요!',
          imageUrl: 'https://ifh.cc/g/mKzKh5.png',
          link: {
            mobileWebUrl: location.href,
            webUrl: location.href
          }
        },
        buttons: [{
          title: '테스트 하러 가기',
          link: {
            mobileWebUrl: location.href,
            webUrl: location.href
          }
        }]
      });
    };
  </script>
</body>
</html>
