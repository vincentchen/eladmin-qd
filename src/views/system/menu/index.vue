<template>
  <div class="app-container">
    <eHeader :roles="roles" :menus="menus" :query="query"/>
    <!--表格渲染-->
    <tree-table v-loading="loading" :data="data" :expand-all="true" :columns="columns" border size="small">
      <el-table-column prop="icon" label="图标" align="center" width="80px">
        <template slot-scope="scope">
          <svg-icon :icon-class="scope.row.icon" />
        </template>
      </el-table-column>
      <el-table-column prop="sort" align="center" width="100px" label="排序">
        <template slot-scope="scope">
          <el-tag>{{ scope.row.sort }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column :show-overflow-tooltip="true" prop="path" label="链接地址"/>
      <el-table-column :show-overflow-tooltip="true" prop="component" label="组件路径"/>
      <el-table-column prop="iframe" width="100px" label="内部菜单">
        <template slot-scope="scope">
          <span v-if="!scope.row.iframe">是</span>
          <span v-else>否</span>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" label="创建日期">
        <template slot-scope="scope">
          <span>{{ time(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="150px" align="center">
        <template slot-scope="scope">
          <edit v-if="checkPermission(['ADMIN','MENU_ALL','MENU_EDIT'])" :roles="roles" :menus="menus" :data="scope.row" :sup_this="sup_this"/>
          <el-popover
            v-if="checkPermission(['ADMIN','MENU_ALL','MENU_DELETE'])"
            v-model="scope.row.delPopover"
            placement="top"
            width="200">
            <p>确定删除吗,如果存在下级节点则一并删除，此操作不能撤销！</p>
            <div style="text-align: right; margin: 0">
              <el-button size="mini" type="text" @click="scope.row.delPopover = false">取消</el-button>
              <el-button :loading="delLoading" type="primary" size="mini" @click="subDelete(scope.$index, scope.row)">确定</el-button>
            </div>
            <el-button slot="reference" type="danger" size="mini" @click="scope.row.delPopover = true">删除</el-button>
          </el-popover>
        </template>
      </el-table-column>
    </tree-table>
  </div>
</template>

<script>
import checkPermission from '@/utils/permission' // 权限判断函数
import { getRoleTree } from '@/api/role'
import treeTable from '@/components/TreeTable'
import initData from '../../../mixins/initData'
import { del, getMenusTree } from '@/api/menu'
import { parseTime } from '@/utils/index'
import eHeader from './module/header'
import edit from './module/edit'
export default {
  components: { eHeader, edit, treeTable },
  mixins: [initData],
  data() {
    return {
      columns: [
        {
          text: '名称',
          value: 'name'
        }
      ],
      delLoading: false, sup_this: this, menus: [], roles: []
    }
  },
  created() {
    this.getRoles()
    this.getMenus()
    this.$nextTick(() => {
      this.init()
    })
  },
  methods: {
    checkPermission,
    beforeInit() {
      this.url = 'api/menus'
      const sort = 'id,desc'
      const query = this.query
      const value = query.value
      this.params = { page: this.page, size: this.size, sort: sort }
      if (value) { this.params['name'] = value }
      return true
    },
    subDelete(index, row) {
      this.delLoading = true
      del(row.id).then(res => {
        this.delLoading = false
        row.delPopover = false
        this.init()
        this.$notify({
          title: '删除成功',
          type: 'success',
          duration: 2500
        })
      }).catch(err => {
        this.delLoading = false
        row.delPopover = false
        console.log(err.response.data.message)
      })
    },
    time(time) {
      return parseTime(time)
    },
    getMenus() {
      getMenusTree().then(res => {
        this.menus = []
        const menu = { id: 0, label: '顶级类目', children: [] }
        menu.children = res
        this.menus.push(menu)
      })
    },
    getRoles() {
      this.roles = []
      getRoleTree().then(res => {
        this.roles = res
      })
    }
  }
}
</script>

<style scoped>

</style>
