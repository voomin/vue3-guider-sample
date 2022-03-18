<template>
  <button class="autoBtn autoBtn01" onclick="swiperBundleMin()"></button>
  <button class="autoBtn autoBtn02" onclick="swiperStart()"></button>
  <!--button class="autoFullScreen" onclick="openFullscreen(this)"></button-->

  <!-- 컨트롤러 버튼 클릭 시 활성화 되어있는 위치로 슬라이드 자동 이동 -->
  <h1 class="slidePositionNum allNumber">{{ pageLength() - 1 }}</h1>
  <h1 class="slidePositionNum pageNumber">{{ currentPageIndex }}</h1>

  <div id="header_container">
    <img
      id="exit"
      src="@/assets/exit.png"
      draggable="false"
      v-on:click="onExit"
    />
    <span id="title">{{ bookName }}</span>
    <div>
      <img
        id="menu"
        v-on:click="onMenu"
        src="@/assets/menu_on.png"
        draggable="false"
        v-if="showMenu"
      />
      <img
        id="menu"
        v-on:click="onMenu"
        src="@/assets/menu.png"
        draggable="false"
        v-else
      />
    </div>
    <div class="toggleBox">
      <div class="toggleWrap" v-on:click="onToggle">
        <img id="toggle" src="@/assets/toggle.png" draggable="false" />
      </div>
    </div>
  </div>

  <!-- 컨트롤러 열려있을 때 캔버스 영역 터치하면 컨트롤러 닫힘 버튼 반짝이는 영역 -->
  <div class="guardsLight" onclick="btnLight()"></div>

  <!-- 엑티비티 유무 확인 아이콘 -->
  <div id="interaction">
    <img src="@/assets/is_activity.png" alt="" />
  </div>

  <div id="thumbnail_container">
    <div class="slideBack" onclick="btnLight()"></div>
    <!-- 모바일 모드 슬라이드 중 class show 안사라지게 하기 -->
    <div
      class="slidePosition"
      v-on:mouseover="onSlide"
      v-on:mousedown="onSlide"
      v-on:mousemove="onSlide"
      v-on:mouseup="onSlide"
      v-on:touchstart="onSlide"
      v-on:touchmove="onSlide"
      v-on:touchend="onSlide"
    >
      <div class="swiper mySwiper">
        <div class="swiper-wrapper">
          <div class="swiper-slide"></div>
          <div
            v-for="(item, index) in pages"
            v-bind:key="index"
            v-on:click="onThumbnail(index)"
            class="swiper-slide repeatCon"
          >
            <div
              class="thumbnail_current"
              v-if="index == currentPageIndex"
            ></div>
            <span class="thumbnail_index">{{ index + 1 }}</span>
            <img class="thumbnail" v-bind:src="item.imagePath" />
            <div class="isActive">
              <img
                class="thumbnail_has_activity"
                src="@/assets/has_activity.png"
                v-if="item.hasInteraction"
              />
              <div>
                <!-- 인터렉션 있는 페이지 상단 아이콘 세 번 깜빡이기 오류 방지 위해 비어있는 div -->
              </div>
            </div>
          </div>
        </div>
        <!-- <div class="swiper-pagination"></div> -->
      </div>
    </div>
  </div>
  <div id="menu_wrapper" v-if="showMenu" v-on:click="onMenuWrapper"></div>
  <div id="menu_container" v-if="showMenu">
    <ul>
      <li id="audio" v-on:click="onAudio">오디오만 듣기</li>
      <li id="vod" v-if="hasVOD" v-on:click="onVOD">영상으로 보기</li>
      <!--li id="quiz">
        <a href="https://imestudy.smartdoodle.net/lginterbook/quiz/index.html">퀴즈 풀기</a>
      </li-->
      <li id="mymenu" v-if="hasMyMenu" v-on:click="onMyMenu">마이메뉴</li>
    </ul>
    <img id="bubble" src="@/assets/bubble.png" alt="" />
  </div>
