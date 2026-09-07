<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DayFlow</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #f4f7fb;
      color: #172033;
    }

    .app {
      max-width: 700px;
      margin: auto;
      padding: 20px 16px 100px;
    }

    h1 {
      margin: 0;
      font-size: 32px;
    }

    h2 {
      margin-top: 0;
    }

    .subtitle {
      color: #667085;
      margin-top: 5px;
    }

    .card {
      background: white;
      border-radius: 20px;
      padding: 18px;
      margin: 14px 0;
      box-shadow: 0 4px 18px rgba(0,0,0,.06);
    }

    .top {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 10px;
    }

    .points {
      background: #eef2ff;
      color: #3730a3;
      padding: 8px 12px;
      border-radius: 20px;
      font-weight: bold;
    }

    .row {
      display: flex;
      gap: 10px;
      align-items: center;
    }

    input,
    textarea {
      width: 100%;
      padding: 12px;
      border: 1px solid #d7dde7;
      border-radius: 12px;
      font-size: 16px;
    }

    textarea {
      min-height: 100px;
      resize: vertical;
    }

    button {
      border: none;
      border-radius: 12px;
      padding: 11px 15px;
      background: #172033;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }

    button.secondary {
      background: #e9eef5;
      color: #172033;
    }

    button.danger {
      background: #b42318;
    }

    button:disabled {
      opacity: .5;
      cursor: not-allowed;
    }

    .task {
      display: flex;
      align-items: center;
      gap: 10px;
      padding: 13px 0;
      border-bottom: 1px solid #edf0f4;
    }

    .task:last-child {
      border-bottom: none;
    }

    .task.done span {
      text-decoration: line-through;
      color: #98a2b3;
    }

    .task input {
      width: 20px;
      height: 20px;
    }

    .delete {
      margin-left: auto;
      background: transparent;
      color: #b42318;
    }

    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
    }

    .big {
      text-align: center;
      font-size: 42px;
      font-weight: bold;
      margin: 12px;
    }

    .center {
      text-align: center;
    }

    .reward {
      border: 1px solid #e5e7eb;
      border-radius: 14px;
      padding: 13px;
      margin-top: 10px;
    }

    .nav {
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      background: white;
      border-top: 1px solid #ddd;
      display: flex;
      justify-content: center;
      gap: 8px;
      padding: 10px;
    }

    .nav button {
      background: transparent;
      color: #667085;
    }

    .nav button.active {
      background: #172033;
      color: white;
    }

    label {
      display: block;
      margin: 10px 0;
    }

    .stat {
      font-size: 28px;
      font-weight: bold;
    }
  </style>
</head>

<body>

<div id="app"></div>

<script>

const STORAGE_KEY = "dayflow_web";

const defaultState = {
  tasks: [],
  water: 0,
  notes: "",
  points: 0,
  claimedRewards: [],

  focusMinutes: 25,
  breakMinutes: 5,

  waterGoal: 8,
  taskGoal: 3,

  appName: "DayFlow",

  developerMode: false
};

let state =
  JSON.parse(localStorage.getItem(STORAGE_KEY)) ||
  defaultState;

let currentTab = "today";

let timer = null;
let timerRunning = false;
let timerMode = "focus";
let secondsLeft = state.focusMinutes * 60;


/* -----------------------------
   SPEICHERN
----------------------------- */

function save() {
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify(state)
  );
}


/* -----------------------------
   APP RENDERN
----------------------------- */

function render() {

  document.getElementById("app").innerHTML = `

    <div class="app">

      <div class="top">

        <div>
          <h1>${state.appName}</h1>

          <div class="subtitle">
            ${new Date().toLocaleDateString(
              "de-DE",
              {
                weekday: "long",
                day: "numeric",
                month: "long"
              }
            )}
          </div>
        </div>

        <div class="points">
          ⭐ ${state.points}
        </div>

      </div>

      <div id="content"></div>

    </div>

    <div class="nav">

      <button
        class="${currentTab === "today" ? "active" : ""}"
        onclick="showTab('today')">
        Heute
      </button>

      <button
        class="${currentTab === "focus" ? "active" : ""}"
        onclick="showTab('focus')">
        Fokus
      </button>

      <button
        class="${currentTab === "stats" ? "active" : ""}"
        onclick="showTab('stats')">
        Stats
      </button>

      <button
        class="${currentTab === "settings" ? "active" : ""}"
        onclick="showTab('settings')">
        Einstellungen
      </button>

    </div>
  `;

  if (currentTab === "today") renderToday();
  if (currentTab === "focus") renderFocus();
  if (currentTab === "stats") renderStats();
  if (currentTab === "settings") renderSettings();
}


/* -----------------------------
   TABS
----------------------------- */

function showTab(tab) {

  currentTab = tab;
  render();

}


/* -----------------------------
   HEUTE
----------------------------- */

