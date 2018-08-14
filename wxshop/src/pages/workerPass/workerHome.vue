<template>
  <section class="service glassClaim">
    <article class="s_contract glassSupport s_glassClaim" style="padding-bottom: .4rem;">
        <div class="i_bottom">
          <div class="g_item" style="padding: .3rem;">
            <div>维修工单上传：<br> 
              <div class="file gong">
                <div class="img" style="" v-for="item in imgList">
                    <!-- <img :id="item.fileId" v-bind:src="item.base64"/> -->
                    <img :id="item.fileId" v-bind:src="imgBaseUrl+'/shop/downLoad/showLogo/'+item.fileId+'?type=thumb'"/>
                    <div class="img_btn" v-if="isUpdata">
                      <!-- <i class="img_big_btn" @click="$refs.child.openTip(item.fileId)" v-if="!isWx">&nbsp;</i> -->
                      <i class="img_big_btn" @click="imgBtn(item.fileId)">&nbsp;</i>
                      <i class="img_delete_btn">&nbsp;</i>
                    </div>
                </div>
                <div class="upImg_gong" v-if="isUpdata">
                  <span class="l_up_file">&nbsp;</span> 
                  <!-- <input type="file" id="file_input" @change="readFile('gong')" capture="camera" accept="image/*">  -->
                  <input type="file" id="file_input" @change="readFile('gong')" accept="image/*"> 
                  <div class="p_20">上传维修单</div>
                </div>
              </div>
            </div>
          </div>
        </div>
    </article>
    <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
    <yd-imgBig-tip ref="child"></yd-imgBig-tip>
  </section>
</template>
 
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,removeSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        tips4:false,
        imgList:[], 
        isUpdata:true,
        imgBaseUrl: window.imgBaseUrl,
        apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=glass&isBase64=y&flag=thumb'
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    methods: {
      //获取本地图片信息
      readFile(type,n,str){ 
        var that = this;
        var file = type =='position' ? document.getElementById(str).files[0]:document.getElementById('file_input').files[0]; //获取file对象
        //判断file的类型是不是图片类型。
        if(!/image\/\w+/.test(file.type)){ 
            alert("文件必须为图片！"); 
            return false; 
        }
        var reader = new FileReader(); //声明一个FileReader实例
        reader.readAsDataURL(file);
        that.$dialog.loading.open('上传中...');
        that.tips4 = true;
        reader.onload = function(e){ 
          // console.log(e);
          
        var data = new Date();
        console.log('img压缩=》',data.getSeconds());
        // that.prodPicUpload(this.result,type,n,str);
          that.appendFile(this.result,function(base64){
            var data = new Date();
            console.log('img压缩完成=》',data.getSeconds());
            // var opt = {base64:base64};
            // that.imgList.push(opt);
            // that.$dialog.loading.close();
            // that.tips4 = false;
            that.prodPicUpload(base64,type,n,str);
          });
        } 
      },
      imgBtn(pId){
        ydFunLib.wxBigImg(this.imgBaseUrl+'/shop/downLoad/showLogo/'+pId);
      },
      // 压缩文件
      appendFile: function (path,callback){
        var orientation = 0;
        var image = new Image();
        image.src = path;        // 传过来的图片路径在这里用。
        
        image.onload = function () {
             EXIF.getData(image, function() {
              orientation = EXIF.getTag(this,'Orientation');
              // console.log('Orientation',orientation);
                  // alert(EXIF.pretty(this));
              });
            var that = this;
            //生成比例 
            var w = that.width,
            h = that.height,
            scale = w / h; 
            w = w || 480;  //压缩到多大值
            // w = 375;  //压缩到多大值
            h = w / scale;

            //生成canvas，使用canvas进行压缩
            var canvas = document.createElement('canvas');

            var ctx = canvas.getContext('2d');
            var anw = document.createAttribute("width");
            anw.nodeValue = w;
            var anh = document.createAttribute("height");
            anh.nodeValue = h;
            canvas.setAttributeNode(anw);
            canvas.setAttributeNode(anh); 

            ctx.drawImage(that, 0, 0, w, h);//渲染画布

            //11.29更新
            var base64 = null;  
            base64 = canvas.toDataURL('image/jpeg', 1 || 0.8 );   
            //1最清晰，越低越模糊。有一点不清楚这里明明设置的是jpeg。弹出 base64 开头的一段 data：image/png;却是png。    
          // this.prodPicUpload(base64);
          callback(base64);
        }
      },
      //上传图片
      prodPicUpload(base64,type,n,str){
        var that = this;
        console.log('imgStr');
        // this.$dialog.loading.open('上传中...');
       
        var data = new Date();
        console.log('img上传=》',data.getSeconds());
        that.$http.post(this.apiUploadFileUrl, JSON.stringify({"imgStr":base64}))
          .then((response) => {
            const data = response.data;
            var data1 = new Date();
            console.log('img上传完成=》',data1.getSeconds());
            if(data.code == "S_OK") {
              var fileId = data["var"];
                  if(fileId){
                    console.log(fileId);
                    if(type=='gong'){
                      var opt = {fileId:fileId};
                      that.imgList.push(opt);
                      // that.imgList[that.imgList.length-1].fileId = fileId;
                    }else{
                      var _n = parseInt(str.split('_')[1]);
                      that.positionData[n].picIds[_n-1].picId = fileId;
                    }
                  }
              
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
            
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      }
    }
  }
</script>