<!-- 执行监听器 -->
<template>
  <div class="panel-tab__content">
    <el-table :data="elementListenersList" size="mini" border>
      <el-table-column label="序号" width="50px" type="index" />
      <el-table-column label="监听器名称" min-width="100px" prop="listenerName" />
      <el-table-column label="事件类型" min-width="100px" prop="event" :formatter="formatterImplementEventType" />
      <!-- <el-table-column label="监听器类型" min-width="100px" show-overflow-tooltip :formatter="row => listenerTypeObject[row.listenerType]" /> -->
      <el-table-column label="监听器类型" min-width="100px" show-overflow-tooltip prop="listenerType" :formatter="formatterValueType" />

      <el-table-column label="操作" width="90px">
        <template slot-scope="{ row, $index }">
          <el-button size="mini" type="text" @click="openListenerForm(row, $index)">编辑</el-button>
          <el-divider direction="vertical" />
          <el-button size="mini" type="text" style="color: #ff4d4f" @click="removeListener(row, $index)">移除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="element-drawer__button">
      <el-button size="mini" type="primary" icon="el-icon-plus" @click="openListenerForm(null)">添加监听器</el-button>
    </div>

    <!-- 监听器 编辑/创建 部分 -->
    <el-drawer :visible.sync="listenerFormModelVisible" :size="`${width}px`" append-to-body destroy-on-close>
      <div slot="title" class="listener-form-model-title">
        <span>执行监听器</span>
        <el-button size="mini" type="primary" @click="selectListenerOpen">选择监听器</el-button>
      </div>
      <el-form size="mini" :model="listenerForm" label-width="106px" ref="listenerFormRef" @submit.native.prevent>
        <el-form-item label="监听器名称" prop="listenerName" :rules="{ required: true, trigger: 'blur' }">
          <el-input v-model="listenerForm.listenerName" clearable />
        </el-form-item>
        <el-form-item label="事件类型" prop="event" :rules="{ required: true, trigger: 'blur' }">
          <el-select v-model="listenerForm.event">
            <!-- <el-option label="开始" value="start" />
            <el-option label="结束" value="end" />
            <el-option label="启用" value="enable" /> -->
            <el-option
              v-for="item in dict.implementEventType"
              :key="item.value"
              :label="item.label"
              :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="监听器值类型" prop="listenerType" :rules="{ required: true, trigger: ['blur'] }">
          <el-select v-model="listenerForm.listenerType">
            <!-- <el-option v-for="i in Object.keys(listenerTypeObject)" :key="i" :label="listenerTypeObject[i]" :value="i" /> -->
            <el-option
              v-for="item in dict.valueType"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <!-- <el-form-item
        v-if="listenerForm.listenerType"
        :label="dict.valueType[listenerForm.listenerType-1].label"
        :prop="dict.valueType[listenerForm.listenerType-1].field"
        :key="'listener-' + dict.valueType[listenerForm.listenerType-1].field"
        :rules="{ required: true, trigger: ['blur', 'change']}">
          <el-input v-model="listenerForm[dict.valueType[listenerForm.listenerType-1].field]" clearable>
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-form-item> -->
        <el-form-item
          v-if="listenerForm.listenerType === 'classListener'"
          label="类"
          prop="class"
          key="listener-class"
          :rules="{ required: true, trigger: ['blur', 'change'] }"
        >
          <el-input v-model="listenerForm.class" clearable>
            <el-button slot="append" @click="openExpression" icon="el-icon-search"></el-button>
          </el-input>
        </el-form-item>
        <el-form-item
          v-if="listenerForm.listenerType === 'expressionListener'"
          label="表达式"
          prop="expression"
          key="listener-expression"
          :rules="{ required: true, trigger: ['blur', 'change'] }"
        >
          <el-input v-model="listenerForm.expression" clearable>
            <el-button slot="append" @click="openExpression" icon="el-icon-search"></el-button>
          </el-input>
        </el-form-item>
        <el-form-item
          v-if="listenerForm.listenerType === 'delegateExpressionListener'"
          label="委托表达式"
          prop="delegateExpression"
          key="listener-delegate"
          :rules="{ required: true, trigger: ['blur', 'change'] }"
        >
          <el-input v-model="listenerForm.delegateExpression" clearable>
            <el-button slot="append" @click="openExpression" icon="el-icon-search"></el-button>
          </el-input>
        </el-form-item>
      </el-form>
      <el-divider />
      <p class="listener-filed__title">
        <span><i class="el-icon-menu"></i>注入字段：</span>
        <el-button size="mini" type="primary" @click="openListenerFieldForm(null)">添加字段</el-button>
      </p>
      <el-table :data="fieldsListOfListener" size="mini" max-height="240" border fit style="flex: none">
        <el-table-column label="序号" width="50px" type="index" />
        <el-table-column label="字段名称" min-width="100px" prop="name" />
        <el-table-column label="字段类型" min-width="80px" show-overflow-tooltip prop="fieldType" :formatter="formatterParamType" />
        <el-table-column label="字段值/表达式" min-width="100px" show-overflow-tooltip :formatter="row => row.string || row.expression" />
        <el-table-column label="是否必填" min-width="60px" show-overflow-tooltip prop="required" :formatter="formatterRequired" />
        <el-table-column label="操作" width="100px">
          <template slot-scope="{ row, $index }">
            <el-button size="mini" type="text" @click="openListenerFieldForm(row, $index)">编辑</el-button>
            <el-divider direction="vertical" />
            <el-button size="mini" type="text" style="color: #ff4d4f" @click="removeListenerField(row, $index)">移除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div class="element-drawer__button">
        <el-button size="mini" @click="listenerFormModelVisible = false">取 消</el-button>
        <el-button size="mini" type="primary" @click="saveListenerConfig">保 存</el-button>
      </div>
    </el-drawer>

    <!-- 注入西段 编辑/创建 部分 -->
    <el-dialog title="字段配置" :visible.sync="listenerFieldFormModelVisible" width="600px" append-to-body destroy-on-close>
      <el-form :model="listenerFieldForm" size="mini" label-width="96px" ref="listenerFieldFormRef" style="height: 136px" @submit.native.prevent>
        <el-form-item label="字段名称：" prop="name" :rules="{ required: true, trigger: ['blur', 'change'] }">
          <el-input v-model="listenerFieldForm.name" clearable />
        </el-form-item>
        <el-form-item label="是否必填：" prop="required" :rules="{ required: true, trigger: ['blur', 'change'] }">
          <el-select v-model="listenerFieldForm.required">
            <el-option v-for="item in dict.required" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="字段类型：" prop="fieldType" :rules="{ required: true, trigger: ['blur', 'change'] }">
          <el-select v-model="listenerFieldForm.fieldType">
            <!-- <el-option v-for="i in Object.keys(fieldTypeObject)" :key="i" :label="fieldTypeObject[i]" :value="i" /> -->
            <el-option v-for="item in dict.paramType" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item
          v-if="listenerFieldForm.fieldType != null"
          :label="dict.paramType[listenerFieldForm.fieldType].label"
          :prop="dict.paramType[listenerFieldForm.fieldType].field"
          :key="'field-' + dict.paramType[listenerFieldForm.fieldType].field"
          :rules="{ required: listenerFieldForm.required === 0, trigger: ['blur', 'change'] }"
        >
          <el-input v-model="listenerFieldForm[dict.paramType[listenerFieldForm.fieldType].field]" clearable />
        </el-form-item>
        <!-- <el-form-item
          v-if="listenerFieldForm.fieldType === 'string'"
          label="字段值："
          prop="string"
          key="field-string"
          :rules="{ required: true, trigger: ['blur', 'change'] }"
        >
          <el-input v-model="listenerFieldForm.string" clearable />
        </el-form-item>
        <el-form-item
          v-if="listenerFieldForm.fieldType === 'expression'"
          label="表达式："
          prop="expression"
          key="field-expression"
          :rules="{ required: true, trigger: ['blur', 'change'] }"
        >
          <el-input v-model="listenerFieldForm.expression" clearable />
        </el-form-item> -->
      </el-form>
      <template slot="footer">
        <el-button size="mini" @click="listenerFieldFormModelVisible = false">取 消</el-button>
        <el-button size="mini" type="primary" @click="saveListenerFiled">确 定</el-button>
      </template>
    </el-dialog>
    <!--  选择执行监听器弹窗 -->
    <SelectListenerDialog :open="selectOpen" :listenerType="1" @close="selectOpen=false" @submit="selectListenerSubmit" />
    <!-- 表达式选择弹窗 -->
    <ExpressionDialog :open="expressionOpen" @close="expressionOpen=false" @submit="expressionSelectSubmit" />
  </div>