</template>

<script lang="ts">
import Timer from '@/util/Timer';
import { ExitCode } from '@/App.vue';
import { Options, Vue } from 'vue-class-component';
import { ControllerData, UiState, Page } from '@/store/ControllerDataModule';

@Options({
  emits: ['pause', 'resume', 'exit', 'audio', 'go'],
})
export default class Guider extends Vue {
  private timer: Timer = null;

  get bookName(): string {
    return ControllerData.bookName;
  }

  get showMenu(): boolean {
    return ControllerData.uiState == UiState.pausedAndMenu;
  }

  get hasVOD(): boolean {
    return ControllerData.hasVOD;
  }

  get hasMyMenu(): boolean {
    return ControllerData.hasMyMenu;
  }

  get currentPageIndex(): number {
    return ControllerData.currentPageIndex;
  }

  get totalPageCount(): number {
    return ControllerData.totalPageCount;
  }

  get pages(): Page[] {
    return ControllerData.pages;
  }

  public pageLength(): number {
    var repeatCon = document.querySelectorAll('.repeatCon');
    return repeatCon.length;
  }

  public mounted() {
    console.log('Guider::mounted');
    var autoBtn01: HTMLElement = document.querySelector('.autoBtn01');
    var autoBtn02: HTMLElement = document.querySelector('.autoBtn02');
    autoBtn01.click();
    autoBtn02.click();
    this.timer = new Timer(5000, this.onTimeout.bind(this));
  }

  public resetTimer() {
    if (this.timer) {
      if (
        ControllerData.uiState == UiState.paused ||
        ControllerData.uiState == UiState.pausedAndMenu
      ) {
        this.timer.reset();
      }
    }
  }

  private showGuider() {
    document.getElementById('header_container').className = 'show';
    document.getElementById('thumbnail_container').className = 'show';
    document.getElementById('toggle').className = 'active';
  }

  private hideGuider() {
    document.getElementById('header_container').className = '';
    document.getElementById('thumbnail_container').className = '';
    document.getElementById('toggle').className = '';
  }

  private onTimeout() {
    ControllerData.setUiState(UiState.playing);
    this.hideGuider();
    this.$emit('resume');
  }

  private onToggle() {
    console.log('onToggle');
    if (ControllerData.uiState == UiState.playing) {
      ControllerData.setUiState(UiState.paused);
      this.showGuider();
      this.$emit('pause');
      this.timer.reset();
    } else {
      ControllerData.setUiState(UiState.playing);
      this.hideGuider();
      this.$emit('resume');
      this.timer.clear();
    }
  }

  private onMenu() {
    console.log('onMenu');
    ControllerData.setUiState(UiState.pausedAndMenu);
    this.timer.reset();
  }

  private onMenuWrapper() {
    console.log('onMenuWrapper');
    ControllerData.setUiState(UiState.paused);
    this.timer.reset();
  }

  private onExit() {
    console.log('onExit');
    ControllerData.setUiState(UiState.paused);
    this.$emit('exit', ExitCode.exitButton);
    this.timer.clear();
  }

  private onAudio() {
    console.log('onAudio');
    ControllerData.setUiState(UiState.paused);
    this.$emit('audio');
    this.timer.clear();
  }

  private onVOD() {
    console.log('onVOD');
    ControllerData.setUiState(UiState.paused);
    this.$emit('exit', ExitCode.viewVOD);
    this.timer.clear();
  }

  private onMyMenu() {
    console.log('onMyMenu');
    ControllerData.setUiState(UiState.paused);
    this.$emit('exit', ExitCode.myMenu);
    this.timer.clear();
  }

  private onThumbnail(pageIndex: number) {
    console.log('onThumbnail');
    this.$emit('go', pageIndex);
    this.timer.reset();
  }

