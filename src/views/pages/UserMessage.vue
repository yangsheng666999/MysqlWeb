<template>
  <div class="profile-settings">
    <el-card class="profile-card" shadow="hover">
      <!-- 头部 -->
      <div class="card-header">
        <div>
          <div class="card-title">账户设置</div>
          <div class="card-subtitle">修改用户名和登录密码</div>
        </div>
      </div>

      <el-divider />

      <!-- 用户名模块 -->
      <section class="section">
        <div class="section-header">
          <span class="section-title">用户名</span>
        </div>

        <div class="username-row">
          <span class="field-label">当前用户名</span>

          <!-- 显示 / 编辑状态 -->
          <span v-if="!usernameChange" class="username-value">
            {{ user.username }}
          </span>
          <el-input
            v-else
            v-model="inputName"
            class="username-input"
            size="large"
            maxlength="16"
            show-word-limit
            placeholder="请输入新的用户名"
          />

          <div class="btn-group">
            <el-button
              v-if="!usernameChange"
              type="primary"
              size="small"
              @click="changName"
            >
              修改
            </el-button>

            <template v-else>
              <el-button
                type="primary"
                size="small"
                @click="changeName"
              >
                保存
              </el-button>
              <el-button
                size="small"
                @click="cancelChangeName"
              >
                取消
              </el-button>
            </template>
          </div>
        </div>
      </section>

      <el-divider />

      <!-- 密码模块 -->
      <section class="section">
        <div class="section-header">
          <span class="section-title">密码</span>
          <span class="section-tip">建议密码长度不少于 6 位，并包含字母和数字</span>
        </div>

        <el-form
          label-width="110px"
          class="password-form"
        >
          <el-form-item label="新密码">
            <el-input
              v-model="password1"
              type="password"
              show-password
              clearable
              placeholder="请输入新密码"
            />
          </el-form-item>

          <el-form-item label="确认密码">
            <el-input
              v-model="password2"
              type="password"
              show-password
              clearable
              placeholder="请再次输入新密码"
            />
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="changepassword">
              确认修改
            </el-button>
            <el-button @click="resetPasswordField">
              重置
            </el-button>
          </el-form-item>
        </el-form>
      </section>
    </el-card>
  </div>
</template>

<script lang="ts" setup>
  import { ref } from 'vue'
  import { ElMessage } from 'element-plus'
  import { userStore } from '@/stores/user.ts'
  import { updatePassword, updateUsername } from '@/hook/UserMessage.ts'

  const usernameChange = ref<boolean>(false)
  const user = userStore()
  const inputName = ref<string>('')

  const password1 = ref<string>('')
  const password2 = ref<string>('')

  function changName() {
    inputName.value = user.username
    usernameChange.value = true
  }

  function cancelChangeName() {
    usernameChange.value = false
    inputName.value = ''
  }

  async function changeName() {
    if (!inputName.value.trim()) {
      ElMessage.error('用户名不能为空')
      return
    }
    if (inputName.value === user.username) {
      ElMessage.info('用户名没有变化')
      usernameChange.value = false
      return
    }

    try {
      await updateUsername({ oldData: user.username, newData: inputName.value })
      // 如果后端返回成功，你这边最好也更新一下本地 store
      user.username = inputName.value
      ElMessage.success('用户名修改成功')
    } catch (e) {
      ElMessage.error('用户名修改失败，请稍后重试')
    } finally {
      usernameChange.value = false
    }
  }

  function resetPasswordField() {
    password1.value = ''
    password2.value = ''
  }

  async function changepassword() {
    if (!password1.value || !password2.value) {
      ElMessage.error('密码不能为空')
      return
    }
    if (password1.value.length < 6) {
      ElMessage.error('密码长度不能少于 6 位')
      return
    }
    if (password1.value !== password2.value) {
      ElMessage.error('两次密码不一致，无法修改')
      return
    }

    try {
      await updatePassword({ oldData: user.username, newData: password1.value })
      ElMessage.success('密码修改成功')
      resetPasswordField()
    } catch (e) {
      ElMessage.error('密码修改失败，请稍后重试')
    }
  }
</script>

<style scoped>
.profile-settings {
  display: flex;
  justify-content: center;
  padding: 32px 16px;
  box-sizing: border-box;
}

.profile-card {
  width: 100%;
  max-width: 720px;
  border-radius: 16px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.card-title {
  font-size: 22px;
  font-weight: 600;
}

.card-subtitle {
  margin-top: 4px;
  font-size: 13px;
  color: #909399;
}

.section {
  margin-top: 8px;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.section-title {
  font-size: 16px;
  font-weight: 600;
}

.section-tip {
  font-size: 12px;
  color: #a0a3b4;
}

.username-row {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 12px;
}

.field-label {
  font-size: 14px;
  color: #606266;
  min-width: 80px;
}

.username-value {
  font-size: 16px;
  font-weight: 500;
}

.username-input {
  max-width: 260px;
}

.btn-group {
  display: flex;
  gap: 8px;
}

.password-form {
  max-width: 480px;
}

/* 响应式简单处理 */
@media (max-width: 600px) {
  .card-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .username-row {
    align-items: flex-start;
  }

  .field-label {
    width: 100%;
  }

  .btn-group {
    width: 100%;
    justify-content: flex-start;
  }
}
</style>
