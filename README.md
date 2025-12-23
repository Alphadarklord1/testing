<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Simple Calendar</title>
  <style>
    body {
      font-family: Arial, Helvetica, sans-serif;
      margin: 0;
      background: #f4f6f8;
      color: #111827;
    }

    .wrap {
      max-width: 950px;
      margin: 30px auto;
      padding: 0 16px 70px;
    }

    .card {
      background: white;
      border-radius: 14px;
      box-shadow: 0 8px 25px rgba(0,0,0,0.08);
      padding: 16px;
    }

    .topbar {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      margin-bottom: 12px;
      flex-wrap: wrap;
    }

    .title {
      font-size: 20px;
      font-weight: 800;
      letter-spacing: 0.2px;
    }

    .controls {
      display: flex;
      gap: 8px;
      align-items: center;
    }

    button {
      border: 0;
      padding: 10px 12px;
      border-radius: 10px;
      background: #2563eb;
      color: white;
      cursor: pointer;
      font-weight: 700;
    }

    button:hover { filter: brightness(0.95); }
    button.secondary { background: #111827; }
    button.ghost {
      background: #e5e7eb;
      color: #111827;
      font-weight: 700;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 10px;
    }

    .dow {
      text-align: center;
      font-weight: 800;
      color: #6b7280;
      padding: 10px 6px;
      border-radius: 10px;
      background: #f9fafb;
    }

    .day {
      min-height: 90px;
      background: #ffffff;
      border: 1px solid #e5e7eb;
      border-radius: 12px;
      padding: 10px;
      cursor: pointer;
      display: flex;
      flex-direction: column;
      gap: 6px;
      transition: transform 0.05s ease;
    }

    .day:hover { transform: translateY(-1px); }

    .day.muted {
      background: #fafafa;
      color: #9ca3af;
    }

    .num {
      font-weight: 900;
      font-size: 14px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 8px;
    }

    .badge {
      font-size: 11px;
      padding: 3px 8px;
      border-radius: 999px;
      background: #e0f2fe;
      color: #0369a1;
      font-weight: 800;
    }

    .event {
      font-size: 12px;
      background: #f3f4f6;
      border-radius: 10px;
      padding: 6px 8px;
      line-height: 1.25;
      overflow: hidden;
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
    }

    .footerHint {
      margin-top: 12px;
      color: #6b7280;
      font-size: 13px;
    }

    /* Simple modal */
    .modalBack {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.35);
      display: none;
      align-items: center;
      justify-content: center;
      padding: 16px;
    }

    .modal {
      width: 100%;
      max-width: 520px;
      background: white;
      border-radius: 14px;
      box-shadow: 0 18px 60px rgba(0,0,0,0.25);
      padding: 16px;
    }

    .modal h2 {
      margin: 0 0 6px;
      font-size: 18px;
    }

    .modal p {
      margin: 0 0 12px;
      color: #6b7280;
      font-size: 13px;
    }

    textarea {
      width: 100%;
      min-height: 90px;
      resize: vertical;
      border: 1px solid #e5e7eb;
      border-radius: 12px;
      padding: 10px;
      font-size: 14px;
      outline: none;
    }

    .modalActions {
      display: flex;
      justify-content: space-between;
      gap: 10px;
      margin-top: 12px;
      flex-wrap: wrap;
    }

    .leftActions, .rightActions {
      display: flex;
      gap: 8px;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <div class="topbar">
        <div class="title" id="monthTitle">Month YYYY</div>
        <div class="controls">
          <button class="ghost" id="prevBtn">◀ Prev</button>
          <button class="ghost" id="todayBtn">Today</button>
          <button class="ghost" id="nextBtn">Next ▶</button>
          <button class="secondary" id="clearAllBtn">Clear All</button>
        </div>
      </div>

      <div class="grid" id="dowRow"></div>
      <div style="height:10px"></div>
      <div class="grid" id="calendarGrid"></div>

      <div class="footerHint">
        Click any day to add/edit an event. (Saved locally in your browser.)
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modalBack" id="modalBack">
    <div class="modal">
      <h2 id="modalTitle">Event</h2>
      <p id="modalSubtitle">YYYY-MM-DD</p>
      <textarea id="eventText" placeholder="Type your event note here..."></textarea>
      <div class="modalActions">
        <div class="leftActions">
          <button class="ghost" id="closeBtn">Close</button>
        </div>
        <div class="rightActions">
          <button class="ghost" id="deleteBtn">Delete</button>
          <button id="saveBtn">Save</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    // ---- Simple Calendar with localStorage events ----

    const dowNames = ["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];
    const monthNames = [
      "January","February","March","April","May","June",
      "July","August","September","October","November","December"
    ];

    const monthTitle = document.getElementById("monthTitle");
    const dowRow = document.getElementById("dowRow");
    const grid = document.getElementById("calendarGrid");

    const prevBtn = document.getElementById("prevBtn");
    const nextBtn = document.getElementById("nextBtn");
    const todayBtn = document.getElementById("todayBtn");
    const clearAllBtn = document.getElementById("clearAllBtn");

    const modalBack = document.getElementById("modalBack");
    const modalTitle = document.getElementById("modalTitle");
    const modalSubtitle = document.getElementById("modalSubtitle");
    const eventText = document.getElementById("eventText");
    const closeBtn = document.getElementById("closeBtn");
    const saveBtn = document.getElementById("saveBtn");
    const deleteBtn = document.getElementById("deleteBtn");

    // current view
    let viewDate = new Date();
    viewDate.setDate(1);

    // modal selected date key
    let activeKey = null;

    function pad2(n){ return String(n).padStart(2, "0"); }
    function keyFromDate(d){
      return `${d.getFullYear()}-${pad2(d.getMonth()+1)}-${pad2(d.getDate())}`;
    }

    function loadEvents(){
      try {
        return JSON.parse(localStorage.getItem("simple_calendar_events") || "{}");
      } catch {
        return {};
      }
    }
    function saveEvents(obj){
      localStorage.setItem("simple_calendar_events", JSON.stringify(obj));
    }

    function openModal(dateObj){
      const k = keyFromDate(dateObj);
      activeKey = k;

      const events = loadEvents();
      const text = events[k] || "";

      modalTitle.textContent = "Event / Note";
      modalSubtitle.textContent = k;
      eventText.value = text;

      modalBack.style.display = "flex";
      eventText.focus();
    }

    function closeModal(){
      modalBack.style.display = "none";
      activeKey = null;
      eventText.value = "";
    }

    function buildDOW(){
      dowRow.innerHTML = "";
      for (const name of dowNames){
        const el = document.createElement("div");
        el.className = "dow";
        el.textContent = name;
        dowRow.appendChild(el);
      }
    }

    function render(){
      const year = viewDate.getFullYear();
      const month = viewDate.getMonth();

      monthTitle.textContent = `${monthNames[month]} ${year}`;
      grid.innerHTML = "";

      const events = loadEvents();

      const firstDayIndex = new Date(year, month, 1).getDay();        // 0..6
      const daysInMonth = new Date(year, month + 1, 0).getDate();      // 28..31
      const daysInPrevMonth = new Date(year, month, 0).getDate();      // last day of prev month

      // We’ll render 6 weeks (42 cells) to keep layout stable.
      const totalCells = 42;

      for (let i = 0; i < totalCells; i++){
        const cell = document.createElement("div");
        cell.className = "day";

        // Determine which date this cell is
        let cellDate;

        if (i < firstDayIndex){
          // previous month trailing days
          const dayNum = daysInPrevMonth - (firstDayIndex - 1 - i);
          cell.classList.add("muted");
          cellDate = new Date(year, month - 1, dayNum);
        } else if (i >= firstDayIndex + daysInMonth){
          // next month leading days
          const dayNum = i - (firstDayIndex + daysInMonth) + 1;
          cell.classList.add("muted");
          cellDate = new Date(year, month + 1, dayNum);
        } else {
          // current month
          const dayNum = i - firstDayIndex + 1;
          cellDate = new Date(year, month, dayNum);
        }

        const k = keyFromDate(cellDate);
        const hasEvent = (events[k] && events[k].trim().length > 0);

        const top = document.createElement("div");
        top.className = "num";

        const left = document.createElement("span");
        left.textContent = cellDate.getDate();

        top.appendChild(left);

        if (hasEvent){
          const badge = document.createElement("span");
          badge.className = "badge";
          badge.textContent = "Event";
          top.appendChild(badge);
        }

        cell.appendChild(top);

        if (hasEvent){
          const preview = document.createElement("div");
          preview.className = "event";
          preview.textContent = events[k];
          cell.appendChild(preview);
        }

        cell.addEventListener("click", () => openModal(cellDate));
        grid.appendChild(cell);
      }
    }

    // Buttons
    prevBtn.addEventListener("click", () => {
      viewDate = new Date(viewDate.getFullYear(), viewDate.getMonth() - 1, 1);
      render();
    });

    nextBtn.addEventListener("click", () => {
      viewDate = new Date(viewDate.getFullYear(), viewDate.getMonth() + 1, 1);
      render();
    });

    todayBtn.addEventListener("click", () => {
      const now = new Date();
      viewDate = new Date(now.getFullYear(), now.getMonth(), 1);
      render();
    });

    clearAllBtn.addEventListener("click", () => {
      const ok = confirm("Clear ALL saved events? (This cannot be undone)");
      if (!ok) return;
      localStorage.removeItem("simple_calendar_events");
      render();
    });

    // Modal actions
    closeBtn.addEventListener("click", closeModal);
    modalBack.addEventListener("click", (e) => {
      if (e.target === modalBack) closeModal();
    });

    saveBtn.addEventListener("click", () => {
      if (!activeKey) return;
      const events = loadEvents();
      const text = eventText.value.trim();

      if (text.length === 0) {
        delete events[activeKey];
      } else {
        events[activeKey] = text;
      }
      saveEvents(events);
      closeModal();
      render();
    });

    deleteBtn.addEventListener("click", () => {
      if (!activeKey) return;
      const events = loadEvents();
      delete events[activeKey];
      saveEvents(events);
      closeModal();
      render();
    });

    // Init
    buildDOW();
    render();
  </script>
</body>
</html>
