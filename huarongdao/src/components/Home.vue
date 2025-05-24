<template>
  <div class="all">
    <div class="title"></div>

    <div class="container">
      <div v-for="(row, rowKey) in boxTextMap" :key="rowKey">
        <div
          v-for="(col, colKey) in row"
          class="box"
          :class="{ 'bg-pink': boxTextMap[rowKey][colKey] }"
          @touchstart="boxClick(rowKey, colKey)"
          :key="colKey">
          <span>{{ boxTextMap[rowKey][colKey] }}</span>
        </div>
      </div>
    </div>

    <div class="setting" v-if="isEdit === 'chooseDifficultity'">
      <div @touchstart="chooseDifficultity(3)">3</div>
      <div @touchstart="chooseDifficultity(4)">4</div>
      <div @touchstart="chooseDifficultity(5)">5</div>
      <div @touchstart="chooseDifficultity(6)">6</div>
      <div @touchstart="chooseDifficultity(7)">7</div>
      <div @touchstart="chooseDifficultity(8)">8</div>
      <div @touchstart="chooseDifficultity(9)">9</div>
      <div @touchstart="chooseDifficultity(10)">10</div>
      <div @touchstart="chooseDifficultity(11)">11</div>
    </div>


    <div class="setting" v-if="isEdit === 'settingModel'">
      <div @touchstart="randomSetting">随机分配</div>
      <div v-if="randomSettingDone" style="display: inline-block;" @touchstart="confirmClick(true)">开始游戏</div>
      <div @touchstart="customSetting">自定义</div>
      <div @touchstart="backToChooseDifficult">选择难度</div>
    </div>

    <div class="setting" v-if="isEdit === 'customSetting'">
      <div @touchstart="redoClick">回退</div>
      <div @touchstart="confirmClick(false)">开始游戏</div>
      <div @touchstart="backToChooseSettingModel">选择模式</div>
    </div>

    <div class="setting" v-if="isEdit === 'playing'">
      <div class="timer">
        {{ time / 100 }} s
      </div>
      <div @touchstart="endGame(true)">结束游戏</div>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      difficulty: 0,
      doneList: [],
      boxTextMap: [],
      isEdit: 'chooseDifficultity',
      randomSettingDone: false,
      time: 0,
      timer: null,
    }
  },
  created() {
    
  },
  methods: {
    backToChooseDifficult() {
      this.difficulty = 0;
      this.settingDifficult();
      this.isEdit = 'chooseDifficultity';
    },
    backToChooseSettingModel() {
      this.isEdit = 'settingModel';
    },
    randomSetting() {
      const list = new Array(this.difficulty * this.difficulty - 1).fill(0).map((i, k) => k + 1);
      for(let rowKey in this.boxTextMap) {
          if (this.boxTextMap.hasOwnProperty(rowKey)) {
          for(let colKey in this.boxTextMap[rowKey]) {
              if (this.boxTextMap[rowKey].hasOwnProperty(colKey)) {
                const r = Math.floor(Math.random() * list.length);

                if (!list.length) {
                  break;
                }
                this.boxTextMap[rowKey][colKey] = list.splice(r, 1)[0];
              }
          }
        }
      }
      this.boxTextMap = JSON.parse(JSON.stringify(this.boxTextMap));
      this.randomSettingDone = true;
    },
    customSetting() {
      this.isEdit = 'customSetting';
      this.settingDifficult();
    },
    settingDifficult() {
      this.boxTextMap = JSON.parse(JSON.stringify(new Array(this.difficulty).fill(
        new Array(this.difficulty).fill(null)
      )));
      this.doneList = [];

      if (this.difficulty === 0) {
        return;
      }

      this.$nextTick(() => {
        const boxes = document.querySelectorAll('.box');
        
        for(let i in boxes) {
          if (boxes.hasOwnProperty(i)) {
            const boxWidth = (900 / this.difficulty - 2);
            boxes[i].style.width = boxWidth + 'px';
            boxes[i].style.height = boxWidth + 'px';
            boxes[i].style.lineHeight = boxWidth + 'px';
          }
        }
      })
    },
    chooseDifficultity(difficulty) {
      this.difficulty = difficulty;
      this.isEdit = 'settingModel';
      this.settingDifficult();
    },
    boxClick(rowKey, colKey) {
      if (this.isEdit === 'customSetting') {
        this.boxEditClick(rowKey, colKey);
      } else if (this.isEdit === 'playing') {
        this.boxPlayClick(rowKey, colKey);
      }
    },
    boxPlayClick(rowKey, colKey) {
      const left = this.boxTextMap[rowKey]?.[colKey - 1];
      const right = this.boxTextMap[rowKey]?.[colKey + 1];
      const top = this.boxTextMap[rowKey + 1]?.[colKey];
      const bottom = this.boxTextMap[rowKey - 1]?.[colKey];
      
      const nullBoxDirectionKey = [left, right, top, bottom].indexOf(null);

      if (nullBoxDirectionKey !== -1) {
        const index = this.getNullBoxIndex(rowKey, colKey, nullBoxDirectionKey)
        this.boxTextMap[index[0]][index[1]] = this.boxTextMap[rowKey][colKey];
        this.boxTextMap[rowKey][colKey] = null;
        this.boxTextMap = JSON.parse(JSON.stringify(this.boxTextMap));
      }
      
      if (this.getDoneSuccess()) {
        this.endGame(false);
        let t = setTimeout(() => {
          alert(`游戏结束，完成时间： ${this.time / 100}秒`);
          clearTimeout(t);
        }, 300)
      }
    },
    getNullBoxIndex(rowKey, colKey, nullBoxDirectionKey) {
      const map = {
        0: [rowKey, colKey - 1],
        1: [rowKey, colKey + 1],
        2: [rowKey + 1, colKey],
        3: [rowKey - 1, colKey],
      }
      return map[nullBoxDirectionKey];
    },
    boxEditClick(rowKey, colKey) {
      if (!this.boxTextMap[rowKey][colKey] && this.doneList.length < this.difficulty * this.difficulty - 1) {
        const curNum = this.doneList.length + 1;
        this.boxTextMap[rowKey][colKey] = curNum;
        
        this.boxTextMap = JSON.parse(JSON.stringify(this.boxTextMap));

        this.doneList.push({
          index: [rowKey, colKey],
          value: curNum
        });
      }
      
    },
    redoClick() {
      if (this.doneList.length) {
        const last = this.doneList.pop();
        this.boxTextMap[last.index[0]][last.index[1]] = null;
        this.boxTextMap = JSON.parse(JSON.stringify(this.boxTextMap));
      }
    },
    confirmClick(isRandom) {
      if (isRandom || this.doneList.length === this.difficulty * this.difficulty - 1) {
        this.isEdit = 'playing';
        this.randomSettingDone = false;
        this.time = 0;
        this.timer = setInterval(() => {
          this.time += 10;
        }, 100)
      } else {
        alert('未完成编辑');
      }
    },
    getDoneSuccess() {
      return this.boxTextMap.every((row, rowKey) => {
        return row.every((col, colKey) => {
          return (rowKey + 1) * (colKey + 1) < this.difficulty * this.difficulty - 1 ?
            col === this.difficulty * rowKey + colKey + 1
            :
            true
        });
      });
    },
    endGame(back) {
      clearInterval(this.timer);
      this.timer = null;

      if (back) {
        this.time = 0;
        this.backToChooseDifficult();
      }
    },
  }
}
</script>

