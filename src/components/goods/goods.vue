<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
    	<ul>
    		<li v-for="(item,index) in goods" @click="selectMenu(index,$event)" class="menu-item" :class="{'current':currentIndex===index}">
  	  		<span class="text border-1px">
  	  			<span v-show="item.type>0" class="icon" :class="iconClassMap[item.type]"></span>{{item.name}}
  	  		</span>
    		</li>
    	</ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
    	<ul>
  	  <li v-for="item in goods" class="food-list food-list-hook">
  	  	<h1 class="title">{{item.name}}</h1>
  	  	<ul>
  	  		<li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
  	  			<div class="icon">
  	  				<img width="57" height="57" :src="food.icon">
  	  			</div>
  	  			<div class="content">
  	  				<h2 class="name">{{food.name}}</h2>
  	  				<p class="desc">{{food.description}}</p>
  	  				<div class="extra">
  	  					<span class="count">月售{{food.sellCount}}份</span>
  	  					<span>好评率{{food.rating}}%</span>
  	  				</div>
  	  				<div class="price">
  	  					<span class="now">¥{{food.price}}</span><span class="old" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
  	  				</div>
              <div class="cartControl-wrapper">
                <cartControl :food="food" @add="addFood"></cartControl>
              </div>
  	  			</div>
  	  		</li>
  	  	</ul>
  	  </li>
  	</ul>
    </div>
    <div class="shopCart-wrapper">
      <shopCart ref="shopCart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopCart> 
      <food @add="addFood" :food="selectedFood" ref="food"></food> 
    </div>
  </div>
</template>

<script>
import cartControl from 'components/cartcontrol/cartcontrol'
import BScroll from 'better-scroll'
import shopCart from 'components/shopcart/shopcart'
import food from 'components/food/food'
import axios from 'axios'

const ERR_OK = '获取数据失败！'

export default {
  props: {
    seller: {
      type: Object
    }
  },
  data () {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0,
      selectedFood: {}
    }
  },
  created () {
    axios.get('api/goods').then((res) => {
      this.goods = res.data.data
      this.$nextTick(() => {
        this._initScroll()
        this._calculateHeight()
      })
    })
    .catch((res) => {
      console.log(ERR_OK)
    })
    this.iconClassMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
  },
  computed: {
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i]
        let height2 = this.listHeight[i + 1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i
        }
      }
      return 0
    },
    selectFoods () {
      let foods = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  methods: {
    selectMenu (index, event) {
      // 去掉自带的点击事件
      if (!event._constructed) {
        return
      }
      // 通过设置ref属性可以获得DOM节点，使用$refs实现操作节点
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let el = foodList[index]
      this.foodsScroll.scrollToElement(el, 300)
    },
    selectFood (food, event) {
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      // 在插入子组件处设置ref属性可以获得子组件的引用，使用$refs引用子组件的方法
      this.$refs.food.show()
    },
    addFood (target) {
      this._drop(target)
    },
    // 通过@符可以实现事件监听并执行回调函数
    _drop (target) {
      this.$refs.shopCart.drop(target)
    },
    _initScroll () {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      })
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      })
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))
      })
    },
    _calculateHeight () {
      // 通过设置ref获得DOM节点
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i]
        height += item.clientHeight
        this.listHeight.push(height)
      }
    }
  },
  components: {
    shopCart,
    cartControl,
    food
  }
}

</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin.styl"
.goods
  display: flex
  position: absolute
  top: 174px
  bottom: 46px
  width: 100%
  overflow: hidden
  .menu-wrapper
    flex: 0 0 80px
    width: 80px
    background: #f3f5f7
    .menu-item
      display: table
      height: 54px
      width: 56px
      padding: 0 12px
      line-height: 14px
      &.current
        position: relative
        z-index: 10
        margin-top: -1px
        background: #fff
        font-weight: 700
        .text
          border-none()
      .icon
        display: inline-block
        vertical-align: top
        width: 12px
        height: 12px
        margin-right: 2px
        background-size: 12px 12px
        background-repeat: no-repeat
        &.decrease
          bg-image('decrease_3')
        &.discount
          bg-image('discount_3')
        &.guarantee
          bg-image('guarantee_3')
        &.invoice
          bg-image('invoice_3')
        &.special
          bg-image('special_3')
      .text
        display: table-cell
        width: 56px
        vertical-align: middle
        border-1px(rgba(7, 17, 27, 0.1))
        font-size: 12px
  .foods-wrapper
    flex: 1
    .title
      padding-left: 14px
      height: 26px
      line-height: 26px
      border-left: 2px solid #d9dde1
      font-size: 12px
      color: rgb(147, 153, 159)
      background: #f3f5f7
    .food-item
      display: flex
      margin: 18px
      padding-bottom: 18px
      border-1px(rgba(7, 17, 27, 0.1))
      &:last-child
        border-none()
        margin-bottom: 0
      .icon
        flex: 0 0 57px
        margin-right: 10px
      .content
        flex: 1
        .name
          margin: 2px 0 8px 0
          height: 20px
          line-height: 14px
          font-size: 14px
          color: rgb(7, 17, 27)
        .desc, .extra
          font-size: 10px
          line-height: 10px
          color: rgb(147, 153, 159)
        .desc
          line-height: 12px
          margin-bottom: 8px
        .extra
          .count
            margin-right: 1px
        .price
          font-weight: 700
          line-height: 24px
          .now
            margin-right: 8px
            font-size: 14px
            color: rgb(240, 20, 20)
          .old
            text-decoration: line-through
            font-size: 10px
            color: rgb(147, 153, 159)
        .cartControl-wrapper
          position: absolute
          right: 0
          bottom: 12px

</style>
