<template>
    <div>
        <yd-popup v-model="tips" position="bottom" class="area" height="42.5%">
            <ul class="i_flex"><li v-for="item in All" class="small" @click="areaBtn(item)">{{item}}</li></ul>
        </yd-popup>
    </div>
</template>

<script type="text/babel">
    export default {
        name: 'yd-cut-area',
        data() {
            return { 
                tips: false,
                All:["京","津","泸","渝","冀","粤","豫","云","黑","湘","皖","鲁","新","苏","浙","贛","鄂","桂","甘","晋","蒙","陕","吉","闽","贵","青","藏","川","宁","琼","辽"]
            }
        },
        methods: {
            openTip() {
                this.tips = !this.tips;
            }, 
            areaBtn(str){
                this.tips = !this.tips;
                this.$emit('child-tell-me-something',str);
            }
        }
    }
</script>

<style lang="less">
    @import "../../../styles/components/cutArea.less";
</style>
