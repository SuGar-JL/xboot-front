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
                  @on-change="selectBirthday"
                  placeholder="选择出生日期"
                  style="width: 150px"
                ></DatePicker>
              </Form-item>
              <Form-item label="学院" prop="collegeName">
                <Select v-model="studentForm.collegeName" placeholder="请选择学院" style="width: 150px">
                  <Option
                    v-for="(item, i) in dictcollegeName"
                    :key="i"
                    :label="item.value"
                  >{{item.title}}</Option>
                </Select>
              </Form-item>
              <Form-item style="margin-left:-20px;" class="br">
                <Button @click="handleSearch" type="primary" icon="ios-search">搜索</Button>
                <Button @click="handleReset">重置</Button>
              </Form-item>
            </Form>
          </Row>
          <!-- 操作 -->
          <Row class="operation">
            <Button @click="add" type="primary" icon="md-add">添加学生</Button>
            <Button @click="delAll" icon="md-trash">批量删除</Button>
            <Button @click="getDataList" icon="md-refresh">刷新</Button>
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
      <Form ref="studentForm" :model="studentForm" :label-width="70" :rules="studentFormValidate">
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
            @on-change="selectBirthday"
            placeholder="选择出生日期"
            style="width: 150px"
          ></DatePicker>
        </Form-item>
        <Form-item label="头像" prop="avatar">
          <upload-pic-input v-model="studentForm.avatar"></upload-pic-input>
        </Form-item>
        <Form-item label="所属学院" prop="collegeName">
          <Select v-model="studentForm.collegeName" placeholder="请选择学院">
            <Option
              v-for="(item, i) in dictcollegeName"
              :key="i"
              :label="item.value"
              :value="item.title"
            >{{item.title}}</Option>
          </Select>
        </Form-item>
        <FormItem label="年级" prop="gradeName">
          <grade-select-choose @on-change="handleSelectgradeName" ref="gName"></grade-select-choose>
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
  getStudentListData,
  getAllRoleList,
  addStudent,
  editStudent,
  enableStudent,
  disableStudent,
  deleteStudent,
  getAllStudentData,
  importStudentData,
  resetStudentPass,
} from "@/api/index";
import departmentChoose from "@/views/my-components/xboot/department-choose";
import departmentTreeChoose from "@/views/my-components/xboot/department-tree-choose";
import gradeSelectChoose from "@/views/my-components/xboot/grade-select-choose";
import uploadPicInput from "@/views/my-components/xboot/upload-pic-input";
import { StudentColumns, StudentData } from "@/libs/importTemplate";
import { exportColumn } from "./exportColumn";
import excel from "@/libs/excel";

