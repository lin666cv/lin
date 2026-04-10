<template>
  <el-form ref="formRef" :model="formData" :inline="true">
    <el-row :gutter="24">
      <template v-for="item in formItemAttrs" :key="item.prop">
       <el-col v-bind="item.col">
    <el-form-item :label="item.label" :prop="item.prop">
      <component v-model="formData[item.prop]" :is="iscomp(item.comp)" :placeholder="item.placeholder" >
        <template v-if="item.comp ==='select'">
        <el-option label="全部" value="" />
        <el-option v-for="opt in item.options" :key="opt.value" :label="opt.label" :value="opt.value" />
      </template>
      </component>
    </el-form-item>
    </el-col>
    </template>
    </el-row>
    <el-row>
      <el-button type="primary" @click="handleSearsh">查询</el-button>
      <el-button  @click="handleReset(formRef)"> 重置</el-button>
    </el-row>
  </el-form>
</template>
<script setup>
import { ref ,reactive,computed} from 'vue'
import { ElFormItem } from 'element-plus';
const emit = defineEmits(['search'])

const formItemAttrs = computed(() => {
  const formItem = props.formItem
  formItem.forEach(item => {
   item.col ={xs:24,sm:12,md:8,lg:6,xl:6}
  })
  return formItem
})

const formRef = ref(null)

const formData = reactive({})

const iscomp = (comp) => {
  return {
    input: 'el-input',
    select: 'el-select'
  }[comp]
}

const handleSearsh = () => {
  emit('search', formData)
}

const handleReset = (formRef) => {
 if(!formRef){
  return
 }
 formRef.resetFields()
 emit('search', formData)
}

const props = defineProps({
  formItem: {
    type: Array,
    default: () => []
  }
})
</script>
