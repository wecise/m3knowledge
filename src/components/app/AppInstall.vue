<template>
    <el-container style="height:calc(100vh - 135px);">
                
        <el-header style="line-height: 60px;">
            <el-tooltip content="刷新">
                <el-button type="default" icon="el-icon-refresh" @click="onRefresh">刷新</el-button> 
            </el-tooltip>
            <el-tooltip content="应用安装">
                <el-button type="success" icon="el-icon-plus" @click="onNewAppInstall">应用安装</el-button> 
            </el-tooltip>
            <el-dropdown style="float:right;">
                <span class="el-dropdown-link">
                    <i class="el-icon-setting el-icon--right"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                    <el-dropdown-item @click.native="onAppExport('mql')">导出应用(MQL)</el-dropdown-item>
                    <el-dropdown-item @click.native="onAppExport('xlsx')">导出应用(Excel)</el-dropdown-item>
                    <el-dropdown-item @click.native="dialog.appImport.show = true;" divided>导入应用</el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </el-header>
        <el-main style="display: flex;flex-wrap: wrap;align-content: flex-start;justify-content: flex-start;padding:0 10px;">
            <template v-if="model">
                <el-button type="default" 
                    style="position: relative;width: 14em;height:10em;border-radius: 8px!important;margin:10px;border: unset;box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);background:#262e48;"
                    v-for="(item,index) in model.list"
                    :key="index">
                    <el-image style="width:64px;height:64px;margin:5px;" :src="item.icon | pickIcon"></el-image>
                    <p style="color:#fff;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;">{{item.cnname}}</p>
                    <div style="position:absolute;top:0px;right:5px;">
                        <el-button type="text"  @click="onAppUpdate(item)">
                            <span class="el-icon-setting"  style="color:#ffffff;"></span> 
                        </el-button>
                    </div>
                </el-button>
            </template>
            <el-dialog :title="dialog.appInstall.action==='new'?'应用安装':'应用编辑 '+dialog.appInstall.data.name"  
                :visible.sync="dialog.appInstall.show" 
                v-if="dialog.appInstall.show" 
                :destroy-on-close="true">
                <el-tabs v-model="dialog.appInstall.tabs.activeName" ref="tabs">
                    <el-tab-pane name="app">
                        <span slot="label" style="font-size:14px;">
                            <i class="el-icon-s-platform"></i> 应用信息
                        </span>
                        <el-container>
                            <el-main style="padding:0px 20px;">
                                <el-form ref="form" :model="dialog.appInstall.data" label-width="80px" >
                                    <el-form-item label="应用类型">
                                        <el-radio-group v-model="dialog.appInstall.data.selected">
                                            <el-radio :label="0">小应用</el-radio>
                                            <el-radio :label="1">URL</el-radio>
                                        </el-radio-group>
                                    </el-form-item>
                                    <el-form-item label="选择应用" v-if="dialog.appInstall.data.selected=='0'">
                                        <el-select v-model="deployedApps.value" placeholder="请选择">
                                            <el-option
                                                v-for="item in deployedApps.list"
                                                :key="item.id"
                                                :label="item.name"
                                                :value="item.id">
                                            </el-option>
                                        </el-select>
                                    </el-form-item>
                                    <el-form-item style="position:absolute;right:10px;">
                                        <el-button type="text" @click="dialog.appInstall.tabs.activeName='icon'" style="background:#444444;border-radius:15px!important;padding:20px;">
                                            <el-image shape="square" fit="scale-down" style="width:64px;" :src="icon.value"></el-image>
                                        </el-button>
                                    </el-form-item>
                                    <el-form-item label="中文名" style="width:75%;">
                                        <el-input v-model="dialog.appInstall.data.cnname"></el-input>
                                    </el-form-item>
                                    <el-form-item label="英文名称" style="width:75%;">
                                        <el-input v-model="dialog.appInstall.data.enname"></el-input>
                                    </el-form-item>
                                    <template v-if="dialog.appInstall.data.selected=='1'">
                                        <el-form-item label="Url" style="width:75%;">
                                            <el-input v-model="dialog.appInstall.data.url"></el-input>
                                        </el-form-item>
                                        <el-form-item label="Target">
                                            <el-radio-group v-model="dialog.appInstall.data.target">
                                                <el-radio label="_blank">打开新窗口</el-radio>
                                                <el-radio label="_parent">当前窗口打开</el-radio>
                                            </el-radio-group>
                                        </el-form-item>
                                    </template>
                                    <el-form-item label="应用分组">
                                        <el-radio-group v-model="dialog.appInstall.data.groups.group">
                                            {{dialog.appInstall.data.groups.group}}
                                            <el-radio :label="item.name" v-for="item in model.groups" :key="item.name">{{item.title}}</el-radio>
                                        </el-radio-group>
                                    </el-form-item>
                                </el-form> 
                            </el-main>
                            <el-footer style="text-align:right;line-height:60px;">
                                <el-button type="default" @click="dialog.appInstall.show = false;">取消</el-button>
                                <el-button type="primary" @click="onAppInstall(dialog.appInstall.data)">{{ dialog.appInstall.action==='edit'?'更新':'安装' }}</el-button>
                                <el-button type="danger" @click="onAppUnInstall(dialog.appInstall.data)" v-if="dialog.appInstall.action==='edit'">卸载应用</el-button>
                            </el-footer>
                        </el-container>
                    </el-tab-pane>
                    <el-tab-pane name="icon">
                        <span slot="label" style="font-size:14px;">
                            <i class="el-icon-picture"></i> 选择图标
                        </span>
                        <el-container style="height:100%;" class="appInstall-icon-list">
                            <el-main style="height:300px;overflow:auto;padding:10px 0px;background:#666666;">
                                <el-radio-group v-model="icon.value" class="mx-app-icon">
                                    <el-button type="default" style="border: unset;width:100px;height:120px;margin:5px;padding:0px;cursor:pointer;background:transparent;" 
                                        v-for="img in icon.list"
                                        :key="img.id"
                                        @click="onTriggerRadioClick(img)"> 
                                        <el-image :src="img | pickExtIcon" fit="fill"  style="width:48px;"></el-image> 
                                        <p>
                                            <el-radio :label="'/static'+img.parent+'/'+img.name" 
                                                :ref="'radio_'+img.id">
                                            </el-radio>
                                        </p>
                                    </el-button>
                                </el-radio-group>
                            </el-main>
                            <el-footer style="padding:20px 0px 50px 0px;display:flex;height:auto;position:releative;">
                                <span style="position:absolute;right:140px;">
                                    <el-button type="default" icon="el-icon-close" @click="dialog.appInstall.tabs.activeName='app';">返回</el-button>
                                    <el-button type="primary" icon="el-icon-refresh" @click="initIconList">刷新图标</el-button>
                                </span>
                                <span style="position:absolute;right:20px;">
                                    <el-upload
                                        multiple
                                        :data="{index:true}"
                                        :action="upload.root+'?issys=true'"
                                        :before-upload="onBeforeUpload"
                                        :on-success="onSuccess"
                                        :on-error="onError"
                                        :show-file-list="false"
                                        name="uploadfile">
                                        <el-button icon="el-icon-upload" type="primary" style="padding-left:20px;" :loading="upload.loading">上传图标</el-button>
                                    </el-upload>
                                </span>
                            </el-footer>
                        </el-container>
                    </el-tab-pane>
                </el-tabs>
            </el-dialog>
        </el-main>
    </el-container>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'AppInstall',
    data(){
        return {
            model: null,
            deployedApps: {
                value: null,
                list: []
            },
            installedApps: {
                list: []
            },
            dialog: {
                appInstall: {
                    loading: false,
                    show: false,
                    action: 'new',
                    data: {
                        selected: '0',
                        cnname: '',
                        enname: '',
                        url: '',
                        target: '',
                        groups: {
                            group: ''
                        }
                    },
                    tabs:{
                        activeName: 'app'
                    }
                }
            },
            icon: {
                value: '',
                list: []
            },
            upload: {
                root: '/assets/images/apps/png',
                url: '/static/assets/images/apps/png',
                fileList: [],
                loading: false
            }
        }
    },
    mounted() {
        this.$nextTick( ()=>{
            this.init(); 
            this.initIconList();
        })
    },
    filters:{
        pickExtIcon(icon) {
            return `/static${icon.parent}/${icon.name}`;
        },
        pickIcon(icon){
            return `/static/assets/images/apps/png/${icon}`;
        }
    },
    methods: {
        onRefresh(){
            this.init();
        },
        onResetForm(formName){
            this.$refs[formName].resetFields();
        },
        init(){
            this.m3.callFS("/matrix/m3appstore/appList.js",null).then( (rtn)=>{
                this.model = rtn.message;
            });

            this.m3.callFS("/matrix/m3appstore/getInstalledApps.js",null).then( (rtn)=>{
                this.installedApps.list = rtn.message;
            });

            this.initDeployedApps();

        },
        initDeployedApps(){
            this.m3.callFS("/matrix/m3appstore/getDeployedApps.js",null).then((rtn)=>{
                this.deployedApps.list = rtn.message;
            });
        },
        initIconList(){
            this.m3.dfsList({fullname:this.upload.root}).then( (rtn)=>{
                this.icon.list = rtn.message;
                this.icon.value = `${this.upload.url}/creative.png`;
            } );   
        },
        
        onTriggerRadioClick(item){
            this.$refs['radio_'+item.id][0].$el.click();
        },
        /* 应用安装 */
        onNewAppInstall(){
            this.dialog.appInstall.show = true;
            this.dialog.appInstall.action = 'new';
            this.dialog.appInstall.data = {
                                            selected: '0',
                                            cnname: '',
                                            enname: '',
                                            url: '',
                                            target: '',
                                            groups: {
                                                group: ''
                                            }
                                        };
        },
        onAppInstall(item){
            
            _.extend(item,{ icon: _.last(this.icon.value.split("/")), action: this.dialog.appInstall.action } );
            
            this.m3.callFS("/matrix/m3appstore/app.js",encodeURIComponent(JSON.stringify(item))).then( (rtn)=>{
                if( _.lowerCase(rtn.status) == "ok"){       
                    this.$message({
                        type: "info",
                        message: "应用更新成功"
                    });
                    
                    this.dialog.appInstall.show = false;
                }
            });
        },
        onAppUnInstall(item) {
            
            this.$confirm(`确认要卸载该应用：${item.name}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                this.m3.callFS("/matrix/m3appstore/app_delete.js",encodeURIComponent(JSON.stringify(item))).then( (rtn)=>{
                    
                    if(rtn.status === 'ok'){
                        
                        this.init();

                    } else {
                        this.$message({
                            type: "error",
                            message: rtn
                        })
                    }

                } );
                
            }).catch(() => {
                this.$message({
                    type: "info",
                    message: "取消卸载操作"
                })
            });

        },
        onAppUpdate(data){
            this.dialog.appInstall.action = 'edit';
            this.dialog.appInstall.data = data;
            this.dialog.appInstall.show = true;
        },
        onAppImport(){
            
            let fileName = this.dialog.appImport.files[0].name;

            this.$confirm(`确认要导入应用：${fileName}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                /* omdbHandler.classDataImport(this.dialog.appImport.files[0]).then( (rtn)=>{
                    if(_.lowerCase(rtn.status) == 'ok'){
                        this.$message({
                            type: "success",
                            message: "导入成功！"
                        });
                    } else {
                        this.$message({
                            type: "error",
                            message: "导入失败：" + rtn.message
                        });
                    }
                } ); */
                
            }).catch(() => {
                this.$message({
                    type: "info",
                    message: "取消导入操作！"
                });
            });
            
        },
        onAppExport(ftype){
            
            /* let template = ftype=='mql'?true:false;

            axios.get(`/mxobject/export?recursive=true&relation_defined=false&filetype=${ftype}&template=${template}&class=%2Fmatrix%2Fportal%2Ftools&ignoreclass=%2Fmatrix%2Ffilesystem&limit=-1`,{
                headers: {
                    "Content-type":"text/csv",
                    "Access-Control-Allow-Origin":"*"
                },
                responseType:"arraybuffer"
            })
            .then((response)=> {
                var blob = new Blob([response.data], ftype=='mql'?{type: "octet/stream"}:{type: "application/vnd.ms-excel"});
                saveAs(blob, `Apps-${moment().format('LLL')}.${ftype}`);
            })
            .catch((error)=> {
                console.error(error);
            }); */
        },
        onBeforeUpload(){
            this.upload.loading = true;
        },
        onSuccess(res,file,FileList){
            this.upload.fileList = FileList;
            this.$message({
                type: "success",
                message: "上传成功！"
            })
            this.upload.loading = false;
            this.initIconList();
        },
        onError(err,file,FileList){
            this.$message({
                type: "error",
                message: "上传失败：" + err
            })
            this.upload.loading = false;
            this.initIconList();
        }
    }
}
</script>

<style>
    .appInstall-icon-list .el-radio .el-radio__label{
        display: none;
    }
</style>
