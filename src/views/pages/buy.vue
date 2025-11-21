<template>
  <div class="contain">
    <el-card class="record-card" shadow="always">
      <div class="card-header">
        <div class="title-wrap">
          <span class="title">购买记录</span>
          <span class="subtitle">共 {{ total }} 条</span>
        </div>
        <span class="tips">只展示最近的购买记录，如需更多可在后台筛选</span>
      </div>

      <el-table
        :data="list"
        border
        stripe
        style="width: 100%"
        v-loading="loading"
        empty-text="当前暂无购买记录"
        highlight-current-row
      >
        <el-table-column
          prop="gname"
          label="购买物品"
          align="center"
          min-width="200"
          show-overflow-tooltip
        />

        <el-table-column
          prop="price"
          label="价格（元）"
          align="center"
          width="140"
        >
          <template #default="{ row }">
            <span class="price">¥ {{ row.price.toFixed(2) }}</span>
          </template>
        </el-table-column>

        <el-table-column
          label="购买时间"
          align="center"
          min-width="220"
        >
          <template #default="{ row }">
            {{ formatTime(row.time) }}
          </template>
        </el-table-column>
      </el-table>

      <div class="pagination-wrapper">
        <el-pagination
          layout="prev, pager, next, jumper"
          :total="total"
          :page-size="15"
          v-model:current-page="Page"
          @current-change="changePage"
        />
      </div>
    </el-card>
  </div>
</template>

<script lang="ts" setup>
  import { onBeforeMount, ref } from 'vue'
  import { userStore } from '@/stores/user.ts'
  import { getAllBuy } from '@/hook/buy.ts'

  interface TableData {
    gid: string
    gname: string
    price: number
    time: string
  }

  const user = userStore()
  const list = ref<TableData[]>([])
  const Page = ref<number>(1)
  const total = ref<number>(0)
  const loading = ref<boolean>(false)

  // 时间格式化：把 2025-11-21T19:21:30 变成 2025-11-21 19:21:30
  function formatTime(value: string) {
    if (!value) return '-'
    const [date, time] = value.split('T')
    if (!time) return value
    // 截到秒，防止后面有 .000Z 之类
    const t = time.split('.')[0]
    return `${date} ${t}`
  }

  async function fetchData(page: number) {
    try {
      loading.value = true
      const username = localStorage.getItem('username') as string
      const response = await getAllBuy(username, page)
      const data = response.data
      list.value = data.list || []
      total.value = data.total || 0
      Page.value = data.pageNum || page
    } finally {
      loading.value = false
    }
  }

  onBeforeMount(async () => {
    await fetchData(Page.value)
  })
  
  async function changePage(val: number) {
    Page.value = val
    await fetchData(val)
  }
</script>

<style scoped>
.contain {
  min-height: calc(100vh - 80px);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 24px 16px;
  box-sizing: border-box;
  background: radial-gradient(circle at top, #fef6e7 0, #f5f7fa 45%, #eef1f6 100%);
}

.record-card {
  width: 80%;
  max-width: 1000px;
  border-radius: 16px;
  box-shadow: 0 14px 30px rgba(0, 0, 0, 0.06);
  border: none;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 12px;
}

.title-wrap {
  display: flex;
  align-items: baseline;
  gap: 8px;
}

.title {
  font-size: 22px;
  font-weight: 700;
  color: #303133;
}

.subtitle {
  font-size: 14px;
  color: #909399;
}

.tips {
  font-size: 13px;
  color: #a0a3b1;
}

.price {
  font-weight: 600;
  font-size: 16px;
  color: #f48a1f;
}

/* 分页居中 */
.pagination-wrapper {
  display: flex;
  justify-content: center;
  margin-top: 18px;
}

/* Element Plus 表头 & 行高优化 */
:deep(.el-table__header th) {
  background-color: #faf3e8;
  color: #606266;
  font-weight: 600;
  font-size: 15px;
}

:deep(.el-table__body td) {
  padding: 10px 0;
  font-size: 14px;
}

/* 分页样式延续你之前的颜色 */
:deep(.number) {
  font-size: 14px;
}

:deep(.el-pagination__jump span),
:deep(.el-pagination__jump input) {
  font-size: 14px;
}

:deep(.el-pager li.is-active) {
  color: white;
  background-color: rgba(239, 135, 9, 0.9);
}

:deep(.el-pager li:hover) {
  color: #fa8610;
}

:deep(.el-pager li) {
  background-color: rgba(111, 114, 115, 0.06);
  margin: 0 4px;
}

:deep(.el-pagination button) {
  background-color: rgba(111, 114, 115, 0.06);
}

:deep(.el-pagination button:hover) {
  color: #fa8610;
}
</style>
