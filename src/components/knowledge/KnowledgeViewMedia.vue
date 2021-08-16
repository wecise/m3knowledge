<template>
    <el-container>
        <el-main v-if="model">
            <video :src="src" width="100%" height="100%" 
                controls="controls" autoplay
                style="background-image: url(/static/assets/images/files/png/matrix.png);
                        background-repeat: no-repeat;
                        background-position-x: center;
                        background-position-y: center;">
                Your browser does not support the video tag.
            </video>
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
    name: 'KnowledgeViewMedia',
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