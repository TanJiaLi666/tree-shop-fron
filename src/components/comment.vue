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
        <el-rate
            show-text
            v-model="star"
            :colors="colors">
        </el-rate>
        <el-button class="reply-btn" size="medium" @click="sendComment" type="primary">发表评论</el-button>
      </div>
    </div>
    <div class="overflowAuto">
      <div v-for="(item,i) in comments" :key="i" class="author-title reply-father">
        <el-avatar class="header-img" :size="40" :src="item.headImg"></el-avatar>
        <div class="author-info">
            <span class="author-commentUser">{{item.commentUser}}</span>
            <span class="author-time">{{item.updatedDate}}</span>
          <el-rate
              show-text
              disabled
              v-model="item.star"
              :colors="colors">
          </el-rate>
        </div>
        <div class="icon-btn">
          <span class="reply-span" @click="showReplyInput(i,item.commentUser,item.commentId)">
            回复
          </span>
          <span :class="['reply-span',deleteReadOnly===true? 'myReadOnly' : '']" @click="deleteComment(item,item.commentId)">
            删除
          </span>
          <span class="reply-span" @click="showReply(i)">
            查看回复
          </span>
        </div>
        <div class="talk-box">
              <p>
                <span class="reply">{{item.commentContent}}</span>
              </p>
            </div>
        <div class="reply-box" v-show="_replyShow(i)">
              <div v-for="(reply,j) in item.reply" :key="j" class="author-title">
                <el-avatar class="header-img" :size="40" :src="reply.headImg"></el-avatar>
                <div class="author-info">
                  <span class="author-commentUser">{{reply.commentUser}}</span>
                  <span class="author-time">{{reply.updatedDate}}</span>
                </div>
                <div class="icon-btn">
                  <span class="reply-span" @click="deleteCommentReply(reply,reply.commentId)">
                    删除
                  </span>
                </div>
                <div class="talk-box">
                  <p>
                    <span class="reply"><span class="reply-span-down">回复</span> @{{reply.toCommentUser}}:</span>
                    <span class="reply">{{reply.commentContent}}</span>
                  </p>
                </div>
              </div>
            </div>
        <div v-show="_inputShow(i)" class="my-reply my-comment-reply">
              <div class="reply-info">
                <div tabindex="0" contenteditable="true" spellcheck="false" placeholder="输入评论..."
                     @input="onDivInput($event)" class="reply-input reply-comment-input"></div>
              </div>
              <div class=" reply-btn-box">
                <el-button class="reply-btn" size="medium" @click="sendCommentReply(i,j)" type="primary">回复内容
                </el-button>
              </div>
            </div>
      </div>
    </div>
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
      deleteReadOnly: false,
      star: null,
      colors: ['#99A9BF', '#F7BA2A', '#FF9900'],
      btnShow: false,
      index: '0',
      num_index: '0',
      myHeader:'',
      replyComment: '',
      myName: '用户名',
      toCommentUser: '',
      parentCommentId: -1,
      comments:null
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
  methods: {
    getCurrentUser(){
      this.axios.get(`/user/info`).then((res)=>{
        //获取当前用户
        this.myName = res.username;
        this.myHeader = res.myHeader;
      })
    },
    getCommentData() {
      this.axios.get(`/comment/list/${this.ProductIdParam}`).then((res)=>{
        //获取评论列表
        this.comments = res;
      })
    },
    saveComment(comment){
      let formData = new FormData();
      formData.append("memberNickName", comment.commentUser);
      formData.append("content", comment.commentContent);
      formData.append("productId", comment.ProductIdParam);
      formData.append("memberIcon", comment.headIcon);
      formData.append("star", comment.star);
      this.axios.post('/comment/opeMain',formData).then(()=>{
        this.$message({
          message: '发布成功',
          type: 'success'
        });
        this.getCommentData()
      })
    },
    saveCommentReply(comment){
      let formData = new FormData();
      formData.append("memberNickName", comment.commentUser);
      formData.append("content", comment.commentContent);
      formData.append("memberIcon", comment.headIcon);
      formData.append("toCommentUser", comment.toCommentUser);
      formData.append("commentId", comment.parentCommentId);//回复评论的id
      this.axios.post('/comment/opeSecondary',formData).then(()=>{
        this.$message({
          message: '回复成功',
          type: 'success'
        });
        this.getCommentData()
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
      if (this.comments[i].inputShow == false) {
        this.comments[this.index].inputShow = false
        this.index = i
        this.comments[i].inputShow = true
        this.toCommentUser = commentUser
        this.parentCommentId = commentId
      }
      else {this.comments[i].inputShow = false;}
    },
    showReply(i) {
      if (this.comments[i].replyShow == false) {
        this.comments[this.num_index].replyShow = false;
        this.num_index = i;
        this.comments[i].replyShow = true;
      }
      else {this.comments[i].replyShow = false;}
    },
    _inputShow(i) {
      return this.comments[i].inputShow
    },
    _replyShow(i) {
      return this.comments[i].replyShow
    },
    judge(item){
      if (item.commentUser===this.myName){
        this.deleteReadOnly = true;
        return true;
      }
      return false;
    },
    deleteComment(item,id){
      if (this.judge(item)) {
        this.axios.post(`/comment/delete/${id}`).then(()=>{
          this.$message({
            message: '删除成功',
            type: 'success'
          });
          this.getCommentData()
        })
      }else {
        this.$message({
          message: '失败,不可操作'
        });
      }
    },
    deleteCommentReply(reply,id){
      if (this.judge(reply)) {
        this.axios.post(`/comment/deleteRe/${id}`).then(()=>{
          this.$message({
            message: '删除成功',
            type: 'success'
          });
          this.getCommentData()
        })
      }else {
        this.$message({
          message: '失败,不可操作'
        });
      }
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
        a.headIcon = this.myHeader
        a.commentContent = this.replyComment
        a.ProductIdParam = this.ProductIdParam
        a.star = this.star
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
        a.headIcon = this.myHeader
        a.toCommentUser = this.toCommentUser
        a.commentContent = this.replyComment
        a.parentCommentId = this.parentCommentId
        if(!this.comments[i].reply){
          this.comments[i].reply = []
        }
        this.comments[i].reply.push(a)
        this.replyComment = ''
        document.getElementsByClassName("reply-comment-input")[i].innerHTML = ""
        this.saveCommentReply(a)
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

<style lang="stylus" >
.center {
  margin 0 auto
  width 598px
  height 600px
  background-color #eee
}
/* 需要在外层套一个div,切记命名特殊一点,防止因缺少scoped，对其他页面会有冲突 */
.center .el-scrollbar__wrap
  overflow-x hidden

.overflowAuto {
    overflow-y: auto;
    position: absolute;
    width: 100%;
    height: 100%;
}
.overflowAuto::-webkit-scrollbar {
  height: 6px;
  width: 6px;
}
.overflowAuto::-webkit-scrollbar-thumb {
  background: rgb(224, 214, 235);
}

.comment-bg
  background-color #b5dcd4
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
  margin 10px
  font-size 15px
  color #909399
.myReadOnly
  pointer-events none
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