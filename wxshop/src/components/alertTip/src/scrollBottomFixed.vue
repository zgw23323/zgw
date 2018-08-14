 <template>
    <div v-if="isFiexShow" class='scroll_bottom_fixed' :style="{'background-color':bStyle}">---我也是有底线滴---</div>
</template>

<script>
    export default {
        name: 'yd-bottom-fixed',
    	data(){
            return{
                tips:false,
                // isShow:false,
                // bStyle:'#fff',
                str:''
            }
        },
        props: {
            isFiexShow: {
                type: Boolean,
                default: false
            },
            bStyle: {
                type: String,
                default: '#fff'
            }
        },
        methods: {
            openFixed(p1,p2){
                this.isShow = p1;
                if(p2){
                    this.bStyle = p2;
                }
                // if(this.isCallback){
                //     typeof this.callback == 'function' && this.callback();
                // }else{
                //     this.$parent[this.tipsType] = !this.tipsType;
                // }
            }
        }
    }
</script>
