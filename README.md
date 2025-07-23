# Zzz
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار</title>
  <style>
    body {
      background-color: #1e1e1e;
      color: #fff;
      font-family: 'Segoe UI', sans-serif;
      direction: rtl;
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #fff;
    }
    label {
      display: block;
      margin-top: 15px;
      color: #ddd;
    }
    select, input {
      width: 100%;
      padding: 10px;
      margin-top: 5px;
      background: #333;
      color: #fff;
      border: 1px solid #555;
      border-radius: 5px;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      background: #4caf50;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    .result {
      background: #2c2c2c;
      margin-top: 20px;
      padding: 15px;
      border-radius: 10px;
    }
    .result-item {
      border-bottom: 1px solid #444;
      padding: 10px 0;
    }
    .result-item:last-child {
      border-bottom: none;
    }
  </style>
</head>
<body>

<h1>حاسبة الأسعار</h1>

<label>اختر التصنيف الرئيسي:</label>
<select id="mainCategory" onchange="updateSubTypes()">
  <option value="">-- اختر --</option>
  <option value="windows">نوافذ</option>
  <option value="doors">أبواب</option>
  <option value="sliding">أبواب سحب</option>
</select>

<label>النوع:</label>
<select id="subType">
  <option value="">-- اختر نوعًا أولاً --</option>
</select>

<label>الطول (متر):</label>
<input type="number" id="height" step="0.01" />

<label>العرض (متر):</label>
<input type="number" id="width" step="0.01" />

<label>الكمية:</label>
<input type="number" id="quantity" value="1" />

<label>إضافات:</label>
<select id="addons" onchange="addonInputCheck()">
  <option value="">لا يوجد</option>
  <option value="ستارة">ستارة داخلية</option>
  <option value="شبك الباب">شبك الباب</option>
  <option value="شبك سلايد">شبك سلايد</option>
  <option value="شبك فولدنج">شبك فولدنج</option>
</select>

<div id="addonSizeInputs" style="display: none;">
  <label>طول الستارة (م):</label>
  <input type="number" id="curtainHeight" step="0.01" />
  <label>عرض الستارة (م):</label>
  <input type="number" id="curtainWidth" step="0.01" />
</div>

<button onclick="calculate()">احسب</button>
<button onclick="downloadWord()">حفظ كـ Word</button>

<div class="result" id="results"></div>

<script>
const prices = {
  "دبل جلاس دبل فريم - ثابتة": { price: 34, type: "m", factor: 0.13 },
  "دبل جلاس دبل فريم - حركة": { price: 73, type: "m", factor: 0.13 },
  "دبل جلاس دبل فريم - حركتين": { price: 92, type: "m", factor: 0.13 },
  "دبل جلاس سنجل فريم - ثابتة": { price: 26, type: "m", factor: 0.13 },
  "دبل جلاس سنجل فريم - حركة": { price: 46, type: "m", factor: 0.13 },
  "دبل جلاس سنجل فريم - حركتين": { price: 58, type: "m", factor: 0.13 },
  "سنجل جلاس سنجل فريم - ثابتة": { price: 20, type: "m", factor: 0.13 },
  "سنجل جلاس سنجل فريم - حركة": { price: 43, type: "m", factor: 0.13 },
  "سنجل جلاس سنجل فريم - حركتين": { price: 47, type: "m", factor: 0.13 },
  "نوافذ سلايدنج": { price: 10, type: "m", factor: 0.13, baseOnly: true },
  "النوافذ الكهربائية": { price: 102, type: "m", factor: 0.13 },
  "سكاي لايت بدون مكينة": { price: 56, type: "m", factor: 0.13 },
  "سكاي لايت مع مكينة": { price: 145, type: "m", factor: 0.13 },
  "كارتن وول - ثقيل": { price: 56, type: "m", factor: 0.13 },
  "كارتن وول - خفيف": { price: 45, type: "m", factor: 0.13 },
  "باب مدخل - زينك": { price: 66, type: "m", factor: 0.2, extra: 10 },
  "باب مدخل - ستينلس ستيل": { price: 120, type: "m", factor: 0.2, extra: 10 },
  "باب مدخل - كاست المنيوم": { price: 168, type: "m", factor: 0.2, extra: 10 },
  "باب WPC - فارغ": { price: 45, type: "door", factor: 0.11 },
  "باب WPC - مع خشب": { price: 50, type: "door", factor: 0.11 },
  "باب WPC - ضد الصوت": { price: 60, type: "door", factor: 0.11 },
  "باب WPC - مع فريم المنيوم": { price: 67, type: "door", factor: 0.11 },
  "باب WPC - سلايد": { price: 65, type: "door", factor: 0.11 },
  "باب المنيوم - فارغ": { price: 65, type: "door", factor: 0.11 },
  "باب المنيوم - مع خشب": { price: 75, type: "door", factor: 0.11 },
  "باب المنيوم - فل المنيوم": { price: 85, type: "door", factor: 0.11 },
  "باب المنيوم - مخفي": { price: 110, type: "door", factor: 0.11 },
  "باب المنيوم - خارجي": { price: 61, type: "door", factor: 0.11 },
  "باب دورة مياه - جديد": { price: 55, type: "door", factor: 0.11 },
  "باب دورة مياه - قديم": { price: 45, type: "door", factor: 0.11 },
  "باب دورة مياه - مخفي زجاجي": { price: 65, type: "door", factor: 0.11 },
  "باب سحب - داخلي زجاج": { price: 38, type: "m", factor: 0.13 },
  "باب سحب - داخلي متين": { price: 41, type: "m", factor: 0.13 },
  "باب سحب - خارجي مفتوح": { price: 55, type: "m", factor: 0.13 },
  "باب سحب - خارجي جزئين": { price: 58, type: "m", factor: 0.13 },
  "باب سحب - WPC": { price: 61, type: "m", factor: 0.11 },
  "باب فولدنج - داخلي": { price: 39, type: "m", factor: 0.13 },
  "باب فولدنج - خارجي": { price: 56, type: "m", factor: 0.13 },
  "شتر دور خارجي": { price: 28, type: "m", factor: 0.13 },
  "باب حديقة": { price: 91, type: "m", factor: 0.2 },
  "حاجز درج": { price: 43, type: "m", factor: 0.05 },
  "حاجز حمام": { price: 32, type: "m", factor: 0.05 },
  "ستارة": { price: 26, type: "m", factor: 0 },
  "شبك الباب": { price: 39, type: "m", factor: 0 },
  "شبك فولدنج": { price: 18, type: "m", factor: 0 },
  "شبك سلايد": { price: 14, type: "m", factor: 0 }
};

