<template>
    <div class="h-full flex flex-col">
        <div class="my-5 flex">
            <div class="flex-grow"></div>
            <div class="menu shadow bg-white">
                <button @click="newForm">建立日誌</button>
                <label>上傳日誌
                    <input type="file" class="hidden" accept="json" @change="loadFile">
                </label>
                <button @click="printForm">列印日誌</button>
                <button @click="exportFile">下載日誌</button>
            </div>
            <div class="flex-grow"></div>
        </div>
        <div v-if="!(Object.keys(logTemp).length === 0 && logTemp.constructor === Object)" class="mb-5 mx-auto w-5/6 p-5 bg-white shadow flex-grow overflow-hidden h-full flex flex-col">
            <div class="text-left flex items-center">
                <div class="title">
                    <input  v-model="logTemp.year" type="number">
                    <span>年</span>
                    <select v-model="logTemp.month">
                        <template v-for="(i, index) in 12" :key="index">
                            <option :value="i">{{i}}</option>
                        </template>
                    </select>
                    <span>月</span>
                </div>
                <div>共 {{logTemp.log.length}} 筆紀錄</div>
                <div>總時數： {{logTemp.total}} 小時</div>
                <div class="flex-grow"></div>
                <div>
                    <button @click="sortList" class="text-sky-400 hover:text-sky-500 duration-300 mr-2">排序</button>
                    <button @click="newLog" class="text-sky-400 hover:text-sky-500 duration-300">新增紀錄</button>
                </div>
            </div>
            <div class="user text-left">
                <span>姓名：</span>
                <input  v-model="logTemp.name" type="text">
                <span>手機：</span>
                <input  v-model="logTemp.phone" type="text">
                <span>學號：</span>
                <input  v-model="logTemp.schoolId" type="text">
                <span>身分證字號：</span>
                <input  v-model="logTemp.unifiedId" type="text">
            </div>
            <hr class="my-2">
            <div class="flex-grow overflow-auto">
                <template v-for="(item, index) in logTemp.log" :key="index">
                    <div class="flex items-center my-2">
                        <div class="flex-grow">
                            <div class="flex items-center mb-2">
                                <div>
                                    <span>日期：</span>
                                    <input class="border-2 rounded p-1" type="date" v-model="logTemp.log[index].date">
                                </div>
                                <div class="ml-2">
                                    <span>時間：</span>
                                    <input class="border-2 rounded p-1" type="time" v-model="logTemp.log[index].startTime">
                                    <span class="inline-block mx-1">到</span>
                                    <input class="border-2 rounded p-1" type="time" v-model="logTemp.log[index].endTime">
                                </div>
                                <div class="ml-2">
                                    <span>時數：</span>
                                    {{timeToHours(logTemp.log[index].date, logTemp.log[index].startTime, logTemp.log[index].endTime, index)}}
                                </div>
                            </div>
                            <div class="flex items-center">
                                <div>
                                    <span>地點：</span>
                                    <select class="border-2 rounded p-1" v-model="logTemp.log[index].location">
                                        <template v-for="(item, index) in locationList" :key="index">
                                            <option>{{item}}</option>
                                        </template>
                                    </select>
                                </div>
                                <div class="ml-2">
                                    <span>代班：</span>
                                    <input type="text" class="border-2 rounded p-1" v-model="logTemp.log[index].fillIn">
                                </div>
                                <div class="ml-2">
                                    <span>出勤狀況：</span>
                                    <select class="border-2 rounded p-1" v-model="logTemp.log[index].status">
                                        <option>正常出勤</option>
                                        <option>未出勤-請假</option>
                                        <option>未出勤-其他原因</option>
                                    </select>
                                </div>
                                <div class="ml-2">
                                    <span>備註：</span>
                                    <input type="text" class="border-2 rounded p-1" v-model="logTemp.log[index].remark">
                                </div>
                            </div>
                        </div>
                        <div class="mx-2">
                            <button @click="removeItem(index)" class="text-red-500 hover:text-red-600 duration-300">刪除</button>
                        </div>
                    </div>
                    <hr>
                </template>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import $ from 'jquery'

