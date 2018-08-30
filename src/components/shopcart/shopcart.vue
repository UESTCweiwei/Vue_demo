<template>
  <div>
  <transition name="fade">
    <div class="list-mask" v-show="listShow" @click="hideList"></div>
  </transition>

  <div class="shopcart">
  <div class="content" @click="toggleList">
    <div class="content-left">
      <div class="logo-wrapper">
        <div class="logo" :class="{'highlight':totalCount>0}">
          <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
        </div>
        <div class="num" v-show="totalCount>0">{{totalCount}}</div>
      </div>
      <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
      <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
    </div>
    <div class="content-right" @click.stop.prevent="pay">
      <div class="pay" :class="payClass">
        {{payDesc}}
      </div>
    </div>
  </div>
  <div class="ball-container">
    <div v-for="ball in balls" :key="ball.index">
      <transition name="drop" v-on:before-enter="beforeEnter" v-on:enter="enter" v-on:after-enter="afterEnter">
        <div v-show="ball.show" class="ball">
          <div class="inner inner-hook"></div>
        </div>
      </transition>
    </div>
  </div>
  <transition name="fade">
    <div class="shopcart-list" v-show="listShow">
      <div class="list-header">
        <h1 class="title">购物车</h1>
        <span class="empty" @click="empty">清空</span>
      </div>
      <div class="list-content" ref="listContent">
        <ul>
          <li class="food" v-for="food in selectFoods" :key="food.id">
            <span class="name">{{food.name}}</span>
            <div class="price">
              <span>￥{{food.price * food.count}}</span>
            </div>
            <div class="cartcontrol-wrapper">
              <!-- 记得将cartcontrol组件import到shopcart中，并注册components-->
              <cartcontrol :food = "food"></cartcontrol>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </transition>
</div>


  </div>

</template>

