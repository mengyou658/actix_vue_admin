<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryRef"
      :inline="true"
      v-show="showSearch"
      label-width="68px"
    >
      <el-form-item label="字典名称" prop="dict_name">
        <el-input
          v-model="queryParams.dict_name"
          placeholder="请输入字典名称"
          clearable

          style="width: 240px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="字典类型" prop="dict_type">
        <el-input
          v-model="queryParams.dict_type"
          placeholder="请输入字典类型"
          clearable

          style="width: 240px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="字典状态"
          clearable

          style="width: 240px"
        >
          <el-option
            v-for="dict in sys_normal_disable"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker
          v-model="dateRange"

          style="width: 240px"
          value-format="YYYY-MM-DD"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search"   @click="handleQuery"
          >搜索</el-button
        >
        <el-button icon="Refresh"   @click="resetQuery"
          >重置</el-button
        >
      </el-form-item>
    </el-form>

     <el-row :gutter="10" class="mb8" style="height: 35px;">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"

          @click="handleAdd"
          v-hasPermi="['system/dict/type/add']"
          >新增</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"

          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['system/dict/type/edit']"
          >修改</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"

          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['system/dict/type/delete']"
          >删除</el-button
        >
      </el-col>
      <!-- <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"

          @click="handleExport"
          v-hasPermi="['system/dict/type/export']"
          >导出</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Refresh"

          @click="handleRefreshCache"
          v-hasPermi="['system/dict/type/refresh']"
          >刷新缓存</el-button
        >
      </el-col> -->
      <right-toolbar
        v-model:showSearch="showSearch"
        @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="typeList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="字典编号" align="center" prop="dict_type_id" width="100" show-overflow-tooltip/>
      <el-table-column
        label="字典名称"
        align="center"
        prop="dict_name"
        :show-overflow-tooltip="true"
      />
      <el-table-column
        label="字典类型"
        align="center"
        :show-overflow-tooltip="true"
      >
        <template #default="scope">
          <router-link
            :to="'/system/dict-data/index/' + scope.row.dict_type_id"
            class="link-type"
          >
            <span>{{ scope.row.dict_type }}</span>
          </router-link>
        </template>
      </el-table-column>
      <el-table-column label="状态" align="center" prop="status">
        <template #default="scope">
          <dict-tag :options="sys_normal_disable" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column
        label="备注"
        align="center"
        prop="remark"
        :show-overflow-tooltip="true"
      />
      <el-table-column
        label="创建时间"
        align="center"
        prop="created_at"
        width="180"
      >
        <template #default="scope">
          <span>{{ parseTime(scope.row.created_at) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
        v-hasPermi="['system/dict/type/edit', 'system/dict/type/delete']"
      >
        <template #default="scope">
          <el-button

            type="text"
            icon="Edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system/dict/type/edit']"
            >修改</el-button
          >
          <el-button

            type="text"
            icon="Delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system/dict/type/delete']"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      v-model:page="queryParams.page_num"
      v-model:limit="queryParams.page_size"
      @pagination="getList"
    />

    <!-- 添加或修改参数配置对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="dictRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="字典名称" prop="dict_name">
          <el-input v-model="form.dict_name" placeholder="请输入字典名称" />
        </el-form-item>
        <el-form-item label="字典类型" prop="dict_type">
          <el-input v-model="form.dict_type" placeholder="请输入字典类型" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in sys_normal_disable"
              :key="dict.value"
              :label="dict.value"
              >{{ dict.label }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="form.remark"
            type="textarea"
            placeholder="请输入内容"
          ></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Dict">
import { getCurrentInstance,ref,toRefs,reactive } from 'vue';
import {
  listType,
  getType,
  delType,
  addType,
  updateType,
} from '@/api/system/dict/type';

const { proxy } = getCurrentInstance();
const { sys_normal_disable } = proxy.useDict('sys_normal_disable');

const typeList = ref([]);
const open = ref(false);
const loading = ref(false);
const showSearch = ref(true);
const ids = ref([]);
const dict_names = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref('');
const dateRange = ref([]);

const data = reactive({
  form: {},
  queryParams: {
    page_num: 1,
    page_size: 10,
    dict_name: undefined,
    dict_type: undefined,
    status: undefined,
  },
  rules: {
    dict_name: [
      { required: true, message: '字典名称不能为空', trigger: 'blur' },
    ],
    dict_type: [
      { required: true, message: '字典类型不能为空', trigger: 'blur' },
    ],
  },
});

const { queryParams, form, rules } = toRefs(data);

/** 查询字典类型列表 */
async function getList() {
  loading.value = true;
  const response = await listType(proxy.addDateRange(queryParams.value, dateRange.value));
  typeList.value = response.list;
  total.value = response.total;
  loading.value = false;
}
/** 取消按钮 */
function cancel() {
  open.value = false;
  reset();
}
/** 表单重置 */
function reset() {
  form.value = {
    dict_type_id: undefined,
    dict_name: undefined,
    dict_type: undefined,
    status: '1',
    remark: undefined,
  };
  proxy.resetForm('dictRef');
}
/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.page_num = 1;
  getList();
}
/** 重置按钮操作 */
function resetQuery() {
  dateRange.value = [];
  proxy.resetForm('queryRef');
  handleQuery();
}
/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = '添加字典类型';
}
/** 多选框选中数据 */
function handleSelectionChange(selection) {
  ids.value = selection.map((item) => item.dict_type_id);
  dict_names.value = selection.map((item) => item.dict_name);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}
/** 修改按钮操作 */
async function handleUpdate(row) {
  reset();
  const dict_type_id = row.dict_type_id || ids.value;
 const data = await getType({dict_type_id})
    form.value = data;
    open.value = true;
    title.value = '修改字典类型';
}
/** 提交按钮 */
function submitForm() {
  proxy.$refs['dictRef'].validate((valid) => {
    if (valid) {
      if (form.value.dict_type_id != undefined) {
        updateType(form.value).then((response) => {
          proxy.$modal.msgSuccess('修改成功');
          open.value = false;
          getList();
        });
      } else {
        addType(form.value).then((response) => {
          proxy.$modal.msgSuccess('新增成功');
          open.value = false;
          getList();
        });
      }
    }
  });
}
/** 删除按钮操作 */
function handleDelete(row) {
  const dict_type_ids = row.dict_type_id ? [row.dict_type_id] : ids.value;
  const dict_type_names = row.dict_type_id  ? row.dict_name : dict_names.value;
  proxy.$modal
    .confirm('是否确认删除字典编号为"' + dict_type_names + '"的数据项？')
    .then(function () {
      return delType({dict_type_ids});
    })
    .then(() => {
      getList();
      proxy.$modal.msgSuccess('删除成功');
    })
    .catch(() => {});
}
/** 导出按钮操作 */
// function handleExport() {
//   proxy.download(
//     'system/dict/type/export',
//     {
//       ...queryParams.value,
//     },
//     `dict_${new Date().getTime()}.xlsx`
//   );
// }
// /** 刷新缓存按钮操作 */
// function handleRefreshCache() {
//   refreshCache().then(() => {
//     proxy.$modal.msgSuccess('刷新成功');
//   });
// }

getList();
</script>
