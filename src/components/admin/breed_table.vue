<template>
    <div>
        <div class="admin-list-pass" v-if="!hideFilter && !releaseType ">
            <el-select @change="fetchData()" width="120" v-if="!hidePass" size="small" v-model="isPass" placeholder="所有数据">
                <el-option
                    v-for="(val, key) in options"
                    :key="val"
                    :label="key"
                    :value="val">
                </el-option>
            </el-select>
            <el-input class="pick-erpai" size="small" v-model="factoryName">
                <template slot="prepend">单位名:</template>
            </el-input>
            <el-input v-if="!hideEartagFilter" class="pick-erpai" size="small" v-model="eartag">
                <template slot="prepend">耳牌号:</template>
            </el-input>
            <el-date-picker
                size="small"
                v-model="gmtCreate"
                type="datetimerange"
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                format="yyyy-MM-dd"
                value-format="yyyy-MM-dd"
                align="right">
            </el-date-picker>
            <el-button @click="fetchData()" size="small" type="primary">查询</el-button>
            <el-button @click="export2xls()" size="small" type="primary" icon="el-icon-download">导出表格</el-button>
        </div>
        <el-table
            v-loading="load"
            ref="table"
            tooltip-effect="dark"
            class="admin-table"
            :data="tableData"
            >
            <el-table-column
                show-overflow-tooltip
                v-for="(th, i) in headers"
                :key="i"
                align='center'
                :prop="th.prop"
                :label="th.label"
                :width="100"
            >
            </el-table-column>
            <el-table-column
                class="action"
                fixed="right"
                label="操作"
                align='center'
                width="160">
                <template slot-scope="scope">
                    <div class="opr" v-if="!releaseType && !isCheck">
                        <span v-if="!hideView" @click="cellClick(scope.row, scope.column)">查看</span>
                        <template>
                            <span @click="edit(scope.$index)" v-if="showEdit">编辑</span>
                            <span @click="deleteItem(scope.$index)">删除</span>
                        </template>
                    </div>
                    <div class="opr" v-else-if="releaseType">
                        <span  @click="viewPlan(scope.$index)">查看</span>
                    </div>
                    <div class="opr" v-else>
                        <span @click="Spv(1, scope.$index)">通过</span>
                        <span @click="Spv(0, scope.$index)">拒绝</span>
                    </div>
                </template>
            </el-table-column>
        </el-table>

        <el-pagination
            layout="prev, pager, next"
            :total="total"
            @current-change="fetchData"
            :current-page.sync="page">
        </el-pagination>
    </div>
</template>

<script>
import { isReqSuccessful } from '@/util/jskit'
import { getUserById, getReleaseByName } from '@/util/getdata'
import XLSX from 'xlsx'
import {
// 监督执行
    patchWelfare,
    patchBreeding,
    patchPrevention,
    patchDisinfect,
    patchImmune,
    patchAntiscolic,
    patchStage,
// 专家审核
    patchProWelfare,
    patchProPrevention,
    patchProBreeding,
    patchProDisinfect,
    patchProImmune,
    patchProAntiscolic,
    patchProStage
} from '@/util/getdata'
import Bus from '@/components/bus.js'

