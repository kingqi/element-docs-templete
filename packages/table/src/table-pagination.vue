<template>
  <div :class="['page-wrap', customClassName]">
    <el-pagination
      :current-page="page.currentPage"
      :page-size="page.size"
      :total="page.total"
      :page-sizes="_pageConfig.pageSizes"
      :layout="_pageConfig.layout"
      v-bind="elProps"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
import { EL_PAGE_PROPS } from './config'

export default {
  props: {
    page: {
      type: Object,
      default: () => ({
        currentPage: 1,
        size: 3,
        total: 0
      })
    },
    pageConfig: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    _pageConfig() {
      return Object.assign(
        {
          pageSizes: [10, 20, 30, 50, 100],
          layout: 'total,prev,pager,next,sizes,jumper,->'
        },
        this.pageConfig
      )
    },
    customClassName() {
      return this._pageConfig.customClassName || ''
    },
    elProps() {
      const copy = {}
      for (const key in this._pageConfig) {
        if (EL_PAGE_PROPS.includes(key)) {
          copy[key] = this._pageConfig[key]
        }
      }
      return copy
    }
  },
  methods: {
    handleCurrentChange(val) {
      this.$emit('current-change', val)
    },
    handleSizeChange(val) {
      this.$emit('size-change', val)
    }
  }
}
</script>

<style scoped>
.page-wrap {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  margin-top: 12px;
}
</style>
