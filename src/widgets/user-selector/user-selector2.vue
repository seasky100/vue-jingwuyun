<!--选择通知人-->
<template>
  <el-dialog v-draggable="dialogDraggableOptions" :visible.sync="dialog" title="选择通知人" :modal-append-to-body="true" :append-to-body="true" :size="size||'large'" :modal="false" :close-on-click-modal="false" @open="initDialog" :before-close="closeEvent" v-loading="true">
    <div class="selector-wrap">
      <div class="p-title">选择人员</div>
      <div class="tree-box">
        <el-input size="mini" v-model="treeKeyword" icon="search" placeholder="通过单位名称搜索"></el-input>
        <el-tree ref="orgTree" class="cat-tree" v-if="tree.length>0" :data="tree" node-key="id" :props="{label:'name',id:'orgId'}" :highlight-current="true" :default-expand-all="false" :filter-node-method="filterNode" @node-click="nodeClick"></el-tree>
      </div>
      <el-transfer v-model="value" :data="data" :currTreeNode="currUnitNode" :titles="['选择人员','已选人员']" filterable @change="acChange" :filter-method="filterMethod"></el-transfer>
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button type="" @click="closeEvent">取消</el-button>
      <el-button type="primary" @click="submit">确定</el-button>
    </div>
  </el-dialog>
</template>
<style lang="scss" scoped>
  .form-box {
    padding: 0 10px;
    .share-content {
      width: 650px;
      margin: 0 auto 20px;
    }
    .el-form-item {
      margin-bottom: 0;
    }
    .content {
      line-height: 22px;
    }
  }

</style>
<script>
  /*
                       传入参数：dialog 是否显示Dialog
                       * */
  import mixin from '../../utils/mixin'
  import Transfer from '../transfer-selector';
  import { mapState } from 'vuex'
  import { FETCH_USER_SELECTOR_COMPANY_CONTACTS } from '../../store/types'
  export default {
    mixins: [mixin],
    components: { 'el-transfer': Transfer },
    data() {
      return {
        value: [],
        data: [],
        tree: [],
        sysTree: [],
        currUnitNode: {},
        treeKeyword: ''
      }
    },
    props: {
      dialog: {
        type: Boolean,
        default: false
      },
      orgList: {
        type: Array,
        default () {
          return []
        }
      },
      size: {
        type: String,
        default: 'large'
      }
    },
    computed: {
      ...mapState(['userSelector'])
    },
    watch: {
      treeKeyword(val) {
        this.$refs.orgTree.filter(val)
      }
    },
    methods: {
      acChange(a, b, c) {
        console.log(a, b, c)
      },
      initDialog() {
        if (this.orgList.length > 0) {
          this.tree = this.orgList
        } else if (this.tree.length === 0) {
          this.tree = this.sysTree
        }
      },
      filterMethod(keyword, item) {
        return item.label.indexOf(keyword) !== -1
      },
      filterNode(value, data) {
        if (!value) {
          return true
        } else {
          return data.name.indexOf(value) !== -1
        }
      },
      submit() {
        this.$emit('update:value', [...this.data].filter(r => this.value.includes(r.key)))
        this.$emit('close')
      },
      closeEvent() {
        this.$emit('close')
      },
      nodeClick(data, obj, node) {
        console.log(data);
        this.currUnitNode = {
          id: data.orgId,
          label: data.name
        };
        this.dispatch(FETCH_USER_SELECTOR_COMPANY_CONTACTS, {
          orgId: data.orgId
        }).then(() => (this.data = this.userSelector.contacts.map((d, i) => {
          return {
            label: d.xm,
            key: d.userId,
            userId: d.userId,
            orgId: d.orgId,
            tzrid: d.userId
          }
        })))
      }
    },
    created() {
      this.getRuleOrgTree().then(d => {
        this.sysTree = d
      })
    }
  }

</script>
