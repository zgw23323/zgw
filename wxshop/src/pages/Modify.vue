<template>
	<transition name="page">
		<section class="ck-doc modify">
            <article class="content_item">
                <div class="m_flex_between modify-hd item">
                    <div class="u-info">
                       <!--  <div class="u-head" v-if="uheadDefault" :style="{backgroundImage: 'url(' + imgBaseUrl+personData.logopath + ')'}"></div>
                         
                        <div class="u-head" v-if="!uheadDefault" :style="{backgroundImage: 'url(' + uhead + ')'}"></div> -->
                        <div class="u-head" v-if="uheadDefault&&personData.logopath" :style="{backgroundImage: 'url(' + imgBaseUrl+personData.logopath + ')'}"></div>
                        <div class="u-head" v-else-if="uheadDefault&&personData.wxHeadPic" :style="{backgroundImage: 'url(' + imgBaseUrl+personData.logopath + ')'}"></div>
                        <div class="u-head" v-else :style="{backgroundImage: 'url(' + uhead + ')'}"></div>
                        <input id="file" class="file" type="file" @change="onFileChange('file')" accept="image/*">
                    </div>
                    <div><i class="i_right">&nbsp;</i></div>
                </div>
                <div class="m_flex_between modify-bd item">
                    <span slot="left">昵称：</span>
                    <input readonly v-model="personData.userName" slot="right" type="text" placeholder="请输入姓名" maxlength="12">
                </div>
                <div class="m_flex_between modify-bd item">
                    <span slot="left">姓名：</span>
                    <input readonly v-model="personData.userName" slot="right" type="text" placeholder="请输入姓名" maxlength="12">
                </div>
                <div class="m_flex_between modify-bd item">
                    <span slot="left">手机号码：</span>
                    <input readonly v-model="personData.phone" slot="right" type="text" placeholder="请输入手机号码" maxlength="11">
                </div>
                <div class="m_flex_between modify-bd item">
                    <span slot="left">身份证：</span>
                    <input readonly v-model="personData.idNumber" slot="right" type="text" placeholder="请输入手机号码" maxlength="11">
                </div>
                <div class="m_flex_between modify-bd item">
                    <span slot="left">车牌号：</span>
                    <input v-model="personData.plateNumber" slot="right" type="text" placeholder="请输入车牌号码" maxlength="10">
                </div>
                <div class="m_flex_between modify-bd item">
                    <span slot="left">车架号：</span>
                    <input v-model="personData.vin" slot="right" type="text" :placeholder="'请输入不少于'+vinlen+'位车架号'" maxlength="17">
                </div>
    		    <div class="m_flex_between item no_bor">
                    <div>
                        <span slot="left">当前车型：</span>
                    </div>
                    <div><input v-model="personData.cartypeName" slot="right" type="text" placeholder="请选择当前车型"><i class="i_right" @click="goToRouter">&nbsp;</i></div>
                </div>
		    </article>
            <div class="saveBtn" @click="submit"><span>保存</span></div>

            <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
		</section>
	</transition>
</template>

