<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper">
        <ul ref="menusUl">
          <!-- current -->
          <li
            class="menu-item"
            v-for="(good, index) in goods"
            :key="index"
            :class="{ current: index === currentIndex }"
            @click="menuItem(index)"
          >
            <span class="text bottom-border-1px">
              <img class="icon" :src="good.icon" v-if="good.icon" />
              {{ good.name }}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper">
        <ul ref="foodsUl">
          <li
            class="food-list-hook"
            v-for="(good, index) in goods"
            :key="index"
            
          >
            <h1 class="title">{{ good.name }}</h1>
            <ul>
              <li
                class="food-item bottom-border-1px"
                v-for="(food, index) in good.foods"
                :key="index"
                @click='showFood(food)'
              >
                <div class="icon">
                  <img width="57" height="57" :src="food.icon" />
                </div>
                <div class="content">
                  <h2 class="name">{{ food.name }}</h2>
                  <p class="desc">{{ food.description }}</p>
                  <div class="extra">
                    <span class="count">月售{{ food.sellCount }}份</span>
                    <span>好评率{{ food.rating }}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{ food.price }}</span>
                    <span class="old" v-if="food.oldPrice"
                      >￥{{ food.oldPrice }}</span
                    >
                  </div>
                  <div class="cartcontrol-wrapper">
                    <CartControl :food="food"></CartControl>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      <ShopCart></ShopCart>
      </div>
    </div>
    <Food :food="food" ref="food"></Food>
  </div>
</template>

<script>
import BScroll from "better-scroll";
import { mapState } from "vuex";
import CartControl from "@/components/CartControl/CartControl";
import Food from "@/components/Food/Food";
import ShopCart from '@/components/ShopCart/ShopCart'
export default {
  data() {
    return {
      scorllY: 0, //右侧滑动的Y轴坐标(滑动过程中实时变化)
      tops: [], //所有右侧分类li的top值组成的数组(列表第一次显示之后就不再变化)
      food: {}
    };
  },
  computed: {
    ...mapState(["goods"]),
    currentIndex() {
      const { scorllY, tops } = this;
      //根据条件产生结果
      const index = tops.findIndex((top, index) => {
        //scorllY要大于等于当前top  并且小于下一个top
        return scorllY >= top && scorllY < tops[index + 1];
      });
      //返回结果
      return index;
    }
  },
  mounted() {
    this.$store.dispatch("getShopGoods", () => {
      //等待列表加载完之后在执行
      this.$nextTick(() => {
        this._initScorll();
        this._initTops();
      });
    });
  },
  methods: {
    _initScorll() {
      new BScroll(".menu-wrapper", {
        click: true
      });
      this.foodsScroll = new BScroll(".foods-wrapper", {
        probeType: 2
      });
      //给右侧列表绑定scorll监听
      this.foodsScroll.on("scroll", ({ x, y }) => {
        this.scorllY = Math.abs(y);
      });
      //给右侧列表绑定scorll结束的监听
      this.foodsScroll.on("scrollEnd", ({ x, y }) => {
        this.scorllY = Math.abs(y);
      });
    },
    _initTops() {
      //1.初始化tops
      let tops = [];
      let top = 0;
      tops.push(top);
      //2.收集tops放到数组中
      const lis = this.$refs.foodsUl.getElementsByClassName("food-list-hook");
      Array.prototype.slice.call(lis).forEach(li => {
        top += li.clientHeight;
        tops.push(top);
      });
      //3.更新数据
      this.tops = tops;
    },
    menuItem(index) {
      //得到目标位置的scrollY
      const scorllY = this.tops[index];
      //立即更新
      this.scorllY = scorllY;
      //平滑滑动右侧列表
      this.foodsScroll.scrollTo(0, -scorllY, 300);
    },
    showFood(food){
      //更新food
      this.food = food;
      //显示food(在父组件中调用子组件对象的方法)
      this.$refs.food.toggleShowFood()

    }
  },
  components: {
    CartControl,
    Food,
    ShopCart
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import '../../../common/stylus/mixins.styl';

.goods {
  display: flex;
  position: absolute;
  top: 195px;
  bottom: 46px;
  width: 100%;
  background: #fff;
  overflow: hidden;

  .menu-wrapper {
    flex: 0 0 80px;
    width: 80px;
    background: #f3f5f7;

    .menu-item {
      display: table;
      height: 54px;
      width: 56px;
      padding: 0 12px;
      line-height: 14px;

      &.current {
        position: relative;
        z-index: 10;
        margin-top: -1px;
        background: #fff;
        color: $green;
        font-weight: 700;

        .text {
          border-none();
        }
      }

      .icon {
        display: inline-block;
        vertical-align: top;
        width: 12px;
        height: 12px;
        margin-right: 2px;
        background-size: 12px 12px;
        background-repeat: no-repeat;
      }

      .text {
        display: table-cell;
        width: 56px;
        vertical-align: middle;
        bottom-border-1px(rgba(7, 17, 27, 0.1));
        font-size: 12px;
      }
    }
  }

  .foods-wrapper {
    flex: 1;

    .title {
      padding-left: 14px;
      height: 26px;
      line-height: 26px;
      border-left: 2px solid #d9dde1;
      font-size: 12px;
      color: rgb(147, 153, 159);
      background: #f3f5f7;
    }

    .food-item {
      display: flex;
      margin: 18px;
      padding-bottom: 18px;
      bottom-border-1px(rgba(7, 17, 27, 0.1));

      &:last-child {
        border-none();
        margin-bottom: 0;
      }

      .icon {
        flex: 0 0 57px;
        margin-right: 10px;
      }

      .content {
        flex: 1;

        .name {
          margin: 2px 0 8px 0;
          height: 14px;
          line-height: 14px;
          font-size: 14px;
          color: rgb(7, 17, 27);
        }

        .desc, .extra {
          line-height: 10px;
          font-size: 10px;
          color: rgb(147, 153, 159);
        }

        .desc {
          line-height: 12px;
          margin-bottom: 8px;
        }

        .extra {
          .count {
            margin-right: 12px;
          }
        }

        .price {
          font-weight: 700;
          line-height: 24px;

          .now {
            margin-right: 8px;
            font-size: 14px;
            color: rgb(240, 20, 20);
          }

          .old {
            text-decoration: line-through;
            font-size: 10px;
            color: rgb(147, 153, 159);
          }
        }

        .cartcontrol-wrapper {
          position: absolute;
          right: 0;
          bottom: 4px;
        }
      }
    }
  }
}
</style>
