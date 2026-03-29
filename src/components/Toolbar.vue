<template>
  <div class="toolbar">
    <div class="toolbar-buttons">
      <button @click="insert('**粗体**', 'inline')" class="toolbar-btn" title="粗体 (Ctrl+B)">
        <span class="btn-icon">𝗕</span>
      </button>
      <button @click="insert('*斜体*', 'inline')" class="toolbar-btn" title="斜体 (Ctrl+I)">
        <span class="btn-icon">𝘪</span>
      </button>
      <button @click="insert('~~删除线~~', 'inline')" class="toolbar-btn" title="删除线">
        <span class="btn-icon">S̶</span>
      </button>
      <button @click="insert('```\n代码块\n```', 'block')" class="toolbar-btn" title="代码块">
        <span class="btn-icon">{ }</span>
      </button>
      <button @click="insert('> 引用', 'block')" class="toolbar-btn" title="引用">
        <span class="btn-icon">"</span>
      </button>
      <button @click="insert('![图片](url)', 'inline')" class="toolbar-btn" title="图片">
        <span class="btn-icon">🖼️</span>
      </button>
      <button @click="insert('[]()', 'inline')" class="toolbar-btn" title="链接">
        <span class="btn-icon">🔗</span>
      </button>
      <button @click="insert('\n- 列表项', 'block')" class="toolbar-btn" title="无序列表">
        <span class="btn-icon">•</span>
      </button>
      <button @click="insert('\n1. 列表项', 'list')" class="toolbar-btn" title="有序列表">
        <span class="btn-icon">1.</span>
      </button>
      <div class="toolbar-btn-group">
        <button class="toolbar-btn" title="表格" @click="showTableDialog = !showTableDialog">
          <span class="btn-icon">⊞</span>
        </button>
        <div v-if="showTableDialog" class="table-dialog">
          <div class="dialog-content">
            <div class="dialog-row">
              <label>行数:</label>
              <input type="number" v-model.number="tableRows" min="1" max="10">
            </div>
            <div class="dialog-row">
              <label>列数:</label>
              <input type="number" v-model.number="tableColumns" min="1" max="10">
            </div>
            <div class="dialog-row">
              <label>表头对齐:</label>
              <select v-model="tableAlignment">
                <option value="left">左对齐</option>
                <option value="center">居中</option>
                <option value="right">右对齐</option>
              </select>
            </div>
            <div class="dialog-buttons">
              <button @click="insertTable" class="dialog-btn primary">确定</button>
              <button @click="showTableDialog = false" class="dialog-btn secondary">取消</button>
            </div>
          </div>
        </div>
      </div>
      <button @click="insert('\n---', 'block')" class="toolbar-btn" title="分隔线">
        <span class="btn-icon">─</span>
      </button>
      <button @click="insert('# 标题1', 'block')" class="toolbar-btn" title="标题1">
        <span class="btn-icon">H1</span>
      </button>
      <button @click="insert('## 标题2', 'block')" class="toolbar-btn" title="标题2">
        <span class="btn-icon">H2</span>
      </button>
      <button @click="insert('### 标题3', 'block')" class="toolbar-btn" title="标题3">
        <span class="btn-icon">H3</span>
      </button>
      <div class="toolbar-divider"></div>
      <button @click="$emit('undo')" class="toolbar-btn" title="撤回 (Ctrl+Z)">
        <span class="btn-icon">↶</span>
      </button>
      <button @click="$emit('redo')" class="toolbar-btn" title="恢复 (Ctrl+Y)">
        <span class="btn-icon">↷</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['insert'])

// 表格相关状态
const showTableDialog = ref(false)
const tableRows = ref(2)
const tableColumns = ref(3)
const tableAlignment = ref('left')

const insert = (markdown, type = 'normal') => {
  emit('insert', { markdown, type })
}

// 插入表格
const insertTable = () => {
  let tableMarkdown = '\n'
  
  // 表头
  tableMarkdown += '|'
  for (let i = 0; i < tableColumns.value; i++) {
    tableMarkdown += ` 表头${i+1} |`
  }
  tableMarkdown += '\n'
  
  // 分隔线
  tableMarkdown += '|'
  for (let i = 0; i < tableColumns.value; i++) {
    let separator = ' --- '
    if (tableAlignment.value === 'center') {
      separator = ' :---: '
    } else if (tableAlignment.value === 'right') {
      separator = ' ---: '
    }
    tableMarkdown += separator + '|'
  }
  tableMarkdown += '\n'
  
  // 数据行
  for (let i = 0; i < tableRows.value; i++) {
    tableMarkdown += '|'
    for (let j = 0; j < tableColumns.value; j++) {
      tableMarkdown += ` 内容${i+1}-${j+1} |`
    }
    tableMarkdown += '\n'
  }
  
  insert(tableMarkdown, 'block')
  showTableDialog.value = false
}

// 快捷键支持
const handleKeydown = (e) => {
  // Ctrl+B 粗体
  if (e.ctrlKey && e.key === 'b') {
    e.preventDefault()
    emit('insert', { markdown: '**粗体**', type: 'inline' })
  }
  // Ctrl+I 斜体
  if (e.ctrlKey && e.key === 'i') {
    e.preventDefault()
    emit('insert', { markdown: '*斜体*', type: 'inline' })
  }
}

