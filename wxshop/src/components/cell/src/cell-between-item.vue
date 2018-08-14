<template>
    <!-- 右图标 -->
    <div class="y_cell" v-if="type=='right'">
        <div class="y_cell_left">{{title}}</div>
        <div>
            <i class="i_right">&nbsp;</i>
        </div>
    </div>
    <!-- 圆角勾-->
    <div class="y_cell " v-else-if="type=='y_gou'">
        <div :class="{'y_cell_left T3C4': !dataOpt.isCur,'y_cell_left T3C3': dataOpt.isCur}">{{dataOpt.name}}</div>
        <div>
            <i :class='{"y_gou i_y_gou1":dataOpt.isCur,"y_gou i_y_gou2":!dataOpt.isCur}' >&nbsp;</i>
        </div>
    </div>
</template>

<script type="text/babel">
    export default{
        name: 'yd-cell-between-item',
        props: {
            title: String,
            type: {
                type: String,
                default:''
            },
            isCur: {
                type: Boolean,
                default: false
            },
            dataOpt: {
                type: Object,
                default: {}
            }
        }
    }
</script>
