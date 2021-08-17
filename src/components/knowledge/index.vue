<template>
    <el-container style="height: calc(100vh - 80px);padding: 10px;">
        <el-header style="height:40px;line-height:40px;padding:0px;">
            <el-input placeholder="请输入搜索关键字" 
                v-model="search.term" 
                v-loading="search.loading"
                @clear="onClear"
                @keyup.enter.native="onSearch"
                clearable>
                <el-button slot="append" type="primary" icon="el-icon-search" @click="onSearch"></el-button>
            </el-input>
        </el-header>
        <el-main style="padding:0px;overflow: hidden;background-color:#f2f2f2;">
            <el-container style="height:100%;">
                <el-main style="padding:0px;overflow:hidden;" ref="mainView">
                    <KnowledgeList :model="search.result" ref="viewListRef" @open-doc="onOpen"  v-if="search.result"></KnowledgeList>
                    <div v-else>
                        <h4>很抱歉，没有找到与 {{search.term}} 相关的记录。</h4>
                        <p>温馨提示：  
                        请检查您的输入是否正确
                        如有任何意见或建议，请及时反馈给我们。
                            </p>
                    </div>
                </el-main>
                <el-aside style="width:300px;overflow:hidden;background:#f2f2f2;margin:0px -10px 0px 10px;" ref="leftView">
                    <el-container style="height:100%;">
                        <el-header style="height:200px;line-height:200px;padding:0px;">
                            <KnowledgeTopN @open-doc="onOpen"></KnowledgeTopN>
                        </el-header>
                        <el-main style="padding:0px;">
                            <KnowledgeTree @open-doc="onOpen" @init="onInit"></KnowledgeTree>
                        </el-main>
                    </el-container>
                </el-aside>
            </el-container>
        </el-main>
        <el-dialog :title="dialog.open.data.name" 
                :visible.sync="dialog.open.show" 
                :destroy-on-close="true"
                :append-to-body="true"
                :close-on-click-modal="false"
                :close-on-press-escape="false"
                width="80%"
                top="0"
                center
                custom-class="knowledge-view-dialog"
                v-if="dialog.open.show">
                <template v-if="dialog.open.data">
                    <KnowledgeView :model="dialog.open.data" v-if="['md','log','txt','svg'].includes(dialog.open.data.ftype)"></KnowledgeView>
                    <KnowledgeViewPdf :model="dialog.open.data" v-else-if="['pdf'].includes(dialog.open.data.ftype)"></KnowledgeViewPdf>
                    <KnowledgeViewPic :model="dialog.open.data" v-else-if="['png','gif','jpg','jpeg'].includes(dialog.open.data.ftype)"></KnowledgeViewPic>
                    <KnowledgeViewMedia :model="dialog.open.data" v-else-if="['mov','mp3','mp4','wav','swf'].includes(dialog.open.data.ftype)"></KnowledgeViewMedia>
                    <KnowledgeViewOffice :model="dialog.open.data" v-else-if="['docx','doc','xlsx','xls','pptx','ppt'].includes(dialog.open.data.ftype)"></KnowledgeViewOffice>
                    <KnowledgeViewOther :model="dialog.open.data" v-else></KnowledgeViewOther>
                </template>
        </el-dialog>
    </el-container>    
</template>

<script>
import _ from 'lodash';
import KnowledgeTopN from './KnowledgeTopn';
import KnowledgeTree from './KnowledgeTree';
import KnowledgeList from './KnowledgeList';
import KnowledgeView from './KnowledgeView';
import KnowledgeViewPdf from './KnowledgeViewPdf';
import KnowledgeViewMedia from './KnowledgeViewMedia';
import KnowledgeViewPic from './KnowledgeViewPic';
import KnowledgeViewOther from './KnowledgeViewOther';
import KnowledgeViewOffice from './KnowledgeViewOffice';

export default {
    name: 'index',
    components: {
        KnowledgeList,
        KnowledgeTopN,
        KnowledgeTree,
        KnowledgeView,
        KnowledgeViewPdf,
        KnowledgeViewMedia,
        KnowledgeViewPic,
        KnowledgeViewOther,
        KnowledgeViewOffice
    },
    data(){
        return {
            model:{},
            search: {
                term: "",
                result: [],
                loading: true
            },
            dialog: {
                open: {
                    show: false
                } 
            }
        }
    },
    watch: {
        'search.term':function(val,oldVal){
            if(_.isEmpty(val)){
                this.onClear();
            }
        }
    },
    mounted() {
        this.onSearch();
    },
    methods: {
        onInit(data){
            this.model = data;
        },
        onSearch(){
            try{
                this.m3.callFS("/matrix/m3knowledge/searchByTerm.js",encodeURIComponent(this.search.term)).then((rt)=>{
                    let rtn = rt.message;
                    
                    if(_.isEmpty(rtn)){
                        this.search.result = [];
                    } else {
                        this.search.result = rtn;
                    }
                });
                
            } catch(err){
                this.search.result = [];
            } finally {
                this.search.loading = false;
            }
        },
        onClear(){
            this.search.term = "";
            this.onSearch();
        },
        onOpen(data){
            
            this.dialog.open.data = data;
            this.dialog.open.show = true;

            // 更新rate
            try{
                let attr = null;
                
                if(_.isEmpty(data.attr)){
                    attr = {remark: '', rate:1};
                } else {
                    attr = _.attempt(JSON.parse.bind(null, data.attr));
                    if(attr.rate){
                        _.extend(attr, {rate:attr.rate + 1});    
                    } else {
                        _.extend(attr, {rate:1}); 
                    }
                }
                
                this.m3.dfsUpdateAttr({parent:data.parent,name:data.name,attr:attr}).then(rtn=>{
                    this.onSearch();
                });
                
            } catch(err){
                console.error(err);
            }

        }
    }
}
</script>

<style scoped>

</style>

<style>
    .el-dialog__wrapper{
        overflow: hidden!important;;
        height: 80vh!important;
    }
    .knowledge-view-dialog.el-dialog{
        height: 100%;    
    }
    .knowledge-view-dialog.el-dialog > .el-dialog__body{
        height: calc(100% - 80px); 
    }
    .knowledge-view-dialog.el-dialog > .el-dialog__body > .el-container{
        height: 100%; 
    }
</style>
