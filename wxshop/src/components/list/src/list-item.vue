<template>
    <div class="list-item">
        <div class="list-img">
            <slot name="img"></slot>
        </div>
        <div class="list-mes">
            <div class="list-title">
                <slot name="title"></slot>
            </div>
            <slot name="other"></slot>
        </div>
    </div>
</template>

<script type="text/babel">
    export default {
        name: 'yd-list-item',
        props: {
            link: String
        }
    }
</script>