<script>
  import cartcontrol from 'components/cartcontrol/cartcontrol'
  import BScroll from 'better-scroll'
    export default {
      name: "shopcart",
      props: {
        selectFoods: {
          type: Array,
          default() {
            return [
              {
                price: 10,
                count: 1
              }
            ]
          }
        },
        deliveryPrice: {
          type: Number,
          default: 0
        },
        minPrice: {
          type: Number,
          default: 0
        }
      },
      data() {
        return {
          balls: [{
            show: false
          },
            {
              show: false
            },
            {
              show: false
            },
            {
              show: false
            },
            {
              show: false
            }
          ],
          dropBalls: [],
          fold:true
        }
      },
      computed: {
        totalPrice() {
          let total = 0
          this.selectFoods.forEach((food) => {
            total += food.price * food.count;
          })
          return total

        },
        totalCount() {
          let count = 0
          this.selectFoods.forEach((food) => {
            count += food.count
          })
          return count
        },
        payDesc() {
          if (this.totalPrice === 0) {
            return `￥${this.minPrice}元起送`
          } else if (this.totalPrice < this.minPrice) {
            let diff = this.minPrice - this.totalPrice
            return `还差￥${diff}元起送`
          } else {
            return '去结算'
          }
        },
        payClass() {
          if (this.totalPrice < this.minPrice) {
            return 'not-enough'
          } else {
            return 'enough'
          }
        },
        listShow() {
          if(!this.totalCount){
            this.fold=true
            return false
          }
          let show=!this.fold
          if(show){
            this.$nextTick(() => {
              if (!this.scroll) { // 没有scroll的时候才需要new，否则只需要刷新一下即可
                this.scroll = new BScroll(this.$refs.listContent, {
                  click: true
                });
              } else {
                this.scroll.refresh();
              }
            })
          }
          return show
        }
      },
      methods: {
        drop(el) {
          for (let i = 0; i < this.balls.length; i++) {
            let ball = this.balls[i];
            if (!ball.show) { //show为false的球
              ball.show = true; //小球下落
              ball.el = el;//保留当前的DOM对象，用来计算位置
              this.dropBalls.push(ball); //dropBalls存的是已经下落的小球,后续要对已经下落的小球进行处理
              return;
            }
          }
        },
        toggleList() {
          if(!this.totalCount){
              return
          }
          this.fold=!this.fold
        },
        beforeEnter(el) { //el为当前执行transition动画的DOM对象
          //先找到所有为true的小球（连续点击的情况）
          let count = this.balls.length;
          while (count--) {
            let ball = this.balls[count];
            if (ball.show) { //这个是要运动的小球true
              let rect = ball.el.getBoundingClientRect();//获得元素相当于视口的位置
              let x = rect.left - 32;
              let y = -(window.innerHeight - rect.top - 22);
              el.style.display = ''; //v-show默认display：none，设置为空，让它显示
              //外层元素是纵向的动画，内层元素是横向的动画
              el.style.webkitTransform = `translate3d(0,${y}px,0)`;
              el.style.transform = `translate3d(0,${y}px,0)`;
              let inner = el.getElementsByClassName('inner-hook')[0];
              inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`;
              inner.style.transform = `translate3d(${x}px, 0, 0)`;
            }
          }
        },
        enter(el) {
          /* 触发浏览器重绘，重绘之后才可以设置transform*/
          /* eslint-disable no-unused-vars */
          let rf = el.offsetHeight;
          this.$nextTick(() => { //样式重置
            el.style.webKitTransform = 'translate3d(0,0,0)';//没有变量时只能用单引，不能用反引
            el.style.transform = 'translate3d(0,0,0)';
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = 'translate3d(0,0,0)';
            inner.style.transform = 'translate3d(0,0,0)';
          });
        },
        afterEnter(el) { //动画完成
          let ball = this.dropBalls.shift();//删除并返回第一个ball
          if (ball) {
            ball.show = false; //重置ball.show的状态
            el.style.display = 'none';
          }
        },
        empty() {
          this.selectFoods.forEach((food) => {
            food.count=0
          })
        },
        hideList () {
          this.fold = true
        },
        pay() {
          if (this.totalPrice<this.minPrice) {
            return

          }
          window.alert(`支付${this.totalPrice}元`)
        }

      },
      components:{
        cartcontrol
      }

    }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin'
.shopcart
  position:fixed
  left:0
  bottom :0
  z-index :50
  width: 100%
  height:48px
  background :#000
  .content
    display:flex
    background:#141d27
    .content-left
      flex:1
      .logo-wrapper
        display:inline-block
        position:relative
        top:-10px
        margin:0 12px
        padding: 6px
        width:56px
        height:56px
        box-sizing : border-box
        vertical-align :top
        border-radius :50%
        background :#141d27
        .logo
          width: 100%
          height:100%
          border-radius :50%
          background :#2b343c
          text-align :center
          &.highlight
            background:rgb(0,160,220)
          .icon-shopping_cart
            font-size :24px
            line-height :44px
            color: #80858a
            &.highlight
              color :#fff
        .num
          position:absolute
          top:0
          right :0
          width :24px
          height:16px
          line-height:16px
          text-align :center
          border-radius :16px
          font-size :9px
          font-weight :700
          color: #ffffff
          background :rgb(240,20,20)
          box-shadow :0 4px 8px 0 rgba(0,0,0,0.4)
      .price
        display:inline-block
        vertical-align:top
        line-height:24px
        padding-right:12px
        margin-top:12px
        box-sizing:border-box
        border-right:1px solid rgba(255,255,255,0.1)
        font-size:16px
        font-weight:700
        color: rgba(255,255,255,0.4)
        &.highlight
          color: #ffffff
      .desc
        display:inline-block
        vertical-align :top
        line-height :24px
        margin:12px 0 0 12px
        color:rgba(255,255,255,0.4)
        font-size :10px
    .content-right
      flex :0 0 105px
      width:105px
      .pay
        height: 48px
        line-height: 48px
        text-align:center
        font-size :12px
        color:rgba(255,255,255,0.4)
        font-weight :700
        background :#2b333b
        &.not-enough
          background :#2b333b
        &.enough
          background :#00b43c
          color:#fff


  .ball-container
    .ball
      position:fixed
      left:32px
      bottom :22px
      z-index: 200
      transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
      .inner
        width 16px
        height 16px
        border-radius 50%
        background rgb(0,160,220)
        transition: all 0.4s linear

  .shopcart-list
    position:absolute
    top:0
    left:0
    z-index: -1
    width: 100%
    transform translate3d(0, -100%, 0)//每个表项相对于当前自身的高度做一个偏移
    &.fade-enter-active, &.fade-leave-active
      transition: all 1s
    &.fade-enter, &.fade-leave-active
      transform translate3d(0, 0, 0)
    .list-header
      height 40px
      line-height 40px
      padding 0 18px
      background #f3f5f7
      border-bottom 1px solid rgba(7, 17, 27, 0.1)
      .title
        float left
        font-size 14px
        color rgb(7,17,27)
      .empty
        float right
        font-size 12px
        color rgb(0,160,220)


    .list-content
      padding 0 18px
      max-height 217px
      overflow hidden
      background  #ffffff
      .food
        position relative
        padding 12px 0
        box-sizing border-box
        border-1px(rgba(7,17,27,0.1))
        .name
          line-height 24px
          font-size 14px
          color rgb(7,17,27)
        .price
          position absolute
          right 90px
          bottom 12px
          line-height 24px
          font-size 14px
          font-weight 700
          color rgb(240, 20, 20)
        .cartcontrol-wrapper
          position absolute
          right 0
          bottom 6px

.list-mask
    position fixed
    top 0
    left 0
    width 100%
    height 100%
    z-index 40 //z-index要小于shopcart的index
    backdrop-filter blur(10px) // 模糊效果
    -webkit-backdrop-filter blur(10px)
    opacity 1
    background rgba(7, 17, 27, 0.6)
    &.fade-enter-active, &.fade-leave-active
      opacity 1
      transition: all 0.5s //设置缓动效果
      background rgba(7, 17, 27, 0.6)
    &.fade-enter, &.fade-leave-active
      opacity 0
      background rgba(7, 17, 27, 0)
</style>
