<template>
    <el-dialog
        :visible.sync="dialogVisible"
        top="20vh"
        width="50%"
        :modal="true"
        class="user-panel-dialog"
        :close-on-click-modal="false"
        :before-close="closeDialog">

        <div slot="title" class="user-pannel-title-box">
            <span class="user-pannel-title">{{ title }}</span>
        </div>

        <div class="component-card">
            <div class="component-body">
                <div class="data-area">
                    <div class="component-body-left">
                        <div class="transfer-top-box">
                            <el-checkbox v-model="sourceChecked" @change="sourceSelectAll()">{{ sourceTitle }}</el-checkbox>
                        </div>
                        <div class="transfer-search">
                            <el-input placeholder="请搜索内容" size="mini" prefix-icon="el-icon-search" @change="searchHosts" v-model="search_word" clearable></el-input>
                        </div>
                        <div class="transfer-show-items">
                            <div class="transfer-list-items">
                                <el-checkbox v-show="source.show" v-for="(source, index) in sourceOrigin" @change="sourceCheckboxClick(source)" :label="source" :key="index" v-model="source.checked">{{source.label}}</el-checkbox>
                            </div>
                        </div>
                    </div>
                    <div class="data-transfer">
                        <el-button type="success" size="mini" @click="selected">选定</el-button>
                        <div class="diviser"></div>
                        <el-button type="danger" size="mini" @click="undo">撤销</el-button>
                    </div>
                    <div class="component-body-right">
                        <div class="transfer-top-box">
                            <el-checkbox v-model="targetChecked" @change="targetSelectAll()">{{ targetTitle }}</el-checkbox>
                        </div>
                        <div class="transfer-show-items">
                            <div class="transfer-list-items">
                                <el-checkbox v-for="(target, index) in targetOrigin" @change="targetCheckboxClick(target)" :label="target" :key="index" v-model="target.checked">{{target.label}}</el-checkbox>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="component-footer">
                <div class="button-box">
                    <el-button size="mini" @click="cancel">取消</el-button>
                    <el-button type="primary" size="mini" @click="save">保存</el-button>
                </div>
            </div>
        </div>

    </el-dialog>
</template>

