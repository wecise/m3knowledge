<template>
    <el-container style="height:100%;">
        <el-header style="height:auto;padding:0px;display:flex;flex-wrap: wrap;align-content: flex-start;">
            <el-button type="default" @click="onFilter('all')" style="margin:5px;">全部({{model.length}})</el-button>
            <el-button type="default" 
                v-for="(item, key) in groupedList" 
                :key="key" 
                @click="onFilter(key)" 
                style="margin:5px;">{{ key | pickParentName}}({{ item.length }})</el-button>
        </el-header>
        <el-main style="height:100%;padding:10px 10px 0px 0px;">
                <div v-for="(item, index) in knowledge.list" :key="index">
                    <template v-if="item && item.ftype!='dir'">
                        <el-card shadow="hover" v-if="item.ftype=='md'">
                            <el-button type="text" @click="onOpen(item)"><h4>{{item.name}}</h4></el-button>
                            <div style="display: flex;
                                        height: auto;
                                        line-height: 40px;
                                        padding: 0px 20px 0 0;"> 
                                <div style="width:90%;display: -webkit-box;">
                                    <span><i class="el-icon-user"></i> {{item.author}}</span>
                                    <el-divider direction="vertical"></el-divider>
                                    编辑于  {{ item | pickTime }}
                                    <el-divider direction="vertical"></el-divider>
                                    位置 {{ item.parent }}
                                    <el-divider direction="vertical"></el-divider>
                                    <i class="el-icon-price-tag" style="color:#409eff;"></i>
                                    <TagView domain='knowledge' :model.sync="item.tags" :id="item.id" :limit="4" ></TagView>
                                </div>
                                <div style="width:10%;text-align:right;">
                                    阅读 ( {{item | pickRate}} )
                                </div>
                            </div>
                        </el-card>
                        <el-card shadow="hover" v-else>
                            <el-button type="text" @click="onOpen(item)"><h4>{{item.name}}</h4></el-button>
                            <div style="display: flex;
                                        height: auto;
                                        line-height: 40px;
                                        padding: 0px 20px 0 0;"> 
                                <div style="width:90%;display: -webkit-box;">
                                    <span><i class="el-icon-user"></i> {{item.author}}</span>
                                    <el-divider direction="vertical"></el-divider>
                                    编辑于 {{ item | pickTime }}
                                    <el-divider direction="vertical"></el-divider>
                                    位置 {{ item.parent }}
                                    <el-divider direction="vertical"></el-divider>
                                    <i class="el-icon-price-tag" style="color:#409eff;"></i>
                                    <TagView domain='knowledge' :model.sync="item.tags" :id="item.id" :limit="4" ></TagView>
                                </div>
                                <div style="width:10%;text-align:right;">
                                    阅读 ( {{item | pickRate}} )
                                </div>
                            </div>
                        </el-card>
                    </template>
                    
                </div>
        </el-main>
    </el-container>
</template>

<script>
import _ from 'lodash';
import TagView from '../tags/TagView';

export default {
    name: 'KnowledgeList',
    components:{
        TagView
    },
    props: {
        model: Array
    },
    data(){
        return {
            knowledge: {
                list: []
            }
        }
    },
    computed:{
        groupedList(){
            return _.groupBy(this.model,'parent');
        }
    },
    filters: {
        pickTime(item){
            
            try{
                let mtime = item.mtime;
                return window.moment(mtime).format(window.global.register.format);
            } catch(err){
                return window.moment(item.vtime).format(window.global.register.format);
            }
        },
        pickTags(item){

            if(!_.isEmpty(item.tags)){
                return item.tags.join(",");
            } else {
                return "";
            }
        },
        pickRate(item){
            
            try{
                return _.attempt(JSON.parse.bind(null, item.attr)).rate || 0;
            } catch(err){
                return 0;
            }
        },
        pickParentName(parent){
            try{
                return _.last(parent.split("/"));
            } catch(err){
                return parent;
            }
        }
    },
    watch:{
        model(val,oldVal){
            this.initData();
        }
    },
    created(){
        this.initData();
    },
    methods: {
        initData(){
            let getRate = function(item){
                try{
                    return _.attempt(JSON.parse.bind(null, item.attr)).rate || 0;
                } catch(err){
                    return 0;
                }
            };

            this.knowledge.list = _.orderBy(_.map(this.model,(v)=>{
                
                if( v.ftype == 'md' ){
                    //return _.extend(v,{content:fsHandler.fsContent(v.parent,v.name)});
                    return _.extend(v,{content:'', rate: getRate(v)});
                } else {
                    return _.extend(v,{content:'', rate: getRate(v)});
                }

            }),['rate'],['desc'])
        },
        onFilter(item){
            
            let filtered = [];
            
            if( item == 'all' ){
                filtered = this.model;
            } else {

                filtered = _.filter(this.model,{parent:item});
            }
            
            this.knowledge.list = _.map(filtered,(v)=>{
                
                if( v.ftype == 'md' ){
                    this.m3.dfsRead({parent:v.parent,name:v.name}).then(res=>{
                        return _.extend(v,{content: res});
                    })
                } else {
                    return _.extend(v,{content:''});
                }
            })
        },
        onOpen(item){
            this.$emit("open-doc",item);
        }

    }
}
</script>

<style scoped>

</style>