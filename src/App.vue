<template>
  <div id="app">
    <v-header :seller="seller">I am header</v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <keep-alive><router-view :seller="seller"></router-view></keep-alive>
  </div>
</template>

<script>
import {urlParse} from 'common/js/url.js'
import header from 'components/header/header.vue'
import axios from 'axios'

const ERR_OK = '获取数据失败！'

export default {
  data () {
    return {
      seller: {
        id: (() => {
          let queryParam = urlParse()
          return queryParam.id
        })()
      }
    }
  },
  created () {
    axios.get('api/seller').then((res) => {
      this.seller = res.data.data
    })
    .catch((res) => {
      console.log(ERR_OK)
    })
  },
  components: {
    'v-header': header
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import './common/stylus/mixin.styl'
  .tab
    display: flex
    width: 100%
    height: 40px
    line-height: 40px
    border-1px: rgba(7, 17, 27, 0.1)
    .tab-item
      flex: 1
      text-align: center
      & > a
        display: block
        font-size: 14px
        color: rgb(77, 85, 93)
        &.active
          color: rgb(240, 20, 20)

</style>
