<template>
    <el-container>
        <el-main v-if="model">
            <iframe :src="src" style="width: 100%;height: 100%;" frameborder="0"></iframe>
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
    name: 'KnowledgeViewPdf',
    props: {
        model: Object
    },
    computed:{
        src(){
            return `/static${this.model.fullname}`;
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
