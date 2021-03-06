<template>
    <div class="shopcart">
        <div class="cart-content" @click="toggleList">
            <div class="cart-left" :class="{'highlight':totalCount>0}">
                <div class="logo-wrapper">
                    <div class="logo-img">
                        <i class="icon-shopping_cart"></i>
                    </div>
                    <div v-show="totalCount>0" class="shopping-count">{{totalCount}}</div>
                </div>
                <div class="cart-price">¥{{totalPrice}}</div>
                <div class="cart-desc">另需费送费¥{{deliveryPrice}}元</div>
            </div>
            <div class="cart-right" :class="payClass">{{payDesc}}</div>
        </div>
        <transition name="fold">
            <div class="shopcart-list" v-show="listShow">
                <div class="list-header">
                    <h1 class="list-title">购物车</h1>
                    <span class="list-clean" @click="cleanList">清空</span>
                </div>
                <div class="list-content" ref="listcontent">
                    <ul>
                        <li class="list-food border-bottom-1px" v-for="food in selectFoods">
                            <span class="list-name">{{food.name}}</span>
                            <span class="list-price">¥{{food.price * food.count}}</span>
                            <div class="cartcontrol-wrapper">
                                <cartcontrol :food="food"></cartcontrol>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </transition>
        <transition name="fade">
            <div class="list-background" v-show="listShow" @click="hideList"></div>
        </transition>
        <div class="ball-container">
            <transition name="drop" v-for="(ball,index) in balls" :key="ball" @before-enter="beforeEnter"
                        @enter="enter" @after-enter="afterEnter">
                <div v-show="ball.show" class="ball">
                    <div class="inner inner-hook"></div>
                </div>
            </transition>
        </div>
    </div>
</template>

