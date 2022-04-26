<template>
  <div class="comment-bg">
    <div v-clickoutside="hideReplyBtn" @click="inputFocus" class="my-reply">
      <el-avatar class="header-img" :size="40" :src="myHeader"></el-avatar>
      <div class="reply-info">
        <div
            tabindex="0"
            contenteditable="true"
            id="replyInput"
            spellcheck="false"
            placeholder="输入评论..."
            class="reply-input"
            @focus="showReplyBtn"
            @input="onDivInput($event)"
        >
        </div>
      </div>
      <div class="reply-btn-box" v-show="btnShow">
        <el-button class="reply-btn" size="medium" @click="sendComment" type="primary">发表评论</el-button>
      </div>
    </div>
<!--    el-scrollbar style="height: 100%"-->
    <el-scrollbar style="height: 100%" >
      <div v-for="(item,i) in comments" :key="i" class="author-title reply-father">
        <el-avatar class="header-img" :size="40" :src="item.headImg"></el-avatar>
        <div class="author-info">
          <span class="author-commentUser">{{item.commentUser}}</span>
          <span class="author-time">{{item.updatedDate}}</span>
        </div>
        <div class="icon-btn">
                <span class="reply-span" @click="showReplyInput(i,item.commentUser,item.commentId)">
                    回复
                </span>
        </div>
        <div class="talk-box">
          <p>
            <span class="reply">{{item.commentContent}}</span>
          </p>
        </div>
        <div class="reply-box">
          <div v-for="(reply,j) in item.reply" :key="j" class="author-title">
            <div class="author-info">
              <span class="author-commentUser">{{reply.commentUser}}</span>
              <span class="author-time">{{reply.updatedDate}}</span>
            </div>
            <div class="icon-btn">
                        <span class="reply-span" @click="showReplyInput(i,reply.commentUser,reply.commentId)">
                            回复</span>
            </div>
            <div class="talk-box">
              <p>
                <span class="reply"><span class="reply-span-down">回复</span> @{{reply.toCommentUser}}:</span>
                <span class="reply">{{reply.commentContent}}</span>
              </p>
            </div>
            <div class="reply-box">

            </div>
          </div>
        </div>
        <div v-show="_inputShow(i)" class="my-reply my-comment-reply">
          <div class="reply-info">
            <div tabindex="0" contenteditable="true" spellcheck="false" placeholder="输入评论..."
                 @input="onDivInput($event)" class="reply-input reply-comment-input"></div>
          </div>

          <div class=" reply-btn-box">
            <el-button class="reply-btn" size="medium" @click="sendCommentReply(i,j)" type="primary">发表评论
            </el-button>
          </div>
        </div>
      </div>
    </el-scrollbar>
  </div>
</template>

<script>

