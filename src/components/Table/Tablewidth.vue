<template>
  <div :class="[prefixCls + '-table', prefixCls + '-table-'+ size, loading && (prefixCls + '-table-loading')]">
    <spin size="sm" v-if="loading"></spin>
    <!-- <table :class="['atui-table-fixed-header','atui-table']" v-if="fixedHeader">
  </table> -->
    <div :class="[prefixCls + '-table-container', fixedHeader && (prefixCls + '-table-fixed-header')]">
      <table>
        <colgroup>
          <col v-if="rowSelection"></col>
          <col v-if="expandedRowRender"></col>
          <col v-for="column in columns" :width="column.width"></col>
        </colgroup>
        <thead>
          <slot name = "head-row" :columns="columns">
          <tr style="border: 1px solid #000000">
            <th v-if="rowSelection" :class="[prefixCls + '-table-selection-column']">
              <input v-if="dataList && dataList.length" type="checkbox" v-bind="{checked:isCheckedAll,disabled:isDisabledAll}" @change="onCheckAll"
              />
            </th>
            <th v-if="expandedRowRender" :class="[prefixCls + '-table-expand-icon-th']"></th>
            <th style="border: 1px solid #000000" v-for="(column, index) in columns" :width="column.width" @mousemove="mousemove"
                @mouseout="mouseout" @mousedown="mousedown($event,column)" >
              <span v-html="column['title']"></span>
              <dropdown ref="filterMenu" v-if="column.filters" trigger="hover">
                <div>
                  <icon type="filter" size="12"></icon>
                </div>
                <div name="dropdown-menu" slot="dropdown-menu" :class="[prefixCls + '-dropdown-menu', prefixCls + '-table-filter-dropdown']">
                  <template v-if="column.filterMultiple !== false">
                    <ul>
                      <li v-for="filter in column.filters">
                        <label>
                          <template v-if="column.filterMultiple === false">
                            <input type="radio" :value="filter.value" v-model="filters[column.dataIndex]" />{{filter.text}}
                          </template>
                          <template v-else>
                            <input type="checkbox" :value="filter.value" v-model="filters[column.dataIndex]" />{{filter.text}}
                          </template>
                        </label>
                      </li>
                    </ul>
                    <div :class="[prefixCls + '-table-filter-dropdown-btns']">
                      <a :class="[prefixCls + '-table-filter-dropdown-link confirm']" @click="onFilter()">确定</a>
                      <a :class="[prefixCls + '-table-filter-dropdown-link', prefixCls + '-table-clear']" @click="resetFilter(column)">重置</a>
                    </div>
                  </template>
                  <template v-else>
                    <ul :class="[prefixCls + '-table-filter-single']">
                      <li :class="[prefixCls + '-table-filter-single-item', column.__selectedText === '全部'?'selected':'']" @click="resetFilter(column)">
                        <icon type="tick" size="10"></icon><span>全部</span>
                      </li>
                      <li v-for="filter in column.filters" @click="onFilter(true, column, filter.value, filter.text)" :class="[column.__selectedText === filter.text?'selected':'', prefixCls + '-table-filter-single-item']">
                        <icon type="tick" size="10"></icon><span>{{filter.text}}</span>
                      </li>
                    </ul>
                  </template>
                </div>
              </dropdown>
              <div v-if="dataList && dataList.length && column.sorter" :class="[prefixCls + '-table-sorter']">
                <icon type="caretup" size="10" @click.native="sortAction(column, index, 'ascend')" :class="[sorderOrder[index] == 'ascend' && (prefixCls + '-table-active')]"></icon>
                <icon type="caretdown" size="10" @click.native="sortAction(column, index, 'descend')" :class="[sorderOrder[index] == 'descend' && (prefixCls + '-table-active')]"></icon>
              </div>
            </th>
          </tr>
          </slot>
        </thead>
        <tbody>
          <tr v-if="!dataList || !dataList.length">
            <td colspan="20" style="text-align: center;" :class="[prefixCls + '-table-empty']">{{noDataTip}}</td>
          </tr>
          <template v-for="(record, rowIndex) in dataList">
            <slot name="row" :record="record" :row-index="row-index">
              <tr :track-by="rowIndex"  @click="onRowClick(rowIndex, record)">
                <td v-if="rowSelection" :class="[prefixCls + '-table-selection-column']">
                  <input type="checkbox" v-model="checkedValues" :value="record[rowKey]" @change.stop="onCheckOne($event,record)" v-bind="rowSelection.getCheckboxProps && rowSelection.getCheckboxProps(record)"
                  />
                </td>
                <td v-if="expandedRowRender" :class="[prefixCls + '-table-row-expand-icon-cell']">
                  <span v-if="!record.__no_expand" :class="[prefixCls + '-table-row-expand-icon', prefixCls + (record.__expanded == 1 ? '-table-row-expanded' : '-table-row-collapsed') ]"  @click="onRowExpand(rowIndex, record)"></span>
                </td>
                <td v-for="column in columns" :class="[column.className || '']" @click="onColumnClick(column,record,rowIndex)">
                  <template v-if="column.render && record">
                    <span v-html="column.render.call(this._context,record[column.dataIndex],record,rowIndex)" />
                  </template>
                  <template v-else-if="column.renderList && record">
                    <span v-for="render in column.renderList"
                          @click.stop="render.renderClick.call(this._context,record[column.dataIndex],record,rowIndex)"
                          v-html="render.renderHtml.call(this._context,record[column.dataIndex],record,rowIndex)"/>
                  </template>
                  <template v-else>
                    <span v-html="record[column.dataIndex]"></span>
                  </template>
                </td>
              </tr>
              <tr v-if="record.__expanded" :class="[prefixCls + '-table-expanded-row']">
                <td>
                  <span :class="[prefixCls + '-expanded-row-indent']" v-if="!record.__no_expand"></span>
                </td>
                <td :colspan="columns.length" v-html="expandedRowRender(record)">
                </td>
              </tr>
            </slot>
          </template>
        </tbody>
      </table>
    </div>
    <div v-show="pagination && pagination.total > 0" :class="[prefixCls + '-table-pagination']">
      <pagination ref="pager" :total="pagination.total" :page-size="pagination.pageSize" :show-jumper="true" :show-size-changer="true" @pagination-page-change="changePage" @pagination-size-change="changeSize"></pagination>
    </div>
    <div class="el-table__column-resize-proxy" style="left: 700px;" ref="resizeProxy" v-show="resizeProxyVisible"></div>
  </div>
