<template>
  <div class="save-manager">
    <button class="save-btn" @click="showModal = true">
      💾 存档管理
    </button>
    
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <div class="modal-header">
          <h3>存档管理</h3>
          <button class="close-btn" @click="showModal = false">✕</button>
        </div>

        <div class="tabs">
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'saves' }"
            @click="activeTab = 'saves'"
          >
            📁 存档
          </button>
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'history' }"
            @click="activeTab = 'history'"
          >
            📅 历史回溯
          </button>
        </div>
        
        <div class="modal-body">
          <div v-if="activeTab === 'saves'" class="tab-content">
            <div class="save-actions">
              <div class="manual-save">
                <input 
                  v-model="newSlotName" 
                  type="text" 
                  placeholder="输入存档名称..."
                  class="slot-input"
                  @keyup.enter="handleSave"
                />
                <button class="action-btn save" @click="handleSave">保存</button>
              </div>
            </div>
            
            <div class="save-slots">
              <h4>存档列表</h4>
              <div v-if="slots.length === 0" class="empty-slots">
                暂无存档
              </div>
              <div v-else class="slots-list">
                <div 
                  v-for="slot in slots" 
                  :key="slot.name" 
                  class="slot-item"
                >
                  <div class="slot-info">
                    <span class="slot-name">{{ slot.name === 'auto' ? '📅 自动存档' : slot.name }}</span>
                    <span class="slot-meta">第 {{ slot.dayCount }} 天 · {{ formatDate(slot.savedAt) }}</span>
                  </div>
                  <div class="slot-actions">
                    <button class="action-btn load" @click="handleLoad(slot.name)">加载</button>
                    <button 
                      v-if="slot.name !== 'auto'" 
                      class="action-btn delete" 
                      @click="handleDelete(slot.name)"
                    >
                      删除
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div v-if="activeTab === 'history'" class="tab-content">
            <div class="history-header">
              <h4>每日存档回溯</h4>
              <p class="history-desc">选择任一天节点，从那天继续生存</p>
            </div>
            
            <div class="save-slots">
              <div v-if="dailySaves.length === 0" class="empty-slots">
                暂无历史存档<br/>
                <span class="empty-tip">随着游戏进行，每天会自动生成一个存档点</span>
              </div>
              <div v-else class="slots-list history-list">
                <div 
                  v-for="save in dailySaves" 
                  :key="save.name" 
                  class="slot-item history-item"
                >
                  <div class="slot-info">
                    <span class="slot-name day-name">🌅 第 {{ save.dayCount }} 天</span>
                    <span class="slot-meta">存档时间：{{ formatDate(save.savedAt) }}</span>
                  </div>
                  <div class="slot-actions">
                    <button class="action-btn load" @click="handleLoad(save.name)">回溯到此天</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  slots: {
    type: Array,
    default: () => []
  },
  dailySaves: {
    type: Array,
    default: () => []
  },
  show: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['save', 'load', 'delete', 'update:show'])

const showModal = ref(props.show)
const newSlotName = ref('')
const activeTab = ref('saves')

watch(() => props.show, (val) => {
  showModal.value = val
})

watch(showModal, (val) => {
  emit('update:show', val)
})

function handleSave() {
  if (newSlotName.value.trim()) {
    emit('save', newSlotName.value.trim())
    newSlotName.value = ''
  }
}

function handleLoad(slotName) {
  emit('load', slotName)
  showModal.value = false
}

function handleDelete(slotName) {
  emit('delete', slotName)
}

function formatDate(timestamp) {
  return new Date(timestamp).toLocaleString('zh-CN', {
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}
</script>

<style scoped>
.save-btn {
  padding: 10px 20px;
  background: linear-gradient(135deg, #3498db, #2980b9);
  border: none;
  border-radius: 10px;
  color: white;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
}

.save-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(52, 152, 219, 0.4);
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
}

.modal-content {
  background: linear-gradient(135deg, #2c3e50, #34495e);
  border-radius: 15px;
  width: 90%;
  max-width: 500px;
  max-height: 80vh;
  overflow: hidden;
  border: 2px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.modal-header h3 {
  color: white;
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  font-size: 20px;
  cursor: pointer;
  padding: 5px 10px;
  border-radius: 5px;
  transition: all 0.2s;
}

.close-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  color: white;
}

.modal-body {
  padding: 20px;
  max-height: 60vh;
  overflow-y: auto;
}

.save-actions {
  margin-bottom: 20px;
}

.manual-save {
  display: flex;
  gap: 10px;
}

.slot-input {
  flex: 1;
  padding: 10px 15px;
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
  background: rgba(0, 0, 0, 0.3);
  color: white;
  font-size: 14px;
  outline: none;
  transition: border-color 0.2s;
}

.slot-input:focus {
  border-color: #3498db;
}

.slot-input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.action-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
}

.action-btn.save {
  background: linear-gradient(135deg, #2ecc71, #27ae60);
  color: white;
}

.action-btn.load {
  background: linear-gradient(135deg, #3498db, #2980b9);
  color: white;
}

.action-btn.delete {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  color: white;
}

.action-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.3);
}

.save-slots h4 {
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 15px;
}

.empty-slots {
  text-align: center;
  color: rgba(255, 255, 255, 0.4);
  padding: 30px;
  font-style: italic;
}

.slots-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.slot-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.slot-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.slot-name {
  color: white;
  font-weight: bold;
}

.slot-meta {
  color: rgba(255, 255, 255, 0.5);
  font-size: 12px;
}

.slot-actions {
  display: flex;
  gap: 8px;
}

.tabs {
  display: flex;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.tab-btn {
  flex: 1;
  padding: 15px 20px;
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
  border-bottom: 2px solid transparent;
}

.tab-btn:hover {
  color: rgba(255, 255, 255, 0.9);
  background: rgba(255, 255, 255, 0.05);
}

.tab-btn.active {
  color: #3498db;
  border-bottom-color: #3498db;
}

.tab-content {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-5px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.history-header {
  margin-bottom: 20px;
}

.history-header h4 {
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 5px;
}

.history-desc {
  color: rgba(255, 255, 255, 0.4);
  font-size: 12px;
  margin: 0;
}

.history-item {
  transition: all 0.2s;
}

.history-item:hover {
  border-color: rgba(52, 152, 219, 0.5);
  background: rgba(52, 152, 219, 0.1);
}

.day-name {
  font-size: 16px;
}

.empty-tip {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.3);
}

.history-list {
  max-height: 300px;
  overflow-y: auto;
}
</style>
