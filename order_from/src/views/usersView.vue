<template>
    <div class="max-box">
        <div class="items-box">
            <el-form :inline="true" :model="formInline" ref="ruleFormRef" class="demo-form-inline">
                <el-form-item label="用户ID" prop="id">
                    <el-input v-model="formInline.id" placeholder="请输入用户的唯一ID" clearable />
                </el-form-item>
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="formInline.username" placeholder="请输入用户名" clearable />
                </el-form-item>
                <el-form-item label="真实姓名" prop="real_name">
                    <el-input v-model="formInline.real_name" placeholder="请输入用户的真实姓名" clearable />
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="onSubmit()">查询</el-button>
                    <el-button @click="resetForm(ruleFormRef)">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
        <div class="base-table">
            <div class="action">
                <el-button type="primary" @click="handleAddUser(dialogFormRef)"
                    v-permission="'/users^POST'">新增</el-button>
                <el-popconfirm v-if="chooseUserIds.length" width="225"
                    :title="'你确定要批量删除这' + chooseUserIds.length + '个用户吗？'" @confirm="handleBatchDelete">
                    <template #reference>
                        <el-button type="danger">批量删除</el-button>
                    </template>
                </el-popconfirm>
                <el-button v-else type="danger" @click="isDelete" v-permission="'/users/delete^POST'">批量删除</el-button>

            </div>
            <div>
                <el-table :data="userTableData" @selection-change="handleSelectionChange">
                    <el-table-column type="selection" width="55"></el-table-column>
                    <el-table-column v-for="item in columns" :prop="item.prop" :label="item.label" :width="item.width"
                        :formatter="item.formatter" :key="item.id" />
                    <el-table-column fixed="right" label="操作" width="160">
                        <template #default="scope">
                            <el-button type="primary" size="small" @click="handleEdit(scope.row)">编辑</el-button>
                            <!-- <el-button type="danger" size="small" @click="handleDelete(scope.row)">删除</el-button> -->
                            <el-popconfirm width="225" :title="'你确定要删除用户：' + scope.row.username + '吗？'"
                                @confirm="confirmEvent(scope.row)">
                                <template #reference>
                                    <el-button size="small" type="danger">删除</el-button>
                                </template>
                            </el-popconfirm>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
            <el-pagination class="pagination" background layout="prev, pager, next" :page-size="pager.size"
                :total="pager.total" @current-change="handleCurrentChange" />
        </div>
        <!-- 弹出的对话框-->
        <el-dialog v-model="centerDialogVisible" :title="title" align-center>
            <el-form ref="dialogFormRef" :model="userForm" label-width="100px" :rules="rules" status-icon>
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="userForm.username" placeholder="用户名" :disabled="action == 'edit'" />
                </el-form-item>

                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="userForm.email" placeholder="邮箱地址" :disabled="action == 'edit'">
                        <template #append>
                            @126.com
                        </template>
                    </el-input>
                </el-form-item>

                <el-form-item label="手机号码" prop="phone">
                    <el-input v-model="userForm.phone" placeholder="手机号码" />
                </el-form-item>

                <el-form-item label="真实姓名" prop="real_name">
                    <el-input v-model="userForm.real_name" placeholder="真实姓名" />
                </el-form-item>


                <el-form-item label="分配角色" prop="roles">
                    <el-select v-model="userForm.roles" placeholder="请选择系统角色" multiple style="width: 100%">
                        <el-option v-for="role in allRoleList" :key="role.id" :label="role.name"
                            :value="role.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="部门" prop="dept_id">
                    <el-cascader placeholder="请选择所属部门" v-model="userForm.dept_id" :options="allDeptList"
                        :props="{ checkStrictly: true, value: 'id', label: 'name' }" @change="handleChange"
                        style="width: 90%" />
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="handleClose(dialogFormRef)">取消</el-button>
                    <el-button type="primary" @click="handleSubmit(dialogFormRef)">确认</el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script setup>
import lodash from 'lodash'
import { reactive, ref } from 'vue'
// import { searchUser, deleteUser, getRoleList, getRoleParent, createUser, updateUser } from '@/api/api'
import { ElMessage } from 'element-plus'
const formInline = reactive({
    id: '',
    username: '',
    real_name: ''
})
function isDelete() {
    if (chooseUserIds.length === 0) {
        ElMessage.error('批量删除：至少要选择一个！')
        return
    }
}
const ruleFormRef = ref()
// 初始化表格的数据
const userTableData = ref()




const columns = reactive([
    { label: '用户ID', prop: 'id', width: 70 },
    { label: '用户名', prop: 'username', width: 100 },
    { label: '手机号码', prop: 'phone', width: 140, formatter(row, column, value) { return value ? value : value = '----' } },
    { label: '用户邮箱', prop: 'email', width: 150, formatter(row, column, value) { return value ? value : value = '----' } },
    { label: '真实姓名', prop: 'real_name', width: 120, formatter(row, column, value) { return value ? value : value = '----' } },
    { label: '所在部门', prop: 'dept.name', width: 130, formatter(row, column, value) { return value ? value : value = '----' } },
    {
        label: '创建时间', prop: 'create_time', width: 190, formatter(row, column, value) {
            let dt = new Date(value)
            return dt.toLocaleString()
        }
    },
])

