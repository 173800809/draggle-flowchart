<template>
    <el-dialog
            title="流程数据信息"
            :visible.sync="dialogVisible"
            width="70%"
    >
        <codemirror
                :value="flowData"
                :options="options"
                class="code"
        ></codemirror>

        <span slot="footer" class="dialog-footer">
  </span>
    </el-dialog>
</template>

<script>
    import {codemirror} from 'vue-codemirror'

    require("codemirror/mode/javascript/javascript.js")

    export default {
        props: {
            data: Object,
        },
        data() {
            return {
                dialogVisible: false,
                options: {
                    mode: {name: "javascript", json: true}, //设置显示代码的类型
                    lineNumbers: true //每行代码前面都显示行号
                }
            }
        },
        components: {
            codemirror
        },
        computed: {
            flowData() {
                return JSON.stringify(this.data, null, 4).toString()
            }
        },
        methods: {
            init() {
                this.dialogVisible = true
            }
        }
    }
</script>

<style scoped>

</style>
