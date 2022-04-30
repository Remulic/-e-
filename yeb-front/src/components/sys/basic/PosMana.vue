<template>
  <div>
    <div>
       <el-input
        placeholder="请输入职称"
        suffix-icon="el-icon-plus"
        v-model="pos.name"
        @keydown.enter.native="addPosition"
        style="width:300px"
        size="small"
        class="inputMargin">
      </el-input>
      <el-button type="primary" size="small" icon="el-icon-plus" @click="addPosition">添加</el-button>
    </div>
    <div>
      <el-table
        :data="positions"
        border
        stripe
        size="small"
        @selection-change="handleSelectionChange"
        style="width: 70%">
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="id" label="编号" width="55"></el-table-column>
        <el-table-column prop="name" label="职位名称" width="120"></el-table-column>
        <el-table-column prop="createDate" label="创建时间"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" @click="showEditView(scope.$index,scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click="handleDelete(scope.$index,scope.row)">删除</el-button>
          </template>
        </el-table-column>
        
      </el-table>
      <el-button size="small" class="marginTop" type="danger" :disabled="this.multipleSelection.length==0" @click="deleteMany">批量删除</el-button>
    </div>
    <el-dialog title="编辑职位" :visible.sync="dialogVisible" width="30%">
      <div>
        <el-tag>职位名称</el-tag>
        <el-input v-model="updatePos.name" size="small" class="updatePosInput"></el-input>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="dialogVisible = false">取消</el-button>
        <el-button size="small" type="primary" @click="doUpdate">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name:'PosMana',
  data() {
    return {
      pos:{
        name:''
      },
      dialogVisible: false,
      updatePos: {
        name: ''
      },
      positions: [],
      multipleSelection:[]
      
    }
  },
  mounted() {
    this.initPositions()
  },
  methods: {
    addPosition(){//添加
      if (this.pos.name) {
        this.postRequest('/system/basic/pos/',this.pos).then((res) => {
          if (res) {
            this.initPositions();
            this.pos.name=''
          }
        })
      }else{
        this.$message.error("职位名称不可以为空！");
      }
    },
    initPositions() {//遍历数据
        this.getRequest('/system/basic/pos/').then(resp => {
          if (resp) {
            this.positions = resp;
          }
        })
    },
    handleSelectionChange(val){
        this.multipleSelection = val;
      },
    deleteMany(){
       //批量删除职位
        this.$confirm('此操作将永久删除该【' + this.multipleSelection.length + '】条职位, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let ids = '?';
          this.multipleSelection.forEach(item => {
            ids += 'ids=' + item.id + '&';
          })
          this.deleteRequest('/system/basic/pos/' +ids).then(resp => {
            if (resp) {
              this.initPositions();
            }
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
    showEditView(index, data) {
        Object.assign(this.updatePos, data);
        this.updatePos.createDate = '';
        this.dialogVisible = true;
      },
    doUpdate() {
        //修改职位
        this.putRequest('/system/basic/pos/', this.updatePos).then(resp => {
          if (resp) {
            this.initPositions();
            this.updatePos.name = '';
            this.dialogVisible = false;
          }
        })
      },
    handleDelete(index, data) {
        //删除职位
        this.$confirm('此操作将永久删除该【' + data.name + '】职位, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.deleteRequest('/system/basic/pos/' + data.id).then(resp => {
            if (resp) {
              this.initPositions();
            }
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
    },
  }
}
</script>

<style>
  .inputMargin{
    margin-right: 8px;
  }
  .updatePosInput{
    width: 200px;
    margin-left: 10px;
  }
  .marginTop{
    margin-top: 10px;
  }
</style>