export default {
    props: {
        // 隐藏操作栏的查看功能
        hideView: {
            type: Boolean,
            default: false
        },
        // 隐藏头部筛选
        hideFilter: {
            type: Boolean,
            default: false
        },
        // 跳转路径
        modpath: {
            type: String
        },
        getData: {
            type: Function
        },
        deleteData: {
            type: Function,
            default () {
                return () => {}
            }
        },
        // 审核接口
        isCheck: {
            type: Boolean,
            default: false
        },
        headers: {
            type: Array
        },

        // 是否显示操作栏的审核功能
        hidePass: {
            type: Boolean,
            default: false
        },
        // 跳转路径是否没有prac
        noPrac: {
            type: Boolean,
            default: false
        },
        // 代理表头需要转换
        isAgent: {
            type: Boolean,
            default: false
        },
        // 方案表
        releaseType: {
            type: String,
            default: ''
        },
        showEdit: {
            type: Boolean,
            default: true
        },
        hideEartagFilter: {
            type: Boolean,
            default: false
        },
        checkModule: {
            type: String
        }
    },

    watch: {
        getData (newV) {
            this.fetchData()
        }
    },

    mounted () {
        let id = this.$route.params.id
        getUserById(id).then(res => {
            if (isReqSuccessful(res)) {
                this.user = res.data.model
            }
        }).then(this.fetchData)
    },

    data () {
        return {
            load: true, // 是否显示loading动画
            page: 1, // 当前页码
            total: 10, // 总共数据条数
            tableData: [
                {
                    id: 1,
                    manage: '2018-01-01',
                    eNutritionT: '2018-05-10'
                }
            ], // 表格数据

            isPass: null, // 筛选条件-是否通过
            factoryName: null, // 筛选条件-工厂名称
            options: { // 表格审核状态列，显示转换映射
                所有数据: null,
                未通过: 0,
                已通过: 1,
                未审核: 2
            },

            eartag: null,
            gmtCreate: null
        }
    },

    methods: {
        export2xls () {
            if (!this.tableData.length) {
                this.$message.warning('表格数据为空')
            }

            let s2ab = s => {
                if (typeof ArrayBuffer !== 'undefined') {
                    var buf = new ArrayBuffer(s.length)
                    var view = new Uint8Array(buf)
                    for (var i = 0; i != s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF
                    return buf
                } else {
                    var buf = new Array(s.length)
                    for (var i = 0; i != s.length; ++i) buf[i] = s.charCodeAt(i) & 0xFF
                    return buf
                }
            }

            let eBody = this.tableData.slice(0)
            eBody.forEach(v => {
                Object.keys(v).forEach(vk => {
                    let inTable = false
                    // 有的表头含有 children ，要特殊处理
                    this.headers.forEach(vh => {
                        if (vh.children) {
                            inTable = vh.children.find(v => v.prop === vk)
                        }
                    })

                    if (!inTable) {
                        inTable = this.headers.find(v => v.prop === vk)
                    }
                    if (inTable && inTable.label) {
                        v[inTable.label] = v[vk]
                    }

                    // this.header 不含有一些公共表头如养殖场 factoryName 等，在这里手动加入
                    if (!['ispassCheck', 'factoryName', 'gmtCreate', 'remark', 'ispassSup', 'operatorName', 'professorName', 'upassReason', 'supervisorName'].includes(vk)) {
                        delete v[vk]
                    } else {
                        let map = {
                            ispassCheck: '审核状态',
                            factoryName: '养殖场',
                            gmtCreate: '提交时间',
                            remark: '备注',
                            ispassSup: '监督执行状态',
                            operatorName: '操作人员',
                            professorName: '技术审核',
                            upassReason: '审核拒绝原因',
                            supervisorName: '监督执行'
                        }
                        v[map[vk]] = v[vk]
                        delete v[vk]
                    }
                })
            })
            // console.log(eBody)
            const wb = { SheetNames: ['Sheet1'], Sheets: {}, Props: {} }
            const wopts = { bookType: 'biff8', bookSST: false, type: 'binary' }
            wb.Sheets['Sheet1'] = XLSX.utils.json_to_sheet(eBody)

            let tmpa = document.createElement("a");
            let obj = new Blob([s2ab(XLSX.write(wb, wopts))], { type: 'application/octet-stream' })
            tmpa.download = '下载.xls'
            tmpa.href = URL.createObjectURL(obj) //绑定a标签
            tmpa.click() //模拟点击实现下载
            setTimeout(function () { //延时释放
                URL.revokeObjectURL(obj) //用URL.revokeObjectURL()来释放这个object URL
            }, 100)
        },

        cellClick(row) {
            let id = row.id
            let pathid = this.$route.params.id
            let path = `/admin/${pathid}/${this.modpath}/more?more=${id}`
            this.$router.push(path)
        },
        Spv (isPass, idx) {
            let {id, ispassCheck} = this.tableData[idx]
            if (ispassCheck !== '未审核') {
                this.$message.warning('该条记录已审核')
                return
            }

            let superviseMap = {
                welfare: patchWelfare,
                prevention: patchPrevention,
                'nutrition/breed': patchBreeding,
                'nutrition/stage': patchStage,
                'health/antiscolic': patchAntiscolic,
                'health/disinfect': patchAntiscolic,
                'health/immune': patchAntiscolic
            }
            let professorMap = {
                welfare: patchProWelfare,
                prevention: patchProPrevention,
                'nutrition/breed': patchProBreeding,
                'nutrition/stage': patchProStage,
                'health/antiscolic': patchProAntiscolic,
                'health/disinfect': patchProAntiscolic,
                'health/immune': patchProAntiscolic
            }
            let data = {
                unpassReason: '',
                factoryNum: this.user.userFactory,
                professor: this.$route.params.id
            }
            // userRole 20羊场监督员
            if (this.user.userRole == 20) {
                data.ispassSup = isPass
                superviseMap[this.checkModule](id, data).then(res => {
                    if (isReqSuccessful(res)) {
                        this.$message.success('监督执行成功')
                        // this.tableData[idx].ispassCheck = isPass ? '已执行' : '未执行'
                    }
                }, _ => {
                    this.$message.error('监督执行失败')
                })
            } else {
                data.ispassCheck = isPass
                professorMap[this.checkModule](id, data).then(res => {
                    if (isReqSuccessful(res)) {
                        this.$message.success('审核成功')
                        this.tableData[idx].ispassCheck = isPass ? '已通过' : '未通过'
                    }
                }, _ => {
                    this.$message.error('审核失败')
                })
            }
        },

        viewPlan (index) {
            let id = this.tableData[index].id
            this.$router.push({name: this.modpath, query: {view: id}})
        },

        async fetchData () {
            let param = {
                page: this.page - 1,
                size: 10
            }
            if (this.isPass !== null) {
                param.ispassCheck = this.isPass
            }
            if (this.factoryName !== null) {
                param.factoryName = this.factoryName
            }
            if (this.gmtCreate !== null) {
                console.log(this.gmtCreate)
                param.startTime = this.gmtCreate[0]
                param.endTime = this.gmtCreate[1]
            }

            let pathid
            let { userFactory, userRealname, id, factoryName } = this.user
            // 代理 工厂 游客
            if (userFactory !== undefined) {
                pathid = userFactory
            } else if (id !== undefined) {
                pathid = id
            }

            this.load = true
            if (!this.releaseType) {
                this.getData(pathid, param).then(res => {
                    if (isReqSuccessful(res)) {
                        let data = res.data;

                        if (this.isAgent) {
                            data.List.forEach(v => {
                                let map = ['', '省级代理', '市级代理', '县级代理']
                                v.agentRank = map[v.agentRank]
                            })
                        }
                        let item = data.List[0]
                        if (item && item.ispassCheck !== null && item.ispassCheck !== undefined) {
                            data.List.forEach(v => {
                                let map = ['未通过', '已通过', '未审核']
                                v.ispassCheck = map[v.ispassCheck]
                            })
                        }
                        if (item && item.killWormDeratization !== undefined) {
                            data.List.forEach(v => {
                                let map = ['否', '是']
                                Object.keys(v).forEach(v2 => {
                                    if (v[v2] === 0 || v[v2] === 1) {
                                        v[v2] = map[v[v2]]
                                    }
                                })
                            })
                        }
                        if (item && item.materialA !== undefined) {
                            data.List.forEach(v => {
                                let map = [
                                    'materialA',
                                    'materialM',
                                    'materialO',
                                    'materialWM',
                                    'materialWO',
                                    'roughageP',
                                    'roughageD',
                                    'roughageO',
                                    'roughageWP',
                                    'roughageWD',
                                    'roughageWO',
                                    'pickingM',
                                    'pickingR',
                                    'pickingO'
                                ]
                                map.forEach(v2 => {
                                    if (v[v2] && v[v2].indexOf('[') !== -1) {
                                        v[v2] = JSON.parse(v[v2]).join(',')
                                    }
                                })
                            })
                        }
                        this.tableData = data.List
                        this.total = data.size
                    }
                    this.load = false
                }, _ => {
                    this.load = false
                    this.$message.error('获取数据失败')
                })
            } else {
                let res = await getReleaseByName(this.releaseType)
                this.tableData = res.data.List
                this.total = res.data.size
                this.load = false
            }
        },

        edit (index, isView) {
            let id = this.tableData[index].id
            let path
            let pathid = this.$route.params.id
            if (this.noPrac) {
                if (isView) {
                    path = `/admin/${pathid}/${this.modpath}?view=${id}`
                } else {
                    path = `/admin/${pathid}/${this.modpath}?edit=${id}`
                }
            } else if (!this.isCheck) {
                if (isView) {
                    path = `/admin/${pathid}/${this.modpath}/prac?view=${id}`
                } else {
                    path = `/admin/${pathid}/${this.modpath}/prac?edit=${id}`
                }
            } else {
                path = `/admin/${pathid}/${this.checkModule}/prac?check=${id}`
            }
            this.$router.push(path)
        },

        deleteItem (index) {
            this.$confirm('将永久删除此条记录, 是否继续?', '提示', {
                type: 'warning'
            }).then(() => {
                let id = this.tableData[index].id
                this.deleteData(id).then(res => {
                    if (isReqSuccessful(res)) {
                        this.fetchData()
                        this.$message.success('删除成功!')
                    }
                })
            }).catch(() => {
                return false
            })
        }
    }
}
</script>

<style lang="stylus">
@import '~@/assets/css/color'
.el-table th
    border-left 2px solid #98c9e6
    color #fff
    background-color color-main !important
</style>
