<template>
  <div
    ref="node"
    :style="flowNodeContainer"
    @mouseenter="showDelete"
    @mouseleave="hideDelete"
    @mouseup="changeNodeSite"
    >
    <div :class="typeTop()">
       <!--<img src="@/assets/node-img.png" class="flow-node-drag">-->
      <i :class="nodeClass" style="position:absolute;right: 58px;top: 8px;"></i>

      <div style="position: absolute;top: 0px;right: 0px;line-height: 25px" v-show="mouseEnter">
        <a href="#" style="" @click="editNode"><img src="@/assets/edit.png"></a>&nbsp;
        <a href="#" style="" @click="deleteNode"><img src="@/assets/delete.png"></a> &nbsp;
      </div>
    </div>
    <div :class="typeBottom()">
    </div>

    <div class="flow-node-body">
      {{node.name}}
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      node: Object
    },
    data()
  {
    return {
      // 控制节点操作显示
      mouseEnter: false,

    }
  }
  ,
  computed: {
    // 节点容器样式
    flowNodeContainer: {
      get()
      {
        return {
          //可以设置jsplumb中锚点图形的区域（即图像锚点的位置）
          position: 'absolute',
          width: '150px',
          height: '60px',
          top: this.node.top,
          left: this.node.left,
          boxShadow: this.mouseEnter ? '#66a6e0 0px 0px 12px 0px' : '',
          backgroundColor: 'transparent',

        }
      }
    }
  ,
    nodeClass()
    {
      var nodeclass = {}
      nodeclass[this.node.ico] = true
      nodeclass['flow-node-drag'] = true
      return nodeclass
    }

  }
  ,
  methods: {
    // 删除节点
    deleteNode()
    {
      this.$emit('deleteNode', this.node.id)
    }
  ,
    // 编辑节点
    editNode()
    {
      this.$emit('editNode', this.node.id)
    }
  ,
    // 鼠标进入
    showDelete()
    {
      this.mouseEnter = true
    }
  ,
    // 鼠标离开
    hideDelete()
    {
      this.mouseEnter = false
    }
  ,
    // 鼠标移动后抬起
    changeNodeSite()
    {
      this.$emit('changeNodeSite', {
        nodeId: this.node.id,
        left: this.$refs.node.style.left,
        top: this.$refs.node.style.top,
      })
    }
  ,
    //确定拖拉的上图形
    typeTop()
    {
      if (this.node.type === "start")return "startTop";
      if (this.node.type === "end")return "endTop";
      if (this.node.type === "judge")return "judgeTop";
      if (this.node.type === "handle")return "handleTop";
    }
  ,
    //确定拖拉的下图形
    typeBottom()
    {
      if (this.node.type === "start")return "startBottom";
      if (this.node.type === "end")return "endBottom";
      if (this.node.type === "judge")return "judgeBottom";
      if (this.node.type === "handle")return "handleBottom";
    }
  }
  }

</script>

<style>

  .flow-node-drag {
    width: 25px;
    height: 25px;
  }

  .flow-node-header a {
    text-decoration: none;
    line-height: 25px;
    vertical-align: middle;
  }

  .flow-node-header a img {
    line-height: 25px;
    vertical-align: middle;
    margin-bottom: 5px;
  }

  .flow-node-body {
    /*background-color: beige;*/
    background-color: transparent;
    text-align: center;
    cursor: pointer;
    height: 25px;
    line-height: 25px;
    width: 150px;
    border-bottom-left-radius: 6px;
    border-bottom-right-radius: 6px;
  }

  .startTop {
    background-color: #66a6e0;
    height: 30px;
    cursor: pointer;
    border-top-left-radius: 15px;
    border-top-right-radius: 15px;

  }

  .endTop {
    background-color: #66a6e0;
    height: 30px;
    cursor: pointer;
    border-top-left-radius: 15px;
    border-top-right-radius: 15px;

  }

  .judgeTop {
    width: 0;
    height: 0;
    border-bottom: 30px solid #66a6e0;
    border-right: 75px solid transparent;
    border-left: 75px solid transparent;

  }

  .handleTop {
    background-color: #66a6e0;
    height: 30px;
    cursor: pointer;
    border-top-left-radius: 0;
    border-top-right-radius: 0;

  }

  .judgeBottom {
    width: 0;
    height: 0;
    border-top: 30px solid #66a6e0;
    border-right: 75px solid transparent;
    border-left: 75px solid transparent;
  }

  .startBottom {
    background-color: #66a6e0;
    height: 30px;
    cursor: pointer;
    border-bottom-left-radius: 15px;
    border-bottom-right-radius: 15px;

  }

  .handleBottom {
    background-color: #66a6e0;
    height: 30px;
    cursor: pointer;
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;

  }

  .endBottom {
    background-color: #66a6e0;
    height: 30px;
    cursor: pointer;
    border-bottom-left-radius: 15px;
    border-bottom-right-radius: 15px;

  }
</style>
