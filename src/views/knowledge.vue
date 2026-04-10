<template>
  <div>
    <PageHead title="知识文章">
      <template #button>
        <el-button @click="handleEdit({})" type="primary">新增</el-button>
      </template>
    </PageHead>
    <TableSearch :formItem="formItem" @search="handleSearsh" />
    <el-table :data="tableData" style="width: 100%; margin-top: 25px;">
      <el-table-column width="200" label="文章标题" fixed="left">
        <template #default="scope">
          <div style="display: flex; align-items: center;">
            <el-icon>
              <timer />
            </el-icon>
            <span>{{scope.row.title}}</span>
          </div>
        </template>
      </el-table-column>
     <el-table-column  label="分类" width="200" >
        <template #default="scope">
          <div style="display: flex; align-items: center;">
            <el-icon>
              <timer />
            </el-icon>
            <span>{{categorymap[scope.row.categoryId]}}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column prop="authorName" label="作者" width="150" />
      <el-table-column prop="readCount" label="阅读量" width="150" />
      <el-table-column prop="updatedAt" label="发布时间" width="150" />
      <el-table-column 
       label="操作"  fixed="right">
        <template #default="scope">
          <div style="display: flex; align-items: center;">
            <el-button text type="primary" @click="handleEdit(scope.row)" >编辑</el-button>
            <el-button @click="handlePublish(scope.row)" v-if="scope.row.status === 0 || scope.row.status === 2" text type="success" >发布</el-button>
            <el-button @click="handleUnpublish(scope.row)" v-if="scope.row.status === 1" text type="warning" >下线</el-button>
            <el-button @click="handleDelete(scope.row)"  text type="danger" >删除</el-button>
          </div>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="margin-top: 25px;"
      layout=" prev, pager, next "
      :total="pagination.total"
      :page-size="pagination.size"
      @change="handleChange"
    />
    <ArticleDialog
      v-model:modelValue="dialogVisible"
      :categories="categories"
      @success="handleSuccess"
      :article="currentArticle"
    />
  </div>
</template>
  <script setup>
  import {onMounted,ref,reactive} from  'vue'
import PageHead from '@/components/PageHead.vue'
import TableSearch from '@/components/TableSearch.vue'
import { categoryTree } from '@/api/admin'
import { articlePage ,getArticleDetail} from '@/api/admin'
import ArticleDialog from '@/components/ArticleDialog.vue'
import { ElMessageBox,ElMessage } from 'element-plus'
import { changeArticleStatus ,deleteArticle} from '@/api/admin'
//新增文章弹窗
const dialogVisible = ref(false)
const currentArticle = ref(null)

const handleSuccess = () => {
  dialogVisible.value = false
  // 刷新列表
  handleSearsh()
}
const handleEdit = (row) => {
  console.log('编辑文章，row.id:', row.id)
if(!row.id){ 
  currentArticle.value = null
  dialogVisible.value = true 
 }
else{
    getArticleDetail(row.id).then(res => {
    console.log('文章详情响应:', res)
    currentArticle.value = res.data.data
    console.log('currentArticle.value:', currentArticle.value)
    console.log('currentArticle.value.id:', currentArticle.value?.id)
    //打开弹窗
    dialogVisible.value = true 
  })
}
}
const handlePublish = (row) => {
  ElMessageBox.confirm('确认发布文章吗？', '确定', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'info'
  }).then(() => {
    // 发布文章逻辑
    changeArticleStatus(row.id,{status:1}).then(res => {
      ElMessage.success('发布成功')
      handleSearsh()
    })
  }).catch(() => {
    // 取消发布
    console.log('取消发布')
  });
}
const handleUnpublish = (row) => {
  ElMessageBox.confirm('确认下线文章吗？', '确定', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    // 下线文章逻辑
    changeArticleStatus(row.id,{status:2}).then(res => {
      ElMessage.success('下线成功')
      handleSearsh()
    })
  }).catch(() => {
    // 取消下线
    console.log('取消下线')
  });
}
const handleDelete = (row) => {
  ElMessageBox.confirm('确认删除文章吗？', '确定', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'danger'
  }).then(() => {
    // 删除文章逻辑
    deleteArticle(row.id).then(res => {
      ElMessage.success('删除成功')
      handleSearsh()
    })
  }).catch(() => {
    // 取消删除
    console.log('取消删除')
 });
}
const pagination = reactive({
  total:0,
 size:10,
  currentPage:1
})
const handleSearsh = async (formData) => {
  console.log(formData,'查询')
    const params = {
      ...formData,
      ...pagination
    }
    const res = await articlePage(params)
  const {records,total}= res.data.data
  console.log(records)
  console.log(total)
  tableData.value = records
  pagination.total = total

}
const handleChange = (val) => {
  pagination.currentPage = val
  handleSearsh()
}

const categorymap = reactive({})
const categories = ref([])
const tableData = ref([])
onMounted(async ()=>{
  const data= await categoryTree()
  console.log(data)
  categories.value = data.data.data.map(item=>{
    categorymap[item.id] = item.categoryName
   return {
    label : item.categoryName,
    value : item.id
  }
})
formItem[1].options = categories.value
//获取列表
handleSearsh()
})
const formItem = [{
  comp:'input',
  prop:'title',
  label:'文章标题',
  placeholder:'请输入文章标题'
},{
  comp:'select',
  prop:'categoryId',
  label:'分类',
  placeholder:'请选择分类'
},
{
  comp:'select',
  prop:'status',
  label:'状态',
  placeholder:'请选择状态',
  options:[
    {
      label:'草稿',
      value:0
    },
    {
      label:'已发布',
      value:1
    },
    {
      label:'已下线',
      value:2
    }
  ]
}]
  </script>