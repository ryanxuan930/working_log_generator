<template>
    <div v-if="data===undefined">無資料</div>
    <div v-else style="padding: 1cm">
        <div style="font-size: 24pt; font-weight: bold;">國立中山大學學務處體育發展組</div>
        <div style="height: 0.5cm"></div>
        <div style="font-size: 18pt; font-weight: bold;">{{data.year}} 年 {{data.month}} 月 工讀日誌</div>
        <div style="height: 0.5cm"></div>
        <div style="font-size: 14pt;">姓名：{{data.name}}&emsp;手機：{{data.phone}}&emsp;學號：{{data.schoolId}}&emsp;身分證字號：{{data.unifiedId}}</div>
        <div style="height: 0.5cm"></div>
        <table class="w-full text-left">
            <tr>
                <th>日期</th>
                <th>時間</th>
                <th>時數</th>
                <th>地點</th>
                <th>出勤</th>
                <th>備註</th>
            </tr>
            <template v-for="(item, index) in data.log" :key="index">
                <tr>
                    <td>{{item.date.substr(5,2)+'/'+item.date.substr(8,2)}} ( {{dateToDay(item.date)}} )</td>
                    <td>{{item.startTime}} - {{item.endTime}}</td>
                    <td>{{item.duration}}</td>
                    <td>{{item.location}}</td>
                    <td>
                        <span v-if="item.status=='正常出勤'" class="text-green-600">正常出勤</span>
                        <span v-else class="text-red-600">{{item.status}}</span>
                    </td>
                    <td>
                        <div v-if="item.fillIn!=''">[代班]-{{item.fillIn}}</div>
                        <div v-if="item.remark!=''">{{item.remark}}</div>
                    </td>
                </tr>
            </template>
            <tr>
                <td colspan="6" class="text-center">總計共 {{data.total}} 小時</td>
            </tr>
            <tr>
                <td colspan="6" class="text-center">以下空白</td>
            </tr>
        </table>
        <div style="height: 0.5cm"></div>
        <div class="flex">
            <div class="basis-1/2 text-left">承辦人：</div>
            <div class="basis-1/2 text-left">單位主管：</div>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
    setup() {
        const dayList = ['日','一','二','三','四','五','六'];
        var data;
        if(localStorage.working_log_temp){
            data = JSON.parse(localStorage.working_log_temp);
        }else{
            data = undefined;
        }
        return {
            data,
            dateToDay: function (date :string) {
                const d = new Date(date);
                return dayList[d.getDay()];
            }
        }
    },
})
</script>

<style lang="less" scoped>
@import url(https://fonts.googleapis.com/earlyaccess/cwtexkai.css);
* {
    font-family: 'cwTeXKai', serif;
}
table {
    th {
        padding: 2mm;
        border-bottom: #424242 1px solid;
        border-top: #BDBDBD 1px solid;
    }
    td {
        padding: 2mm;
        border-bottom: #BDBDBD 1px solid;
    }
}
</style>

