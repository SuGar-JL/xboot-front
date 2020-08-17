<template>
  <div class="search">
    <Row>
      <Col>
        <Card>
          <Row @keydown.enter.native="handleSearch">
            <Form ref="searchForm" :model="searchForm" inline :label-width="60" class="search-form">
              <Form-item label="姓名" prop="name">
                <Input
                  type="text"
                  v-model="searchForm.name"
                  clearable
                  placeholder="请输入姓名"
                  style="width: 150px"
                />
              </Form-item>
              <Form-item label="性别" prop="sex">
                <Select v-model="searchForm.sex" placeholder="请选择" clearable style="width: 150px">
                  <Option v-for="(item, i) in dictSex" :key="i" :value="item.value">{{item.title}}</Option>
                </Select>
              </Form-item>
              <Form-item label="生日">
                <DatePicker
                  v-model="selectDate"
                  type="date"
                  format="yyyy-MM-dd"
                  clearable
                  @on-change="selectDate"
                  placeholder="选择出生日期"
                  style="width: 150px"
                ></DatePicker>
              </Form-item>
              <Form-item label="学院" prop="college">
                <!-- 学院选择组件 -->
                <department-choose @on-change="handleSelectDep" style="width: 150px" ref="dep"></department-choose>
              </Form-item>
              <Form-item style="margin-left:-20px;" class="br">
                <Button @click="handleSearch" type="primary" icon="ios-search">搜索</Button>
                <Button @click="handleReset">重置</Button>
              </Form-item>
            </Form>
          </Row>
          <!-- 操作 -->
          <Row class="operation">
            <Button @click="add" type="primary" icon="md-add">添加用户</Button>
            <Button @click="delAll" icon="md-trash">批量删除</Button>
            <Button @click="getDataList" icon="md-refresh">刷新</Button>
            <!-- <Dropdown @on-click="handleDropdown">
              <Button>
                更多操作
                <Icon type="md-arrow-dropdown" />
              </Button>
              <DropdownMenu slot="list">
                <DropdownItem name="refresh">刷新</DropdownItem>
                <DropdownItem name="reset">重置用户密码</DropdownItem>
                <DropdownItem name="exportData">导出所选数据</DropdownItem>
                <DropdownItem name="exportAll">导出全部数据</DropdownItem>
                <DropdownItem name="importData">导入数据(付费)</DropdownItem>
              </DropdownMenu>
            </Dropdown> -->
          </Row>
          <!-- 选择计数 -->
          <Row>
            <Alert show-icon>
              已选择
              <span class="select-count">{{selectCount}}</span> 项
              <a class="select-clear" @click="clearSelectAll">清空</a>
            </Alert>
          </Row>
          <!-- 学生信息列表 -->
          <Row>
            <Table
              :loading="loading"
              border
              :columns="columns"
              :data="data"
              sortable="custom"
              @on-sort-change="changeSort"
              @on-selection-change="showSelect"
              ref="table"
            ></Table>
          </Row>

          <Row type="flex" justify="end" class="page">
            <Page
              :current="searchForm.pageNumber"
              :total="total"
              :page-size="searchForm.pageSize"
              @on-change="changePage"
              @on-page-size-change="changePageSize"
              :page-size-opts="[10,20,50]"
              size="small"
              show-total
              show-elevator
              show-sizer
            ></Page>
          </Row>
        </Card>
      </Col>
    </Row>
    <!-- 对话框，添加和编辑时用 -->
    <Modal
      :title="modalTitle"
      v-model="studentModalVisible"
      :mask-closable="false"
      :width="500"
      :styles="{top: '30px'}"
    >
      <!-- 添加学生 -->
      <Form ref="studentForm" :model="studentForm" :label-width="60" :rules="studentFormValidate">
        <FormItem label="姓名" prop="name">
          <Input v-model="studentForm.name" autocomplete="off" />
        </FormItem>
        <FormItem label="性别" prop="sex">
          <RadioGroup v-model="studentForm.sex">
            <Radio v-for="(item, i) in dictSex" :key="i" :label="item.value">{{item.title}}</Radio>
          </RadioGroup>
        </FormItem>
        <Form-item label="生日">
          <DatePicker
            v-model="selectDate"
            type="date"
            format="yyyy-MM-dd"
            clearable
            @on-change="selectDate"
            placeholder="选择出生日期"
            style="width: 150px"
          ></DatePicker>
        </Form-item>
        <Form-item label="头像" prop="avatar">
          <upload-pic-input v-model="studentForm.avatar"></upload-pic-input>
        </Form-item>
        <!-- 做组件 -->
        <Form-item label="所属学院">
          <department-tree-choose @on-change="handleSelectDepTree" ref="depTree"></department-tree-choose>
        </Form-item>
        <FormItem label="年级" prop="type">
          <Select v-model="studentForm.type" placeholder="请选择">
            <Option :value="0">2000</Option>
            <Option :value="1">2001</Option>
            <Option :value="2">...</Option>
          </Select>
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="text" @click="cancelStudent">取消</Button>
        <Button type="primary" :loading="submitLoading" @click="submitStudent">提交</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import {
  getUserListData,
  getAllRoleList,
  addUser,
  editUser,
  enableUser,
  disableUser,
  deleteUser,
  getAllUserData,
  importUserData,
  resetUserPass,
} from "@/api/index";
// import expandRow from "./expand.vue";
// import { validateMobile, validatePassword } from "@/libs/validate";
import departmentChoose from "@/views/my-components/xboot/department-choose";
import departmentTreeChoose from "@/views/my-components/xboot/department-tree-choose";
import uploadPicInput from "@/views/my-components/xboot/upload-pic-input";
// import checkPassword from "@/views/my-components/xboot/check-password";
// 模版导入文件表数据
import { userColumns, userData } from "@/libs/importTemplate";
// 指定导出列数据
import { exportColumn } from "./exportColumn";
import excel from "@/libs/excel";