function updateSubTypes() {
  const subType = document.getElementById("subType");
  const mainCat = document.getElementById("mainCategory").value;
  subType.innerHTML = "<option>-- اختر --</option>";
  for (let key in prices) {
    if (
      (mainCat === "windows" && key.includes("دبل") || key.includes("سنجل") || key.includes("سلايدنج") || key.includes("كارتن") || key.includes("سكاي") || key.includes("النوافذ الكهربائية")) ||
      (mainCat === "doors" && key.includes("باب") && !key.includes("سحب")) ||
      (mainCat === "sliding" && key.includes("باب سحب"))
    ) {
      let opt = document.createElement("option");
      opt.value = key;
      opt.innerText = key;
      subType.appendChild(opt);
    }
  }
}

function addonInputCheck() {
  const addon = document.getElementById("addons").value;
  const addonInputs = document.getElementById("addonSizeInputs");
  addonInputs.style.display = addon === "ستارة" ? "block" : "none";
}

function calculate() {
  const type = document.getElementById("subType").value;
  const h = parseFloat(document.getElementById("height").value);
  const w = parseFloat(document.getElementById("width").value);
  const qty = parseInt(document.getElementById("quantity").value) || 1;
  const addon = document.getElementById("addons").value;
  const curtainH = parseFloat(document.getElementById("curtainHeight").value || 0);
  const curtainW = parseFloat(document.getElementById("curtainWidth").value || 0);

  if (!type || isNaN(h) || isNaN(w)) return;

  const area = h * w;
  const item = prices[type];
  let basePrice = item.baseOnly ? item.price + (area * item.price) : (item.type === "door" ? item.price : item.price * area);
  if (item.extra) basePrice += item.extra;
  const shipping = (area * item.factor * 48);
  let total = (basePrice + shipping) * qty;

  // Add addon if present
  if (addon && prices[addon]) {
    const addArea = addon === "ستارة" ? (curtainH * curtainW) : area;
    total += prices[addon].price * addArea;
  }

  const resultEl = document.getElementById("results");
  const result = document.createElement("div");
  result.className = "result-item";
  result.innerHTML = `
    <strong>النوع:</strong> ${type}<br>
    <strong>المقاس:</strong> ${h} × ${w} م<br>
    <strong>الكمية:</strong> ${qty}<br>
    <strong>سعر الشحن:</strong> ${shipping.toFixed(2)} ريال<br>
    <strong>السعر النهائي:</strong> ${total.toFixed(2)} ريال
  `;
  resultEl.appendChild(result);
}

function downloadWord() {
  let content = document.getElementById("results").innerText;
  const blob = new Blob([content], { type: "application/msword" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "النتائج.doc";
  a.click();
}
</script>

</body>
</html>
