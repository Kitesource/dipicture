<template>
  <view>
    <!-- 专辑背景 -->
    <view class="album_bg">
      <image :src="album.cover" mode="widthFix" />
      <view class="album_info">
        <view class="album_name">{{ album.name }}</view>
        <view class="album_btn">关注专辑</view>
      </view>
    </view>

    <!-- 专辑作者 -->
    <view class="album_author">
      <view class="album_author_info">
        <image :src="album.user.avatar" mode="widthFix" />
        <view class="album_author_name">{{ album.user.name }}</view>
      </view>
      <view class="album_author_desc">
        <text>{{ album.desc }}</text>
      </view>
    </view>

    <!-- 列表 -->
    <view class="album_list">
      <view class="album_item" v-for="item in wallpaper" :key="item.id">
        <image
          :src="item.thumb + item.rule.replace('$<Height>', 360)"
          mode="widthFix"
        />
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      params: {
        limit: 30,
        order: "new",
        skip: 0,
        // “1” 表示第一次请求  “0”表示不是第一次请求
        first: 1
      },
      id: -1,
      album: {},
      wallpaper: [],
      hasMore: true
    };
  },
  onLoad(args) {
    // console.log(args);
    this.id = args.id;
    // this.id = "5e5cf679e7bce739db1281e4";
    this.getList();
  },
  methods: {
    getList() {
      this.request({
        url: `http://157.122.54.189:9088/image/v1/wallpaper/album/${this.id}/wallpaper`,
        data: this.params
      }).then(result => {
        console.log(result);
        // console.log(JSON.stringify(result.res.album.desc));
        if (Object.keys(this.album).length === 0) {
          //album是空对象
          this.album = result.res.album;
        }
        if (result.res.wallpaper.length === 0) {
          this.hasMore = false;
          return;
        }
        this.wallpaper = [...this.wallpaper, ...result.res.wallpaper];
      });
    }
  },
  //页面触底事件
  onReachBottom() {
    // console.log('触底了喔');
    if (this.hasMore) {
      this.params.first = 0;
      this.params.skip += this.params.limit;
      this.getList();
    } else {
      uni.showToast({
        title: "没有更多数据了",
        icon: "none"
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.album_bg {
  position: relative;
  .album_info {
    position: absolute;
    width: 100%;
    left: 0;
    bottom: 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 80rpx;
    padding: 0 20rpx;
    color: #fff;
    .album_name {
      font-size: 40rpx;
    }

    .album_btn {
      background-color: $color;
      width: 150rpx;
      height: 60rpx;
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 10rpx;
    }
  }
}

.album_author {
  padding: 10rpx;
  .album_author_info {
    padding: 10rpx 0;
    display: flex;
    image {
      width: 50rpx;
    }

    .album_author_name {
      color: #000;
      margin-left: 15rpx;
    }
  }

  .album_author_desc {
    font-size: 28rpx;
  }
}

.album_list {
  padding-top: 20rpx;
  display: flex;
  flex-wrap: wrap;
  .album_item {
    width: 33.33%;
    border: 3rpx solid #fff;
  }
}
</style>