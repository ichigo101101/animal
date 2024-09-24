<template>
    <div>
        <div class="search">
            <el-input placeholder="请输入商品名称" style="width: 200px" v-model="name"></el-input>
            <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
            <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
        </div>

        <div class="table">
            <div style="padding: 10px 15px">
                <el-row>
                    <el-col :span="6" v-for="item in tableData" style="margin-bottom: 20px">
                        <div style="display: flex">
                            <div style="flex: 1; padding-right: 5px">
                                <img :src="item.img" alt="" style="width: 100%; height: 155px; border-radius: 10px; border: 1px solid #cccccc">
                            </div>
                            <div style="flex: 1; padding-left: 5px">
                                <div style="font-size: 15px; font-weight: 550; color: #817a70">{{ item.name }}</div>
                                <div style="margin-top: 5px; font-weight: 600; font-size: 20px; color: red">￥ {{ item.price }}</div>
                                <div style="margin-top: 5px; font-size: 13px; color: #4b4949">剩余数量：{{ item.num }}</div>
                                <div style="margin-top: 10px">
                                    <el-input-number v-model="item.tmpNum" :min="1" :max="item.num" size="mini"></el-input-number>
                                </div>
                                <div style="margin-top: 15px">
                                    <el-button type="warning" size="mini" :disabled="item.num === 0">购买</el-button>
                                </div>
                            </div>
                        </div>
                    </el-col>
                </el-row>
            </div>
            <div class="pagination">
                <el-pagination
                        background
                        @current-change="handleCurrentChange"
                        :current-page="pageNum"
                        :page-sizes="[5, 10, 20]"
                        :page-size="pageSize"
                        layout="total, prev, pager, next"
                        :total="total">
                </el-pagination>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        name: "GoodsUser",
        data() {
            return {
                tableData: [],  // 所有的数据
                pageNum: 1,   // 当前的页码
                pageSize: 12,  // 每页显示的个数
                total: 0,
                name: null,
                fromVisible: false,
                form: {},
                user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
                rules: {
                },
                ids: []
            }
        },
        created() {
            this.load(1)
        },
        methods: {
            handleAdd() {   // 新增数据
                this.form = {}  // 新增数据的时候清空数据
                this.fromVisible = true   // 打开弹窗
            },
            handleEdit(row) {   // 编辑数据
                this.form = JSON.parse(JSON.stringify(row))  // 给form对象赋值  注意要深拷贝数据
                this.fromVisible = true   // 打开弹窗
            },
            save() {   // 保存按钮触发的逻辑  它会触发新增或者更新
                this.$refs.formRef.validate((valid) => {
                    if (valid) {
                        this.$request({
                            url: this.form.id ? '/goods/update' : '/goods/add',
                            method: this.form.id ? 'PUT' : 'POST',
                            data: this.form
                        }).then(res => {
                            if (res.code === '200') {  // 表示成功保存
                                this.$message.success('保存成功')
                                this.load(1)
                                this.fromVisible = false
                            } else {
                                this.$message.error(res.msg)  // 弹出错误的信息
                            }
                        })
                    }
                })
            },
            del(id) {   // 单个删除
                this.$confirm('您确定删除吗？', '确认删除', {type: "warning"}).then(response => {
                    this.$request.delete('/goods/delete/' + id).then(res => {
                        if (res.code === '200') {   // 表示操作成功
                            this.$message.success('操作成功')
                            this.load(1)
                        } else {
                            this.$message.error(res.msg)  // 弹出错误的信息
                        }
                    })
                }).catch(() => {
                })
            },
            handleSelectionChange(rows) {   // 当前选中的所有的行数据
                this.ids = rows.map(v => v.id)   //  [1,2]
            },
            delBatch() {   // 批量删除
                if (!this.ids.length) {
                    this.$message.warning('请选择数据')
                    return
                }
                this.$confirm('您确定批量删除这些数据吗？', '确认删除', {type: "warning"}).then(response => {
                    this.$request.delete('/goods/delete/batch', {data: this.ids}).then(res => {
                        if (res.code === '200') {   // 表示操作成功
                            this.$message.success('操作成功')
                            this.load(1)
                        } else {
                            this.$message.error(res.msg)  // 弹出错误的信息
                        }
                    })
                }).catch(() => {
                })
            },
            load(pageNum) {  // 分页查询
                if (pageNum) this.pageNum = pageNum
                this.$request.get('/goods/selectPage', {
                    params: {
                        pageNum: this.pageNum,
                        pageSize: this.pageSize,
                        name: this.name,
                    }
                }).then(res => {
                    this.tableData = res.data?.list
                    this.total = res.data?.total
                })
            },
            reset() {
                this.name = null
                this.load(1)
            },
            handleCurrentChange(pageNum) {
                this.load(pageNum)
            },
            handleAvatarSuccess(response, file, fileList) {
                this.form.img = response.data
            },
        }
    }
</script>

<style scoped>

</style>