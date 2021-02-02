<template>
  <div class="home" :style="{'background-image': `url(${backgroundImage})`}">
    <div v-if="!init" class="init-small" :style="{color: `${winnerColor}`}">{{initText.replace(/\\n/g, '')}}</div>
    <div v-if="init" class="init" :style="{'font-size': `${initTextFontSize}px`, color: `${winnerColor}`}" v-html="initText.replace(/\\n/g, '<br>')"></div>
    <div v-if="!init" class="winner" :style="{'font-size': `${winnerFontSize}px`, color: `${winnerColor}`, transform: `translate(${winnerTranslateX}px,${winnerTranslateY}px)`}">
      <div v-for="(name, index) in select" :key="index">{{name}}</div>
    </div>
    <div @click="onSelectClick" class="button" :style="{'background-image': `url(${buttonImage})`, 'font-size': `${buttonFontSize}px`, transform: `translate(${buttonTranslateX}px,${buttonTranslateY}px)`, width: `${buttonWidth}px`, height: `${buttonHeight}px`, 'line-height': `${buttonHeight}px`}">{{init ? buttonText : buttonText2}}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      select: [],
      init: true,

      count: 0,
      names: '',
      chosen: '',
      repeat: false,
      initText: '',
      initTextFontSize: 100,
      backgroundImage: '',
      buttonText: '开始',
      buttonText2: '结束',
      buttonImage: '',
      buttonFontSize: 16,
      buttonColor: '#000000',
      buttonWidth: 80,
      buttonHeight: 60,
      buttonTranslateX: 0,
      buttonTranslateY: 0,
      winnerFontSize: 40,
      winnerColor: '#000000',
      winnerTranslateX: 0,
      winnerTranslateY: 0,

      interval: null,
    };
  },
  methods: {
    getWinners() {
      const selection = JSON.parse(localStorage.getItem('selection'));
      const winners = new Set();
      selection.forEach(item => {
        if (item.valid) {
          item.select.forEach(name => winners.add(name));
        }
      });
      return Array.from(winners);
    },
    getNames() {
      let names = this.names.split(',').map(name => name.trim());
      if (!this.repeat) {
        const winners = this.getWinners();
        names = names.filter(name => !winners.includes(name));
      }
      return names;
    },
    roll() {
      const names = this.getNames();
      const all = names;
      if (all.length < this.count) {
        throw '可中奖人数已不足抽取人数';
      }

      const result = [];
      while (result.length < this.count) {
        const select = all[Math.floor(Math.random() * all.length)];
        if (result.indexOf(select) === -1) {
          result.push(select);
        }
      }
      return result;
    },
    randomSelect() {
      const names = this.getNames();

      const result = [];

      while (result.length < this.count) {
        const select = names[Math.floor(Math.random() * names.length)];
        if (result.indexOf(select) === -1) {
          result.push(select);
        }
      }

      return result.sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5);
    },
    resetAction() {
      const config = JSON.parse(localStorage.getItem('config'));
      config.action = null;
      localStorage.setItem('config', JSON.stringify(config));
    },
    start() {
      if (!this.interval && this.init) {
        try {
          this.init = false;
          this.select = this.roll();
          this.interval = setInterval(() => {
            this.select = this.roll();
          }, 100);
        } catch (e) {
          alert(e);
          this.init = true;
        }
      }
    },
    stop() {
      if (this.interval) {
        clearInterval(this.interval);
        this.interval = null;
        this.select = this.randomSelect();
        this.saveSelect();
      }
    },
    saveSelect() {
      const selection = JSON.parse(localStorage.getItem('selection'));
      const date = new Date(); 
      selection.push({
        initText: this.initText,
        time: date.getTime(),
        select: this.select,
        valid: true,
      });
      localStorage.setItem('selection', JSON.stringify(selection));
    },
    onSelectClick() {
      if (!this.interval) {
        this.start();
      } else {
        this.stop();
      }
    }
  },
  mounted() {
    let config = {};
    if (localStorage.getItem('config')) {
      config = JSON.parse(localStorage.getItem('config'));
    }
    console.log(config);
    Object.assign(this.$data, config);
    window.addEventListener('storage', () => {
      console.log('receive message');
      const config = JSON.parse(localStorage.getItem('config'));
      Object.assign(this.$data, config);

      const action = config.action;
      if (action) {
        console.log('action', action);
        if (action === 'init') {
          this.init = true;
        } else if (action === 'start') {
          this.start();
        } else if (action === 'stop') {
          this.stop();
        }
        this.resetAction();
      }
    });
    this.getWinners();
  },
  created() {
    let selection = localStorage.getItem('selection');
    if (!selection) {
      localStorage.setItem('selection', JSON.stringify([]));
    }
  }
};
</script>
<style scoped>
.home {
  display: flex;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-size: cover;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  background-position: center;
}
.winner {
  display: flex;
  max-width: 70vw;
  text-align: center;
  flex-wrap: wrap;
  /* font-family: monospace; */
  justify-content: center;
}
.winner > div {
  margin: 20px;
}
.init {
  max-width: 70vw;
  text-align: center;
}
.button {
  background-repeat: no-repeat;
  background-size: cover;
  overflow: hidden;
  text-align: center;
  cursor: pointer;
}
.init-small {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translate(-50%);
}
</style>
