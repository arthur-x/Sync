<script setup>
import { ref, computed } from 'vue'
const time1 = ref()
const time2 = ref()
const inputURL = ref('')
const vl_h = ref(24)
const vl_m = ref(60)
const vl_s = ref(60)
const countdown = ref('')
const control = ref('')
const appState = ref(-1) // -1: Src not selected; 0: Video not selected; 1: Video selected, timer not set; 2: Conuting down; 3: Full screen playing
const uploadsignal = ref(0)

const confirm_disable = computed(_=>{
  let t1 = time1.value
  let t2 = time2.value
  let check1 = (t1 != undefined && t1 != null & t1 != '')
  let check2 = (t2 != undefined && t2 != null & t2 != '')
  return !(check1 && check2)
})

const getNextMinute = _=> {
  let nd = new Date()
  nd.setSeconds(0)
  let ndd = time2duration(nd) + 60
  if (ndd >= 24 * 3600) nd -= 24*3600
  let {h, m, s} = duration2hms(ndd)
  nd.setHours(h, m, s)
  return nd
}
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
function hms2time(h, m, s) {
  let time = new Date()
  time.setHours(h, m, s)
  return time
}
function formatTime(h, m, s) {
  h = h < 10 ? '0'+h : h;
  m = m < 10 ? '0'+m : m;
  s = s < 10 ? '0'+s : s;
  return h+':'+m+':'+s
}

function processURL() {
  const video = document.getElementById("video")
  video.src = inputURL.value
  video.oncanplay = _=> {
    uploadsignal.value = 1
  }
  video.onemptied = _=> {
    uploadsignal.value = 0
  }
}

function processFilm(ev) {
  let curFiles = ev.target.files
  if (curFiles.length == 0) return
  if (curFiles[0].type != 'video/mp4') {
    alert('only mp4 files are supported')
    return
  }
  const video = document.getElementById("video")
  video.src = URL.createObjectURL(curFiles[0])
  uploadsignal.value = 1
}

function processSubtitle(ev) {
  let curFiles = ev.target.files
  if (curFiles.length == 0) return
  if (curFiles[0].name.substr(-4) != '.vtt') {
    alert('only vtt files are supported')
    return
  }
  const track = document.getElementById("track")
  track.src = URL.createObjectURL(curFiles[0])
}

function confirm0() {
  appState.value = 1
}

function confirm() {
  const video = document.getElementById("video")
  video.pause()
  video.currentTime = time2duration(time1.value)
  if (video.textTracks) video.textTracks[0].mode = 'hidden'
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
  if (video.textTracks) video.textTracks[0].mode = 'showing'
  video.play()
}

function updateCountdown(std) {
  let time = new Date();
  let ctd = time2duration(time)
  if (std < ctd) std += 24 * 3600
  let {h, m, s} = duration2hms(std - ctd)
  countdown.value = formatTime(h, m, s)
}

document.onfullscreenchange = (ev) => {
  if (document.fullscreenElement == null) {
    if (appState.value >= 2) {
      const video = document.getElementById("video")
      if (appState.value == 3) {
        video.pause()
        let vc = video.currentTime
        let {h, m, s} = duration2hms(vc)
        time1.value = hms2time(h, m, s)
      }
      else {
        if (video.textTracks) video.textTracks[0].mode = 'showing'
      }
      time2.value = getNextMinute()
      appState.value = 1
    }
  }
}

document.onkeydown = (ev) => {
  if (ev.key == 's' && ev.altKey) {
    const video = document.getElementById("video")
    video.src = ''
    const track = document.getElementById("track")
    track.src = ''
    inputURL.value = ''
    uploadsignal.value = 0
    if (appState.value == 0) {
      appState.value = -1
      control.value = ''
    }
    else if (appState.value == -1) {
      appState.value = 0
      control.value = 'nodownload'
    }
  }
}

function time1change() {
  const video = document.getElementById("video")
  video.pause()
  video.currentTime = time2duration(time1.value)
}

function time1focus() {
  const video = document.getElementById("video")
  video.pause()
  if (vl_h.value == 24) {
    let {h, m, s} = duration2hms(video.duration)
    vl_h.value = h
    vl_m.value = m
    vl_s.value = s
  }
  let vc = video.currentTime
  let {h, m, s} = duration2hms(vc)
  time1.value = hms2time(h, m, s)
}

function time2focus() {
  time2.value = getNextMinute()
}
</script>

<template>
<div style="width:100%; height:100%; display: flex; justify-content: space-evenly; align-items: center; flex-direction: column;">
    <div class="video-container" :class="{fullscreen: appState>=2}">
      <video
        id="video"
        disablepictureinpicture
        :controlslist="control"
        :controls="appState<2"
        :class="{video_pale: appState==2}">
        <track id="track" default kind="captions">
      </video>
      <div class="banner" :class="{visible: appState==2}">
        <p>SCREENING</p>
        <p>{{ countdown }}</p>
        <p>COUNTDOWN</p>
      </div>
    </div>
    <div v-if="appState<2" style="display: flex; place-items: center;">
      <el-input v-if="appState==-1" v-model="inputURL" placeholder="Input Video Source URL ('Alt-S' To Select Local Files Instead)" @change="processURL"/>
      <label for="film" class="el-button" v-if="appState==0">Select Video</label>
      <input type="file" id="film" style="display: none;" v-if="appState==0" accept="video/mp4" @change="processFilm"/>
      <label for="subtitle" class="el-button" v-if="appState==0">Select Subtitle</label>
      <input type="file" id="subtitle" style="display: none;" v-if="appState==0" accept=".vtt" @change="processSubtitle"/>
      <el-button v-if="appState<=0" :disabled="uploadsignal==0" @click="confirm0">Confirm</el-button>
      <el-time-picker  
        :disabled-hours="disabledHours"
        :disabled-minutes="disabledMinutes"
        :disabled-seconds="disabledSeconds" 
        :editable=false 
        :clearable=false
        prefix-icon="Film"
        v-if="appState==1"
        v-model="time1" 
        :default-value="hms2time(0, 0, 0)"
        placeholder="Jump To:"
        @change="time1change"
        @focus="time1focus"
        style="display: inline-flex; width: var(--component-width); transition: width .25s;"
      />
      <el-time-picker 
        :editable=false
        :clearable=false
        v-if="appState==1"
        v-model="time2"
        placeholder="Screen At:"
        @focus="time2focus"
        style="display: inline-flex; width: var(--component-width); transition: width .25s;"
      />
      <el-button v-if="appState==1" :disabled="confirm_disable" @click="confirm">Confirm</el-button>
    </div>
</div>
</template>

<style scoped>
.video-container {
  width: 70vw;
  height: 80vh;
  position: relative;
  box-shadow: 0 0 2px;
  border-radius: 4px; 
  overflow: hidden;
  transition: width, .25s, height, .25s;
}

video {
  width: 100%;
  height: 100%;
  transition: opacity .25s;
}

video::cue {
  background-color:transparent;
  line-height: 1.5em;
  font-weight: 600;
  font-family: 'Courier New', Courier, monospace;
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
  transition: opacity .25s;;
}

p {
  margin-top: 0.6em;
  margin-bottom: 0.6em;
}

::selection {
  background: inherit;
}

.visible {
  opacity: 60%;
  z-index: 1;
}

.video_pale {
  opacity: 50%;
}

.fullscreen {
  width: 100%;
  height: 100%;
  box-shadow: none;
  cursor: none;
}

.el-button {
  width: var(--component-width);
  transition: width .25s;
}

.el-input {
  width: calc(2 * var(--component-width));
  transition: width .25s;
}
</style>
