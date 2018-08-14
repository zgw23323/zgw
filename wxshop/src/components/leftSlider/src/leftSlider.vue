<template>
    <li
         @touchstart="_touchStart"
         @touchmove="_touchMove"
         @touchend="_touchEnd"
         :style="txtStyle">
        <slot></slot>
    </li>
</template>

<script>
    import Vue from 'vue';
    import {getSlideAngle, getSlideDirection} from '../../../utils/assist';

    export default {
        name: 'yd-leftSlider',
        props: {
            index: Number
        },
        data() {
            return {
                startX: 0,       //触摸位置
                moveX: 0,       //滑动时的位置
                disX: 0,       //移动距离
                txtStyle: ''
            }
        },
        methods: {
            _touchStart: function(ev) {
                ev = ev || event;
                if(ev.touches.length == 1){
                    // 手指按下的时候记录按下的位置
                    this.startX = ev.touches[0].clientX;
                    this.startY = ev.touches[0].clientY;
                }
            },
            _touchMove: function(ev) {
                ev = ev || event;
                 
            },
            _touchEnd: function(ev) {
                var endX, endY;
                endX = ev.changedTouches[0].clientX;
                endY = ev.changedTouches[0].clientY;
                var direction = getSlideDirection(this.startX, this.startY, endX, endY);
                var domLi = document.getElementsByTagName("li");
                switch (direction) {
                    case 0:
                        break;
                    case 'swipeup':
                        break;
                    case 'swipedown':
                        break;
                    case 'swipeleft':
                        console.log(this);
                        this.txtStyle = "transform:translateX(-1.2rem)";
                        // domLi[this.index].style = "transform:translateX(-1.2rem)";
                        break;
                    case 'swiperight':
                        // domLi[this.index].style = "transform:translateX(0)";
                       this.txtStyle = "transform:translateX(0)";
                        break;
                    default:
                }
            }
        }
    }
</script>