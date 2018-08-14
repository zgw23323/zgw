 <template>
    <div class="alet_container">
	    <section class="tip_text_container">
            <div class="tip_icon" @click="closeTip(1)">&nbsp;</div>
            <p class="tip_text" v-html="alertText"></p>
            <div class="confrim" @click="closeTip(2)">{{btnText}}</div>
        </section>
    </div>
</template>

<script>
    export default {
        name: 'yd-alert-tip',
    	data(){
            return{
                tips:false,
                str:''
            }
        },
       mounted() {
           // this.tips = this.tipsType;
           // console.log({'ishow':this.alertText,'tips':this.tips});
        },
        props: {
            alertText: {
                type: String
            },
            isShow: {
                type: Boolean,
                default: false

            },
            btnText: {
                type: String,
                default: '确定'
            },
            callback: {
                type: Function
            },
            closeCallback: {
                type: Function
            },
            isCallback: {
                type: Boolean,
                default: false
            },
            tipsType: {
                type: String,
                default: ''
            },
            tipNumber: {
                type: Number,
                default: 0
            }

        },
        methods: {
            closeTip(n){
                if(n==1){    
                    typeof this.closeCallback == 'function' && this.closeCallback(this.tipsType);
                }else{
                    if(this.isCallback){
                        typeof this.callback == 'function' && this.callback();
                    }else{
                        typeof this.closeCallback == 'function' && this.closeCallback(this.tipsType);
                    } 
                }              
            }
        }
    }
</script>

<style lang="less">  
  @import '../../../styles/components/alertTip.less';  
</style>
