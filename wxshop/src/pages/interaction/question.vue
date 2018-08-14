<template>

  <section class="question">
      <!-- 车辆选择 -->
      <ul class="cur_user_carts i_flex_justify_center" v-if="cartList && cartList.length>0">
         <li v-for="carts,index in cartList" @click="cartBtn(carts)" :class="{'pa_left':index==0}">
           <div :class="{'cart_item i_flex_justify_center':applyOpt.carId != carts.carId,'cart_item i_flex_align_end cur_btn':applyOpt.carId == carts.carId}">
              <div class="left">
                <div><i class="cart_logo" :style="{backgroundImage: 'url('+carts.carLogo + ')'}">&nbsp;</i></div>
                <p>{{carts.name}}</p>
              </div>
              <div class="rigth" v-if="carts.carId == applyOpt.carId">
                <div class="T6C2">{{carts.distance}}km</div>
                <div class="edit_cart T7C7" @click="openTip(carts.id)">编辑</div>
              </div>
           </div>
         </li>
      </ul>
      <div class="qu_content i_border_tommon">
        <textarea type="text" maxlength="200" v-model="applyOpt.questionCtx" class="T5C3" placeholder="请详细描述您爱车出现问题的时间和具体现象；添加图片会得到更准确的解答哦，例：从7天前开始，每天启动车子时车头部都会出现金属撞击异响，气温越低越严重。"></textarea @focus="handleFocus" v-on:blur="changeInput">
        <ul class="up_quest_img i_flex_align_center">
          <li v-for="imgUrl,index in imgList" v-if="imgList &&imgList.length>0">
            <img :src="imgUrl" />
          </li>
           <li v-if="isShowUpBtn">
            <form id="input-file" enctype="multipart/form-data" :action="apiUploadFileUrl" method="post" target="uploadIframe">
              <input type="text" name="token" v-show="false" id="token" :value="interactionOpt.token">
              <i class="q_img_btn">&nbsp;</i>
              <input type="file" class="up_file_input" name="upload" @change="readFile" id="input_file" accept="image/*" />
            </form>
           </li>
        </ul>
      </div>
      <!-- 其它选项 -->
      <div class="other_data T5C3">
        <div class="te">
         <!--  <div class="i_flex_align">
            <div><i class="i_logo a_cart_logo">&nbsp;</i><span>添加车辆</span></div>
            <div><i class="i_right_logo">&nbsp;</i></div>
          </div> -->
          <div class="i_flex_align" @click="goTo('tags')">
            <div><i class="i_logo a_tages_logo">&nbsp;</i><span>选择标签</span></div>
            <div>
              <span class="T6C5" v-if="applyOpt.tags !=''">{{applyOpt.tags}}</span>
              <span class="T6C5" v-else>选择问题标签</span>
              <i class="i_right_logo">&nbsp;</i>
            </div>
          </div>
          <div class="i_flex_align no_bor" @click="goTo('masterWorker')">
            <div><i class="i_logo a_shi_logo">&nbsp;</i><span>邀请师傅回答</span></div>
            <div>
              <span class="T6C5" v-if="applyOpt.inviterName!=''">{{applyOpt.inviterName}}</span>
              <span class="T6C5" v-else>选择师傅</span>
              <i class="i_right_logo">&nbsp;</i>
            </div>
          </div>
        </div>
      </div>
    <div class="interaction_btn3 T1C7" @click="submit">提交</div>

    <yd-popup v-model="tips1" position="center" width="80%">
      <div class="updata_carts_tip">
        <div class="o_close" @click="closeTip"><i>&nbsp;</i></div>
        <p class="T1C3">请输入当前里程</p>
        <div class="c_input i_flex_justify_center">
          <span class="T1C2">l</span>
          <input type="number" v-model ="applyOpt.mileage" />
          <span class="T1C2">km</span>
        </div>
        <div class="updata_btn" @click="updataCartSubmit">确定</div>
      </div>
     
    </yd-popup>
    <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>

    <iframe id="apiUploadFileUrl" name="apiUploadFileUrl" style="display:none;"></iframe>
  </section>

</template>

