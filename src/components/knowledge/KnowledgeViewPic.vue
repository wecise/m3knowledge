<template>
    <el-container>
        <el-main v-if="model">
            <el-image 
                v-loading="loading"
                style="width: 100%; height: 100%"
                :src="src" 
                :preview-src-list="srcList"
                @load="loading=false"
                @error="loading=false">
            </el-image>
        </el-main>
        <el-footer style="height:30px;line-height:30px;">
            <span><i class="el-icon-user"></i> {{model.author}}</span>
            <el-divider direction="vertical"></el-divider>
            编辑于 {{ model | pickTime }}
        </el-footer>
    </el-container>
</template>

<script>

export default {
    name: 'KnowledgeViewPic',
    props: {
        model: Object
    },
    data(){
        return {
            loading: true
        }   
    },
    computed:{
        src(){
            return `/static${this.model.fullname}`;
        },
        srcList(){
            return [`/static${this.model.fullname}`];
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
        }
    }
}
</script>

<style scoped>
    .el-container{
        height: 80vh;
    }
    .el-main{
        overflow: hidden;
    }
</style>
