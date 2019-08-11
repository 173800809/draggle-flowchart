<template>
  <el-dialog
    title="连线信息"
    :close-on-click-modal="false"
    :visible.sync="visible"
    >
    <el-form :model="line" ref="dataLine" label-width="80px">
      <el-form-item label="名称">
        <el-input ></el-input>
      </el-form-item>
      <el-form-item label="from">
        <el-input v-model="line.from"></el-input>
      </el-form-item>
      <el-form-item label="to">
        <el-input v-model="line.to"></el-input>
      </el-form-item>
    </el-form>
        <span slot="footer" class="dialog-footer">
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
      line: {}
    }
  },
  methods: {
    init(data, from, to) {
        this.visible = true

      this.line.from = "null"
      this.line.from = from
      this.line.to = to
    }
  }
  }
</script>

<style scoped>

</style>
