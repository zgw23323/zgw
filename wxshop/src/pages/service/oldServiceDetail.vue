 <template>
	<section class="service drivingServiceDetail serviceDetail details">
    <article class="header">
       <div class="details-hd">
          <div class="details-banner" v-if="serviceCode != 'GLASSSUPPORT'">
             <div class="service_conmon" v-lazy:background-image="imgBaseUrl+detailsData.prodPic" v-if="serviceCode != 'GLASSSUPPORT'">&nbsp;</div>
          </div>
          <div class="glass_GLASSSUPPORT" v-else>&nbsp;</div>
          <!-- 限时折扣 -->
          <div class='details-action' v-if="detailsData.restrict">
            <!-- 抢购中、即将开始 -->
            <div class="details-action-time action-common" v-if="actionStatu==1">
                <div class="de_ac_left"><span class="le_lo">&nbsp;</span><span class='le_mes'>抢购中</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acEndTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <div class="details-action-time action-common" v-if="actionStatu==3">
                <div class="de_ac_left"><span class="le_lo">&nbsp;</span><span class='le_mes'>即将开始</span></div>
                <div class="de_ac_right">
                   <p>距开始时间</p>
                   <yd-countdown class="s1" ref='countDown' :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acStarTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <!-- 活动已结束 -->
            <div class="details-action-over action-common" v-if="actionStatu==2">
                <div class="de_ac_left"><span class="le_lo">&nbsp;</span><span class='le_mes'>活动已结束</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="'0'" :countType="'1'" :time="'0'"  format=" {%h}:{%m}:{%s}"></yd-countdown>
                </div>
            </div>
          </div>
          <!-- 秒杀-->
          <div class='details-action' v-if="detailsData.skipe">
            <!-- 抢购中、即将开始 -->
            <div class="details-action-miao action-common" v-if="actionStatu==1">
                <div class="de_ac_left"><span class="le_lo_m">&nbsp;</span><span class='le_mes'>秒杀中</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acEndTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <div class="details-action-miao action-common" v-if="actionStatu==3">
                <div class="de_ac_left"><span class="le_lo_m">&nbsp;</span><span class='le_mes'>预约中</span></div>
                <div class="de_ac_right">
                   <p>距开始时间</p>
                   <yd-countdown class="s1" ref='countDown' :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acStarTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <!-- 活动已结束 -->
            <div class="details-action-over action-common" v-if="actionStatu==2">
                <div class="de_ac_left"><span class="le_lo_m_over">&nbsp;</span><span class='le_mes'>活动已结束</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="'0'" :countType="'1'" :time="'0'"  format=" {%h}:{%m}:{%s}"></yd-countdown>
                </div>
            </div>
          </div>
          <div class="details-mes con_mess" v-if="serviceCode != 'GLASSSUPPORT'">
            <!-- <div class="details-title" v-text="detailsData.prodTitle"></div> -->
            <p class="details-title" v-if="detailsData.serveType==1">{{detailsData.prodTitle}} (无限次)</p>
            <p class="details-title" v-else>{{detailsData.prodTitle}} <span v-if="detailsData.serveCount">({{detailsData.serveCount+'次'}})</span></p>
            <div class="details-other">
              <!-- <div class="details-price"> -->
              <div>
                <div v-if="detailsData.skipe==true && actionStatu != 2" class="p_red">￥{{detailsData.skipePrice}}</div>
                <div v-else-if="detailsData.restrict==true && actionStatu != 2" class="p_red">￥{{detailsData.discountPrice}}</div>
                <div class="p_red" v-else>￥{{detailsData.memberPrice}}</div>
                <div class="p_del list-del-price">市场价￥{{detailsData.orgPrice}}</div>
                <!-- 秒杀活动未开始 -->
                <span class='action-appointment' v-if='detailsData.skipe==true && actionStatu == 3'  @click='reserveBtn(detailsData.activityId)'>立即预约</span>
                <div class="ser_time_mess">有效期为一年</div>
              </div>
              <!-- 进度条 -->
              <div class="progress" v-if='(detailsData.skipe || detailsData.restrict) && actionStatu != 3'>
                  <div class="progress-bar" v-if="source == 2">
                      <span class="sr-only" :style="{'width': (detailsData.saleTotal/(detailsData.skipeStore+detailsData.saleTotal)*100).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                  </div>
                   <div class="progress-bar" v-if="source == 3">
                      <!-- <span class="sr-only" :style="{'width': (detailsData.saleTotal/detailsData.stocksCount*100).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span> -->
                      <span v-if="!detailsData.initStocks" class="sr-only" :style="{'width': (100-detailsData.stocksCount).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                      <span v-else-if="detailsData.stocksCount>0" class="sr-only" :style="{'width': (detailsData.initStocks-detailsData.stocksCount).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                      <span v-else class="sr-only" :style="{'width': '100%','max-width': '100%'}">&nbsp;</span>
                  </div>
                  <div class="progress-count" v-if='source == 2'>剩余{{detailsData.skipeStore}}</div>
                  <div class="progress-count" v-if='source == 3'>剩余{{detailsData.stocksCount}}</div>
              </div>
            </div>
          </div>
        </div>
    </article>
    <div class="details-action-menu" style="margin-bottom:.2rem;" v-if="detailsData.skipe==true || detailsData.restrict ==true || detailsData.springProdId">
          <!-- 秒杀中、秒杀结束限购提示 -->
          <p class="action-appointment-ing" v-if="detailsData.skipe==true && actionStatu != 3 && detailsData.restrictType ==2"><span class="action-mess no-mess">限购</span><span>每人限购{{detailsData.restrictCount}}件，超出不再享受秒杀价</span></p>
          <!-- 秒杀未开始预约提示 -->
          <div class='coupon-item action-miao' v-if="detailsData.skipe==true && actionStatu == 3">
             <span class="action-song no-r" >秒杀</span>
             <span>预计{{detailsData.startTime}}开始，请提前预约</span>
          </div>
           <!-- 领取优惠券 -->
          <!-- <div class='coupon-item' v-if="couponList.length>0"  @click='openTip(4)'>
             <span class="icon-coupon" >&nbsp;</span>
             <span class="">领取优惠券</span>
             <span class="right-btn">&nbsp;</span>
          </div> -->
          <div class='coupon-item i_flex_betwent' v-if="detailsData.springProdId">
            <div>
              <span class="icon-coupon" >&nbsp;</span>
              <span class="">一元领取代金券</span>
            </div>
            <router-link class="get_coupon" :to="{path:'actionProductDetail',query:{pid:detailsData.springProdId}}">领取</router-link>
          </div>
          <!-- 满减送详情 -->
  
          <div class='coupon-item' @click='getAppListReduced(3)' v-if="reducedInfo !=null">
             <span class="action-song" >满减送</span>
             <span class="">查看详情</span>
             <span class="right-btn">&nbsp;</span>
          </div>
          
      </div>
    <article class="content">
      <!-- 高铁贵宾厅详情 -->
      <div class="con_info"  v-if="serviceCode == 'GTVIPROOM'">
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
         
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“高铁贵宾厅”。</p>
              <img src="../../img/serviceImg/detail_5.png" class="d_img2" />
            </li>
            <li>
              <p>2、点击“我要领取”，系统会自动发送卡号与密码给您，您去贵宾厅找到该网店时，出示卡号即可享受服务。</p>
              <img src="../../img/serviceImg/detail_6.png" class="d_img2" style="width: 2.71rem;" />
            </li> 
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
        <div class="s_dj_instructions">
          <p><i class="l_instructions"></i>代驾须知:</p>
          <ul>
            <li>1、代驾服务地域范围只限同城，指在同一城区范围内；</li>
            <li>2、此服务只限会员本人使用；</li>
            <li>3、服务范围请参考服务覆盖区域列表、实行点对点服务。</li>
          </ul>
        </div>
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“代驾服务”。</p>
              <img src="../../img/serviceImg/detail_1.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“拨号”，连接人工客服，为您提供服务。查看自己购买服务的相关信息。</p>
              <img src="../../img/serviceImg/detail_2.png" class="d_img2" />
            </li> 
          </ul>
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
         
        <div class="s_dj_instructions">
          <p><i class="l_instructions"></i>道路救援须知:</p>
          <ul>
            <li>1、拖车服务：不包括交通部门或其他部门规定限制第三方道路救援活动的路段以及在高速公路、隧道、大桥、高架道路等收费路段；车辆在拖起放下过程中，通过隧道、大桥、高架道路等收费路段、所有通行费用需由会员承担。</li>
            <li>2、更换轮胎：会员须自行备足汽车正常备胎。需用轮胎螺丝钥匙或特殊轮胎更换工具的，会员需自备。</li>
            <li>3、送油服务：如因特殊情况无法提供桶装送油，则采用拖车将会员车辆拖至距离最近加油站进行燃油加注。车道路紧急救援，包括：拖车、搭电、换胎、送油服务。</li>
          </ul>
        </div>
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“道路救援”。</p>
              <img src="../../img/serviceImg/detail_3.png" class="d_img2" />
            </li>
            <li>
              <p>2、点击“拨号”，连线人工客户，为您提供服务。</p>
              <img src="../../img/serviceImg/detail_4.png" class="d_img2" />
            </li> 
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
         
        <div class="s_dj_instructions">
          <p><i class="l_instructions"></i>国际驾照须知：</p>
          <ul>
            <li>1、只能使用一次</li>
          </ul>
        </div>
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“国际驾照”。</p>
              <img src="../../img/serviceImg/detail_7.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“资料提交”，录入相关资料，即可提供相关服务。</p>
              <img src="../../img/serviceImg/detail_8.png" class="d_img2 img_r" style="width:2.37rem;margin-right:.95rem;" />
              <img src="../../img/serviceImg/detail_9.png" class="d_img2" style="width:2rem;" />
            </li> 
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
        
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“家庭律师”。</p>
              <img src="../../img/serviceImg/detail_10.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“拨号”，使用人工服务为您解答相关 法律问题；同时也可以点击“合同审查”，提交 相关文件，进行文书审查服务。</p>
              <img src="../../img/serviceImg/detail_11.png" class="d_img2 img_r" style="width:2.32rem;margin-right: .93rem;" />
              <img src="../../img/serviceImg/detail_12.png" class="d_img2" style="width:2.2rem;" />
            </li> 
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
        
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“家庭医生”。</p>
              <img src="../../img/serviceImg/detail_13.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“拨号”，连接人工客服，为您提供 服务。</p>
              <img src="../../img/serviceImg/detail_14.png" class="d_img2" style="width:2.66rem;" />
            </li> 
          </ul>
        </div>
      </div>
      <!-- 保养取送车 -->
      <div class="con_info" v-else-if="serviceCode == 'BYTAKEOUTANDPICK'||serviceCode == 'BYTAKECAR'">
        <p class="s_pro_title">商品详情</p>
        <p class="s_dj_title no_width" style="width: 99%;">保养取送车服务是特定为会员打造的一款爱车助理产品。只需一个电话，我们即可为车主提供不限公里数和不限次数的保养代预约以及上门取送车服务。这款产品能够帮助车主免去保养爱车的往返奔波之苦，有更多的时间、可以投入到事业和家庭中，让生活变得更加美好。</p>
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
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“保养取送车”。</p>
              <img src="../../img/serviceImg/detail_15.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“提交资料”，跳转到资料录入界面，输入相关信息，为您提供相应服务；点击“拨号”，连接人工客服，为您提供人工服务。</p>
              <!-- <img src="../../img/serviceImg/detail_16.png" class="d_img2" style="width:2.75rem;margin-right:.39rem;margin-top: .6rem;" /> -->
              <img src="../../img/serviceImg/detail_111.png" class="d_img2" style="width:2.75rem;margin-right:.39rem;margin-top: .6rem;" v-if="serviceCode == 'BYTAKEOUTANDPICK'" />
              <img src="../../img/serviceImg/detail_112.png" class="d_img2" style="width:2.75rem;margin-right:.39rem;margin-top: .6rem;" v-else />
              <img src="../../img/serviceImg/detail_17.png" class="d_img2" style="width:2.91rem;vertical-align: top;margin-top: .6rem;" />
            </li> 
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
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“绿色通到”。</p>
              <img src="../../img/serviceImg/detail_18.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“我要预约”，跳转到预约资料录入界面，输入相关信息，为您提供相应服务；点击“拨号”，连接人工客服，为您提供人工服务。</p>
              <img src="../../img/serviceImg/detail_19.png" class="d_img2" style="width:2.67rem;" />
              <img src="../../img/serviceImg/detail_20.png" class="d_img2" style="width:2.67rem;vertical-align: top;margin-left: .5rem;" />
            </li> 
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
        <div class="s_dj_instructions">
          <p><i class="l_instructions"></i>服务须知:</p>
          <ul>
            <li>1、需按照医保、医院规定（配药金额、数量等）执行；</li>
            <li>2、患者需要在此医院有配药处方记录；</li>
            <li>3、代配药仅服务会员本人；</li>
          </ul>
        </div>
        <div class="s_use_method">
          <p class="method_t"><i class="l_method"></i>使用方法:</p>
          <ul>
            <li>
              <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“代配药”。</p>
              <img src="../../img/serviceImg/detail_21.png" class="d_img1" />
            </li>
            <li>
              <p>2、点击“拨号”，连接人工客服，为您提供 服务。</p>
              <img src="../../img/serviceImg/detail_22.png" class="d_img2" style="width:2.98rem;" />
            </li> 
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
          <div class="s_use_method">
            <p class="method_t"><i class="l_method"></i>使用方法:</p>
            <ul>
              <li>
                <p>1、购买服务成功后，在微信公众号找到“会员服务中心” 进入会员中心，找到“尊享陪诊”。</p>
                <img src="../../img/serviceImg/detail_23.png" class="d_img1" style="width:2.99rem;" />
              </li>
              <li>
                <p>2、点击“我要预约”，跳转到预约资料录入界面，输入相关信息，为您提供相应服务；点击“拨号”，连接人工客服，即可享受该服务。</p>
                <img src="../../img/serviceImg/detail_24.png" class="d_img2" style="width:2.8rem;" />
                <img src="../../img/serviceImg/detail_25.png" class="d_img2" style="width:2.8rem;vertical-align: top;margin-left: .5rem;" />
              </li> 
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
      <div class="s_attention" v-if="serviceCode != 'GLASSSUPPORT'">注意：此服务购买成功后，24小时后即可使用</div>
    </article>
    <div class="s_btn" @click="buy" v-if="serviceCode != 'GLASSSUPPORT'">立即购买</div>
    <div class="s_btn" v-else><router-link :to="{path: 'appointmentBuy'}">预约购买</router-link></div>
    <!-- <yd-popup v-model="show2" position="bottom" class="height" :height="height"> -->
    <yd-bottom-tip v-if="show2" :tipsType='"show2"'>
      <div class="service_tip_head">
        <div>信息确认</div>
        <div><i class="s_close" @click="closeTip">&nbsp;</i></div>
      </div>
      <!-- <div :class="{'con noBou':isKeyUp,'con':!isKeyUp}"> -->
      <div class="con">
        <div class="item">
          <span>手机号：</span><input type="text" @focus="handleFocus" maxlength="11" v-on:blur="changeInput" v-model="personData.mobile" placeholder="请输入手机号" />
        </div>
        <div class="item">
          <span>用户名：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="personData.userName" placeholder="请输入用户名" />
        </div>
        <div class="item">
          <span>身份证：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="personData.idNumber" placeholder="请输入身份证" />
        </div>
        <div class="item">
          <span>车牌号：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="personData.plateNumber" placeholder="请输入车牌号" />
        </div>
        <div class="item">
          <span>车架号：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="personData.vinNumber" maxlength="vinlen" placeholder="请输入车架号" />
        </div>
      </div>
      <!-- <div :class="{'s_submit_btn dis':isKeyUp,'s_submit_btn':!isKeyUp}" @click="submit" v-if="!isApply">确定</div> -->
      <div class="s_submit_btn" @click="submit" v-if="!isApply">确定</div>
      <div class='s_submit_btn isApply' v-if="isApply">确定</div>
    </yd-bottom-tip>
    <!-- </yd-popup> -->

    <!-- 满减送弹窗 -->
    <yd-bottom-tip v-if="tips3" :tipsType='"tips3"'>
      <ul class="action-full-song">
        <li>活动名称：<span>{{appListReduced.title}}</span></li>
        <li class="item3" v-for="reduced in appListReduced.preferentialSets" v-if="reduced.setType==0"><em class="li-em">&nbsp;</em>满<span class="red">{{reduced.threshold}}</span>元，减<span class='red'>{{reduced.reducedMoney}}</span>元。</li>
         <li class="item3" v-for="reduced in appListReduced.preferentialSets" v-if="reduced.setType==1"><em class="li-em">&nbsp;</em>满<span class="red">{{reduced.threshold}}</span>元，送<span class='red'>{{reduced.methods[0].couponAmount}}</span>元优惠券。</li>
      </ul>
      <div class="ck-fixed-btn" @click='tipClose(3)'>完成</div>
    </yd-bottom-tip>
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
        tips3:false,
        height:'80%',
        isKeyUp:false,
        isApply:false,
        actionStatu:1,//1、活动开始；2、活动结束；3、活动即将开始
        optAction:{//限时购数据
          currTime:"",
          acStarTime:'',
          acEndTime:'',
          acformat:'{%d}:{%h}:{%m}:{%s}'
        },
        imgBaseUrl: window.imgBaseUrl,
        apiUserInfoUrl: window.baseUrl+window.appName+'/shopapi/func/serve/wxFindUserInfo?shopsid='+window.shopsid,
        apiUserInfoSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxConfirmUserInfo?shopsid='+window.shopsid,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/product/getProductInfo?orgCode='+window.orgCode,
        apiVinUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxGetVinLength?orgCode='+window.orgCode,
         // 获取满减送列表接口
        apiAppListReduced:window.baseUrl+window.appName+'/shopapi/func/reduced/appListReduced?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
        apiAddToOrderUrl: window.baseUrl+window.appName+'/shopapi/func/product/addToOrder?orgCode='+window.orgCode,
        myCart: {
          prodId: "",
          prodCount: 1
        },
        optsAddToOrder: {
          prodIds: [],
          genOrderSource: 2,
          isContinue: 2
        },
        source:'',
        appListReduced:{},
        reducedInfo:{},
        detailsData: {
          serveCount:''
        },
        personData: {
            userName: '',
            mobile: '',
            idNumber: '',
            plateNumber: '',
            vinNumber: ''
        },
        opts:{prodId:""},
        vinlen: 6,
        optDetail:{},
        appListReduced: {},//满减送
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
      this.fetchVinLength();
      if(this.serviceCode !="GLASSSUPPORT"){
        this.fetchProductData(this.log);
        // this.fetchVinLength();
      }
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
            sItem.itemid = this.myCart.prodId, 
            sItem.name = this.detailsData.prodName, 
            sItem.price = this.detailsData.memberPrice, 
            sItem.picid = this.detailsData.prodPic;
        mod = mod || LOGCODE.SERVICE.PAGE_SERVICEDETAIL;
        this.$store.dispatch('log', {account:mod, ShopItem: sItem});
      },
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.serviceCode = this.$route.query.serveCode;
        this.opts.prodId =  this.$route.query.pid;
        this.myCart.prodId = this.$route.query.pid;
        //获取商量列表进详情缓存基本信息
        this.source = this.$route.query.psource ? this.$route.query.psource :'1';
        var personData = getSessionStore('serviceSubmitInfo',true);//拿到缓存确认信息
        if(typeof(personData) !='boolean') {
          this.personData = personData;
        }

        var loginPhone = getStore('serviceInfoPhone',false);//拿到登录手机号
        if(typeof(loginPhone) !='boolean') {
          this.personData.mobile = loginPhone;
        }
        //判断设备，给定弹框高度
        var width_body = document.documentElement.clientWidth;
        var height_body = document.documentElement.clientHeight;
        if(width_body == 320 && height_body == 480){
            this.height = '91%';
        }
      },
      handleFocus(){
        this.isKeyUp = true;
        setTimeout(function () {//由于键盘弹出是有动画效果的，要获取完全弹出的窗口高度，使用了计时器
          var windowInnerHeight = window.innerHeight;//获取弹出android软键盘后的窗口高度
          console.log('windowInnerHeight1',windowInnerHeight);
        }, 500);
      },
      tipClose(tips){
        this['tips'+tips] = false;
        // document.getElementsByTagName('body')[0].removeAttribute('style','overflow:hidden');
      },
      openTip(tips){
        this['tips'+tips] = true;
        // document.getElementsByTagName('body')[0].style='overflow:hidden';
      },
       //获取满减送列表
      getAppListReduced(tips) {
         let that = this;
          that.$http.post(that.apiAppListReduced)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.appListReduced = data['var'];
              that.openTip(tips);
              // that['tips'+tips] = true;
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
       //秒杀活动立即预约
      reserveBtn(str) {
        let that = this;
          that.$http.post(that.apiReserve,{"activityId":str,'prodId':this.myCart.prodId})
          .then((response) => {
            let data = response.data;
            if(data.code == "S_OK") {
              that.$dialog.notify({mes: '预约成功', icon: 'error', timeout: 2000});
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      changeInput(){
        var that = this;
        that.isKeyUp = false;
        setTimeout(function () {//由于键盘弹出是有动画效果的，要获取完全弹出的窗口高度，使用了计时器
          var windowInnerHeight = window.innerHeight;//获取弹出android软键盘后的窗口高度
          console.log('windowInnerHeight2',windowInnerHeight);
        }, 500);
      },
      closeTip(){
        this.show2 = false;
        // document.getElementsByTagName('body')[0].style.overflow='auto';
      },
      buy(){
        // document.getElementsByTagName('body')[0].style.overflow='hidden';
        this.show2 = true;
      },
      submit() {

        //提交数据
        let that = this;
        //手机号验证
        if(that.personData.mobile =='') {
            that.$dialog.notify({mes: '手机号不能为空', icon: 'error', timeout: 1200});
            return;
        }else{
          if(!ydFunLib.telInvalid(that.personData.mobile)) {
            that.$dialog.notify({mes: '输入手机号不正确', icon: 'error', timeout: 1200});
            return;
          }
        }
        //名字校验
        if(ydFunLib.isEmptyValid(that.personData.userName)) {
            that.$dialog.notify({mes: '名字不能为空', icon: 'error', timeout: 1200});
            return;
        }else{
          if(ydFunLib.strlenValid(that.personData.userName) > 12) {
            that.$dialog.notify({mes: '输入名字过长', icon: 'error', timeout: 1200});
            return;
          }
        }
        //身份证校验
        if(ydFunLib.isEmptyValid(that.personData.idNumber)) {
          that.$dialog.notify({mes: '身份证不能为空', icon: 'error', timeout: 1200});
          return;
        }
        if(/(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/.test(that.personData.idNumber) === false) {
          that.$dialog.notify({mes: '身份证输入有误', icon: 'error', timeout: 1200});
          return;
        }
         
        if(ydFunLib.isEmptyValid(that.personData.plateNumber)) {
            that.$dialog.notify({mes: '车牌号不能为空', icon: 'error', timeout: 1200});
            return;
        }
        //车架号校验
        if(!ydFunLib.isEmptyValid(that.personData.vinNumber)) {
          if(ydFunLib.strlenValid(that.personData.vinNumber) > 17) {
            that.$dialog.notify({mes: '输入车架号不能超过17位', icon: 'error', timeout: 1200});
            return;
          }else if(ydFunLib.strlenValid(that.personData.vinNumber) < that.vinlen) {
            that.$dialog.notify({mes: '输入车架号不能低于'+that.vinlen+'位', icon: 'error', timeout: 1200});
            return;
          }else if(!ydFunLib.isNumberLetters(that.personData.vinNumber)) {
            that.$dialog.notify({mes: '请输入您的车架号格式不正确', icon: 'error', timeout: 1200});
            return;
          }
        }
        document.getElementsByTagName('body')[0].removeAttribute('style','overflow:hidden');
        //开始提交
        that.$dialog.loading.open('保存中...');
        setSessionStore('serviceSubmitInfo',that.personData);//缓存确认信息
        localStorage.setItem("serviceInfoPhone", that.personData.mobile);
        if(this.isApply){
          return false;
        }else{
          this.isApply = true;
        }
        this.log(LOGCODE.SERVICE.PAGE_SERVICEDETAIL_BTN_USERINFO_SAVE);
        that.$http.post(that.apiUserInfoSubmitUrl,that.personData)
          .then((response) => {
            const data = response.data;
            setTimeout(()=>{
              that.$dialog.loading.close();
              if(data.code == "S_OK") {
                that.$dialog.notify({mes: '保存成功', icon: 'error', timeout: 1200, callback: () => {
                    that.show2 = false;
                    that.submitBuy();
                }});
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                    }});
              }else if(data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else {
                this.isApply = false;
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            },300);
          },(response) => {
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      fetchVinLength() {
        //获取车架号最低位
        let that = this;
        that.$http.post(that.apiVinUrl+'&shopsid='+window.shopsid)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.vinlen = data.var.vinLength;
            }
          });
      },
      //获取用户信息
      fetchUserData(){
        let that = this;
        this.log(LOGCODE.SERVICE.PAGE_SERVICEDETAIL_BTN_USERINFO_GET);
        that.$dialog.loading.open('加载数据中...');
        localStorage.setItem("serviceInfoPhone", that.personData.mobile);
        that.$http.post(that.apiUserInfoUrl,{"mobile":that.personData.mobile})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var !=null){
                that.personData.userName = data.var.userName !=''?data.var.userName:that.personData.userName;
                that.personData.idNumber = data.var.idNumber !=''?data.var.idNumber:that.personData.idNumber;
                that.personData.plateNumber = data.var.plateNumber!=''?data.var.plateNumber:that.personData.plateNumber;
                that.personData.vinNumber = data.var.vin!=''?data.var.vin:that.personData.vinNumber;
              }
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      submitBuy() {
        this.log(LOGCODE.SERVICE.PAGE_SERVICEDETAIL_BTN_BUY);

        //点击购买按钮触发相关数据请求逻辑
        let that = this;
        if(that.source==2 || that.source ==3){//限时进来的商品详情
          that.optsAddToOrder.activityId = that.$route.query.activityId;
        }
        that.optsAddToOrder.prodIds[0] = that.myCart;
        that.$dialog.loading.open('请求数据中...');
        that.$http.post(that.apiAddToOrderUrl+'&shopsid='+window.shopsid + '&fromSource='+this.source,that.optsAddToOrder)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();

            if(data.code == "S_OK") {
              //跳去订单详情
              that.$router.push({
                  path: '/orderdetails',
                  query: {
                    source: 2,
                    serveCode: that.serviceCode,
                    oid: data.var
                  }
              });
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "USER_NOT_MEMBER"){
                that.toast3 = true;
              }else if(data.code == "PRODUCT_BUY_COUNT_EXCEED"){
                that.toast4 = true;
                that.tipText = '亲，此限购商品每人只能购买<span class="red">'+that.detailsData.restrictCount+'</span>件';
              }else{
                this.isApply = false;
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
        },(response) => {
          that.$dialog.loading.close();
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        });
      },
      //活动倒计时结束回调
      countDownCall(){
        console.log('countDownBack');
        this.isCoundownCall = true;
        this.fetchProductData(this.log);
      },
      fetchProductData(cb) {
        //获取详情数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
          if(this.$route.query.psource==2 || this.$route.query.psource==3){//优惠券进来的商品详情
            that.opts.activityId = this.$route.query.activityId;
          }
        that.$http.post(that.apiUrl+'&fromSource='+this.source,that.opts)
          .then((response) => {
            const data = response.data;
            if(data.code=='S_OK'){
              that.detailsData = data.var.productInfo;
              that.couponList = data.var.applicableCouponList;
              that.reducedInfo = data.var.reducedInfo;
              // that.serviceCode = data.var.productInfo.serveCode.split('_')[0];
              //限时购判断时间
              // if(that.detailsData.register || that.detailsData.skipe){
                that.optAction.currTime = that.detailsData.currentTime;
                let _currTime = Math.floor(new Date((that.optAction.currTime).replace(/-/g,"\/")).getTime() / 1000);
                let actionEndTime = Math.floor(new Date((that.detailsData.endTime).replace(/-/g,"\/")).getTime() / 1000);
                if(_currTime < actionEndTime || _currTime == actionEndTime){
                    if(that.detailsData.isCyclePepeat ==1){//有重复周期                      
                      let _currDate = that.detailsData.currentTime.split(' ')[0];
                      let curDate = new Date((that.optAction.currTime).replace(/-/g,"\/"));
                      let endDate = new Date((that.detailsData.endTime).replace(/-/g,"\/"));
                      let cycleStartMinute = that.detailsData.cycleStartMinute>9?that.detailsData.cycleStartMinute:'0'+that.detailsData.cycleStartMinute;
                      let newEndDate = Math.floor(endDate.getTime() / 1000)-_currTime;
                      that.optAction.acStarTime = _currDate +' '+ that.detailsData.cycleStartHour +':'+ cycleStartMinute+':00';
                      that.optAction.acEndTime = _currDate +' '+ that.detailsData.cycleEndHour +':'+ that.detailsData.cycleEndMinute+':00';
                      //判断今天抢购活动是否结束
                        if(Math.floor(curDate.getTime() / 1000)>Math.floor(new Date((that.optAction.acEndTime).replace(/-/g,"\/")).getTime() / 1000)){             
                          if(newEndDate>0){
                            that.optAction.acStarTime = curDate.getFullYear()+'-'+(curDate.getMonth()+1)+'-'+(curDate.getDate()+1) +' '+ that.detailsData.cycleStartHour +':'+ that.detailsData.cycleStartMinute+':00';    
                          }
                        }
                    }else{//无重复周期
                      that.optAction.acStarTime = that.detailsData.startTime;
                      that.optAction.acEndTime = that.detailsData.endTime;
                    }
                    var _starTime = Math.floor(new Date((that.optAction.acStarTime).replace(/-/g,"\/")).getTime() / 1000);
                    var _endTime = Math.floor(new Date((that.optAction.acEndTime).replace(/-/g,"\/")).getTime() / 1000);
                    console.log('2=>',_starTime,_endTime,that.optAction.acStarTime,that.optAction.acEndTime,that.isCoundownCall);
                    if(_currTime<_starTime){//活动未开始
                      that.actionStatu = 3;      
                    }else if((_starTime==_currTime || _starTime <_currTime)&&_currTime<_endTime){//活动开始
                        that.actionStatu = 1;    
                      if(that.isCoundownCall){
                        console.log('count=>',that.optAction);
                        that.$refs.countDown.run(that.optAction);
                      } 
                    }else{//活动已结束
                      that.actionStatu = 2; 
                    }
                }else{
                  that.actionStatu = 2;
                }
              // }  
              //判断是否可以加入购物车
              if((that.detailsData.register || that.detailsData.skipe) && that.detailsData.status != 3){
                that.cartStatus = true;
              }else if ((that.detailsData.register || that.detailsData.skipe) && that.actionStatu == 3){
                that.cartStatus = false;
              }
              if(that.detailsData.stocksCount == 0) {
                that.stockoun = true;
              }
              if(that.detailsData.picList.length == 1) {
                that.isSingle = true;
              }else {
                that.start1 = true;
              }
              that.productDetail = data.var;
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
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