 <template>
    <div class="alet_container">
        <!-- <section class="tip_text_container">
            <p class="tip_text">加载中...</p>
        </section> -->
    </div>

</template>

<script>
    export default {
        name: 'yd-imgLoading-tip',
    	data(){
            return{
                tips:false,
                str:''
            }
        },
        props: {
            alertText: {
                type: String
            }
        }
    }
</script>

<style lang="less">  
  // @import '../../../styles/components/alertTip.less';  
</style>
