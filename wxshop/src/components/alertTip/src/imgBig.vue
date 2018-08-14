 <template>
    <div class="ragb1" v-if="tips">
        <div class="head"><span @click='closeTip'>关闭</span></div>
        <div class="img-item"><img :src="imgUrl" /></div>
    </div>

</template>

<script>
    export default {
        name: 'yd-imgBig-tip',
        data(){
            return{
                tips:false,
                imgBaseUrl: window.imgBaseUrl,
                imgUrl:'',
            }
        },
        props: {
            imgId: {
                type: String,
                default:''
            },
            tipsType: {
                type: String,
                default:''
            }
        },
        methods: {
            closeTip(){
                this.tips=false;
            },
            openTip(fileId){
                this.imgUrl = fileId;
                this.tips=true;
            }

        }
    }
</script>
