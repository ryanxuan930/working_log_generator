<template>
    <div class="bg-gray-100 h-screen flex flex-col">
        <div class="navbar">
            <div>體育組工作日誌產生器</div>
            <div class="flex-grow"></div>
        </div>
        <div class="flex-grow overflow-auto">
            <LogGenerator></LogGenerator>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import LogGenerator from '../components/LogGenerator.vue';

export default defineComponent({
    setup() {
        return {

        }
    },
    components: {
        LogGenerator,
    }
});
</script>

<style lang="less" scoped>
.navbar {
    @apply flex p-3 bg-sky-500 text-white text-2xl shadow-lg bg-opacity-90;
    & > button {
        @apply block mx-2 hover:text-gray-200 duration-300;
    }
}
</style>