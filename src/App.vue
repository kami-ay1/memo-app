<script setup>
import { ref, computed, onMounted } from 'vue'

// 备忘录数据
const memos = ref([])
const newMemo = ref('')
const filter = ref('all') // all, completed, pending

// 添加备忘录
const addMemo = () => {
  if (newMemo.value.trim()) {
    const memo = {
      id: Date.now(),
      content: newMemo.value.trim(),
      completed: false,
      createdAt: new Date().toLocaleString()
    }
    memos.value.push(memo)
    newMemo.value = ''
    saveMemos()
  }
}

// 删除备忘录
const deleteMemo = (id) => {
  memos.value = memos.value.filter(memo => memo.id !== id)
  saveMemos()
}

// 切换完成状态
const toggleMemo = (id) => {
  const memo = memos.value.find(m => m.id === id)
  if (memo) {
    memo.completed = !memo.completed
    saveMemos()
  }
}

// 清除所有已完成
const clearCompleted = () => {
  memos.value = memos.value.filter(memo => !memo.completed)
  saveMemos()
}

// 过滤后的备忘录
const filteredMemos = computed(() => {
  switch (filter.value) {
    case 'completed':
      return memos.value.filter(memo => memo.completed)
    case 'pending':
      return memos.value.filter(memo => !memo.completed)
    default:
      return memos.value
  }
})

// 统计
const stats = computed(() => {
  const total = memos.value.length
  const completed = memos.value.filter(memo => memo.completed).length
  const pending = total - completed
  return { total, completed, pending }
})

// 保存到本地存储
const saveMemos = () => {
  localStorage.setItem('vue-memos', JSON.stringify(memos.value))
}

// 从本地存储加载
const loadMemos = () => {
  const saved = localStorage.getItem('vue-memos')
  if (saved) {
    memos.value = JSON.parse(saved)
  }
}

// 组件挂载时加载数据
onMounted(() => {
  loadMemos()
})

// 处理回车键
const handleKeyPress = (event) => {
  if (event.key === 'Enter') {
    addMemo()
  }
}
</script>

<template>
  <div class="memo-app">
    <header class="header">
      <h1>📝 Vue3 备忘录</h1>
      <p class="subtitle">管理你的日常任务和备忘事项</p>
    </header>

    <!-- 添加新备忘录 -->
    <div class="add-memo">
      <div class="input-container">
        <input 
          v-model="newMemo" 
          @keypress="handleKeyPress"
          type="text" 
          placeholder="输入新的备忘录内容..."
          class="memo-input"
        >
        <button @click="addMemo" class="add-btn" :disabled="!newMemo.trim()">
          ➕ 添加
        </button>
      </div>
    </div>

    <!-- 统计信息 -->
    <div class="stats">
      <div class="stat-item">
        <span class="stat-number">{{ stats.total }}</span>
        <span class="stat-label">总计</span>
      </div>
      <div class="stat-item">
        <span class="stat-number pending">{{ stats.pending }}</span>
        <span class="stat-label">待完成</span>
      </div>
      <div class="stat-item">
        <span class="stat-number completed">{{ stats.completed }}</span>
        <span class="stat-label">已完成</span>
      </div>
    </div>

    <!-- 过滤器 -->
    <div class="filters">
      <button 
        @click="filter = 'all'" 
        :class="{ active: filter === 'all' }"
        class="filter-btn"
      >
        全部
      </button>
      <button 
        @click="filter = 'pending'" 
        :class="{ active: filter === 'pending' }"
        class="filter-btn"
      >
        待完成
      </button>
      <button 
        @click="filter = 'completed'" 
        :class="{ active: filter === 'completed' }"
        class="filter-btn"
      >
        已完成
      </button>
    </div>

    <!-- 备忘录列表 -->
    <div class="memo-list">
      <div v-if="filteredMemos.length === 0" class="empty-state">
        <div class="empty-icon">📋</div>
        <p class="empty-text">
          {{ filter === 'all' ? '还没有备忘录，快来添加一个吧！' : 
             filter === 'pending' ? '没有待完成的备忘录' : 
             '没有已完成的备忘录' }}
        </p>
      </div>
      
      <div 
        v-for="memo in filteredMemos" 
        :key="memo.id" 
        :class="{ completed: memo.completed }"
        class="memo-item"
      >
        <div class="memo-content">
          <input 
            type="checkbox" 
            :checked="memo.completed" 
            @change="toggleMemo(memo.id)"
            class="memo-checkbox"
          >
          <div class="memo-text">
            <p class="memo-content-text" :class="{ 'line-through': memo.completed }">
              {{ memo.content }}
            </p>
            <span class="memo-time">{{ memo.createdAt }}</span>
          </div>
        </div>
        <button @click="deleteMemo(memo.id)" class="delete-btn">
          🗑️
        </button>
      </div>
    </div>

    <!-- 操作按钮 -->
    <div class="actions" v-if="stats.completed > 0">
      <button @click="clearCompleted" class="clear-btn">
        🧹 清除已完成 ({{ stats.completed }})
      </button>
    </div>

    <footer class="footer">
      <p>使用 Vue 3 + Composition API 构建</p>
    </footer>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.memo-app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 30px;
  color: white;
}

