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
　<button id="hg-copy" style="display:none;">複製結果</button>

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
#hg-history li:first-child { border-top: none;
}
#hg-copy {
  margin-top: 0.6rem;
  padding: 0.4rem 0.9rem;
  border: 1px solid var(--border, #555);
  border-radius: 6px;
  background: transparent;
  color: inherit;
  font-size: 0.85rem;
  cursor: pointer;
}
#hg-copy:hover { opacity: 0.8; }
</style>

<script>
(function () {
  var pools = {
    adj: ["失蹤的", "被詛咒的", "身分不明的", "滿身是傷的", "來歷不明的", "眾人畏懼的", "行蹤成謎的", "德高望重的", "臭名昭彰的", "衣衫襤褸的", "沉默寡言的", "野心勃勃的", "走投無路的", "暗中觀察的", "失憶的", "失意的", "珠光寶氣的", "眾人喜愛的", "眾人愛戴的", "深受信任的", "眾人唾棄的", "不被信任的", "能言善道的", "不停說話的", "樂於與人結交的", "心懷憤恨的", "一心復仇的", "充滿冤屈的", "被人懷疑的", "人們以為已經死去的", "身有殘疾的", "樂於分享的", "自私自利的", "喪失心神的", "專注一心的", "玩家許久未見的", "充滿天賦的"],
    person: ["旅店老闆", "流浪法師", "退役傭兵", "村莊長老", "神秘商人", "逃亡貴族", "年輕祭司", "獨行獵人", "叛逃衛兵", "吟遊詩人", "盜賊頭目", "隱居鍊金術士", "孤兒少年", "神殿守衛", "邊境斥候", "獨眼乞丐", "紅衣女孩", "小孩", "帶著一對穿著連身裙的雙胞胎的男人", "病人", "強盜", "學者", "賣藝人", "匆忙的差役", "士兵", "逃兵", "法師", "術士", "巫師", "薩滿", "德魯伊", "聖騎士", "戰士", "部落戰士", "狂戰士", "魔術師", "小丑", "歌手", "樂師", "反抗組織的領袖", "反抗組織的成員", "朋友", "遠房親戚", "盲眼乞丐", "宮廷大臣", "魔法學院的學生", "魔法學院的資優生"],
    event: ["失竊案", "連續失蹤事件", "祭典騷動", "神秘瘟疫", "遺跡崩塌", "商隊遇襲", "將要展開的叛亂", "叛亂", "將要展開的革命", "革命", "詛咒蔓延", "怪物入侵", "秘密結社集會", "繼承權糾紛", "禁忌儀式", "邊境衝突", "礦坑坍塌", "亡靈騷動", "連續殺人事件", "魔法失控", "詭異天象的發生", "群架", "攻城戰役", "圍城戰役", "召喚出惡魔的儀式", "召喚出不死生物的儀式", "召喚出怪物的儀式", "召喚出異界生物的儀式", "夫妻吵架", "魔法師的爭辯", "學者的辯論", "成功的魔法實驗", "失敗的魔法實驗", "哥布林的吵架", "狗頭人的吵架", "龍對城市的攻擊", "遭敵軍包圍的戰鬥", "喪禮", "婚禮", "貴族的婚禮", "互相敵視的家族的紛爭", "魔王的復甦", "顛覆王國的陰謀", "魔法學院的運動會", "魔法學院的考試", "魔法學院的考試舞弊", "兄弟吵架", "兄妹吵架", "姐弟吵架", "姐妹吵架", "農民暴動", "情侶吵架", "失敗的暗殺", "成功的暗殺", "預謀犯罪的會議"],
    time: ["黎明前", "滿月之夜", "收穫祭當天", "連續大雨的第三天", "集市開市日", "冬至前夕", "日蝕發生時", "深夜時分", "新王加冕當日", "節慶的最後一晚", "夏至前夕", "開始下雪的夜晚", "多日大雪紛飛的夜晚", "正舉辦貴族的宴會", "日正當中時", "黃昏時", "晴朗的夜晚", "沒有月光的夜晚", "充滿濃霧的早晨", "充滿濃霧的夜晚", "開戰前夕", "雷雨交加的夜晚", "宜人的秋日午後", "春天來臨時", "新年的第一天", "今年的最後一日", "節慶的第一晚", "城市異常冷清的夜晚", "城市熱鬧的夜晚", "宗教儀式進行時", "魔法學院開學的第一天", "魔法學院畢業典禮的當天"],
    place: ["廢棄礦坑", "邊境小鎮", "古老墓園", "森林深處的神殿", "河岸碼頭", "貴族莊園", "地下水道", "山間隘口", "荒廢燈塔", "集市廣場", "修道院遺址", "邊境哨站", "貧民窟", "沙漠中央", "紮營處", "城市街道", "驛站", "武器店內", "鍊金術師的店內", "藥劑師的店內", "好友家中", "神廟", "王宮內", "國王的王座前", "魔法學院的教室", "競技場", "龍的巢穴", "哥布林的巢穴", "精靈的城市", "激戰的戰場中", "屍橫遍野的戰場上", "魔法學院的密室", "魔法學院的校長室", "魔法學院的食堂", "深山", "山頂", "王國最高峰的山頂", "半山腰", "山腳下", "山林的溪流旁", "山中的一間小屋", "囚車上", "地牢中", "貴族的金庫", "王國的寶物庫", "當舖", "銀行"],
    item: ["一枚古老的印章戒指", "一封未拆封的密信", "一把來歷不明的鑰匙", "一塊發光的碎片", "一份殘破的地圖", "一只裝著奇怪液體的瓶子", "一副刻著符文的護符", "一本上鎖的日誌", "一枚陌生的硬幣", "一件染血的斗篷", "一本上鎖的魔法書", "一本帶有詛咒的書", "一把斷裂的劍", "一顆不明的蛋", "一本藏有祕密的帳本", "一份遺囑", "一個帶著笑容的娃娃", "一塊倒轉的懷錶", "一份有特殊標記的地圖", "一本暗號書", "一本以密碼寫成的書", "一封有領主封蠟的信", "一件華麗的首飾", "一個發出不祥音調的音樂盒", "一盒似乎具有魔法的桌遊", "一副用於占卜的紙牌", "一件會發出奇怪聲響的木盒", "一個鼓滿的錢袋", "一瓶具有治療效果的藥水", "一顆動過手腳的骰子", "一袋稀有的菸草", "一件魔法學院的制服", "一把會說話的劍", "一份魔法學院的作業", "一份國家魔法師考試的試卷與答案", "一個反抗組織的信物", "一枚國王的玉璽"]
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
    var copyBtn = document.getElementById("hg-copy");
    copyBtn.style.display = "inline-block";
    copyBtn.onclick = function () {
      navigator.clipboard.writeText(sentence).then(function () {
        copyBtn.textContent = "已複製！";
        setTimeout(function () {
          copyBtn.textContent = "複製結果";
        }, 1500);
      });
    };

    var history = loadHistory();
    history.unshift(sentence);
    if (history.length > 5) history = history.slice(0, 5);
    saveHistory(history);
    renderHistory();
  });

  renderHistory();
})();
</script>