const clickoutside = {
  // 初始化指令
  bind(el, binding, vnode) {
    function documentHandler(e) {
      // 这里判断点击的元素是否是本身，是本身，则返回
      if (el.contains(e.target)) {
        return false;
      }
      // 判断指令中是否绑定了函数
      if (binding.expression) {
        // 如果绑定了函数 则调用那个函数，此处binding.value就是handleClose方法
        binding.value(e);
      }
    }
    // 给当前元素绑定个私有变量，方便在unbind中可以解除事件监听
    el.vueClickOutside = documentHandler;
    document.addEventListener('click', documentHandler);
  },
  update() {
  },
  unbind(el, binding) {
    // 解除事件监听
    document.removeEventListener('click', el.vueClickOutside);
    delete el.vueClickOutside;
  },
};
export default {
  name: 'ArticleComment',
  data() {
    return {
      count: 100,
      loading: false,
      btnShow: false,
      index: '0',
      myHeader:'https://ae01.alicdn.com/kf/Hd60a3f7c06fd47ae85624badd32ce54dv.jpg',
      replyComment: '',
      myName: '用户名',
      toCommentUser: '',
      parentCommentId: -1,
      comments: [
        {
          commentUser: '',
          commentId: '',
          commentContent: '',
          updatedDate: '',
          inputShow: false,
          reply: [
            {
              commentUser: '',
              commentId: '',
              toCommentUser: '',
              parentCommentId: '',
              commentContent: '',
              updatedDate: '',
              inputShow: false
            }
          ]
        }
      ]
    }
  },
  props: {
    // 接收父组件传值的变量
    ProductIdParam: {
      type: String,
      default: () => {
        return null
      }
    }
  },
  directives: {clickoutside},
  created() {
    this.getCurrentUser()
    this.getCommentData()
  },
  computed: {
    noMore () {
      return this.count >= 20
    },
    disabled () {
      return this.loading || this.noMore
    }
  },
  methods: {
    getCurrentUser(){
      this.axios.get(`/comment/getCurrentUser`).then((res)=>{
        //获取当前用户
        this.myName = res.member;
      })
    },
    getCommentData() {
      this.axios.get(`/comment/listMain/${this.ProductIdParam}`).then((res)=>{
        //获取评论列表
        this.comments = res.commentList;
      })
    },
    saveComment(comment){
      this.axios.post('/comment/opeMain').then((res)=>{
        this.comments = res.commentList;
      })
    },
    inputFocus() {
      var replyInput = document.getElementById('replyInput');
      replyInput.style.padding = "8px 8px"
      replyInput.style.border = "2px solid #409EFF"
      replyInput.focus()
    },
    showReplyBtn() {
      this.btnShow = true
    },
    hideReplyBtn() {
      var replyInput = document.getElementById('replyInput');
      this.btnShow = false
      replyInput.style.padding = "10px"
      replyInput.style.border = "none"
    },
    showReplyInput(i, commentUser, commentId) {
      this.comments[this.index].inputShow = false
      this.index = i
      this.comments[i].inputShow = true
      this.toCommentUser = commentUser
      this.parentCommentId = commentId
    },
    _inputShow(i) {
      return this.comments[i].inputShow
    },
    sendComment() {
      if (!this.replyComment) {
        this.$message({
          showClose: true,
          type: 'warning',
          message: '评论不能为空'
        })
      } else {
        let a = {}
        let input = document.getElementById('replyInput')
        a.commentUser = this.myName
        a.commentContent = this.replyComment
        a.knowlgId = this.knowlgIdParam
        this.comments.push(a)
        this.replyComment = ''
        input.innerHTML = ''
        this.saveComment(a)
      }
    },
    sendCommentReply(i, j) {
      if (!this.replyComment) {
        this.$message({
          showClose: true,
          type: 'warning',
          message: '评论不能为空'
        })
      } else {
        let a = {}
        a.commentUser = this.myName
        a.toCommentUser = this.toCommentUser
        a.commentContent = this.replyComment
        a.parentCommentId = this.parentCommentId
        a.knowlgId = this.knowlgIdParam
        if(!this.comments[i].reply){
          this.comments[i].reply = []
        }
        this.comments[i].reply.push(a)
        this.replyComment = ''
        document.getElementsByClassName("reply-comment-input")[i].innerHTML = ""
        this.saveComment(a)
      }
    },
    onDivInput: function (e) {
      this.replyComment = e.target.innerHTML;
    },
    load () {
      this.loading = true
      setTimeout(() => {
        this.count += 2
        this.loading = false
      }, 2000)
    }
  },
}
</script>

<style lang="stylus" scoped>
.comment-bg
  background-color #7ECF68
.my-reply
  padding 10px
  background-color #fafbfc

  .header-img
    display inline-block
    vertical-align top

  .reply-info
    display inline-block
    margin-left 5px
    width 90%
    @media screen and (max-width: 1200px) {
      width 80%
    }

    .reply-input
      min-height 20px
      line-height 22px
      padding 10px 10px
      color #ccc
      background-color #fff
      border-radius 5px

      &:empty:before
        content attr(placeholder)

      &:focus:before
        content none

      &:focus
        padding 8px 8px
        border 2px solid #409EFF
        box-shadow none
        outline none

  .reply-btn-box
    height 25px
    margin 10px 0

    .reply-btn
      position relative
      float right
      margin-right 15px

.my-comment-reply
  margin-left 50px

  .reply-input
    width flex

.reply-span
  font-size 14px
  color #909399

.reply-span-down
  font-size 12px
  color #909399

.author-title:not(:last-child)
  border-bottom: 1px solid rgba(178, 186, 194, .3)

.author-title
  padding 10px

  .header-img
    display inline-block
    vertical-align top

  .author-info
    display inline-block
    margin-left 5px
    width 60%
    height 40px
    line-height 20px

    > span
      display block
      cursor pointer
      overflow hidden
      white-space nowrap
      text-overflow ellipsis

    .author-commentUser
      font-size 14px
      font-weight bold

    .author-time
      font-size 12px
      color #909399

  .icon-btn
    width 30%
    padding 0 !important
    float right
    @media screen and (max-width: 1200px) {
      width 20%
      padding 7px
    }

    > span
      cursor pointer

    .iconfont
      margin 0 5px

  .talk-box
    margin 0 50px

    > p
      margin 0

    .reply
      font-size 14px

  .reply-box
    margin 10px 0 0 50px
</style>