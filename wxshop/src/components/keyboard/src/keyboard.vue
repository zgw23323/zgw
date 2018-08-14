<template>
    <div class="key-board">
        <div class="u_btn" @click="closeTip"><i class="up_i">&nbsp;</i></div>
        <ul>
            <li v-for="item in count" @click="countBtn(item)">{{item}}</li>
            <li class="n_h">&nbsp;</li>
            <li @click="countBtn(0)">0</li>
            <li class="n_h" @click="delBtn">删除</li>
        </ul>
    </div>
</template>

<script type="text/babel">
    export default{
        name: 'yd-keyboard',
        data(){
            return{
                tips:false,
                count:[1,2,3,4,5,6,7,8,9]
            }
        },
        props: {
            title: String,
            tipsType: {
                type: String,
                default: ''
            }
        },
        methods: {
            closeTip(){
                this.$parent[this.tipsType] = !this.tipsType;
            },
            countBtn(coun) {
                this.$parent.getCode(coun);
            },
            delBtn() {
                this.$parent.deleteCode();
            }
        }
    }
</script>
<style lang="less">
    @import '../../../styles/components/codeBoard.less';
</style>
