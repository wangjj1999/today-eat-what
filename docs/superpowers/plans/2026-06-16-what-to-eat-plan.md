# 今天吃什么 — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML app that lets users manage a food list and randomly pick one via a 3-stage rolling animation.

**Architecture:** Single `今天吃什么.html` file with inline `<style>` and `<script>`. Data persists in `localStorage` under key `food-list`. The UI has a header, lottery area with rolling text, action button, and a food list with inline edit/delete controls. A modal overlay handles add/edit forms.

**Tech Stack:** Vanilla HTML5, CSS3 (flexbox, keyframes, media queries), ES6 JavaScript (no frameworks, no build tools).

---

## File Structure

| File | Responsibility |
|------|---------------|
| `今天吃什么.html` | All HTML structure, CSS styles, and JS logic in one file |

---

### Task 1: HTML Skeleton

**Files:**
- Create: `今天吃什么.html`

- [ ] **Step 1: Write the full HTML structure**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>今天吃什么</title>
  <style>
    /* CSS will go here in Task 2 */
  </style>
</head>
<body>
  <!-- Header -->
  <header class="header">
    <h1 class="header-title">🍜 今天吃什么</h1>
    <p class="header-subtitle">选择困难症终结者</p>
  </header>

  <main class="main">
    <!-- Lottery Section -->
    <section class="lottery-section">
      <div class="lottery-box" id="lotteryBox">
        <span class="lottery-label">🎰 点击下方按钮开始抽奖</span>
        <span class="lottery-result" id="lotteryResult"></span>
      </div>
      <button class="btn-lottery" id="btnLottery" disabled>🎲 开抽！</button>
    </section>

    <!-- Food List Section -->
    <section class="list-section">
      <div class="list-header">
        <span class="list-title">📋 可吃清单 (<span id="countDisplay">0</span>)</span>
        <button class="btn-add" id="btnAdd">+ 添加</button>
      </div>
      <ul class="food-list" id="foodList"></ul>
      <p class="empty-hint" id="emptyHint">还没有添加吃的，先加点吧~</p>
    </section>
  </main>

  <!-- Modal Overlay -->
  <div class="modal-overlay" id="modalOverlay">
    <div class="modal" id="modal">
      <h3 class="modal-title" id="modalTitle">添加食物</h3>
      <input
        type="text"
        class="modal-input"
        id="modalInput"
        placeholder="输入食物名称..."
        maxlength="30"
        autocomplete="off"
      />
      <p class="modal-error" id="modalError"></p>
      <div class="modal-actions">
        <button class="btn-cancel" id="btnCancel">取消</button>
        <button class="btn-confirm" id="btnConfirm">确认</button>
      </div>
    </div>
  </div>

  <!-- Delete Confirm Modal -->
  <div class="modal-overlay" id="deleteOverlay">
    <div class="modal modal-sm" id="deleteModal">
      <h3 class="modal-title">确认删除</h3>
      <p class="modal-body" id="deleteMsg">确定删除「XXX」？</p>
      <div class="modal-actions">
        <button class="btn-cancel" id="btnDeleteCancel">取消</button>
        <button class="btn-confirm btn-danger" id="btnDeleteConfirm">删除</button>
      </div>
    </div>
  </div>

  <!-- Toast -->
  <div class="toast" id="toast"></div>

  <script>
    // JS will go here in Task 3+
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify file exists and is valid HTML**

```bash
wc -l 今天吃什么.html
```
Expected: ~60+ lines of HTML structure.

---

### Task 2: CSS Styles — Base + Responsive Layout

**Files:**
- Modify: `今天吃什么.html` (replace `<style>` placeholder)

- [ ] **Step 1: Write the complete CSS**

Replace the `/* CSS will go here in Task 2 */` comment with:

