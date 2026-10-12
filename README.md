# PLC
plc作业
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>八段数码管</title>
<style>
  * { box-sizing: border-box; }

  body {
    margin: 0;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 24px;
    background: radial-gradient(circle at 50% 25%, #1c2130 0%, #0a0c11 65%);
    font-family: system-ui, -apple-system, "Segoe UI", "Microsoft YaHei", sans-serif;
    color: #e8ebf2;
  }

  .app {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 26px;
  }

  h1 {
    margin: 0;
    font-size: 17px;
    font-weight: 600;
    letter-spacing: .14em;
    color: #8b93a7;
  }

  /* ===== 数码管外壳 ===== */
  .tube-frame {
    padding: 26px 30px;
    background: linear-gradient(150deg, #262a34, #12141a);
    border-radius: 26px;
    border: 1px solid rgba(255, 255, 255, .07);
    box-shadow:
      0 24px 50px rgba(0, 0, 0, .6),
      inset 0 1px 0 rgba(255, 255, 255, .08);
  }

  .tube {
    position: relative;
    width: 180px;
    height: 306px;
  }

  /* ===== 单个小灯（段） ===== */
  .seg {
    position: absolute;
    background: #1c1f27;
    border-radius: 8px;
    box-shadow: inset 0 0 0 1px rgba(255, 255, 255, .03);
    transition: background .18s ease, box-shadow .18s ease;
  }

  .seg.on {
    background: #ff3b30;
    box-shadow:
      0 0 10px #ff3b30,
      0 0 26px rgba(255, 59, 48, .65),
      inset 0 0 0 1px rgba(255, 255, 255, .25);
  }

  .seg.h { width: 120px; height: 22px; left: 30px; }
  .seg.v { width: 22px;  height: 120px; }

  /* 七段 + 小数点 的位置 */
  .seg[data-seg="a"]  { top: 0;    left: 30px;  }
  .seg[data-seg="b"]  { top: 22px; left: 128px; }
  .seg[data-seg="c"]  { top: 164px; left: 128px; }
  .seg[data-seg="d"]  { top: 284px; left: 30px;  }
  .seg[data-seg="e"]  { top: 164px; left: 30px;  }
  .seg[data-seg="f"]  { top: 22px;  left: 30px;  }
  .seg[data-seg="g"]  { top: 142px; left: 30px;  }
  .seg[data-seg="dp"] {
    top: 284px; left: 158px;
    width: 22px; height: 22px;
    border-radius: 50%;
  }

  /* ===== 控制区 ===== */
  .controls {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 14px;
  }

  input[type="text"] {
    width: 190px;
    padding: 12px 16px;
    font-size: 20px;
    letter-spacing: .12em;
    text-align: center;
    color: #fff;
    background: #14161d;
    border: 1px solid rgba(255, 255, 255, .1);
    border-radius: 12px;
    outline: none;
    transition: border-color .2s, box-shadow .2s;
  }
  input[type="text"]::placeholder { color: #4d566a; letter-spacing: .1em; }
  input[type="text"]:focus {
    border-color: #ff3b30;
    box-shadow: 0 0 0 3px rgba(255, 59, 48, .18);
  }

  .keypad {
    display: grid;
    grid-template-columns: repeat(6, 46px);
    gap: 8px;
  }

  .keypad button {
    height: 46px;
    border-radius: 11px;
    border: 1px solid rgba(255, 255, 255, .09);
    background: #1b1f27;
    color: #dfe4ee;
    font-size: 16px;
    font-family: inherit;
    cursor: pointer;
    transition: background .15s, transform .08s, color .15s;
  }
  .keypad button:hover { background: #262c37; }
  .keypad button:active { transform: scale(.93); }
  .keypad button.wide { grid-column: span 2; font-size: 14px; color: #8b93a7; }
</style>
</head>
<body>

<div class="app">
  <h1>八段数码管 · 8 个小灯</h1>

  <div class="tube-frame">
    <div class="tube" id="tube">
      <!-- 七段 + 小数点 -->
      <div class="seg h" data-seg="a"></div>
      <div class="seg v" data-seg="b"></div>
      <div class="seg v" data-seg="c"></div>
      <div class="seg h" data-seg="d"></div>
      <div class="seg v" data-seg="e"></div>
      <div class="seg v" data-seg="f"></div>
      <div class="seg h" data-seg="g"></div>
      <div class="seg"   data-seg="dp"></div>
    </div>
  </div>

  <div class="controls">
    <input id="valueInput" type="text" maxlength="2" placeholder="输入 0-9" autocomplete="off" />

    <div class="keypad" id="keypad">
      <button data-value="0">0</button>
      <button data-value="1">1</button>
      <button data-value="2">2</button>
      <button data-value="3">3</button>
      <button data-value="4">4</button>
      <button data-value="5">5</button>
      <button data-value="6">6</button>
      <button data-value="7">7</button>
      <button data-value="8">8</button>
      <button data-value="9">9</button>
      <button data-value=".">·</button>
      <button data-value="clear" class="wide">清空</button>
    </div>
  </div>
</div>

<script>
  /* 每个数字点亮的段 */
  const DIGIT_MAP = {
    '0': ['a', 'b', 'c', 'd', 'e', 'f'],
    '1': ['b', 'c'],
    '2': ['a', 'b', 'g', 'e', 'd'],
    '3': ['a', 'b', 'g', 'c', 'd'],
    '4': ['f', 'g', 'b', 'c'],
    '5': ['a', 'f', 'g', 'c', 'd'],
    '6': ['a', 'f', 'g', 'e', 'c', 'd'],
    '7': ['a', 'b', 'c'],
    '8': ['a', 'b', 'c', 'd', 'e', 'f', 'g'],
    '9': ['a', 'b', 'c', 'd', 'f', 'g']
  };

  const segEls = {};
  document.querySelectorAll('.seg').forEach(el => {
    segEls[el.dataset.seg] = el;
  });

  const input = document.getElementById('valueInput');

  /* 根据当前状态点亮/熄灭小灯 */
  function render(digit, dot) {
    const on = new Set(digit ? DIGIT_MAP[digit] : []);
    if (dot) on.add('dp');

    for (const key in segEls) {
      segEls[key].classList.toggle('on', on.has(key));
    }
  }

  /* 解析输入框内容 */
  function updateFromInput() {
    const raw = input.value;
    const m = raw.match(/\d/);              // 取第一个数字
    render(m ? m[0] : null, raw.includes('.'));
  }

  /* 输入框过滤：只保留数字和一个小数点 */
  input.addEventListener('input', () => {
    let v = input.value.replace(/[^0-9.]/g, '');
    const i = v.indexOf('.');
    if (i !== -1) {
      v = v.slice(0, i + 1) + v.slice(i + 1).replace(/\./g, '');
    }
    input.value = v;
    updateFromInput();
  });

  /* 虚拟按键 */
  document.getElementById('keypad').addEventListener('click', e => {
    const btn = e.target.closest('button');
    if (!btn) return;

    const v = btn.dataset.value;
    const cur = input.value;

    if (v === 'clear') {
      input.value = '';
    } else if (v === '.') {
      const d = (cur.match(/\d/) || [''])[0];
      input.value = cur.includes('.') ? d : d + '.';
    } else {
      input.value = v + (cur.includes('.') ? '.' : '');
    }

    updateFromInput();
    input.focus();
  });

  /* 键盘直接输入（输入框未聚焦时） */
  window.addEventListener('keydown', e => {
    if (e.target === input) return;

    if (e.key >= '0' && e.key <= '9') {
      input.value = e.key + (input.value.includes('.') ? '.' : '');
      updateFromInput();
    } else if (e.key === '.') {
      const d = (input.value.match(/\d/) || [''])[0];
      input.value = d + '.';
      updateFromInput();
    } else if (e.key === 'Backspace' || e.key === 'Escape') {
      input.value = '';
      updateFromInput();
    }
  });

  /* 初始显示 0 */
  input.value = '0';
  updateFromInput();
</script>

</body>
</html>
