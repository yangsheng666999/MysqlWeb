<template>
  <div class="contain">
    <el-table
      :data="list"
      border
      stripe
      style="width: 90%"
      v-loading="loading"
    >
      <el-table-column
        prop="gname"
        label="物品名"
        align="center"
      />
      <el-table-column
        prop="price"
        label="价格"
        align="center"
      />
      <el-table-column
        label="创单时间"
        align="center"
      >
        <template #default="{ row }">
          {{ formatTime(row.time) }}
        </template>
      </el-table-column>
      <el-table-column
        label="是否在售"
        align="center"
      >
        <template #default="{ row }">
          <el-tag v-if="row.status === '0'" type="success">在售</el-tag>
          <el-tag v-else type="info">已售出</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
      >
        <template #default="{ row }">
          <template v-if="row.status === '0'">
            <el-button
              type="primary"
              size="small"
              @click="diashow(row.gname, row.gid, row.price)"
            >
              修改售价
            </el-button>

            <el-button
              type="danger"
              plain
              size="small"
              style="margin-left: 8px"
              @click="delgood(row.gid)"
            >
              取消销售
            </el-button>
          </template>

          <span v-else>无</span>
        </template>
      </el-table-column>

      <!-- 空数据提示 -->
      <template #empty>
        <el-empty description="暂无出售记录" :image-size="80" />
      </template>
    </el-table>

    <el-pagination
      class="pagination"
      layout="prev, pager, next, jumper"
      :total="total"
      :page-size="15"
      v-model:current-page="Page"
      @current-change="changePage"
    />

    <el-dialog v-model="dialogVisible" :title="dialogtitle" width="500">
      <div class="dialog-body">
        <span class="dialog-label">新的售价：</span>
        <el-input-number
          v-model="diaNum"
          :min="0"
          :step="1"
          controls-position="right"
        />
        <span class="dialog-unit">元</span>
      </div>

      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="changeprice">确认</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
  import { onBeforeMount, ref } from 'vue'
  import { deletegood, getAllrecode, updatePrice } from '@/hook/sold.ts'
  import { userStore } from '@/stores/user.ts'

  interface tabledata {
    gid: string
    gname: string
    price: number
    status: string
    time: string
  }

  const total = ref<number>(1)
  const Page = ref<number>(1)
  const dialogVisible = ref<boolean>(false)
  const dialogtitle = ref<string>('')
  const diaNum = ref<number>(0)
  const diagid = ref<string>('')
  const diaName = ref<string>('')

  const user = userStore()
  const list = ref<Array<tabledata>>([])
  const loading = ref<boolean>(false)   // 用来控制 v-loading

  // 公共的表格加载函数：统一控制 loading & 数据
  async function loadTableData(page = Page.value) {
    loading.value = true
    try {
      const username =
        user.username || (localStorage.getItem('username') as string)

      const res = await getAllrecode(page, username)
      if (res !== null) {
        const data = res.data
        list.value = data.list
        total.value = data.total
        Page.value = data.pageNum
      }
    } catch (e) {
      console.error('加载出售记录失败', e)
    } finally {
      loading.value = false
    }
  }

  function diashow(gname: string, gid: string, price: number) {
    dialogtitle.value = '修改' + gname + '的售价'
    diaName.value = gname
    diaNum.value = price
    diagid.value = gid
    dialogVisible.value = true
  }

  // 修改价格
  async function changeprice() {
    try {
      await updatePrice(diagid.value, diaNum.value)
      dialogVisible.value = false
      await loadTableData(Page.value)
    } catch (e) {
      console.error('修改价格失败', e)
    }
  }

  // 删除商品
  async function delgood(gid: string) {
    try {
      await deletegood(gid)
      await loadTableData(Page.value)
    } catch (e) {
      console.error('取消销售失败', e)
    }
  }

  onBeforeMount(async () => {
    await loadTableData(1)
  })

  // 换页
  async function changePage(val: number) {
    Page.value = val
    await loadTableData(val)
  }

  // 简单版本的 formatTime
  function formatTime(time: string) {
    if (!time) return ''
    const arr = time.split('T')
    return arr[0] + ' ' + arr[1].split('.')[0] // 防止带小数秒
  }
</script>

<style scoped>
.contain {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 20px;
}

/* 原生 table 的样式可以删了，这里只保留分页和弹窗相关的 */
.pagination {
  margin-top: 16px;
}

:deep(.el-pagination__jump span) {
  font-size: 16px;
}
:deep(.el-pagination__jump input) {
  font-size: 16px;
}
:deep(.el-pager li.is-active) {
  color: white;
  background-color: rgba(239, 135, 9, 0.84);
}
:deep(.el-pager li:hover) {
  color: #fa8610;
}
:deep(.el-pager li) {
  background-color: rgba(111, 114, 115, 0.06);
  margin-right: 5px;
  margin-left: 5px;
}
:deep(.el-pagination button) {
  background-color: rgba(111, 114, 115, 0.06);
}
:deep(.el-pagination button:hover) {
  color: #fa8610;
}

/* 弹窗内布局稍微美化一下 */
.dialog-body {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px 0 20px;
}
.dialog-label {
  font-size: 14px;
  margin-right: 10px;
}
.dialog-unit {
  margin-left: 10px;
}
</style>
