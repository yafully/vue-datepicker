<template>
  <div v-click-outside>
      <input type="text" :value="formatDate" readonly>
      <div v-if="isVisible" class="dp-box">
            <!--@picker-->
            <div class="dp-inner">
                <div class="dp-nav">
                    <span @click="prevYear">&lt;</span>
                    <span @click="prevMonth">&lt;&lt;</span>
                    <span>{{time.year}}年</span>
                    <span>{{time.month +1}}月</span>
                    <span @click="nextMonth">&gt;&gt;</span>
                    <span @click="nextYear">&gt;</span>
                </div>

                <div class="dp-header">
                    <span class="dp-cell" v-for="k in 7" :key="'w'+k">{{weekDays[k-1]}}</span>
                </div>
                <div class="dp-body">
                    <div class="dp-days">
                        
                        <div v-for="i in 6" :key="i">
                            <span v-for="j in 7" :key="j" class="dp-cell" 
                            :class="[
                            {notCurrentMonth:!isCurrentmonth(visibleDays[(i-1)*7+(j-1)])},
                            {today:isToday(visibleDays[(i-1)*7+(j-1)])},
                            {selected:isSelected(visibleDays[(i-1)*7+(j-1)])}
                            ]" 
                            @click="chooseDay(visibleDays[(i-1)*7+(j-1)])">
                                {{visibleDays[(i-1)*7+(j-1)].getDate()}}
                            </span>
                        </div>
                    </div>
                </div>
                <div class="dp-footer" @click="chooseDay(new Date())">今天</div>


            </div>
            <!--\\picker-->

      </div>
  </div>
</template>

<script>
import * as utils from './util'
export default {
    directives:{
        clickOutside:{
            bind(el, bindings, vnode){
                //把事件绑定到document上，看点击是否在当前元素内部
                let handler = (e)=>{
                    if(el.contains(e.target)){
                        //获取虚拟dom上下文中的方法
                        if(!vnode.context.isVisible){
                            vnode.context.focusEvent();
                        }
                    }else{
                        if(vnode.context.isVisible){
                            vnode.context.blurEvent();
                        }
                    }
                }
                el.handler = handler;
                document.addEventListener('click', handler);
            },
            unbind(el){
                document.removeEventListener('click', el.handler);
            }
        }
    },
    data(){
        let {year,month} = utils.getYearMonthDay(this.value);
        return{
            weekDays:['日','一','二','三','四','五','六'],
            time:{year,month},
            isVisible: false
        }
    },
    props:{
        value:{
            type:Date,
            default:()=>new Date
        }
    },
    computed:{
        formatDate () {
            let {year,month,day} = utils.getYearMonthDay(this.value);
            return `${year}-${month+1}-${day}`
        },
        visibleDays(){
            //获取当前是星期几 就把天数往前移动几天,这里不能用当前默认时间否则改变年月就无法计算
            
            let {year,month} = utils.getYearMonthDay(utils.getDate(this.time.year, this.time.month ,1));
            //获取当前月的第一天
            let currentFirstDay = utils.getDate(year,month,1);
            let week = currentFirstDay.getDay();
            //往前移动N天
            let startDay = currentFirstDay - week * 60 * 60 * 1000 * 24;
            //循环42天
            let arr = [];
            for(let i = 0; i<42; i++){
                arr.push(new Date(startDay+i * 60 * 60 * 1000 * 24))
            }
            return arr;
        }
    },
    methods: {
        focusEvent(){
            this.isVisible = true;
        },
        blurEvent(){
            this.isVisible = false;
        },
        isCurrentmonth (date){
            let {year,month} = utils.getYearMonthDay(utils.getDate(this.time.year, this.time.month ,1));
            let {year:y,month:m} = utils.getYearMonthDay(date);
            return year === y && month ===m;
        },
        isToday(date){
            let {year,month,day} = utils.getYearMonthDay(new Date());
            let {year:y,month:m,day:d} = utils.getYearMonthDay(date);
            return year === y && month === m && day === d;
        },
        chooseDay(date){
            this.time = utils.getYearMonthDay(date);
            this.$emit('input',date);
            this.blurEvent();
        },
        isSelected(date){
            let {year,month,day} = utils.getYearMonthDay(this.value);
            let {year:y,month:m,day:d} = utils.getYearMonthDay(date);
            return year === y && month === m && day === d;
        },
        prevMonth(){
            let d = utils.getDate(this.time.year, this.time.month,1);
            d.setMonth(d.getMonth()-1);
            this.time = utils.getYearMonthDay(d);
        },
        nextMonth(){
            let d = utils.getDate(this.time.year, this.time.month,1);
            d.setMonth(d.getMonth()+1);
            this.time = utils.getYearMonthDay(d);
        },
        prevYear(){
            let d = utils.getDate(this.time.year, this.time.month,1);
            d.setFullYear(d.getFullYear()-1);
            this.time = utils.getYearMonthDay(d);
        },
        nextYear(){
            let d = utils.getDate(this.time.year, this.time.month,1);
            d.setFullYear(d.getFullYear()+1);
            this.time = utils.getYearMonthDay(d);
        }
    },
    components: {

    }
}
</script>

<style lang="stylus">
@import url(./issets/style.css);
.dp-box
    position absolute;
    width 240px;
    height 304px;
    background #fff;
    padding 5px;
    border 1px solid #465e76;
    background #F3FAFF;
    border-radius 2px;
    font-size 13px;
    .dp-inner
        border 1px solid #5C9CC0;
        background #fff;
        .dp-cell
            display inline-flex;
            justify-content center;
            align-items center;
            width 32px;
            height 32px;
            border 1px solid transparent;
    .dp-nav
        height 30px;
        line-height 30px;
        display flex;
        justify-content space-around;
        user-select none;

    .dp-header
        border 1px solid #5C9CC0;
        background #BBE1F1;
        border-width 1px 0;
        font-weight 700;
    .dp-body
        .dp-cell
            cursor pointer;
        .dp-cell:hover
            background #ebf8ff;
            border 1px solid #0066cc;
            border-radius 2px;
        .notCurrentMonth
            position relative;
            color #999;
        .notCurrentMonth:after
            position absolute;
            width 100%;
            height 100%;
            text-align center;
            line-height 32px;
            content '\e900';
            font-family  'icomoon';
            color #888;
            font-size:26px;
        .today
            color #f00;
        .selected
            position relative;
        .selected:after
                position absolute;
                width 100%;
                height 100%;
                text-align center;
                line-height 32px;
                content '\e901';
                font-family  'icomoon';
                color #c00;
                font-size:26px;
    .dp-footer
        height 30px;
        line-height 30px;
        text-align center;
        border-top 1px solid #5C9CC0;
        cursor pointer;
</style>
