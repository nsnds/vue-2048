<template>
  <div class="wrapper">
    <div class="head">
      <h1 class="title">2048</h1>
      <div class="score">
        <div>
          <span>SCORE</span>
          <span class="num">{{score}}</span>
        </div>
        <div>
          <span>BEST</span>
          <span class="num">{{bestScore}}</span>
        </div>
      </div>
    </div>
    <div class="btn btn-mg" @click="newGame">新游戏</div>
    <div>
      <div class="over" v-if="over">
        <p>Game over!</p>
        <div class="btn" @click="newGame">Try again</div>
      </div>
      <div class="box">
        <div class="row" v-for="(row, i) in list" :key="i">
          <div class="col"
               :class="`n-${col}`"
               v-for="(col, ii) in row"
               :key="ii"
          >{{col}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Index',
  data () {
    return {
      score: 0, // 当前分数
      bestScore: localStorage.getItem('bestScore'), // 最高分数
      over: false, // 是否游戏结束
      list: [], // 格子数据
      size: 4, // 格子数 n*n
      pr: 0.9,
      intiNum: [2, 4],
      direction: [
        { x: 0, y: -1 }, // 上
        { x: 0, y: 1 }, // 下
        { x: -1, y: 0 }, // 左
        { x: 1, y: 0 } // 右
      ]
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      this.newGame()
      document.addEventListener('keyup', this.keyDown)
    },
    // 开始新游戏
    newGame () {
      this.score = 0
      this.over = false
      // 生成[[...], [...], [...], [...]]数据结构
      this.list = Array.from(Array(this.size)).map(() => Array(this.size).fill(undefined))
      this.setRandom()
    },
    // 插入一个非空格子
    setRandom () {
      if (this.hasAvailableCells()) {
        // 获取一个随机空格子坐标
        let [x, y] = this.randomAvailableCells()
        // 格子赋值
        this.list[x][y] = this.randomValue()
      }
    },
    // 获取数值
    randomValue () {
      let value = Math.random() < this.pr ? this.intiNum[0] : this.intiNum[1]
      return value
    },
    // 获取一个随机空格子坐标
    randomAvailableCells () {
      let cells = this.availableCells()
      if (cells.length) return cells[Math.floor(Math.random() * cells.length)]
    },
    // 获取所有空格子坐标
    availableCells () {
      let i
      let j
      let cells = []
      let length = this.size
      let _list = this.list

      for (i = 0; i < length; i++) {
        for (j = 0; j < length; j++) {
          if (!_list[i][j]) {
            cells.push([i, j])
          }
        }
      }
      return cells
    },
    // 是否存在空格子
    hasAvailableCells () {
      return !!this.availableCells().length
    },
    // 是否存在可合并的格子
    hasMergedCells () {
      let _size = this.size
      for (let i = 0; i < _size; i++) {
        for (let j = 0; j < _size; j++) {
          let cell = this.list[i][j]
          if (cell) {
            for (let dir = 0; dir < 4; dir++) {
              let vector = this.direction[dir]
              if (this.withinBounds(i + vector.x, j + vector.y)) {
                let other = this.list[i + vector.x][j + vector.y]
                if (other && other === cell) return true
              }
            }
          }
        }
      }
      return false
    },
    // 是否还在格子区域中
    withinBounds (x, y) {
      let _size = this.size
      return x > 0 && y > 0 && x < _size && y < _size
    },
    // 游戏是否继续
    isAvailable () {
      return this.hasAvailableCells() || this.hasMergedCells()
    },
    // 监听键盘方向键事件
    keyDown (e) {
      switch (e.keyCode) {
        // 上
        case 38:
          this.move(1)
          break
        // 下
        case 40:
          this.move(3)
          break
        // 左
        case 37:
          this.move(0)
          break
        // 右
        case 39:
          this.move(2)
          break
      }
      this.setRandom()
    },
    // 移动算法, i=旋转次数
    move (i) {
      let arr = this.rotate(Array.from(this.list), i).map((item) => {
        return this.moveLeft(item)
      })
      this.list = this.rotate(arr, 4 - i)
      this.setLocalstorage()
      if (!this.isAvailable()) {
        this.over = true
      }
    },
    // 单行左移
    moveLeft (list) {
      let _list = [] // 当前行非空格子
      let _size = this.size

      // 获取当前行非空格子数组
      for (let i = 0; i < _size; i++) {
        if (list[i]) {
          _list.push({
            x: i,
            merged: false,
            value: list[i]
          })
        }
      }

      _list.forEach(item => {
        let farthest = this.farthestPosition(list, item)
        let next = list[farthest - 1] // 最远空格子的左一格子

        if (next && next === item.value && !_list[farthest - 1].merged) {
          // 合并: 左一存在 & 值与当前格子值相同 & 左一未合并过
          list[farthest - 1] = next * 2
          list[item.x] = undefined
          item = { x: farthest - 1, merged: true, value: next }
          this.score += next * 2
        } else {
          if (farthest !== item.x) {
            // 当前格子不是最远空格子进行移动
            list[farthest] = item.value
            list[item.x] = undefined
            item.x = farthest
          }
        }
      })
      return list
    },
    // 逆时针旋转
    rotate (arr, n) {
      n = n % 4
      if (n === 0) return arr

      let i
      let j
      let _size = this.size
      let tmp = Array.from(Array(_size)).map(() => Array(_size).fill(undefined))
      /* 旋转一次
       *
       * (0, 0), (0, 1), (0, 2), (0, 3)
       * (1, 0), (1, 1), (1, 2), (1, 3)
       * (2, 0), (2, 1), (2, 2), (2, 3)
       * (3, 0), (3, 1), (3, 2), (3, 3)
       *
       * (3, 0), (2, 0), (1, 0), (0, 0)
       * (3, 1), (2, 1), (1, 1), (0, 1)
       * (3, 2), (2, 2), (1, 2), (0, 2)
       * (3, 3), (2, 3), (1, 3), (0, 3)
       */
      for (i = 0; i < _size; i++) {
        for (j = 0; j < _size; j++) {
          tmp[_size - 1 - i][j] = arr[j][i]
        }
      }
      if (n > 1) tmp = this.rotate(tmp, n - 1)
      return tmp
    },
    // 左边最远空格的位置
    farthestPosition (list, cell) {
      let farthest = cell.x
      while (farthest > 0 && !list[farthest - 1]) {
        farthest = farthest - 1
      }
      return farthest
    },
    setLocalstorage () {
      let score = localStorage.getItem('bestScore')
      if (score) {
        if (this.score > score) {
          localStorage.setItem('bestScore', this.score)
          this.bestScore = this.score
        }
      } else {
        localStorage.setItem('bestScore', this.score)
        this.bestScore = this.score
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
.wrapper {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  .head {
    width: 400px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #776e65;
    .title {
        font-size: 60px;
    }
    .score {
      display: flex;
      justify-content: space-between;
      height: 80px; // line-height: 60px;
      div {
          // width: 100px;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          padding-left: 5px;
          padding-right: 5px;
          border-radius: 5px;
          background: #bbada0;
          .num {
              font-size: 25px;
              font-weight: bold;
              color: #ffffff;
          }
          &:last-child {
              margin-left: 5px;
          }
      }
    }
  }
  .over {
      position: absolute;
      width: 400px;
      height: 400px;
      background: rgba(238, 228, 218, 0.73);
      z-index: 1000;
      border-radius: 5px;
      text-align: center;
      color: #8f7a66;
      p {
          font-size: 60px;
          font-weight: bold;
          height: 60px;
          line-height: 60px;
      }
  }
  .btn {
      display: inline-block;
      padding: 0 20px;
      height: 40px;
      line-height: 40px;
      border-radius: 5px;
      cursor: pointer;
      text-align: center;
      color: #f9f6f2;
      background: #8f7a66;
      &.btn-mg {
          margin-bottom: 10px;
      }
  }
  .box {
      width: 400px;
      height: 400px;
      padding: 15px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      box-sizing: border-box;
      border-radius: 5px;
      background: #bbada0;
      .row {
          width: 100%;
          height: 23%;
          display: flex;
          flex-direction: row;
          justify-content: space-between;
          .col {
              width: 23%;
              height: 100%;
              display: flex;
              align-items: center;
              justify-content: center;
              font-size: 24px;
              border-radius: 3px;
              background: #cec1b3;
              &.n-2 {
                  background: #f8f3e8;
              }
              &.n-4 {
                  background: #ede0c8;
              }
              &.n-8 {
                  background: #f26179;
              }
              &.n-16 {
                  background: #f59563;
              }
              &.n-32 {
                  background: #f67c5f;
              }
              &.n-64 {
                  background: #f65e36;
              }
              &.n-128 {
                  background: #edcf72;
              }
              &.n-256 {
                  background: #edcc61;
              }
              &.n-512 {
                  background: #9c0;
              }
              &.n-1024 {
                  background: #3365a5;
              }
              &.n-2048 {
                  background: #09c;
              }
              &.n-4096 {
                  background: #a6bc;
              }
              &.n-8192 {
                  background: #93c;
              }
          }
      }
  }
}
</style>
