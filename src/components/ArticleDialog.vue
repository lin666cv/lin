<template>
  <el-dialog
    :title="isEdit ? '编辑文章' : '新增文章'"
    v-model="dialogVisible"
    width="50%"
    @close="handleClose"
  >
  <el-form :model="formData" :rules="rules" ref="formRef" label-width="120px">
    <el-form-item label="文章标题" prop="title">
      <el-input v-model="formData.title" placeholder="请输入文章标题"  maxlength="200" show-word-limit clearable />
    </el-form-item>
    <el-form-item label="所属分类" prop="categoryId">
      <el-select v-model="formData.categoryId" placeholder="请选择分类">
        <el-option v-for="item in props.categories" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>
    <el-form-item label="文章摘要" prop="summary">
      <el-input type="textarea" v-model="formData.summary" placeholder="请输入文章摘要（可选）" maxlength="1000" show-word-limit :rows="4" />
    </el-form-item>
    <el-form-item label="文章标签" prop="tags">
      <el-select v-model="formData.tagArray" placeholder="请输入文章标签（用逗号隔开）" multiple filterable allow-create  style="width: 100%;" >
        <el-option v-for="tag in commonTags" :key="tag" :label="tag" :value="tag" />
      </el-select>
    </el-form-item>
    <el-form-item lable="封面图片" >
      <div>
        <el-upload
          class="avatar-uploader"
           action="#"
          :before-upload="beforeAvatarUpload"
          :http-request="handleUploadRequest"
          accept="image/*"
          :show-file-list="false"
        >
          <div v-if="!imgUrl" class="cover-placeholder">
            <p>点击上传封面图片</p>
          </div>
          <img v-else :src="imgUrl" alt="封面图片" class="cover-image" />
        </el-upload>
        <div v-if="imgUrl" class="cover-remove">
          <el-button type="danger" size="mini" @click="handleRemove">
            删除封面图片
          </el-button>
        </div>
      </div>
    </el-form-item>
    <el-form-item label="文章内容" prop="content">
      <RichTextEditor v-model="formData.content" placeholder="请输入文章内容" :maxCharCount="5000"  @change="handleContentChange" 
      @create="handleEditorCreate" min-height="400px"/>
    </el-form-item>
  </el-form>
  <div v-if="btnPreview" >
    <h3>内容预览</h3>
    <div v-html="formData.content">
    </div>
  </div>
  <template #footer>
    <el-button  @click="btnPreview = !btnPreview" >{{btnPreview ? '隐藏预览' : '预览效果'}}</el-button>
    <el-button @click="handleClose">取消</el-button>
    <el-button type="primary" @click="handleSubmit" :loading="loading">{{isEdit ? '确定更新文章' : '确定创建文章'}}</el-button>
  </template>
  </el-dialog>
 </template>
<script setup>
import { ref, reactive,computed,nextTick,watch } from 'vue'
import { ElMessage } from 'element-plus'
import { uploadFile ,createArticle ,updateArticle } from '@/api/admin'
import { fileBaseUrl } from '@/config/index.js'
import RichTextEditor from '@/components/RichTextEditor.vue'

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false
  },
  //分类列表
  categories: {
    type: Array,
    default: () => []
  },
  //文章详情
  article: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['update:modelValue','success'])
const handleClose = () => {
  // 关闭弹窗时，重置表单数据
 formRef.value.resetFields()
 businessId.value = ''
 formData.tagArray =[]
handleRemove()
emit('update:modelValue', false)
}
const dialogVisible = computed({
  get(){
     return props.modelValue
  },
  set(val){
    emit('update:modelValue', val)
  }
})
console.log('ArticleDialog - props.article:', props.article)
const isEdit = computed(() => {
  const result = !!props.article?.id
  console.log('ArticleDialog - isEdit:', result, 'props.article?.id:', props.article?.id)
  return result
})
watch(() => props.article, (newVal) => {
  if (newVal) {
   nextTick(() => {
     Object.assign(formData, newVal)
    businessId = newVal.id
    imgUrl.value = fileBaseUrl + newVal.coverImage
   })
    
  }
})

//表单数据
const formData = reactive({
    "title": "",
    "content": "",
    "coverImage": "",
    "categoryId": 1,
    "summary": "",
    "tags": "",
    "id": ""
})
//表单校验规则
const rules = reactive({
  title: [
    { required: true, message: '请输入文章标题', trigger: 'blur' },
    { max: 200, message: '文章标题最多200个字符', trigger: 'blur' }
  ],
  categoryId: [
    { required: true, message: '请选择分类', trigger: 'change' },

  ],
  content: [
    { required: true, message: '请输入文章内容', trigger: 'blur' },
    { max: 5000, message: '文章内容最多5000个字符', trigger: 'blur' }
  ]
})
const commonTags = [
  '情绪管理', '焦虑', '抑郁', '压力', '睡眠', 
  '冥想', '正念', '放松', '心理健康', '自我成长',
  '人际关系', '工作压力', '学习方法', '生活技巧'
]
const beforeUpload = ()=>{
  
}
const imgUrl = ref('')
const beforeAvatarUpload = (file) => {
  // 验证文件类型
 const isImage = file.type.startsWith('image/')
 const isLt5M = file.size / 1024 * 1024 < 5
 if (!isImage) {
  ElMessage.error('请上传图片文件')
  return false
 }
 if (!isLt5M) {
  ElMessage.error('图片大小不能超过5MB')
  return false
 }
 return true
}
const businessId = ref('')
const handleUploadRequest = async ({file}) =>{
  businessId.value = crypto.randomUUID()

  const fileRes = await uploadFile(file,{
          businessId:businessId.value
        })
        imgUrl.value = fileBaseUrl + fileRes.filePath
        formData.coverImage = fileRes.filePath
}
const handleRemove = () => {
  imgUrl.value = ''
  formData.coverImage = ''
}
const handleContentChange = (content) => {
  formData.content = content.html
}
const editorInstance = ref(null)
const handleEditorCreate = (editor) => {
  editorInstance.value = editor
   if(formData.content && editor) {
    nextTick(() => {
      editor.setHTML(formData.content)
    })
   }
}
const formRef = ref()
const btnPreview = ref(false)
const loading = ref(false)
const handleSubmit = () => {
   formRef.value.validate((valid, fields) =>{
             if(valid){
              loading.value = true
             }
             const submitData = {
               ...formData,
               tags: formData.tagArray.join(',')
             }
             delete submitData.tagArray
              if(!isEdit.value){
                submitData.id = businessId.value
                createArticle(submitData).then(res => {
              console.log(submitData)
               loading.value = false
               emit('success')
             })
              }else{
                updateArticle(props.article.id,submitData).then(res => {
               loading.value = false
               emit('success')
             })
              }
             
   })
    
}
</script>
<style lang="scss" scoped>
.cover-placeholder{
  width: 200px;
  height: 120px;
  flex-direction: column;
 background: #f6f8fa;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #909399;
}
.cover-image{
  width: 200px;
  height: 120px;
  display: block;
}
</style>