<template>
  <section class="questionnaire">
    <div class="bg">
      <div class="top">
        <p class="top-p">友道会代步车调研问卷</p>
      </div>
      <div>
        <div :id='"start"+(index+1)' class="start" v-for="item,index in timuOptionArray">
          <div class="start-top">
            <p class="start-title"><span v-html="item.title"></span><span class="red"> *</span></p>
            <p class="start-prompt" v-if="item.answerOption == ''">这道题必须回答哦</p>
          </div>
          <div class="start-bottom">
            <div class="start-option" :class="{'start-option-select': item.answerOption == valueItem.key}" v-for="valueItem in item.valueList" @click="choiceBtn(index,valueItem.key)">
              <span :class="{'check':item.answerOption == valueItem.key,'no': item.answerOption != valueItem.key}">&nbsp;</span>
              <span class="start-option-name">{{valueItem.value}}</span>
            </div>
          </div>
        </div>
        <button class="submit" @click="submit">提交</button>
      </div>
    </div>
  </section>
</div>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,getStore,setStore,getSessionStore,removeSessionStore} from '../../utils/assist';
import {codeKeyValue} from '../../utils/errorCode';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default { 
  data() {
    return {
        title: '代步车调研问卷',
        storeId:{},
        timuOptionArray:[
          {
            timuNumber:"001",answerOption:"",title:'1.您平均每天工作或上下班开车里程数约多少公里？',
            valueList:[
              {key:'A',value:'30公里'},
              {key:'B',value:'50公里'},
              {key:'C',value:'100公里'},
              {key:'D',value:'100公里以上'}
            ]
          },
          {
            timuNumber:"002",answerOption:"",title:'2.如果受开四停四影响,您是否考虑寻找代步车?(有道集团提供原车停放和免费检测服务)',
            valueList:[
              {key:'A',value:'是'},
              {key:'B',value:'否'}
            ]
          },
          {
            timuNumber:"003",answerOption:"",title:'3.如果电动代步车的租金是燃油代步车的一半,是否愿意选择电动代步车?(有道集团提供充电解决方案)',
            valueList:[
              {key:'A',value:'是'},
              {key:'B',value:'否'}
            ]
          },
          {
            timuNumber:"004",answerOption:"",title:'4.您能接受的电动代步车最低续航里程是多少公里?',
            valueList:[
              {key:'A',value:'200公里'},
              {key:'B',value:'250公里'},
              {key:'C',value:'300公里'},
              {key:'D',value:'400公里以上'}
            ]
          },
          {
            timuNumber:"005",answerOption:"",title:'5.您能接受的电动代步车一天租金在多少钱内?(不含充电费以及不含司机)',
            valueList:[
              {key:'A',value:'100元-150元'},
              {key:'B',value:'150元-200元'},
              {key:'C',value:'200元-250元'},
              {key:'D',value:'250元-300元'}
            ]
          },
          {
            timuNumber:"006",answerOption:"",title:'6.您能接受的燃油代步车一天租金在多少钱内?(不含充油料费以及不含司机)',
            valueList:[
              {key:'A',value:'200元-250元'},
              {key:'B',value:'250元-300元'},
              {key:'C',value:'300元-350元'},
              {key:'D',value:'350元-400元'}
            ]
          },
          {
            timuNumber:"007",answerOption:"",title:'7.如果会员包月的代步车(限天数)的价格比单次的价格更优惠,您愿意包月还是按天/次租用?',
            valueList:[
              {key:'A',value:'包月'},
              {key:'B',value:'按天'},
              {key:'C',value:'按次'},
              {key:'D',value:'4天'}
            ]
          },
          {
            timuNumber:"008",answerOption:"",title:'8.您希望可提供的代步车是?',
            valueList:[
              {key:'A',value:'两厢车'},
              {key:'B',value:'三厢车'}
            ]
          },
        ]
    }
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
  },
   
  methods: {
     
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      var _store = getSessionStore("information_selected_store",true);
      if(typeof(_store) != 'boolean'){
        this.storeOpt = _store;
      }

      this.$store.dispatch('log', {account:LOGCODE.SURVEY.PAGE_QUESTIONNAIRE});
    },
    scrollTo:function(obj){
      window.scrollTo(0,obj.offsetTop-10);
    },
    choiceBtn: function(pIndex,pValue){
      console.log('index',pIndex,pValue);
      this.timuOptionArray[pIndex].answerOption = pValue;
    },
    submit() {
      let that = this;
      let isRequest = true;
      for(var i=0;i<8;i++){
        if(this.timuOptionArray[i].answerOption == ''){
          this.scrollTo(document.getElementById('start'+(i+1)));
          isRequest = false;
          this.$dialog.notify({mes: '请勾选所有项', icon: 'error', timeout: 2000});
          break;
        }
      }
      if(!isRequest){
        return false;
      }
      let opt = {
        orgId:this.storeOpt.id,
        timuOptionArray: this.timuOptionArray
      };
      this.$store.dispatch('log', {account:LOGCODE.SURVEY.BTN_PAGE_QUESTIONNAIRE_SUBMIT_APPLY});
      ajaxModel.wxSubmitQuestionnaire(this,(result)=>{
        this.$store.dispatch('log', {account:LOGCODE.SURVEY.MESS_PAGE_QUESTIONNAIRE_SUBMIT_APPLY_OK});
        if(result.receiveStatus==1){//已登录
          var _phone = getSessionStore('userInfoPhone',false);
          ajaxModel.wxReceiveReward(that,(result)=>{
            if(result){
              that.$router.push({
                path: '/surveyStatu',
                query:{
                  statu:'3',
                  conponId: result
                }
              });
            }
          },{phone:_phone})
        }else{
          that.$router.push({
            path: '/surveyLogin'
          });
        }
      },opt)
    }
    
     
  }
}
</script>
<style lang="less">
  @import "../../styles/common/survey.less";
</style>
