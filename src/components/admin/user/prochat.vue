<template>
    <div class="pro_wrapper">
        <el-container>
            <el-container>
                <el-main>
                    <div class="pro-dialog_box" ref="dialog">
                        <div class="pro-dialog-item" :class="{self: item.self}" v-for="(item, i) in items" :key="i"><span v-text="item.self ? expert.name : user.name"></span><span class="msg" v-html="item.html"></span></div>
                    </div>
                    <div class="my_input_box">
                        <div class="my_chat_option">
                            <i class="iconfont icon-smile emoji" ref='btn' @click='showEmoji = !showEmoji'></i>
                            <input hidden ref="file" type="file" @change="sendFile()" class="file">
                            <i class="iconfont icon-3801wenjian" @click="$refs.file.click()"></i>
                            <i title="邀请专家" class="iconfont icon-icon_users my_inviation" @click="invite()"></i>
                            <el-select
                                v-model="currentExpression"
                                filterable
                                allow-create
                                default-first-option
                                clearable
                                placeholder="常用语"
                                @change="addExpression"
                                >
                                <el-option
                                v-for="(item, index) in expressionList"
                                :key="index"
                                :label="item.label"
                                :value="item.label"
                                >
                                <span>{{ item.label }}</span>
                                <i class="el-icon-delete" style="font-size:18px" @click.stop="deleteExpression(index)"></i>
                                </el-option>
                            </el-select>
                            <el-button size="small" type="primary">聊天记录</el-button>
                        </div>
                        <vue-emoji
                            v-show='showEmoji'
                            ref='emoji'
                            @select='handleEmojiSelect'
                            @hide="showEmoji = false"
                        ></vue-emoji>
                        <div class="my_chat_input">
                            <div contenteditable ref='edit' name="" class="chat_area"></div>
                        </div>
                        <el-button type="primary" size="small" @click="send()" class="send">发送</el-button>
                    </div>
                </el-main>

                <el-aside class="aside" width="200px">
                    <el-input
                        size="small"
                        placeholder="请输入用户姓名"
                        v-model="filterUser">
                        <i slot="prefix" class="el-input__icon el-icon-search"></i>
                    </el-input>

                    <el-tree
                        @node-click="selectClient"
                        class="filter-tree"
                        :data="userList"
                        :props="defaultProps"
                        empty-text="用户列表为空"
                        :filter-node-method="filterNode"
                        ref="userTree">
                    </el-tree>
                </el-aside>
            </el-container>
        </el-container>
    </div>
</template>

<script>
import 'rui-vue-emoji/dist/vue-emoji.css'
import VueEmoji from 'rui-vue-emoji'
import { keepLastIndex, isReqSuccessful, resetFile } from '@/util/jskit'
import { getExpert, getUserById, postExpressions, getExpressions, getClients, getTalkRecord } from '@/util/getdata'
import { wsUrl, baseUrl, tokenStr, authStr } from '@/util/fetch'

