<template>
  <div class="post-detail">
    <!-- 头部 -->
    <div class="header">
      <div class="left">
        <span class="iconfont iconjiantou2" @click="$router.back()"></span>
      </div>
      <div class="center">
        <span class="iconfont iconnew"></span>
      </div>
      <div class="right">
        <div
          class="followed"
          v-if="detail.has_follow"
          @click="unFollow(detail.user.id)"
        >
          已关注
        </div>
        <div class="follow" v-else @click="follow(detail.user.id)">关注</div>
      </div>
    </div>

    <!-- 文章内容 -->
    <div class="detail-content">
      <!-- 新闻标题 -->
      <div class="title">{{ detail.title }}</div>
      <!-- 新闻作者和时间 -->
      <div class="user">
        <span>{{ detail.user.nickname }}</span>
        <span>{{ detail.user.create_date | date }}</span>
      </div>
      <!-- 内容 -->
      <div class="content" v-html="detail.content"></div>

      <!--  点赞按钮 -->
      <div class="btns">
        <div
          class="like btn"
          :class="{ active: detail.has_like }"
          @click="like"
        >
          <span class="iconfont icondianzan"></span>
          <span>{{ detail.like_length || 0 }}</span>
        </div>
        <div class="share btn">
          <span class="iconfont iconweixin"></span>
          <span>微信</span>
        </div>
      </div>
    </div>

    <!-- 底部发表评论 -->
    <div class="footer">
      <!-- textarea 输入框 -->
      <div class="textarea" v-if="isShow">
        <textarea
          placeholder="回复"
          @blur="handleBlur"
          ref="textarea"
        ></textarea>
        <div class="send">发送</div>
      </div>
      <div class="input" v-else>
        <input type="text" placeholder="写跟帖" @focus="handleFocus" />
        <span class="iconfont iconpinglun-">
          <span v-show="detail.comment_length > 0">{{
            detail.comment_length
          }}</span>
        </span>
        <span
          class="iconfont iconshoucang"
          :class="{ active: detail.has_star }"
          @click="star"
        ></span>
        <span class="iconfont iconfenxiang"></span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // 数据
  data() {
    return {
      // 用于存放文章详情
      detail: {
        user: {}
      },
      isShow: false
    }
  },
  //   勾子函数 渲染页面
  async created() {
    this.getDetail()
  },
  //方法
  methods: {
    //拿文章的信息
    async getDetail() {
      const id = this.$route.params.id
      const res = await this.$axios.get(`/post/${id}`)
      const { statusCode, data } = res.data
      if (statusCode === 200) {
        this.detail = data
        console.log(this.detail)
      }
    },
    //关注
    async follow(id) {
      //关注功能
      // 思路1：先判断是否登录，如果登录了，就发送请求，如果没登录，让他跳转到登录页
      const token = localStorage.getItem('token')
      if (!token) {
        this.$router.push({
          name: 'login',
          params: { back: true }
        })
        this.$toast('请先登录')
        return
      }
      // 思路2：已经登录过的 直接发送请求进行关注
      const res = await this.$axios.get(`/user_follows/${id}`)
      const { statusCode, message } = res.data
      if (statusCode === 200) {
        this.$toast.success(message)
        this.getDetail()
      }
    },
    // 取消关注功能
    async unFollow(id) {
      //直接发请求取消关注
      const res = await this.$axios.get(`/user_unfollow/${id}`)
      const { statusCode, message } = res.data
      if (statusCode === 200) {
        this.$toast.success(message)
        this.getDetail()
      }
    },
    // 点赞功能
    async like() {
      //判断是否登录过
      const token = localStorage.getItem('token')
      if (!token) {
        this.$router.push({
          name: 'login',
          params: { back: true }
        })
        return this.$toast('请先登录')
      }
      //发送请求 去点赞
      const res = await this.$axios.get(`/post_like/${this.detail.id}`)
      const { statusCode, message } = res.data
      if (statusCode === 200) {
        this.$toast.success(message)
        this.getDetail()
      }
    },
    //收藏 与取消收藏
    async star() {
      //判断是否登录过
      const token = localStorage.getItem('token')
      if (!token) {
        this.$router.push({
          name: 'login',
          params: { back: true }
        })
        return this.$toast('请先登录')
      }
      const res = await this.$axios.get(`/post_star/${this.detail.id}`)
      const { statusCode, message } = res.data
      if (statusCode === 200) {
        this.$toast.success(message)
        this.getDetail()
      }
    },
    // input 获取焦点时 让textarea显示(切换input 和textarea 框)
    async handleFocus() {
      this.isShow = true
      //数据更新完 dom 不是马上更新  所以要用$nextTick 来拿到更新后的dom
      await this.$nextTick()
      this.$refs.textarea.focus()
    },
    // textarea失去焦点时 让input显示(切换input 和textarea 框)
    handleBlur() {
      this.isShow = false
    }
  } //方法
} //导出
</script>

