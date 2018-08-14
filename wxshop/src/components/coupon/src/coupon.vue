<template>
<ul>
    <!-- <li class="coupon_item_theme coupon_theme3" v-if="queryType == 'detail'" v-for='coupon in couponData'>
            <div class="item2 c_ing" v-if="coupon.couponCount>coupon.receiveCount">
                <div class="p_price">￥<span class="p_1">{{coupon.couponAmount}}</span></div>
                 <div class="c_user"><a href="">去使用</a></div>
            </div>
            <div class="item2 c_over" v-else>
                <div class="p_price">￥<span class="p_1">{{coupon.couponAmount}}</span></div>
                 <div class="c_user"><a href="">去使用</a></div>
                 <div class="c_user"已过期</div>
            </div>
            <div :class='{"item1 over_mess":coupon.isOver,"item1 user_mess":!coupon.isOver}'>
                <div class="title">{{coupon.couponName}}</div>
                <ul class="c_mess">
                    <li><em></em>满{{coupon.leastConsumpMoney}}元可使用</li>
                    <li><em></em>{{coupon.endTime}}到期</li>
                    <li v-if="coupon.prodRestrict==1"><em></em>适用全部商品</li>
                    <li v-else-if="coupon.prodRestrict==2"><em></em>部分商品可用</li>
                </ul>
            </div>
        </li> -->
        <li class="coupon_item_theme coupon_theme3" v-for='coupon in couponData'>
            <div class='item1'>
                <div class="title">{{coupon.couponName}}</div>
                <div class="price">￥<span class='p_1'>{{coupon.couponAmount}}</span></div>
                <ul class="c_mess">
                    <li><em></em>满{{coupon.leastConsumpMoney}}元可使用</li>
                    <li><em></em>{{coupon.endTime}}到期</li>
                    <li v-if="coupon.prodRestrict==1"><em></em>适用全部商品</li>
                    <li v-else-if="coupon.prodRestrict==2"><em></em>部分商品可用</li>
                </ul>
            </div>
            <div class="item2">
                <span>立即</span></br>
                <span>领取</span>
            </div>
        </li>
   <!--  <li class="coupon_item_theme coupon_theme2">
            <div class="item2 c_ing">
                <div class="p_price">￥<span class="p_1">100</span></div>
                <div class="c_user"><a href="">去使用</a></div>
            </div>
            <div class='item1 user_mess'>
                <div class="title">618活动优惠券</div>
                <ul class="c_mess">
                    <li><em></em>满2000元可使用</li>
                    <li><em></em>2017年3月30日到期</li>
                    <li><em></em>全国通用</li>
                </ul>
            </div>
        </li>
        <li class="coupon_item_theme coupon_theme2">
            <div class="item2 c_over">
                <div class="p_price">￥<span class="p_1">100</span></div>
                <div class="c_user">已过期</div>
            </div>
            <div class='item1 over_mess'>
                <div class="title">618活动优惠券</div>
                <ul class="c_mess">
                    <li><em></em>满2000元可使用</li>
                    <li><em></em>2017年3月30日到期</li>
                    <li><em></em>全国通用</li>
                </ul>
            </div>
        </li> -->

</ul>
</template>

<script>
    import Vue from 'vue';
    import {getSlideAngle, getSlideDirection} from '../../../utils/assist';

    export default {
        name: 'yd-coupon',
        props: {
            couponData: {
                type:Array,
                default:[]
            },
            index: Number
        },
        data() {
            return { 
                queryType: ''
            }
        },
        methods: {
             
        },
         mounted() {
             //初始化页面标题
           console.log(11);
        }
    }
</script>
<style lang="less">
    @import "../../../styles/components/coupon.less";
</style>