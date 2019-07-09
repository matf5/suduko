<template>
<div class="sudokus-page">
  <p class="hint">{{ errorHint }}</p>
  <div class="sudokus" id="sudokus">
    <div 
      class="sudokus-block"
      v-for="(block, i) in numArr"
      :key="i"
      id="sudokus-block">
        <div 
          class="sudokus-block-items"
          v-for="(item, j) in block"
          :class="{'sudokus-block-items__active': isActive[i][j]}"
          :key="j"
          @click="onItemClick(i,j)">
          <div
            class="sudokus-block-items__write"
            v-if="!ableArr[i][j]">
            {{ numArr[i][j] }}
          </div>
          <div
            class="sudokus-block-items__normal"
            v-else>
            {{ numArr[i][j] }}
          </div>
        </div>
    </div>
  </div>
  <div class="sudokus-option">
    <div class="sudokus-option-item" 
      v-for="i in 9" 
      :key="i"
      @click="onOptionClick(i)">{{ i + 1 }}
    </div>
    <div class="sudokus-option-item" @click="onClearItem()">X
    </div>
  </div>
  <button @click="onSubmit">提交</button>
  <button @click="onFresh">刷新</button>
</div>
</template>

<script>
// Use Vuex
const QUALITY_NUM = {
  hard: () => Math.floor(Math.random() + 5.5),
  medium: () => Math.floor(Math.random() + 4.5),
  easy: () => Math.floor(Math.random() + 3.5),
};