export default {
    components: {
        VueEmoji
    },

    data () {
        return {
            expert: {},
            user: {},

            showEmoji: false,
            items: [],
            activeIndex: '1',

            options: [],
            filterUser: '',
            
            currentExpression: '',
            expressionList: [],
            userList: [
                {label: '所有用户', children: []}
            ],
            defaultProps: {
                children: 'children',
                label: 'label'
            },

            msgCount: 0
        }
    },

    mounted () {
        let id = this.$route.params.id

        getUserById(id).then(res => {
            if (isReqSuccessful(res)) {
                let { userRealname } = res.data.model
                this.expert = {
                    id,
                    name: userRealname
                }
            }
        }).then(_ => {
            // TODO: this.expert.id 根据专家 ID 获取常用语列表
            getExpressions(this.expert.id).then(res => {
                if (isReqSuccessful(res)) {
                    let arr = []
                    res.data.List.forEach(v => {
                        arr.push({label: v.expression})
                    })
                    this.expressionList = arr
                }
            })
            getClients(id).then(res => {
                if (isReqSuccessful(res)) {
                    let arr = []
                    res.data.List.forEach((item) => {
                        arr.push({
                            id: item.id,
                            label: item.user_realname || `用户${item.id}`
                        })
                    })
                    this.userList[0].children = arr
                    // TODO: id-name 点击name获取聊天记录
                }
            })
        })

        this.$refs.emoji.appendTo({
            area: this.$refs.edit,
            btn: this.$refs.btn,
            position: 'top left'
        })

        let ws = wsUrl + '/' + id
        this.websocket = new WebSocket(ws)
        let cb = evt => {
            this.$notify.error({
                duration: 5000,
                title: '错误',
                message: evt.message || '连接已关闭'
            })
        }
        this.websocket.onerror = cb 
        this.websocket.onclose = cb

        this.websocket.onmessage = evt => {
            console.log(evt)
            let data = JSON.parse(evt.data)
            let html = ''
            this.msgCount++
            if (this.msgCount === 1) {
                this.user = {
                    id: data.talk_id,
                    name: data.name
                }
                console.log(this.user)
            }
            if (data.order === 'link') {
                let msg = data.message
                let idx = msg.lastIndexOf(':')
                let name = msg.substr(idx + 1)
                let addr = msg.substr(0, idx)
                html = `<a href="${addr}"><i class="el-icon-document"></i>${name}</a>`
            } else {
                html = data.message
            }
            this.pushChatMessage(html, data.order === 'self')
        }

        window.onbeforeunload = function () {
            return false
        }
    },

    destroyed () {
        window.onbeforeunload = undefined
    },

    methods: {
        async selectClient (data, node) {
            if (node.isLeaf) {
                let res = await getTalkRecord(data.id)
                let arr = []
                let id = this.$route.params.id
                res.data.List.forEach(v => {
                    let obj = {}
                    if (v.talker_id === id) {
                        obj.self = 1
                        this.expert.name = v.talker_name
                    } else {
                        this.user.name = v.talker_name
                    }
                    obj.html = v.content
                    arr.push(obj)
                })
                this.items = arr
                this.$nextTick(_ => {
                    let dialog = this.$refs.dialog
                    if (dialog) {
                        dialog.scrollTop = dialog.scrollHeight
                    }
                })
                // console.log(arr)
            }
        },

        filterNode (value, data) {
            if (!value) return true
            return data.label.indexOf(value) !== -1
        },

        handleEmojiSelect (img) {
            let edit = this.$refs.edit
            img.src = img.src.replace('/images', '/static/images')
            edit.appendChild(img)
            keepLastIndex(edit)
        },
        // TODO: 添加常用语插入光标所在位置
        // 添加常用语
        addExpression (expression) {
            if ( expression.length <= 1 || expression.length > 20 ) {
                this.$message('常用语长度应保持在2--20以内')
                return false 
            }
            if ( !this.expressionList.some(val =>  val.label === expression) ) {
                postExpressions({
                    expert_id: this.expert.id,
                    expression
                }).then(res => {
                    if (isReqSuccessful(res)) {
                        this.$message('成功添加常用语')
                        getExpressions(this.expert.id).then(res => {
                            if (isReqSuccessful(res)) {
                                let arr = []
                                res.data.List.forEach(v => {
                                    arr.push({label: v.expression})
                                })
                                this.expressionList = arr
                            }
                        })
                    } else {
                        this.$message('添加失败')
                    }
                }).catch(res => {
                    this.$message('添加失败')
                });
            }
            this.$refs.edit.innerHTML += expression
        },

        deleteExpression (index) {
            console.log( index )
        },

        send (e) {
            e && e.preventDefault()

            let edit = this.$refs.edit
            if (edit.innerHTML === '') {
                this.$message.warning('请输入内容')
                return
            }
            let data = {
                message: edit.innerHTML,
                isExpert: true,
                user_id: this.expert.id,
                name: this.user.name,
                talk_id: this.user.id,
                mode: 0
            }
            try {
                this.websocket.send(JSON.stringify(data))
            } catch (e) {
                this.$message.error('消息发送失败')
                return
            }
            edit.innerHTML = ''
        },

        sendFile () {
            let file = this.$refs.file.files[0]

            if (file === undefined) {
                this.$message.warning('未选择文件')
                return
            }
            let form = new FormData()
            form.append('file', file)
            form.append('user_id', this.user.id)
            form.append('user_name', this.user.name)
            form.append('talk_id', this.expert.id)
            form.append('mode', 0)

            let headers = {}
            headers[authStr] = window.localStorage.getItem(tokenStr)

            window.fetch(baseUrl + '/talk/upload', {
                method: 'POST',
                body: form,
                headers
            }).then(res => {
                if (isReqSuccessful(res)) {
                    this.$message.success('文件发送成功')
                    this.data.push({html: '<i class="el-icon-document"></i>' + file.name})
                    console.log(res)
                }
            }, _ => {
                this.$notify.error({
                    duration: 5000,
                    title: '错误',
                    message: '文件发送失败'
                })
            })
            resetFile(this.$refs.file)
        },

        invite () {
            // agent id
            getExpert(2).then(res => {
                if (isReqSuccessful(res)) {
                    this.$message.success('邀请成功')
                    let data = {
                        mode: 1,
                        to: [+this.user.id, this.expert.id, res.data.expert_id],
                        user_id: this.expert.id
                    }
                    // console.log(data)
                    this.websocket.send(JSON.stringify(data))
                }
            }, _ => {
                this.$message.error('邀请失败')
            })
        },

        pushChatMessage (html, isSelf) {
            this.items.push({html, self: isSelf})
            this.$nextTick(_ => {
                let dialog = this.$refs.dialog
                if (dialog) { 
                    dialog.scrollTop = dialog.scrollHeight
                }
            })
        }
    },
    watch: {
        filterUser (val) {
            this.$refs.userTree.filter(val)
        }
    }
}
</script>

