<template>
    <div>
      <!-- 行内表单 -->
      <el-form
        :inline="true"
        :model="searchMap"
        class="demo-form-inline"
        style="margin-top:20px"
        ref="searchForm"
      >
        <el-form-item prop="cardNum">
          <el-input v-model="searchMap.cardNum" placeholder="会员卡号"></el-input>
        </el-form-item>
        <el-form-item prop="name">
          <el-input v-model="searchMap.name" placeholder="会员姓名"></el-input>
        </el-form-item>
        <el-form-item prop="payType">
          <el-select v-model="searchMap.payType" placeholder="支付类型">
            <el-option
              v-for="option in payTypeOptions"
              :key="option.type"
              :label="option.name"
              :value="option.type"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item prop="birthday">
          <el-date-picker
            value-format="yyyy-MM-dd"
            v-model="searchMap.birthday"
            type="date"
            placeholder="出生日期"
          ></el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="fetchData">查询</el-button>
          <el-button type="primary" @click="handleAdd">新增</el-button>
          <el-button @click="resetForm('searchForm')">重置</el-button>
        </el-form-item>
      </el-form>
  
      <!-- 弹出新增表单 -->
      <el-dialog title="会员编辑" :visible.sync="dialogFormVisible" width="500px">
        <!--width="500px" Dialog 的宽度 -->
        <!-- style="width:400px" input宽度 -->
        <el-form
          ref="pojoForm"
          :model="pojo"
          style="width:400px"
          label-position="right"
          label-width="100px"
          :rules="rules"
        >
          <el-form-item label="会员卡号" prop="cardNum">
            <el-input v-model="pojo.cardNum" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="会员姓名" prop="name">
            <el-input v-model="pojo.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item prop="birthday" label="会员生日">
            <el-date-picker
              value-format="yyyy-MM-dd"
              v-model="pojo.birthday"
              type="date"
              placeholder="会员生日"
            ></el-date-picker>
          </el-form-item>
          <el-form-item label="手机号码" prop="phone">
            <el-input v-model="pojo.phone" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="开卡金额" prop="money">
            <el-input v-model="pojo.money" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="可用积分" prop="integral">
            <el-input v-model="pojo.integral" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item prop="payType" label="支付类型">
            <el-select v-model="pojo.payType" placeholder="支付类型">
              <el-option
                v-for="option in payTypeOptions"
                :key="option.type"
                :label="option.name"
                :value="option.type"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="会员地址" prop="address">
            <el-input type="textarea" v-model="pojo.address" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="pojo.id === null? addData('pojoForm'):updateData('pojoForm')"
          >确 定</el-button>
        </div>
      </el-dialog>
  
      <!-- 表格信息 -->
      <el-table :data="list" height="380" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60px"></el-table-column>
        <el-table-column prop="cardNum" label="会员卡号"></el-table-column>
        <el-table-column prop="name" label="会员姓名"></el-table-column>
        <el-table-column prop="birthday" label="会员生日"></el-table-column>
        <el-table-column prop="phone" label="手机号码"></el-table-column>
        <el-table-column prop="integral" label="可用积分"></el-table-column>
        <el-table-column prop="money" label="开卡金额"></el-table-column>
        <el-table-column prop="payType" label="支付类型">
          <template slot-scope="scope">
            <span>{{scope.row.payType | payTypeFilter}}</span>
          </template>
        </el-table-column>
        <el-table-column prop="address" label="会员地址"></el-table-column>
        <el-table-column label="操作" width="150px">
          <template slot-scope="scope">
            <el-button size="mini" @click="handleEdit(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" @click="handleDelete(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
  
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[10, 20, 50]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>
  </template>
  
  
  <script>
  const payTypeOptions = [
    { type: "1", name: "现金" },
    { type: "2", name: "微信" },
    { type: "3", name: "支付宝" },
    { type: "4", name: "银行卡" }
  ];
  
  import memberApi from "@/api/member";
  
  export default {
    data() {
      return {
        list: [],
        total: 0, //总条数
        currentPage: 1, //当前页，默认1
        pageSize: 10, //每页条数
        searchMap: {
          cardNum: "",
          name: "",
          payType: "",
          birthday: ""
        }, //条件查询绑定条件值
        dialogFormVisible: false,
        payTypeOptions,
        pojo: {
          cardNum: "",
          name: "",
          birthday: "",
          phone: "",
          money: 0.0,
          integral: 0.0,
          payType: "",
          address: "",
          id: null
        },
        rules: {
          cardNum: [{ required: true, message: "卡号不能为空", trigger: "blur" }],
          name: [{ required: true, message: "姓名不能为空", trigger: "blur" }],
          payType: [
            { required: true, message: "请选择支付类型", trigger: "change" }
          ]
        }
      };
    },
  
    created() {
      this.fetchData();
    },
  
    methods: {
      // 弹出新增窗口
      handleAdd() {
        this.dialogFormVisible = true;
        this.$nextTick(() => {
          this.$refs["pojoForm"].resetFields();
        });
      },
  
      // 提交新增数据
      addData(formName) {
        this.$refs[formName].validate(valid => {
          if (valid) {
            memberApi.add(this.pojo).then(response => {
              const resp = response.data;
              if (resp.flag) {
                this.fetchData();
                this.dialogFormVisible = false;
              } else {
                this.$message({
                  message: "提交失败",
                  type: "warning"
                });
              }
            });
          } else {
            return false;
          }
        });
      },
  
      // 重置行内表单
      resetForm(formName) {
        this.$refs[formName].resetFields();
      },
  
      // 当前页显示条数改变
      handleSizeChange(val) {
        this.pageSize = val;
        this.fetchData();
      },
  
      // 页码改变
      handleCurrentChange(val) {
        this.currentPage = val;
        this.fetchData();
      },
  
      // 获取数据
      fetchData() {
        //   memberApi.getList().then(response => {
        memberApi
          .search(this.currentPage, this.pageSize, this.searchMap)
          .then(response => {
            const resp = response.data;
            this.list = resp.data.rows;
            this.total = resp.data.total;
          });
      },
  
      // 编辑查询数据
      handleEdit(id) {
        this.handleAdd();
        memberApi.getById(id).then(response => {
          const resp = response.data;
          this.pojo = resp.data;
        });
      },
  
      // 编辑更新数据
      updateData(formName) {
        this.$refs[formName].validate(valid => {
          if (valid) {
            memberApi.update(this.pojo).then(response => {
              const resp = response.data;
              if (resp.flag) {
                this.fetchData();
                this.dialogFormVisible = false;
              } else {
                this.$message({
                  message: "更新失败",
                  type: "warning"
                });
              }
            });
          } else {
            return false;
          }
        });
      },
  
      // 删除数据
      handleDelete(id) {
        this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            memberApi.deleteById(id).then(response => {
              const resp = response.data;
              this.fetchData();
              this.$message({
                type: resp.flag ? "success" : "error",
                message: resp.message
              });
            });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消删除"
            });
          });
      }
    },
  
    filters: {
      payTypeFilter(type) {
        const payObj = payTypeOptions.find(obj => obj.type === type);
        return payObj ? payObj.name : null;
      }
    }
  };
  </script>