</template>

<script type="text/babel">
import Vue from 'vue'
import Icon from '../Icon/'
import Dropdown from '../Dropdown/'
import Spin from '../Spin/'
import Pagination from '../Pagination/'

export default {
  name: 'Table',
  props: {
    pagination: {
      type: Object,
      default () {
        return {
          total: 0,
          pageSize: 10,
          currPage: 1
        }
      }
    },
    dataSource: {
      type: Array,
      default () {
        return []
      }
    },
    noDataTip: {
      type: String,
      default: '没有任何数据'
    },
    columns: {
      type: Array,
      default () {
        return []
      }
    },
    expandedRowRender: Function,
    rowSelection: Object,
    rowKey: String,
    loading: {
      type: Boolean,
      default: false
    },
    fixedHeader: Boolean,
    size: {
      type: String,
      default: 'default'
    },
    prefixCls: {
      type: String,
      default: 'atui'
    }
  },
  components: {
    Icon,
    Dropdown,
    Spin,
    Pagination
  },
  data () {
    const filters = {}
    const columnMap = {}
    this.columns.forEach((item) => {
      Vue.set(item, '__selectedText', '全部')
      columnMap[item.dataIndex] = item
      if (item.filters) {
        // 如果有filter的情况，则把filter保存为一个空对象，filter时的chechbox需要用到双向绑定
        filters[item.dataIndex] = []
      }
    })
    return {
      originDataSource: Object.assign(this.dataSource || [], []),
      dataList: Object.assign(this.dataSource || [], []),
      isCheckedAll: false,
      isDisabledAll: false,
      sorderOrder: [],
      checkedRows: [],
      filterOpened: false,
      filters: filters,
      columnMap: columnMap,
      sorter: {},
      resizeProxyVisible: true
    }
  },
  mounted () {
    if (this.dataList) {
      this.dataList.forEach((record) => {
        Vue.set(record, '__expanded', 0)
      })
    }

    if (this.pagination.total > 0) {
      let pager = this.$refs.pager
      this.dataList = this.originDataSource.slice(pager.currPage || 0, pager.pageSize)
    }
  },
  computed: {
    checkedValues () {
      let me = this
      let checkedKeys = me.checkedRows.map((record) => {
        return record[me.rowKey]
      })
      if (me.rowSelection && me.rowSelection.onChange) {
        me.rowSelection.onChange(checkedKeys, me.checkedRows)
      }
      return checkedKeys
    },
    checkebleRows () {
      let me = this
      let rows = []
      // 过滤出非禁用的项供选择使用
      if (me.dataList && me.dataList.length) {
        rows = me.dataList.filter((record) => {
          if (me.rowSelection) {
            return !me.rowSelection.getCheckboxProps || !me.rowSelection.getCheckboxProps(record).disabled
          }
        })
      }
      return rows
    }
  },
  watch: {
    dataSource (data, oldData) {
      this.dataList = data
    },
    dataList: {
      handler (data, oldData) {
        let me = this
        data.forEach && data.forEach((record) => {
          if (!record.hasOwnProperty('__expanded')) {
            Vue.set(record, '__expanded', 0)
          }
        })
        me.compileTbody()
        // 如果有删除行为或者清空行为，则需要把选中行数据重新计算出，否则checkedRow一直存在没变化
        me.checkedRows = data.filter((record) => {
          if (me.checkedValues) {
            return me.checkedValues.indexOf(record[me.rowKey]) >= 0
          }
        })
        if (me.checkebleRows.length) {
          me.isCheckedAll = me.checkedRows.length === me.checkebleRows.length
        }
      },
      deep: true
    }
  },
  methods: {
    onColumnClick (column, record, rowIndex) {
      if (column.columnClick && typeof column.columnClick === 'function') {
        column.columnClick(column, record, rowIndex)
      }
    },
    onRowClick (rowIndex, record) {
      this.$emit('row-click', rowIndex, record)
    },
    onRowExpand (rowIndex, record) {
      record.__expanded = !record.__expanded
    },
    compileTbody () {
      let me = this
      //  因为table里有html和事件绑定，所以需要重新调用$compile，而马上调用时可能页面还没有重新渲染完成
      me.$nextTick(() => {
        // me._context.$compile(me.$el)
        me.isDisabledAll = !me.checkebleRows.length
      })
    },
    sortAction (column, index, order) {
      if (typeof column.sorter === 'function') {
        // TODO:客户端排序
      }
      this.sorderOrder[index] = order
      this.sorderOrder = Object.assign([], this.sorderOrder)
      this.$emit('table-change', this.pagination, this.filters, {
        field: column.dataIndex,
        order: order
      })
    },
    // 点击全选框触发
    onCheckAll (event) {
      let me = this
      let changeRows = []
      let input = event.srcElement || event.target
      let checked = input.checked
      if (checked) {
        me.checkebleRows.forEach((record, i) => {
          if (me.checkedRows.indexOf(record) < 0) {
            me.checkedRows.push(record)
            changeRows.push(record)
          }
        })
        // 不能使用计算属性，因为会与点击全选的行为相冲突
        me.isCheckedAll = true
      } else {
        me.checkebleRows.forEach((record, i) => {
          if (me.checkedRows.indexOf(record) >= 0) {
            changeRows.push(record)
          }
        })
        me.checkedRows = []
      }
      if (me.rowSelection.onSelectAll) {
        me.rowSelection.onSelectAll(checked, me.checkedRows, changeRows)
      }
    },
    // 选中某一个单选框时触发
    onCheckOne (event, record) {
      const me = this
      let input = event.srcElement || event.target
      const checked = input.checked
      if (checked) {
        if (me.checkedRows.indexOf(record) === -1) {
          me.checkedRows.push(record)
        }
      } else {
        me.checkedRows = me.checkedRows.filter((item) => {
          return record[me.rowKey] !== item[me.rowKey]
        })
      }
      if (me.rowSelection.onSelect) {
        me.rowSelection.onSelect(record, checked, me.checkedRows)
      }
      me.isCheckedAll = me.checkedRows.length === me.checkebleRows.length
    },
    // isSingle:是否为单选模式
    onFilter (isSingle, column, fileterValue, filterText) {
      // 每次filter条件变化都应该拿原始数据把所有条件都过滤一遍
      let me = this
      let filters = me.filters
      if (isSingle) {
        filters[column.dataIndex] = fileterValue
        Vue.set(column, '__selectedText', filterText)
      }
      me.$refs.filterMenu.forEach((dropdown) => {
        dropdown.$emit('closeDropdown')
      })
      this.dataList = this.originDataSource
      for (let currColumnKey in filters) {
        let currColumn = me.columnMap[currColumnKey]
        let value = filters[currColumnKey]

        if (currColumn.hasOwnProperty('filterMultiple') && currColumn.filterMultiple === false) {
          /* vue的v-model会把radio的值转换成一个字符串，这里为了参数格式与checkbox相同
          则再转换成数组 */
          // 先注释这个逻辑了，这样filterMultiple为true的时候返回一个数组，false的时候返回单个字符串
          // value = value.constructor === Array ? value : [value]
        }
        if (currColumn.onFilter) {
          //  配置了onFilter就是本地模式，否则就派发 事件
          me.dataList = me.dataList.filter((record) => {
            // reset时value是空数组，此时应该true
            return value.length === 0 || currColumn.onFilter.call(this, value, record)
          })
        } else {
          me.$emit('table-change', me.pagination, me.filters, me.sorter)
        }
      }
    },
    resetFilter (column) {
      Vue.set(column, '__selectedText', '全部')
      this.filters[column.dataIndex] = []
      this.onFilter()
    },
    changePage (pageNum) {
      // console.log('changePage:', pageNum)
      let me = this
      let pager = me.$refs.pager
      // 如果原始originDataSource有多余数据，证明是客户端分页
      if (me.originDataSource.length > pager.pageSize) {
        me.dataList = me.originDataSource.slice(
          (pageNum - 1) * pager.pageSize,
          pageNum * pager.pageSize
        )
      }
      // this.pagination.onChange && this.pagination.onChange(pageNum)
      me.pagination.currPage = pageNum
      me.$emit('table-change', this.pagination, me.filters, me.sorter)
    },
    changeSize (current, pageSize) {
      let pager = this.$refs.pager
      this.dataList = this.originDataSource.slice(
        (current - 1) * pager.pageSize,
        current * pager.pageSize
      )
      this.pagination.onShowSizeChange && this.pagination.onShowSizeChange(current, pageSize)
    },
    // fixedHeaderAction () {
    //   if (this.fixedHeader) {
    //     let header = this.$el.querySelector('.table-thead')
    //     let colgroup = this.$el.querySelector('colgroup').cloneNode(true)
    //     let fixedTable = this.$el.querySelector('.atui-table-fixed-header')
    //     fixedTable.appendChild(colgroup)
    //     fixedTable.appendChild(header)
    //   }
    // }
    mousemove () {
      let target = event.target
      let rect = target.getBoundingClientRect()
      const bodyStyle = document.body.style
      if (rect.width > 12 && rect.right - event.pageX < 8) {
        bodyStyle.cursor = 'col-resize'
      }
    },
    mouseout () {
      document.body.style.cursor = ''
    },
    mousedown (event, column) {
      this.dragging = true
      this.resizeProxyVisible = true
      console.log(this)
      const tableEl = this.$el
      // 计算div前面空多少加入到相对位置计算竖线位置
      const tableLeft = tableEl.getBoundingClientRect().left
      const columnEl = event.target
      console.log(columnEl)
      const columnRect = columnEl.getBoundingClientRect()
      const minLeft = columnRect.left - tableLeft + 30

      columnEl.classList.add('noclick')
      this.dragState = {
        startMouseLeft: event.clientX,
        startLeft: columnRect.right - tableLeft,
        startColumnLeft: columnRect.left - tableLeft,
        tableLeft
      }

      const resizeProxy = this.$refs.resizeProxy
      resizeProxy.style.left = this.dragState.startLeft + 'px'

      document.onselectstart = function () {
        return false
      }
      document.ondragstart = function () {
        return false
      }

      const handleMouseMove = (event) => {
        const deltaLeft = event.clientX - this.dragState.startMouseLeft
        const proxyLeft = this.dragState.startLeft + deltaLeft
        console.log(event.clientX)
        console.log(proxyLeft)
        resizeProxy.style.left = Math.max(minLeft, proxyLeft) + 'px'
      }

      const handleMouseUp = () => {
        if (this.dragging) {
          const finalLeft = parseInt(resizeProxy.style.left, 10)
          const columnWidth = finalLeft - this.dragState.startColumnLeft
          column.width = column.realWidth = columnWidth
          console.log('column.width' + column.width)
//            this.store.scheduleLayout()

          document.body.style.cursor = ''
          this.dragging = false
          this.draggingColumn = null
          this.dragState = {}

          this.resizeProxyVisible = false
        }

        document.removeEventListener('mousemove', handleMouseMove)
        document.removeEventListener('mouseup', handleMouseUp)
        document.onselectstart = null
        document.ondragstart = null

        setTimeout(function () {
          columnEl.classList.remove('noclick')
        }, 0)
      }

      document.addEventListener('mousemove', handleMouseMove)
      document.addEventListener('mouseup', handleMouseUp)
    }
  }
}
</script>
<style>
  .el-table__column-resize-proxy {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 0;
    border-left: 3px solid #000000;
    z-index: 9999;
  }

  .tbclass {
    border: 1px solid #002a80;
  }
  .tbclass tr td{
    border: 1px solid #002a80;
  }
</style>
