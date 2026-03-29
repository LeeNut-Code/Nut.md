<template>
  <div class="editor-wrapper">
    <div class="editor-header">
      <h3>编辑区</h3>
    </div>
    <textarea
      ref="editorRef"
      v-model="localContent"
      class="editor"
      placeholder="在此输入Markdown内容..."
      @input="handleInput"
      @scroll="handleScroll"
    ></textarea>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['update:modelValue', 'scroll'])

const localContent = ref(props.modelValue)
const editorRef = ref(null)

defineExpose({
  editorRef
})

watch(() => props.modelValue, (newValue) => {
  localContent.value = newValue
})

const handleInput = () => {
  emit('update:modelValue', localContent.value)
}

const handleScroll = (event) => {
  emit('scroll', event)
}
</script>

<style scoped>
.editor-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: #fff;
  border-right: 1px solid #e0d4c0;
  transition: all 0.3s ease;
  position: relative;
}

.app.dark-mode .editor-wrapper {
  background-color: #2d2d2d;
  border-right-color: #444;
}

.editor-header {
  padding: 0.8rem 1.2rem;
  border-bottom: 1px solid #e0d4c0;
  background: linear-gradient(135deg, #f8f5f0, #f0e6d2);
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.app.dark-mode .editor-header {
  background: linear-gradient(135deg, #333, #3a3a3a);
  border-bottom-color: #444;
}

.editor-header h3 {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: #8B4513;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.editor-header h3::before {
  content: '✏️';
  font-size: 1.1rem;
}

.app.dark-mode .editor-header h3 {
  color: #ccc;
}

.editor {
  flex: 1;
  border: none;
  outline: none;
  padding: 1.5rem;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.7;
  resize: none;
  background-color: transparent;
  color: inherit;
  transition: all 0.3s ease;
  tab-size: 2;
}

.editor:focus {
  background-color: rgba(255, 255, 255, 0.05);
}

.editor::placeholder {
  color: #999;
  font-style: italic;
}

.app.dark-mode .editor::placeholder {
  color: #666;
}

/* 行号效果 */
.editor-wrapper::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 40px;
  background: linear-gradient(135deg, #f8f5f0, #f0e6d2);
  border-right: 1px solid #e0d4c0;
  pointer-events: none;
}

.app.dark-mode .editor-wrapper::before {
  background: linear-gradient(135deg, #333, #3a3a3a);
  border-right-color: #444;
}

.editor {
  padding-left: 55px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .editor-header {
    padding: 0.6rem 1rem;
  }
  
  .editor-header h3 {
    font-size: 0.9rem;
  }
  
  .editor {
    padding: 1rem;
    padding-left: 45px;
    font-size: 13px;
  }
  
  .editor-wrapper::before {
    width: 35px;
  }
}
</style>