// 监听全局键盘事件
window.addEventListener('keydown', handleKeydown)

// 组件卸载时移除事件监听
import { onUnmounted } from 'vue'
onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
.toolbar {
  background: linear-gradient(135deg, #f8f5f0, #f0e6d2);
  border-bottom: 1px solid #e0d4c0;
  padding: 0.8rem 1.2rem;
  box-shadow: 0 2px 8px rgba(139, 69, 19, 0.1);
  transition: all 0.3s ease;
  position: relative;
}

.app.dark-mode .toolbar {
  background: linear-gradient(135deg, #2d2d2d, #3a3a3a);
  border-bottom-color: #444;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.toolbar-buttons {
  display: flex;
  gap: 0.6rem;
  flex-wrap: wrap;
  align-items: center;
}

.toolbar-btn {
  padding: 0.6rem 0.8rem;
  border: 1px solid #e0d4c0;
  border-radius: 8px;
  background-color: rgba(255, 255, 255, 0.8);
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
  color: #8B4513;
  min-width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.toolbar-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transition: left 0.5s ease;
}

.toolbar-btn:hover::before {
  left: 100%;
}

.app.dark-mode .toolbar-btn {
  background-color: rgba(255, 255, 255, 0.1);
  border-color: #444;
  color: #e0e0e0;
}

.toolbar-btn:hover {
  background-color: rgba(255, 255, 255, 1);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.2);
}

.app.dark-mode .toolbar-btn:hover {
  background-color: rgba(255, 255, 255, 0.15);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.toolbar-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 6px rgba(139, 69, 19, 0.15);
}

.btn-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  font-weight: 500;
}

.toolbar-divider {
  width: 1px;
  height: 28px;
  background-color: #e0d4c0;
  margin: 0 0.8rem;
  border-radius: 1px;
}

.app.dark-mode .toolbar-divider {
  background-color: #444;
}

.toolbar-btn-group {
  position: relative;
}

.table-dialog {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 0.8rem;
  background: linear-gradient(135deg, #fff, #f9f5f0);
  border: 1px solid #e0d4c0;
  border-radius: 12px;
  padding: 1.2rem;
  box-shadow: 0 8px 24px rgba(139, 69, 19, 0.2);
  z-index: 1000;
  min-width: 220px;
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-10px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.app.dark-mode .table-dialog {
  background: linear-gradient(135deg, #2d2d2d, #3a3a3a);
  border-color: #444;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
}

.dialog-content {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.dialog-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dialog-row label {
  font-size: 0.9rem;
  color: #8B4513;
  font-weight: 500;
}

.app.dark-mode .dialog-row label {
  color: #ccc;
}

.dialog-row input {
  width: 70px;
  padding: 0.5rem;
  border: 1px solid #e0d4c0;
  border-radius: 6px;
  background-color: #fff;
  color: #333;
  font-size: 0.9rem;
  transition: all 0.3s ease;
}

.dialog-row input:focus {
  outline: none;
  border-color: #D2691E;
  box-shadow: 0 0 0 3px rgba(210, 105, 30, 0.1);
}

.app.dark-mode .dialog-row input {
  background-color: #3a3a3a;
  border-color: #444;
  color: #e0e0e0;
}

.app.dark-mode .dialog-row input:focus {
  border-color: #ff9800;
  box-shadow: 0 0 0 3px rgba(255, 152, 0, 0.1);
}

.dialog-buttons {
  display: flex;
  gap: 0.8rem;
  justify-content: flex-end;
  margin-top: 1rem;
}

.dialog-btn {
  padding: 0.5rem 1rem;
  border: 1px solid #e0d4c0;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: all 0.3s ease;
  min-width: 60px;
  text-align: center;
}

.dialog-btn.primary {
  background: linear-gradient(135deg, #8B4513, #D2691E);
  color: white;
  border-color: #8B4513;
}

.dialog-btn.primary:hover {
  background: linear-gradient(135deg, #D2691E, #CD853F);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.3);
}

.dialog-btn.secondary {
  background-color: #f9f5f0;
  color: #8B4513;
  border-color: #e0d4c0;
}

.dialog-btn.secondary:hover {
  background-color: #f0e6d2;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.2);
}

.app.dark-mode .dialog-btn {
  border-color: #444;
}

.app.dark-mode .dialog-btn.primary {
  background: linear-gradient(135deg, #444, #555);
  border-color: #555;
}

.app.dark-mode .dialog-btn.secondary {
  background-color: #3a3a3a;
  color: #e0e0e0;
  border-color: #444;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .toolbar {
    padding: 0.5rem 0.8rem;
  }
  
  .toolbar-btn {
    padding: 0.4rem 0.6rem;
    font-size: 0.8rem;
    min-width: 36px;
    height: 36px;
  }
  
  .toolbar-buttons {
    gap: 0.4rem;
  }
  
  .toolbar-divider {
    margin: 0 0.6rem;
    height: 24px;
  }
  
  .table-dialog {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    right: auto;
    min-width: 200px;
    padding: 1rem;
  }
}
</style>