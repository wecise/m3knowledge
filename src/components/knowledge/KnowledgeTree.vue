<template>
    <el-container style="height:100%;">
        <el-header style="height:50px;line-height:50px;padding:0px 10px;display:flex;background:#f2f2f2;">
            <span style="width:40%;">
                <h5 style="margin:0px;">知识分类</h5>
            </span>
            <span style="width:60%;text-align:right;">
                <el-button type="text" @click="initData" icon="el-icon-refresh"></el-button>
                <el-button type="text" @click="onNewDir({'fullname':root})" icon="el-icon-folder-add"></el-button>
                <el-button type="text" @click="onNewFile({'fullname':root})" icon="el-icon-plus"></el-button>
            </span>
        </el-header>
        <el-main style="padding:0px 10px; height: 100%;overflow-x:hidden;">
            <el-tree :data="treeData" 
                    :props="defaultProps" 
                    node-key="fullname"
                    :highlight-current="true"
                    :default-expand-all="true"
                    @node-click="onNodeClick"
                    :filter-node-method="onFilterNode"
                    :expand-on-click-node="false"
                    style="background:transparent;"
                    ref="tree">
                <span slot-scope="{ node, data }" style="width:100%;height:30px;line-height: 30px;"  @mouseenter="onMouseEnter(data)" @mouseleave="onMouseLeave(data)">
                    <span v-if="data.ftype=='dir'">
                        <i class="el-icon-folder" style="color:#FFC107;"></i> 
                        <el-tooltip placement="top" :content="node.label">
                            <span> {{node.label | pickShortLabel}}
                                <span v-if="data.children">({{data.children.length}})</span>
                            </span>
                        </el-tooltip>
                        <el-dropdown v-show="data.show" style="float:right;width:14px;margin:0 5px;" trigger="click">
                            <span class="el-dropdown-link">
                                <i class="el-icon-more el-icon--right" style="color:#000000;"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown">
                                <el-dropdown-item @click.native="onDelete(data,$event)" icon="el-icon-delete">删除</el-dropdown-item>
                                <el-dropdown-item @click.native="onNodeClick(data)" icon="el-icon-refresh">刷新</el-dropdown-item>
                                <el-dropdown-item @click.native="onNewFile(data,$event)" icon="el-icon-plus">新建文件</el-dropdown-item>
                                <el-dropdown-item @click.native="onNewDir(data,$event)" icon="el-icon-folder-add">新建目录</el-dropdown-item>
                                <el-dropdown-item @click.native="onUpload(data,$event)" icon="el-icon-upload">上传</el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </span>
                    <span v-else>
                        <i class="el-icon-c-scale-to-original" style="color:#0088cc;"></i> 
                        <el-tooltip placement="top" :content="node.label">
                            <span> {{node.label | pickShortLabel}}</span>
                        </el-tooltip>
                        <el-button v-show="data.show" type="text" @click.stop="onDownload(data,$event)" style="float:right;width:14px;margin:0 5px;" icon="el-icon-download"></el-button>
                        <el-button v-show="data.show" type="text" @click.stop="onDelete(data,$event)" style="float:right;width:14px;margin:0 5px;" icon="el-icon-delete"></el-button>
                    </span>
                </span>   
            </el-tree>
        </el-main>
    </el-container>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'KnowledgeTopN',
    data() {
        return {
            defaultProps: {
                children: 'children',
                label: 'name'
            },
            treeData: [],
            root: `/opt/knowledge`,
            filterText: ""
        }
    },
    filters: {
        pickShortLabel(item){
            return _.truncate(item, {
                        'length': 16,
                        'omission': ' ...'
                    });
        }
    },
    watch: {
        filterText(val) {
            if(_.isEmpty(val)){
                this.initData();
            } else {
                this.$refs.tree.filter(val);
            }
        }
    },
    created(){
        this.initData();
    },
    methods: {
        onMouseEnter(item){
            this.$set(item, 'show', true)
        },
        onMouseLeave(item){
            this.$set(item, 'show', false)
        },
        onRefresh(item,index){
            
            if(_.isEmpty(index)){
                this.initData();
            } else {
                this.m3.dfsList(item.fullname).then((rtn)=>{
                    this.$set(item, 'children', rtn);
                });
            }
        
        },
        onNewDir(item,index){
            this.$prompt('请输入目录名称', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消'
                }).then(({ value }) => {

                if(_.isEmpty(value)){
                    this.$message({
                        type: 'warning',
                        message: '请输入目录名称！'
                    });
                    return false;
                }

                let _attr = {remark: '', rate: 0};

                /* fsHandler.fsNewAsync('dir', item.fullname, value, null, _attr).then((rtn)=>{
                    if(rtn == 1){
                        this.$message({
                            type: "success",
                            message: "新建目录成功！"
                        })

                        // 刷新
                        this.onRefresh(item,index);

                    } else {
                        this.$message({
                            type: "error",
                            message: "新建目录失败，" + rtn.message
                        })
                    }
                }); */
                
                
                }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消输入'
                });       
                });
            
        },
        onNewFile(item,index){
            this.$prompt('请输入知识文件名称', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消'
                }).then(({ value }) => {
                if(_.isEmpty(value)){
                    this.$message({
                        type: 'warning',
                        message: '请输入名称！'
                    });
                    return false;
                }

                let _attr = {remark: '', rate: 0};

                this.m3.callFS("/matrix/m3knowledge/getDefaultContent.js",null).then((rtn)=>{
                    let content = rtn.message;

                    /* fsHandler.fsNewAsync('md', item.fullname, [value,'md'].join("."), content, _attr).then((rt)=>{
                        if(rt == 1){
                            this.$message({
                                type: "success",
                                message: "新建成功！"
                            })

                            // 刷新
                            this.onRefresh(item,index);

                        } else {
                            this.$message({
                                type: "error",
                                message: "新建失败，" + rt.message
                            })
                        }
                    }); */
                
                });

                
                }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消输入'
                });       
                });
        },
        onDelete(item,index){
            
            this.$confirm(`确认要删除该知识：${item.name}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                this.m3.dfsDelete({parent:item.parent,name:item.name}).then((rtn)=>{
                    if(rtn == 1){
                        this.$message({
                            type: "success",
                            message: "删除成功！"
                        })
                        
                        // 刷新
                        try{
                            this.m3.dfsList({fullname:item.parent}).then(res=>{
                                let childrenData = res.message;
                                let parent = this.$refs.tree.getNode(item.parent)
                                this.$set(parent.data, 'children', childrenData);
                            })
                            
                        } catch(err){
                            this.initData();
                        }
                        
                        

                    } else {
                        this.$message({
                            type: "error",
                            message: "删除失败！"
                        })
                    }
                });
                

            }).catch((err) => {
                console.log(err)
            });
        },
        onUpload(item,index){
            /* const self = this;

            let wnd = null;
            let wndID = `jsPanel-upload-${objectHash.sha1(item.id)}`;

            try{
                if(jsPanel.activePanels.getPanel(wndID)){
                    jsPanel.activePanels.getPanel(wndID).close();
                }
            } catch(error){

            }
            finally{
                wnd = maxWindow.winUpload('文件上传', `<div id="${wndID}"></div>`, null, null);
            }
            
            new Vue({
                delimiters: ['#{', '}#'],
                template:   `<el-container>
                                <el-main>
                                    <el-upload drag
                                        multiple
                                        show-file-list="false"
                                        :action="upload.url"
                                        :data="upload.ifIndex"
                                        :on-success="onSuccess"
                                        :on-error="onError"
                                        :before-upload="onBeforeUpload"
                                        :on-remove="onRemove"
                                        list-type="text"
                                        name="uploadfile">
                                        <i class="el-icon-upload"></i>
                                    </el-upload>
                                </el-main>
                                <el-footer>
                                    <i class="fas fa-clock"></i> 上传文件：#{upload.fileList.length}# 
                                </el-footer>
                            </el-container>`,
                data: {
                    upload: {
                        url: `/fs${item.fullname}?issys=true`,
                        fileList: [],
                        ifIndex: {index:true}
                    }
                },
                methods: {
                    onBeforeUpload(file){
                        
                    },
                    onSuccess(res,file,FileList){
                        this.upload.fileList = FileList;
                        
                        _.forEach(FileList,(v)=>{
                            let attr = {remark: '', rate:0};
                            fsHandler.fsUpdateAttrAsync(item.parent, v.name, attr);
                        })

                        // 刷新
                        self.onRefresh(item,index);

                        this.$message({
                            type: "success",
                            dangerouslyUseHTMLString: true,
                            message: `上传成功！`
                        })

                    },
                    onError(res,file,FileList){
                        this.$message({
                            type: "error",
                            dangerouslyUseHTMLString: true,
                            message: `上传失败，请确认！`
                        })
                    },
                    onRemove(file, fileList) {
                        fsHandler.fsDeleteAsync(item.fullname,file.name).then((rtn)=>{
                            if(rtn == 1){
                                // 刷新
                                self.onRefresh(item,index);
                            }
                        });
                        
                    },
                    onPreview(file) {
                        
                    }
                }
            }).$mount(`#${wndID}`); */
            
        },
        onDownload(item,index){
            let url = `/fs/${item.fullname}?type=download&issys=true`;
            window.open(url,"_blank");
        },
        onFilterNode:_.debounce(function(value, data) {
            if (!value) return true;
            try{
                this.m3.callFS("/matrix/fs/getFsByTerm.js", encodeURIComponent(value)).then((rtn)=>{
                    this.treeData = rtn.message;
                });
            } catch(err){
                this.treeData = [];
            }
        },1000),
        onNodeClick(data){
            
            try{
                // 文件
                if(!data.isdir) {
                    
                    // 打开操作
                    this.$root.onOpen(data);

                } 
                // 目录
                else {
                    this.m3.dfsList({fullname:data.fullname}).then((rtn)=>{
                        this.$set(data, 'children', rtn);
                    });
                }
            } catch(err){
                console.log(err)
            }
        },
        initData(){
            this.m3.callFS("/matrix/m3knowledge/getFsForTree.js", encodeURIComponent(this.root)).then((rt)=>{
                let rtn = rt.message;

                this.treeData = _.map(rtn,(v)=>{
                    return _.extend(v,{show:false});
                });
                
                try{
                    // let childrenData = fsHandler.fsList(this.treeData[2].fullname);
                    // this.$set(this.treeData[2], 'children', childrenData);

                    // 默认首页
                    let homeNode = _.find(this.treeData,{name: '知识通介绍.md'}) || _.find(_.flattenDeep(_.map(this.treeData,'children')),{name: '知识通介绍.md'});
                    
                    this.m3.dfsList({fullname:homeNode.parent}).then(res=>{
                        let item = _.extend(homeNode,{
                            size: _.find(res.message,{name: homeNode.name}).size || 0
                        });
                    })
                    
                    
                    this.m3.dfsRead({parent:homeNode.parent, name:homeNode.name}).then(res=>{
                        this.$root.model = {item:homeNode, content: res.message};
                    })

                } catch(err){
                    this.$root.model = null;
                }
            });
            
        }
    }
}
</script>

<style scoped>

</style>