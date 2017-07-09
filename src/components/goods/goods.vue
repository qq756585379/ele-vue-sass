<template>
  <div class="goods">

    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item, index) in goods" class="menu-item" :class="{'current': currentIndex === index}" @click="selectMenu(index, $event)">
          <span class="text border-1px">
              <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{ item.name }}
          </span>
        </li>
      </ul>
    </div>

    <div class="foods-wrapper" ref="foodWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{ item.name }}</h1>
          <ul>
            <li v-for="(food,index) in item.foods" class="food-item border-1px" @click="selectFood(food, $event)">
              <div class="icon"><img :src="food.icon" width="57" height="57"></div>
              <div class="content">
                <h2 class="name">{{ food.name }}</h2>
                <p class="desc">{{ food.description }}</p>
                <div class="extra">
                  <span class="count">月售{{ food.sellCount }}份</span><span>好评率{{ food.rating }}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{ food.price }}</span><span v-show="food.oldPrice" class="old">￥{{ food.oldPrice }}</span>
                </div>
                <div class="cartcontrol-wrapper"><cartcontrol :food="food"></cartcontrol></div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>

    <shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    <food :food="selectedFood" ref="food"></food>

  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import Bus from '../../common/js/eventBus';
  import food from 'components/food/food';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import shopcart from 'components/shopcart/shopcart';

  const ERR_OK = 0;
  export default {
    props: {
      seller: { type: Object }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          // DOM 更新了 操作dom时一定要在$nextTick里
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
      Bus.$on('cart.add', el => {
        this.$nextTick(() => { this.$refs.shopcart.drop(el); });
      });
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    methods: {
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, { click: true });
        // 监测实时滚动的位置 probeType: 3 除了手指 move 的时候派发scroll事件，在 swipe（手指迅速滑动一小段距离）的情况下，列表会有一个长距离的滚动动画，
        // 这个滚动的动画过程中也会实时派发滚动事件
        this.foodScroll = new BScroll(this.$refs.foodWrapper, { click: true, probeType: 3 });
        this.foodScroll.on('scroll', (pos) => {
          // 拿到实时的y坐标
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      _calculateHeight() {
        // 获取食物的li Dom节点列表
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },
      selectMenu(index, event) {
        // better-scroll 可以监听到此事件，浏览器原生不能监听到  防止pc端出现两次点击
        if (!event._constructed) {
          return;
        }
        // 获取食物的li Dom节点列表
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        // 调用better-scroll 方法滚动到响应位置
        this.foodScroll.scrollToElement(el, 300);
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      }
    },
    components: {
      cartcontrol,
      shopcart,
      food
    }
  };
</script>

<style rel="stylesheet/scss" lang="scss" scope>
  @import "../../common/scss/mixins";
  @import "./goods.scss";
</style>
