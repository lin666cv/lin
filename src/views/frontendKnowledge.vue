<template>
  <div class="knowledge-container">
    <div class="header-section">
      <div class="header-content">
          <el-image :src="imageUrl" alt="知识库" style="width: 60px; height: 60px;" />
          <h1>知识库</h1>
      </div>

    </div>
    <div class="content">
      <div class="recommend-section">
        <div class="section-title">
          推荐知识
        </div>
         <div class="recommend-list">
          <div v-for="item in recommendList" :key="item.id" class="recommend-item" @click="goToArticle(item.id)">
            <h4 class="title">
              {{item.title}}
            </h4>
            <p class="read-count">
              <el-icon><Histogram /></el-icon>
             阅读量 {{item.readCount}}
            </p>
          </div>
         </div>
      </div>
      <div class="article-list">
        <div v-for="item in articleList" :key="item.id" class="article-item" @click="goToArticle(item.id)">
         <el-image :src="getImageUrl(item.coverImage)"  style="width: 240px; height: 150px;" />
         <div class="info">
          <div class="title">
                <h3>{{item.title}}</h3>
                <el-tag type="primary" Plain >{{item.categoryName}}</el-tag>
         </div>
         <div style="margin-top: 10px;">
          <div class="flex-box">
            <el-icon><Avatar /></el-icon>
            <span>{{item.authorName}}</span>
          </div>
          <div class="flex-box">
            <el-icon><List /></el-icon>
            <span>{{dayjs(item.updatedAt).format('YYYY-MM-DD HH:mm:ss')}}</span>
          </div>
         </div>
         <div style="margin-top: 10px;">
          <div class="flex-box">
            <el-icon><Platform /></el-icon>
            <span>观看人数 {{item.readCount}}</span>
          </div>
         </div>
        </div>
        </div>
      </div>
    </div>
    <div class="pagination-wrapper">
        <el-pagination
          v-model="pagination.currentPage"
          :page-sizes="pagination.size"
          :total="pagination.total"
          layout="prev, pager, next"
          @change="handlePageChange"
        />
      </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive } from 'vue'
import imageUrl from '@/assets/images/book.png'
import { getKnowledgeList } from '@/api/frontend'
import {dayjs} from 'element-plus'
import {useRouter} from 'vue-router'
const router = useRouter()



const recommendList = ref([])
const pagination = reactive({
  currentPage: 1,
  size: 10,
  total: 0
})
const articleList = ref([])
const goToArticle = (id) => {
  router.push(`/frontendKnowledge/article/${id}`)
}



const getPageList = () => {
       const params = {
        sortField: 'publishedAt',
        sortDirection: 'desc',
        ...pagination
      }
      getKnowledgeList(params).then(res => {
        console.log(res)
        articleList.value = res.data.data.records
        pagination.total = res.data.data.total
      })
}

const getImageUrl = (url) => {
  if (url) {
    return 'http://159.75.169.224:1235' + url
  } else {
    return 'http://file.itndedu.com/psychology_ai.png'
  }
}

const handlePageChange = (val) => {
  pagination.currentPage = val
  getPageList()
}

onMounted(() => {
  const params = {
    sortField: 'readCount',
    sortDirection: 'desc',
    currentPage: 1,
    size: 5
  }
  getPageList()
  getKnowledgeList(params).then(res => {
    console.log(res)
    recommendList.value = res.data.data.records
  })
})
</script>

<style scoped lang="scss">
.knowledge-container {
    background: linear-gradient(135deg, #fafbfc 0%, #f7f9fc 50%, #f2f6fa 100%);
    .flex-box {
        display: flex;
        align-items: center;
        span {
            margin-left: 10px;
        }
    }
    .header-section {
        background: linear-gradient(135deg, #f59e0b 0%, #8b5cf6 100%);
        color: white;
        padding: 48px;
        .header-content {
            display: flex;
            align-items: center;
            gap: 12px;
        }
    }
    .content {
        display: flex;
        gap: 20px;
        margin: 0 auto;
        width: 1200px;
        padding: 20px;
        .recommend-section {
            width: 280px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.08);
            padding: 15px;
            height: 400px;
            .section-title {
                font-size: 12;
                font-weight: 600;
                color: #374151;
                margin-bottom: 10px;
                display: flex;
                align-items: center;
                gap: 5px;
            }
            .recommend-list {
                display: flex;
                flex-direction: column;
                gap: 1rem;
                .recommend-item {
                    border-left: 4px solid #f59e0b;
                    padding-left: 10px;
                    cursor: pointer;
                    .read-count {
                        margin-top: 15px;
                        font-size: 12px;
                        color: #6b7280;
                        display: flex;
                        align-items: center;
                        gap: 10px;
                    }
                }
            }
        }
        .article-list {
            flex: 1;
            .article-item {
                background: white;
                border-radius: 12px;
                box-shadow: 0 2px 10px rgba(0, 0, 0, 0.08);
                padding: 15px;
                margin-bottom: 20px;
                display: flex;
                .info {
                    margin-left: 20px;
                    .title {
                        display: flex;
                        align-items: center;
                        gap: 10px;
                    }
                }
            }
        }
    }
    .pagination-wrapper {
        display: flex;
        justify-content: center;
        padding-bottom: 30px;
    }
}</style>