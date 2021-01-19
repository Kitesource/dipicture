<template>
  <scroll-view
    v-if="recommends.length > 0"
    class="recommend_view"
    scroll-y
    @scrolltolower="handleToLower"
  >
    <!-- 推荐区域 -->
    <view class="recommend_wrap">
      <navigator
        class="recommend_item"
        v-for="item in recommends"
        :key="item.id"
        :url="`/pages/album/index?id=${item.target}`"
      >
        <image mode="widthFix" :src="item.thumb"></image>
      </navigator>
    </view>

    <!-- 月份 -->
    <view class="months_wrap">
      <view class="months_title">
        <view class="months_title_info">
          <view class="months_info">
            <text> {{ months.DD }}/ </text>
            {{ months.MM }} 月
          </view>
          <view class="months_text">{{ months.title }}</view>
        </view>
        <view class="months_title_more">更多 > </view>
      </view>
      <view class="months_content">
        <view
          class="months_item"
          v-for="(item, index) in months.items"
          :key="item.id"
        >
          <go-detail :list="months.items" :index="index">
            <image
              mode="aspectFill"
              :src="item.thumb + item.rule.replace('$<Height>', 360)"
            ></image>
          </go-detail>
        </view>
      </view>
    </view>

    <!-- 热门 -->
    <view class="hots_wrap">
      <view class="hots_title">
        <text>热门</text>
      </view>
      <view class="hots_content">
        <view class="hots_item" 
        v-for="(item,index) in hots" 
        :key="item.id">
          <goDetail :list="hots" :index="index">
            <image :src="item.thumb" mode="widthFix" />
          </goDetail>
        </view>
      </view>
    </view>
  </scroll-view>
</template>

<script>
import moment from "moment";
import goDetail from "@/components/goDetail";
export default {
  components: {
    goDetail
  },
  data() {
    return {
      //推荐列表
      recommends: [],
      //月份
      months: {},
      //热门
      hots: [],
      //请求的参数
      params: {
        //要获取几条数据
        limit: 30,
        //关键字
        order: "hot",
        //要跳过几条
        skip: 0
      },
      //表示是否还有下一页
      hasMore: true
    };
  },
  //组件挂载完成时触发
  mounted() {
    this.getList();
  },
  methods: {
    //获取接口数据
    getList() {
      this.request({
        url: "http://157.122.54.189:9088/image/v3/homepage/vertical",
        data: this.params
      }).then(result => {
        // console.log(result);

        //判断还有没有下一页数据
        if (result.res.vertical.length === 0) {
          this.hasMore = false;
          //弹窗提示
          uni.showToast({
            title: "没有数据了",
            icon: "none",
            mask: true
          });
          return;
        }

        if (this.recommends.length === 0) {
          //第一次发送请求
          //推荐模块
          this.recommends = result.res.homepage[1].items;
          //月份模块
          this.months = result.res.homepage[2];
          // 将时间戳 改成 18号/月 moment.js
          this.months.MM = moment(this.months.stime).format("MM");
          this.months.DD = moment(this.months.stime).format("DD");
        }

        //热门数据列表
        //数组拼接 es6
        this.hots = [...this.hots, ...result.res.vertical];
      });
    },

    //滚动条触底事件
    handleToLower() {
      // console.log("111");
      /* 
        1 修改参数 skip += limit;
        2 重新放请求 getList() 
        3 请求回来的 hots 数据叠加
      */
      if (this.hasMore) {
        this.params.skip += this.params.limit;
        this.getList();
      } else {
        //弹窗提示
        uni.showToast({
          title: "没有数据了",
          icon: "none",
          mask: true
        });
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.recommend_view {
  //height = 屏幕高度 - 头部标题高度
  height: calc(100vh - 36px);
}

.recommend_wrap {
  display: flex;
  flex-wrap: wrap;
  .recommend_item {
    width: 50%;
    border: 5rpx solid #fff;
  }
}

//月份
.months_wrap {
  .months_title {
    display: flex;
    justify-content: space-between;
    padding: 20rpx;
    .months_title_info {
      color: $color;
      font-size: 30rpx;
      font-weight: 600;
      display: flex;
      .months_info {
        text {
          font-size: 36rpx;
        }
      }

      .months_text {
        font-size: 34rpx;
        color: #666;
        margin-left: 30rpx;
      }
    }

    .months_title_more {
      font-size: 24rpx;
      color: $color;
    }
  }

  .months_content {
    display: flex;
    flex-wrap: wrap;
    .months_item {
      width: 33.33%;
      border: 5rpx solid #fff;
    }
  }
}

.hots_wrap {
  .hots_title {
    padding: 20rpx;
    text {
      border-left: 5px solid $color;
      padding-left: 20rpx;
      font-size: 34rpx;
      font-weight: 600;
    }
  }

  .hots_content {
    display: flex;
    flex-wrap: wrap;
    .hots_item {
      width: 33.33%;
      border: 5rpx solid #fff;
    }
  }
}
</style>