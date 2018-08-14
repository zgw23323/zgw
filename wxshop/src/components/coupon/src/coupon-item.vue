<template>
    <ul>
        <!-- 优惠券列表 -->
        <li class="coupon_item_theme coupon_theme1" v-if="queryType =='list'" v-for='coupon in couponData'>
            <div class='item1'>
                <div class="title">{{coupon.couponName}}</div>
                <div class='c_price'>
                    ￥<span class="p_1">{{coupon.couponAmount}}</span><span class="p_2">满{{coupon.leastConsumpMoney}}元</span>
                </div>
                <ul class="c_mess">
                    <li v-if="coupon.isShopCoupon==1 && coupon.isRechargeCoupon == 1"><em></em>适用部分商品、适用充值</li>
                    <li v-else-if="coupon.isShopCoupon==1 && coupon.isRechargeCoupon==2"><em></em>部分商品可用</li>
                    <li v-else-if="coupon.isShopCoupon==2 && coupon.isRechargeCoupon==1"><em></em>适用全部商品、适用充值</li>
                    <li v-else-if="coupon.isShopCoupon==2 && coupon.isRechargeCoupon==2"><em></em>适用全部商品</li>
                    <li v-else><em></em>适用充值</li>
                    <li><em></em>活动时间：{{coupon.startTime}} - {{coupon.endTime}}</li>
                </ul>
            </div>
            <!-- 未过期 -->
            <div class="item2 theme1_ing" @click="couponBtn(coupon.id)" v-if="coupon.couponCount>coupon.receiveCount">
                <a href="">&nbsp;</a>
            </div>
            <div class="item2 theme1_over" v-else>&nbsp;</div>
        </li>
        <!-- 个人优惠券列表 -->
        <li class="coupon_item_theme coupon_theme2" v-if="queryType =='person'" v-for='coupon in couponData'>
            <div class="item2 c_ing" v-if="coupon.receiveCount>coupon.usedCount && coupon.couponStatus">
                <div class="p_price">￥<span class="p_1">{{coupon.couponAmount}}</span></div>
                 <div class="c_user" v-if="coupon.couponStatus" @click='goToUserCoupon(coupon)'>去使用</div>
            </div>
            <div class="item2 c_over" v-else>
                <div class="p_price">￥<span class="p_1">{{coupon.couponAmount}}</span></div>
                <div class="c_user" v-if="coupon.receiveCount==coupon.usedCount">已使用</div>
                <div class="c_user" v-else-if="coupon.couponStatus==false">已过期</div>
            </div>
            <div :class='{"item1 over_mess":coupon.isOver,"item1 user_mess":!coupon.isOver}'>
                <div class="title">{{coupon.couponName}}</div>
                <ul class="c_mess">
                    <li><em></em>满{{coupon.leastConsumpMoney}}元可使用</li>
                    <li><em></em>{{coupon.endTime}}到期</li>
                    <li v-if="coupon.isShopCoupon==1 && coupon.isRechargeCoupon == 1"><em></em>适用部分商品、适用充值</li>
                    <li v-else-if="coupon.isShopCoupon==1 && coupon.isRechargeCoupon==2"><em></em>部分商品可用</li>
                    <li v-else-if="coupon.isShopCoupon==2 && coupon.isRechargeCoupon==1"><em></em>适用全部商品、适用充值</li>
                    <li v-else-if="coupon.isShopCoupon==2 && coupon.isRechargeCoupon==2"><em></em>适用全部商品</li>
                    <li v-else><em></em>适用充值</li>
                </ul>
            </div>
        </li>
        <li class="coupon_item_theme coupon_theme3" v-if="queryType == 'detail'" v-for='coupon in couponData'>
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
    </ul>
</template>

<script>
    import Vue from 'vue';
    import {getSlideAngle, getSlideDirection} from '../../../utils/assist';

    export default {
        name: 'yd-coupon-item',
        props: {
            index: Number,
            couponData: {
                type:Array,
                default:[]
            },
            typeCoupon:{
                type:String,
                default:''
            }
        },
        data() {
            return {
                //领取优惠券
                apiAddUserCoupon:window.baseUrl+window.appName+'/shopapi/func/coupon/addUserCoupon?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
                queryType: ''
            }
        },
        methods: {
             //领取优惠券
            couponBtn(str){
              let that = this;   
                that.$http.post(that.apiAddUserCoupon,{'couponId':str,'couponCount':1})
                .then((response) => {
                  const data = response.data;
                  if(data.code == "S_OK") {
                    that.$dialog.notify({mes: '领取成功', icon: 'error', timeout: 2000});
                  }else if(data.code == "USER_COUPON_ALREADY_RECEIVED") {
                    that.$dialog.notify({mes: '您已经领取过了哦~', icon: 'error', timeout: 1200});
                  }else if(data.code == "COMMON_ADD_FAIL") {
                    that.$dialog.notify({mes: '领取失败', icon: 'error', timeout: 1200});
                  }else if(data.code == "PARAM_ERROR") {
                    that.$dialog.notify({mes: '参数错误', icon: 'error', timeout: 1200});
                  }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                        that.$router.push('/login');
                    }});
                  }else if(data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                        that.$router.push('/login');
                    }});
                  }else {
                    that.$dialog.notify({mes: '系统异常', icon: 'error', timeout: 1200});
                  }
                },(response) => {
                  that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
                  setTimeout(()=>{
                    that.$dialog.loading.close();
                  },300);
                });
            },
            //去使用优惠券
            goToUserCoupon(opt){
                var _url = ''
                if(opt.isRechargeCoupon==1 && opt.isShopCoupon == 0){ //跳去充值
                    _url = '/recharge';
                }else{
                    _url = '/goods';
                }
                this.$router.push({
                    path: _url,
                    query: {
                      pId: 'coupon'
                    }
                });
                sessionStorage.setItem('couponId',opt.couponId);
            }
        },
        mounted() {
             //初始化页面标题
            if(this.$route.query.mark == 'person'){
              document.title = '我的优惠券';
              this.queryType = 'person';
            }else{     
                // this.queryType = this.typeCoupon;
                this.queryType = 'list';
            }
        }
    }
</script>

<style lang="less">
    // @import "../../../styles/components/coupon.less";
</style>