<template>
  <view>
    <!-- 用户信息 -->
    <view class="user_info">
      <view class="user_icon">
        <image :src="imgDetail.user.avatar" />
      </view>
      <view class="user_desc">
        <view class="user_name">{{ imgDetail.user.name }}</view>
        <view class="user_time">{{ imgDetail.cnTime }}</view>
      </view>
    </view>

    <!-- 高清大图 -->
    <view class="high_img">
      <swiper-action @swiperAction="handleSwiperAction">
        <image :src="imgDetail.thumb" mode="widthFix" />
      </swiper-action>
    </view>

    <!-- 点赞收藏 -->
    <view class="user_operation">
      <view class="user_rank">
        <text class="iconfont icon-dianzan">{{ imgDetail.rank }}</text>
      </view>
      <view class="user_collect">
        <text class="iconfont icon-shoucang">收藏</text>
      </view>
    </view>

    <!-- 专辑区域 -->
    <!-- 最热评论 -->
    <view v-if="hot.length" class="comment hot">
      <view class="comment_title">
        <text class="iconfont icon-hot"></text>
        <text class="comment_text">最热评论</text>
      </view>
      <view class="comment_list">
        <view class="comment_item" v-for="item in hot" :key="item.id">
          <!-- 用户信息 -->
          <view class="comment_user">
            <!-- 用户头像 -->
            <view class="user_icon"
              ><image :src="item.user.avatar" mode="widthFix"
            /></view>
            <!-- 用户名称 -->
            <view class="user_name">
              <view class="user_nickname">{{ item.user.name }}</view>
              <view class="user_time">{{ item.cntime }}</view>
            </view>
            <!-- 用户徽章 -->
            <view class="user_badge">
              <image
                v-for="item2 in item.user.title"
                :key="item2.icon"
                :src="item2.icon"
              />
            </view>
          </view>
          <!-- 评论数据 -->
          <view class="comment_desc">
            <view class="comment_content">{{ item.content }}</view>
            <view class="comment_like">
              <text class="iconfont icon-dianzan">{{ item.size }}</text>
            </view>
          </view>
        </view>
      </view>
    </view>

    <!-- 最新评论 -->
    <view v-if="comment.length" class="comment new">
      <view class="comment_title">
        <text class="iconfont icon-pinglun"></text>
        <text class="comment_text">最新评论</text>
      </view>
      <view class="comment_list">
        <view class="comment_item" v-for="item in comment" :key="item.id">
          <!-- 用户信息 -->
          <view class="comment_user">
            <!-- 用户头像 -->
            <view class="user_icon"
              ><image :src="item.user.avatar" mode="widthFix"
            /></view>
            <!-- 用户名称 -->
            <view class="user_name">
              <view class="user_nickname">{{ item.user.name }}</view>
              <view class="user_time">{{ item.cntime }}</view>
            </view>
            <!-- 用户徽章 -->
            <view class="user_badge">
              <image
                v-for="item2 in item.user.title"
                :key="item2.icon"
                :src="item2.icon"
              />
            </view>
          </view>
          <!-- 评论数据 -->
          <view class="comment_desc">
            <view class="comment_content">{{ item.content }}</view>
            <view class="comment_like">
              <text class="iconfont icon-dianzan">{{ item.size }}</text>
            </view>
          </view>
        </view>
      </view>
    </view>

    <!-- 下载 -->
    <view class="download">
      <view class="download_btn" @click="handleDownload">下载图片</view>
    </view>
  </view>
</template>

