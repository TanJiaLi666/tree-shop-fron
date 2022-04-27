<template>
  <div class="nav-bar" :class="{'is_fixed':isFixed}">
    <div class="container">
      <div class="pro-title">
        {{title}}
      </div>
      <div class="pro-param">
        <a href="javascript:;">概述</a><span>|</span>
        <a href="javascript:;">参数</a><span>|</span>
        <a href="javascript:;" @click="drawer=true">用户评价</a>
        <slot name="buy"></slot>
      </div>
    </div>
    <el-drawer
        class="el-drawer__body"
        title="用户评论"
        :visible.sync="drawer"
        size="30%">
      <article-comment :product-id-param="productId"></article-comment>
    </el-drawer>
  </div>
</template>
<script>
  import ArticleComment from "./comment";
  export default{
    name:'nav-bar',
    components: {ArticleComment},
    props:{
      title:String,
      productId:String
    },
    data(){
      return {
        activeName: '2',
        drawer: false,
        isFixed:false
      }
    },
    mounted(){
      window.addEventListener('scroll',this.initHeight)
    },
    methods:{
      handleClose(done) {
        this.drawer = false;
        this.indrawer = false;
      },
      initHeight(){
        let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;
        this.isFixed = scrollTop > 152? true:false;
      }
    },
    destroyed(){
      window.removeEventListener('scroll',this.initHeight,false)
    }
  }
</script>
<style lang="scss">
  @import './../assets/scss/config.scss';
  @import './../assets/scss/mixin.scss';
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
  .nav-bar{
    height:70px;
    line-height:70px;
    border-top:1px solid $colorH;
    background-color:$colorG;
    z-index:10;
    &.is_fixed{
      position:fixed;
      top:0;
      width:100%;
      box-shadow: 0 5px 5px $colorE;
    }
    .container{
      @include flex();
      .pro-title{
        font-size:$fontH;
        color:$colorB;
        font-weight:bold;
      }
      .pro-param{ 
        span{
          margin:0 10px;
        }
        a{
          color:$colorC;
        }
      }
    }
  }
</style>