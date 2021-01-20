/* 
   1实现插槽功能 
   2向父组件传递滑动的方向 
 */
<template>
  <view 
  @touchstart="handleTouchStart"
  @touchend="handleTouchEnd">
    <slot></slot>
  </view>
</template>

<script>
export default {
  data() {
    return {
      //按下时间
      startTime:0,
      //按下坐标
      startX:0,
      startY:0,
    }
  },
  methods: {
    //按下屏幕
    handleTouchStart(event) {
      // console.log('手指按下屏幕');
      // console.log("按下：" + event.changedTouches[0].clientX);
      // console.log("按下：" + event.changedTouches[0].clientY);
      this.startTime = Date.now();
      this.startX = event.changedTouches[0].clientX;
      this.startY = event.changedTouches[0].clientY;
    },

    //离开屏幕
    handleTouchEnd(event) {
      // console.log('手指离开屏幕');
      // console.log("离开："+event.changedTouches[0].clientX);
      // console.log("离开："+event.changedTouches[0].clientY);
      const endTime = Date.now();
      const endX = event.changedTouches[0].clientX;
      const endY = event.changedTouches[0].clientY;

      //判断按下的时长
      if(endTime - this.startTime >2000) {
        return;
      }

      //滑动的方向
      let direction = "";

      //先判断滑动的距离,若合法再判断滑动方向 注意：加上绝对值
      if(Math.abs(endX - this.startX) > 10) {
        //滑动的方向
        direction = endX - this.startX > 0 ? "right" : "left";
      }else {
        return;
      }

      //用户做了合法的滑动操作
      // console.log(direction);
      this.$emit("swiperAction", {direction});
    }
  }
}
</script>

<style>

</style>