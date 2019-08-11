<template>
    <div v-if="easyFlowVisible">
        <el-row>
            <el-col :span="3" ref="flowTool">
              <flowTool @addNode="addNode"></flowTool>
            </el-col>
            <el-col :span="21">
                <el-row>
                    <el-col :span="24">
                        <div style="margin-bottom: 5px; margin-left: 10px">
                            <el-link type="primary">{{data.name}}</el-link>
                            <el-button type="info" icon="el-icon-document"  @click="dataInfo">流程信息</el-button>
                            <el-button type="primary" @click="dataReloadA" icon="el-icon-refresh">切换流程A</el-button>
                            <el-button type="success" @click="dataReloadB" icon="el-icon-refresh">切换流程B</el-button>
                            <el-button type="warning" @click="dataReloadC" icon="el-icon-refresh">切换流程C</el-button>
                            <!--<el-button type="warning" @click="changeGridline" icon="el-icon-refresh">待定</el-button>-->
                        </div>
                    </el-col>
                </el-row>
                <el-row>
                    <el-col :span="24">
                        <!--画布-->
                        <div id="flowContainer"  class="container">
                            <template v-for="node in data.nodeList">
                                <flow-node
                                        v-show="node.show"
                                        :id="node.id"
                                        :node="node"
                                        @deleteNode="deleteNode"
                                        @changeNodeSite="changeNodeSite"
                                        @nodeRightMenu="nodeRightMenu"
                                        @editNode="editNode"
                                >
                                </flow-node>
                            </template>
                        </div>
                    </el-col>
                </el-row>
            </el-col>
        </el-row>
        <!--流程信息页面的展示-->
        <flow-info v-if="flowInfoVisible" ref="flowInfo" :data="data"></flow-info>
        <!--节点信息页面的展示-->
        <flow-node-form v-if="nodeFormVisible" ref="nodeForm"></flow-node-form>
        <!--连线信息页面的展示-->
        <flow-line-form v-if="nodeLineVisible" ref="nodeLine"></flow-line-form>
    </div>
</template>