function renderToday() {

  const content =
    document.getElementById("content");

  content.innerHTML = `

    <div class="card">

      <div class="top">
        <h2>Aufgaben</h2>

        <span>
          ${
            state.tasks.filter(t => t.done).length
          }
          /
          ${state.taskGoal}
        </span>
      </div>

      <form
        onsubmit="
          event.preventDefault();
          addTask(this.task.value);
          this.reset();
        "
      >

        <div class="row">

          <input
            name="task"
            placeholder="Neue Aufgabe..."
          >

          <button>+</button>

        </div>

      </form>

      ${
        state.tasks.length === 0
          ? "<p>Keine Aufgaben vorhanden.</p>"
          : state.tasks.map(
              (task, index) => `

              <div
                class="task ${
                  task.done ? "done" : ""
                }"
              >

                <input
                  type="checkbox"
                  ${
                    task.done
                      ? "checked"
                      : ""
                  }
                  onchange="
                    toggleTask(${index})
                  "
                >

                <span>
                  ${escapeHTML(task.text)}
                </span>

                <button
                  class="delete"
                  onclick="
                    deleteTask(${index})
                  "
                >
                  Löschen
                </button>

              </div>

            `
            ).join("")
      }

    </div>


    <div class="grid">

      <div class="card">

        <h3>💧 Wasser</h3>

        <div class="big">
          ${state.water}
        </div>

        <div class="center">

          <button onclick="addWater()">
            +1 Glas
          </button>

        </div>

        <p>
          Ziel: ${state.waterGoal} Gläser
        </p>

      </div>


      <div class="card">

        <h3>🏆 Punkte</h3>

        <div class="big">
          ${state.points}
        </div>

        <div class="center">

          <button
            class="secondary"
            onclick="showTab('stats')"
          >
            Belohnungen
          </button>

        </div>

      </div>

    </div>


    <div class="card">

      <h2>📝 Tagesnotiz</h2>

      <textarea
        id="notes"
        placeholder="Was möchtest du heute festhalten?"
      >${escapeHTML(state.notes)}</textarea>

      <br><br>

      <button onclick="saveNotes()">
        Speichern
      </button>

    </div>

  `;
}


/* -----------------------------
   AUFGABEN
----------------------------- */

function addTask(text) {

  text = text.trim();

  if (!text) return;

  state.tasks.push({
    text: text,
    done: false
  });

  save();
  render();

}


function toggleTask(index) {

  const task = state.tasks[index];

  if (!task.done) {
    state.points += 5;
  }

  task.done = !task.done;

  save();
  render();

}


function deleteTask(index) {

  state.tasks.splice(index, 1);

  save();
  render();

}


/* -----------------------------
   WASSER
----------------------------- */

function addWater() {

  state.water++;

  state.points += 2;

  save();
  render();

}


/* -----------------------------
   NOTIZ
----------------------------- */

function saveNotes() {

  state.notes =
    document.getElementById("notes").value;

  save();

  alert("Notiz gespeichert!");

}


/* -----------------------------
   FOKUS TIMER
----------------------------- */

function renderFocus() {

  const content =
    document.getElementById("content");

  const minutes =
    Math.floor(secondsLeft / 60);

  const seconds =
    secondsLeft % 60;

  content.innerHTML = `

    <div class="card center">

      <h2>
        ${
          timerMode === "focus"
            ? "🎯 Fokus"
            : "☕ Pause"
        }
      </h2>

      <div class="big">

        ${String(minutes).padStart(2,"0")}
        :
        ${String(seconds).padStart(2,"0")}

      </div>

      <button onclick="startTimer()">
        ${
          timerRunning
            ? "Läuft..."
            : "Start"
        }
      </button>

      <button
        class="secondary"
        onclick="resetTimer()"
      >
        Zurücksetzen
      </button>

      <p>
        Fokus: ${state.focusMinutes} Min.
        <br>
        Pause: ${state.breakMinutes} Min.
      </p>

    </div>

  `;

}


function startTimer() {

  if (timerRunning) return;

  timerRunning = true;

  timer = setInterval(() => {

    secondsLeft--;

    if (secondsLeft <= 0) {

      clearInterval(timer);

      timerRunning = false;

      if (timerMode === "focus") {

        state.points += 10;

        timerMode = "break";

        secondsLeft =
          state.breakMinutes * 60;

        alert(
          "Fokus geschafft! +10 Punkte 🎉"
        );

      } else {

        timerMode = "focus";

        secondsLeft =
          state.focusMinutes * 60;

        alert(
          "Pause vorbei!"
        );

      }

      save();

    }

    render();

  }, 1000);

}


function resetTimer() {

  clearInterval(timer);

  timerRunning = false;

  timerMode = "focus";

  secondsLeft =
    state.focusMinutes * 60;

  render();

}


/* -----------------------------
   STATS / BELOHNUNGEN
----------------------------- */

