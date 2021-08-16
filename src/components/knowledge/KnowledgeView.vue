<template>
    <el-container>
        <el-main style="background: #f2f2f2;" v-if="doc">
            <mavonEditor v-model="doc.content" style="height: 100%"></mavonEditor>
        </el-main>
        <el-footer style="height:30px;line-height:30px;">
            <span><i class="el-icon-user"></i> {{doc.author}}</span>
            <el-divider direction="vertical"></el-divider>
            编辑于 {{ doc | pickTime }}
        </el-footer>
    </el-container>
</template>

<script>
import _ from 'lodash';
import { mavonEditor } from 'mavon-editor';
import 'mavon-editor/dist/css/index.css';

export default {
    name: 'KnowledgeView',
    props: {
        model: Object
    },
    components: {
        mavonEditor
    },
    data(){
        return {
            mode:"view",
            doc: {},
            tip: {
                message: "",
                loading: false
            }
        }
    },
    filters:{
        pickTime(item){
            try{
                let mtime = item.mtime;
                return window.moment(mtime).format(window.global.register.format);
            } catch(err){
                return window.moment(item.vtime).format(window.global.register.format);
            }
        },
        pickName(name){
            try{
                return _.head(name.split(".md"));
            } catch(err){
                return name;
            }
        }
    },
    created(){
        this.doc = this.model;
        this.m3.dfsRead({parent:this.model.parent, name: this.model.name}).then(res=>{
                _.extend(this.doc, {content:res});
            }).catch(err=>{
                console.error(err)
            })
    },
    methods: {
        onSaveNow(){
            this.tip.loading = true;
            this.tip.message = "更新中...";

            try{
                let attr = null;
                
                if(_.isEmpty(this.model.item.attr)){
                    attr = {remark: '', rate:1};
                } else {
                    attr = _.attempt(JSON.parse.bind(null, this.model.item.attr));
                    if(attr.rate){
                        _.extend(attr, {rate:attr.rate + 1});    
                    } else {
                        _.extend(attr, {rate:1}); 
                    }
                }
                
                /* fsHandler.fsNewAsync(this.model.item.ftype, this.model.item.parent, this.model.item.name, this.editor.getValue(), attr).then( (rtn)=>{
                    if(rtn == 1){
                        this.tip.message = "更新成功";
                    } else {
                        this.tip.message = "更新失败";
                    }

                    setTimeout(()=>{
                        this.tip.message = "";
                        this.tip.loading = false;
                    },1000)
                } ); */
                
            } catch(err){
                console.error(err);
            }
            
        },
        onSave: _.debounce(function(){
                    const self = this;
                    self.tip.loading = true;
                    self.onSaveNow();
                },3000)
    }

}
</script>

<style>
    .knowledge-view-dialog.el-dialog{
        width: 80%;
    }
</style>