```css
/* ===== Reset & Base ===== */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans SC", sans-serif;
  background: #f9fafb;
  color: #1f2937;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* ===== Header ===== */
.header {
  background: linear-gradient(135deg, #f97316, #ef4444);
  color: #fff;
  text-align: center;
  padding: 20px 16px 18px;
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.25);
}
.header-title { font-size: 22px; font-weight: 800; }
.header-subtitle { font-size: 12px; opacity: 0.8; margin-top: 2px; }

/* ===== Main Container ===== */
.main {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px 16px 32px;
  max-width: 900px;
  width: 100%;
  margin: 0 auto;
}

/* ===== Lottery Section ===== */
.lottery-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.lottery-box {
  width: 100%;
  max-width: 420px;
  background: #fef3c7;
  border: 2px dashed #f59e0b;
  border-radius: 16px;
  padding: 28px 16px;
  text-align: center;
  min-height: 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: border-color 0.3s, background 0.3s;
}
.lottery-box.rolling {
  border-color: #f97316;
  background: #fff7ed;
}
.lottery-box.result {
  border-style: solid;
  border-color: #10b981;
  background: #ecfdf5;
}

.lottery-label {
  font-size: 13px;
  color: #92400e;
}
.lottery-result {
  font-size: 28px;
  font-weight: 800;
  color: #d97706;
  min-height: 40px;
  line-height: 40px;
  word-break: break-all;
}
.lottery-result.bounce {
  animation: resultBounce 0.5s ease;
}

@keyframes resultBounce {
  0%   { transform: scale(1); }
  30%  { transform: scale(1.25); }
  60%  { transform: scale(0.95); }
  100% { transform: scale(1); }
}

.btn-lottery {
  background: linear-gradient(135deg, #f97316, #ef4444);
  color: #fff;
  border: none;
  padding: 14px 56px;
  font-size: 18px;
  font-weight: 700;
  border-radius: 50px;
  cursor: pointer;
  box-shadow: 0 4px 14px rgba(239, 68, 68, 0.35);
  transition: transform 0.15s, box-shadow 0.15s, opacity 0.2s;
  -webkit-tap-highlight-color: transparent;
  user-select: none;
}
.btn-lottery:hover:not(:disabled) { transform: translateY(-2px); box-shadow: 0 6px 20px rgba(239, 68, 68, 0.4); }
.btn-lottery:active:not(:disabled) { transform: translateY(0); }
.btn-lottery:disabled { opacity: 0.5; cursor: not-allowed; }

/* ===== Food List Section ===== */
.list-section { flex: 1; }

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}
.list-title { font-size: 15px; font-weight: 700; }

.btn-add {
  background: #10b981;
  color: #fff;
  border: none;
  padding: 6px 16px;
  font-size: 13px;
  font-weight: 600;
  border-radius: 20px;
  cursor: pointer;
  transition: background 0.15s;
  -webkit-tap-highlight-color: transparent;
}
.btn-add:hover { background: #059669; }

.food-list {
  list-style: none;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  overflow: hidden;
  background: #fff;
  max-height: 360px;
  overflow-y: auto;
}

.food-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 14px;
  border-bottom: 1px solid #f3f4f6;
  transition: background 0.15s;
}
.food-list li:last-child { border-bottom: none; }
.food-list li:hover { background: #f9fafb; }

.food-name {
  font-size: 15px;
  font-weight: 500;
  word-break: break-all;
}

.food-actions {
  display: flex;
  gap: 8px;
  flex-shrink: 0;
  margin-left: 8px;
}

.btn-icon {
  background: none;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  padding: 4px 8px;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.15s;
  -webkit-tap-highlight-color: transparent;
  line-height: 1;
}
.btn-icon:hover { background: #f3f4f6; }
.btn-icon.danger:hover { background: #fef2f2; border-color: #fecaca; }

.empty-hint {
  text-align: center;
  color: #9ca3af;
  font-size: 14px;
  padding: 32px 0;
}
.empty-hint.hidden { display: none; }

/* ===== Modal ===== */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 20px;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s;
}
.modal-overlay.active {
  opacity: 1;
  pointer-events: auto;
}

.modal {
  background: #fff;
  border-radius: 14px;
  padding: 24px 20px;
  width: 100%;
  max-width: 360px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  gap: 14px;
}
.modal-sm { max-width: 320px; }

.modal-title { font-size: 17px; font-weight: 700; text-align: center; }
.modal-body { font-size: 14px; color: #6b7280; text-align: center; }

.modal-input {
  width: 100%;
  padding: 12px 14px;
  border: 2px solid #e5e7eb;
  border-radius: 10px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.15s;
}
.modal-input:focus { border-color: #f97316; }

.modal-error {
  color: #ef4444;
  font-size: 12px;
  min-height: 16px;
  text-align: center;
}

.modal-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.btn-cancel, .btn-confirm {
  padding: 10px 28px;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: background 0.15s;
}
.btn-cancel { background: #f3f4f6; color: #4b5563; }
.btn-cancel:hover { background: #e5e7eb; }
.btn-confirm { background: #f97316; color: #fff; }
.btn-confirm:hover { background: #ea580c; }
.btn-confirm:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-danger { background: #ef4444; }
.btn-danger:hover { background: #dc2626; }

/* ===== Toast ===== */
.toast {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background: #1f2937;
  color: #fff;
  padding: 10px 22px;
  border-radius: 24px;
  font-size: 14px;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.25s;
  z-index: 200;
  white-space: nowrap;
}
.toast.show { opacity: 1; }

/* ===== Emoji Confetti ===== */
.confetti {
  position: fixed;
  font-size: 24px;
  pointer-events: none;
  z-index: 150;
  animation: confettiFall 1.2s ease-out forwards;
}
@keyframes confettiFall {
  0%   { opacity: 1; transform: translateY(0) rotate(0deg) scale(1); }
  100% { opacity: 0; transform: translateY(-120px) rotate(360deg) scale(0.3); }
}

/* ===== PC Responsive: ≥768px ===== */
@media (min-width: 768px) {
  .main {
    flex-direction: row;
    align-items: flex-start;
    gap: 32px;
    padding: 32px 24px;
  }
  .lottery-section { flex: 1; position: sticky; top: 20px; }
  .list-section { flex: 1; }
  .header { padding: 24px 16px 20px; }
  .header-title { font-size: 26px; }
  .lottery-box { padding: 40px 24px; min-height: 140px; }
  .lottery-result { font-size: 34px; line-height: 48px; }
  .food-list { max-height: 460px; }
}
```

