<template>
<div>
  <div class="app-toolbar">
    <!-- 查询条件 -->
    <el-form :inline="true" label-position="right" label-width="80px">
      <el-form-item prop="keyword" label="身份证号">
        <el-input v-model="queryForm.zjhm" placeholder="请输入身份证号码"></el-input>
      </el-form-item>
      <el-form-item prop="keyword" label="姓名">
        <el-input v-model="queryForm.xm" placeholder="请输入姓名"></el-input>
      </el-form-item>
      <el-form-item prop="keyword" label="警号">
        <el-input v-model="queryForm.jh" placeholder="请输入警号"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="info" native-type="submit" @click.prevent="query">查询</el-button>
      </el-form-item>
    </el-form>
    <div class="app-search__buttons" style="right:600px;">
      <el-button type="primary" icon="plus" @click="add">新增</el-button>
    </div>
    <div class="app-search__buttons" style="right:490px;">
      <el-button type="primary" icon="star-off" @click="batchEnabled">批量启用</el-button>
    </div>
    <div class="app-search__buttons" style="right:380px;">
      <el-button type="primary" icon="star-on" @click="noBatchEnabled">批量停用</el-button>
    </div>
    <div class="app-search__buttons" style="right:240px;">
      <el-button type="primary" icon="upload" @click="synchronize">同步警员信息</el-button>
    </div>
     <div class="app-search__buttons" style="right:130px;">
      <el-button type="primary" icon="upload" @click="uploadIng" >模板下载
        <form ref="docDownForm" action="/static/白名单数据导入模板.xlsx" method="get" style="display:none;"></form>
      </el-button>
    </div>
     <div class="app-search__buttons">
      <el-button type="primary" icon="upload" @click="updataIng">导入文件</el-button>
    </div>
  </div>

  <!-- 查询结果 -->
  <div class="app-content-box">
    <el-table @cell-mouse-enter="cellMouseEnter" class="app-table" v-loading.body="loading" border :stripe="true" :data="list" @sort-change="sortChange" @selection-change="selectionChange" style="wodth:100%;">
      <el-table-column type="selection" width="50" align="center"></el-table-column>
      <el-table-column type="index" label="序号" align="center" width="50px">
        <template scope="scope">
          {{scope.$index + 1 + pagenav.limit * (pagenav.currentpage - 1)}}
        </template>
</el-table-column>
<el-table-column prop="name" label="姓名" align="center" width="150px" sortable="'custom'"></el-table-column>
<el-table-column prop="gender" label="性别" align="center" width="80px"></el-table-column>
<el-table-column prop="zjhm" label="证件号码" align="center" width="170px"></el-table-column>
<el-table-column prop="cellphoneNum" label="手机号码" align="center" width="120px"></el-table-column>
<el-table-column prop="policeNum" label="警号" align="center"></el-table-column>
<el-table-column prop="position" label="职务" align="center"></el-table-column>
<el-table-column prop="dept" label="单位名称" align="center" sortable="'custom'"></el-table-column>
<el-table-column label="操作">
  <template scope="scope">
    <el-button type="primary" size="mini" icon="icon iconfont icon-brush" @click="startUsing(scope.row)">{{Number(scope.row.isEnable) === 1 ?'停用':'启用'}}</el-button>
    <el-button type="primary" size="mini" icon="icon iconfont icon-brush" @click="edit(scope.row)">修改</el-button>
    <el-button type="danger" size="mini" icon="icon iconfont icon-trash" @click="remove(scope.row)">删除</el-button>
  </template>
</el-table-column>
</el-table>
</div>
<!-- 分页栏 -->
<div class="app-toolbar">
  <pagination :count="count" :pagenav="pagenav" @update:pagenav="val=>pagenav=val"></pagination>
</div>

<!-- 编辑模态窗 -->
<dialog-form :dialog="editDialog" :id="target!==null ? target.id : ''" @close="editDialog=false" @success="reload"></dialog-form>
<!-- 上传模态窗 -->
<up-form :dialog="upDialog" @close="upDialog=false" @success="reload"></up-form>
</div>
</template>
<style scoped>
  .app-toolbar .el-form-item {
    margin-bottom: 0;
  }

  .tree-box {
    max-height: 270px;
    overflow: auto;
  }