<script type="text/babel">
import {setShopsId,setSessionStore,getSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import {ydFunLib} from '../utils/ydFunLib';
export default {
    data() {
        return {
        	title: '个人资料',
        	uheadDefault: true,
          tips4:false,
          imgBaseUrl: window.imgBaseUrl,
          apiPersonUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxUserPersonalCenter?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        	apiUpdateUserInfoUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxUpdateUserInfo?orgCode='+window.orgCode,
        	personData: {
        		logopath: '',
        		userName: '',
        		phone: '',
        		idNumber: '',
        		cartypeName: '',
        		plateNumber: '',
            vin: ''
        	},
        	uhead: '',
          apiVinUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxGetVinLength?orgCode='+window.orgCode,
          vinlen: 6
        }
    },
    watch: {
    	optsAddressData: { //深度监听地址数据
	        handler() {
	        },
	        deep: true
	    },
	    '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      // this.fetchPersonList();
      this.fetchVinLength();

  	},
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取缓存个人数据
        var _personData = getSessionStore('_personData',true);
        if(typeof(_personData) !='boolean'){
            this.personData = _personData;
            var optCar = getSessionStore('carName',true);
            if(typeof(optCar) !='boolean'){
              this.personData.cartypeName = optCar.carsName;
            }
        }else{
          this.fetchPersonList(); 
        }

        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_MODIFY});
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
      //跳转路由
      goToRouter(){
          setSessionStore("_personData",this.personData);
          // setSessionStore('carName',{})
          this.$router.push({
            path: '/carList',
            query: {
              from:'modify'
            }
          });
      },
      fetchPersonList() {
          //获取个人中心页面数据
          let that = this;
          that.$dialog.loading.open('加载数据中...');
          that.$http.post(that.apiPersonUrl)
            .then((response) => {
              const data = response.data;
              that.tips1 = false;
              if(data.code == "S_OK") {
                that.personData = data.var;
                if(data.var.wxHeadPic||data.var.logopath){
                  that.uheadDefault = true;
                }else{
                  that.uheadDefault = false;
                }
                //缓存详情数据
                // sessionStorage.setItem("_personData", JSON.stringify(that.personData));
                setSessionStore("_personData",that.personData);

                // 更新 vuex中的变量  type: 0普通，1会员
                //status  1:有卡，2:卡申请中，3:无卡
                if(that.personData.type == 1 && that.personData.status == 1){                
                  this.$store.commit('UPDATE_CARDINFO', {'cardNo': that.personData.vipCard, 'linkTo':'/vipCard'});
                }              
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else {
                that.tips1 = true;
                this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
              }
              setTimeout(()=>{
                that.$dialog.loading.close();
              },300);
            },(response) => {
              that.tips1 = true;
              that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
              setTimeout(()=>{
                that.$dialog.loading.close();
              },300);
            });
      },
      submit() {
          //提交数据
          let that = this;

          //名字校验
          if(!ydFunLib.isEmptyValid(that.personData.userName)) {
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
          //车牌校验
          if(!ydFunLib.isEmptyValid(that.personData.plateNumber)) {
            if(/^[\u4e00-\u9fa5]{1}[A-Z]{1}[A-Z_0-9]{5}$/.test(that.personData.plateNumber) === false) {
              that.$dialog.notify({mes: '车牌号输入有误', icon: 'error', timeout: 1200});
              return;
            }
          }
          
          //车架号校验
          if(!ydFunLib.isEmptyValid(that.personData.vin)) {
            if(ydFunLib.strlenValid(that.personData.vin) > 17) {
              that.$dialog.notify({mes: '输入车架号不能超过17位', icon: 'error', timeout: 1200});
              return;
            }else if(ydFunLib.strlenValid(that.personData.vin) < that.vinlen) {
              that.$dialog.notify({mes: '输入车架号不能低于'+that.vinlen+'位', icon: 'error', timeout: 1200});
              return;
            }else if(!ydFunLib.isNumberLetters(that.personData.vin)) {
              that.$dialog.notify({mes: '请输入您的车架号格式不正确', icon: 'error', timeout: 1200});
              return;
            }
          }
          //车型号校验
          if(!ydFunLib.isEmptyValid(that.personData.cartypeName)) {
            if(ydFunLib.strlenValid(that.personData.cartypeName) > 50) {
              that.$dialog.notify({mes: '输入车型号过长', icon: 'error', timeout: 1200});
              return;
            }
          }
          //开始提交
          that.$dialog.loading.open('保存中...');

          let optsPersonData = {};

          optsPersonData.id = that.personData.id;
          if(that.personData.logopath.indexOf('showLogo') > 0) {
            optsPersonData.logopath = that.personData.logopath.split('showLogo/')[1].split('?t')[0];
          }else {
            optsPersonData.logopath = that.personData.logopath;
          }
          optsPersonData.userName = that.personData.userName;
          optsPersonData.phone = that.personData.phone;
          optsPersonData.idNumber = that.personData.idNumber;
          optsPersonData.cartypeName = that.personData.cartypeName;
          optsPersonData.plateNumber = that.personData.plateNumber;
          optsPersonData.vin = that.personData.vin;
          that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_MODIFY_SAVE});
          that.$http.post(that.apiUpdateUserInfoUrl+'&shopsid='+window.shopsid,optsPersonData)
            .then((response) => {
              const data = response.data;
              setTimeout(()=>{
                that.$dialog.loading.close();
                if(data.code == "S_OK") {
                  that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_MODIFY_SAVE_OK});
                  that.$dialog.notify({mes: '保存成功', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/personal');
                  }});
                }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                    }});
                  that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_MODIFY_SAVE_FAIL});
                }else if(data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                    }});
                    that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_MODIFY_SAVE_FAIL});
                }else {
                  that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
                  that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_MODIFY_SAVE_FAIL});
                }
              },300);
            },(response) => {
              setTimeout(()=>{
                that.$dialog.loading.close();
                that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
              },300);
        
            });
      },
	    onFileChange(id) {
        //选择头像图片文件，裁剪压缩并且转换成base64数据
	    	let that = this;
	    	let options = {
		        type: "base64",
		        compress: {
		            width: 200,
		            height: 200,
		            quality: 1
		        }
		    }
	      	getImgData(id, function(file) {
	      		that.uheadDefault = false;
	      		that.uhead = file.base64;
	      		that.upLoadImg({"imgStr": file.base64});
		  	}, options);
	    },
	    upLoadImg(baseData) {
        //提交头像图片base64数据到服务器
	    	let that = this;
	    	that.$http.post(window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=user&isBase64=base64', baseData)
	          .then((response) => {
	            const data = response.data;
	            if(data.code == "S_OK") {
                    // that.uheadDefault = true;
	            	that.personData.logopath = data.var;
				}else {
					that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
				}
	          },(response) => {
	          	setTimeout(()=>{
		          	that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
	          	},300);
	          });
	    }
    }
}
/**
 * 获取图片处理后base64数据（以下为WEUI的处理图片的相关方法）
 */
