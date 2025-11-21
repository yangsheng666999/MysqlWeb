<template>
  <div class="contain">
    <div class="card">
      <div class="card-header">
        <h2 class="card-title">全站商品管理</h2>
        <p class="card-subtitle">查看所有商品的当前状态并进行强制下架操作</p>
      </div>

      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>商品名</th>
              <th>卖家名</th>
              <th>价格（¥）</th>
              <th>当前状态</th>
              <th>时间</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody v-if="list.length > 0">
            <tr v-for="item in list" :key="item.gid">
              <td class="col-name" :title="item.gname">{{ item.gname }}</td>
              <td class="col-seller">{{ item.uid }}</td>
              <td class="col-price">¥ {{ item.price.toFixed(2) }}</td>
              <td>
                <span
                  class="status-tag"
                  :class="item.status === '0' ? 'status-on' : 'status-off'"
                >
                  {{ item.status === '0' ? '在售' : '已售出' }}
                </span>
              </td>
              <td class="col-time">
                {{ item.time.split('T')[0] + ' ' + item.time.split('T')[1] }}
              </td>
              <td>
                <button
                  v-if="item.status === '0'"
                  class="btn btn-danger"
                  @click="delgood(item.gid)"
                >
                  强制下架
                </button>
                <span v-else class="text-muted">无</span>
              </td>
            </tr>
          </tbody>

          <tbody v-else>
            <tr>
              <td colspan="6" class="empty">
                当前暂无数据
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="pagination-wrapper">
        <el-pagination
          layout="prev, pager, next, jumper"
          :total="total"
          :page-size="11"
          v-model="Page"
          @current-change="changePage"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { onBeforeMount, ref } from 'vue'
  import { getAllBuyMessages } from '@/hook/AllBuyMessage.ts'
  import { deletegood } from '@/hook/sold.ts'

  interface tabledata {
    gid: string
    gname: string
    price: number
    status: string
    time: string
    uid: string
  }
  const list = ref<Array<tabledata>>([])
  const Page = ref<number>(1)
  const total = ref<number>(0)

  onBeforeMount(async () => {
    try {
      const res = await getAllBuyMessages(Page.value)
      if (res !== null) {
        const data = res.data
        list.value = data.list
        total.value = data.total
        Page.value = data.pageNum
      }
    } catch {}
  })

  async function changePage(val: number) {
    Page.value = val
    try {
      const res = await getAllBuyMessages(Page.value)
      if (res !== null) {
        const data = res.data
        list.value = data.list
        total.value = data.total
        Page.value = data.pageNum
      }
    } catch {}
  }

  async function delgood(gid: string) {
    try {
      await deletegood(gid)
    } catch {
    } finally {
      const res = await getAllBuyMessages(Page.value)
      if (res !== null) {
        const data = res.data
        list.value = data.list
        total.value = data.total
        Page.value = data.pageNum
      }
    }
  }
</script>

<style scoped>
  .contain {
    display: flex;
    justify-content: center;
    padding: 30px 16px;
    background: #f5f7fa;
    min-height: calc(100vh - 60px);
    box-sizing: border-box;
  }

  .card {
    width: 100%;
    max-width: 1100px;
    background: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 18px rgba(0, 0, 0, 0.06);
    padding: 20px 24px 16px;
    box-sizing: border-box;
  }

  .card-header {
    margin-bottom: 12px;
    border-bottom: 1px solid #bda650;
    padding-bottom: 10px;
  }

  .card-title {
    margin: 0;
    font-size: 20px;
    font-weight: 600;
    color: #303133;
  }

  .card-subtitle {
    margin-top: 6px;
    margin-bottom: 0;
    font-size: 13px;
    color: #909399;
  }

  .table-wrapper {
    width: 100%;
    overflow-x: auto;
    margin-top: 10px;
  }

  table {
    border-collapse: collapse;
    width: 100%;
    table-layout: fixed;
    font-size: 14px;
  }

  thead {
    background: linear-gradient(90deg, #ffa640, #e9e932);
    color: #ffffff;
  }

  th,
  td {
    padding: 10px 8px;
    text-align: center;
    word-break: break-all;
  }

  th {
    font-weight: 500;
  }

  tbody tr:nth-child(even) {
    background-color: #f9fafc;
  }

  tbody tr:nth-child(odd) {
    background-color: #ffffff;
  }

  tbody tr:hover {
    background-color: #ecf5ff;
  }

  .col-name {
    text-align: left;
    padding-left: 12px;
  }

  .col-seller {
    color: #606266;
  }

  .col-price {
    font-weight: 600;
    color: #f56c6c;
  }

  .col-time {
    font-size: 13px;
    color: #909399;
  }

  .status-tag {
    display: inline-block;
    padding: 3px 10px;
    border-radius: 999px;
    font-size: 12px;
    line-height: 1.4;
  }

  .status-on {
    background: #ecf9f1;
    color: #41b883;
  }

  .status-off {
    background: #f4f4f5;
    color: #909399;
  }

  .btn {
    border: none;
    outline: none;
    padding: 6px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 13px;
    transition: all 0.2s;
    white-space: nowrap;
  }

  .btn-danger {
    background-color: #f56c6c;
    color: #ffffff;
  }

  .btn-danger:hover {
    background-color: #f78989;
    box-shadow: 0 2px 6px rgba(245, 108, 108, 0.35);
  }

  .text-muted {
    color: #c0c4cc;
    font-size: 13px;
  }

  .empty {
    padding: 40px 0;
    color: #909399;
    font-size: 14px;
  }

  .pagination-wrapper {
    display: flex;
    justify-content: center;
    margin-top: 12px;
  }

  :deep(.el-pagination) {
    font-size: 13px;
  }

  :deep(.el-pagination__jump span),
  :deep(.number) {
    font-size: 13px;
  }

  :deep(.el-pagination__jump input) {
    font-size: 13px;
  }

  :deep(.el-pager li.is-active) {
    color: #ffffff;
    background-color: #409eff;
  }

  :deep(.el-pager li:hover) {
    color: #409eff;
  }

  :deep(.el-pager li) {
    background-color: #f4f4f5;
    margin: 0 4px;
  }

  :deep(.el-pagination button) {
    background-color: #f4f4f5;
  }

  :deep(.el-pagination button:hover) {
    color: #409eff;
  }

  @media (max-width: 768px) {
    .card {
      padding: 16px 12px;
    }

    .card-title {
      font-size: 18px;
    }

    th,
    td {
      padding: 8px 6px;
      font-size: 12px;
    }

    .btn {
      padding: 4px 8px;
      font-size: 12px;
    }
  }
</style>
