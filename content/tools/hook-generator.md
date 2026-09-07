+++
title = '隨機冒險鉤子及遭遇產生器'
draft = false
summary = '快速產生奇幻冒險鉤子與遭遇——輸入你想要的詞彙，其餘欄位隨機補上，會記住你最近5次的生成結果。'
showtoc = false
+++

點擊下方按鈕，隨機產生一個冒險鉤子或遭遇。你也可以自行輸入特定欄位的內容。

<div id="hook-generator">
  <div class="hg-fields">
    <div class="hg-field">
      <label>形容詞</label>
      <input type="text" id="hg-adj" placeholder="隨機">
    </div>
    <div class="hg-field">
      <label>人物</label>
      <input type="text" id="hg-person" placeholder="隨機">
    </div>
    <div class="hg-field">
      <label>事件</label>
      <input type="text" id="hg-event" placeholder="隨機">
    </div>
    <div class="hg-field">
      <label>時間</label>
      <input type="text" id="hg-time" placeholder="隨機">
    </div>
    <div class="hg-field">
      <label>地點</label>
      <input type="text" id="hg-place" placeholder="隨機">
    </div>
    <div class="hg-field">
      <label>物品</label>
      <input type="text" id="hg-item" placeholder="隨機">
    </div>
  </div>

  <button id="hg-generate">產生冒險鉤子</button>

  <div id="hg-result"></div>

  <div id="hg-history-wrap">
    <h4>最近 5 次生成紀錄</h4>
    <ul id="hg-history"></ul>
  </div>
</div>

<style>
#hook-generator {
  max-width: 640px;
  margin: 1.5rem 0;
  padding: 1.2rem;
  border: 1px solid var(--border, #444);
  border-radius: 10px;
}
.hg-fields {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.7rem;
  margin-bottom: 1rem;
}
.hg-field label {
  display: block;
  font-size: 0.8rem;
  opacity: 0.75;
  margin-bottom: 0.25rem;
}
.hg-field input {
  width: 100%;
  box-sizing: border-box;
  padding: 0.4rem 0.5rem;
  border-radius: 6px;
  border: 1px solid var(--border, #555);
  background: transparent;
  color: inherit;
  font-size: 0.9rem;
}
#hg-generate {
  display: block;
  width: 100%;
  padding: 0.6rem;
  border: none;
  border-radius: 8px;
  background: #d4af37;
  color: #1b1b1d;
  font-weight: bold;
  font-size: 0.95rem;
  cursor: pointer;
}
#hg-generate:hover { opacity: 0.9; }
#hg-result {
  margin-top: 1rem;
  padding: 0.9rem;
  border-radius: 8px;
  background: rgba(212,175,55,0.08);
  border-left: 3px solid #d4af37;
  min-height: 1.5rem;
  line-height: 1.6;
}
#hg-history-wrap {
  margin-top: 1.3rem;
}
#hg-history-wrap h4 {
  font-size: 0.85rem;
  opacity: 0.7;
  margin-bottom: 0.4rem;
}
#hg-history {
  list-style: none;
  padding: 0;
  margin: 0;
  font-size: 0.85rem;
  opacity: 0.85;
}
#hg-history li {
  padding: 0.4rem 0;
  border-top: 1px dashed var(--border, #444);
}
#hg-history li:first-child { border-top: none; }
</style>

<script>
(function () {
  var pools = {
    adj: ["失蹤的", "被詛咒的", "垂垂老矣的", "身分不明的", "滿身是傷的", "來歷不明的", "眾人畏懼的", "行蹤成謎的", "德高望重的", "臭名昭彰的", "衣衫襤褸的", "沉默寡言的", "野心勃勃的", "走投無路的", "暗中觀察的"],
    person: ["旅店老闆", "流浪法師", "退役傭兵", "村莊長老", "神秘商人", "逃亡貴族", "年輕祭司", "獨行獵人", "叛逃衛兵", "吟遊詩人", "盜賊頭目", "隱居鍊金術士", "孤兒少年", "神殿守衛", "邊境斥候", "獨眼乞丐", "紅衣女孩", "小孩", "穿著連身裙的雙胞胎", "帶著外傷的人", "面容枯槁的病人", "結夥的強盜". "學者", "賣藝人", "匆忙的差役"],
    event: ["失竊案", "連續失蹤事件", "祭典騷動", "神秘瘟疫", "遺跡崩塌", "商隊遇襲", "叛亂徵兆", "詛咒蔓延", "怪物入侵", "秘密結社集會", "繼承權糾紛", "禁忌儀式", "邊境衝突", "礦坑坍塌", "亡靈騷動", "連續殺人事件", "魔法失控", "詭異的天象", "群架", "攻城", "圍城", "召喚出惡魔", "召喚出不死生物", "召喚出怪物", "召喚出異界生物", "夫妻吵架", "一群魔法師正在爭辯", "一群學者正在爭辯", "魔法實驗成功", "魔法實驗失敗"],
    time: ["黎明前", "滿月之夜", "收穫祭當天", "連續大雨的第三天", "集市開市日", "冬至前夕", "日蝕發生時", "深夜時分", "新王加冕當日", "節慶的最後一晚"],
    place: ["廢棄礦坑", "邊境小鎮", "古老墓園", "森林深處的神殿", "河岸碼頭", "貴族莊園", "地下水道", "山間隘口", "荒廢燈塔", "集市廣場", "修道院遺址", "邊境哨站"],
    item: ["一枚古老的印章戒指", "一封未拆封的密信", "一把來歷不明的鑰匙", "一塊發光的碎片", "一份殘破的地圖", "一只裝著奇怪液體的瓶子", "一副刻著符文的護符", "一本上鎖的日誌", "一枚陌生的硬幣", "一件染血的斗篷"]
  };

  var fieldMap = {
    adj: "hg-adj",
    person: "hg-person",
    event: "hg-event",
    time: "hg-time",
    place: "hg-place",
    item: "hg-item"
  };

  function pick(arr) {
    return arr[Math.floor(Math.random() * arr.length)];
  }

  function getValue(key) {
    var input = document.getElementById(fieldMap[key]);
    var typed = input.value.trim();
    return typed !== "" ? typed : pick(pools[key]);
  }

  function loadHistory() {
    try {
      var raw = localStorage.getItem("hookGeneratorHistory");
      return raw ? JSON.parse(raw) : [];
    } catch (e) {
      return [];
    }
  }

  function saveHistory(list) {
    try {
      localStorage.setItem("hookGeneratorHistory", JSON.stringify(list));
    } catch (e) {}
  }

  function renderHistory() {
    var list = loadHistory();
    var ul = document.getElementById("hg-history");
    ul.innerHTML = "";
    if (list.length === 0) {
      ul.innerHTML = "<li>尚無生成紀錄</li>";
      return;
    }
    list.forEach(function (text) {
      var li = document.createElement("li");
      li.textContent = text;
      ul.appendChild(li);
    });
  }

  document.getElementById("hg-generate").addEventListener("click", function () {
    var adj = getValue("adj");
    var person = getValue("person");
    var event = getValue("event");
    var time = getValue("time");
    var place = getValue("place");
    var item = getValue("item");

    var sentence = "在" + time + "的" + place + "，一名" + adj + person +
      "捲入了一場" + event + "，隨身帶著" + item + "。";

    document.getElementById("hg-result").textContent = sentence;

    var history = loadHistory();
    history.unshift(sentence);
    if (history.length > 5) history = history.slice(0, 5);
    saveHistory(history);
    renderHistory();
  });

  renderHistory();
})();
</script>