function renderStats() {

  const content =
    document.getElementById("content");

  const rewards = [
    {
      name: "Starter",
      points: 20
    },
    {
      name: "Bronze",
      points: 50
    },
    {
      name: "Silber",
      points: 100
    },
    {
      name: "Gold",
      points: 250
    }
  ];

  content.innerHTML = `

    <div class="card">

      <h2>🏆 Belohnungen</h2>

      <p>
        Sammle Punkte durch Aufgaben,
        Wasser und Fokus-Sessions.
      </p>

      ${
        rewards.map(
          reward => `

          <div class="reward">

            <div class="top">

              <strong>
                ${reward.name}
              </strong>

              <span>
                ${reward.points} Punkte
              </span>

            </div>

            <br>

            <button
              ${
                state.points < reward.points ||
                state.claimedRewards.includes(
                  reward.points
                )
                  ? "disabled"
                  : ""
              }

              onclick="
                claimReward(${reward.points})
              "
            >

              ${
                state.claimedRewards.includes(
                  reward.points
                )
                  ? "Eingelöst"
                  : "Einlösen"
              }

            </button>

          </div>

        `
        ).join("")
      }

    </div>


    <div class="grid">

      <div class="card">

        <div class="stat">
          ${
            state.tasks.filter(
              task => task.done
            ).length
          }
        </div>

        Erledigte Aufgaben

      </div>


      <div class="card">

        <div class="stat">
          ${state.water}
        </div>

        Gläser Wasser

      </div>

    </div>

  `;

}


function claimReward(points) {

  if (
    state.points >= points &&
    !state.claimedRewards.includes(points)
  ) {

    state.points -= points;

    state.claimedRewards.push(points);

    save();

    render();

    alert(
      "Belohnung eingelöst! 🎉"
    );

  }

}


/* -----------------------------
   EINSTELLUNGEN
----------------------------- */

function renderSettings() {

  const content =
    document.getElementById("content");

  content.innerHTML = `

    <div class="card">

      <h2>⚙️ Einstellungen</h2>

      <label>
        App-Name

        <input
          id="appName"
          value="${escapeHTML(state.appName)}"
        >
      </label>

      <label>
        Tagesziel Aufgaben

        <input
          id="taskGoal"
          type="number"
          min="1"
          value="${state.taskGoal}"
        >
      </label>

      <label>
        Wasserziel

        <input
          id="waterGoal"
          type="number"
          min="1"
          value="${state.waterGoal}"
        >
      </label>

      <label>
        Fokus-Minuten

        <input
          id="focusMinutes"
          type="number"
          min="1"
          value="${state.focusMinutes}"
        >
      </label>

      <label>
        Pausen-Minuten

        <input
          id="breakMinutes"
          type="number"
          min="1"
          value="${state.breakMinutes}"
        >
      </label>

      <button onclick="saveSettings()">
        Speichern
      </button>

    </div>


    <div class="card">

      <h2>🔧 Entwickler-Modus</h2>

      ${
        state.developerMode

          ? `

            <p>
              Entwickler-Modus ist aktiv.
            </p>

            <button
              onclick="
                state.points += 100;
                save();
                render();
              "
            >
              +100 Testpunkte
            </button>

            <button
              class="secondary"
              onclick="
                state.developerMode=false;
                save();
                render();
              "
            >
              Sperren
            </button>

          `

          : `

            <button onclick="unlockDeveloper()">
              Entsperren
            </button>

          `
      }

    </div>


    <div class="card">

      <button
        class="danger"
        onclick="deleteAllData()"
      >
        Alle Daten löschen
      </button>

    </div>

  `;

}


function saveSettings() {

  state.appName =
    document.getElementById("appName").value ||
    "DayFlow";

  state.taskGoal =
    Number(
      document.getElementById("taskGoal").value
    ) || 3;

  state.waterGoal =
    Number(
      document.getElementById("waterGoal").value
    ) || 8;

  state.focusMinutes =
    Number(
      document.getElementById("focusMinutes").value
    ) || 25;

  state.breakMinutes =
    Number(
      document.getElementById("breakMinutes").value
    ) || 5;

  secondsLeft =
    state.focusMinutes * 60;

  save();
  render();

}


function unlockDeveloper() {

  const password =
    prompt("Entwickler-Passwort:");

  if (password === "Lukas") {

    state.developerMode = true;

    save();
    render();

  } else if (password !== null) {

    alert("Falsches Passwort.");

  }

}


function deleteAllData() {

  if (
    confirm(
      "Wirklich alle DayFlow-Daten löschen?"
    )
  ) {

    localStorage.removeItem(
      STORAGE_KEY
    );

    location.reload();

  }

}


/* -----------------------------
   SICHERHEIT / HTML
----------------------------- */

function escapeHTML(text) {

  return String(text)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");

}


/* -----------------------------
   START
----------------------------- */

render();

</script>

</body>
</html>
