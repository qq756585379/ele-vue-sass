<template>
  <div class="cartcontrol">
    <transition name="move">
      <div class="cart-decrease" v-show="food.count > 0" @click.stop.prevent="decreaseCart">
        <span class="inner icon-remove_circle_outline"></span>
      </div>
    </transition>
    <div class="cart-count" v-show="food.count > 0">{{ food.count }}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue';
  import Bus from '../../common/js/eventBus';
  export default {
    props: {
      food: { type: Object }
    },
    methods: {
      addCart(event) {
        // 防止多次被点击
        if (!event._constructed) {
          return;
        }
        if (!this.food.count) {
          // 添加food不存在的字段时 需要调用vue.set方法添加
          Vue.set(this.food, 'count', 1);
        } else {
          this.food.count++;
        }
        Bus.$emit('cart.add', event.target);
      },
      decreaseCart(event) {
        // 防止多次被点击
        if (!event._constructed) {
          return;
        }
        if (this.food.count) {
          this.food.count--;
        }
      }
    }
  };
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "./cartcontrol.scss";
</style>
