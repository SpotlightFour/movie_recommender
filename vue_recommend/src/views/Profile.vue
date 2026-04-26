<template>
  <div class="user-profile">
    <h2>我的画像</h2>

    <!-- 添加全局解释区域 -->
    <el-alert 
      title="偏好分数说明" 
      type="info" 
      :closable="false"
      class="explanation-alert"
    >
      <div class="explanation-content">
        <p>偏好分数反映了您对不同电影类型、导演和演员的喜爱程度：</p>
        <ul>
          <li><el-tag type="success">非常高 (80%-100%)</el-tag>：表示您非常喜欢这类元素</li>
          <li><el-tag type="primary">较高 (60%-80%)</el-tag>：表示您对这类元素有较强兴趣</li>
          <li><el-tag type="warning">中等 (40%-60%)</el-tag>：表示您对这类元素兴趣一般</li>
          <li><el-tag type="info">较低 (20%-40%)</el-tag>：表示您对这类元素兴趣较低</li>
          <li><el-tag type="danger">非常低 (0%-20%)</el-tag>：表示您对这类元素兴趣很低</li>
        </ul>
        <p>分数基于您的历史评分、观看行为和偏好设置计算得出。</p>
      </div>
    </el-alert>

    <el-card class="profile-card" v-loading="loading">
      <h3>偏好分析</h3>

      <div class="preference-section">
        <h4>最喜欢的类型</h4>
        <div class="genres" v-if="Object.keys(profile.favorite_genres).length">
          <el-tooltip 
            v-for="(score, genre) in profile.favorite_genres" 
            :key="genre" 
            effect="light" 
            placement="top"
          >
            <template #content>
              <div class="tooltip-content">
                <p>您对<span class="highlight">{{ genre }}</span>的偏好程度为<span class="highlight">{{ (score * 20).toFixed(0) }}%</span></p>
                <p v-if="score > 4">您非常喜欢这类电影，系统会优先推荐相关影片</p>
                <p v-else-if="score > 3">您对这类电影有较强兴趣</p>
                <p v-else-if="score > 2">您对这类电影兴趣一般</p>
                <p v-else-if="score > 1">您对这类电影兴趣较低</p>
                <p v-else>您对这类电影兴趣很低</p>
              </div>
            </template>
            <el-tag :type="getTagType(score)">
              {{ genre }} ({{ (score * 20).toFixed(0) }}%)
            </el-tag>
          </el-tooltip>
        </div>
        <p v-else class="no-data">暂无数据</p>
      </div>

      <div class="preference-section">
        <h4>最喜欢的导演</h4>
        <div class="directors" v-if="Object.keys(profile.preferred_directors).length">
          <el-tooltip 
            v-for="(score, director) in profile.preferred_directors" 
            :key="director" 
            effect="light" 
            placement="top"
          >
            <template #content>
              <div class="tooltip-content">
                <p>您对导演<span class="highlight">{{ director }}</span>的偏好程度为<span class="highlight">{{ (score * 20).toFixed(0) }}%</span></p>
                <p v-if="score > 4">您非常喜欢这位导演的作品</p>
                <p v-else-if="score > 3">您对这位导演的作品有较强兴趣</p>
                <p v-else-if="score > 2">您对这位导演的作品兴趣一般</p>
                <p v-else-if="score > 1">您对这位导演的作品兴趣较低</p>
                <p v-else>您对这位导演的作品兴趣很低</p>
              </div>
            </template>
            <el-tag :type="getTagType(score)">
              {{ director }} ({{ (score * 20).toFixed(0) }}%)
            </el-tag>
          </el-tooltip>
        </div>
        <p v-else class="no-data">暂无数据</p>
      </div>

      <div class="preference-section">
        <h4>最喜欢的演员</h4>
        <div class="actors" v-if="Object.keys(profile.preferred_actors).length">
          <el-tooltip 
            v-for="(score, actor) in profile.preferred_actors" 
            :key="actor" 
            effect="light" 
            placement="top"
          >
            <template #content>
              <div class="tooltip-content">
                <p>您对演员<span class="highlight">{{ actor }}</span>的偏好程度为<span class="highlight">{{ (score * 20).toFixed(0) }}%</span></p>
                <p v-if="score > 4">您非常喜欢这位演员出演的电影</p>
                <p v-else-if="score > 3">您对这位演员出演的电影有较强兴趣</p>
                <p v-else-if="score > 2">您对这位演员出演的电影兴趣一般</p>
                <p v-else-if="score > 1">您对这位演员出演的电影兴趣较低</p>
                <p v-else>您对这位演员出演的电影兴趣很低</p>
              </div>
            </template>
            <el-tag :type="getTagType(score)">
              {{ actor }} ({{ (score * 20).toFixed(0) }}%)
            </el-tag>
          </el-tooltip>
        </div>
        <p v-else class="no-data">暂无数据</p>
      </div>

      <div class="preference-section">
        <h4>观影时间模式</h4>
        <el-tooltip effect="light" placement="top">
          <template #content>
            <p>根据您的观影记录分析出的观影时间偏好</p>
          </template>
          <p>{{ profile.watch_time_pattern ? watchTimeLabels[profile.watch_time_pattern] : '暂无数据' }}</p>
        </el-tooltip>
      </div>

      <div class="preference-section">
        <h4>偏好的年代</h4>
        <el-tooltip effect="light" placement="top">
          <template #content>
            <p>根据您评分的电影分析出的年代偏好</p>
          </template>
          <p>{{ profile.preferred_decade || '暂无数据' }}</p>
        </el-tooltip>
      </div>
    </el-card>

    <el-button type="primary" @click="updateProfile" :loading="updating">
      {{ updating ? '更新中...' : '更新画像' }}
    </el-button>
    
    <!-- 添加更新说明 -->
    <el-alert 
      v-if="updating" 
      title="更新说明" 
      type="info" 
      :closable="false"
      class="update-explanation"
    >
      <p>更新用户画像可能需要一些时间，系统将重新分析您的：</p>
      <ul>
        <li>偏好设置</li>
        <li>评分记录</li>
        <li>观看行为</li>
      </ul>
      <p>完成后将显示最新的偏好分析结果</p>
    </el-alert>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import { ElMessage } from 'element-plus'

