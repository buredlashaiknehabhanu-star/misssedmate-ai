<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MissedMate AI</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0b1020;
  color: #f5f3ff;
}

main {
  max-width: 1000px;
  margin: auto;
  padding: 25px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
}

.brand {
  font-size: 25px;
  font-weight: bold;
  color: #c4b5fd;
}

.badge {
  color: #86efac;
  border: 1px solid #315746;
  border-radius: 20px;
  padding: 8px 12px;
  font-size: 12px;
}

.hero { padding: 40px 0 20px; }

small {
  color: #67e8f9;
  letter-spacing: 2px;
  font-weight: bold;
}

h1 { font-size: clamp(35px, 7vw, 58px); }

h1 span { color: #a78bfa; }

p { color: #aab5d0; line-height: 1.6; }

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin: 25px 0;
}

.card, .panel, .result {
  background: #151d32;
  border: 1px solid #293450;
  border-radius: 16px;
  padding: 20px;
}

.card strong {
  display: block;
  font-size: 28px;
  margin: 10px 0;
}

.layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

textarea {
  width: 100%;
  height: 190px;
  background: #0b1020;
  color: white;
  border: 1px solid #35415e;
  border-radius: 10px;
  padding: 14px;
  font-size: 15px;
  resize: vertical;
}

button {
  padding: 12px 15px;
  border: none;
  border-radius: 9px;
  cursor: pointer;
  font-weight: bold;
  margin: 8px 5px 0 0;
}

.primary {
  background: #8b5cf6;
  color: white;
}

.secondary {
  background: #28334d;
  color: white;
}

button:hover { opacity: 0.85; }

.output {
  white-space: pre-wrap;
  line-height: 1.8;
  overflow-wrap: anywhere;
}

.result { margin-top: 20px; }

.task {
  display: flex;
  gap: 10px;
  padding: 12px 0;
  border-bottom: 1px solid #293450;
}

.done {
  text-decoration: line-through;
  color: #8994ad;
}

footer {
  text-align: center;
  margin-top: 30px;
  color: #aab5d0;
  font-size: 12px;
}

@media(max-width:650px) {
  main { padding: 15px; }
  .layout { grid-template-columns: 1fr; }
  .grid { gap: 7px; }
  .card { padding: 12px; }
  .card strong { font-size: 22px; }
}
</style>
</head>

<body>
<main>

<header>
  <div class="brand">✦ MissedMate AI</div>
  <div class="badge">● SMART MESSAGE ASSISTANT</div>
</header>

<section class="hero">
  <small>YOUR PERSONAL MESSAGE ASSISTANT</small>
  <h1>Missed something?<br>
    <span>We've got you.</span>
  </h1>
  <p>Turn long college messages into useful summaries,
  find important deadlines, and create your personal task list.</p>
</section>

<section class="grid">
  <div class="card">📨 Messages
    <strong id="count">0</strong>
    <p>Analyzed this session</p>
  </div>
  <div class="card">✅ Tasks
    <strong id="total">0</strong>
    <p>Tasks discovered</p>
  </div>
  <div class="card">⏳ Pending
    <strong id="pending">0</strong>
    <p>Tasks to complete</p>
  </div>
</section>

<div class="layout">
  <section class="panel">
    <h2>✦ Message Analyzer</h2>
    <p>Paste a college announcement, email, or group message.</p>

    <textarea id="message"
      placeholder="Paste your college message here..."></textarea>

    <button class="primary" onclick="analyze()">
      ✦ Analyze Message
    </button>

    <button class="secondary" onclick="sample()">
      Try Sample
    </button>

    <button class="secondary" onclick="clearAll()">
      Clear
    </button>
  </section>

  <section class="panel">
    <h2>🪄 Quick Summary</h2>
    <div class="output" id="summary">
      Your summary will appear here.
    </div>

    <h2 style="margin-top:24px">📅 Important Dates</h2>
    <div class="output" id="dates">No dates found yet.</div>

    <button class="secondary" onclick="downloadSummary()">
      ⬇ Download Summary
    </button>
  </section>
</div>

<section class="result">
  <h2>✅ Your Action List</h2>
  <div id="tasks"><p>No tasks yet.</p></div>
  <button class="secondary" onclick="clearCompleted()">
    Clear Completed Tasks
  </button>
</section>

<footer>
  MissedMate AI · Never miss what matters ✦
</footer>
</main>

<script>
let messageCount = 0;
let tasks = [];
let lastSummary = "";
let lastDates = "";

const keywords = [
  "deadline", "submit", "submission", "upload",
  "bring", "prepare", "complete", "finish",
  "attend", "meeting", "tomorrow", "today",
  "urgent", "remember", "send", "register",
  "assignment", "report", "exam", "test",
  "project", "required", "must", "before", "by"
];

function updateStats() {
  document.getElementById("count").textContent = messageCount;
  document.getElementById("total").textContent = tasks.length;
  document.getElementById("pending").textContent =
    tasks.filter(t => !t.done).length;
}

function analyze() {
  const raw = document.getElementById("message").value.trim();

  if (!raw) {
    alert("Please paste a message first!");
    return;
  }

  messageCount++;

  const sentences = raw.split(/(?<=[.!?])\s+|\n+/)
    .filter(s => s.trim());

  const important = sentences.filter(sentence =>
    keywords.some(word =>
      new RegExp("\\b" + word + "\\b", "i").test(sentence)
    )
  );

  const chosen = (important.length ? important : sentences)
    .slice(0, 5);

  lastSummary = chosen.map(s => "• " + s.trim()).join("\n\n");

  document.getElementById("summary").textContent = lastSummary;

  const datePattern =
    /\b(?:\d{1,2}(?:st|nd|rd|th)?\s+(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:t(?:ember)?)?|Oct(?:ober)?|Nov(?:ember)?|Dec(?:ember)?)|(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:t(?:ember)?)?|Oct(?:ober)?|Nov(?:ember)?|Dec(?:ember)?)\s+\d{1,2}(?:st|nd|rd|th)?|\d{1,2}[/-]\d{1,2}(?:[/-]\d{2,4})?|\b(?:today|tomorrow|Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday)\b|\b\d{1,2}(?::\d{2})?\s?(?:AM|PM)\b)/gi;

  const found = [...new Set(raw.match(datePattern) || [])];

  lastDates = found.length
    ? found.map(d => "📌 " + d).join("\n")
    : "No clear dates or times detected.";

  document.getElementById("dates").textContent = lastDates;

  important.forEach(sentence => {
    const exists = tasks.some(t =>
      t.text.toLowerCase() === sentence.trim().toLowerCase()
    );

    if (!exists) {
      tasks.push({ text: sentence.trim(), done: false });
    }
  });

  renderTasks();
  updateStats();
}