export default {
  name: "Student-manage",
  components: {
    // expandRow,
    departmentChoose,
    departmentTreeChoose,
    gradeSelectChoose,
    uploadPicInput,
    // checkPassword,
  },
  data() {
    return {
      height: 510,
      loading: true, //加载表格

      importLoading: false, //导入数据时的加载
      loadingExport: true, //导出加载
      exportModalVisible: false, //导出的对话框显示
      importModalVisible: false, //导入的对话框显示
      drop: false, //没地方用到
      dropDownContent: "展开", //我应该用不到
      dropDownIcon: "ios-arrow-down", //展开的按钮图标

      selectCount: 0, //选中表格数据数
      selectList: [],
      //搜索用到的字段
      searchForm: {
        name: "",
        sex: "",
        birthday: "asdsa",
        collegeName: "",
        pageNumber: 1,
        pageSize: 10,
        sort: "createTime",
        order: "desc",
      },
      selectDate: null, //选择时间
      modalType: 0, //标记打开的对话框类型（添加/编辑）
      studentModalVisible: false, //对话框是否显示
      modalTitle: "", //对话框标题
      studentForm: {
        //添加/编辑的表单字段
        name: "",
        sex: "",
        birthday: "asdsad",
        collegeName: "dsds",
        gradeName: "",
      },

      //用不到
      StudentRoles: [],
      roleList: [],
      errorPass: "",

      studentFormValidate: {
        //表单验证
        name: [{ required: true, message: "姓名为空", trigger: "blur" }],
        sex: [{ required: true, message: "性别没选", trigger: "blur" }],
        collegeName: [{ required: true, message: "学院没选", trigger: "blur" }],
        gradeName: [{ required: true, message: "年级没选", trigger: "blur" }],
      },
      submitLoading: false, //提交加载
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
          key: "birthday", /////////
          sortable: true,
          sortType: "desc",
          width: 120,
        },
        {
          title: "学院",
          key: "collegeName", ////
          width: 120,
        },
        {
          title: "年级",
          key: "gradeName", ////
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
      tempColumns: StudentColumns,
      tempData: StudentData,
      data: [],
      exportData: [],
      total: 0,
      dictSex: this.$store.state.dict.sex, //性别的数据字典
      dictcollegeName: this.$store.state.dict.college,
    };
  },
  methods: {
    init() {
      this.getStudentList(); /////

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
    handleSelectgradeName(v) {
      this.studentForm.gradeName = v;
    },
    handleSelectDep(v) {
      this.searchForm.departmentId = v;
    },
    changePage(v) {
      this.searchForm.pageNumber = v;
      this.getStudentList();
      this.clearSelectAll();
    },
    changePageSize(v) {
      this.searchForm.pageSize = v;
      this.getStudentList();
    },
    selectBirthday(v) {
      if (v) {
        this.searchForm.birthday = v;
        this.studentForm.birthday = v;
      }
    },
    getStudentList() {
      // 多条件搜索学生列表
      this.loading = true;
      getStudentListData(this.searchForm).then((res) => {
        this.loading = false;
        if (res.success) {
          this.data = res.result.content;
          this.total = res.result.totalElements;
        }
      });
    },
    handleSearch() {
      //
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      
      this.getStudentList();
    },
    handleReset() {
      //搜索表单重置
      this.$refs.searchForm.resetFields();
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      this.selectDate = null;
      this.$refs.dep.clearSelect();
      this.searchForm.departmentId = "";
      // 重新加载数据
      this.getStudentList();
    },
    changeSort(e) {
      //更改排序
      this.searchForm.sort = e.key;
      this.searchForm.order = e.order;
      if (e.order == "normal") {
        this.searchForm.order = "";
      }
      this.getStudentList();
    },
    /////////////////////////
    getRoleList() {
      //获取角色列表
      getAllRoleList().then((res) => {
        if (res.success) {
          this.roleList = res.result;
        }
      });
    },

    handleDropdown(name) {
      if (name == "refresh") {
        //刷新
        this.getStudentList();
      } else if (name == "reset") {
        //重置用户密码
        if (this.selectCount <= 0) {
          this.$Message.warning("您还未选择要重置密码的用户");
          return;
        }
        this.$refs.checkPass.show();
      } else if (name == "exportData") {
        //导出选择的数据
        if (this.selectCount <= 0) {
          this.$Message.warning("您还未选择要导出的数据");
          return;
        }
        this.exportType = "part";
        this.exportModalVisible = true;
        this.exportTitle = "确认导出 " + this.selectCount + " 条数据(付费)";
      } else if (name == "exportAll") {
        //导出所有数据
        this.exportType = "all";
        this.exportModalVisible = true;
        this.exportTitle = "确认导出全部 " + this.total + " 条数据(付费)";
        getAllStudentData().then((res) => {
          if (res.success) {
            this.exportData = res.result;
          }
        });
      } else if (name == "importData") {
        //导入数据
        this.importModalVisible = true;
      }
    },
    resetPass() {
      //重置密码
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
          resetStudentPass({ ids: ids }).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("操作成功");
              this.clearSelectAll();
              this.getStudentList();
            }
          });
        },
      });
    },
    exportCustomData() {
      //导出用户数据
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
      //上传导入前
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
      //下载模板
      let title = [];
      let key = [];
      StudentColumns.forEach((e) => {
        title.push(e.title);
        key.push(e.key);
      });
      const params = {
        title: title,
        key: key,
        data: StudentData,
        autoWidth: true,
        filename: "导入用户数据模版",
      };
      excel.export_array_to_excel(params);
    },
    importData() {
      //导入数据
      this.importLoading = true;
      importStudentData(this.importTableData).then((res) => {
        this.importLoading = false;
        if (res.success) {
          this.importModalVisible = false;
          this.getStudentList();
          this.$Modal.info({
            title: "导入结果",
            content: res.message,
          });
        }
      });
    },
    cancelStudent() {
      //取消学生
      this.studentModalVisible = false;
    },
    submitStudent() {
      //提交学生
      this.$refs.studentForm.validate((valid) => {
        if (valid) {
          if (this.modalType == 0) {
            // 添加学生 避免编辑后传入id
            delete this.studentForm.id;
            this.submitLoading = true;
            addStudent(this.studentForm).then((res) => {
              this.submitLoading = false;
              if (res.success) {
                this.$Message.success("操作成功");
                this.getStudentList();
                this.studentModalVisible = false;
              }
            });
          } else {
            // 编辑
            this.submitLoading = true;
            console.log("id:" + this.studentForm.id);
            editStudent(this.studentForm).then((res) => {
              this.submitLoading = false;
              if (res.success) {
                this.$Message.success("操作成功");
                this.getStudentList();
                this.studentModalVisible = false;
              }
            });
          }
        }
      });
    },
    handleUpload(v) {
      //上传头像
      this.studentForm.avatar = v;
    },
    add() {
      //添加学生
      this.modalType = 0;
      this.modalTitle = "添加学生";
      this.$refs.studentForm.resetFields();
      this.studentModalVisible = true;
    },
    edit(v) {
      //编辑
      this.modalType = 1;
      this.modalTitle = "编辑学生";
      this.$refs.studentForm.resetFields();
      // 转换null为""
      for (let attr in v) {
        if (v[attr] == null) {
          v[attr] = "";
        }
      }
      let str = JSON.stringify(v);
      console.log("str:" +str);
      let data = JSON.parse(str);
      this.studentForm = data;
      console.log("id:" + this.studentForm.id);
      this.selectDate = this.studentForm.birthday;//日期绑定的是selectDate
      this.$refs.gName.setData(data.gradeName);//用于将数据设置到子组件
      this.studentModalVisible = true;
    },
    enable(v) {
      //启用
      this.$Modal.confirm({
        title: "确认启用",
        content: "您确认要启用用户 " + v.name + " ?",
        onOk: () => {
          this.$store.commit("setLoading", true);
          enableStudent(v.id).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("操作成功");
              this.getStudentList();
            }
          });
        },
      });
    },
    disable(v) {
      //禁用
      this.$Modal.confirm({
        title: "确认禁用",
        content: "您确认要禁用用户 " + v.name + " ?",
        onOk: () => {
          this.$store.commit("setLoading", true);
          disableStudent(v.id).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("操作成功");
              this.getStudentList();
            }
          });
        },
      });
    },
    remove(v) {
      //删除
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除用户 " + v.name + " ?",
        onOk: () => {
          this.$store.commit("setLoading", true);
          deleteStudent(v.id).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("删除成功");
              this.getStudentList();
            }
          });
        },
      });
    },
    dropDown() {
      //展开
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
      //显示已选
      this.exportData = e;
      this.selectList = e;
      this.selectCount = e.length;
    },
    clearSelectAll() {
      //取消全选
      this.$refs.table.selectAll(false);
    },
    delAll() {
      //全删
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
          deleteStudent(ids).then((res) => {
            this.$store.commit("setLoading", false);
            if (res.success) {
              this.$Message.success("删除成功");
              this.clearSelectAll();
              this.getStudentList();
            }
          });
        },
      });
    },
  },
  mounted() {
    //挂载
    // 计算高度
    this.height = Number(document.documentElement.clientHeight - 230);
    this.init();
    this.getRoleList();
  },
};
</script>
<style lang="less">
@import "./StudentManage.less";
</style>