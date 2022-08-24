<script setup>
import { ref, computed } from 'vue'
const time1 = ref()
const time1_default = ref(new Date(2022, 8, 24, 0, 0))
const time2 = ref()
const vl_h = ref(24)
const vl_m = ref(60)
const vl_s = ref(60)
const countdown = ref('')
const appState = ref(0) // 0: Film not selected; 1: Film selected, timer not set; 2: Conuting down; 3: Full screen playing

const confirm_disable = computed(_=>{
  return !(time1.value != null && time2.value != null)
})

const time2_default = computed(_=>{
  let check = time2.value == null
  let nd = new Date()
  nd.setSeconds(0)
  if (!check) return nd
  let ndd = time2duration(nd) + 120
  if (ndd > 24 * 3600) nd -= 24*3600
  let {h, m, s} = duration2hms(ndd)
  nd.setHours(h, m, s)
  return nd
})

const makeRange = (start, end) => {
  const result = []
  for (let i = start; i <= end; i++) {
    result.push(i)
  }
  return result
}
const disabledHours = () => {
  return makeRange(vl_h.value+1, 23)
}
const disabledMinutes = (hour) => {
  if (hour == vl_h.value) {
    return makeRange(vl_m.value+1, 59)
  }
}
const disabledSeconds = (hour, minute) => {
  if (hour == vl_h.value && minute == vl_m.value) {
    return makeRange(vl_s.value+1, 59)
  }
}
function duration2hms(duration) {
  let h = parseInt(duration / 3600)
  let m = parseInt((duration - h * 3600) / 60)
  let s = Math.floor(duration - h * 3600 - m * 60)
  return {h, m, s}
}
function time2duration(time) {
  let h = time.getHours();
  let m = time.getMinutes();
  let s = time.getSeconds();
  return h * 3600 + m * 60 + s
}
function formatTime(h, m, s) {
  h = h < 10 ? '0'+h : h;
  m = m < 10 ? '0'+m : m;
  s = s < 10 ? '0'+s : s;
  return h+':'+m+':'+s
}

function processFile(ev) {
  let curFiles = ev.target.files
  if (curFiles.length == 0) return
  // reject non-mp4 videos
  if (curFiles[0].type != 'video/mp4') {
    alert('only mp4 files are supported')
    return
  }
  const video = document.getElementById("video")
  video.src = URL.createObjectURL(curFiles[0])
  video.oncanplay = (_=> {
    let {h, m, s} = duration2hms(video.duration)
    vl_h.value = h
    vl_m.value = m
    vl_s.value = s
  })
  appState.value = 1
}

function confirm() {
  const video = document.getElementById("video")
  video.pause()
  video.currentTime = time2duration(time1.value)
  let del = document.documentElement
  del.requestFullscreen()
  beginCountdown()
}

function sleep(time) {
    return new Promise(resolve => setTimeout(resolve, time))
}

async function beginCountdown() {
  updateCountdown()
  appState.value = 2
  let std = time2duration(time2.value)
  while (countdown.value != '00:00:00') {
    await sleep(100)
    updateCountdown(std)
    if (appState.value != 2) return
  }
  appState.value = 3
  const video = document.getElementById("video")
  video.play()
}

document.onfullscreenchange = (ev) => {
  if (document.fullscreenElement == null) {
    if (appState.value >= 2) {
      if (appState.value == 3) {
        const video = document.getElementById("video")
        video.pause()
        let vc = video.currentTime
        let {h, m, s} = duration2hms(vc)
        time1.value = new Date(2022, 8, 24, h, m, s)
      }
      time2.value = null
      appState.value = 1
    }
  }
}

function updateCountdown(std) {
  // get current time
  let time = new Date();
  let ctd = time2duration(time)
  if (std < ctd) std += 24 * 3600
  let {h, m, s} = duration2hms(std - ctd)
  countdown.value = formatTime(h, m, s)
}
</script>

<template>
<div style="width:100%; height:100%; display: flex; justify-content: space-evenly; align-items: center; flex-direction: column;">
    <div class="video-container" :class="{fullscreen: appState>=2}">
      <video
        id="video"
        disablepictureinpicture
        controlslist="nodownload"
        :controls="appState<2"
        :class="{video_pale: appState==2}">
      </video>
      <div class="banner" :class="{visible: appState==2}">
        <p>SCREENING</p>
        <p>{{ countdown }}</p>
        <p>COUNTDOWN</p>
      </div>
    </div>
    <div v-if="appState<2" style="display: flex; place-items: center;">
      <label for="file" class="el-button" v-if="appState==0">Select Film</label>
      <input type="file" id="file" style="display: none;" v-if="appState==0" accept="video/mp4" @change="processFile"/>
      <el-time-picker  
        :disabled-hours="disabledHours"
        :disabled-minutes="disabledMinutes"
        :disabled-seconds="disabledSeconds" 
        :editable=false 
        :clearable=false
        v-if="appState==1"
        v-model="time1" 
        placeholder="Jump To:"
        :default-value="time1_default"
      />
      <el-time-picker 
        :editable=false
        :clearable=false
        v-if="appState==1"
        v-model="time2"
        placeholder="Screen On:"
        :default-value="time2_default"/>
      <el-button v-if="appState==1" :disabled="confirm_disable" @click="confirm">Confirm</el-button>
    </div>
</div>
</template>

<style scoped>
.video-container {
  width: 800px;
  height: 450px;
  position: relative;
  box-shadow: 0 0 2px; 
  overflow: hidden;
}

video {
  width: 100%;
  height: 100%;
}

.banner {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-left: 2.5vw;
  letter-spacing: 5vw;
  font-size: 10vw;
  font-weight: 600;
  opacity: 0%;
  z-index: -1;
  font-family:'Courier New', Courier, monospace;
}

p {
  margin-top: 0.6em;
  margin-bottom: 0.6em;
}

::selection {
  background: inherit;
}

.visible {
  opacity: 50%;
  z-index: 1;
}

.video_pale {
  opacity: 20%;
}

.fullscreen {
  width: 100%;
  height: 100%;
}
</style>