- [ ] **Step 2: Verify the CSS is syntactically valid**

Open the file in a browser or validate no obvious CSS typos exist. All selectors use standard properties.

---

### Task 3: JavaScript — Data Layer + Initialization

**Files:**
- Modify: `今天吃什么.html` (replace `<script>` placeholder)

- [ ] **Step 1: Write the data layer JS**

Replace the `// JS will go here in Task 3+` comment with:

```html
  <script>
    // ===== Data Layer =====
    const STORAGE_KEY = 'food-list';

    function loadFoods() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (raw) {
          const parsed = JSON.parse(raw);
          if (Array.isArray(parsed)) return parsed.filter(f => typeof f === 'string' && f.trim());
        }
      } catch (e) { /* localStorage corrupt or unavailable */ }
      return [];
    }

    function saveFoods(foods) {
      try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(foods));
      } catch (e) {
        showToast('存储空间不足，请清理一些数据');
      }
    }

    // In-memory fallback
    let foods = loadFoods();

    // Seed some defaults only on first visit
    if (foods.length === 0) {
      foods = ['兰州拉面', '黄焖鸡米饭', '沙县小吃', '麦当劳', '麻辣烫'];
      saveFoods(foods);
    }
  </script>
```

- [ ] **Step 2: Verify in browser console**

Open `今天吃什么.html` in a browser, open DevTools Console, and run:

```js
JSON.parse(localStorage.getItem('food-list'))
```

Expected: `["兰州拉面", "黄焖鸡米饭", "沙县小吃", "麦当劳", "麻辣烫"]`

---

### Task 4: JavaScript — UI Rendering + DOM References

**Files:**
- Modify: `今天吃什么.html` (append to `<script>` block)

- [ ] **Step 1: Add DOM refs and render function**

Append after the data layer code, before the closing `</script>`:

```js
    // ===== DOM References =====
    const lotteryBox  = document.getElementById('lotteryBox');
    const lotteryResult = document.getElementById('lotteryResult');
    const btnLottery  = document.getElementById('btnLottery');
    const foodList    = document.getElementById('foodList');
    const countDisplay = document.getElementById('countDisplay');
    const emptyHint   = document.getElementById('emptyHint');
    const btnAdd      = document.getElementById('btnAdd');

    // Modal — add/edit
    const modalOverlay = document.getElementById('modalOverlay');
    const modalTitle   = document.getElementById('modalTitle');
    const modalInput   = document.getElementById('modalInput');
    const modalError   = document.getElementById('modalError');
    const btnCancel    = document.getElementById('btnCancel');
    const btnConfirm   = document.getElementById('btnConfirm');

    // Modal — delete
    const deleteOverlay    = document.getElementById('deleteOverlay');
    const deleteMsg        = document.getElementById('deleteMsg');
    const btnDeleteCancel  = document.getElementById('btnDeleteCancel');
    const btnDeleteConfirm = document.getElementById('btnDeleteConfirm');

    // Toast
    const toast = document.getElementById('toast');

    // ===== State =====
    let editIndex = -1;       // -1 = adding, >= 0 = editing
    let deleteIndex = -1;
    let isRolling = false;

    // ===== Render =====
    function renderList() {
      const count = foods.length;
      countDisplay.textContent = count;

      // Empty state
      if (count === 0) {
        emptyHint.classList.remove('hidden');
        foodList.innerHTML = '';
        btnLottery.disabled = true;
        btnLottery.textContent = '🎲 开抽！';
        lotteryResult.textContent = '';
        lotteryResult.classList.remove('bounce');
        lotteryBox.classList.remove('result');
        return;
      }

      emptyHint.classList.add('hidden');
      btnLottery.disabled = false;

      foodList.innerHTML = foods.map((name, i) => `
        <li>
          <span class="food-name">${escapeHtml(name)}</span>
          <span class="food-actions">
            <button class="btn-icon" data-edit="${i}" title="编辑">✏️</button>
            <button class="btn-icon danger" data-delete="${i}" title="删除">🗑️</button>
          </span>
        </li>
      `).join('');
    }

    function escapeHtml(str) {
      const div = document.createElement('div');
      div.textContent = str;
      return div.innerHTML;
    }

    // ===== Initial render =====
    renderList();
```

