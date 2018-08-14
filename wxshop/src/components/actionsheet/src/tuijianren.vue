<template>
    <div>
        <div class="m-actionsheet" id="actionSheet">
            <a href="#" class="actionsheet-item">示例菜单 - Data API</a>
            <a href="#" class="actionsheet-item">示例菜单 - Data API</a>
            <a href="#" class="actionsheet-item">示例菜单 - Data API</a>
            <a href="#" class="actionsheet-item">示例菜单 - Data API</a>
            <a href="javascript:;" class="actionsheet-action" id="cancel">取消</a>
        </div>
        <div class="m-button">
            <button type="button" class="btn-block btn-primary" data-ydui-actionsheet="{target:'#actionSheet',closeElement:'#cancel'}">ActionSheet(Data API调用)</button>
        </div>
    </div>
</template>

<script type="text/babel">
    export default {
        name: 'yd-tuijian-select',
        data() {
            return {                
                itemHeight: 40
            }
        },
        props: {
            status: String
        },
        methods: {
            init() {                
                
            }
        },
        mounted() {
            this.init();
        },
        destroyed() {
            //this.close();
        }
    }
</script>

<style lang="less">
    @import '../../../styles/components/actionsheet.less';
</style>
