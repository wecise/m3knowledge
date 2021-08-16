<template>
    <el-container style="height:100%;">
        <el-header style="height:50px;line-height:50px;padding:0px 10px;display:flex;background:#f2f2f2;">
            <span style="width:40%;">
                <h5 style="margin:0px;">知识分类</h5>
            </span>
            <span style="width:60%;text-align:right;">
                <el-button type="text" @click="initData" icon="el-icon-refresh"></el-button>
                <el-button type="text" @click="onNewDir({'parent':root})" icon="el-icon-folder-add"></el-button>
                <el-button type="text" @click="onNewFile({'parent':root})" icon="el-icon-plus"></el-button>
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
        <el-dialog title="上传" 
            :visible.sync="dialog.upload.show" 
            :destroy-on-close="true"
            :append-to-body="true"
            :close-on-click-modal="false"
            :close-on-press-escape="false"
            width="40%"
            center
            custom-class="knowledge-upload-dialog"
            v-if="dialog.upload.show">
            <el-container>
                <el-main style="text-align: center;">
                    <el-upload drag
                        multiple
                        :action="dialog.upload.url"
                        :data="dialog.upload.ifIndex"
                        :on-success="onUploadSuccess"
                        :on-error="onUploadError"
                        :on-remove="onUploadRemove"
                        :list-type="text"
                        :show-file-list="true"
                        name="uploadfile">
                        <i class="el-icon-upload"></i>
                    </el-upload>
                </el-main>
                <el-footer>
                    <i class="fas fa-clock"></i> 上传文件：{{dialog.upload.fileList.length}}
                </el-footer>
            </el-container>
    </el-dialog>
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
            filterText: "",
            dialog: {
                upload: {
                    show: false,
                    url: '',
                    fileList: [],
                    ifIndex: {index:true}
                } 
            }
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
                this.m3.dfsList(item.fullname).then(rtn=>{
                    this.$set(item, 'children', rtn.message);
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

                let ftype = 'dir';
                let attr = {remark: '', rate: 0};

                let param = {
                    parent: item.parent, name: value, 
                    data: { content: '', ftype: ftype, attr: attr, index: true }    
                };

                this.m3.dfsWrite(param).then((rtn)=>{
                    this.$message({
                        type: "success",
                        message: "新建目录成功！"
                    })

                    // 刷新
                    this.onRefresh(item,index);
                        
                }).catch(err=>{
                    this.$message({
                        type: "error",
                        message: "新建目录失败，" + err
                    })
                });
                
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消新建目录操作'
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

                this.m3.callFS("/matrix/m3knowledge/getDefaultContent.js",null).then(rtn=>{
                    
                    let ftype = 'md';
                    let attr = {remark: '', rate: 0};
                    let name = [value,ftype].join(".");

                    let param = {
                        parent: item.fullname, name: name, 
                        data: { content: rtn.message, ftype: ftype, attr: attr, index: true }    
                    };

                    this.m3.dfsWrite(param).then(res=>{
                        
                        this.$message({
                            type: "success",
                            message: "新建成功！"
                        })

                        // 刷新
                        this.onRefresh(item,index);

                    }).catch(err=>{
                        this.$message({
                            type: "error",
                            message: "新建失败，" + err
                        })
                    })
                
                })

                
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消新建文件操作'
                });       
            });
        },
        onDelete(item,index){
            
            this.$confirm(`确认要删除该知识：${item.name}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                this.m3.dfsDelete({parent:item.parent,name:item.name}).then(rtn=>{
                    
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
                        
                }).catch(err=>{
                    this.$message({
                        type: "error",
                        message: "删除失败，" + err
                    })
                })
                

            }).catch((err) => {
                console.log(err)
            });
        },
        onUpload(item,index){
            this.dialog.upload.data = item;
            this.dialog.upload.data.url = `/fs${item.fullname}?issys=true`;
            this.dialog.upload.show = true;
        },
        onUploadSuccess(res,file,FileList){
            this.dialog.upload.fileList = FileList;
            
           /*  _.forEach(FileList,(v)=>{
                let attr = {remark: '', rate:0};
                fsHandler.fsUpdateAttrAsync(item.parent, v.name, attr);
            }) */

            // 刷新
            this.onRefresh(this.dialog.upload.data, null);

            this.$message({
                type: "success",
                dangerouslyUseHTMLString: true,
                message: `上传成功！`
            })

        },
        onUploadError(){
            this.$message({
                type: "error",
                dangerouslyUseHTMLString: true,
                message: `上传失败，请确认！`
            })
        },
        onUploadRemove(file, fileList) {
            
            this.m3.dfsDelete({parent:this.dialog.upload.data.parent, name:file.name}).then(rtn=>{
                this.onRefresh(this.dialog.upload.data,null);
            });
            
        },
        onDownload(item,index){
            let url = `/static${item.fullname}`;
            window.open(url,"_blank");
        },
        onFilterNode:_.debounce(function(value, data) {
            if (!value) return true;
            try{
                this.m3.callFS("/matrix/m3knowledge/getFsByTerm.js", encodeURIComponent(value)).then((rtn)=>{
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
                    this.$emit("open-doc",data);

                } 
                // 目录
                else {
                    this.m3.dfsList({fullname:data.fullname}).then((rtn)=>{
                        this.$set(data, 'children', rtn.message);
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
                    this.m3.dfsList(this.treeData[2].fullname).then(res=>{
                        this.$set(this.treeData[2], 'children', res.message);
                    });

                    // 默认首页
                    let homeNode = _.find(this.treeData,{name: '知识通介绍.md'}) || _.find(_.flattenDeep(_.map(this.treeData,'children')),{name: '知识通介绍.md'});
                    
                    this.m3.dfsList({fullname:homeNode.parent}).then(res=>{
                        let item = _.extend(homeNode,{
                            size: _.find(res.message,{name: homeNode.name}).size || 0
                        });
                    })
                    
                    this.m3.dfsRead({parent:homeNode.parent, name:homeNode.name}).then(res=>{
                        this.$emit("init",{item:homeNode, content: res.message});
                    })

                } catch(err){
                    this.$emit("init",null);
                }
            });
            
        }
    }
}
</script>

<style scoped>

</style>