// 初始化分页的数据
const pager = reactive({
    page: 1, // 初始页面
    size: 5 // 每页最多显示的条数
})

// 查询用户列表数据
async function searchUserList() {
    const params = { ...lodash.pickBy(formInline, item => item), ...pager }
    const result = await searchUser(params)
    if (result) {
        userTableData.value = result.items
        pager.total = result.total
    }
}
// 批量删除的ID
const chooseUserIds = reactive([])

// 获得需要批量删除的ID
const handleSelectionChange = (row_all) => {
    chooseUserIds.length = 0
    row_all.forEach(item => {
        chooseUserIds.push(item.id)
    });
}

// 批量删除
async function handleBatchDelete() {
    await deleteUser(chooseUserIds)
    ElMessage.success('删除成功,本次批量删除了' + chooseUserIds.length + '个用户')
    searchUserList() // 刷新数据
}

// 搜索数据
const onSubmit = () => {
    searchUserList()
}

// 分页函数
function handleCurrentChange(page) {
    pager.page = page
    searchUserList()
}

// 删除单个用户
// async function handleDelete(row) {

//     // const res = await deleteUser([row.id])
//     ElMessage.success('成功删除' + row.username + '用户')
//     searchUserList()
// }


// 重置搜索表单数据
function resetForm(formEl) {
    if (!formEl) return;
    formEl.resetFields()
}

async function confirmEvent(row) {
    await deleteUser([row.id])
    ElMessage.success('成功删除' + row.username + '用户')
    searchUserList()
}

// 控制对话框->是否显示
const centerDialogVisible = ref(false)
// 验证表单数据
const dialogFormRef = ref()
// 对话框的标题 -> 
const title = ref()
// 表单的数据
const userForm = reactive({
    username: '',
    email: '',
    phone: '',
    real_name: '',
    roles: [],
    dept_id: ''

})
//新增用户rules表单验证规则
const rules = reactive({
    username: [
        { required: true, message: "用户名不能为空", trigger: "blur" }
    ],
    email: [
        { required: true, message: "邮箱不能为空", trigger: "blur" }
    ],
    phone: [
        { required: true, message: "手机号不能为空", trigger: "blur" },
        { pattern: /1\d{10}/, message: "手机号格式不正确", trigger: "blur" }
    ],
    roles: [
        { required: true, message: "分配角色不能为空", trigger: "blur" },
    ],
    dept_id: [
        { required: true, message: "所属部门不能为空", trigger: "blur" },
    ]
})

// 控制输入框禁用->新增和修改区分变量
const action = ref()
// role角色列表
const allRoleList = ref([])
// 获取角色列表
async function getAllRoleList() {
    const result = await getRoleList()
    allRoleList.value.push(...result)
}
// 所有部门的列表
const allDeptList = ref([])
// 获取所有父部门
async function getAllDeptParent() {
    const result = await getRoleParent()
    allDeptList.value.push(...result)
}
// 编辑按钮
function handleEdit(row) {
    centerDialogVisible.value = true
    action.value = 'edit'
    title.value = '修改用户'
    const dates = {}
    Object.assign(dates, row)
    const RoleIDs = []
    row.roles.forEach((role) => {
        RoleIDs.push(role.id)
    })
    dates.roles = RoleIDs
    Object.assign(userForm, dates)
}

// 新增按钮事件
function handleAddUser(ref) {
    centerDialogVisible.value = true
    action.value = 'add'
    title.value = '新增用户'
    resetForm(ref)
}
// 用户点击了确认
function handleSubmit(formEl) {
    formEl.validate(async (valid) => {
        if (valid) {
            if (action.value === 'add') {
                const dates = {}
                Object.assign(dates, userForm)
                dates.dept_id = dates.dept_id[dates.dept_id.length - 1]
                dates.email += '@126.com'
                const result = await createUser(dates)
                if (result) {
                    ElMessage.success('添加用户成功')
                    centerDialogVisible.value = false
                    searchUserList()
                    resetForm(formEl)
                }
            }
            else if (action.value === 'edit') {
                const dates = {}
                Object.assign(dates, userForm)
                dates.dept_id = dates.dept_id[dates.dept_id.length - 1]
                const result = await updateUser(dates, dates.id)
                if (result) {
                    ElMessage.success('修改用户成功')
                    centerDialogVisible.value = false
                    searchUserList()
                    resetForm(formEl)
                }
            } else {
                ElMessage.error('参数出错！')
            }
        }
    })
}
// 用户点击了取消
function handleClose(formEl) {
    centerDialogVisible.value = false
    resetForm(formEl)
}

searchUserList()
getAllRoleList()
getAllDeptParent()
</script>

<style lang='scss' scoped>
.max-box {
    .items-box {
        background-color: white;
        display: flex;
        justify-content: center;
        align-items: center;
        padding-top: 20px;
    }
}

.base-table {
    border-radius: 5px;
    background: #ffffff;
    margin-top: 20px;
    margin-bottom: 20px;
}

.action {
    border-radius: 5px 5px 0px 0px;
    background: #ffffff;
    padding: 20px;
    border-bottom: 1px solid #ece8e8;
}

.pagination {
    justify-content: flex-end;
    padding: 14px;
}
</style>