<script>
import moment from "moment";
import swiperAction from "@/components/swiperAction";
//设置语言为中文
moment.locale("zh-cn");
export default {
  components: {
    swiperAction
  },
  data() {
    return {
      //图片信息对象 包含着用户信息
      imgDetail: {},
      //专辑数据
      // album:[],
      //最热评论
      hot: [],
      //最新评论
      comment: [],
      //图片的索引
      imgIndex: 0
    };
  },
  onLoad(args) {
    // console.log(getApp().globalData);
    const { imgIndex } = getApp().globalData;
    this.imgIndex = imgIndex;
    this.getData();
  },
  methods: {
    //给当前页面赋值
    getData() {
      const { imgList } = getApp().globalData;
      this.imgDetail = imgList[this.imgIndex];
      //利用momentjs转换时间戳 => 几年前
      this.imgDetail.cnTime = moment(this.imgDetail.atime * 1000).fromNow();

      //获取图片详情的id
      // this.imgDetail.id;
      this.getComments(this.imgDetail.id);
    },
    //发送请求获取数据
    getComments(id) {
      this.request({
        url: `http://157.122.54.189:9088/image/v2/wallpaper/wallpaper/${id}/comment`
      }).then(result => {
        // console.log(result);
        // this.album = result.res.album;

        //给hot数组中的对象添加一个事件属性 xxx月前
        result.res.hot.forEach(
          v => (v.cnTime = moment(v.atime * 1000).fromNow())
        );
        result.res.comment.forEach(
          v => (v.cnTime = moment(v.atime * 1000).fromNow())
        );

        this.hot = result.res.hot;
        this.comment = result.res.comment;
      });
    },
    //滑动事件
    handleSwiperAction(e) {
      // console.log(e);
      /* 
        用户左滑， imgIndex++
        用户右滑， imgIndex--
        判断数组是否越界问题！！
          左滑 e.direction==="left"&& this.imgIndex<imgList.length-1
          右滑 e.direction==="right"&& this.imgIndex>0
      */
      const { imgList } = getApp().globalData;
      if (e.direction === "left" && this.imgIndex < imgList.length - 1) {
        //可以进行左滑
        this.imgIndex++;
        this.getData();
      } else if (e.direction === "right" && this.imgIndex > 0) {
        //可以进行右滑
        this.imgIndex--;
        this.getData();
      } else {
        uni.showToast({
          title: "没有数据了",
          icon: "none"
        });
      }
    },
    //图片下载
    async handleDownload() {
      // 下载中
      await uni.showLoading({
        title: '下载中...',
        mask: true
      })

      //1.将远程文件下载到小程序的内存中 tempFilePath
      const result = await uni.downloadFile({url: this.imgDetail.img});
      const {tempFilePath} = result[1];

      //2. 将小程序内存中的临时文件下载到本地上
      const result2 = await uni.saveImageToPhotosAlbum({filePath:tempFilePath});

      //3. 提示用户下载成功
      // console.log(result2);
      // console.log('下载成功');
      uni.hideLoading();
      await uni.showToast({
        title: '下载成功',
        icon: 'success'
      })
    }
  }
};
</script>

<style lang="scss" scoped>
.user_info {
  display: flex;
  padding: 20rpx;
  .user_icon {
    padding: 0 20rpx;
    image {
      width: 88rpx;
      height: 88rpx;
      border-radius: 50%;
    }
  }
  .user_name {
    color: #000;
    font-weight: 600;
  }
  .user_time {
    color: #ccc;
    font-size: 24rpx;
    padding: 10rpx 0;
  }
}

//高清大图
.high_img {
  image {
    border-bottom: 1px solid #eee;
  }
}

.user_operation {
  display: flex;
  height: 80rpx;
  border-bottom: 4px solid #eee;
  .user_rank {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .user_collect {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
  }
}

//最热评论
.comment {
  .comment_title {
    padding: 15rpx;
    .iconfont {
      color: red;
      font-size: 40rpx;
    }
    .comment_text {
      font-weight: 600;
      font-size: 28rpx;
      color: #666;
      margin-left: 10rpx;
    }
  }
  .comment_list {
    .comment_item {
      border-bottom: 8px solid #eee;
      //用户信息
      .comment_user {
        display: flex;
        padding: 20rpx 0;
        .user_icon {
          width: 15%;
          display: flex;
          justify-content: center;
          align-items: center;
          image {
            width: 90%;
          }
        }

        .user_name {
          flex: 1;
          .user_nickname {
            color: #777;
          }
          .user_time {
            color: #ccc;
            font-size: 24rpx;
            padding: 5rpx 0;
          }
        }

        .user_badge {
          image {
            width: 40rpx;
            height: 40rpx;
            display: inline-block;
          }
        }
      }
      //评论信息
      .comment_desc {
        display: flex;
        padding: 30rpx 0;
        .comment_content {
          flex: 1;
          padding-left: 15%;
          color: #000;
        }

        .comment_like {
          text-align: right;
        }
      }
    }
  }
}

//最新评论
.new {
  .icon-pinglun {
    color: aqua !important;
  }
}

//图片下载
.download{
  height: 120rpx;
  display: flex;
  justify-content: center;
  align-items: center;
  .download_btn{
    width: 90%;
    height: 80%;
    background-color: $color;
    color: #fff;
    font-size: 50rpx;
    font-weight: 600;
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
</style>