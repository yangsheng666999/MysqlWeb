<template>
  <div class="user-manage">
    <!-- 顶部标题 -->
    <div class="header">
      <h2>用户管理</h2>
      <p class="subtitle">查看并管理系统中的所有用户账户</p>
    </div>

    <!-- 用户表格卡片 -->
    <el-card class="table-card" shadow="hover">
      <el-table
        :data="UserList"
        border
        stripe
        style="width: 100%"
        v-loading="loading"
      >
        <el-table-column
          prop="username"
          label="用户名"
          align="center"
          min-width="140"
        />

        <el-table-column
          label="权限"
          align="center"
          min-width="120"
        >
          <template #default="{ row }">
            <el-tag :type="row.pass === '0' ? 'info' : 'warning'">
              {{ row.pass === '0' ? '普通用户' : '管理员' }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column
          label="状态"
          align="center"
          min-width="120"
        >
          <template #default="{ row }">
            <el-tag :type="row.isBanned === 1 ? 'danger' : 'success'">
              {{ row.isBanned === 1 ? '已封禁' : '正常' }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column
          label="操作"
          align="center"
          min-width="260"
          fixed="right"
        >
          <template #default="{ row }">
            <el-button
              size="small"
              @click="dialogshow(row.username, row.pass)"
            >
              修改权限
            </el-button>
            <el-button
              size="small"
              type="danger"
              @click="delUser(row.username)"
            >
              删除用户
            </el-button>
            <el-button
              size="small"
              type="warning"
              @click="dialogBanUser(row.username, row.isBanned)"
            >
              {{ row.isBanned === 1 ? '解封' : '封禁' }}
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination-wrapper">
        <el-pagination
          layout="prev, pager, next, jumper"
          :total="Usertotal"
          :page-size="15"
          v-model:current-page="Page"
          @current-change="changepage"
        />
      </div>
    </el-card>

    <!-- 修改权限对话框 -->
    <el-dialog v-model="dialogVisible" :title="dialogtitle" width="420">
      <el-radio-group v-model="radiocheck">
        <el-radio value="0" size="large">普通用户</el-radio>
        <el-radio value="1" size="large">管理员用户</el-radio>
      </el-radio-group>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="changepass">确认</el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 封禁用户对话框 -->
    <el-dialog v-model="dialogVisibleBanUser" :title="dialogtitle" width="420">
      <el-radio-group v-model="checkbanned">
        <el-radio :value="0" size="large">解封用户</el-radio>
        <el-radio :value="1" size="large">封禁用户</el-radio>
      </el-radio-group>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisibleBanUser = false">取消</el-button>
          <el-button type="primary" @click="banUser">确认</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
  import { onBeforeMount, ref } from 'vue'
  import { getAllUser } from '@/hook/getAllUser.ts'
  import { updatapass } from '@/hook/updatapass.ts'
  import { deleteUser } from '@/hook/deleteUser.ts'
  import { updateIsBanned } from '@/hook/updateIsBanned'

  // 修改权限对话框
  const dialogVisible = ref<boolean>(false)
  const dialogtitle = ref<string>('')

  const checkname = ref<string>('')
  const radiocheck = ref<string | null>('')

  // 封禁用户对话框
  const checkbanned = ref<number>(0)
  const dialogVisibleBanUser = ref<boolean>(false)

  // 表格 & 分页
  interface tabledata {
    username: string
    pass: string
    isBanned: number
  }
  interface PageResponse {
    pageNum: number
    total: number
    list: tabledata[]
  }

  const UserList = ref<tabledata[]>([])
  const Usertotal = ref<number>(0)
  const Page = ref<number>(1)
  const loading = ref<boolean>(false)

  // 统一获取用户列表
  async function fetchUsers(page = Page.value) {
    try {
      loading.value = true
      const reponse = await getAllUser(page)
      if (reponse !== null) {
        const data: PageResponse = reponse.data
        UserList.value = data.list
        Usertotal.value = data.total
        Page.value = data.pageNum
      }
    } finally {
      loading.value = false
    }
  }

  onBeforeMount(() => {
    fetchUsers()
  })

  // 打开封禁对话框
  function dialogBanUser(name: string, isBanned: number) {
    dialogtitle.value = '封禁用户 ' + name + ' 的权限'
    checkbanned.value = isBanned
    checkname.value = name
    dialogVisibleBanUser.value = true
  }

  // 打开修改权限对话框
  function dialogshow(name: string, pass: string) {
    dialogtitle.value = '修改用户 ' + name + ' 的权限'
    radiocheck.value = pass
    checkname.value = name
    dialogVisible.value = true
  }

  // 修改权限
  async function changepass() {
    try {
      await updatapass({ username: checkname.value, pass: radiocheck.value as string })
      dialogVisible.value = false
      fetchUsers()
    } catch {
      dialogVisible.value = false
    }
  }

  // 封禁 / 解封用户
  async function banUser() {
    try {
      await updateIsBanned({ username: checkname.value, isBanned: checkbanned.value })
      dialogVisibleBanUser.value = false
      fetchUsers()
    } catch (error) {
      console.error('封禁失败', error)
    }
  }

  // 删除用户
  async function delUser(username: string) {
    try {
      await deleteUser(username)
      fetchUsers()
    } catch {}
  }

  // 换页
  async function changepage(val: number) {
    Page.value = val
    fetchUsers(val)
  }
</script>

<style scoped>
.user-manage {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px;
  background: #f5f7fa;
  min-height: calc(100vh - 60px);
  box-sizing: border-box;
}

.header {
  width: 100%;
  max-width: 1000px;
  margin-bottom: 16px;
}

.header h2 {
  margin: 0;
  font-size: 24px;
  font-weight: 600;
  color: #303133;
}

.header .subtitle {
  margin-top: 6px;
  font-size: 14px;
  color: #909399;
}

.table-card {
  width: 100%;
  max-width: 1000px;
}

/* 分页 */
.pagination-wrapper {
  margin-top: 16px;
  display: flex;
  justify-content: center;
}

/* 对话框 footer 统一靠右 */
.dialog-footer {
  text-align: right;
}

/* 调整分页字体大小 */
:deep(.el-pagination__jump span),
:deep(.el-pagination__jump input),
:deep(.el-pager li),
:deep(.el-pagination button) {
  font-size: 14px;
}
:deep(.el-pager li.is-active) {
  color: #fff;
  background-color: rgba(239, 135, 9, 0.84);
}
:deep(.el-pager li:hover) {
  color: #fa8610;
}
:deep(.el-pagination button) {
  background-color: rgba(111, 114, 115, 0.06);
}
:deep(.el-pagination button:hover) {
  color: #fa8610;
}
</style>
