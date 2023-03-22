<!--
 * @Author: Tom
 * @LastEditors: Tom
 * @Date: 2021-08-30 10:10:39
 * @LastEditTime: 2023-03-22 14:39:47
 * @Email: str-liang@outlook.com
 * @FilePath: \xjc:\Users\Tom\Desktop\ITable.vue
 * @Environment: Win 10 Python 3.8
 * @Description: 公共table

  data        table需要的数据
  columns     table里要展示需要的参数   可以设置每条item的宽度 默认160
  operation   每行有操作的话需要配置  没有直接等于空就行
  page        分页    需要分页的话就传，不需要的话就传个 {} 

  插槽 提供了两个插槽
    select： 可以在table上面加下拉
    other    table里 你想要整别的事的插槽,用的这个插槽的话 需要这样          <el-table-column label="" width="180" slot="other"></el-table-column>


    使用方法:
    <i-table
      :data="tableList"
      :columns="columns"
      :operation="operation"
      :page="page"
      :tableOrder="tableOrder"
      @childPageSize="parentPageSize"
      @childCurrentPage="parentCurrentPage"
    >
      <div slot="select">
      </div>
    </i-table>

 // 属性
  columns: [{ attrName: "actionTime", label: "时间" }],
  
-->
<template>
  <div>
    <div><slot name="select"></slot></div>
    <div class="table">
      <!-- table -->
      <el-table
        highlight-current-row
        :data="data"
        :border="true"
        style="width: 100%"
        :row-class-name="tableRowClassName"
        :cell-style="tableCellStyle"
        :row-key="getRowKeys"
        :expand-row-keys="expands"
        @selection-change="handleSelectionChange"
        @current-change="handleClickCurrentChange"
      >
        <el-table-column type="selection" v-if="tableSelect" width="55"></el-table-column>
        <el-table-column type="index" v-if="tableOrder" label="序号" width="60"></el-table-column>

        <el-table-column
          v-for="(item, index) in columns"
          :key="index"
          :sortable="item.sort"
          :prop="item.attrName"
          :label="item.label"
          :width="item.tableItemWidth"
        ></el-table-column>

        <el-table-column width="180" v-if="restsUpa.length > 0 && servBtnShow">
          <template slot="header" slot-scope="scope">
            <div class="slot_header">
              <el-tooltip content="面议服务请直接输入价格，价格1元起整数填写" placement="top">
                <span>数量(价格)修改</span>
              </el-tooltip>
            </div>
          </template>
          <template slot-scope="scope" class="restsUpa_content">
            <el-input v-model="scope.row.quantity" type="Number" size="mini"></el-input>
            <el-button
              size="mini"
              type="primary"
              v-for="(item, index) in restsUpa"
              :key="index"
              v-model="scope.row.quantity"
              @click="handleBtnUpa($event, scope.row, item.type)"
            >
              {{ item.label }}
            </el-button>
          </template>
        </el-table-column>

        <slot name="other"></slot>

        <el-table-column label="操作" width="250" v-if="operation.length > 0 && servBtnShow">
          <template slot-scope="scope">
            <el-button-group>
              <el-button
                v-for="(item, index) in operation"
                :type="item.type"
                :key="index"
                :title="item.operateName"
                size="mini"
                @click="operateType(item.operateType, scope.row, scope.$index)"
              >
                {{ item.operateName }}
              </el-button>
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="page" v-show="pageShow">
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page.currentPage"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="page.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="page.totalNumber"
      ></el-pagination>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ITable',
  props: {
    data: { type: null, default: () => [] },
    columns: { type: Array, default: () => [] },
    operation: { type: Array, default: () => [] },
    page: { type: Object, default: () => {} },
    rests: { type: Array, default: () => [] },
    restsUpa: { type: Array, default: () => [] },
    servBtnShow: { type: Boolean, default: () => true },
    tableItemWidth: { type: Number, default: () => 160 },
    tableOrder: { type: Boolean, default: () => true },
    tableSelect: { type: Boolean, default: () => false }
  },
  data() {
    return {
      expands: [],
      getRowKeys: (row) => {
        return row.id
      }
    }
  },
  computed: {
    pageShow() {
      return JSON.stringify(this.page) !== '{}'
    }
  },
  methods: {
    // 点击 操作
    operateType(type, row, index) {
      this.$emit('childLookClick', row, type)
    },
    //分页方法--改变每页的条数
    handleSizeChange(data) {
      this.$emit('childPageSize', data)
    },
    //分页方法--改变页码
    handleCurrentChange(data) {
      this.$emit('childCurrentPage', data)
    },
    // 修改按钮
    handleBtnUpa(event, row, type) {
      this.$emit('childhandleBtnUpa', event, row, type)
    },
    // 多选框
    handleSelectionChange(val) {
      this.$emit('childSelectionChange', val)
    },
    // 点击某一条数据
    handleClickCurrentChange(val) {
      this.$emit('childClickCurrentChange', val)
    },

    tableRowClassName({ row, rowIndex }) {
      if (row.deleted == true) {
        return 'deleted-row'
      }
      return ''
    },

    tableCellStyle({ row, column, columnIndex }) {
      if (column.label == '是否派车' && row.carQuestionMark == true) {
        return 'background:rgb(153, 230, 133)'
      } else if (column.label == '是否到期' && row.expire == true) {
        return 'background:rgb(236 192 189)'
      } else {
        return ''
      }
    }
  }
}
</script>

<style lang="less" scoped>
// @import "../../../assets/css/iconfont.css";
/deep/ .deleted-row {
  background-color: rgb(247, 177, 177);
}

/deep/ .el-button-group {
  width: 100%;
  display: flex;
  justify-content: space-around;
}
.restsUpa_content {
  display: flex;
}
/deep/.cell {
  display: flex;
  justify-content: space-between;
  .el-input {
    width: 60%;
  }
  .el-button {
    width: 40%;
    margin-left: 10px;
  }
}
/deep/ .el-table--enable-row-hover .el-table__body tr:hover > td {
  background-color: #84cbd9;
}

.slot_header {
  display: flex;
  .iconfont_put {
    padding-left: 10px;
    color: red;
    cursor: pointer;
  }
}
</style>
