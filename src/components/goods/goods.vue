<template>
	<div class="goods">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="(item,index) of goods" class="menu-item" :class="{'current':currentIndex === index}" @click="selectMenu(index,$event)">
					<span class="text border-1px">
						<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
						{{item.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodsWrapper">
			<ul>
				<li v-for="item of goods" class="food-list" ref="foodList">
					<h1 class="title" v-text="item.name"></h1>
					<ul>
						<li @click="selectFood(food,$event)" v-for="food of item.foods" class="food-item border-1px">
							<div class="icon">
								<img width="57" height="57" :src="food.icon"/>
							</div>
							<div class="content">
								<h2 class="name" v-text="food.name"></h2>
								<p class="des" v-text="food.description"></p>
								<div class="extra">
									<span class="count" v-text="'月售'+food.sellCount"></span>
									<span v-text="'好评率'+food.rating+'%'"></span>
								</div>
								<div class="price">
									<span class="now" v-text="'￥'+food.price"></span>
									<span class="old" v-show="food.oldPrice" v-text="'￥'+food.oldPrice"></span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartcontrol @add="addFood" :food="food"></cartcontrol>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shopcart ref="shopcart" :selectFoods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
		<food @add="addFood" :food="selectedFood" ref="food"></food>
	</div>
</template>

<script>
	import BScroll from 'better-scroll';
	import shopcart from '@/components/shopcart/shopcart.vue';
	import cartcontrol from '@/components/cartcontrol/cartcontrol.vue';
	import food from '@/components/food/food.vue';
	const response = require('../../common/data/goods.json');
	
	const ERR_OK = 0;
	
	  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
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
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];

//    this.$http.get('/api/goods').then((response) => {
//      response = response.body;
//      if (response.errno === ERR_OK) {
//        this.goods = response.data;
//        this.$nextTick(() => {
//          this._initScroll();
//          this._calculateHeight();
//        });
//      }
//    });
      if (response.errno === ERR_OK) {
          this.goods = response.data;
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
    },
    methods: {
      selectMenu(index, event) {
        if (!event._constructed) {
          return;
        }
        let foodList = this.$refs.foodList;
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      },
      addFood(target) {
        this._drop(target);
      },
      _drop(target) {
        // 体验优化,异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },
      _initScroll() {
        this.meunScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        });

        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          probeType: 3
        });

        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      _calculateHeight() {
        let foodList = this.$refs.foodList;
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food
    }
  };
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
	/*flex第一个参数是等分第二个参数是缩放第三个参数是占位*/
		flex: 0 0 80px
		width: 80px
		background: #f3f5f7
		.menu-item
			display: table
			width: 56px
			height: 54px
			line-height: 14px
			padding: 0 12px
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
				position: relative
				display: table-cell
				width: 56px
				vertical-align: middle
				text-align: center
				font-size: 12px
				border-1px(rgba(7,17,27,0.1))
	.foods-wrapper
		flex: 1
		.title
			padding-left: 14px
			height: 26px
			line-height: 26px
			border-left: 2px solid #d9dde1
			font-size: 12px
			color: rgb(147,153,159)
			background: #f3f5f7
		.food-item
			display: flex
			margin: 18px
			padding-bottom: 18px
			border-1px(rgba(7,17,27,0.1))
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
					height: 14px
					line-height: 14px
					font-size: 14px
					color: rgb(7,17,27)
				.des,.extra 
					font-size: 10px
					line-height: 10px
					font-size: 10px
					color: rgb(147,153,159)
				.des
					margin-bottom: 8px
					line-height: 12px
				.extra
					.count
						margin-right: 12px
				.price
					font-weight: 700
					line-height: 24px
					.now
						margin-right: 8px
						font-size: 14px
						color: rgb(240,20,20)
					.old
						text-decoration: line-through
						color: rgb(147,153,159)
						font-size: 10px
				.cartcontrol-wrapper
					position: absolute
					right: 0
					bottom: 12px
</style>