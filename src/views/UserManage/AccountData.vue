<template>
  <div class="account-data">
   <div class="add-box">
      <el-button @click="addAccount" type="primary">新增账户</el-button>
    </div>
     <el-table :data="tableData" border style="width:100%">
       <el-table-column label="角色" width="180">
        <template slot-scope="scope">
          <el-select
            @change="selectChange(scope.row)"
            v-if="scope.row.edit"
            v-model="scope.row.role"
          >
            <el-option
              v-for="option in options"
              :label="option.role"
              :value="option.role"
              :key="option.key"
            ></el-option>
          </el-select>
          <span v-else>{{scope.row.role}}</span>
        </template>
      </el-table-column>

       <el-table-column label="账号" width="180">
        <template slot-scope="scope">
          <el-input v-model="scope.row.username" v-if="scope.row.edit"></el-input>
          <span v-else>{{scope.row.username}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="des" label="描述"></el-table-column>
      <el-table-column label="操作" width="180">
        <template slot-scope="scope" v-if="scope.row.username != 'admin'">
          <el-button
            @click="handleEdit(scope.$index,scope.row)"
            v-if="!scope.row.edit"
            size="mini"
          >编辑</el-button>
          <el-button
            @click="handleSave(scope.$index,scope.row)"
            v-else
            type="success"
            size="mini"
          >完成</el-button>

          <el-button @click="handleDelete(scope.$index,scope.row)" size="mini" type="danger">删除</el-button>
        </template>
      </el-table-column>
       </el-table>
       <AddAccount
        @update="getData"
        @closeDialog="closeDialog"
        :dialogVisible="dialogVisible"
        :options="options"
    ></AddAccount>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Provide } from "vue-property-decorator";
import AddAccount from "./AddAccount.vue";
@Component({
  components:{
    AddAccount
  }
})
export default class AccountData extends Vue {
  @Provide() tableData: any = [];

  @Provide() dialogVisible: Boolean = false;

  @Provide() account: object = {
    key: "",
    role: "",
    username: "",
    des: ""
  };

  // select数据
  @Provide() options: any = [
    {
      key: "admin",
      role: "管理员",
      des: "Super Administrator. Have access to view all pages."
    },
    {
      key: "editor",
      role: "客服",
      des: "Normal Editor. Can see all pages except permission page"
    },
    {
      key: "visitor",
      role: "游客",
      des: "Just a visitor. Can only see the home page and the document page"
    }
  ];
  // 新增
  addAccount() {
     this.dialogVisible = true;
    console.log('新增',this.dialogVisible);
  }
  // 关闭
  closeDialog() {
    this.dialogVisible = false;
  }
 // 编辑
  handleEdit(index: number, row: any): void {
    row.edit = true;
  }
   // 完成
 handleSave(index: number, row: any): void { 
    row.edit = false;
    (this as any).$axios
      .post(`/api/users/editUser/${row._id}`, row)
      .then((res: any) => {
        this.$message({
          message: res.data.msg,
          type: "success"
        });
      });
  }
  handleDelete(index: number, row: any): void {
    // 删除
    (this as any).$axios
      .delete(`/api/users/deleteUser/${row._id}`)
      .then((res: any) => {
        this.$message({
          message: res.data.msg,
          type: "success"
        });

        this.tableData.splice(index, 1);
      });
  }

  //  选择角色
  selectChange(item: any) {
    this.options.map((option: any) => {
      if (option.role == item.role) {
        item.key = option.key;
        item.des = option.des;
      }
    });
  }
   created() {
    this.getData();
  }

 // 添加角色后重新获取数据
  getData() {
    (this as any).$axios("/api/users/allUsers").then((res: any) => {
      // 设置编辑状态
      res.data.datas.forEach((item: any) => {
        item.edit = false;
      });
      this.tableData = res.data.datas;
    });
  }
}
</script>
