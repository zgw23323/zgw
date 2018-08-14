<template>
  <!-- 家庭律师 -->
	<section class="service dj" v-if="serveCode=='JTLAWYER'">
    <article class="header">
      <div class="familyLawyer_bg serviceDe_con">&nbsp;</div>
    </article>
    <article class="content t_lawyer">
      <p class="s_dj_title">家庭律师服务是提供专业的法律咨询服务。可无限次为车主及其家人提供交通出行，婚姻、财产等方面的专业家庭法律电话咨询及文书审查服务。</p>
      <div class="s_dj_trait ser_lawyer">
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
    </article>
    <div class="f_btn">
      <div class="dj_btn one"><router-link :to="{path:'contract',query:{'code':serveCode}}">合同审核</router-link></div>
      <div class="dj_btn"><a href="tel:4009925855">拨号</a></div>
    </div>
  </section>
  <!-- 国际驾照 -->
  <section class="service dj" v-else-if="serveCode=='GJLICENSE'">
    <article class="header">
      <div class="drivingLicense_bg serviceDe_con">&nbsp;</div>
    </article>
    <article class="content t_roadside">
      <p>国际驾照服务是依据《联合国道路交通公约》并严格按照国际标准将中国驾照翻译成9国语言的标准驾照翻译文件。</p>
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
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>国际驾照须知：</p>
        <ul>
          <li>1、只能使用一次</li>
        </ul>
      </div>
    </article>
    <div class="d_call_phone">
      <router-link :to="{path:'drivingLicenseSubmit',query:{'code':serveCode}}">资料提交</router-link>
    </div>
  </section>
  <!-- 道路救援 -->
  <section class="service dj" v-else-if="serveCode=='DLASSISTANCE'">
    <article class="header">
      <div class="roadside_bg serviceDe_con">&nbsp;</div>
    </article>
    <article class="content t_roadside">
      <p>道路救援服务是汽车道路紧急救援，包括：拖车、搭电、换胎、送油服务。</p>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、无限次拖车服务-全国50KM免费</li>
          <li>2、无限次搭电服务-全国免费</li>
          <li>3、无限次理换轮胎-全国免费</li>
          <li>4、无限次紧急送油-全国免费</li>
        </ul>
      </div>
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>道路救援须知:</p>
        <ul>
          <li>1、拖车服务：不包括交通部门或其他部门规定限制第三方道路救援活动的路段以及在高速公路、隧道、大桥、高架道路等收费路段；车辆在拖起放下过程中，通过隧道、大桥、高架道路等收费路段、所有通行费用需由会员承担。</li>
          <li>2、更换轮胎：会员须自行备足汽车正常备胎。需用轮胎螺丝钥匙或特殊轮胎更换工具的，会员需自备。</li>
          <li>3、送油服务：如因特殊情况无法提供桶装送油，则采用拖车将会员车辆拖至距离最近加油站进行燃油加注。车道路紧急救援，包括：拖车、搭电、换胎、送油服务。</li>
        </ul>
      </div>
    </article>
    <a href="tel:4009925855" class="s_call_phone">拨号</a>
  </section>
  <!-- 家庭医生 -->
  <section class="service dj" v-else-if="serveCode=='JTDOCTOR'">
    <article class="header">
      <div class="familyDoctor_bg serviceDe_con">&nbsp;</div>
    </article>
    <article class="content t_roadside">
      <p style="line-height:.48rem;">家庭医生服务是指为会员及其家人的日常伤、病、和医疗疑问提供解答与建议；日常遇到的各类就医疑问的解答等相关在线医疗服务。</p>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、一人购买、全家受益</li>
          <li>2、三甲医院全科医生</li>
          <li>3、7*24小时在线</li>
        </ul>
      </div>
    </article>
    <div class="d_call_phone">
      <a href="tel:4009925855">拨号</a>
    </div>
  </section>
  <!-- 保养取送车 -->
  <section class="service dj" v-else-if="serveCode=='BYTAKEOUTANDPICK' || serveCode=='BYTAKECAR'">
    <article class="header">
      <div class="maintain_bg serviceDe_con">&nbsp;</div>
    </article>
    <article class="content t_roadside" style="padding-bottom: 2.5rem;">
      <p style="line-height:.48rem;">保养取送车服务是特定为会员打造的一款爱车助理产品。只需一个电话，我们即可为车主提供不限公里数和不限次数的保养代预约以及上门取送车服务。这款产品能够帮助车主免去保养爱车的往返奔波之苦，有更多的时间、可以投入到事业和家庭中，让生活变得更加美好。</p>
      <div class="s_dj_trait">
        <div class="s_dj_trait">
          <p><i class="l_trait"></i>服务特点</p>
          <ul>
            <li>1、覆盖全国298个重点城市；</li>
            <li>2、25万+名专业认主的优质司机；</li>
            <li>3、免去车主来回路上的时间；</li>
            <li>4、无需在4S店里等待保养过程、省时省力；</li>
            <li>5、高额投保、省心安心；</li>
          </ul>
        </div>
        <div class="s_dj_instructions">
          <p><i class="l_instructions"></i>服务须知:</p>
          <ul>
            <li>1、预约服务时间是每个工作日的9:00-17：00；</li>
            <li>2、请提前3个工作日进行预约；</li>
            <li>3、取送车服务，车辆的取车地点和送车地点须保持一致；</li>
            <li>4、本产品的服务对象是车辆的保养业务，不包含事故车业务；</li>
            <li>5、取送车服务仅针对车主获取本项服务的4S店；</li>
            <li>6、享受送车服务的车辆，必须是事先登记过的车辆（以车牌号码或车架号为准），若取车或送车时，车辆与事先登记的不一致，则无法提供相关服务；</li>
          </ul>
        </div>
      </div>
    </article>
    <div class="f_btn">
      <div class="dj_btn one"><router-link :to="{path:'maintainSubmit',query:{code:serveCode}}">提交资料</router-link></div>
      <div class="dj_btn"><a href="tel:4009925855">拨号</a></div>
    </div>
  </section>
  <!-- 其它有服务次数列表的服务项 -->
  <section class="service dj ser_info" v-else>
    <article class="header">
      <div class="s_dj" v-for="service in serviceList" v-if="serveCode !='GLASSSUPPORT'">
        <div class="top">
          <span>{{service.serveName}}</span>
          <router-link :to="{path:'serviceDetail',query:{'serveCode':serveCode,'pid':service.productId,'psource':1}}">去购买</router-link>
        </div>
        <div class="bottom">
          <p v-if="service.serveType==1">服务次数：无限次</p>
          <p v-else="service.serveType==0">服务次数：{{service.totalCount}}次</p>
          <p v-if="service.serveType==1 && serveCode == 'GTVIPROOM'">剩余次数：无限次</p>
          <p v-else-if="serveCode == 'GTVIPROOM'">剩余次数：{{service.remainCount}}次</p>
          <p>有效期：{{service.effBegintimeString}} - {{service.effEndtimeString}}</p>
        </div>
      </div>
      <div class="s_dj g_service" v-for="(service,index) in serviceList" v-if="serveCode =='GLASSSUPPORT'">
        <div class="g_top">
          <span>用户信息</span>
          <div class="g_top_item" v-if="service.isShow" @click="showHideBtn(index)">收起<i class="up_icon" >&nbsp;</i></div>
          <div class="g_top_item" v-if="!service.isShow" @click="showHideBtn(index)">展开<i class="down_icon">&nbsp;</i></div>
        </div>
        <div class="bottom" v-if="service.isShow">
          <ul>
            <li>车主姓名：{{service.customerName}}</li>
            <li>手机号：{{service.phone}}</li>
            <li>车牌号：{{service.licensePlate}}</li>
            <li>车架号：{{service.frameNumber}}</li>
            <li>已服务数：{{service.usedCount}}次</li>
            <li>有效日期：{{service.startDateString}} 至 {{service.endDateString}}</li>
          </ul>
        </div>
      </div>
    </article>
    <!-- 高铁贵宾厅 -->
    <article class="content" v-if="serveCode=='GTVIPROOM'">
      <div class="s_vipRoom">
        <p>查询网点：</p>
        <router-link :to="{path:'vipRoomDetail'}" class="s_address">
          <input type="text" placeholder="请选择服务站点" name="" />
          <i class="right">&nbsp;</i>
        </router-link>
      </div>
      <div class="s_vipRoom">
        <p>使用方法：</p>
        <ul>
          <li>进入高铁贵宾厅后出示本况换券及密码即可。</li>
        </ul>
      </div>
      <div class="s_vipRoom">
        <p>特别提醒：</p>
        <ul>
          <li>1、兑换码领取后48小时内有效、过期作废。</li>
          <li>2、非春运期间高铁贵宾厅使用时间为3小时；春运期间高铁贵宾厅使用时间为1小时。</li>
        </ul>
      </div>
    </article>
    <!-- 代驾服务 -->
    <article class="content" v-else-if="serveCode=='DJSERVICES'">
      <p class="s_dj_title">代驾服务是指为会员提供不限公里数的免费代驾服务。</p>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、全国<span class="red">200</span>多个城市覆盖<router-link :to="{path: 'servicesCity'}" >查看城市</router-link></li>
          <li>2、<span class="red">24</span>小时服务</li>
          <li>3、<span class="red">不限</span>公里数</li>
        </ul>
      </div>
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>代驾须知:</p>
        <ul>
          <li>1、代驾服务地域范围只限同城，指在同一城区范围内；</li>
          <li>2、此服务只限会员本人使用；</li>
          <li>3、服务范围请参考服务覆盖区域列表、实行点对点服务。</li>
        </ul>
      </div>
    </article>
    <!-- 专享陪诊 -->
    <article class="content" style="padding-bottom: 2.56rem;" v-else-if="serveCode=='ZXCLINIC'">
      <p class="s_dj_title">尊享陪诊由“就医帮手-绿色通道”和“就医帮手-陪诊”两项服务组合构成。就医帮手-绿色通道是为会员用户提供的一项医疗服务权益，它解决了专家门诊资源有限，经常连着几个月挂不上号这一就医难题，帮助会员快速预约，节约时间成本，并且能够保障专家服务。就医帮手-陪诊是由专职的陪诊员在会员就诊时陪同，提供导诊咨询、并帮助会员排队取号、预约检查 、排队缴费、排队取药的全程门诊就医陪伴服务。通过绿色通道和陪诊服务、系统地解决了会员就医过程中的专家预约和陪护问题，既能帮助会员节约时间、保障专家服务，又能通过陪诊人员的全程陪同给予会员贵宾待遇。</p>
      <div class="s_vipRoom">
        <p>查询网点：</p>
        <router-link :to="{path: 'hospital',query: {type: '2'}}" class="s_address">
          <input type="text" placeholder="请选择服务医院" name="" />
          <i class="right">&nbsp;</i>
        </router-link>
      </div>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、快速预约，专人陪诊；</li>
          <li>2、节省时间，提高效率；</li>
          <li>3、覆盖全国29个城市；</li>
        </ul>
      </div>
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>服务须知:</p>
        <ul>
          <li>1、请提前7个工作日以上预约；</li>
          <li>2、客户如指定医生，则优先预约该医生，如无法预约该医生则提供1-2名同级别或上上级别医生供客户选择；</li>
          <li>3、如因医生停诊或其他不可抗力因素，导致不能按时提供预约服务，我们会安排最近一次预约；</li>
          <li>4、如需取消请至少提前2个工作日，否则视为违约；同一身份证或医保卡违约2次，则次年不再提供预约专家服务；</li>
          <li>5、如指定医生预约则不可取消；</li>
          <li>6、预约时间为周一至周五9:00-18:00，周六、周日及国家法定节假日不接受预约和陪诊；</li>
          <li>7、绿色通道和陪诊服务仅服务会员本人；</li>
        </ul>
      </div>
    </article>
    <!-- 绿色通道 -->
    <article class="content" style="padding-bottom: 2.56rem;" v-else-if="serveCode=='GREENCHANNEL'">
      <p class="s_dj_title">就医帮手-绿色通道是为会员用户提供的一项医疗服务权益，它解决了专家门诊资源有限，经常连着几个月挂不上号这一就医难题，帮助会员快速预约，节约时间成本，并且能够保障专家服务。</p>
      <div class="s_vipRoom">
        <p>查询网点：</p>
        <router-link :to="{path: 'hospital',query: {type: '2'}}" class="s_address">
          <input type="text" placeholder="请选择服务站点" name="" />
          <i class="right">&nbsp;</i>
        </router-link>
      </div>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、服务范围覆盖全国94个城市、1549家知名医院；</li>
          <li>2、含571家三甲医院，北上广三甲医院全覆盖；</li>
          <li>3、8小时内闪电反馈；</li>
        </ul>
      </div>
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>服务须知:</p>
        <ul>
          <li>1、请提前7个工作日以上预约；</li>
          <li>2、客户如指定医生，则优先预约该医生，如无法预约该医生则提供1-2名同级别或上上级别医生供客户选择；</li>
          <li>3、如因医生停诊或其他不可抗力因素，导致不能按时提供预约服务，我们会安排最近一次预约；</li>
          <li>4、如需取消请至少提前2个工作日，否则视为违约；同一身份证或医保卡违约2次，则次年不再提供绿色通道服务；</li>
          <li>5、如指定医生预约则不可取消；</li>
          <li>6、预约时间为周一至周五9:00-18:00，周六、周日及国家法定节假日不接受预约；</li>
          <li>7、绿色通道仅服务会员本人；</li>
        </ul>
      </div>
    </article>
    <!-- 代配药 -->
    <article class="content" style="padding-bottom: 2.56rem;" v-else-if="serveCode=='DISPENSING'">
      <p class="s_dj_title">就医帮手-代配药是为会员打造的一项医疗服务权益。针对因行动不便、工作忙碌等原因无法亲自去医院配药的会员，我们派专门的服务人员替会员到医院配药。这一服务为会员节约了往返医院并且排队的时间，同时，又能及时地配到所需要的药品，不至于耽误治疗。</p>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、覆盖全国29个城市</li>
          <li>2、快速办理、节省时间</li>
        </ul>
      </div>
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>服务须知:</p>
        <ul>
          <li>1、需按照医保、医院规定（配药金额、数量等）执行；</li>
          <li>2、患者需要在此医院有配药处方记录；</li>
          <li>3、代配药仅服务会员本人；</li>
        </ul>
      </div>
    </article>
    <!-- 玻璃保障服务 -->
    <article class="content t_roadside" style="padding-bottom:3.7rem;" v-else="serveCode=='GLASSSUPPORT'">
      <p>玻璃保障服务是为客户车辆前、后挡风玻璃或四门车窗的单独破碎，提供玻璃免费更换服务。</p>
      <div class="s_dj_trait">
        <p><i class="l_trait"></i>服务特点</p>
        <ul>
          <li>1、4S店配备100%原产玻璃</li>
          <li>2、每年服务价格一致，不会因为您的保修次数而增加</li>
          <li>3、1年无限次数更换玻璃</li>
          <li>4、4S店高级技工专业服务</li>
        </ul>
      </div>
      <div class="s_dj_instructions">
        <p><i class="l_instructions"></i>服务须知：</p>
        <ul>
          <li>1、玻璃保障服务需提前预约购买，到4S店审核车辆玻璃情况才可成功购买此服务。</li>
          <li>2、玻璃保障服务与保险公司玻璃单独破碎服务性质一致</li>
          <li>3、单独破碎是指无其他部位损坏，比如撞车连带玻璃一起破碎的情况不属于此服务更换范围内。</li>
        </ul>
      </div>
    </article>
    <!-- 设置底部按钮star -->
    <a href="tel:4009925855" class="s_call_phone" v-if="serveCode=='DJSERVICES' || serveCode=='DISPENSING'">拨号</a>
    <div class="d_call_phone" v-else-if="serveCode=='GTVIPROOM'">
      <router-link :to="{path:'vipRoomDetail'}">我要领取</router-link>
    </div>
    <!-- 玻璃保障服务按钮 -->
    <div :class="{'f_btn':!isNone,'f_btn scroll_padding_top':isNone}" v-else-if="serveCode=='GLASSSUPPORT'">
      <div class="g_buy_again"><router-link :to="{path:'serviceDetail',query:{'serveCode':'GLASSSUPPORT','psource':1}}">再次购买</router-link></div>
      <div class="dj_btn" @click="goToRouter">我要保修</div>
    </div>
    <div class="f_btn" v-else>
      <div class="dj_btn one"><router-link :to="{path:'medicalAssistantSubmit',query:{code:serveCode}}">我要预约</router-link></div>
      <div class="dj_btn"><a href="tel:4009925855">拨号</a></div>
    </div>
    <!-- 设置底部按钮end -->
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore} from '../../utils/assist';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title: '',
        serveCode:'',
        isShow:false,
        isNone:false,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsFindServeDetail?shopsid='+ window.shopsid,
        apiNewestRecord: window.baseUrl+window.appName+'/shopapi/func/insurance/wxMatchApplyInfo?shopsid='+ window.shopsid,
        serviceList: [{isShow:false}]
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.serveCode = this.$route.query.code;
      if(this.serveCode == "GLASSSUPPORT"){
        this.fetchServiceDetail();
      }
      this.markInit();
      
      document.body.scrollTop = document.documentElement.scrollTop = 0;
      document.addEventListener('scroll', this.scrollChange);
    },
    methods: {
      markInit() {
        //初始化页面标题
        var str = '';
        if(this.serveCode == "DJSERVICES"){
          str = "代驾服务";
          this.fetchServiceDetail();
        }else if(this.serveCode == "DLASSISTANCE"){
          str = "道理救援";
        }else if(this.serveCode == "JTLAWYER"){
          str = "家庭律师";
        }else if(this.serveCode == "JTDOCTOR"){
          str = "家庭医生";
        }else if(this.serveCode == "GJLICENSE"){
          str = "国际驾证/认证";
        }else if(this.serveCode == "GTVIPROOM"){
          str = "高铁贵宾厅";
          this.fetchServiceDetail();
        }else if(this.serveCode == "BYTAKECAR"){
          str = "保养取送车(单程)";
        }else if(this.serveCode == "BYTAKEOUTANDPICK"){
          str = "保养取送车(双程)";
        }else if(this.serveCode == "ZXCLINIC"){
          str = "就医帮手-尊享陪诊";
          this.fetchServiceDetail();
        }else if(this.serveCode == "DISPENSING"){
          str = "就医帮手-代配药";
          this.fetchServiceDetail();
        }else if(this.serveCode == "GREENCHANNEL"){
          str = "就医帮手-绿色通道";
          this.fetchServiceDetail();
        }else{
          str = "玻璃保障服务";
        }
        this.title = str;
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_SERVICEINFO});
      },
      goToRouter () {
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiNewestRecord,{'recordType':2})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              setSessionStore('plateNumberList',data.var.openFrameNumberList);
              if(data.var.isYuyue == 2){//没有预约记录
                data.var.historyInfo.customerName = data.var.historyInfo.name;
                data.var.historyInfo.frameNumber = data.var.openFrameNumberList.length>0?data.var.openFrameNumberList[0]:data.var.historyInfo.frameNumber;
                setSessionStore('glassSupportInfo',data.var.historyInfo);
                if(data.var.loginOrgId){
                  var storOpt = {
                    id: data.var.loginOrgId,
                    orgShortName: data.var.orgName,
                    mobilePhone: data.var.mobilePhone,
                    address: data.var.address
                  }
                  setSessionStore('serviceStore_btn_info',storOpt);
                } 
                that.$router.push({
                  path: '/glassSupport',
                  query: {'code':'GLASSSUPPORT'}
                });
              }else{//有
                that.$router.push({
                  path: '/glassSupportEdit',
                  query: {'id':data.var.recordId,'code':'GLASSSUPPORT',type:'1'}//1=查看信息，2=修改
                });
              }
            }else{
              if(data.code =="NO_OPEN_SERVICE"){
                that.$dialog.notify({mes: '您的玻璃保障服务还未生效', icon: 'error', timeout: 2500});
              }
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      //监听滚动条事件
      scrollChange(){
        var scrollTop = document.body.scrollTop;
    　　var wScroll = document.body.scrollHeight;
    　　var wHeight = document.body.offsetHeight;
    　　if(scrollTop + wHeight == wScroll){
    　　　this.isNone = true;
    　　}else{
          this.isNone = false;
        }
      },
      //服务展示收缩按钮
      showHideBtn(pIndex){
        this.isShow = !this.isShow;
        this.serviceList[pIndex].isShow = !this.serviceList[pIndex].isShow;
      },
      fetchServiceDetail: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl,{'code':this.serveCode})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(that.serveCode == 'GLASSSUPPORT'){
                for(var i=0;i<data.var.length;i++){
                  data.var[i].isShow = false;
                }
                this.serviceList = data.var;
              }else{
                this.serviceList = data.var;
                for(var i=0;i<that.serviceList.length;i++){
                  var star = that.serviceList[i].effBegintimeString;
                  var end = that.serviceList[i].effEndtimeString;
                  that.serviceList[i].effBegintimeString = star.slice(0,4)+'.'+star.slice(5,7)+'.'+star.slice(8,10);
                  that.serviceList[i].effEndtimeString = end.slice(0,4)+'.'+end.slice(5,7)+'.'+end.slice(8,10);
                }
              }
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>