  // 슬라이드 중 class show 안사라짐
  private onSlide() {
    this.timer.reset();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#header_container {
  position: absolute;
  left: 0;
  top: -80px;
  width: 100%;
  height: 80px;
  box-sizing: border-box;
  background-color: rgba(0, 0, 0, 0.5);
  transition: top 0.3s ease-in;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  justify-content: space-between;
  z-index: 9;
}
#header_container.show {
  top: 0;
}
#title {
  color: #fff;
  font-size: 25px;
  font-weight: bold;
  cursor: default;
}
#toggle {
  position: absolute;
  right: 30px;
  top: 100%;
  transition: transform 0.2s ease-in;
}
#menu_wrapper {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.1);
  z-index: 9;
}
#menu_container {
  position: absolute;
  right: 100px;
  top: 100px;
  width: 300px;
  background-color: rgb(23, 29, 65);
}
#menu_container > li {
  color: #fff;
  font-size: 30px;
  font-weight: bold;
  cursor: default;
}
#audio {
  background-color: rgba(0, 0, 0, 0.5);
}
#vod {
  background-color: rgba(0, 0, 0, 0.5);
}
#mymenu {
  background-color: rgba(0, 0, 0, 0.5);
}

/* image drag exhibition */
img {
  -webkit-user-drag: none;
}
img {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* transition */
#header_container {
  transition-duration: 0.5s;
}
#thumbnail_container {
  transition-duration: 0.5s;
}
#menu_container {
  transition-duration: 0.5s;
}
#header_container * {
  transition-duration: 0.5s;
}
#thumbnail_container * {
  transition-duration: 0.5s;
}
#menu_container * {
  transition-duration: 0.5s;
}

/* add */
#header_container {
  height: 100px;
  background-color: rgba(0, 0, 0, 0);
  top: -200px;
}
#header_container::before {
  content: '';
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 110%;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.8),
    rgba(0, 0, 0, 0.45) 50%,
    rgba(0, 0, 0, 0.14) 79%,
    rgba(0, 0, 0, 0.03) 93%,
    rgba(0, 0, 0, 0)
  );
}
#toggle {
  filter: brightness(500%);
  position: absolute;
  display: block;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  right: auto;
}
#toggle.active {
  filter: brightness(100%);
}
#exit {
  position: absolute;
  top: 50%;
  left: 60px;
  transform: translateY(-50%);
  cursor: pointer;
}
#title {
  display: block;
  position: absolute;
  top: 50%;
  left: 120px;
  transform: translateY(-50%);
  color: #fff;
  font-size: 50px;
  line-height: 1.6em;
  font-weight: normal;
  width: 90%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-family: 'Sandoll';
}
#menu {
  display: block;
  position: absolute;
  top: 50%;
  right: 150px;
  transform: translateY(-50%);
  cursor: pointer;
}

/* 슬라이드 */
#thumbnail_container {
  position: absolute;
  bottom: -200px;
  left: 0;
  width: 100%;
  height: 100px;
  background-color: rgba(0, 0, 0, 0);
  transition: bottom 0.5s ease-in;
}
#thumbnail_container::before {
  content: '';
  display: block;
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 222%;
  background-image: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.8),
    rgba(0, 0, 0, 0.45) 50%,
    rgba(0, 0, 0, 0.14) 79%,
    rgba(0, 0, 0, 0.03) 93%,
    rgba(0, 0, 0, 0)
  );
}
#thumbnail_container.show .slideBack {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: none;
  background-color: rgba(0, 0, 0, 0);
}
#thumbnail_container.show {
  bottom: 0;
}
.slidePosition {
  position: absolute;
  bottom: -300px;
  right: 0px;
  width: 100%;
  max-width: 1920px;
  height: auto;
  padding-bottom: 13px;
}
#thumbnail_container.show .slidePosition {
  bottom: 30px;
}
.slidePosition .swiper {
  overflow: visible;
}