export default {
  data() {
    return {
      numArr: [],
      curTimes: 0,
      ableArr: [], // 代表不可修改的位置
      isActive: [],
      validateArr: [],
      errorHint: '',
      currentX: -1,
      currentY: -1,
      quality: 'hard',
    };
  },
  computed: {
  },
  mounted() {
    this.quality = this.$root.$mp.query.quality;
    this.init();
  },
  methods: {
    onFresh() {
      this.init();
    },
    init() {
      this.numArr = Array(9).fill([]).map(() => Array(9).fill(0));
      this.ableArr = Array(9).fill([]).map(() => Array(9).fill(true));
      this.isActive = Array(9).fill([]).map(() => Array(9).fill(false));
      this.generateSuduko();
      this.cutNum();
    },
    /**
     * 检查右部是否需要加粗
    */
    isItemRight(col) {
      return col % 3 === 2;
    },
    /**
     * 检查下部是否需要加粗
    */
    isItemBottom(row) {
      return row % 3 === 2;
    },
    /**
     * 提交
    */
    onSubmit() {
      const arr = Array(9).fill([]).map(() => Array(9).fill(0));
      for (let i = 0; i < 9; i += 1) {
        for (let j = 0; j < 9; j += 1) {
          if (typeof this.numArr[i][j] === 'string' && this.numArr[i][j].length === 0) {
            wx.showToast({
              title: '请补充完整',
              icon: 'none',
            });
            return;
          }
        }
      }
      for (let i = 0; i < 9; i += 1) {
        for (let j = 0; j < 9; j += 1) {
          if (typeof this.numArr[i][j] === 'string') {
            arr[i][j] = parseInt(this.numArr[i][j], 10);
          } else {
            arr[i][j] = this.numArr[i][j];
          }
        }
      }
      if (!this.validate(arr)) {
        wx.showToast({
          title: '存在错误，请检查',
          icon: 'none',
        });
      } else {
        wx.showToast({
          title: '恭喜您，验证成功',
        });
      }
    },
    /**
     * 是否正确的数字 检查横纵 九宫格内是否有重复的数字
    */
    isCorret(row, col, arr) {
      return this.checkRow(row, col, arr)
      && this.checkCol(row, col, arr) && this.checkNine(row, col, arr);
    },
    /**
     * 清空值
    */
    resetValuesToZeros() {
      this.numArr = Array(9).fill([]).map(() => Array(9).fill(0));
    },
    /**
     * 清空某一行的值
    */
    resetValuesInRowToZero(row) {
      for (let j = 0; j < 9; j += 1) {
        this.numArr[row][j] = 0;
      }
    },
    /**
     * 寻找候选数字
    */
    isCanditateNumFound(arr, row, col) {
      for (let i = 0; i < 9; i += 1) {
        this.numArr[row][col] = arr[i];
        if (this.isCorret(row, col, this.numArr)) {
          return true;
        }
      }
      return false;
    },
    /**
     * 选择输入某一个数字
    */
    onOptionClick(i) {
      // this.numArr[this.currentX][this.currentY] = i;
      this.$set(this.numArr[this.currentX], this.currentY, i);
    },
    /**
     * 选择某一个item
    */
    onItemClick(x, y) {
      if (this.ableArr[x][y]) {
        return;
      }
      this.currentX = x;
      this.currentY = y;
      this.isActive = Array(9).fill([]).map(() => Array(9).fill(false));
      for (let i = 0; i < 9; i += 1) {
        for (let j = 0; j < 9; j += 1) {
          if (i === this.currentX || j === this.currentY) {
            this.$set(this.isActive[i], j, true);
          }
        }
      }
    },
    /**
     * 清空当前item的值
    */
    onClearItem() {
      if (this.currentX === -1 || this.currentY === -1) {
        return;
      }
      this.$set(this.numArr[this.currentX], this.currentY, '');
    },
    /**
     * 生成数独
    */
    generateSuduko() {
      for (let i = 0; i < 9; i += 1) {
        if (i === 0) {
          this.curTimes = 0;
          this.numArr[i] = this.generateNumArr();
        } else {
          let tmpArr = this.generateNumArr();
          for (let j = 0; j < 9; j += 1) {
            if (this.curTimes < 220) {
              if (!this.isCanditateNumFound(tmpArr, i, j)) {
                this.resetValuesInRowToZero(i);
                i -= 1;
                j = 8;
                tmpArr = this.generateNumArr();
              }
            } else {
              i = -1;
              j = 8;
              this.resetValuesToZeros();
              this.curTimes = 0;
            }
          }
        }
      }
    },
    validate(arr) {
      for (let i = 0; i < 9; i += 1) {
        for (let j = 0; j < 9; j += 1) {
          if (!this.isCorret(i, j, arr)) {
            return false;
          }
        }
      }
      return true;
    },
    cutNum() {
      for (let i = 0; i < 9; i += 1) {
        const arr = this.generateRandomArr(QUALITY_NUM[this.quality]());
        arr.forEach((item) => {
          this.numArr[i][item] = '';
          this.ableArr[i][item] = false;
        });
      }
    },
    generateNum() {
      const ran = Math.random() * 8;
      return Math.floor(ran + 1);
    },
    generateNumArr() {
      this.curTimes += 1;
      const arr = [];
      for (let i = 1; i < 10; i += 1) {
        arr.push(i);
      }
      let num;
      for (let i = 0; i < 20; i += 1) {
        num = this.generateNum();
        const tmp = arr[0];
        arr[0] = arr[num];
        arr[num] = tmp;
      }
      return arr;
    },
    generateRandomArr(len) {
      const arr = this.generateNumArr();
      arr.forEach((item, index) => {
        arr[index] = item - 1;
      });
      return arr.slice(0, len);
    },
    checkRow(row, col, arr) {
      const curValue = arr[row][col];
      for (let j = 0; j < col; j += 1) {
        if (curValue === arr[row][j]) {
          return false;
        }
      }
      return true;
    },
    checkCol(row, col, arr) {
      const curValue = arr[row][col];
      for (let i = 0; i < row; i += 1) {
        if (curValue === arr[i][col]) {
          return false;
        }
      }
      return true;
    },
    checkNine(row, col, arr) {
      const rowStart = Math.floor(row / 3) * 3;
      const colStart = Math.floor(col / 3) * 3;
      for (let i = rowStart; i < row; i += 1) {
        const end = colStart + 3;
        for (let j = colStart; j < end; j += 1) {
          if (arr[i][j] === arr[row][col]) {
            return false;
          }
        }
      }
      return true;
    },
  },
};
</script>

<style lang="scss" scoped>
.sudokus-page {
.hint {
  text-align: center;
  color: red;
  height: 40px;
  line-height: 40px;
}
#sudokus :nth-of-type(3n) {
  border-bottom: 1px #000000 solid !important;
}
.sudokus {
  border-top: 1px #000000 solid;
  border-left: 1px #000000 solid;
  text-align: center;
  height: 100%;
  width: 100%;
  #sudokus-block :nth-of-type(3n) {
    border-bottom: 1px #ccc solid !important;
    border-right: 1px #000000 solid !important;
  }
  &-block {
    background: #ffffff;
    display: grid;
    grid-template-columns: repeat(9, 11%);
      &-items {
        background: #ffffff;
        border: 1px #ccc solid;
        height: 0;
        padding-bottom: 100%;
        div {
          margin-top: 5px;
        }
        &__active {
          background: #FFFFBB;
        }
        &__write {
          color: #3880E0;
        }
      }
    }
  }
  .sudokus-option {
    display: grid;
    grid-template-columns: repeat(10, 9%);
    grid-column-gap: 1%;
    &-item {
      text-align: center;
      height: 0;
      padding-bottom: 100%;
      background: #3880E0;
      color: white;
    }
  }
}
</style>
