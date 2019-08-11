<template>
    <el-dialog
            title="节点信息"
            :close-on-click-modal="false"
            :visible.sync="visible"
    >
        <el-form :model="node" ref="dataForm" label-width="80px">
            <el-form-item label="名称">
                <el-input v-model="node.name"></el-input>
            </el-form-item>
            <el-form-item label="left">
                <el-input v-model="node.left"></el-input>
            </el-form-item>
            <el-form-item label="top">
                <el-input v-model="node.top"></el-input>
            </el-form-item>
            <el-form-item label="ico">
                <el-input v-model="node.ico"></el-input>
            </el-form-item>
            <el-form-item label="from">
              <el-input v-model="node.from"></el-input>
            </el-form-item>
            <el-form-item label="to">
              <el-input v-model="node.to"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <!--@click="visible = false"就是点击之后页面关闭-->
            <el-button @click="visible = false" icon="el-icon-close">取消</el-button>
            <el-button @click="visible = false" type="primary" icon="el-icon-check">确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
    export default {
        data() {
            return {
                visible: false,
                node: {}
            }
        },
        methods: {
            init(data, id,from,to) {
                this.visible = true
                data.nodeList.filter((node) => {
                    if (node.id === id) {
                        this.node = node
                    }
                })

            this.node.from="null"
                //获取节点的上节点和下节点的字符串并同步到视图中
                this.node.from=from
                this.node.to=to
                if(this.node.from==="")this.node.from="null"
                if(this.node.to==="")this.node.to="null"
            }
        }
    }
</script>

<style scoped>

</style>