/* 슬라이드 콘텐츠 */
.slidePosition .swiper-slide {
  border-radius: 15px;
  top: 7px;
  padding: 4px;
  background-color: #fff;
}
.thumbnail {
  display: block;
  position: relative;
  width: 100%;
  border-radius: 15px;
}
.slidePosition .swiper-slide:first-child {
  visibility: hidden;
  opacity: 0;
}
.thumbnail_current {
  position: absolute;
  width: calc(100% + 10px);
  height: calc(100% + 10px);
  top: -5px;
  left: -5px;
  background-color: #794def;
  border-radius: 20px;
}
.thumbnail_has_activity {
  display: block;
  position: absolute;
  top: 4px;
  right: 4px;
  border-radius: 0 10px 0 0;
  transition-duration: 0s !important;
}
.thumbnail_current + .thumbnail_index + .thumbnail + .thumbnail_has_activity {
  top: 3px;
  right: 3px;
}
.thumbnail_index {
  display: table;
  position: absolute;
  bottom: 3px;
  left: 3px;
  width: auto;
  padding: 7px 36px;
  border-radius: 0 30px 0 5px;
  background-color: #fff;
  z-index: 9;
  color: #333;
  font-size: 22px;
  line-height: 1.6em;
  font-weight: 700;
  transition-duration: 0s !important;
  font-family: 'Noto Sans KR';
}
.thumbnail_current + .thumbnail_index {
  background-color: #794def;
  color: #fff;
}

/* 슬라이드 드래그 방지 */
#thumbnail_container * {
  -webkit-user-drag: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* 자동 풀 스크린 */
.autoFullScreen {
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: none;
  outline: none;
  background-color: rgba(0, 0, 0, 0);
  opacity: 0;
  z-index: 9999;
}

/* 비율 맞춘 헤더 */
#exit {
  width: 1.72vw;
  left: 5%;
}
#title {
  font-size: 3.28125vw;
  left: 10.15625%;
}
#toggle {
  width: 3.438vw;
}
#menu {
  width: 2.188vw;
  right: 10.9375%;
}
/**/
#header_container {
  height: 15%;
}
.toggleBox {
  position: fixed;
  top: 0;
  right: 3%;
  width: 6vw;
  height: 15vh;
}
.toggleWrap {
  position: absolute;
  top: 50%;
  right: 0;
  transform: translateY(-50%);
  width: 100%;
  height: auto;
  padding-bottom: 100%;
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 100%;
  cursor: pointer;
}
.show .toggleWrap {
  background-color: rgba(0, 0, 0, 0);
  transform: rotate(180deg) translateY(50%);
}
/**/
#menu_container {
  display: block;
  position: absolute;
  top: 15%;
  right: 2%;
  background-color: #606e84;
  border-radius: 1vw;
  width: 30.15625%;
  z-index: 9;
}
#menu_container ul {
  display: block;
  width: 100%;
}
#menu_container li {
  display: block;
  padding: 8.295% 9.33%;
  background-color: rgba(0, 0, 0, 0);
  border-bottom: 2px solid #707d91;
  cursor: pointer;
  color: #fff;
  font-size: 2.1875vw;
  line-height: 1em;
  font-weight: normal;
  font-family: 'Sandoll';
}
#menu_container li:last-child {
  border-bottom: none;
}
#bubble {
  display: block;
  position: absolute;
  top: -1vw;
  right: 8.77vw;
  width: 2.5vw;
}

/** 컨트롤러 열려있을 때 캔버스 영역 터치하면 컨트롤러 닫힘 버튼 반짝이는 영역 **/
.guardsLight {
  display: none;
}
#header_container.show + .guardsLight {
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.toggleWrap.lightBtn {
  transition-duration: 0s !important;
}
.show .toggleWrap.lightBtn {
  animation: shineBtn 0.4s linear 0s infinite alternate;
}
@keyframes shineBtn {
  0% {
    filter: brightness(100%);
    opacity: 0.5;
  }
  100% {
    filter: brightness(500%);
    background-color: rgba(225, 209, 58, 0.5);
    opacity: 0.5;
  }
}

