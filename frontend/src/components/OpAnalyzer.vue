<template>
    <div class="page">
      <h1>Сервис для сравнения образовательных программ НГУ</h1>
  
      <div class="top">
        <p>
          Страница с полями загрузки для двух образовательных программ (ОП1 — слева, ОП2 — справа).
          После загрузки файлов нажмите кнопку «Построить диаграммы».
        </p>
      </div>
  
      <div class="layout">
        <!-- ================= LEFT COLUMN (ОП1) ================= -->
        <div class="column">
          <div class="upload-card">
            <form @submit.prevent="submitLeft">
              <label>Поле для загрузки учебного плана ОП1</label>
              <input type="file" accept=".xlsx,.xls,.pdf,.csv" required @change="onUpFile1Change">
  
              <label>Поле для загрузки РПД ОП1 (несколько файлов или папка/архив)</label>
              <input type="file" multiple webkitdirectory directory @change="onRpdFiles1Change">
  
              <button class="btn" style="margin-top:8px">Построить диаграммы для ОП1</button>
            </form>
          </div>
  
          <!-- DOWNLOADS LEFT -->
          <div v-if="left.chartsLoaded" class="downloads">
            <h4>Скачать таблицы ОП1</h4>
            <a class="download-btn" :href="left.table1Url" download>Таблица 1 (Excel)</a>
            <a class="download-btn" :href="left.table2Url" download>Таблица 2 (Excel)</a>
          </div>
  
          <!-- CHARTS LEFT -->
          <div v-if="left.chartsLoaded" class="charts-grid">
            <h3>Диаграммы — ОП1</h3>
  
            <div class="charts-table">
              <template v-for="i in 9" :key="'L'+i">
                <div class="chart-cell">
                  <img class="chart-img" :src="left.charts[i - 1]" :alt="'Диаграмма ' + i + ' ОП1'">
                  <div class="chart-caption">Диаграмма {{ i }} — ОП1</div>
                </div>
              </template>
            </div>
          </div>
        </div>
  
        <!-- ================= RIGHT COLUMN (ОП2) ================= -->
        <div class="column">
          <div class="upload-card">
            <form @submit.prevent="submitRight">
              <label>Поле для загрузки учебного плана ОП2</label>
              <input type="file" accept=".xlsx,.xls,.pdf,.csv" required @change="onUpFile2Change">
  
              <label>Поле для загрузки РПД ОП2 (несколько файлов или папка/архив)</label>
              <input type="file" multiple webkitdirectory directory @change="onRpdFiles2Change">
  
              <button class="btn" style="margin-top:8px">Построить диаграммы для ОП2</button>
            </form>
          </div>
  
          <!-- DOWNLOADS RIGHT -->
          <div v-if="right.chartsLoaded" class="downloads">
            <h4>Скачать таблицы ОП2</h4>
            <a class="download-btn" :href="right.table1Url" download>Таблица 1 (Excel)</a>
            <a class="download-btn" :href="right.table2Url" download>Таблица 2 (Excel)</a>
          </div>
  
          <!-- CHARTS RIGHT -->
          <div v-if="right.chartsLoaded" class="charts-grid">
            <h3>Диаграммы — ОП2</h3>
  
            <div class="charts-table">
              <template v-for="i in 9" :key="'R'+i">
                <div class="chart-cell">
                  <img class="chart-img" :src="right.charts[i - 1]" :alt="'Диаграмма ' + i + ' ОП2'">
                  <div class="chart-caption">Диаграмма {{ i }} — ОП2</div>
                </div>
              </template>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script lang="ts" setup>
  import { ref } from "vue"
  
  // -------------- STATE -----------------
  
  const left = ref({
    upFile: null as File | null,
    rpdFiles: [] as File[],
    charts: [] as string[],
    chartsLoaded: false,
    table1Url: "",
    table2Url: ""
  })
  
  const right = ref({
    upFile: null as File | null,
    rpdFiles: [] as File[],
    charts: [] as string[],
    chartsLoaded: false,
    table1Url: "",
    table2Url: ""
  })
  
  
  // -------------- INPUT HANDLERS -----------------
  
  const onUpFile1Change = (e: Event) => {
    const files = (e.target as HTMLInputElement).files
    if (files && files.length) left.value.upFile = files[0]
  }
  
  const onRpdFiles1Change = (e: Event) => {
    const files = (e.target as HTMLInputElement).files
    left.value.rpdFiles = files ? Array.from(files) : []
  }
  
  const onUpFile2Change = (e: Event) => {
    const files = (e.target as HTMLInputElement).files
    if (files && files.length) right.value.upFile = files[0]
  }
  
  const onRpdFiles2Change = (e: Event) => {
    const files = (e.target as HTMLInputElement).files
    right.value.rpdFiles = files ? Array.from(files) : []
  }
  
  
  // -------------- REQUESTS -----------------
  
  async function sendToServer(endpoint: string, upFile: File | null, rpdFiles: File[]) {
    const fd = new FormData()
    if (upFile) fd.append("up", upFile)
    for (const f of rpdFiles) fd.append("rpd", f)
  
    const resp = await fetch(endpoint, {
      method: "POST",
      body: fd
    })
  
    if (!resp.ok) throw new Error("Ошибка HTTP " + resp.status)
  
    return await resp.json()
  }
  
  
  // -------------- SUBMIT HANDLERS -----------------
  
  async function submitLeft() {
    try {
      const data = await sendToServer("/build_left", left.value.upFile, left.value.rpdFiles)
  
      left.value.charts = data.charts
      left.value.table1Url = data.table1
      left.value.table2Url = data.table2
      left.value.chartsLoaded = true
    } catch (err: any) {
      alert("Ошибка: " + err.message)
    }
  }
  
  async function submitRight() {
    try {
      const data = await sendToServer("/build_right", right.value.upFile, right.value.rpdFiles)
  
      right.value.charts = data.charts
      right.value.table1Url = data.table1
      right.value.table2Url = data.table2
      right.value.chartsLoaded = true
    } catch (err: any) {
      alert("Ошибка: " + err.message)
    }
  }
  </script>
  
  <style scoped>
  /* ===== ВСЕ СТИЛИ ИЗ cur2.html ВОССТАНОВЛЕНЫ ===== */
  
  :root {
    --gap: 16px;
    --border: #222;
    --muted: #777;
    --card-padding: 12px;
    --chart-size: 220px;
  }
  
  .page {
    font-family: Arial, Helvetica, sans-serif;
    margin: 20px;
    color: #111;
  }
  
  h1 {
    font-size: 20px;
    margin-bottom: 8px;
  }
  
  .top {
    display: flex;
    gap: 16px;
    margin-bottom: 18px;
  }
  
  .layout {
    display: flex;
    gap: var(--gap);
    align-items: flex-start;
  }
  
  .column {
    flex: 1;
    min-width: 280px;
  }
  
  .upload-card {
    border: 2px solid var(--border);
    padding: var(--card-padding);
    margin-bottom: 12px;
    border-radius: 6px;
  }
  
  label {
    display: block;
    margin-bottom: 6px;
    font-weight: 600;
  }
  
  input[type="file"] {
    display: block;
    margin-bottom: 10px;
  }
  
  .btn {
    padding: 8px 12px;
    border-radius: 6px;
    background: #2d89ef;
    color: white;
    border: 0;
    cursor: pointer;
    font-weight: 600;
  }
  
  .charts-grid {
    margin-top: 8px;
    border-top: 1px solid #eee;
    padding-top: 12px;
  }
  
  .charts-table {
    display: grid;
    grid-template-columns: 1fr;
    gap: 12px;
  }
  
  .chart-cell {
    display: flex;
    align-items: center;
    gap: 12px;
    min-height: 120px;
  }
  
  .chart-img {
    width: var(--chart-size);
    border: 1px solid #ddd;
    background: #fafafa;
  }
  
  .chart-caption {
    font-size: 13px;
    color: var(--muted);
  }
  
  .downloads {
    margin-top: 25px;
    padding: 15px;
    background: #eef3ff;
    border-radius: 8px;
  }
  
  .download-btn {
    margin-right: 15px;
    background: #2b66e0;
    color: white;
    padding: 8px 16px;
    border-radius: 6px;
    text-decoration: none;
    font-size: 14px;
  }
  
  @media (max-width: 900px) {
    .layout {
      flex-direction: column;
    }
  
    .chart-img {
      width: 160px;
    }
  }
  </style>
  