<template>
  <div id="app">
    <h1 class="heading">Simon The Game</h1>
    <form action="#" class="mode-controls" :class="toggleModeDisabled ? 'disabled' : null">
      <div 
        class="control-mode-item" 
        v-for="item of modes" 
        :key="item" 
        :title="toggleModeDisabled ? 'Finish current game before changing dificult level' : null"
      >
        <input 
          :disabled="toggleModeDisabled" 
          type="radio" 
          :id="item" 
          :value="item" 
          v-model="currentMode"
        >
        <label :for="item">{{item}}</label>
      </div>
    </form>
    <div class="game-controls" >
      <div 
        v-for="(item, key) in controls" 
        class="audio-btn" 
        :class="[{disactive: isPlaying, clickedClass: item.clicked}, `audio-item-${key}`]"
        :key="key" 
        @click="clickHandler(key)"
      />
    </div>
    <button class="start-btn" @click="start">play</button>
    <p class="counter">Round: {{line.length}}</p>
    <p class="res">{{loose ? `Sorry! You loose after ${line.length} rounds` : ''}}</p>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {},
  data() {
    return {
      controls: {
        1: {
          clicked: false,
          audio: new Audio()
        },
        2: {
          clicked: false,
          audio: new Audio()
        },
        3: {
          clicked: false,
          audio: new Audio()
        },
        4: {
          clicked: false,
          audio: new Audio()
        },
      },
      modes: ['easy', 'medium', 'hard'],
      currentMode: 'medium',
      toggleModeDisabled: false,
      line: [],
      userline: [],
      isPlaying: true,
      loose: false,
    }
  },
  computed: {
    // устанавливает величину интервала между сигналами в завиимости от выбранного уровня сложности
    chosenMode() {
      switch (this.currentMode) {
        case 'medium':
          return 1000;
        case 'hard':
          return 400;
        default:
          return 1500;
      } 
    }
  },
  // устанавливаем источники для воспроизведения
  mounted() {
    for (let key in this.controls) {
      this.controls[key].audio.src = this.controls[key].audio.canPlayType('audio/mpeg') ? 
        (`./sounds/${key}.mp3`) : (`./sounds/${key}.ogg`);
    }
  },
  methods: {
    // обрабатывает нажатия игрока
    clickHandler(num) {
      if (this.loose || this.isPlaying) return;
      this.controls[num].clicked = true
      setTimeout(() => {this.controls[num].clicked = false}, 300);
      this.userline.push(+num);
      this.controls[num].audio.currentTime = 0;
      this.controls[num].audio.play();
      this.compare();
    },
    // запускает новую игру
    start() {
      this.loose = false;
      this.line = [];
      this.toggleModeDisabled = true;
      this.play();
    },
    // запускает новый раунд игры
    play() {
      this.isPlaying = true;
      this.userline = [];
      this.line.push(Math.ceil(Math.random() * 4));
      this.line.forEach((el, idx) => {
        setTimeout(() => {
          this.controls[el].audio.currentTime = 0;
          this.controls[el].audio.play();
          this.controls[el].clicked = true;
          setTimeout(() => {
            this.controls[el].clicked = false;
            (idx === this.line.length - 1) && (this.isPlaying = false);
          }, this.chosenMode / 2);
        }, idx * this.chosenMode);
      })
    },
    // проверяет ответы игрока
    compare() {
      let res = this.userline.every((el, idx) => el === this.line[idx]);
      !res && (this.loose = true);
      !res && (this.toggleModeDisabled = false);
      (res && this.line.length === this.userline.length) && setTimeout(() => this.play(), 1000);
    }
  }
}
</script>

<style lang="sass">
#app
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;


.heading
  font-weight: 500;


.mode-controls
  display: flex;
  justify-content: center;
  padding-top: 20px;

  &.disabled
    filter: opacity(.5);

  input[type="radio"]
    display: none;

    & + label
      display: block;
      margin: 0 10px;
      border-radius: 3px;
      cursor: pointer;

    &:checked + label
      background-color: peru;
      color: #fff;
      transition: all .2s;

    &:disabled + label
      cursor: default;

.game-controls
  display: grid;
  grid-template-columns: 100px 100px;
  grid-template-rows: 100px 100px;
  width: 200px;
  margin: 20px auto;

  .audio-btn
    display: block;
    width: 100px;
    height: 100px;
    border-radius: 80%;
    filter: opacity(.5);
    -webkit-tap-highlight-color: transparent;
    cursor: pointer;
    transition: all .2s;

    &.disactive
      cursor: default;
    
    &.audio-item-1
      background-color: blue;
      border-top-right-radius: 0;
      border-bottom-right-radius: 0;
      border-bottom-left-radius: 0;

    &.audio-item-2
      background-color: green;
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
      border-bottom-right-radius: 0;

    &.audio-item-3
      background-color: red;
      border-top-right-radius: 0;
      border-bottom-right-radius: 0;
      border-top-left-radius: 0;

    &.audio-item-4
      background-color: yellow;
      border-top-right-radius: 0;
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;

    &.clickedClass
      filter: none;

.start-btn
  width: 100px;
  height: 30px;
  margin: 15px;
  border: none;
  outline: none;
  background-color: rgb(235, 210, 183);
  color: peru;
  border-radius: 10px;
  cursor: pointer;
  transition: all .2s;

  &:hover
    background-color: peru;
    color: #fff;

.counter
  margin-bottom: 10px;
</style>
