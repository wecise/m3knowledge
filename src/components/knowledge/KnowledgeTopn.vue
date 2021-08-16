<template>
    <el-container style="height:100%;">
        <el-header style="height:50px;line-height:50px;padding:0px 10px;">
            <h5 style="margin:0px;">
                <el-link :underline="false" @click="initData" style="color:#000000;">热点排名</el-link>
            </h5>
        </el-header>
        <el-main style="padding:0px 10px; height: 100%;overflow-x:hidden;display: flex;
                        flex-wrap: wrap;
                        overflow: auto;
                        align-content: flex-start;">
            <template v-if="knowledge.list">
                <el-tooltip placement="top" :content="item.name" 
                    v-for="(item, index) in knowledge.list"
                    :key="item.id">
                    <el-button type="text" 
                        @click="onOpen(item)" 
                        style="height:30px;color:#555555;width: 100%;text-align: left;margin-left: 10px;">
                        {{index+1}}. {{item.name | pickShortLabel}} <span style="font-size:8px;color:#999999;"> 阅读({{item.rate}})</span>
                    </el-button>
                </el-tooltip>
            </template>
        </el-main>
    </el-container>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'KnowledgeTopN',
    data() {
        return {
            knowledge: {
                list:[]
            },
            timer: null
        }
    },
    filters: {
        formatTime(item){
            return this.moment(item).format("YYYY-MM-DD HH:mm:ss");
        },
        pickShortLabel(item){
            return _.truncate(item, {
                        'length': 20,
                        'omission': ' ...'
                    });
        }
    },
    created(){
        this.initData();

        this.timer = setInterval(()=>{
                        this.initData();
                    },30 * 1000);
    },
    methods: {
        initData(){
            
            this.m3.callFS("/matrix/m3knowledge/knowledgeListTopN.js",encodeURIComponent('')).then((rtn)=>{
                this.$set(this.knowledge,'list', rtn.message);
            })
            
        },
        onOpen(data){
            this.$emit("open-doc",data);
        }
    },
    beforeDestroy() {
        clearInterval(this.timer);
    }
}
</script>

<style scoped>

</style>