- [ ] **Step 2: Verify rendering**

Open the file in a browser. Expected:
- 5 default food items visible in the list
- Count shows "(5)"
- Lottery button is enabled
- Empty hint is hidden

---

### Task 5: JavaScript — Modal System (Add & Edit)

**Files:**
- Modify: `今天吃什么.html` (append to `<script>` block)

- [ ] **Step 1: Add modal open/close and save logic**

Append before the closing `</script>`:

```js
    // ===== Modal: Add / Edit =====
    function openAddModal() {
      editIndex = -1;
      modalTitle.textContent = '添加食物';
      modalInput.value = '';
      modalError.textContent = '';
      modalOverlay.classList.add('active');
      setTimeout(() => modalInput.focus(), 150);
    }

    function openEditModal(index) {
      editIndex = index;
      modalTitle.textContent = '编辑食物';
      modalInput.value = foods[index];
      modalError.textContent = '';
      modalOverlay.classList.add('active');
      setTimeout(() => { modalInput.focus(); modalInput.select(); }, 150);
    }

    function closeModal() {
      modalOverlay.classList.remove('active');
      modalInput.value = '';
      editIndex = -1;
    }

    function confirmModal() {
      const name = modalInput.value.trim();
      if (!name) {
        modalError.textContent = '名称不能为空';
        modalInput.focus();
        return;
      }

      if (editIndex === -1) {
        // Add
        foods.push(name);
        showToast(`已添加「${name}」`);
      } else {
        // Edit
        const oldName = foods[editIndex];
        foods[editIndex] = name;
        showToast(`已更新「${oldName}」→「${name}」`);
      }

      saveFoods(foods);
      renderList();
      closeModal();
    }

    // ===== Delete =====
    function openDeleteModal(index) {
      deleteIndex = index;
      deleteMsg.textContent = `确定删除「${foods[index]}」？`;
      deleteOverlay.classList.add('active');
    }

    function closeDeleteModal() {
      deleteOverlay.classList.remove('active');
      deleteIndex = -1;
    }

    function confirmDelete() {
      if (deleteIndex === -1) return;
      const name = foods[deleteIndex];
      foods.splice(deleteIndex, 1);
      saveFoods(foods);
      renderList();
      closeDeleteModal();
      showToast(`已删除「${name}」`);

      // Reset lottery display after deletion
      resetLottery();
    }
```

- [ ] **Step 2: Wire up event listeners**

Append after the modal logic:

```js
    // ===== Modal Event Listeners =====
    btnAdd.addEventListener('click', openAddModal);
    btnCancel.addEventListener('click', closeModal);
    btnConfirm.addEventListener('click', confirmModal);

    // Close modal on overlay click
    modalOverlay.addEventListener('click', (e) => {
      if (e.target === modalOverlay) closeModal();
    });

    // Enter key to confirm
    modalInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') confirmModal();
      if (e.key === 'Escape') closeModal();
    });

    // Delete modal listeners
    btnDeleteCancel.addEventListener('click', closeDeleteModal);
    btnDeleteConfirm.addEventListener('click', confirmDelete);
    deleteOverlay.addEventListener('click', (e) => {
      if (e.target === deleteOverlay) closeDeleteModal();
    });

    // List item click delegation for edit/delete icons
    foodList.addEventListener('click', (e) => {
      const editBtn = e.target.closest('[data-edit]');
      const deleteBtn = e.target.closest('[data-delete]');

      if (editBtn) {
        const index = parseInt(editBtn.dataset.edit, 10);
        if (!isNaN(index) && index >= 0 && index < foods.length) {
          openEditModal(index);
        }
      }

      if (deleteBtn) {
        const index = parseInt(deleteBtn.dataset.delete, 10);
        if (!isNaN(index) && index >= 0 && index < foods.length) {
          openDeleteModal(index);
        }
      }
    });
```