/** 인터렉션 있는 페이지 책 상단 이미지 3번 깜빡임 **/
#interaction {
  position: fixed;
  top: 0;
  left: 3%;
  width: 4.6875%;
  height: 15vh;
  z-index: 99;
  opacity: 0;
}
#interaction img {
  display: block;
  position: absolute;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  width: 100%;
}
#interaction.active {
  animation: shineBtn02 0.5s linear 0s alternate;
  animation-iteration-count: 6;
}
@keyframes shineBtn02 {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

/** 컨트롤러 버튼 클릭 시 활성화 되어있는 위치로 슬라이드 자동 이동 **/
.slidePositionNum {
  position: relative;
  font-size: 100px;
  z-index: -9;
  opacity: 0;
  visibility: hidden;
}

/** 컨트롤러 활성화 중에는 엑티비티 유무 확인 아이콘이 완전 사라진다 **/
#header_container.show ~ #interaction {
  display: none !important;
}

/* 반응형 시작 */
@media screen and (max-width: 1280px) {
  /* 슬라이드 */
  .slidePosition {
    width: 1280px;
  }
}
@media screen and (max-width: 1024px) {
  /**
  #header_container{height:60px}
  #menu{top:10px; right:70px}
  #toggle{top:20px; right:18px}
  .toggleWrap{width:60px; height:60px; top:5px; right:9px}
  #exit{top:14px; left:15px}
  #title{width:calc(100% - 180px) ;left:60px; font-size:30px}
  #menu_container{top:80px; border-radius:14px}
  #bubble{right:36px}
  #menu_container li{border-bottom:1px solid #707d91; padding:8px 20px; font-size:20px}
  **/
  /* 슬라이드 */
  #thumbnail_container {
    height: 60px;
  }
  .slidePosition {
    width: 1024px;
  }
  #thumbnail_container.show .slidePosition {
    bottom: 10px;
  }
  .thumbnail_index {
    font-size: 18px;
    padding: 4px 24px;
    border-radius: 0 20px 0 5px;
  }
  .thumbnail_has_activity {
    width: 36px;
  }
}
@media screen and (max-width: 900px) {
  .slidePosition {
    width: 900px;
  }
  .thumbnail_index {
    font-size: 16px;
    padding: 3px 18px;
    border-radius: 0 20px 0 5px;
  }
  .thumbnail_has_activity {
    width: 32px;
  }
}
@media screen and (max-width: 768px) {
  /** #title{left:60px; font-size:26px} **/
  /* 슬라이드 */
  .slidePosition {
    width: calc(100% - 30px);
    left: 15px;
  }
  .slidePosition .swiper-slide:first-child {
    display: none;
  }
  .thumbnail_current {
    width: calc(100% + 6px);
    height: calc(100% + 6px);
    top: -3px;
    left: -3px;
  }
}
@media screen and (max-width: 500px) {
  /**
  #header_container{height:50px}
  #exit{width:15px; top:14px}
  #title{font-size:22px; left:50px; width:calc(100% - 150px)}
  #toggle{width:30px; top:19px}
  .toggleWrap{width:50px; height:50px}
  #menu{width:20px; top:11px; right:60px}
  #menu_container{top:70px; right:15px; width:300px}
  #bubble{right:37px}
  #menu_container li{padding:8px 18px; font-size:18px}
  **/
  /* 슬라이드 */
  #thumbnail_container {
    height: 50px;
  }
  #thumbnail_container.show .slidePosition {
    bottom: 5px;
  }
}
@media screen and (max-width: 360px) {
  /* menu */
  /**
  #menu_container{width:250px}
  #menu_container li{padding:8px 16px; font-size:16px}
  **/
}
</style>
