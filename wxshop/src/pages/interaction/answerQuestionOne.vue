<template>

  <section class="ck-doc answerQuestion">
      <div class="question_mess i_flex i_border_tommon">
         <div class="T5C2">问题：</div>
         <div class="T5C3">{{answerData.questionCtx}}</div>
      </div>
      <div class="answer_content i_border_tommon">
        <textarea type="text" maxlength="200" v-model="answerData.answerCtx" class="T5C3" placeholder="题主正着急等待捷达，快写下你的回答吧！"></textarea>
      </div>
      <div class="answer_up_image">
        <div class="img_logo"><i>&nbsp;</i></div>
        <ul class="img_li i_flex_align_center">
          <li v-for="imgUrl,index in imgList">
            <img :src="imgUrl" />
            <i class="delete_icon" @click="deleteImg(index)">&nbsp;</i>
          </li>
          <li class="up_img_btn" v-if="isShowUpBtn">
            <form id="input-file" enctype="multipart/form-data" :action="apiUploadFileUrl" method="post" target="uploadIframe">
              <input type="text" name="token" v-show="false" id="token" :value="answerData.token">
              <i class="up_img_logo">&nbsp;</i>
              <input type="file" name="upload" @change="readFile" id="input_file" accept="image/*" />
            </form>
          </li>
        </ul>
      </div>
    <div class="interaction_btn2 T3C2" @click="nextAnswer">下一步</div>
    <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
  </section>

</template>

<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '回答',
        tips4:false,
        imgList:[],
        isShowUpBtn:true,
        imgBaseUrl: window.imgBaseUrl,
        apiUploadFileUrl: window.loginRegisterUrl+'/upload/uploadImg',
        answerData: {answerCtx:''}//活动信息对象
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
      var _questionDetailData = getSessionStore('questionDetailData',true);
      if(typeof(_questionDetailData) != 'boolean'){
        this.answerData = ydFunLib.ydExtend(this.answerData,_questionDetailData,true);
        if(this.answerData.images){
          this.imgList = this.answerData.images;
        }
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_ANSWERQUESTIONONE});
      },
      //删除图片
      deleteImg(pIndex) {
        this.imgList.splice(pIndex,1);
      },
      nextAnswer() {
        //跳去下一步
        console.log(this.answerData,'formData11');
        if(this.answerData.answerCtx==''){
          this.$dialog.notify({mes: '回答内容不能为空！', icon: 'error', timeout: 1200});
          return;
        }else if(this.answerData.answerCtx.length<10){
          this.$dialog.notify({mes: '回答内容过短！', icon: 'error', timeout: 1200});
          return;
        }
        this.answerData.images = this.imgList;
        setSessionStore('questionDetailData',this.answerData);
        this.$router.push({ path: 'maintainParts',query:{type:this.$route.query.type}});
      },
      //获取本地图片信息
      readFile(type,n,str){ 
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