export default {
  data() {
    return {
      profile: {
        favorite_genres: {},
        preferred_directors: {},
        preferred_actors: {},
        watch_time_pattern: '',
        preferred_decade: ''
      },
      watchTimeLabels: {
        'weekday_evening': '工作日晚上',
        'weekday_afternoon': '工作日下午',
        'weekday_morning': '工作日早上',
        'weekend_evening': '周末晚上',
        'weekend_afternoon': '周末下午',
        'weekend_morning': '周末早上'
      },
      loading: false,
      updating: false
    }
  },
  computed: {
    ...mapState(['user'])
  },
  async mounted() {
    await this.fetchProfile()
  },
  methods: {
    async fetchProfile() {
      this.loading = true
      try {
        const response = await this.$http.get(`/profile/${this.user.id}`)
        this.profile = response.data
      } catch (error) {
        ElMessage.error('获取用户画像失败')
        console.error('获取画像错误:', error)
      } finally {
        this.loading = false
      }
    },
    async updateProfile() {
      this.updating = true
      try {
        await this.$http.post(`/profile/update/${this.user.id}`)
        ElMessage.success('用户画像更新成功')
        await this.fetchProfile() // 刷新数据
      } catch (error) {
        ElMessage.error('更新用户画像失败')
        console.error('更新画像错误:', error)
      } finally {
        this.updating = false
      }
    },
    getTagType(score) {
      if (score > 4) return 'success'      // 80%-100%：非常高
      if (score > 3) return 'primary'     // 60%-80%：较高
      if (score > 2) return 'warning'     // 40%-60%：中等
      if (score > 1) return 'info'        // 20%-40%：较低
      return 'danger'                      // 0%-20%：非常低
    }
  }
}
</script>

<style scoped>
.user-profile {
  padding: 20px;
}

.explanation-alert {
  margin-bottom: 20px;
  text-align: left;
}

.explanation-content {
  padding: 10px;
}

.explanation-content ul {
  padding-left: 20px;
  margin: 10px 0;
}

.explanation-content li {
  margin-bottom: 5px;
}

.profile-card {
  margin: 20px 0;
  padding: 20px;
}

.preference-section {
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #f0f0f0;
}

.preference-section:last-child {
  border-bottom: none;
}

.genres,
.directors,
.actors {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.no-data {
  color: #999;
  font-style: italic;
}

/* Tooltip内容样式 */
.tooltip-content {
  text-align: center;
  padding: 5px;
}

.highlight {
  color: #409EFF;
  font-weight: bold;
  margin: 0 3px;
}

.update-explanation {
  margin-top: 20px;
}
</style>