<script type="text/ecmascript-6">
    import Vue from 'vue';
    import cartcontrol from 'components/cartcontrol/cartcontrol';
    import BScroll from 'better-scroll';
    export default {
        props: {
            selectFoods: {
                type: Array,
                default() {
                    return [];
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
        data () {
            return {
                fold: false,
                balls: [
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
                    },
                    {
                        show: false
                    }
                ],
                dropBalls: []
            };
        },
        computed: {
            totalPrice() {
                let total = 0;
                this.selectFoods.forEach((food) => {
                    total += food.price * food.count;
                });
                return total;
            },
            totalCount() {
                let count = 0;
                this.selectFoods.forEach((food) => {
                    count += food.count;
                });
                return count;
            },
            payDesc() {
                if (this.totalPrice === 0) {
                    return `¥${this.minPrice}元起送`;
                } else if (this.totalPrice < this.minPrice) {
                    let diff = this.minPrice - this.totalPrice;
                    return `还差¥${diff}元起送`;
                } else {
                    return '去结算';
                }
            },
            payClass() {
                if (this.totalPrice >= this.minPrice) {
                    return 'enough-pay';
                } else {
                    return '';
                }
            },
            listShow() {
                if (!this.totalCount) {
                    this.fold = false;
                    return false;
                }
                if (this.fold) {
                    this.$nextTick(() => {
                        if (!this.scroll) {
                            this.scroll = new BScroll(this.$refs.listcontent, {
                                click: true
                            });
                        } else {
                            this.scroll.refresh();
                        }
                    });
                }
                return this.fold;
            }
        },
        methods: {
            toggleList() {
                if (!this.totalCount) {
                    return;
                }
                this.fold = !this.fold;
            },
            hideList() {
                this.fold = false;
            },
            cleanList() {
                this.selectFoods.forEach((food) => {
                    if (food.count) {
                        Vue.delete(food, 'count');
                    }
                });
            },
            drop(el) {
                for (let i = 0; i < this.balls.length; i++) {
                    let ball = this.balls[i];
                    if (!ball.show) {
                        ball.show = true;
                        ball.el = el.target;
                        this.dropBalls.push(ball);
                        return;
                    }
                }
            },
            beforeEnter(el) {
                let count = this.balls.length;
                while (count--) {
                    let ball = this.balls[count];
                    if (ball.show) {
                        let rect = ball.el.getBoundingClientRect();
                        let x = rect.left - 32;
                        let y = -(window.innerHeight - rect.top - 22);
                        el.style.display = '';
                        el.style.webkitTransform = `translate3d(0,${y}px,0)`;
                        el.style.transform = `translate3d(0,${y}px,0)`;
                        let inner = el.getElementsByClassName('inner-hook')[0];
                        inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
                        inner.style.transform = `translate3d(${x}px,0,0)`;
                    }
                }
            },
            enter(el, done) {
                /* eslint-disable no-unused-vars */
//                触发重绘html
                let refresh = el.offsetHeight;
                this.$nextTick(() => {
                    el.style.webkitTransform = 'translate3d(0,0,0)';
                    el.style.transform = 'translate3d(0,0,0)';
                    let inner = el.getElementsByClassName('inner-hook')[0];
                    inner.style.webkitTransform = 'translate3d(0,0,0)';
                    inner.style.transform = 'translate3d(0,0,0)';
//                    Vue为了知道过渡的完成，必须设置相应的事件监听器。
                    el.addEventListener('transitionend', done);
                });
            },
            afterEnter(el) {
                let ball = this.dropBalls.shift();
                if (ball) {
                    ball.show = false;
                    el.style.display = 'none';
                }
            }
        },
        components: {
            'cartcontrol': cartcontrol
        }
    };
</script>

<style scoped>
    .shopcart {
        position: fixed;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 46px;
        z-index: 100;
    }

    .cart-content {
        display: flex;
        background-color: #141d27;
        font-size: 0;
    }

    .logo-wrapper {
        display: inline-block;
        position: relative;
        top: -10px;
        margin: 0 12px;
        padding: 6px;
        width: 56px;
        height: 56px;
        box-sizing: border-box;
        border-radius: 50%;
        background-color: #141d27;
    }

    .logo-img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: #2b343c;
        text-align: center;
    }

    .logo-img .icon-shopping_cart {
        color: #80858a;
        line-height: 44px;
        font-size: 24px;
    }

    .shopping-count {
        position: absolute;
        top: 0;
        right: 0;
        width: 24px;
        height: 16px;
        text-align: center;
        line-height: 16px;
        font-size: 9px;
        background-color: #f01414;
        color: #fff;
        border-radius: 16px;
        font-weight: 700;
        box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4);
    }

    .cart-left {
        flex: 1;
        font-size: 0;
    }

    .cart-price {
        display: inline-block;
        margin: 12px 0;
        padding-right: 12px;
        vertical-align: top;
        color: #919396;
        font-size: 16px;
        font-weight: 700;
        line-height: 24px;
        border-right: 1px solid rgba(255, 255, 255, 0.1);
    }

    .cart-desc {
        display: inline-block;
        font-size: 12px;
        vertical-align: top;
        color: rgba(255, 255, 255, 0.2);
        font-weight: 700;
        line-height: 48px;
        margin-left: 12px;
    }

    .highlight .logo-img {
        background-color: #00a0dc;
    }

    .highlight .icon-shopping_cart {
        color: #fff;
    }

    .highlight .cart-price {
        color: #fff;
    }

    .cart-right {
        flex: 0 0 100px;
        width: 100px;
        height: 48px;
        text-align: center;
        line-height: 48px;
        color: rgba(255, 255, 255, 0.4);
        font-size: 12px;
        font-weight: 700;
        background-color: #2b333b;
    }

    .enough-pay {
        background-color: #00b43c;
        color: #fff;
    }

    .fold-enter-active, .fold-leave-active {
        transition: all .25s linear;
    }

    .fold-enter, .fold-leave-active {
        transform: translate3d(0, 100%, 0);
    }

    .shopcart-list {
        position: absolute;
        bottom: 46px;
        left: 0;
        z-index: -1;
        width: 100%;
    }

    .list-header {
        height: 40px;
        line-height: 40px;
        padding: 0 18px;
        background-color: #f3f5f7;
        border-bottom: 1px solid rgba(7, 17, 27, 0.1);
    }

    .list-title {
        float: left;
        font-size: 14px;
        color: rgb(7, 17, 27);
    }

    .list-clean {
        float: right;
        font-size: 12px;
        color: rgb(0, 160, 220);
    }

    .list-content {
        padding: 0 18px;
        max-height: 217px;
        background-color: #fff;
        overflow: hidden;
    }

    .list-food {
        position: relative;
        padding: 12px 0;
        box-sizing: border-box;
    }

    .list-name {
        font-size: 14px;
        color: rgb(7, 17, 27);
        font-weight: 700;
        line-height: 24px;
    }

    .list-price {
        position: absolute;
        right: 90px;
        bottom: 12px;
        line-height: 24px;
        font-size: 14px;
        font-weight: 700;
        color: rgb(240, 20, 20);
    }

    .cartcontrol-wrapper {
        position: absolute;
        right: 0;
        bottom: 6px;
    }

    .list-background {
        width: 100%;
        height: 100vh;
        position: absolute;
        bottom: 0;
        left: 0;
        background-color: rgba(7, 17, 27, 0.6);
        -webkit-backdrop-filter: blur(10px);
        backdrop-filter: blur(10px);
        z-index: -2;
    }

    .ball-container .ball {
        position: fixed;
        left: 32px;
        bottom: 22px;
        z-index: 2000;
        transition: all .4s cubic-bezier(0.49, -0.29, 0.75, 0.41);
    }

    .ball .inner {
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background-color: rgb(0, 160, 220);
        transition: all .4s linear;
    }
</style>