</template>
<script>
import { createListenerObject, updateElementExtensions } from "../../utils";
import { initListenerType, initListenerForm, listenerType, fieldType } from "./utilSelf";
import SelectListenerDialog from './components/SelectListenerDialog';
import dict from './components/dict'
import { getConfigureAll } from '@/api/processManagement/processListener'
import ExpressionDialog from '../components/expression/expressionDialog.vue'
export default {
  name: "ElementListeners",
  components:{ SelectListenerDialog, ExpressionDialog },
  props: {
    id: String,
    type: String
  },
  mixins: [dict],
  inject: {
    prefix: "prefix",
    width: "width"
  },
  data() {
    return {
      elementListenersList: [], // 监听器列表
      listenerForm: {}, // 监听器详情表单
      listenerFormModelVisible: false, // 监听器 编辑 侧边栏显示状态
      fieldsListOfListener: [],
      listenerFieldForm: {}, // 监听器 注入字段 详情表单
      listenerFieldFormModelVisible: false, // 监听器 注入字段表单弹窗 显示状态
      editingListenerIndex: -1, // 监听器所在下标，-1 为新增
      editingListenerFieldIndex: -1, // 字段所在下标，-1 为新增
      listenerTypeObject: listenerType,
      fieldTypeObject: fieldType,
      selectOpen: false,
      expressionOpen: false,
    };
  },
  watch: {
    id: {
      immediate: true,
      handler(val) {
        val && val.length && this.$nextTick(() => this.resetListenersList());
      }
    }
  },
  methods: {
    resetListenersList() {
      this.bpmnElement = window.bpmnInstances.bpmnElement;
      this.otherExtensionList = [];
      this.bpmnElementListeners =
        this.bpmnElement.businessObject?.extensionElements?.values?.filter(ex => ex.$type === `${this.prefix}:ExecutionListener`) ?? [];
      this.elementListenersList = this.bpmnElementListeners.map(listener => initListenerType(listener));
    },
    // 打开 监听器详情 侧边栏
    openListenerForm(listener, index) {
      if (listener) {
        this.listenerForm = initListenerForm(listener);
        // xml取出的数据是string类型  需要转为number
        this.listenerForm.event && (this.listenerForm.event = Number(this.listenerForm.event))
        this.editingListenerIndex = index;
      } else {
        this.listenerForm = {};
        this.editingListenerIndex = -1; // 标记为新增
      }
      if (listener && listener.fields) {
        this.fieldsListOfListener = listener.fields.map(field => ({ ...field}));
      } else {
        this.fieldsListOfListener = [];
        this.$set(this.listenerForm, "fields", []);
      }
      // 打开侧边栏并清楚验证状态
      this.listenerFormModelVisible = true;
      this.$nextTick(() => {
        if (this.$refs["listenerFormRef"]) this.$refs["listenerFormRef"].clearValidate();
      });
    },
    // 打开监听器字段编辑弹窗
    openListenerFieldForm(field, index) {
      this.listenerFieldForm = field ? JSON.parse(JSON.stringify(field)) : {};
      this.editingListenerFieldIndex = field ? index : -1;
      this.listenerFieldFormModelVisible = true;
      field && (this.listenerFieldForm.required = Number(this.listenerFieldForm.required))
      field && (this.listenerFieldForm.fieldType = Number(this.listenerFieldForm.fieldType))
      this.$nextTick(() => {
        if (this.$refs["listenerFieldFormRef"]) this.$refs["listenerFieldFormRef"].clearValidate();
      });
    },
    // 保存监听器注入字段
    async saveListenerFiled() {
      let validateStatus = await this.$refs["listenerFieldFormRef"].validate();
      if (!validateStatus) return; // 验证不通过直接返回
      if (this.editingListenerFieldIndex === -1) {
        this.fieldsListOfListener.push(this.listenerFieldForm);
        this.listenerForm.fields.push(this.listenerFieldForm);
      } else {
        this.fieldsListOfListener.splice(this.editingListenerFieldIndex, 1, this.listenerFieldForm);
        this.listenerForm.fields.splice(this.editingListenerFieldIndex, 1, this.listenerFieldForm);
      }
      this.listenerFieldFormModelVisible = false;
      this.$nextTick(() => (this.listenerFieldForm = {}));
    },
    // 移除监听器字段
    removeListenerField(field, index) {
      this.$confirm("确认移除该字段吗？", "提示", {
        confirmButtonText: "确 认",
        cancelButtonText: "取 消"
      })
        .then(() => {
          this.fieldsListOfListener.splice(index, 1);
          this.listenerForm.fields.splice(index, 1);
        })
        .catch(() => console.info("操作取消"));
    },
    // 移除监听器
    removeListener(listener, index) {
      this.$confirm("确认移除该监听器吗？", "提示", {
        confirmButtonText: "确 认",
        cancelButtonText: "取 消"
      })
        .then(() => {
          this.bpmnElementListeners.splice(index, 1);
          this.elementListenersList.splice(index, 1);
          this.otherExtensionList = this.bpmnElement.businessObject?.extensionElements?.values?.filter(ex => ex.$type !== `${this.prefix}:ExecutionListener`) ?? [];
          updateElementExtensions(this.bpmnElement, this.otherExtensionList.concat(this.bpmnElementListeners));
        })
        .catch(() => console.info("操作取消"));
    },
    // 保存监听器配置
    async saveListenerConfig() {
      let validateStatus = await this.$refs["listenerFormRef"].validate();
      if (!validateStatus) return; // 验证不通过直接返回
      const listenerObject = createListenerObject(this.listenerForm, false, this.prefix);
      if (this.editingListenerIndex === -1) {
        this.bpmnElementListeners.push(listenerObject);
        this.elementListenersList.push(this.listenerForm);
      } else {
        this.bpmnElementListeners.splice(this.editingListenerIndex, 1, listenerObject);
        this.elementListenersList.splice(this.editingListenerIndex, 1, this.listenerForm);
      }
      // 保存其他配置
      this.otherExtensionList = this.bpmnElement.businessObject?.extensionElements?.values?.filter(ex => ex.$type !== `${this.prefix}:ExecutionListener`) ?? [];
      updateElementExtensions(this.bpmnElement, this.otherExtensionList.concat(this.bpmnElementListeners));
      // 4. 隐藏侧边栏
      this.listenerFormModelVisible = false;
      this.listenerForm = {};
    },
    // 选择监听器
    selectListenerOpen () {
      this.selectOpen = true
    },
    // 选择好的监听器
    selectListenerSubmit (select) {
      this.listenerForm = {
        listenerName: select.listenerName,
        event: select.eventType,
        listenerType: select.valueType,
        [this.dict.valueType.find(item => item.value === select.valueType).field]: select.listenerValue
      }
      this.getParams(select.id)
    },
    // 通过拿到的监听器id获取字段信息
    async getParams (id) {
      const res = await getConfigureAll({listenerId: id});
      this.fieldsListOfListener = res.data.map((item) => {
        return {
          name: item.paramName,
          fieldType: item.paramType,
          [this.dict.paramType[item.paramType].field] : item.paramValue,
          required: item.required
        }
      })
      this.listenerForm.fields = [...this.fieldsListOfListener]
    },
    openExpression () {
      // 开启表达式弹窗
      this.expressionOpen = true;
    },
    expressionSelectSubmit (obj) {
      this.expressionOpen = false;
      switch (this.listenerForm.listenerType) {
        case 'classListener':
          this.listenerForm.class = obj.expressionValue;
          break;
        case 'expressionListener':
          this.listenerForm.expression = obj.expressionValue;
          break;
        case 'delegateExpressionListener':
          this.listenerForm.delegateExpression = obj.expressionValue;
          break;
      }
    }
  }
};
</script>
<style lang="scss" scoped>
.listener-form-model-title{
  display: flex;
  justify-content: space-between;
}

</style>