<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,setStore,getStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '提问',
        tips1:false,
        tips4:false,
        imgList:[],
        isShowUpBtn:true,
        interactionOpt:{},
        tagsList:['故障','保养'],
        masterWorkerList:[],
        cartList: [],
        apiQuestionSubmitUrl:window.interactionBaseUrl+ '/question/add?token=',
        imgBaseUrl: window.imgBaseUrl,
        apiUploadFileUrl: window.interactionBaseUrl+'/upload/uploadImg2',
        // apiUploadFileUrl:'http://192.168.2.17/platform/upload/uploadImg2',
        applyOpt:{
          memberId:'',
          questionCtx:'',
          carId:'',
          mileage:'',//车辆里程
          images:'',
          tags:'',//问题标签
          // inviter:'',//被要求回答问题的人Id
          inviterName:'',//邀请回答的人名称，多个逗号隔开，格式为:@xx,@xx
          inviterList:''//邀请推荐回答的师傅id，最多3个，逗号分隔
        }
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由0.005rem助初始化）
      'answerData.answerCtx'(){
        var _text = this.answerData.answerCtx.replace(/(^\s*)|(\s*$)/g, "");
        this.answerData.answerCtx = _text;
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      var _questionOptData = getSessionStore('questionSubmitData',true);
      if(typeof(_questionOptData) != 'boolean'){
        this.imgList = _questionOptData.images !=''?_questionOptData.images.split(','):[];
        this.applyOpt = _questionOptData;
      }
      var _interactionData = getStore('interactionData',true);
      if(typeof(_interactionData) != 'boolean'){
        this.interactionOpt = _interactionData;
        this.applyOpt.memberId = _interactionData.memberId;
      }
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
       
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_QUESTION});
      },
       //登录
      getLoginData(n) {
        var that = this;
        var _type = n;
        ydFunLib.interactionLogin(function(data){
          console.log('data',data);
            that.interactionOpt.memberId = data.memberId;
            that.interactionOpt.token = data.token;
            setStore('interactionData',that.interactionOpt);
        })
      },
      //删除图片
      deleteImg(pIndex) {
        this.imgList.splice(pIndex,1);
      },
      closeTip() {
        this.tips1 = false;
      },
      openTip(str) {
        this.tips1 = true;
      },
      setSessionFun(){
        this.applyOpt.images = this.imgList.join(',');
        setSessionStore('questionSubmitData',this.applyOpt);
      },
      updataCartSubmit(){
        if(this.applyOpt.mileage==''){
          this.$dialog.notify({mes: '请输入车辆里程！', icon: 'error', timeout: 1200});
          return;
        }
        for(var i=0;i<this.cartList.length;i++){
          if(this.cartList[i].carId==this.applyOpt.carId){
            this.cartList[i].distance = this.applyOpt.mileage;
            break;
          }
        }
        this.closeTip();
      },
      goTo(str) {
        //跳去下一步
        this.setSessionFun();
        this.$router.push({ path: str});
      },
      //车辆点击
      cartBtn(opt){
        this.applyOpt.carId = opt.carId;
        this.applyOpt.mileage = opt.distance;
      },
     
      submit() {
        let that = this;
        if(this.applyOpt.questionCtx==''){
          this.$dialog.notify({mes: '提问内容不能为空！', icon: 'error', timeout: 1200});
          return;
        }else if(this.applyOpt.questionCtx.length<10){
          this.$dialog.notify({mes: '提问内容过短！', icon: 'error', timeout: 1200});
          return;
        }
        if(this.imgList.length>0){
          this.applyOpt.images = this.imgList.join(',');
        }
        if(this.applyOpt.tags==''){
          this.$dialog.notify({mes: '请选择问题标签！', icon: 'error', timeout: 1200});
          return;
        }
        this.applyOpt.memberId = this.interactionOpt.memberId;
        this.applyOpt.token = this.interactionOpt.token;
        var _url = this.interactionOpt.token+ '&memberId='+this.interactionOpt.memberId+'&images='+this.applyOpt.images+'&questionCtx='+this.applyOpt.questionCtx+'&tags='+this.applyOpt.tags+'&inviterName='+this.applyOpt.inviterName+'&inviterList='+this.applyOpt.inviterList+'&carId='+this.applyOpt.carId+'&mileage='+this.applyOpt.mileage;

        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_QUESTION_APPLY});
        this.$http.post(this.apiQuestionSubmitUrl+_url)
          .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "s_ok") {
            that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_QUESTION_APPLY_OK});
            that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 2000 ,callback: () => {
              this.$router.push({ 
                path: '/interactionHome'
              });
            }});
          }else {
            that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_QUESTION_APPLY_FAIL});
            that.$dialog.notify({mes: '发布失败', icon: 'error', timeout: 1200});
          }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      },
      readFile(){
        let that = this;
        var formDOM = document.getElementById('input-file');
        //将form的DOM对象当作FormData的构造函数
        var formData = new FormData(formDOM);
        console.log(formData,'formData');
        var req = new XMLHttpRequest();
        this.$dialog.loading.open('上传中...');
        this.tips4 = true;
        req.open("POST", window.interactionBaseUrl+'/upload/uploadImg2');
        //请求完成回调
        req.onload = function(){
            if(this.status === 200){
              var _data = JSON.parse(this.response);
              that.imgList.push(_data.result[0]);
              that.isShowUpBtn = that.imgList.length==6? false : true;//最多可上传6张图片
            }
            that.$dialog.loading.close();
            that.tips4 = false;
         }
         //将form数据发送出去
        req.send(formData);
　　　　　　 //避免内存泄漏
　　　　　req = null; 
      }
    }
    
  }
</script>