<script>
    import draggable from 'vuedraggable'
    import {jsPlumb} from 'jsplumb'
    import flowNode from '@/components/flow/node'
    import flowTool from '@/components/flow/tool'
    import FlowInfo from '@/components/flow/info'
    import FlowNodeForm from './node_form'
    import FlowLineForm from './line_form'
    import lodash from 'lodash'
    import {getDataA} from './data_A'
    import {getDataB} from './data_B'
    import {getDataC} from './data_C'

    export default {
        name: "easyFlow",
        data() {
            return {
                jsPlumb: null,// jsPlumb 实例
                easyFlowVisible: true,
                flowInfoVisible: false,
                nodeFormVisible: false,
                nodeLineVisible: false,//一开始不显示线的信息
                index: 1,
                // 默认设置参数
                jsplumbSetting: {
                    // 动态锚点、位置自适应
                    Anchors: ['Top', 'TopCenter',  'Right', 'RightTop', 'Bottom', 'BottomCenter', 'BottomRight', 'BottomLeft', 'Left', 'LeftMiddle'],
                    Container: 'flowContainer',
                    // 连线的样式 StateMachine、Flowchart
                    Connector: 'Flowchart',
                    // 鼠标不能拖动删除线
                    ConnectionsDetachable: true,
                    // 删除线的时候节点不删除
                    DeleteEndpointsOnDetach: false,
                    // 连线的端点
                     Endpoint: ["Dot", {radius: 10}],
//                    Endpoint: ["Rectangle", {height: 12, width: 20}],
                    // 线端点的样式
                    EndpointStyle: {fill: 'rgba(255,255,255,0)', outlineWidth: 1},
                    LogEnabled: true,//是否打开jsPlumb的内部日志记录
                    // 绘制线(可以调整线的粗细)
                    PaintStyle: {stroke: 'black', strokeWidth: 3},
                    // 鼠标移至线时的样式
                    HoverPaintStyle:{stroke:'rgba(0,0,0,0.3)', strokeWidth: 15},
                    // 绘制箭头（可以调整箭头的大小）
                    Overlays: [['Arrow', {width: 12, length: 12, location: 1}]],
                    RenderMode: "svg"
                },
                // jsplumb连接参数
                jsplumbConnectOptions: {
                    isSource: true,
                    isTarget: true,
                    // 动态锚点、提供了4个方向 Continuous、AutoDefault
                    anchor: "Continuous"
                },
                jsplumbSourceOptions: {
                    filter: ".flow-node-drag",/*"span"表示标签，".className"表示类，"#id"表示元素id*/
                    filterExclude: false,
                    anchor: "Continuous",
                    allowLoopback: false
                },
                jsplumbTargetOptions: {
                    filter: ".flow-node-drag",/*"span"表示标签，".className"表示类，"#id"表示元素id*/
                    filterExclude: false,
                    anchor: "Continuous",
                    allowLoopback: false
                },
                // 是否加载完毕
                loadEasyFlowFinish: false,
                // 数据
                data:  {
                }
            }
        },
        components: {
            draggable, flowNode, flowTool, FlowInfo,FlowNodeForm,FlowLineForm
        },
        mounted() {
            this.jsPlumb = jsPlumb.getInstance()
            this.$nextTick(()=>{
                this.dataReloadA()
            })
        },
        methods: {
            jsPlumbInit() {
                const _this = this
                this.jsPlumb.ready(function () {
                    // 导入默认配置
                    _this.jsPlumb.importDefaults(_this.jsplumbSetting)
                    // 会使整个jsPlumb立即重绘。
                    _this.jsPlumb.setSuspendDrawing(false,true);
                    // 初始化节点
                    _this.loadEasyFlow()

                     // 鼠标右键事件：右击了连接线
                    _this.jsPlumb.bind('contextmenu', function (conn, originalEvent) {
                        console.log("contextmenu", conn)
                        _this.$confirm('确定删除所点击的线吗?', '提示', {
                            confirmButtonText: '确定',
                            cancelButtonText: '取消',
                            type: 'warning'
                        }).then(() => {
                            _this.jsPlumb.deleteConnection(conn)
                        }).catch(() => {
                        })
                        event.preventDefault()
                    })

                    // 单击了连接线,进入编辑连线信息页面
                    _this.jsPlumb.bind('click', function (evt) {
                      console.log("click", evt)
                      _this.editLine(evt.sourceId,evt.targetId)
                    })

                    // 连线this.data.lineList（info.vue可以获取改变的数据）和this.lineList（info.vue不可获取）有区别
                    _this.jsPlumb.bind("connection", function (evt) {
                        console.log('connection', evt)
                        let from = evt.source.id
                        let to = evt.target.id
                        if (_this.loadEasyFlowFinish) {
                            _this.data.lineList.push({
                                from: from,
                                to: to
                            })
                        }
                    })

                    // 删除连线
                    _this.jsPlumb.bind("connectionDetached", function (evt) {
                        console.log('connectionDetached', evt)
                        _this.deleteLine(evt.sourceId, evt.targetId)
                    })

                    // 改变线的连接节点
                    _this.jsPlumb.bind("connectionMoved", function (evt) {
                        console.log('connectionMoved', evt)
                        _this.changeLine(evt.originalSourceId, evt.originalTargetId)
                    })

                    // 连接建立前事件beforeDrop
                    _this.jsPlumb.bind("beforeDrop", function (evt) {
                        console.log('beforeDrop', evt)
                        let from = evt.sourceId
                        let to = evt.targetId
                        if (from === to) {
                            _this.$message.error('不能连接自己');
                            return false
                        }
                        if (_this.hasLine(from, to)) {
                            _this.$message.error('不能重复连线');
                            return false
                        }
                        if (_this.hashOppositeLine(from, to)) {
                            _this.$message.error('不能回环哦');
                            return false
                        }
                        return true
                    })

                    // 连接断开前事件beforeDetach
                    _this.jsPlumb.bind("beforeDetach", function (evt) {
                        console.log('beforeDetach', evt)
                    })
                })
            },
            // 加载流程图
            loadEasyFlow() {

                // 初始化节点
                for (var i = 0; i < this.data.nodeList.length; i++) {
                    let node = this.data.nodeList[i]
                    // 设置源点，可以拖出线连接其他节点
                    this.jsPlumb.makeSource(node.id, this.jsplumbSourceOptions)
                    // 设置目标点，其他源点拖出的线可以连接该节点
                    this.jsPlumb.makeTarget(node.id, this.jsplumbTargetOptions)
                    // 设置可拖拽
                    this.jsPlumb.draggable(node.id, {
                        containment: 'parent'
                    })
                }

                // 初始化连线
                for (var i = 0; i < this.data.lineList.length; i++) {
                    let line = this.data.lineList[i]
                    this.jsPlumb.connect({
                        source: line.from,
                        target: line.to,
                    }, this.jsplumbConnectOptions)
                }
                this.$nextTick(function () {
                    this.loadEasyFlowFinish = true
                })
            },
            getNodes() {
                console.log(jsPlumb)
                console.log(jsPlumb.Defaults)
            },
            getLines() {
                console.log('线', jsPlumb.getConnections())
            },
            // 删除线
            deleteLine(from, to) {
                this.data.lineList = this.data.lineList.filter(function (line) {
                    //删除首尾是from和to的线（从lineList中删除）
                    return !(line.from == from && line.to == to)
                })
            },
            // 改变连线
            changeLine(oldFrom, oldTo) {
                this.deleteLine(oldFrom, oldTo)
            },
            // 改变节点的位置
            changeNodeSite(data) {
                for (var i = 0; i < this.data.nodeList.length; i++) {
                    let node = this.data.nodeList[i]
                    if (node.id === data.nodeId) {
                        node.left = data.left
                        node.top = data.top
                    }
                }
            },
            // 添加新的节点
            addNode(evt, nodeMenu) {
                console.log('添加节点', evt, nodeMenu)
                let width = this.$refs.flowTool.$el.clientWidth
                const index = this.index++
                let nodeId =  'node' +index
                this.data.nodeList.push({
                    id:  'node' +index,
                    name: '节点' + index,
                    type: nodeMenu.type,
                    left: evt.originalEvent.layerX - width + 'px',
                    top: evt.originalEvent.clientY - 50 + 'px',
                    ico: nodeMenu.ico,
                    show: true
                })
                this.$nextTick(function () {

                    this.jsPlumb.makeSource(nodeId, this.jsplumbSourceOptions)

                    this.jsPlumb.makeTarget(nodeId, this.jsplumbTargetOptions)

                    this.jsPlumb.draggable(nodeId, {
                        containment: 'parent'
                    })

                })
            },
            // 是否具有该线
            hasLine(from, to) {
                for (var i = 0; i < this.data.lineList.length; i++) {
                    var line = this.data.lineList[i]
                    if (line.from === from && line.to === to) {
                        return true
                    }
                }
                return false
            },
            // 是否含有相反的线
            hashOppositeLine(from, to) {
                return this.hasLine(to, from)
            },
            nodeRightMenu(nodeId, evt) {
                this.menu.show = true
                this.menu.curNodeId = nodeId
                this.menu.left = evt.x + 'px'
                this.menu.top = evt.y + 'px'
            },
            //删除节点
            deleteNode(nodeId) {
                //把node+index显示为只有index
                var showId=nodeId.substring(4)
                this.$confirm('确定要删除节点' + showId + '?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                    closeOnClickModal: false
                }).then(() => {

                    this.data.nodeList =  this.data.nodeList.filter(function (node) {
                        if (node.id === nodeId){
                            node.show = false
                        }
                        return true
                    })

                    this.$nextTick(function () {
                        console.log('删除'+nodeId)
                        this.jsPlumb.removeAllEndpoints(nodeId);

                    })
                    //将节点完全从this.data.nodeList中删除
//                    for (var i = 0; i < this.data.nodeList.length; i++) {
//                      if (this.data.nodeList[i].id === nodeId) {
//                        this.data.nodeList.splice(i,1)
//                      }
//                    }
                }).catch(() => {
                })
                return true
            },
            //编辑节点
            editNode(nodeId) {
                console.log('编辑节点', nodeId)
                this.nodeFormVisible = true
            //获取一个节点的上节点和下节点的字符串
            var lineFrom=""
            var lineTo=""
            for (var i = 0; i < this.data.lineList.length; i++) {
              var line = this.data.lineList[i]
              if (line.to === nodeId) {
                if(lineFrom!=""){
                  lineFrom=lineFrom+','+line.from
                }else{
                  lineFrom=line.from
                }
              }
              if (line.from === nodeId) {
                if(lineTo!=""){
                  lineTo=lineTo+','+line.to
                }else{
                  lineTo=line.to
                }
              }
            }
                this.$nextTick(function () {
                    this.$refs.nodeForm.init(this.data,nodeId,lineFrom,lineTo)
                })
            },
            //编辑连线
            editLine(from,to){
              this.nodeLineVisible = true
              this.$nextTick(function () {
                this.$refs.nodeLine.init(this.data,from,to)
              })
            },
            // 流程数据信息
            dataInfo() {
                this.flowInfoVisible = true
                this.$nextTick(function () {
                    this.$refs.flowInfo.init()
                })
            },
            //加载数据
            dataReload(data){
                this.easyFlowVisible = false
                this.data.nodeList = []
                this.data.lineList = []
                this.$nextTick(() => {
                    // 这里模拟后台获取数据、然后加载
                    data = lodash.cloneDeep(data)
                    this.easyFlowVisible = true
                    this.data = data
                    this.$nextTick(() => {
                        this.jsPlumb = jsPlumb.getInstance()
                        this.$nextTick(() => {
                            this.jsPlumbInit()
                        })
                    })
                })
            },
            // 数据重新载入
            dataReloadA() {
                this.dataReload(getDataA())
            },
            dataReloadB() {
                this.dataReload(getDataB())
            },
            dataReloadC() {
                this.dataReload(getDataC())
            },
//          changeGridline(){
//            this.loadEasyFlow()
//          }
        }
    }
</script>

<style>
    #flowContainer {
        height:100vh;
        background-image: linear-gradient(90deg, rgba(0, 0, 0, 0.15) 10%, rgba(0, 0, 0, 0) 10%), linear-gradient(rgba(0, 0, 0, 0.15) 10%, rgba(0, 0, 0, 0) 10%);
        background-size: 10px 10px;
        background-color: rgb(251, 251, 251);
        /*background-color: #42b983;*/
        position: relative;
    }

    .labelClass {
        background-color: white;
        padding: 5px;
        opacity: 0.7;
        border: 1px solid #346789;
        /*border-radius: 10px;*/
        cursor: pointer;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }
</style>
