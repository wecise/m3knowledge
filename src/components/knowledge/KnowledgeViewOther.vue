<template>
    <el-container>
        <el-main v-if="model">
            <el-image 
                v-loading="loading"
                :src="src"
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
    name: 'KnowledgeViewOther',
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
            return `/static/assets/images/files/png/${this.model.ftype}.png`;
        }
    },
    created(){
        let url = `/static${this.model.fullname}`;
        window.open(url,"_blank");
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
    .el-main{
        text-align: center;
    }
</style>