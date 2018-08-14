<template>
    <span>
        <span class="count_down" v-html="str"></span>
        <span ref="tpl" v-if="showTpl"><slot></slot></span>
    </span>
</template>

<script type="text/babel">
    export default {
        name: 'yd-countdown',
        data() {
            return {
                str: '',
                timer: null,
                cType:'1',//倒计时类型
                tempFormat: '',
                showTpl: true
            }
        },
        props: {
            time: {
                type: String
            },
            cursystem: {
                type: String
            },
            netxtTime: {
                type:String
            },
            format: {
                type: String,
                default: '{%d}天{%h}时{%m}分{%s}秒'
            },
            callback: {
                type: Function
            },
            countType: {
                type: String
            },
            isContiue: {
                type: String,
                default: 'false'
            },
            doneText: {
                type: String,
                default: '已结束'
            }
        },
        methods: {
            run(pObj) {
                if(pObj !=undefined){
                    console.log('11111');
                    var lastTime = Math.floor(new Date((pObj.acEndTime).replace(/-/g,"\/")).getTime() / 1000);
                    var systemTime = Math.floor(new Date((pObj.currTime).replace(/-/g,"\/")).getTime() / 1000);
                    console.log('222',lastTime,systemTime);
                }else{
                    console.log('this.time',this.time);
                    var lastTime = Math.floor(new Date((this.time).replace(/-/g,"\/")).getTime() / 1000);
                    var systemTime = Math.floor(new Date((this.cursystem).replace(/-/g,"\/")).getTime() / 1000);
                }
                
                let leftTime = lastTime - systemTime; 
                console.log('333',leftTime);
                // console.log(this);
                
                // console.log({'time':this.time,'systemTime':this.cursystem,'text':this.isContiue,'o':pObj});
                this.timer = setInterval(() => {
                    //let leftTime = lastTime - Math.floor(new Date().getTime() / 1000);
                    console.log('335');
                    leftTime = leftTime - 1;
                    if (leftTime > 0) {
                        // this.str = this.timestampTotime(leftTime);
                        if(this.countType=='1'){
                            let _str = this.timestampTotime(leftTime).split(":");
                            let _hour = parseInt(_str[0])*24+ parseInt(_str[1]);
                            _str[1] = _hour>9?_hour:'0'+_hour.toString();
                            this.str = "<span class='s2-t'>"+_str[1]+"</span>"+':'+"<span class='s2-t'>"+_str[2]+'</span>'+':'+"<span class='s2-t'>"+_str[3]+'</span>';
                        }else if(this.countType == '2'){
                            let _str = this.timestampTotime(leftTime).split(":");
                            let _hour = parseInt(_str[0])*24+ parseInt(_str[1]);
                            _str[1] = _hour>9?_hour:'0'+_hour.toString();
                            this.str = "<span class='s_3'>"+_str[1]+"</span>"+'<span class="s_2">时</span>'+"<span class='s_3'>"+_str[2]+'</span>'+'<span class="s_2">分</span>'+"<span class='s_3'>"+_str[3]+'</span>'+'<span class="s_2">秒</span>';
                        }else{
                            this.str = this.timestampTotime(leftTime);
                        }
                    } else {
                        typeof this.callback == 'function' && this.callback();
                        if(this.countType=='1'){
                            let _str = this.timestampTotime(leftTime).split(":"); 
                            this.str = "<span class='s2-t'>00</span>:<span class='s2-t'>00</span>:<span class='s2-t'>00</span>";
                        }else{
                            this.str = this.doneText;
                        }
                        clearInterval(this.timer);
                    }
                }, 1000);
            },
            timestampTotime(time) {
                let format = this.tempFormat;
                let t = {};

                t.s = time % 60;
                time = Math.floor(time / 60);
                t.m = time % 60;
                time = Math.floor(time / 60);
                t.h = time % 24;
                t.d = Math.floor(time / 24);

                const ment = function (a) {
                    if (a <= 0)  return '00';
                    return a < 10 ? '0' + a : a;
                };

                const arr = ['d', 'h', 'm', 's'];

                arr.forEach((val) => {
                    const day = ment(t[val]).toString().split('');
                    format = format.replace('{%' + val + '}', ment(t[val]));
                    format = format.replace('{%' + val + '0}', ~~day[0] != 0 ? day[0] : '');
                    format = format.replace('{%' + val + '1}', ~~day[day.length - 2]);
                    format = format.replace('{%' + val + '2}', ~~day[day.length - 1]);
                });
                return format;
            }
        },
        mounted() {
            this.$nextTick(() => {
                this.tempFormat = !!this.$slots.default ? this.$refs.tpl.innerHTML : this.format;
                this.showTpl = false;
                this.run();
            });
        },
        destroyed() {
            clearInterval(this.timer);
        }
    }
</script>