<style lang="less" scoped>
// 头部
.header {
  display: flex;
  padding: 0 20px;
  height: 50px;
  line-height: 50px;
  border-bottom: 1px solid #ccc;
  justify-content: space-between;
  align-items: center;
  .center {
    flex: 1;
    padding-left: 5px;
    height: 50px;
    line-height: 50px;
    span {
      font-size: 40px;
    }
  }
  .right {
    //   已关注
    .followed {
      border: 1px solid #ccc;
      height: 30px;
      line-height: 30px;
      border-radius: 15px;
      padding: 0 15px;
      background-color: skyblue;
    }
    // 未关注
    .follow {
      height: 30px;
      line-height: 30px;
      border: 1px solid #ff0000;
      background-color: #ff0000;
      padding: 0 15px;
      border-radius: 15px;
      color: #fff;
    }
  }
} //头部

// 内容
.detail-content {
  padding: 0 20px;
  .title {
    font-weight: 700;
    font-size: 24px;
    padding: 20px 0;
  }
  .user {
    color: #999;
    font-size: 14px;
    margin-bottom: 20px;
    span {
      margin-right: 10px;
    }
  }
  .content {
    font-size: 14px;
    /deep/img {
      width: 100%;
      display: block;
    }
  }
  //点赞 分享
  .btns {
    display: flex;
    justify-content: space-around;
    padding: 20px 0;
    .btn {
      width: 80px;
      height: 3opx;
      line-height: 30px;
      border: 1px solid #000;
      text-align: center;
      border-radius: 15px;
      padding: 0 15px;
      font-size: 14px;
      span:first-child {
        margin-right: 5px;
      }
      &.active {
        border-color: red;
        color: red;
      }
    }
  }
  .share {
    color: #00c800;
  }
} //内容

// 底部评论
.footer {
  .input {
    display: flex;
    height: 50px;
    padding: 0 20px;
    justify-content: space-between;
    align-items: center;
    border-top: 1px solid #ccc;
    input {
      height: 30px;
      width: 180px;
      border-radius: 15px;
      border: none;
      outline: none;
      background-color: #ddd;
      padding-left: 20px;
    }
    .iconfont {
      font-size: 22px;
    }
    .iconpinglun- {
      position: relative;
      span {
        position: absolute;
        top: -7px;
        left: 9px;
        background-color: red;
        font-size: 12px;
        border-radius: 5px;
        padding: 0 5px;
        color: #fff;
      }
    }
    .active {
      color: red;
    }
  }
  //textarea 输入框
  .textarea {
    display: flex;
    height: 100px;
    padding: 20px;
    align-items: flex-end;
    border-top: 1px solid #ccc;
    textarea {
      height: 80px;
      background-color: #ddd;
      flex: 1;
      border: none;
      outline: none;
      border-radius: 10px;
      padding: 10px;
    }
    .send {
      height: 30px;
      line-height: 30px;
      background-color: red;
      padding: 0 15px;
      border-radius: 15px;
      color: #fff;
      margin: 0 10px;
    }
  }
}
</style>
