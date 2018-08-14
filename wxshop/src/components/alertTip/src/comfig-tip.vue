 <template>
    <div class="alet_container" v-if="tips">
	    <section class="tip_comfig">
            <!-- <p class="tip_text" v-html="alertText"></p> -->
            <div class="comfig_con">
                <p class="q_submit" v-if="comfigTip.title">{{comfigTip.title}}</p>
                <p class="q_warn" v-if="comfigTip.mess">{{comfigTip.mess}}</p>
            </div>
            <div class="comfig_btn i_flex_between">
                <div @click="closeTip(2)">取消</div>
                <div @click="submit">确定</div>
            </div>
        </section>
    </div>
</template>

<script>
    export default {
        name: 'yd-comfig-tip',
    	data(){
            return{
                tips:false,
                str:''
            }
        },
       mounted() {

        },
        props: {
            comfigTip: {
                type: Object,
            },
            isShow: {
                type: Boolean,
                default: false
            },
            callback: {
                type: Function
            },
            tipNumber: {
                type: Number,
                default: 0
            }

        },
        methods: {
            closeTip(n){
                this.tips = false;           
            },
            openTip(n){
                this.tips = true; 
                this.str = n;          
            },
            submit(){
               typeof this.callback == 'function' && this.callback(this.str);
            }
        }
    }
</script>