<style lang="less" scoped>
.all {
  width: 100%;
}
.container {
  position: relative;
  top: 50px;
  margin: 0 auto;
  width: 900px;
  height: 900px;
  border: 1px solid black;
}

.box {
  float: left;
  margin: 0px;
  padding: 0;
  // width: 200px;
  // height: 200px;
  font-size: 40px;
  font-weight: 800;
  text-align: center;
  // line-height: 200px;
  vertical-align: middle;
  border: 1px solid black;
  background-color: white;
}

.setting {
  position: relative;
  width: 200px;
  top: 100px;
  margin: 0 auto;

  div {
    position: relative;
    width: 200px;
    height: 100px;
    left: calc(50% - 100px);
    margin-bottom: 10px;
    font-size: 30px;
    text-align: center;
    line-height: 100px;
    border: 1px solid black;
    background-color: pink;
  }
}

.redo {
  position: relative;
  width: 200px;
  height: 100px;
  left: calc(50% - 100px);
  font-size: 30px;
  text-align: center;
  line-height: 100px;
  border: 1px solid black;
  background-color: pink;
}

.confirm {
  position: relative;
  width: 200px;
  height: 100px;
  top: 30px;
  left: calc(50% - 100px);
  font-size: 30px;
  text-align: center;
  line-height: 100px;
  border: 1px solid black;
  background-color: pink;
}

.bg-pink {
  background-color: pink;
}
</style>