- [ ] **Step 3: Verify add/edit/delete flow**

Manual test in browser:
1. Click "+ 添加" → modal opens → type "酸辣粉" → confirm → item appears, count = 6
2. Click ✏️ on "酸辣粉" → modal opens with name pre-filled → change to "酸辣粉(微辣)" → confirm → name updated
3. Click 🗑️ on "酸辣粉(微辣)" → delete confirm modal → confirm → item removed, count = 5

---

### Task 6: JavaScript — Lottery Animation

**Files:**
- Modify: `今天吃什么.html` (append to `<script>` block)

- [ ] **Step 1: Add the lottery animation logic**

Append before the closing `</script>`:

```js
    // ===== Lottery =====
    function resetLottery() {
      lotteryResult.textContent = '';
      lotteryResult.classList.remove('bounce');
      lotteryBox.classList.remove('result');
      btnLottery.textContent = '🎲 开抽！';
      if (foods.length > 0) btnLottery.disabled = false;
    }

    function startLottery() {
      const count = foods.length;
      if (count === 0 || isRolling) return;
      isRolling = true;
      btnLottery.disabled = true;
      lotteryResult.classList.remove('bounce');
      lotteryBox.classList.add('rolling');
      lotteryBox.classList.remove('result');

      // Pick the winner upfront
      const winnerIndex = Math.floor(Math.random() * count);
      const winner = foods[winnerIndex];

      // Build the rolling sequence
      // Phase 1: fast roll ~1.5s at 80ms/step → ~19 steps
      // Phase 2: slow down ~1.0s from 80ms → 300ms
      const fastSteps = 18;
      const slowSteps = 6;
      const sequence = [];

      // Phase 1: random picks, fast
      for (let i = 0; i < fastSteps; i++) {
        sequence.push({ name: foods[Math.floor(Math.random() * count)], delay: 80 });
      }

      // Phase 2: gradually slowing down, end on winner
      for (let i = 0; i < slowSteps; i++) {
        const progress = i / slowSteps;
        const delay = 80 + (300 - 80) * progress; // 80 → 300ms
        const name = i === slowSteps - 1 ? winner : foods[Math.floor(Math.random() * count)];
        sequence.push({ name, delay: Math.round(delay) });
      }

      // Play the sequence
      let step = 0;
      function nextStep() {
        if (step >= sequence.length) {
          // Animation complete — show result
          finishLottery(winner);
          return;
        }
        lotteryResult.textContent = sequence[step].name;
        step++;
        setTimeout(nextStep, sequence[step - 1].delay);
      }
      nextStep();
    }

    function finishLottery(winner) {
      lotteryBox.classList.remove('rolling');
      lotteryBox.classList.add('result');
      lotteryResult.textContent = winner;
      lotteryResult.classList.add('bounce');
      btnLottery.textContent = '🔄 再来一次';
      btnLottery.disabled = false;
      isRolling = false;
      spawnConfetti();
    }
```

- [ ] **Step 2: Add confetti effect**

Append after the lottery logic:

```js
    // ===== Confetti =====
    function spawnConfetti() {
      const emojis = ['🎉', '✨', '🍜', '🍛', '🥟', '🍔', '🍕', '🥡'];
      const boxRect = lotteryBox.getBoundingClientRect();

      for (let i = 0; i < 8; i++) {
        const el = document.createElement('span');
        el.className = 'confetti';
        el.textContent = emojis[Math.floor(Math.random() * emojis.length)];
        el.style.left = (boxRect.left + Math.random() * boxRect.width) + 'px';
        el.style.top  = (boxRect.top + boxRect.height / 2 + Math.random() * 30) + 'px';
        el.style.animationDelay = (Math.random() * 0.3) + 's';
        el.style.fontSize = (20 + Math.random() * 16) + 'px';
        document.body.appendChild(el);
        // Clean up after animation
        setTimeout(() => el.remove(), 1500);
      }
    }
```

- [ ] **Step 3: Wire up the lottery button**