<style lang="stylus">
@import '../../../assets/css/color'

.el-select-dropdown__item
    display flex
    justify-content space-between
    align-items center

.pro_wrapper
    padding 10px 0
    height auto
    font-size 15px
    .el-input input
        border 1px solid #dcdfe6
        border-radius 5px
    .el-header
        height 80px !important
        padding 0
        img
            position absolute
            z-index 777
        li
            float right
            height 100%
            line-height 80px
            &:hover
                background-color color-main
            &:active
                background-color color-main
            .select
                width 75px
                background-color color-main
                    input
                        background-color color-main
                        border none
                        color #fff
    .el-menu--horizontal
        background-color color-main
        width 100%
        height 100%
    .aside
        box-sizing border-box
        padding 0 10px
        .filter-tree
            margin-top 5px
    .el-main
        overflow hidden
        .pro-dialog_box
            overflow-y auto
            height 250px
            border 1px solid #e4e7ed
            border-radius 5px
            .pro-dialog-item
                overflow hidden
                font-size 16px
                margin 10px 20px
                line-height 25px
                span, .msg
                    float left
                    padding 5px 5px 5px 0
                .msg
                    float left
                    max-width 80%
                    padding-left 15px
                    padding-right 15px
                    border-radius 10px
                    color #000
                    background-color rgba(240,240,240,0.5)
                    i
                        font-size 20px
                    a
                        color #fff
                &.self
                    margin-left 0
                    margin-right 15px
                    span, .msg
                        float right
                    .msg
                        color #fff
                        background-color #3385ff
        .my_input_box
            position relative
            .my_chat_option
                height 50px
                line-height 50px
                width 100%
                i
                    margin-right 5px
                    font-size 20px
                    cursor pointer
                el-select
                    padding 3px 5px
                    border-radius 3px
                    font-size 13px
                    color #fff
                    cursor pointer
                    background-color color-main
            .rui-emoji
                top 80px
                left 20px
            .send
                position absolute
                right 10px
                bottom 10px
        .my_chat_input
            padding 10px 20px 50px 20px
            border 1px solid #e4e7ed
            border-radius 5px
            color #6f7180
            .chat_area
                min-height 100px
                outline 0
</style>