function getImgData(id, callback, options) {
    var file = document.getElementById(id).files[0];
    var reader = new FileReader();
    var that = this;
    reader.onload = function (evt) {
        if (options.compress === false) {
            // 不启用压缩 & base64上传 的分支
            file.base64 = evt.target.result;
            callback(file);
            return;
        }

        // 启用压缩的分支
        var img = new Image();
        img.onload = function () {
            var ratio = detectVerticalSquash(img);
            var canvas = document.createElement('canvas');
            var ctx = canvas.getContext('2d');

            var maxW = options.compress.width;
            var maxH = options.compress.height;
            var w = img.width;
            var h = img.height;
            var dataURL = void 0;

            if (w < h && h > maxH) {
                w = parseInt(maxH * img.width / img.height);
                h = maxH;
            } else if (w >= h && w > maxW) {
                h = parseInt(maxW * img.height / img.width);
                w = maxW;
            }

            canvas.width = w;
            canvas.height = h;

            ctx.drawImage(img, 0, 0, w, h / ratio);

            if (/image\/jpeg/.test(file.type) || /image\/jpg/.test(file.type)) {
                dataURL = canvas.toDataURL('image/jpeg', options.compress.quality);
            } else {
                dataURL = canvas.toDataURL(file.type);
            }

            if (options.type == 'file') {
                if (/;base64,null/.test(dataURL) || /;base64,$/.test(dataURL)) {
                    // 压缩出错，以文件方式上传的，采用原文件上传
                    console.warn('Compress fail, dataURL is ' + dataURL + '. Next will use origin file to upload.');
                    callback(file);
                } else {
                    var blob = dataURItoBlob(dataURL);
                    blob.id = file.id;
                    blob.name = file.name;
                    blob.lastModified = file.lastModified;
                    blob.lastModifiedDate = file.lastModifiedDate;
                    callback(blob);
                }
            } else {
                if (/;base64,null/.test(dataURL) || /;base64,$/.test(dataURL)) {
                    // 压缩失败，以base64上传的，直接报错不上传
                    options.onError(file, new Error('Compress fail, dataURL is ' + dataURL + '.'));
                    callback();
                } else {
                    file.base64 = dataURL;
                    callback(file);
                }
            }
        };
        img.src = evt.target.result;
    };
    reader.readAsDataURL(file);
}
/**
 * 检查图片是否有被压扁，如果有，返回比率
 */
function detectVerticalSquash(img) {
    // 拍照在IOS7或以下的机型会出现照片被压扁的bug
    var data;
    var ih = img.naturalHeight;
    var canvas = document.createElement('canvas');
    canvas.width = 1;
    canvas.height = ih;
    var ctx = canvas.getContext('2d');
    ctx.drawImage(img, 0, 0);
    try {
        data = ctx.getImageData(0, 0, 1, ih).data;
    } catch (err) {
        console.log('Cannot check verticalSquash: CORS?');
        return 1;
    }
    var sy = 0;
    var ey = ih;
    var py = ih;
    while (py > sy) {
        var alpha = data[(py - 1) * 4 + 3];
        if (alpha === 0) {
            ey = py;
        } else {
            sy = py;
        }
        py = ey + sy >> 1; // py = parseInt((ey + sy) / 2)
    }
    var ratio = py / ih;
    return ratio === 0 ? 1 : ratio;
}
/**
 * dataURI to blob, ref to https://gist.github.com/fupslot/5015897
 * @param dataURI
 */
function dataURItoBlob(dataURI) {
    var byteString = atob(dataURI.split(',')[1]);
    var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];
    var ab = new ArrayBuffer(byteString.length);
    var ia = new Uint8Array(ab);
    for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
    }
    return new Blob([ab], { type: mimeString });
}
</script>
