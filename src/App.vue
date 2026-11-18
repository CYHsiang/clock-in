<template>
  <div class="min-h-screen bg-slate-50 text-slate-800 flex flex-col items-center justify-center p-4 selection:bg-theme-100 transition-colors duration-300">
    <main class="w-full max-w-md bg-white rounded-3xl shadow-xl border border-slate-100 overflow-hidden transition-colors duration-300">
      
      <!-- 頂部狀態與設定按鈕 -->
      <header class="bg-theme-600 px-6 py-5 text-white flex justify-between items-center transition-colors duration-300">
        <div>
          <p class="text-theme-200 text-xs font-medium tracking-wider uppercase">{{ todayDateStr }}</p>
          <h1 class="text-xl font-bold mt-0.5 flex items-center gap-2">
            <Clock class="w-5 h-5" /> 彈性上下班助手
          </h1>
        </div>
        <div class="flex gap-1">
          <!-- 佈景主題切換按鈕 -->
          <button @click="showThemePicker = !showThemePicker" class="p-2 rounded-full hover:bg-white/20 transition-colors" title="切換主題">
            <Palette class="w-5 h-5" />
          </button>
          <!-- 設定按鈕 -->
          <button @click="showSettings = !showSettings" class="p-2 rounded-full hover:bg-white/20 transition-colors" title="工時設定">
            <Settings class="w-5 h-5" />
          </button>
        </div>
      </header>

      <!-- 主題調色盤 -->
      <div v-if="showThemePicker" class="bg-theme-50 px-6 py-4 border-b border-theme-100 flex justify-center gap-4 transition-colors">
        <button @click="changeTheme('indigo')" class="w-8 h-8 rounded-full bg-indigo-500 ring-2 ring-offset-2 transition-transform hover:scale-110" :class="currentTheme === 'indigo' ? 'ring-indigo-500' : 'ring-transparent'"></button>
        <button @click="changeTheme('green')" class="w-8 h-8 rounded-full bg-emerald-500 ring-2 ring-offset-2 transition-transform hover:scale-110" :class="currentTheme === 'green' ? 'ring-emerald-500' : 'ring-transparent'"></button>
        <button @click="changeTheme('rose')" class="w-8 h-8 rounded-full bg-rose-500 ring-2 ring-offset-2 transition-transform hover:scale-110" :class="currentTheme === 'rose' ? 'ring-rose-500' : 'ring-transparent'"></button>
        <button @click="changeTheme('amber')" class="w-8 h-8 rounded-full bg-amber-500 ring-2 ring-offset-2 transition-transform hover:scale-110" :class="currentTheme === 'amber' ? 'ring-amber-500' : 'ring-transparent'"></button>
      </div>

      <!-- 可折疊的工時設定區 (支援核心工時與午休分離) -->
      <div v-if="showSettings" class="bg-theme-50/70 px-6 py-4 border-b border-theme-100 text-sm space-y-3 transition-colors">
        <div class="flex justify-between items-center">
          <label class="font-medium text-slate-700">每日核心工時</label>
          <div class="flex items-center gap-2">
            <input type="number" step="0.5" min="1" max="16" v-model.number="coreHours" class="w-20 px-2.5 py-1 text-center rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-theme-500 transition-colors" />
            <span class="text-slate-600">小時</span>
          </div>
        </div>
        <div class="flex justify-between items-center">
          <label class="font-medium text-slate-700">午休/休息時間</label>
          <div class="flex items-center gap-2">
            <input type="number" step="0.5" min="0" max="4" v-model.number="lunchHours" class="w-20 px-2.5 py-1 text-center rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-theme-500 transition-colors" />
            <span class="text-slate-600">小時</span>
          </div>
        </div>
        <p class="text-xs text-slate-500 pt-1">
          * 總在班時間將自動計算為：{{ totalWorkHours }} 小時。修改會自動存檔。
        </p>
      </div>

      <!-- 主操作區 -->
      <div class="p-6 space-y-6">
        
        <!-- 上班打卡時間輸入 -->
        <section class="space-y-3">
          <div class="flex justify-between items-center">
            <label class="text-sm font-semibold text-slate-600">早上打卡時間</label>
            <span v-if="clockInTime" class="text-xs px-2.5 py-0.5 bg-emerald-100 text-emerald-700 rounded-full font-medium">
              今日已記錄
            </span>
          </div>

          <div class="flex gap-3">
            <input 
              type="time" 
              v-model="clockInTime"
              class="flex-1 text-2xl font-bold text-center bg-slate-100 border border-slate-200 rounded-2xl px-4 py-3 focus:outline-none focus:ring-2 focus:ring-theme-500 transition-all"
            />
            <button 
              @click="recordNow"
              class="bg-theme-600 hover:bg-theme-700 active:scale-95 text-white font-medium px-5 py-3 rounded-2xl shadow-sm flex items-center gap-2 transition-all shrink-0"
            >
              <Zap class="w-4 h-4" /> 現在打卡
            </button>
          </div>

          <!-- 快速微調按鈕 -->
          <div v-if="clockInTime" class="flex justify-center gap-2 pt-1">
            <button @click="adjustMinutes(-5)" class="text-xs px-3 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 font-medium transition-colors">-5 分</button>
            <button @click="adjustMinutes(-1)" class="text-xs px-3 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 font-medium transition-colors">-1 分</button>
            <button @click="adjustMinutes(1)" class="text-xs px-3 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 font-medium transition-colors">+1 分</button>
            <button @click="adjustMinutes(5)" class="text-xs px-3 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 font-medium transition-colors">+5 分</button>
          </div>
        </section>

        <!-- 備註輸入區 -->
        <section v-if="clockInTime" class="space-y-2">
           <div class="flex items-center gap-2 text-sm font-semibold text-slate-600">
             <MessageSquare class="w-4 h-4" /> 今日備註 (選填)
           </div>
           <input 
             type="text" 
             v-model="dailyNote"
             @change="saveDailyNote"
             placeholder="例如：早上去其他院區開會、忘記帶識別證補登..."
             class="w-full bg-slate-50 border border-slate-200 rounded-xl px-4 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-theme-500 transition-all text-slate-700"
           />
        </section>

        <!-- 下班時間與倒數面板 -->
        <section v-if="clockInTime" class="bg-gradient-to-br from-slate-900 to-theme-950 text-white rounded-2xl p-6 text-center space-y-4 shadow-inner transition-colors duration-500">
          <div>
            <p class="text-theme-200 text-xs uppercase tracking-wider font-medium">預計下班打卡時間</p>
            <div class="text-5xl font-extrabold tracking-tight mt-1 text-amber-400">
              {{ clockOutTime }}
            </div>
          </div>

          <!-- 行事曆按鈕 -->
          <div class="flex justify-center mt-2">
            <button @click="addToCalendar" class="bg-white/10 hover:bg-white/20 text-white text-xs px-4 py-2 rounded-lg flex items-center gap-2 transition-colors border border-white/20">
              <CalendarPlus class="w-4 h-4" /> 加入行事曆提醒下班
            </button>
          </div>

          <!-- 進度條與狀態 -->
          <div class="space-y-1.5 pt-2">
            <div class="w-full h-2.5 bg-slate-700/80 rounded-full overflow-hidden">
              <div 
                class="h-full bg-gradient-to-r from-theme-400 to-theme-300 transition-all duration-500"
                :style="{ width: `${progressPercent}%` }"
              ></div>
            </div>
            <div class="flex justify-between text-xs text-slate-300 font-medium">
              <span>今日進度 {{ progressPercent.toFixed(0) }}%</span>
              <span>{{ countdownText }}</span>
            </div>
          </div>
        </section>

        <!-- 尚未打卡時的空白提示 -->
        <section v-else class="border-2 border-dashed border-slate-200 rounded-2xl p-8 text-center text-slate-400 space-y-2">
          <CalendarClock class="w-10 h-10 mx-auto text-slate-300 stroke-[1.5]" />
          <p class="text-sm font-medium text-slate-500">今天還沒有記錄打卡時間喔！</p>
          <p class="text-xs">點擊上方「現在打卡」或手動選擇時間</p>
        </section>

        <!-- 最近打卡歷史紀錄 -->
        <section v-if="historyList.length > 0" class="space-y-2 pt-2 border-t border-slate-100">
          <div class="flex justify-between items-center">
            <h2 class="text-xs font-semibold text-slate-400 uppercase tracking-wider">近期打卡紀錄</h2>
            <button @click="clearHistory" class="text-xs text-slate-400 hover:text-rose-500 transition-colors">清空</button>
          </div>
          <div class="space-y-2 max-h-48 overflow-y-auto pr-1">
            <div 
              v-for="item in historyList" 
              :key="item.date"
              class="flex flex-col bg-slate-50 px-3 py-2.5 rounded-xl border border-slate-100"
            >
              <div class="flex justify-between items-center text-xs">
                <span class="font-medium text-slate-600">{{ item.date }}</span>
                <span class="text-slate-500">上班 <strong class="text-slate-700">{{ item.inTime }}</strong> → 下班 <strong class="text-theme-600">{{ item.outTime }}</strong></span>
              </div>
              <div v-if="item.note" class="text-xs text-slate-500 mt-1.5 bg-white px-2 py-1 rounded border border-slate-100 flex items-start gap-1">
                <MessageSquare class="w-3 h-3 mt-0.5 shrink-0 text-slate-400" />
                <span class="leading-relaxed">{{ item.note }}</span>
              </div>
            </div>
          </div>
        </section>

        <!-- 底部操作 -->
        <footer class="pt-2 border-t border-slate-100 flex justify-between items-center text-xs text-slate-400">
          <span>現在時間：{{ currentTimeStr }}</span>
          <button 
            v-if="clockInTime"
            @click="clearToday" 
            class="text-rose-500 hover:underline flex items-center gap-1 font-medium"
          >
            <RotateCcw class="w-3.5 h-3.5" /> 清除今日
          </button>
        </footer>

      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import { Clock, Settings, Zap, CalendarClock, RotateCcw, CalendarPlus, Palette, MessageSquare } from 'lucide-vue-next'

