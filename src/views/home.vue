<style lang="less" scoped>
.box {
  margin:auto;
  position: relative;
  border: 6px solid #999;
  border-radius:4px;

  div {
    position: absolute;
    background-image: url("../assets/chai.jpg");
  }
}
</style>

<template>
  <div>
    <div class="box" id="puzzle_box" 
      :style="{width: box_width +'px', height: box_height+'px'}"
      @click="handleClick">
      <div v-for="(item, index) in position_arr" :key="'puzzle'+index"
            :style="{width: width+'px',
                    height: height+'px',
                    top: item.top + 'px',
                    left: item.left + 'px',
                    'background-position-x': item.background_x +'px',
                    'background-position-y': item.background_y +'px'}"
            :data-ori_index="item.ori_index"
            :data-cur_index="item.cur_index"
            v-show="!item.is_blank"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "home",
  data() {
    return {
      box_width: 300,
      box_height: 300,

      width: 0,
      height: 0,

      row: 3,
      column: 3,

      position_arr:[],        //每块拼图的坐标
      blank_puzzle_index: 0,    //空白块所在的位置，从0开始计算
    };
  },
  created() {
    this.width = this.box_width / this.row;
    this.height = this.box_height / this.column;
  
    // 循环行数 列数
    let index = 0;  //标记当前属于第几块拼图，从0开始
    for (let r = 0; r < this.row; r++) {
      for (let c = 0; c < this.column; c++) {
        this.position_arr.push({
          background_y: - (r * this.height), //背景图片偏移，不会变化
          background_x: - (c * this.width),
          ori_index: index,   //应所在的位置,不会变化
          cur_index: index,   //当前所在位置,会变化
          cur_r: r,           //当前所在的行，会变化
          cur_c: c,           //当前所在的列，会变化
        });
        index ++;
      }
    }
    //空白块位置在最后一个
    this.blank_puzzle_index = this.row * this.column - 1;
 
    //打乱顺序
    this.position_arr = this.random_puzzle_sort(this.position_arr);

  },
  methods:{
    handleClick(e){
      let cur = e.target
      //当前拼图排序
      let cur_index = cur.getAttribute('data-cur_index');
      //如果不是点击在拼图上
      if (!cur_index) {
        return;
      }
      //如果点击空白块
      if (cur_index == this.blank_puzzle_index) {
        return;
      }

      let cur_puzzle = this.position_arr[cur_index]
      let blank_puzzle = this.position_arr[this.blank_puzzle_index]


      console.log({cur_puzzle},{blank_puzzle});

      //判断和空白块是否相邻，且是同行或同列，否则不能对换
      if (!((Math.abs(cur_index - this.blank_puzzle_index) == 1 || Math.abs(cur_index - this.blank_puzzle_index) == 3)
        && (cur_puzzle.cur_r == blank_puzzle.cur_r || cur_puzzle.cur_c == blank_puzzle.cur_c))) {
        return;
      }

      //和空白拼图对换位置
      //先计算出点击的这一块，改变后的位置
      cur_puzzle = this.setPosition(cur_puzzle, this.blank_puzzle_index);

      //再计算空白块的位置
      blank_puzzle = this.setPosition(blank_puzzle, cur_index);

      this.position_arr[this.blank_puzzle_index] = cur_puzzle
      this.position_arr[cur_index] = blank_puzzle


      this.blank_puzzle_index = cur_index
      // console.log('position_arr', this.position_arr);

      if (this.checkSuccess()) {
        console.log('success!!!');
      }
    
    },
    random_puzzle_sort(arr) {
      //先取出最后一块，不做随机
      let blank_puzzle = arr.splice(arr.length-1, 1)[0];

      let result = []
      //其余8块随机顺序
      let length = arr.length;
      for (let i = 0; i < length; i++) {
        let random = Math.floor(Math.random() * arr.length) 
        let puzzle = arr.splice(random, 1)[0]; 

        let puzzle_index = result.length; //随机后所在的位置
        //根据随机后所在的位置，计算div定位
        puzzle = this.setPosition(puzzle, puzzle_index)
        result.push(puzzle);
      }

      //计算空白块的位置
      blank_puzzle = this.setPosition(blank_puzzle, result.length)
      blank_puzzle.is_blank = true;
      result.push(blank_puzzle)
      return result
    },
    //计算定位 打乱顺序或点击更换位置时需要
    setPosition(obj, index) {
      obj.cur_r = Math.floor(index / this.row);
      obj.cur_c = index % this.column;
      obj.top = obj.cur_r * this.height
      obj.left = obj.cur_c * this.width
      obj.cur_index = index
      return obj
    },
    checkSuccess(){
      //校验位置是否回归初始
      let flag = true
      for (let i = 0; i < this.position_arr.length; i++) {
        let cur = this.position_arr[i]
        if (cur.ori_index != cur.cur_index) {
          flag = false
          break;
        }
      }
      return flag;
    }

  }
};
</script>