export default defineComponent({
    setup() {
        const router = useRouter();
        const logTemp: any = ref({});
        if(localStorage.working_log_temp){
            logTemp.value = JSON.parse(localStorage.working_log_temp);
        }
        const formTemplate = {
            timestamp: Date.now(),
            name: "",
            phone: "",
            schoolId:"",
            unifiedId: "",
            year: "111",
            month: "1",
            total: 0,
            log: []
        };
        const logTemplate = {
            date: "",
            startTime: "",
            endTime: "",
            duration: 0,
            location: "辦公室",
            status: "正常出勤",
            remark: "",
            fillIn: "",
        };
        const locationList = ['辦公室','第一健身房','第二健身房','網球場','游泳池'];
        function newForm(){
            if($.isEmptyObject(logTemp.value)){
                logTemp.value = JSON.parse(JSON.stringify(formTemplate));
            }else{
                const r = confirm("確定覆蓋現有的資料？");
                if(r){
                    logTemp.value = JSON.parse(JSON.stringify(formTemplate));
                }
            }
        }
        function newLog(){
            logTemp.value.log.push(JSON.parse(JSON.stringify(logTemplate)))
        }
        watch(logTemp, ()=>{
            console.log('change');
            localStorage.working_log_temp = JSON.stringify(logTemp.value);
            logTemp.value.total=0;
            for(let i=0; i<logTemp.value.log.length; i++){
                if(logTemp.value.log[i].status=='正常出勤'){
                    logTemp.value.total+=logTemp.value.log[i].duration;
                }
            }
        },{ deep: true });
        function exportFile() {
            if(localStorage.working_log_temp){
                let dataStr = localStorage.working_log_temp;
                let dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);

                let exportFileDefaultName = 'export.json';

                let linkElement = document.createElement('a');
                linkElement.setAttribute('href', dataUri);
                linkElement.setAttribute('download', exportFileDefaultName);
                linkElement.click();
            }else{
                alert("無資料");
            }
        }
        function loadFile(event: any) {
            function action (){
                const reader: any = new FileReader();
                reader.onload = function (e: any) {
                    console.log(e);
                    const result = JSON.parse(e.target.result);
                    logTemp.value = result;
                }
                reader.readAsText(event.target.files[0]);
            }
            if($.isEmptyObject(logTemp.value)){
                action();
            }else{
                const r = confirm("確定覆蓋現有資料？");
                if(r){
                    action();
                }
            }
        }
        return {
            newForm,
            newLog,
            logTemp,
            timeToHours: function (date: string, startTime: string, endTime: string, index:number){
                const d1: any = new Date(date+' '+startTime);
                const d2: any = new Date(date+' '+endTime);
                console.log(d2-d1);
                let r = 0;
                if(logTemp.value.log[index].status=='正常出勤'){
                    r = (d2 - d1) / 1000 / 60 / 60;
                }
                logTemp.value.log[index].duration = isNaN(r)?0:r;
                return (!isNaN(r) && r>0)?r:0;
            },
            removeItem: function (index: number){
                const r = confirm("確定刪除？此筆資料無法恢復！");
                if(r){
                    logTemp.value.log.splice(index, 1);
                }
            },
            printForm: function(){
                const url = router.resolve({ path: "/print" });
                window.open(url.href);
            },
            sortList: function(){
                logTemp.value.log.sort((a: any, b: any)=> {
                    const d1: any = new Date(a.date+' '+a.startTime);
                    const d2: any = new Date(b.date+' '+b.endTime);
                    return d1 - d2;
                });
            },
            locationList,
            exportFile,
            loadFile,
        }
    },
})
</script>

<style lang="less" scoped>
.menu {
    & > button, & label {
        @apply p-5 text-xl text-gray-700 hover:bg-sky-400 hover:text-white duration-300 cursor-pointer;
    } 
}
.title {
    @apply p-1;
    & > input, & > span, & > select {
        @apply inline-block mx-1;
    }
    & > input, & > select {
        @apply p-1 border-2 rounded w-16;
    }
}
.user {
    @apply p-1;
    & > input, & > span, & > select {
        @apply inline-block mx-1;
    }
    & > input, & > select {
        @apply p-1 border-2 rounded;
    }
}
</style>
