 <template>
  <section class="service actionServiceInfo">
    <article class="header">
       <div class="details-hd" v-if="productDetail.picFileId">
          <div class="details-banner" v-if="serviceCode != 'GLASSSUPPORT'">
            <div class="service_conmon"  v-lazy:background-image="imgBaseUrl+'/shop/downLoad/showLogo/'+productDetail.picFileId" v-if="serviceCode == 'product'">&nbsp;</div>
            <div class="service_conmon" v-lazy:background-image="imgBaseUrl+'/shop/downLoad/showLogo/'+productDetail.picFileId" v-else>&nbsp;</div>
          </div>
          <div class="glass_GLASSSUPPORT" v-else>&nbsp;</div>
        </div>
    </article>
    <article class="content">
      <!-- 高铁贵宾厅详情 -->
      <div class="con_info coupon"  v-if="serviceCode == 'GTVIPROOM'">
        <p class="s_pro_title" style="margin-bottom: 0;">商品详情</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务包含：</p>
          <ul>
            <li>贵宾厅休息</li>
            <li>行李协助</li>
            <li>登车提醒</li>
            <li>精美餐点</li>
            <li>报刊杂志</li>
            <li>视听娱乐</li>
            <li>WIFI上网</li>
          </ul>
        </div>
      </div>
      <!-- 代驾服务详情 -->
      <div class="con_info" v-else-if="serviceCode == 'DJSERVICES'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title">代驾服务是指为会员提供不限公里数的免费代驾服务。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、全国<span class="red">200</span>多个城市覆盖</li>
            <li>2、<span class="red">24</span>小时服务</li>
            <li>3、<span class="red">不限</span>公里数</li>
          </ul>
        </div>
        <div class="s_detail_city">
          <span>城市范围（省级）</span>
          <div>安徽、北京、重庆、福建、甘肃、广东、广西、贵州、海南、河北 、黑龙江、湖北、湖南、吉林、江苏、江西、辽宁、内蒙古、宁夏、青海、山东、山西、陕西、上海、四川、天津西藏、新疆、云南、浙江</div>
        </div>
      </div>
      <!-- 道路救援详情 -->
      <div class="con_info" v-else-if="serviceCode == 'DLASSISTANCE'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">道路救援服务是汽车道路紧急救援，包括：拖车、搭电、换胎、送油服务。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、无限次拖车服务-全国50KM免费</li>
            <li>2、无限次搭电服务-全国免费</li>
            <li>3、无限次理换轮胎-全国免费</li>
            <li>4、无限次紧急送油-全国免费</li>
          </ul>
        </div>
      </div>
      <!-- 国际驾照 -->
      <div class="con_info" v-else-if="serviceCode == 'GJLICENSE'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">国际驾照服务是依据《联合国道路交通公约》并严格按照国际标准将中国驾照翻译成9国语言的标准驾照翻译文件。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、9国语言翻译</li>
            <li>2、畅行180多国</li>
            <li>3、100%合法自驾</li>
            <li>4、香港公证行公证</li>
            <li>5、澳洲NAATI认证翻译</li>
            <li>6、500多家境外租车公司认可</li>
          </ul>
        </div>
      </div>
      <!-- 家庭律师详情 -->
      <div class="con_info" v-else-if="serviceCode == 'JTLAWYER'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">家庭律师服务是提供专业的法律咨询服务。可无限次为车主及其家人提供交通出行，婚姻、财产等方面的专业家庭法律电话咨询及文书审查服务。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>提供的咨询服务：</p>
          <ul>
            <li>交通出行</li>
            <li>债权债务</li>
            <li>劳动争议</li>
            <li>合同纠纷</li>
            <li>婚姻继承</li>
            <li>知识产权</li>
            <li>房屋买卖</li>
            <li>行政诉讼等。</li>
          </ul>
        </div>
      </div>
      <!-- 家庭医生详情 -->
      <div class="con_info" v-else-if="serviceCode == 'JTDOCTOR'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">家庭医生服务是指为会员及其家人的日常伤、病、和医疗疑问提供解答与建议；日常遇到的各类就医疑问的解答等相关在线医疗服务。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、一人购买、全家受益</li>
            <li>2、三甲医院全科医生</li>
            <li>3、7*24小时在线</li>
          </ul>
        </div>
      </div>
      <!-- 保养取送车 -->
      <div class="con_info" v-else-if="serviceCode == 'BYTAKEOUTANDPICK'||serviceCode == 'BYTAKECAR'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width" style="width: 99%;">保养取送车服务是特定为会员打造的一款爱车助理产品。只需一个电话，我们即可为车主提供不限公里数和不限次数的保养维修代预约以及上门取送车服务。这款产品能够帮助车主免去保养爱车的往返奔波之苦，有更多的时间、可以投入到事业和家庭中，让生活变得更加美好。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、覆盖全国298个重点城市；</li>
            <li>2、25万+名专业认主的优质司机；</li>
            <li>3、免去车主来回路上的时间；</li>
            <li>4、无需在4S店里等待维修保养过程、省时省力；</li>
            <li>5、高额投保、省心安心；</li>
          </ul>
        </div>
      </div>
      <!-- 绿色通道 -->
      <div class="con_info" v-else-if="serviceCode == 'GREENCHANNEL'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">就医帮手-绿色通道是为会员用户提供的一项医疗服务权益，它解决了专家门诊资源有限，经常连着几个月挂不上号这一就医难题，帮助会员快速预约，节约时间成本，并且能够保障专家服务。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、服务范围覆盖全国94个城市、1549家知名医院；</li>
            <li>2、含571家三甲医院，北上广三甲医院全覆盖；</li>
            <li>3、8小时内闪电反馈；</li>
          </ul>
        </div>
      </div>
      <!-- 代配药 -->
      <div class="con_info" v-else-if="serviceCode == 'DISPENSING'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">就医帮手-代配药是为会员打造的一项医疗服务权益。针对因行动不便、工作忙碌等原因无法亲自去医院配药的会员，我们派专门的服务人员替会员到医院配药。这一服务为会员节约了往返医院并且排队的时间，同时，又能及时地配到所需要的药品，不至于耽误治疗。</p>
        <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、覆盖全国29个城市</li>
          <li>2、快速办理、节省时间</li>
        </ul>
        </div>
      </div>
      <!-- 尊享陪诊 -->
      <div class="con_info" v-else-if="serviceCode == 'ZXCLINIC'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width">尊享陪诊由“就医帮手-绿色通道”和“就医帮手-陪诊”两项服务组合构成。就医帮手-绿色通道是为会员用户提供的一项医疗服务权益，它解决了专家门诊资源有限，经常连着几个月挂不上号这一就医难题，帮助会员快速预约，节约时间成本，并且能够保障专家服务。就医帮手-陪诊是由专职的陪诊员在会员就诊时陪同，提供导诊咨询、并帮助会员排队取号、预约检查 、排队缴费、排队取药的全程门诊就医陪伴服务。通过绿色通道和陪诊服务、系统地解决了会员就医过程中的专家预约和陪护问题，既能帮助会员节约时间、保障专家服务，又能通过陪诊人员的全程陪同给予会员贵宾待遇。</p>
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、快速预约，专人陪诊；</li>
            <li>2、节省时间，提高效率；</li>
            <li>3、覆盖全国29个城市；</li>
          </ul>
        </div>
      </div>
      <div class="con_info glassSupport" style="padding: .59rem 0;" v-else-if="serviceCode == 'GLASSSUPPORT'">
        <div><img src="../../img/serviceImg/detail_26.png" style="width:7.13rem;margin-bottom: .67rem;" /></div>
        <div><img src="../../img/serviceImg/detail_27.png" style="margin-bottom:.57rem;" /></div>
        <div><img src="../../img/serviceImg/detail_28.png" style="width:6.9rem;margin-bottom: .76rem;" /></div>
        <div><img src="../../img/serviceImg/detail_29.png" style="width:6.89rem;margin-bottom: .65rem;" /></div>
        <div><img src="../../img/serviceImg/detail_30.png" style="width:6.9rem;margin-bottom: .55rem;" /></div>
        <div><img src="../../img/serviceImg/detail_31.png" style="margin-bottom:.58rem;" /></div>
        <div><img src="../../img/serviceImg/detail_32.png" style="width:6.59rem;margin-bottom: 1.1rem;" /></div>
      </div>
      <div class="con_info product" v-else-if="serviceCode == 'product'">
          <div class="p_title" v-text="productDetail.prodTitle"></div>
          <div v-html="productDetail.prodContent"></div>
      </div>
    </article>
    <div class="fixed_bottom_1" v-if="show2">
      <div class="shop_btn" v-if="serviceCode != 'product'"><router-link :to="{path:'serviceDetail',query:{'serveCode':serviceCode,'pid':productDetail.prodId,'psource':1}}">进入商城了解更多</router-link></div>  
      <div class="a_btn" @click="submit" v-if="!buyFlag || structureType==2">
        <span>一元抢拍</span>
      </div>
      <div class="a_btn h_y" v-if="buyFlag && structureType==1">
        <span>已购买过实物商品</span>
      </div>
    </div>

  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,getStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        tips1: false,
        title: '商品详情',
        serviceCode:'',
        show2:false,
        height:'80%',
        isKeyUp:false,
        isApply:false,
        imgBaseUrl: window.imgBaseUrl,
        apiUserInfoSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxConfirmUserInfo?shopsid='+window.shopsid,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/springwarm/wxYiyuanProdInfo?sid='+window.shopsid,
        apiPayOrOrderDetailUrl: window.baseUrl+window.appName+'/shopapi/func/product/payOrOrderDetail?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        apiAddOrderUrl: window.baseUrl+window.appName+"/shopapi/func/springwarm/wxYiyuanAddOrder?sid="+window.shopsid+"&orgCode="+window.orgCode,//立即购买生成订单
        optsPayOrOrderDetail: {
          orderId: '',
          dealType: '',
          payType: '',
          payMoney: '',
          payTime: '',
          wxPayCode: '',
          serialNumber: ''
        },
        optsAddToOrder: {prodId: ''},
        payOpt:{},//生成订单相关数据
        opts:{prodId:""},
        buyFlag: false,
        structureType:'',
        productDetail: {}
      }
    },
    watch: {
      '$route.path': 'markInit', //监听当前路由变化（辅助初始化）
      'personData.mobile'() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.personData.mobile)) {
          this.fetchUserData();
        }else {
          this.phoneStatus = "error";
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchProductData();
      this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_ACTIONPRODUCTDETAIL});
      document.body.scrollTop = document.documentElement.scrollTop = 0;
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
      log(mod) {
        var sItem = {};
            sItem.itemid = this.productDetail.prodId, 
            sItem.name = this.productDetail.prodName, 
            sItem.price = this.productDetail.memberPrice, 
            sItem.picid = this.productDetail.thumbnailUrl;
        mod = mod || LOGCODE.ACTION.PAGE_ACTIONPRODUCTDETAIL;
        this.$store.dispatch('log', {account:mod, ShopItem: sItem});
      },
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        // this.serviceCode = this.$route.query.serveCode;
        this.opts.prodId =  this.$route.query.pid;
      },
      submit() {
        this.log(LOGCODE.ACTION.BTN_PAGE_ACTIONPRODUCTDETAIL_BUY);
        //立即支付
        let that = this;
        //判断是否提交过，避免重复提交
        if(that.isApply){
          return false;
        }else{
          that.isApply = true;
        }
        if(this.$route.query.orderNumber){
          that.payOpt.orderNumber = that.$route.query.orderNumber;
          that.payOpt.payMoney = parseFloat(that.productDetail.memberPrice);
          that.payOpt.orderId = that.$route.query.orderId;
          that.payOpt.payType = 2;
          console.log('that.payOpt',that.payOpt);
          that.getWxPayOpts();
        }else{
          that.orderBandReceiver();
        }
      },
      
      orderBandReceiver() {
        //订单绑定
        let that = this;
        console.log('that.productDetail',that.productDetail);
        that.optsAddToOrder.prodId = that.productDetail.prodId;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiAddOrderUrl,that.optsAddToOrder)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
              that.payOpt = data.var;
              that.getWxPayOpts();
            }else{
              that.isApply = false;
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_DATA"){
                that.$dialog.notify({mes: "请选择商品", icon: 'error', timeout: 1200});
              }else if(data.code == "PRODUCT_NOEXIST"){
                that.$dialog.notify({mes: "商品不存在", icon: 'error', timeout: 1200});
              }else if(data.code == "PRODUCT_ALREADY_OFFLINE"){
                that.$dialog.notify({mes: "商品已下架", icon: 'error', timeout: 1200});
              }else if(data.code == "STOCK_NOENOUGH"){
                that.$dialog.notify({mes: "库存不足", icon: 'error', timeout: 1200});
              }else if(data.code == "PROD_ALREADY_BUY"){
                that.$dialog.notify({mes: "已购买过实物类型商品", icon: 'error', timeout: 1200});
              }else if(data.code == "SPRING_ORDER_EXISTS"){
                that.$dialog.notify({mes: "存在未付款的实物商品订单", icon: 'error', timeout: 1200});
              }else if(data.code == "USER_NOT_MEMBER"){
                that.$dialog.notify({mes: '非会员需要注册', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/Register');
                }});
              }
            }
            
          },(response) => {
            that.$dialog.loading.close();
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      getWxPayOpts() {
        //请求服务器获取微信支付需要相关参数
        let that = this;
        let aipWxpayUrl = window.shopUrl+window.appName+'/wxpay/finalpay?orderNo='+that.payOpt.orderNumber+'&money='+that.payOpt.payMoney+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+window.orgCode +'&payMType='+ that.payOpt.payType;

        if (window.wxdebug) {
          alert("获取的订单号是：" + that.payOpt.orderNumber);
          alert("发起支付请求的路径是: " + aipWxpayUrl);
        }
        if(window.isWXpay == true) {
          that.$dialog.loading.open('请求数据中...');
          that.$http.get(aipWxpayUrl)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
               that.onBridgeReady(data.var.appId, data.var.timeStamp, data.var.nonceStr, data.var.package, data.var.paySign);
            }else {
              that.isApply = false;
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
            }
          });
        }else {
          /*that.$dialog.loading.open('请求数据中...');
          setTimeout(()=>{
            that.$dialog.loading.close();
            that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:ok";
            that.payOrOrderDetail(1);
          },1000);*/
          //dealType = 1;
          var formData = {
            func:function(){
              that.payOrOrderDetail(1);
            },
            cancel:function(){
              that.isApply = false;
              console.log("取消了支付......");
              that.$dialog.notify({mes: '您已取消支付', icon: 'error', timeout: 1000});

              that.log(LOGCODE.ACTION.BTN_PAGE_ACTIONPRODUCTDETAIL_PAY_CANCEL);

            },
            data:{"paytype":2, "paymoneytype": that.payOpt.payType},// 2表示商品, this.payType: 原价、会员价、会员积分价
            forms:{"orgCode": window.orgCode, "orderNo": that.payOpt.orderNumber, "payMoney": that.payOpt.payMoney, "vipCard":"xx"}
          }
          that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:ok";
          this.$store.dispatch('simulationPay', formData);
        }
      },
      onBridgeReady(appId, timeStamp, nonceStr, packageValue, paySign ) {
        //调用微信支付（弹窗支付）
        var that = this;
        WeixinJSBridge.invoke(
         'getBrandWCPayRequest', {
             "appId":appId,     //公众号名称，由商户传入
             "timeStamp":timeStamp,         //时间戳，自1970年以来的秒数
             "nonceStr":nonceStr, //随机串
             "package":packageValue,
             "signType":"MD5",         //微信签名方式：
             "paySign":paySign //微信签名
         },function(res) {
             if(res.err_msg == "get_brand_wcpay_request:ok" ) {
                that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:ok";
                that.optsPayOrOrderDetail.payTime = timeStamp;
                that.payOrOrderDetail(1);
             }else if(res.err_msg == "get_brand_wcpay_request:cancel" ) {
                 that.$router.replace({
                   path: '/obligations',
                   query: {
                     type: 2,
                     mark: 'end'
                   }
                 });
             }else if(res.err_msg == "get_brand_wcpay_request:fail" ) {
                 that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:fail";
                 that.optsPayOrOrderDetail.payTime = timeStamp;
                 that.payOrOrderDetail(1);
                //that.$router.replace('/payerror');
             }
             if(window.wxdebug == true) {
                that.$dialog.alert({mes: JSON.stringify(res)});
             }
            that.isApply = false;
             // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。
         }
        );
      },
      payOrOrderDetail(dealType) {
        //支付提交
        let that = this;
        that.optsPayOrOrderDetail.dealType = dealType;
        that.optsPayOrOrderDetail.orderId = that.payOpt.orderId;
        that.optsPayOrOrderDetail.payMoney = that.payOpt.payMoney;
        that.optsPayOrOrderDetail.payType =  that.payOpt.payType;
        that.optsPayOrOrderDetail.serialNumber = that.payOpt.orderNumber;
        that.$dialog.loading.open('提交数据中...');
        console.log("payOrOrderDetail=", that.optsPayOrOrderDetail);
        that.$http.post(that.apiPayOrOrderDetailUrl,that.optsPayOrOrderDetail)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
              //商城过来的订单回到商城商品详情页
              if(that.$route.query.type=='shop'){
                that.$router.back();
              }else{
                //跳去支付详情
                that.$router.replace({
                  path: '/actionOrderInfo',
                  query: {
                    id: that.optsPayOrOrderDetail.orderId
                  }
                });
              }
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
          },(response) => {
            that.$dialog.loading.close();
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      }, 
       
      fetchProductData(cb) {
        //获取详情数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl,that.opts)
          .then((response) => {
            const data = response.data;
            if(data.code=='S_OK'){
              that.show2 = true;
              that.buyFlag = data.var.buyFlag;
              that.productDetail = data.var.productInfo;
              that.structureType = data.var.structureType;
              that.serviceCode = data.var.serveCodes&&data.var.serveCodes.length>0?data.var.serveCodes[0].split('_')[0]:'product';
            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/actionLogin');
              }});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
            if (typeof cb == 'function') {
              cb();
            }
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>