Append after the confetti code:

```js
    btnLottery.addEventListener('click', () => {
      if (foods.length === 0) return;
      if (foods.length === 1) {
        // Single item: skip animation
        lotteryBox.classList.add('result');
        lotteryResult.textContent = foods[0];
        lotteryResult.classList.add('bounce');
        btnLottery.textContent = '🔄 就这一个，别犹豫了！';
        return;
      }
      startLottery();
    });
```

- [ ] **Step 4: Verify animation**

Manual test in browser:
1. Click "🎲 开抽！" → names scroll rapidly → slow down → stop on result
2. Button changes to "🔄 再来一次" after animation
3. Confetti emojis appear and float up
4. Click again → new random result

---

### Task 7: JavaScript — Toast & Polish

**Files:**
- Modify: `今天吃什么.html` (append to `<script>` block)

- [ ] **Step 1: Add toast function**

Append before the closing `</script>`:

```js
    // ===== Toast =====
    let toastTimer = null;

    function showToast(msg) {
      if (toastTimer) clearTimeout(toastTimer);
      toast.textContent = msg;
      toast.classList.add('show');
      toastTimer = setTimeout(() => {
        toast.classList.remove('show');
        toastTimer = null;
      }, 2000);
    }
```

- [ ] **Step 2: Add keyboard shortcut for lottery**

Append after toast code:

```js
    // ===== Keyboard Shortcut =====
    document.addEventListener('keydown', (e) => {
      // Don't trigger shortcuts when typing in the modal
      if (modalOverlay.classList.contains('active')) return;
      if (deleteOverlay.classList.contains('active')) return;

      // Space or Enter to trigger lottery
      if (e.key === ' ' || e.key === 'Enter') {
        e.preventDefault();
        if (!isRolling && foods.length > 0) {
          if (foods.length === 1) {
            lotteryBox.classList.add('result');
            lotteryResult.textContent = foods[0];
            lotteryResult.classList.add('bounce');
            btnLottery.textContent = '🔄 就这一个，别犹豫了！';
          } else {
            startLottery();
          }
        }
      }
    });
```

- [ ] **Step 3: Reset lottery when list becomes empty after deletion**

This is already handled in `confirmDelete()` via the `resetLottery()` call and in `renderList()` which disables the button. Verify: delete all items → lottery area shows empty state → button disabled.

---

### Task 8: Final Verification

**Files:**
- Modify: `今天吃什么.html` (no changes, verify only)

- [ ] **Step 1: End-to-end test checklist**

Run through each scenario in a browser:

| # | Scenario | Expected |
|---|----------|----------|
| 1 | First open | 5 default items, button enabled |
| 2 | Add item | Modal opens → type name → confirm → appears in list |
| 3 | Edit item | Click ✏️ → modal with name → modify → confirm → updated |
| 4 | Delete item | Click 🗑️ → confirm modal → confirm → removed |
| 5 | Delete all items | Empty hint visible, button disabled |
| 6 | Draw with multiple items | Rolling animation → result + confetti |
| 7 | Draw with 1 item | Skips animation, shows "就这一个" |
| 8 | Draw with 0 items | Button disabled, no action |
| 9 | Press Space/Enter | Triggers lottery (when not in modal) |
| 10 | Mobile viewport (<768px) | Single column, full width |
| 11 | PC viewport (≥768px) | Two columns side-by-side |
| 12 | Refresh browser | Data persists (localStorage) |
| 13 | Add with empty name | Error message shown |
| 14 | Click overlay to close modal | Modal closes, no save |

- [ ] **Step 2: Verify code quality**

```bash
wc -l 今天吃什么.html
```

Expected: ~380-430 lines (within the < 400 lines target range, or slightly above with generous formatting).

- [ ] **Step 3: Verify no external dependencies**

```bash
grep -E 'src=|href=.*http|import ' 今天吃什么.html
```

Expected: No output (no external resources loaded).

---

### Task 9: Commit

- [ ] **Step 1: Initialize git and commit**

```bash
cd /Users/wangjunjie/00_Inbox/临时文件/未命名文件夹
git init
git add 今天吃什么.html docs/
git commit -m "feat: 今天吃什么 — 食物抽奖选择器

- Single HTML file, zero dependencies
- localStorage persistence with first-visit seeding
- Add / Edit / Delete food items via modals
- 3-stage rolling lottery animation with confetti
- Responsive layout (mobile single column, PC two columns)
- Keyboard shortcut (Space/Enter to draw)"
```
