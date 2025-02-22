<script setup lang="ts">
/**
 * 2025/1/22 18:12
 * @author <a href="https://github.com/wzh-devin">devin</a>
 * @description 弹窗组件封装
 * @version 1.0
 * @since 1.0
 */
import { ref, defineProps, reactive, defineEmits } from 'vue'
import type { IModalProps } from '@/components/page/page-modal/modal'

// 定义props
const props = defineProps<IModalProps>()

// 定义事件
const emit = defineEmits(['confirm'])

// 定义内部属性
const dialogVisible = ref(false)
const isNew = ref(true)
const initData: any = {}
const selectOptions = ref<Array<{ label: string; value: number }>>([])

// 初始化表单数据
for (const item of props.modalConfig.formItems) {
  initData[item.prop] = item.initValue ?? ''
}

const formData = reactive<any>(initData)

const modalConfig = props.modalConfig

/**
 * 设置弹窗显示状态
 */
const setModalVisible = (tag: boolean, itemData?: any) => {
  dialogVisible.value = true
  isNew.value = tag
  if (!tag && itemData) {
    Object.assign(formData, itemData)
  } else {
    clearFormData()
  }
}

/**
 * 设置下拉框选项数据
 */
const setSelectData = (data: any) => {
  selectOptions.value = data
}

/**
 * 确认函数
 */
const handleConfirm = async () => {
  try {
    // 触发确认事件，将表单数据传递给父组件
    await emit('confirm', formData)
    dialogVisible.value = false
    clearFormData()
  } catch (error) {
    console.error('确认操作失败:', error)
  }
}

// 重置表单信息
const clearFormData = () => {
  // 关闭时重置表单
  Object.keys(formData).forEach((key) => {
    formData[key] = ''
  })
}

defineExpose({
  setModalVisible,
  setSelectData
})
</script>

<template>
  <el-dialog
    v-model="dialogVisible"
    :title="isNew ? modalConfig.header.newTitle : modalConfig.header.editTitle"
    width="30%"
    :modal-append-to-body="true"
    :append-to-body="true"
    :lock-scroll="false"
    :close-on-click-modal="false"
    destroy-on-close
  >
    <el-form :model="formData" label-width="80px">
      <template v-for="(item, index) in modalConfig.formItems" :key="index">
        <el-form-item :label="item.label" :prop="item.prop">
          <!-- 输入文本 -->
          <template v-if="item.type === 'input'">
            <el-input
              v-model="formData[item.prop]"
              :placeholder="item.placeholder"
            ></el-input>
          </template>
          <template v-if="item.type === 'textarea'">
            <el-input
              type="textarea"
              v-model="formData[item.prop]"
              :placeholder="item.placeholder"
            ></el-input>
          </template>
          <!-- 下拉框 -->
          <template v-if="item.type === 'select'">
            <el-select
              v-model="formData[item.prop]"
              :placeholder="item.placeholder || '请选择'"
              class="w-full"
            >
              <el-option
                v-for="option in selectOptions"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </el-select>
          </template>
        </el-form-item>
      </template>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleConfirm">确认</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<style scoped lang="less">
:deep(.el-dialog) {
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);

  .el-dialog__header {
    margin: 0;
    padding: 15px 20px;
    border-bottom: 1px solid #e5e7eb;
    background: linear-gradient(to right, #f8fafc, #f1f5f9);

    .el-dialog__title {
      font-size: 16px;
      font-weight: 600;
      color: #1e293b;
    }
  }

  .el-dialog__body {
    padding: 20px;
  }

  .el-dialog__footer {
    padding: 10px 20px;
    border-top: 1px solid #e5e7eb;
    background-color: #f8fafc;
  }
}

// 遮罩层样式
:deep(.el-overlay) {
  background-color: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(2px);
}

.w-full {
  width: 100%;
}
</style>