function renderTasks() {
  const box = document.getElementById("tasks");
  box.innerHTML = "";

  if (!tasks.length) {
    box.textContent = "No tasks yet. Analyze a message first.";
    updateStats();
    return;
  }

  tasks.forEach((task, index) => {
    const row = document.createElement("label");
    row.className = "task";

    const checkbox = document.createElement("input");
    checkbox.type = "checkbox";
    checkbox.checked = task.done;

    checkbox.addEventListener("change", () => {
      tasks[index].done = checkbox.checked;
      renderTasks();
    });

    const span = document.createElement("span");
    span.textContent = task.text;

    if (task.done) span.className = "done";

    row.append(checkbox, span);
    box.appendChild(row);
  });

  updateStats();
}

function sample() {
  document.getElementById("message").value =
    "Reminder! Our project report is due on 15 October. " +
    "Upload your PDF before 5 PM. " +
    "Team meeting tomorrow at 10 AM. " +
    "Bring your laptop and finish your slides. " +
    "Register for the workshop by Friday.";

  analyze();
}

function clearAll() {
  document.getElementById("message").value = "";
  document.getElementById("summary").textContent =
    "Your summary will appear here.";
  document.getElementById("dates").textContent =
    "No dates found yet.";

  lastSummary = "";
  lastDates = "";
}

function clearCompleted() {
  tasks = tasks.filter(t => !t.done);
  renderTasks();
}

function downloadSummary() {
  const content =
    "MissedMate AI - Summary\n\n" +
    (lastSummary || "No summary yet.") +
    "\n\nImportant Dates\n" +
    (lastDates || "None detected.");

  const blob = new Blob([content], { type: "text/plain" });
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");

  link.href = url;
  link.download = "missedmate-summary.txt";
  link.click();

  URL.revokeObjectURL(url);
}
</script>
</body>
</html>
