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
        <el-main style="padding:0px;">
            <el-container style="height:100%;">
                <el-main style="padding:0px;overflow:hidden;" ref="mainView">
                    <KnowledgeList :model="search.result" ref="viewListRef" v-if="search.result"></KnowledgeList>
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
                            <KnowledgeTopN ref="topn"></KnowledgeTopN>
                        </el-header>
                        <el-main style="padding:0px;">
                            <KnowledgeTree ref="tree"></KnowledgeTree>
                        </el-main>
                    </el-container>
                </el-aside>
            </el-container>
        </el-main>
    </el-container>    
</template>

<script>
import _ from 'lodash';
import KnowledgeTopN from './KnowledgeTopn';
import KnowledgeTree from './KnowledgeTree';
import KnowledgeList from './KnowledgeList';

export default {
    name: 'index',
    components: {
        KnowledgeList,
        KnowledgeTopN,
        KnowledgeTree
    },
    data(){
        return {
            model:{},
            search: {
                term: "",
                result: [],
                loading: true
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
            /* try{
                if(_.includes(['md','txt','log'],data.ftype)){
                    
                    let wnd = null;

                    try{
                        if(jsPanel.activePanels.getPanel(`jsPanel-${data.name}`)){
                            jsPanel.activePanels.getPanel(`jsPanel-${data.name}`).close();
                        }
                    } catch(error){

                    }
                    finally{
                        wnd = maxWindow.winApp(data.name, `<div id="mdView"></div>`, null,null);
                    }

                    new Vue({
                        delimiters: ['#{', '}#'],
                        data: {
                            model: null,
                            item: data
                        },
                        template: `<el-container style="width:100%;height:100%;">
                                        <el-main style="overflow:hidden;padding:0px;">
                                            <knowledge-view :model="model" ref="viewRef" v-if="!_.isEmpty(model)"></knowledge-view>
                                        </el-main>
                                        <el-footer style="height:30px;line-height:30px;">
                                            <span><i class="el-icon-user"></i> #{item.author}#</span>
                                            <el-divider direction="vertical"></el-divider>
                                            编辑于 #{ item | pickTime }#
                                        </el-footer>
                                    </el-container>`,
                        filters:{
                            pickTime(item){

                                try{
                                    let mtime = item.mtime;
                                    return moment(mtime).format(mx.global.register.format);
                                } catch(err){
                                    return moment(item.vtime).format(mx.global.register.format);
                                }
                            }
                        },
                        created(){
                            this.model = {item:data, content:fsHandler.fsContent(data.parent, data.name)};
                        }
                    }).$mount("#mdView");
                    
                } else if(_.includes(['pdf'],data.ftype)){
                    let contents = `<section class="is-vertical el-container" style="width:100%;height:100%;">
                                        <main class="el-main" style="overflow:hidden;padding:0px;">
                                            <iframe src="/fs${data.fullname}?type=open&issys=${window.SignedUser_IsAdmin}" style="width: 100%;height: 100%;" frameborder="0"></iframe>
                                        </main>
                                        <footer class="el-footer" style="height:30px;line-height:30px;"></footer>
                                    </section>`;

                    let wnd = maxWindow.winApp(data.name, contents, null,null);
                } else if(_.includes(['png','gif','jpg','jpeg'],data.ftype)){
                    
                    let wnd = maxWindow.winApp(data.name, `<div id="picView"></div>`, null,null);
                    new Vue({
                        delimiters: ['#{', '}#'],
                        data: {
                            item: data,
                            loading: true,
                            url: `/fs${data.fullname}?type=open&issys=${window.SignedUser_IsAdmin}`,
                            srcList: [`/fs${data.fullname}?type=open&issys=${window.SignedUser_IsAdmin}`]
                        },
                        template: `<el-container style="width:100%;height:100%;">
                                        <el-main style="overflow:hidden;background:#333333;">
                                            <el-image 
                                                v-loading="loading"
                                                style="width: 100%; height: 100%"
                                                :src="url" 
                                                :preview-src-list="srcList"
                                                @load="loading=false"
                                                @error="loading=false">
                                            </el-image>
                                        </el-main>
                                        <el-footer style="height:30px;line-height:30px;">
                                            <span><i class="el-icon-user"></i> #{item.author}#</span>
                                            <el-divider direction="vertical"></el-divider>
                                            编辑于 #{ item | pickTime }#
                                        </el-footer>
                                    </el-container>`,
                        filters:{
                            pickTime(item){

                                try{
                                    let mtime = item.mtime;
                                    return moment(mtime).format(mx.global.register.format);
                                } catch(err){
                                    return moment(item.vtime).format(mx.global.register.format);
                                }
                            }
                        },
                        created(){
                            this.model = {item:data, content:fsHandler.fsContent(data.parent, data.name)};
                        }
                    }).$mount("#picView");

                } else if(_.includes(['mov','mp3','mp4','wav','swf'],data.ftype)){
                    
                    let wnd = maxWindow.winApp(data.name, `<div id="movView"></div>`, null,null);

                    new Vue({
                        delimiters: ['#{', '}#'],
                        data: {
                            item: data,
                            url: `/fs${data.fullname}?type=open&issys=${window.SignedUser_IsAdmin}`
                        },
                        template: `<el-container style="width:100%;height:100%;">
                                        <el-main style="overflow:hidden;background:#333333;">
                                            <video :src="url" width="100%" height="100%" 
                                                controls="controls" autoplay
                                                style="background-image: url(/fs/assets/images/files/png/matrix.png?type=open&issys=true);
                                                        background-repeat: no-repeat;
                                                        background-position-x: center;
                                                        background-position-y: center;">
                                                Your browser does not support the video tag.
                                            </video>
                                        </el-main>
                                        <el-footer style="height:30px;line-height:30px;">
                                            <span><i class="el-icon-user"></i> #{item.author}#</span>
                                            <el-divider direction="vertical"></el-divider>
                                            编辑于 #{ item | pickTime }#
                                        </el-footer>
                                    </el-container>`,
                        filters:{
                            pickTime(item){

                                try{
                                    let mtime = item.mtime;
                                    return moment(mtime).format(mx.global.register.format);
                                } catch(err){
                                    return moment(item.vtime).format(mx.global.register.format);
                                }
                            }
                        },
                        created(){
                            this.model = {item:data, content:fsHandler.fsContent(data.parent, data.name)};
                        }
                    }).$mount("#movView");

                } else {
                    let url = `/fs${data.fullname}?type=download&issys=true`;
                    window.open(url,"_blank");
                }
            } catch(err){

            } finally{
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
                    
                    fsHandler.fsUpdateAttrAsync(data.parent,data.name,attr).then((rtn)=>{
                        this.onSearch();
                    });
                    
                } catch(err){

                }
            } */

        }
    }
}
</script>

<style scoped>

</style>