export default {
  name: "user-manage",
  components: {
    // expandRow,
    departmentChoose,
    departmentTreeChoose,
    uploadPicInput,
    // checkPassword,
  },
  data() {
    return {
      height: 510,
      loading: true,//加载表格
      
      importLoading: false,//导入数据时的加载
      loadingExport: true,//导出加载
      exportModalVisible: false,//导出的对话框显示
      importModalVisible: false,//导入的对话框显示
      drop: false,//没地方用到
      dropDownContent: "展开",//我应该用不到
      dropDownIcon: "ios-arrow-down",//展开的按钮图标

      selectCount: 0,//选中表格数据数
      selectList: [],
      searchForm: {
        name: "",
        sex: "",
        birthday: "",
        college: "",
        pageNumber: 1,
        pageSize: 10,
        sort: "createTime",
        order: "desc",
      },
      selectDate: null,//选择时间
      modalType: 0,//标记打开的对话框类型
      studentModalVisible: false,//对话框是否显示
      modalTitle: "",//对话框标题
      studentForm: {//用在添加和编辑时的对话框
        name: "",
        sex: "",
        departmentId: "",
        departmentTitle: "",
      },

      userRoles: [],
      roleList: [],
      errorPass: "",

      studentFormValidate: {//表单验证
        name: [
          { required: true, message: "姓名为空", trigger: "blur" },
        ],
      },
      submitLoading: false,//提交加载
      //表格的数据
      columns: [
        {
          type: "selection",
          width: 60,
          align: "center",
          fixed: "left",
        },
        {
          type: "index",
          width: 60,
          align: "center",
          fixed: "left",
        },
        {
          title: "姓名",
          key: "name",
          minWidth: 100,
          sortable: true,
          fixed: "left",
        },
        {
          title: "头像",
          key: "avatar",
          width: 80,
          align: "center",
          render: (h, params) => {
            return h("Avatar", {
              props: {
                src: params.row.avatar,
              },
            });
          },
        },
        {
          title: "性别",
          key: "sex",
          width: 70,
          align: "center",
        },
        {
          title: "生日",
          key: "createTime",
          sortable: true,
          sortType: "desc",
          width: 120,
        },
        {
          title: "学院",
          key: "departmentTitle",////
          width: 120,
        },
        {
          title: "年级",
          key: "departmentTitle",////
          width: 120,
        },
        {
          title: "创建时间",
          key: "createTime",
          sortable: true,
          sortType: "desc",
          width: 150,
        },
        {
          title: "操作",
          key: "action",
          width: 200,
          align: "center",
          fixed: "right",
          render: (h, params) => {
            let enableOrDisable = "";
            if (params.row.status == 0) {
              enableOrDisable = h(
                "Button",
                {
                  props: {
                    size: "small",
                  },
                  style: {
                    marginRight: "5px",
                  },
                  on: {
                    click: () => {
                      this.disable(params.row);
                    },
                  },
                },
                "禁用"
              );
            } else {
              enableOrDisable = h(
                "Button",
                {
                  props: {
                    type: "success",
                    size: "small",
                  },
                  style: {
                    marginRight: "5px",
                  },
                  on: {
                    click: () => {
                      this.enable(params.row);
                    },
                  },
                },
                "启用"
              );
            }
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small",
                  },
                  style: {
                    marginRight: "5px",
                  },
                  on: {
                    click: () => {
                      this.edit(params.row);
                    },
                  },
                },
                "编辑"
              ),
              enableOrDisable,
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small",
                  },
                  on: {
                    click: () => {
                      this.remove(params.row);
                    },
                  },
                },
                "删除"
              ),
            ]);
          },
        },
      ],
      exportColumns: exportColumn,
      chooseColumns: [],
      filename: "用户数据",
      exportTitle: "确认导出",
      exportType: "",
      importTableData: [],
      importColumns: [],
      uploadfile: {
        name: "",
      },
      tempColumns: userColumns,
      tempData: userData,
      data: [],
      exportData: [],
      total: 0,
      dictSex: this.$store.state.dict.sex,
    };
  },
  methods: {
    init() {
      this.getUserList();
      // 初始化导出列数据
      let array = [];
      this.exportColumns.forEach((e) => {
        // 指定列限制权限
        if (
          !this.getStore("roles").includes("ROLE_ADMIN") &&
          e.key == "mobile"
        ) {
          e.title = "[无权导出] " + e.title;
          e.disabled = true;
        } else {
          e.disabled = false;
        }
        array.push(e.title);
      });
      this.chooseColumns = array;
    },
    handleSelectDepTree(v) {
      this.studentForm.departmentId = v[0];
    },
    handleSelectDep(v) {
      this.searchForm.departmentId = v;
    },
    changePage(v) {
      this.searchForm.pageNumber = v;
      this.getUserList();
      this.clearSelectAll();
    },
    changePageSize(v) {
      this.searchForm.pageSize = v;
      this.getUserList();
    },
    selectDate(v) {
      if (v) {
        this.searchForm.birthday = v;
      }
    },
    getUserList() {
      // 多条件搜索用户列表
      this.loading = true;
      // 避免后台默认值
      if (!this.searchForm.type) {
        this.searchForm.type = "";
      }
      if (!this.searchForm.status) {
        this.searchForm.status = "";
      }
      getUserListData(this.searchForm).then((res) => {
        this.loading = false;
        if (res.success) {
          this.data = res.result.content;
          this.total = res.result.totalElements;
        }
      });
    },
    handleSearch() {
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      this.getUserList();
    },
    handleReset() {
      this.$refs.searchForm.resetFields();
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      this.selectDate = null;
      this.searchForm.startDate = "";
      this.searchForm.endDate = "";
      this.$refs.dep.clearSelect();
      this.searchForm.departmentId = "";
      // 重新加载数据
      this.getUserList();
    },
    changeSort(e) {
      this.searchForm.sort = e.key;
      this.searchForm.order = e.order;
      if (e.order == "normal") {
        this.searchForm.order = "";
      }
      this.getUserList();
    },
    getRoleList() {
      getAllRoleList().then((res) => {
        if (res.success) {
          this.roleList = res.result;
        }
      });
    },
    handleDropdown(name) {
      if (name == "refresh") {
        this.getUserList();
      } else if (name == "reset") {
        if (this.selectCount <= 0) {
          this.$Message.warning("您还未选择要重置密码的用户");
          return;
        }
        this.$refs.checkPass.show();
      } else if (name == "exportData") {
        if (this.selectCount <= 0) {
          this.$Message.warning("您还未选择要导出的数据");
          return;
        }
        this.exportType = "part";
        this.exportModalVisible = true;
        this.exportTitle = "确认导出 " + this.selectCount + " 条数据(付费)";
      } else if (name == "exportAll") {
        this.exportType = "all";
        this.exportModalVisible = true;
        this.exportTitle = "确认导出全部 " + this.total + " 条数据(付费)";
        getAllUserData().then((res) => {
          if (res.success) {
            this.exportData = res.result;
          }
        });
      } else if (name == "importData") {
        this.importModalVisible = true;
      }
    },
    resetPass() {
      this.$Modal.confirm({
        title: "确认重置",
        content:
          "您确认要重置所选的 " +
          this.selectCount +
          " 条用户数据密码为【123456】?",
        onOk: () => {
          let ids = "";
          this.selectList.forEach(function (e) {
            ids += e.id + ",";
          });
          ids = ids.substring(0, ids.length - 1);
          this.$store.commit("setLoading", true);
          resetUserPass({ ids: ids }).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("操作成功");
              this.clearSelectAll();
              this.getUserList();
            }
          });
        },
      });
    },
    exportCustomData() {
      if (this.filename == "") {
        this.filename = "用户数据";
      }
      // 判断勾选导出列
      let array = [];
      this.exportColumns.forEach((e) => {
        this.chooseColumns.forEach((c) => {
          if (e.title == c && !e.disabled) {
            array.push(e);
          }
        });
      });
      this.exportColumns = array;
      this.exportModalVisible = false;
      let title = [];
      let key = [];
      this.exportColumns.forEach((e) => {
        title.push(e.title);
        key.push(e.key);
      });
      const params = {
        title: title,
        key: key,
        data: this.exportData,
        autoWidth: true,
        filename: this.filename,
      };
      excel.export_array_to_excel(params);
    },
    beforeUploadImport(file) {
      this.uploadfile = file;
      const fileExt = file.name.split(".").pop().toLocaleLowerCase();
      if (fileExt == "xlsx" || fileExt == "xls") {
        this.readFile(file);
        this.file = file;
      } else {
        this.$Notice.warning({
          title: "文件类型错误",
          desc:
            "所选文件‘ " +
            file.name +
            " ’不是EXCEL文件，请选择后缀为.xlsx或者.xls的EXCEL文件。",
        });
      }
      return false;
    },
    // 读取文件
    readFile(file) {
      const reader = new FileReader();
      reader.readAsArrayBuffer(file);
      reader.onerror = (e) => {
        this.$Message.error("文件读取出错");
      };
      reader.onload = (e) => {
        this.$Message.success("读取数据成功");
        const data = e.target.result;
        const { header, results } = excel.read(data, "array");
        const tableTitle = header.map((item) => {
          return { title: item, key: item };
        });
        this.importTableData = results;
        this.importColumns = tableTitle;
      };
    },
    downloadTemple() {
      let title = [];
      let key = [];
      userColumns.forEach((e) => {
        title.push(e.title);
        key.push(e.key);
      });
      const params = {
        title: title,
        key: key,
        data: userData,
        autoWidth: true,
        filename: "导入用户数据模版",
      };
      excel.export_array_to_excel(params);
    },
    importData() {
      this.importLoading = true;
      importUserData(this.importTableData).then((res) => {
        this.importLoading = false;
        if (res.success) {
          this.importModalVisible = false;
          this.getUserList();
          this.$Modal.info({
            title: "导入结果",
            content: res.message,
          });
        }
      });
    },
    cancelStudent() {
      this.studentModalVisible = false;
    },
    submitStudent() {
      this.$refs.studentForm.validate((valid) => {
        if (valid) {
          if (this.modalType == 0) {
            // 添加用户 避免编辑后传入id
            delete this.studentForm.id;
            delete this.studentForm.status;
            if (
              this.studentForm.password == "" ||
              this.studentForm.password == undefined
            ) {
              this.errorPass = "密码不能为空";
              return;
            }
            if (this.studentForm.password.length < 6) {
              this.errorPass = "密码长度不得少于6位";
              return;
            }
            this.submitLoading = true;
            addUser(this.studentForm).then((res) => {
              this.submitLoading = false;
              if (res.success) {
                this.$Message.success("操作成功");
                this.getUserList();
                this.studentModalVisible = false;
              }
            });
          } else {
            // 编辑
            this.submitLoading = true;
            editUser(this.studentForm).then((res) => {
              this.submitLoading = false;
              if (res.success) {
                this.$Message.success("操作成功");
                this.getUserList();
                this.studentModalVisible = false;
              }
            });
          }
        }
      });
    },
    handleUpload(v) {
      this.studentForm.avatar = v;
    },
    add() {
      this.modalType = 0;
      this.modalTitle = "添加用户";
      this.$refs.studentForm.resetFields();
      this.studentModalVisible = true;
    },
    edit(v) {
      this.modalType = 1;
      this.modalTitle = "编辑用户";
      this.$refs.studentForm.resetFields();
      // 转换null为""
      for (let attr in v) {
        if (v[attr] == null) {
          v[attr] = "";
        }
      }
      let str = JSON.stringify(v);
      let data = JSON.parse(str);
      this.studentForm = data;
      this.$refs.depTree.setData([data.departmentId], data.departmentTitle);
      let selectRolesId = [];
      this.studentForm.roles.forEach(function (e) {
        selectRolesId.push(e.id);
      });
      this.studentForm.roles = selectRolesId;
      this.studentModalVisible = true;
    },
    enable(v) {
      this.$Modal.confirm({
        title: "确认启用",
        content: "您确认要启用用户 " + v.name + " ?",
        onOk: () => {
          this.$store.commit("setLoading", true);
          enableUser(v.id).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("操作成功");
              this.getUserList();
            }
          });
        },
      });
    },
    disable(v) {
      this.$Modal.confirm({
        title: "确认禁用",
        content: "您确认要禁用用户 " + v.name + " ?",
        onOk: () => {
          this.$store.commit("setLoading", true);
          disableUser(v.id).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("操作成功");
              this.getUserList();
            }
          });
        },
      });
    },
    remove(v) {
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除用户 " + v.name + " ?",
        onOk: () => {
          this.$store.commit("setLoading", true);
          deleteUser(v.id).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("删除成功");
              this.getUserList();
            }
          });
        },
      });
    },
    dropDown() {
      if (this.drop) {
        this.dropDownContent = "展开";
        this.dropDownIcon = "ios-arrow-down";
      } else {
        this.dropDownContent = "收起";
        this.dropDownIcon = "ios-arrow-up";
      }
      this.drop = !this.drop;
    },
    showSelect(e) {
      this.exportData = e;
      this.selectList = e;
      this.selectCount = e.length;
    },
    clearSelectAll() {
      this.$refs.table.selectAll(false);
    },
    delAll() {
      if (this.selectCount <= 0) {
        this.$Message.warning("您还未选择要删除的数据");
        return;
      }
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除所选的 " + this.selectCount + " 条数据?",
        onOk: () => {
          let ids = "";
          this.selectList.forEach(function (e) {
            ids += e.id + ",";
          });
          ids = ids.substring(0, ids.length - 1);
          this.$store.commit("setLoading", true);
          deleteUser(ids).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("删除成功");
              this.clearSelectAll();
              this.getUserList();
            }
          });
        },
      });
    },
  },
  mounted() {
    // 计算高度
    this.height = Number(document.documentElement.clientHeight - 230);
    this.init();
    this.getRoleList();
  },
};
</script>
<style lang="less">
@import "./userManage.less";
</style>