<script>
    /* 以下是组件API异步加载函数 */
    import { searchHostsAPI } from '@/server/service'

    export default {
        name: 'hostsPanel',
        beforecreate: function() {},
        created: function() {
            this.searchHosts()
        },
        computed: {
            sourceTitle: function() {
                return ['全选 (', this.sourceCheckedCounts, '/', this.sourceAllCounts, ')'].join('')
            },
            targetTitle: function() {
                return ['全选 (', this.targetCheckedCounts, '/', this.targetAllCounts, ')'].join('')
            }
        },
        data: function() {
            return {
                sourceChecked: false,
                sourceAllCounts: 0,
                sourceCheckedCounts: 0,
                targetChecked: false,
                targetAllCounts: 0,
                targetCheckedCounts: 0,
                dialogVisible: true,
                search_word: '',
                checkedCities: [],
                sourceOrigin: [],
                targetOrigin: [],
                firstTime: true
            }
        },
        methods: {
            sourceSelectAll() {
                let sourceOrigin
                sourceOrigin = this.sourceOrigin.map((r) => {
                    return {
                        ...r,
                        checked: this.sourceChecked
                    }
                })
                if (this.sourceChecked) {
                    this.sourceCheckedCounts = this.sourceAllCounts
                } else {
                    this.sourceCheckedCounts = 0
                }
                this.sourceOrigin = sourceOrigin
            },
            targetSelectAll() {
                let targetOrigin
                targetOrigin = this.targetOrigin.map((r) => {
                    return {
                        ...r,
                        checked: this.targetChecked
                    }
                })
                if (this.targetChecked) {
                    this.targetCheckedCounts = this.targetAllCounts
                } else {
                    this.targetCheckedCounts = 0
                }
                this.targetOrigin = targetOrigin
            },
            sourceCheckboxClick(item) {
                if (item.checked) {
                    this.sourceCheckedCounts += 1
                } else {
                    this.sourceCheckedCounts -= 1
                }
            },
            targetCheckboxClick(item) {
                if (item.checked) {
                    this.targetCheckedCounts += 1
                } else {
                    this.targetCheckedCounts -= 1
                }
            },
            closeDialog: function() {
                this.$emit('closeDialog')
            },
            transferSourceList: function(source) {
                const sourceOrigin = source.map((r) => {
                    return {
                        key: r.key,
                        label: r.label,
                        checked: false,
                        show: true
                    }
                })
                this.sourceOrigin = sourceOrigin
                this.sourceAllCounts = sourceOrigin.length
                this.sourceCheckedCounts = 0
            },
            transferTargetList: function(target) {
                const targetOrigin = target.map((r) => {
                    return {
                        key: r.key,
                        label: r.label,
                        checked: false
                    }
                })
                this.targetOrigin = targetOrigin
                this.targetAllCounts = targetOrigin.length
                this.targetCheckedCounts = 0
            },
            getSourceSelectItems: function() {
                return this.sourceOrigin.filter((r) => {
                    return r.checked && r.show
                }).map((r) => {
                    return {
                        ...r,
                        checked: false
                    }
                })
            },
            getTargetSelectItems: function() {
                return this.targetOrigin.filter((r) => {
                    return r.checked
                })
            },
            showSourceSelectItems: function(source) {
                /* 显示选定项目 */
                const key = []
                source.forEach((_c, _i, _a) => {
                    key.push(_c.key)
                })
                const sourceOrigin = this.sourceOrigin.map((r) => {
                    let show
                    let checked
                    if (key.indexOf(r.key) !== -1) {
                        show = true
                        checked = false
                    } else {
                        show = r.show
                        checked = r.checked
                    }
                    return {
                        ...r,
                        show,
                        checked
                    }
                })
                this.sourceOrigin = sourceOrigin
            },
            hideSourceSelectItems: function(source) {
                /* 隐藏选定项目 */
                const key = []
                source.forEach((_c, _i, _a) => {
                    key.push(_c.key)
                })
                const sourceOrigin = this.sourceOrigin.map((r) => {
                    let show
                    if (key.indexOf(r.key) !== -1) {
                        show = false
                    } else {
                        show = r.show
                    }
                    return {
                        ...r,
                        show
                    }
                })
                this.sourceOrigin = sourceOrigin
            },
            addItemsToTargetList(target) {
                /* 添加项目到Target */
                const targetItems = target.map((r) => {
                    return {
                        ...r,
                        checked: false
                    }
                })

                this.targetOrigin.push(...targetItems)
                this.targetAllCounts += targetItems.length
            },
            delItemsToTargetList(target) {
                const keys = []
                target.forEach((_c, _i, _a) => {
                    keys.push(_c.key)
                })
                let deleteTargetCounts = 0
                const targetOrigin = this.targetOrigin.filter((r) => {
                    if (keys.indexOf(r.key) !== -1) {
                        deleteTargetCounts += 1
                        return false
                    } else {
                        return true
                    }
                })

                this.targetOrigin = targetOrigin
                this.targetAllCounts -= deleteTargetCounts
                this.targetCheckedCounts -= deleteTargetCounts
            },
            selected: function() {
                /* 选定操作 */
                // this.originTarget
                const selectSourceOrigin = this.getSourceSelectItems()
                /* 隐藏选定项 */
                this.hideSourceSelectItems(selectSourceOrigin)
                /* 同步到Target容器 */
                this.addItemsToTargetList(selectSourceOrigin)
            },
            undo: function() {
                /* 撤消操作 */
                const targetOrigin = this.getTargetSelectItems()
                /* 渲染 */
                this.delItemsToTargetList(targetOrigin)
                /* 恢复源数据 */
                this.showSourceSelectItems(targetOrigin)
            },
            getSourceItemsFromTable: function() {
                const keys = this.originTarget.split(',').map(r => {
                    return r.trim()
                }).filter((r) => {
                    return !!r.length
                })
                return this.sourceOrigin.filter((r) => {
                    const index_ip = r.key.split(',')[0]
                    if (keys.indexOf(index_ip) !== -1) {
                        return true
                    } else {
                        return false
                    }
                })
            },
            initTransferData: function() {
                /* 初始化IP池 */
                const selectSourceOrigin = this.getSourceItemsFromTable()
                /* 隐藏选定项 */
                this.hideSourceSelectItems(selectSourceOrigin)
                /* 同步到Target容器 */
                this.addItemsToTargetList(selectSourceOrigin)
            },
            cancel: function() {
                /* 取消 */
                this.closeDialog()
            },
            save: function() {
                /* 保存 */
                this.$emit('save', this.symbol, this.targetOrigin)
            },
            cutIPString: function(string) {
                return string.split('_').slice(0, -1).join('_')
            },
            searchHosts() {
                const search = this.search_word
                searchHostsAPI(search).then(res => {
                    const results = res.data.results
                    const resource = results.map((r) => {
                        const item_name = this.cutIPString(r.name)
                        const key = r.private_ip
                        let first_private_ip = ''
                        if (key.length) {
                            first_private_ip = key.split(',')[0]
                        }
                        const label = [first_private_ip, '(', item_name, ')'].join('')
                        return {
                            key: key,
                            label: label
                        }
                    })
                    this.transferSourceList(resource)
                    if (this.firstTime) {
                        this.initTransferData()
                        this.firstTime = false
                    }
                })
            }
        },
        mounted: function() {},
        watch: {},
        props: {
            sourceUrl: {
                type: String,
                default: ''
            },
            originTarget: {
                type: String,
                default: ''
            },
            symbol: {
                type: String,
                required: true
            },
            title: {
                type: String,
                required: true
            }
        }
    }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
    $border_schtch: #dddddd;
    $subject_title: #514963;
    $padding_to_left: 8px;
    .user-panel-body{
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .user-pannel-title-box{
        background-color: #47415a;
        color: white;
        .user-pannel-title{
            height: 36px;
            padding-left: 15px;
            line-height: 36px;
            vertical-align: center;
        }
    }
    .component-card {
        width: 100%;
        height: 100%;
        .component-header{
            height: 80px;
        }
        .component-body{
            width: 100%;
            height: 100%;
            .data-area{
                width: 100%;
                height: 100%;
                .component-body-left{
                    height: 208px;
                    border: 1px solid $border_schtch;
                    .transfer-top-box{
                        height: 28px;
                        width: 100%;
                        padding-left: $padding_to_left;
                        display: flex;
                        align-items: center;
                        background-color: $subject_title;
                        border-bottom: 1px solid $border_schtch;
                    }
                    .transfer-search{
                        padding: 0;
                    }
                    .transfer-show-items{
                        /*height: calc(100% - 120px);*/
                        height: 150px;
                        overflow-y: scroll;
                        padding: $padding_to_left;
                        .transfer-list-items{
                            display: flex;
                            flex-direction: column;
                        }
                    }
                }
                .data-transfer{
                    display:flex;
                    padding: 5px 0;
                    justify-content:center;
                    .diviser{
                        width: 20px;
                        height: 100%;
                    }
                }
                .component-body-right{
                    height: 200px;
                    border: 1px solid $border_schtch;
                    .transfer-top-box{
                        height: 28px;
                        width: 100%;
                        padding-left: $padding_to_left;
                        display: flex;
                        align-items: center;
                        background-color: $subject_title;
                        border-bottom: 1px solid $border_schtch;
                    }
                    .transfer-show-items{
                        /*height: calc(100% - 120px);*/
                        height: 150px;
                        overflow-y: scroll;
                        padding: $padding_to_left;
                        .transfer-list-items{
                            display: flex;
                            flex-direction: column;
                        }
                    }
                }
            }
        }
        .component-footer{
            width: 100%;
            height: 40px;
            display: flex;
            padding: 5px;
            flex-direction: row-reverse;
        }
    }
</style>

<docs>
    <hosts-panel
            v-if="hostsPanelShow"
            :title="transferTitle"
            :origin-target="originTarget"
            :symbol="hostSymbol"
            @save="saveHostsPanel"
            @cancel="cancelHostsPanel"
            @closeDialog="closeHostsPanel"
    ></hosts-panel>
    hostsPanelShow: 是否显示弹框
    title: 标题
    originTarget： 原始目标key数组
    symbol: 组件标志，在save函数emit时会传入参数
    cancel: 取消
    closeDialog: 关闭按钮,修改hostsPanelShow值
    ----------------------------------------------------
    注：arrayDeduplication可以传入一个函数任务获取源数据
</docs>
