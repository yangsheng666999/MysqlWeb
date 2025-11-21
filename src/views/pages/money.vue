<template>
  <div class="recharge-page">
    <div class="recharge-card">
      <h2 class="title">账户充值</h2>
      <p class="subtitle">请选择充值金额或输入自定义金额</p>

      <!-- 输入金额 -->
      <div class="moneyin">
        <div class="moneyin-label">
          <span>充值金额（人民币）</span>
          <span class="moneyin-min">最低充值金额 10 元</span>
        </div>
        <el-input-number
          v-model="input1"
          :min="10"
          :step="10"
          size="large"
          class="money-input"
          placeholder="请输入金额"
        />
      </div>

      <!-- 快捷金额选择 -->
      <div class="preset-block">
        <div class="preset-title">快捷选择</div>

        <div class="itemgroup">
          <div
            class="item"
            v-for="price in money1"
            :key="price"
            :class="{ active: input1 === price }"
            @click="input1 = price"
          >
            <div class="price">{{ price }} 元</div>
            <div class="tag" v-if="price === 30">推荐</div>
          </div>
        </div>

        <div class="itemgroup">
          <div
            class="item"
            v-for="price in money2"
            :key="price"
            :class="{ active: input1 === price }"
            @click="input1 = price"
          >
            <div class="price">{{ price }} 元</div>
            <div class="tag" v-if="price === 648">大额充值</div>
          </div>
        </div>
      </div>

      <!-- 底部按钮 -->
      <div class="foot">
        <el-button
          type="primary"
          size="large"
          :disabled="!input1"
          @click="rechargemoney"
        >
          确认充值
        </el-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { reactive, ref, watch } from 'vue'
  import { userStore } from '@/stores/user.ts'
  import { recharge } from '@/hook/recharge.ts'
  import { getuser } from '@/hook/Login.ts'
  import { ElMessage } from 'element-plus'

  const money1 = reactive([10, 20, 30])
  const money2 = reactive([100, 200, 648])

  const input1 = ref<number | undefined>()

  // 金额不得小于 10 元
  watch(input1, (value) => {
    if (value != null && value < 10) {
      input1.value = 10
    }
  })

  const user = userStore()
  
  async function rechargemoney() {
    try {
      if (input1.value == null) {
        ElMessage.error('金额为空，无法充值')
        return
      }
      await recharge({ username: user.username, money: input1.value as number })
      await getuser(user.username)
      ElMessage.success('充值成功')
    } catch {
      ElMessage.error('充值失败，请稍后重试')
    }
  }
</script>

<style scoped>
.recharge-page {
  min-height: calc(100vh - 60px);
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #f3f6ff, #f8fbff);
  padding: 40px 16px;
}

.recharge-card {
  width: 600px;
  max-width: 100%;
  background: #ffffff;
  border-radius: 18px;
  padding: 32px 36px 28px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.06);
  box-sizing: border-box;
}

.title {
  margin: 0;
  font-size: 24px;
  font-weight: 600;
  color: #1f2f3d;
}

.subtitle {
  margin-top: 6px;
  margin-bottom: 24px;
  font-size: 14px;
  color: #909399;
}

/* 输入区域 */
.moneyin {
  margin-bottom: 24px;
}

.moneyin-label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  font-size: 14px;
  color: #606266;
}

.moneyin-min {
  font-size: 12px;
  color: #909399;
}

.money-input {
  width: 260px;
}

/* 快捷金额选择 */
.preset-block {
  margin-top: 8px;
}

.preset-title {
  font-size: 14px;
  color: #606266;
  margin-bottom: 10px;
}

.itemgroup {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 12px;
  flex-wrap: wrap;
}

.item {
  flex: 1;
  min-width: 160px;
  height: 90px;
  border-radius: 12px;
  border: 1px solid #dcdfe6;
  background: #f9fafc;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition:
    all 0.2s ease,
    transform 0.15s ease;
}

.item .price {
  font-size: 20px;
  font-weight: 500;
  color: #303133;
}

.item .tag {
  margin-top: 6px;
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 999px;
  background: rgba(64, 158, 255, 0.08);
  color: #409eff;
}

.item:hover {
  transform: translateY(-2px);
  border-color: #409eff;
  box-shadow: 0 8px 20px rgba(64, 158, 255, 0.15);
}

.item.active {
  border-color: #409eff;
  background: linear-gradient(135deg, #e6f3ff, #f3f8ff);
}

.item.active .price {
  color: #409eff;
}

/* 底部按钮 */
.foot {
  margin-top: 24px;
  text-align: right;
}

.foot :deep(.el-button) {
  border-radius: 999px;
  padding: 0 28px;
}
</style>