.header h1 {
  margin: 0 0 10px 0;
  font-size: 2.5rem;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.subtitle {
  margin: 0;
  opacity: 0.9;
  font-size: 1.1rem;
}

.add-memo {
  margin-bottom: 30px;
}

.input-container {
  display: flex;
  gap: 10px;
  background: white;
  padding: 8px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.memo-input {
  flex: 1;
  border: none;
  padding: 12px 16px;
  font-size: 16px;
  border-radius: 8px;
  outline: none;
  background: #f8f9fa;
}

.memo-input:focus {
  background: white;
  box-shadow: 0 0 0 2px #667eea;
}

.add-btn {
  padding: 12px 20px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: all 0.2s;
}

.add-btn:hover:not(:disabled) {
  background: #5a67d8;
  transform: translateY(-1px);
}

.add-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.stats {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  justify-content: center;
}

.stat-item {
  background: white;
  padding: 15px 20px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  min-width: 80px;
}

.stat-number {
  display: block;
  font-size: 1.8rem;
  font-weight: bold;
  color: #2d3748;
}

.stat-number.pending {
  color: #ed8936;
}

.stat-number.completed {
  color: #48bb78;
}

.stat-label {
  display: block;
  font-size: 0.9rem;
  color: #718096;
  margin-top: 4px;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  justify-content: center;
}

.filter-btn {
  padding: 8px 16px;
  border: 2px solid white;
  background: transparent;
  color: white;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.2s;
  font-weight: 500;
}

.filter-btn:hover {
  background: white;
  color: #667eea;
}

.filter-btn.active {
  background: white;
  color: #667eea;
}

.memo-list {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
  margin-bottom: 20px;
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #718096;
}

.empty-icon {
  font-size: 3rem;
  margin-bottom: 10px;
}

.empty-text {
  margin: 0;
  font-size: 1.1rem;
}

.memo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 15px 0;
  border-bottom: 1px solid #e2e8f0;
  transition: all 0.2s;
}

.memo-item:last-child {
  border-bottom: none;
}

.memo-item:hover {
  background: #f7fafc;
  margin: 0 -20px;
  padding-left: 20px;
  padding-right: 20px;
  border-radius: 8px;
}

.memo-content {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  flex: 1;
}

.memo-checkbox {
  width: 18px;
  height: 18px;
  cursor: pointer;
  margin-top: 2px;
  flex-shrink: 0;
}

.memo-text {
  flex: 1;
}

.memo-content-text {
  margin: 0 0 4px 0;
  font-size: 16px;
  color: #2d3748;
  transition: all 0.2s;
  word-wrap: break-word;
  word-break: break-word;
  white-space: pre-wrap;
  line-height: 1.5;
}

.memo-content-text.line-through {
  text-decoration: line-through;
  color: #a0aec0;
}

.memo-time {
  font-size: 12px;
  color: #a0aec0;
}

.delete-btn {
  background: #fed7d7;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 14px;
  flex-shrink: 0;
  align-self: flex-start;
  margin-top: 2px;
}

.delete-btn:hover {
  background: #feb2b2;
  transform: scale(1.1);
}

.actions {
  text-align: center;
  margin-bottom: 20px;
}

.clear-btn {
  background: #e53e3e;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s;
}

.clear-btn:hover {
  background: #c53030;
  transform: translateY(-1px);
}

.footer {
  text-align: center;
  color: white;
  opacity: 0.8;
  margin-top: 30px;
}

.footer p {
  margin: 0;
  font-size: 14px;
}

.memo-item.completed {
  opacity: 0.7;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .memo-app {
    padding: 15px;
  }
  
  .header h1 {
    font-size: 2rem;
  }
  
  .stats {
    flex-wrap: wrap;
  }
  
  .input-container {
    flex-direction: column;
  }
  
  .add-btn {
    width: 100%;
  }
  
  .filters {
    flex-wrap: wrap;
  }
}

/* 添加按钮点击效果 */
.add-btn:active {
  transform: translateY(-1px) scale(0.98);
}
</style>
