<template>
  <!-- <div style="width:600px">
    <el-input placeholder="请输入部门名称进行搜索..."
              prefix-icon="el-icon-search"
              v-model="filterText">
    </el-input>
     
    <el-tree
      class="filter-tree"
      :data="deps"
      :props="defaultProps"
      :filter-node-method="filterNode"
      :expand-on-click-node="false"
      ref="tree">
      <span class="custom-tree-node" slot-scope="{ node, data }" style="display: flex;justify-content: space-between;width: 100%;">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="primary"
            size="mini"
            class="Btn"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="danger"
            size="mini"
            class="Btn"
            @click="() => remove(data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose">
      <div>
        <table>
          <tr>
            <td>
              <el-tag>上级部门</el-tag>
            </td>
            <td>{{pname}}</td>
          </tr>
          <tr>
            <td>
              <el-tag>部门名称</el-tag>
            </td>
            <td>
              <el-input v-model="dep.name" placeholder="请输入部门名称..."></el-input>
            </td>
          </tr>
        </table>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDep">确 定</el-button>
      </span>
    </el-dialog>
  </div> -->
  <div style="width: 500px">
    <el-input placeholder="请输入部门名称进行搜索..."
              prefix-icon="el-icon-search"
              v-model="filterText">
    </el-input>
    <el-tree
      :data="deps"
      :props="defaultProps"
      :filter-node-method="filterNode"
      ref="tree">
        <span class="custom-tree-node" style="display: flex;justify-content: space-between;width: 100%;"
              slot-scope="{node,data}">
          <span>{{node.label}}</span>
          <span>
            <el-button
              type="primary"
              size="mini"
              class="depBtn"
              @click="()=>showAddDepView(data)">
              添加部门
            </el-button>
            <el-button
              type="danger"
              size="mini"
              class="depBtn"
              @click="()=>deleteDep(data)">
              删除部门
            </el-button>
          </span>
        </span>
    </el-tree>
    <el-dialog
      title="添加部门"
      :visible.sync="dialogVisible"
      width="30%">
      <div>
        <table>
          <tr>
            <td>
              <el-tag>上级部门</el-tag>
            </td>
            <td>{{pname}}</td>
          </tr>
          <tr>
            <td>
              <el-tag>部门名称</el-tag>
            </td>
            <td>
              <el-input v-model="dep.name" placeholder="请输入部门名称..."></el-input>
            </td>
          </tr>
        </table>
      </div>
      <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible= false">取消</el-button>
          <el-button type="primary" @click="doAddDep">确定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "DepMana",
    data() {
      return {
        filterText: '',
        deps: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        },
        dep: {
          name: '',
          parentId: -1
        },
        pname: '',
        dialogVisible: false
      }
    },
    watch: {
      filterText(val) {
        this.$refs.tree.filter(val);
      }
    },
    mounted() {
      this.initDeps();
    },
    methods: {
      initDep() {
        this.dep = {
          name: '',
          parentId: -1
        }
        this.pname = '';
      },
      initDeps() {
        this.getRequest('/system/basic/department/').then(resp => {
          if (resp) {
            this.deps = resp;
          }
        })
      },
      removeDepFromDeps(p, deps, id) {
        for (let i = 0; i < deps.length; i++) {
          let d = deps[i];
          if (d.id == id) {
            deps.splice(i, 1);
            if (deps.length == 0) {
              p.parent = false;
            }
            return;
          } else {
            this.removeDepFromDeps(d, d.children, id);
          }
        }
      },
      addDep2Deps(deps, dep) {
        for (let i = 0; i < deps.length; i++) {
          let d = deps[i];
          if (d.id == dep.parentId) {
            d.children = d.children.concat(dep);
            if (d.children.length > 0) {
              d.parent = true;
            }
            return;
          } else {
            this.addDep2Deps(d.children, dep);
          }
        }
      },
      doAddDep() {
        this.postRequest('/system/basic/department/',
          this.dep).then(resp => {
          if (resp) {
            this.addDep2Deps(this.deps, resp.obj);
            this.dialogVisible = false;
            this.initDep();
          }
        })
      },
      showAddDepView(data) {
        this.pname = data.name;
        this.dep.parentId = data.id;
        this.dialogVisible = true;
      },
      deleteDep(data) {
        if (data.parent) {
          this.$message.error("父部门删除失败！");
        } else {
          this.$confirm('此操作将永久删除该【' + data.name + '】部门, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.deleteRequest('/system/basic/department/' +
              data.id).then(resp => {
              if (resp) {
                this.removeDepFromDeps(null, this.deps, data.id);
              }
            });
          }).catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            });
          });
        }
      },
      filterNode(value, data) {
        if (!value) return true;
        return data.name.indexOf(value) !== -1;
      }
    }
  }
// export default {
//   name:'DepMana',
//   data() {
//     return {
//       filterText:'',
//       deps:[],
//       defaultProps: {
//           children: 'children',
//           label: 'name'
//         },
//        dialogVisible: false,
//        pname:'',
//        dep:{
//          name:'',
//          parentId:''
//        }
//     }
//   },
//   mounted() {
//     this.initDeps()
//   },
//   watch: {
//       filterText(val) {
//         this.$refs.tree.filter(val);
//       }
//     },

//   methods: {
//     filterNode(value, data) {
//         if (!value) return true;
//         return data.name.indexOf(value) !== -1;
//     },
//     initDeps(){
//       this.getRequest('/system/basic/department/').then((res) => {
//         if(res){
//           this.deps=res
//         }
//       })
//     },
//     append(data){
//       this.dialogVisible=true;
//       this.pname=data.name;
//       this.dep.parentId = data.id;
//     },
//     addDep(){
//       this.postRequest('/system/basic/department/',this.dep).then((res) => {
//         if (res) {
//           this.initDeps()
//           this.dialogVisible=false
//           this.initDep()
//         }
//       })
//     },
//     initDep() {
//         this.dep = {
//           name: '',
//           parentId: -1
//         }
//         this.pname = '';
//       },
//     remove(data){
//          this.$confirm('此操作将永久删除【' + data.name + '】职称, 是否继续?', '提示', {
//           confirmButtonText: '确定',
//           cancelButtonText: '取消',
//           type: 'warning'
//         }).then(() => {
//           this.deleteRequest('/system/basic/department/' +
//             data.id).then(resp => {
//             if (resp) {
//               this.initDeps();
//             }
//           });
//         }).catch(() => {
//           this.$message({
//             type: 'info',
//             message: '已取消删除'
//           });
//         });
//     },
//   }
// }
</script>

<style>
  .Btn{
    padding: 2px;
  }
</style>