// === 狀態管理 ===
const showSettings = ref(false)
const showThemePicker = ref(false)
const currentTheme = ref('indigo')

const coreHours = ref(8)      // 核心工時
const lunchHours = ref(1)     // 午休時間
const clockInTime = ref('')
const dailyNote = ref('')
const historyList = ref([])
const now = ref(new Date())
let timer = null

// === 計算屬性 ===
const totalWorkHours = computed(() => Number(coreHours.value) + Number(lunchHours.value))

const getTodayKey = () => {
  const d = new Date()
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`
}

const todayDateStr = computed(() => {
  return now.value.toLocaleDateString('zh-TW', { year: 'numeric', month: 'long', day: 'numeric', weekday: 'short' })
})

const currentTimeStr = computed(() => now.value.toTimeString().slice(0, 8))

const clockOutTime = computed(() => {
  if (!clockInTime.value) return '--:--'
  const [h, m] = clockInTime.value.split(':').map(Number)
  const totalMinutes = h * 60 + m + Math.round(totalWorkHours.value * 60)
  const outH = Math.floor(totalMinutes / 60) % 24
  const outM = totalMinutes % 60
  return `${String(outH).padStart(2, '0')}:${String(outM).padStart(2, '0')}`
})

// 下班倒數計算邏輯
const workStatus = computed(() => {
  if (!clockInTime.value) return { percent: 0, text: '' }

  const [inH, inM] = clockInTime.value.split(':').map(Number)
  const start = new Date(now.value)
  start.setHours(inH, inM, 0, 0)

  const durationMs = totalWorkHours.value * 60 * 60 * 1000
  const end = new Date(start.getTime() + durationMs)
  const diffMs = end - now.value
  const elapsedMs = now.value - start
  const percent = Math.min(100, Math.max(0, (elapsedMs / durationMs) * 100))

  if (diffMs > 0) {
    const remainTotalSec = Math.floor(diffMs / 1000)
    const rHours = Math.floor(remainTotalSec / 3600)
    const rMinutes = Math.floor((remainTotalSec % 3600) / 60)
    const rSeconds = remainTotalSec % 60
    return {
      percent,
      text: `還有 ${rHours} 小時 ${String(rMinutes).padStart(2, '0')} 分 ${String(rSeconds).padStart(2, '0')} 秒`
    }
  } else {
    return {
      percent: 100,
      text: `🎉 已經可以下班打卡囉！`
    }
  }
})

const progressPercent = computed(() => workStatus.value.percent)
const countdownText = computed(() => workStatus.value.text)

// === 動作函式 ===
const changeTheme = (theme) => {
  currentTheme.value = theme
  document.documentElement.setAttribute('data-theme', theme)
  localStorage.setItem('flex_theme', theme)
}

const recordNow = () => {
  const d = new Date()
  clockInTime.value = `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`
}

const adjustMinutes = (delta) => {
  if (!clockInTime.value) return
  const [h, m] = clockInTime.value.split(':').map(Number)
  let total = h * 60 + m + delta
  if (total < 0) total += 24 * 60
  total = total % (24 * 60)
  clockInTime.value = `${String(Math.floor(total / 60)).padStart(2, '0')}:${String(total % 60).padStart(2, '0')}`
}

const updateHistory = (dateKey, inTime, outTime, note) => {
  const existingIdx = historyList.value.findIndex(item => item.date === dateKey)
  if (existingIdx !== -1) {
    historyList.value[existingIdx] = { date: dateKey, inTime, outTime, note }
  } else {
    historyList.value.unshift({ date: dateKey, inTime, outTime, note })
  }
  historyList.value = historyList.value.slice(0, 10)
  localStorage.setItem('flex_clock_history', JSON.stringify(historyList.value))
}

const saveDailyNote = () => {
  if (clockInTime.value) {
    const today = getTodayKey()
    localStorage.setItem('flex_daily_note', dailyNote.value)
    updateHistory(today, clockInTime.value, clockOutTime.value, dailyNote.value)
  }
}

const clearToday = () => {
  clockInTime.value = ''
  dailyNote.value = ''
  localStorage.removeItem('flex_clock_in_record')
  localStorage.removeItem('flex_daily_note')
  document.title = '彈性上下班打卡助手'
}

const clearHistory = () => {
  if(confirm('確定要清空所有歷史紀錄嗎？')) {
    historyList.value = []
    localStorage.removeItem('flex_clock_history')
  }
}

// 加入行事曆 (產生 .ics 檔案)
const addToCalendar = () => {
  const [outH, outM] = clockOutTime.value.split(':').map(Number)
  const d = new Date()
  
  // 建立 UTC 時間字串 YYYYMMDDTHHMMSSZ
  const getICSDateStr = (date) => {
    return date.toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z'
  }

  // 設定下班時間為事件開始
  const startTime = new Date(d.getFullYear(), d.getMonth(), d.getDate(), outH, outM, 0)
  // 設定事件長度 15 分鐘
  const endTime = new Date(startTime.getTime() + 15 * 60000)

  const icsContent = [
    'BEGIN:VCALENDAR',
    'VERSION:2.0',
    'PRODID:-//Flex Clock In App//TW',
    'BEGIN:VEVENT',
    `DTSTART:${getICSDateStr(startTime)}`,
    `DTEND:${getICSDateStr(endTime)}`,
    'SUMMARY:該下班打卡囉！',
    'DESCRIPTION:辛苦了一天，記得去打卡下班。\n' + (dailyNote.value ? `今日備註: ${dailyNote.value}` : ''),
    'BEGIN:VALARM',
    'TRIGGER:-PT0M', // 0分鐘前提醒 (準點提醒)
    'ACTION:DISPLAY',
    'DESCRIPTION:該下班打卡囉！',
    'END:VALARM',
    'END:VEVENT',
    'END:VCALENDAR'
  ].join('\r\n')

  const blob = new Blob([icsContent], { type: 'text/calendar;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.setAttribute('download', `下班提醒_${getTodayKey()}.ics`)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}

// === 監聽與生命週期 ===
watch([clockInTime, coreHours, lunchHours], () => {
  localStorage.setItem('flex_core_hours', String(coreHours.value))
  localStorage.setItem('flex_lunch_hours', String(lunchHours.value))
  
  if (clockInTime.value) {
    const today = getTodayKey()
    localStorage.setItem('flex_clock_in_record', JSON.stringify({
      date: today,
      time: clockInTime.value
    }))
    updateHistory(today, clockInTime.value, clockOutTime.value, dailyNote.value)
    document.title = `(${clockOutTime.value} 下班) 打卡助手`
  } else {
    document.title = '彈性上下班打卡助手'
  }
})

onMounted(() => {
  // 載入主題設定
  const savedTheme = localStorage.getItem('flex_theme')
  if (savedTheme) {
    currentTheme.value = savedTheme
    document.documentElement.setAttribute('data-theme', savedTheme)
  }

  // 載入工時設定與紀錄
  if (localStorage.getItem('flex_core_hours')) coreHours.value = parseFloat(localStorage.getItem('flex_core_hours'))
  if (localStorage.getItem('flex_lunch_hours')) lunchHours.value = parseFloat(localStorage.getItem('flex_lunch_hours'))
  if (localStorage.getItem('flex_daily_note')) dailyNote.value = localStorage.getItem('flex_daily_note')
  
  if (localStorage.getItem('flex_clock_history')) {
    try { historyList.value = JSON.parse(localStorage.getItem('flex_clock_history')) } catch (e) {}
  }

  const savedRecord = localStorage.getItem('flex_clock_in_record')
  if (savedRecord) {
    try {
      const parsed = JSON.parse(savedRecord)
      if (parsed.date === getTodayKey()) {
        clockInTime.value = parsed.time
      } else {
        // 如果是新的一天，清空昨天的備註
        dailyNote.value = ''
        localStorage.removeItem('flex_daily_note')
      }
    } catch (e) {}
  }

  // 每秒更新時鐘
  timer = setInterval(() => {
    now.value = new Date()
  }, 1000)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>
