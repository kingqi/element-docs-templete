<template>
  <div>
    <!--批量操作栏，勾选行时显示-->
    <div
      v-show="_config.enableMultiSelect && selection.length > 0"
      class="multi-menu"
    >
      <span style="margin-left:12px;">已选中{{ selection.length }}项</span>
      <el-divider direction="vertical" />
      <slot
        name="multiSelectMenu"
        :selection="selection"
      />
      <i
        class="el-icon-close btn-close"
        @click="clearSelection"
      />
    </div>
    <!--顶部操作栏占位，勾选行时不显示-->
    <div
      v-show="_config.enableMultiSelect && selection.length === 0"
      class="top-menu"
    >
      <slot name="topMenu" />
    </div>
    <el-table
      ref="table"
      v-loading="loading"
      :data="data"
      v-bind="elProps"
      v-on="listeners"
    >
      <el-table-column
        v-if="_config.enableMultiSelect"
        type="selection"
        width="55"
      />
      <el-table-column
        v-if="_config.showIndexColumn"
        type="index"
        width="50"
      />
      <table-column
        v-for="column in columns"
        :key="column.prop"
        :column="column"
      >
        <slot
          v-if="column.slot"
          slot-scope="props"
          :name="column.prop"
          v-bind="props"
        />

        <slot
          v-if="column.slot"
          slot="header"
          slot-scope="props"
          :name="column.prop + '-header'"
          v-bind="props"
        />
      </table-column>
      <el-table-column
        v-if="_config.showHandler"
        v-bind="handlerColumn"
      >
        <slot
          slot-scope="{ row }"
          name="handler"
          :row="row"
        />
      </el-table-column>
    </el-table>
    <table-pagination
      v-if="page"
      :page="page"
      :page-config="pageConfig"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
import { EL_TABLE_PROPS, EL_TABLE_METHODS } from './config'
import { get } from 'main/utils/util'
import tableColumn from './table-column'
import tablePagination from './table-pagination'

export default {
  name: 'MgTable',
  mgName: 'MgTable',
  components: {
    tableColumn,
    tablePagination
  },
  props: {
    data: {
      type: Array,
      default: () => []
    },
    columns: {
      type: Array,
      default: () => []
    },
    loading: Boolean,
    config: {
      type: Object,
      default: () => ({})
    },
    pageConfig: { type: Object, default: () => ({}) },
    page: {
      type: Object,
      default: () => null
    }
  },
  data() {
    return {
      selection: []
    }
  },
  computed: {
    _config() {
      return Object.assign(
        {
          enableMultiSelect: false,
          showHandler: false,
          handlerColumn: {},
          highlightSelect: true,
          showIndexColumn: false,
          tooltipEffect: 'dark'
        },
        this.config
      )
    },
    // 操作列配置
    handlerColumn() {
      return Object.assign(
        {
          label: '操作',
          minWidth: 100,
          fixed: 'right'
        },
        this._config.handlerColumn
      )
    },
    // el-table组件属性
    elProps() {
      const copy = {}
      for (const key in this._config) {
        if (EL_TABLE_PROPS.includes(key)) {
          copy[key] = this._config[key]
        }
      }
      if (this._config.highlightSelect) {
        Object.assign(copy, {
          'row-class-name': this.rowClassName
        })
      }
      return copy
    },
    // el-table监听事件
    listeners() {
      return Object.assign({}, this.$listeners, {
        select: this.handleSelect,
        'select-all': this.handleSelectAll
      })
    }
  },
  watch: {
    // 如果当前数据有已被选中的则设置为已勾选
    data: function(val) {
      this.$nextTick(() => {
        if (val.length > 0 && this.selection.length > 0) {
          val.forEach((row) => {
            if (
              this.selection.findIndex((item) => this._getRowKey(item) === this._getRowKey(row)) >=
              0
            ) {
              this.$refs['table'].toggleRowSelection(row, true)
            }
          })
        }
      })
    }
  },
  methods: {
    _getRowKey(row) {
      const config = this._config
      if (typeof config.rowKey === 'function') {
        return config.rowKey(row)
      } else if (typeof config.rowKey === 'string') {
        return get(row, config.rowKey)
      } else {
        return row.id
      }
    },
    handleCurrentChange(val) {
      this.$emit('current-page-change', val)
    },
    handleSizeChange(val) {
      this.$emit('page-size-change', val)
    },
    handleSelect(selection, row) {
      if (selection.includes(row)) {
        this.selection.push(row)
      } else {
        this.selection.splice(this.selection.indexOf(row), 1)
      }
      this.$emit('select', this.selection, row)
    },
    handleSelectAll(selection) {
      let index
      this.data.forEach((row) => {
        index = this.selection.findIndex((item) => this._getRowKey(item) === this._getRowKey(row))
        // 全选时
        if (selection.length > 0 && index < 0) {
          this.selection.push(row)
          // 全不选时
        } else if (selection.length === 0 && index >= 0) {
          this.selection.splice(index, 1)
        }
      })
      this.$emit('select-all', this.selection)
    },
    // 高亮当前选中行
    rowClassName({ row }) {
      for (let index = 0; index < this.selection.length; index++) {
        if (this.selection[index] === row) {
          return 'row__active'
        }
      }
      return ''
    },
    clearSelection() {
      this.selection = []
      this.$refs['table'].clearSelection()
    },
    ...EL_TABLE_METHODS.reduce((acc, method) => {
      acc[method] = function() {
        return this.$refs['table'][method](...arguments)
      }
      return acc
    }, {})
  }
}
</script>
<style lang="scss" scoped>
::v-deep .row__active {
  background: #f7fcff;
}
.multi-menu {
  display: inline-block;
  width: 100%;
  line-height: 40px;
  height: 40px;
  margin-bottom: 12px;
}
.multi-menu .el-button {
  padding-top: 0;
  padding-bottom: 0;
}
.btn-close {
  line-height: 40px;
  margin-right: 10px;
  float: right;
  cursor: pointer;
}
.top-menu {
  height: 40px;
  margin-bottom: 12px;
}
</style>
