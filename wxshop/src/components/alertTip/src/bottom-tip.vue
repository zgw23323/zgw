 <template>
    <div>
        <div class='tip_f' @click="closeTip">&nbsp;</div>
        <div class='ck-tipsFormat bottom-tip' :class="{'no_pad':tipsType == 'serviceShow'}">
            <div class="tip_icon" @click="closeTip" v-if="isShow">&nbsp;</div>
            <slot></slot>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'yd-bottom-tip',
    	data(){
            return{
                tips:false,
                str:''
            }
        },
        props: {
            isShow: {
                type: Boolean,
                default: false

            },
            callback: {
                type: Function
            },
            isCallback: {
                type: Boolean,
                default: false
            },
            tipsType: {
                type: String,
                default: ''
            }

        },
        methods: {
            closeTip(){
                if(this.isCallback){
                    typeof this.callback == 'function' && this.callback();
                }else{
                    this.$parent[this.tipsType] = !this.tipsType;
                }
            }
        }
    }
</script>

<style lang="less">  
  // @import '../../../styles/components/alertTip.less';  
</style>