</style>
<script>
  import mixin from '../../../utils/mixin'
  import { mapState } from 'vuex'
  import { FETCH_WAITE_LIST, DELETE_WAITE_LIST, UPDATE_WAITE_IS_ENABLE, UPDATE_WAITE_IS_ENABLE_BATCH, SYNCHRONIZE } from '../../../store/types'
  import dialogForm from './dialog-form'
  import upForm from './up-form'
  import pagination from '../../../widgets/pagination/pagination'
  import { tableFill } from '../../../utils/util'

  export default {
    mixins: [mixin],
    components: {
      pagination,
      dialogForm,
      upForm
    },
    data() {
      return {
        ifUpdata: false,
        count: 0,
        list: [],
        editDialog: false,
        upDialog: false,
        dialogTitle: '',
        target: null,
        pagenav: {
          limit: 17,
          currentpage: 1
        },
        queryForm: {
          zjhm: '',
          xm: '',
          jh: '',
          sortField: '',
          sortBy: ''
        },
        checkedSelects: [],
        ids: []
      }
    },
    computed: {
      ...mapState(['waiteList'])
    },
    watch: {
      pagenav: {
        handler() {
          this.query()
        },
        deep: true
      }
    },
    methods: {
       // 模板下载
      uploadIng() {
        if (this.$refs.docDownForm) {
          this.$refs.docDownForm.submit()
          this.ifUpdata = true
        }
      },
      // 导入文件
      updataIng() {
        if (!this.ifUpdata) {
          this.$message({
            type: 'error',
            message: '请先下载模板',
            showClose: true
          })
          return;
        }
        this.upDialog = true;
      },
       // 排序
      sortChange(column) {
        console.log(column);
        if(column.prop == null) {
          this.queryForm.sortField = '';
          this.queryForm.sortBy = '';
        } else {
          this.queryForm.sortField = column.prop;
          if(column.order === 'ascending') {
            this.queryForm.sortBy = 'asc';
          } else {
            this.queryForm.sortBy = 'desc';
          }
        }
        this.query();
      },
      reload() {
        this.editDialog = false
        if (this.pagenav.currentpage === 1) {
          this.query()
        } else {
          this.pagenav.currentpage = 1
        }
      },
      query() {
        this.dispatch(FETCH_WAITE_LIST, {
          zjhm: this.queryForm.zjhm,
          name: this.queryForm.xm,
          policeNum: this.queryForm.jh,
          sortField: this.queryForm.sortField,
          sortBy: this.queryForm.sortBy,
          PageSize: this.pagenav.limit,
          PageIndex: this.pagenav.currentpage
        }).then(() => {
          this.count = this.waiteList.list.DataCount
          this.list = this.waiteList.list.PageData
        })
      },
      add() {
        this.dialogTitle = '新增'
        this.target = null
        this.editDialog = true
      },
      edit(item) {
        this.dialogTitle = '修改'
        this.target = item
        this.editDialog = true
      },
      remove(item) {
        this.$confirm('确定要删除这些选项?', '温馨提示', {
          type: 'warning'
        }).then(() => {
          this.dispatch(DELETE_WAITE_LIST, { id: item.id }).then(() => {
            this.$message({
              type: 'success',
              message: '选项删除成功',
              showClose: true
            })
            this.query()
          })
        })
      },
      startUsing(item) {
        let isQy = item.isEnable
        this.$confirm(isQy === 0 ? '确定要启用此选项' : '确定要停用此选项', '温馨提示', {
          type: 'warning'
        }).then(() => {
          this.dispatch(UPDATE_WAITE_IS_ENABLE, { id: item.id, isEnable: item.isEnable === 1 ? '0' : '1' }).then(() => {
            this.$message({
              type: 'success',
              message: '修改成功',
              showClose: true
            })
            this.query()
          })
        })
      },
      selectionChange(val) {
        this.checkedSelects = val
        this.ids = val.map(val => val['id'])
      },
      batchEnabled() {
        let ids = []
        if (this.checkedSelects.length === 0) {
          this.$message({
            type: 'success',
            message: '请先选择选项',
            showClose: true
          })
          return false
        } else {
          this.checkedSelects.filter(d => { if (d.isEnable === 0) ids.push(d.id) })
          if (ids.sort().toString() === this.ids.sort().toString()) {
            this.dispatch(UPDATE_WAITE_IS_ENABLE_BATCH, { ids: ids.toString(), isEnable: '1' }).then(() => {
              this.$message({
                type: 'success',
                message: '修改成功',
                showClose: true
              })
              this.dis = []
              ids = []
              this.query()
            })
          } else {
            this.$message({
              type: 'success',
              message: '选项中有停用选项',
              showClose: true
            })
            return false
          }
        }
      },
      noBatchEnabled() {
        let ids = []
        if (this.checkedSelects.length === 0) {
          this.$message({
            type: 'success',
            message: '请先选择选项',
            showClose: true
          })
          return false
        } else {
          this.checkedSelects.filter(d => { if (d.isEnable === 1) ids.push(d.id) })
          console.log('dis', ids)
          if (ids.sort().toString() === this.ids.sort().toString()) {
            this.dispatch(UPDATE_WAITE_IS_ENABLE_BATCH, { ids: ids.toString(), isEnable: '0' }).then(() => {
              this.$message({
                type: 'success',
                message: '修改成功',
                showClose: true
              })
              this.ids = []
              ids = []
              this.query()
            })
          } else {
            this.$message({
              type: 'success',
              message: '选项中有启用选项',
              showClose: true
            })
            return false
          }
        }
      },
      synchronize() {
        this.dispatch(SYNCHRONIZE, {}).then(() => {
          this.$message({
            type: 'success',
            message: '同步成功',
            showClose: true
          })
        })
      }
    },
    mounted() {
      this.query()
      tableFill(this.$, false, false, this.$(window).height() - 112)
      window.addEventListener('resize', (e) => {
        tableFill(this.$, false, false, this.$(window).height() - 112)
      })
    },
    created() {}
  }

</script>
