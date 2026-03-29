<template>
  <div class="app" :class="{ 'dark-mode': isDarkMode }">
    <header class="app-header">
      <div class="brand">
        <div class="brand-icon">🌰</div>
        <h1 class="artistic-title">
          <span class="nut-text">NUT</span>
          <span class="md-text">.md</span>
        </h1>
      </div>
      <div class="header-actions">
        <button @click="exportHtml" class="export-btn">导出HTML</button>
        <button @click="copyContent" class="export-btn">复制内容</button>
        <button @click="toggleTheme" class="theme-toggle">
          {{ isDarkMode ? '🌞' : '🌙' }}
        </button>
      </div>
    </header>
    
    <Toolbar @insert="insertMarkdown" @undo="undo" @redo="redo" />
    
    <div class="editor-container">
      <Editor 
        ref="editorRef"
        v-model="content" 
        @update:modelValue="handleContentChange"
        @scroll="handleEditorScroll"
      />
      <Preview 
        :content="content" 
        ref="previewRef"
        @scroll="handlePreviewScroll"
      />
    </div>
    

  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import Editor from './components/Editor.vue'
import Preview from './components/Preview.vue'
import Toolbar from './components/Toolbar.vue'

// 状态管理
const content = ref('')
const isDarkMode = ref(false)
const previewRef = ref(null)
const editorRef = ref(null)

// 历史记录管理
const history = ref([''])
const historyIndex = ref(0)
const MAX_HISTORY = 50
let historyTimer = null

// 初始化
onMounted(() => {
  loadContentFromStorage()
  loadThemeFromStorage()
  setupEventListeners()
})

// 组件卸载时清理资源
onUnmounted(() => {
  cleanupEventListeners()
  cleanupHistory()
})

// 从本地存储加载内容
const loadContentFromStorage = () => {
  const savedContent = localStorage.getItem('markdown-content')
  if (savedContent) {
    content.value = savedContent
  }
}

// 从本地存储加载主题设置
const loadThemeFromStorage = () => {
  const savedTheme = localStorage.getItem('dark-mode')
  if (savedTheme) {
    isDarkMode.value = savedTheme === 'true'
  }
}

// 设置事件监听器
const setupEventListeners = () => {
  window.addEventListener('keydown', handleKeydown)
}

// 清理事件监听器
const cleanupEventListeners = () => {
  window.removeEventListener('keydown', handleKeydown)
  clearTimeout(historyTimer)
}

// 清理历史记录
const cleanupHistory = () => {
  history.value = []
  historyIndex.value = 0
}

// 内容变化处理
const handleContentChange = (value) => {
  // 自动保存到本地存储
  localStorage.setItem('markdown-content', value)
  
  // 防抖处理历史记录
  clearTimeout(historyTimer)
  historyTimer = setTimeout(() => {
    updateHistory(value)
  }, 300)
}

// 更新历史记录
const updateHistory = (value) => {
  if (value !== history.value[historyIndex.value]) {
    // 移除当前索引之后的历史记录
    history.value = history.value.slice(0, historyIndex.value + 1)
    // 添加新的历史记录
    history.value.push(value)
    // 限制历史记录长度
    if (history.value.length > MAX_HISTORY) {
      history.value.shift()
    } else {
      historyIndex.value++
    }
  }
}

// 主题切换
const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  localStorage.setItem('dark-mode', isDarkMode.value)
}

// Markdown配置对象
const markdownConfig = {
  inline: {
    '**粗体**': {
      wrap: (text) => `**${text}**`,
      placeholder: '****',
      cursorOffset: 2
    },
    '*斜体*': {
      wrap: (text) => `*${text}*`,
      placeholder: '**',
      cursorOffset: 1
    },
    '~~删除线~~': {
      wrap: (text) => `~~${text}~~`,
      placeholder: '~~~~',
      cursorOffset: 2
    },
    '![图片](url)': {
      wrap: (text) => `![${text}]()`,
      placeholder: '![]()',
      cursorOffset: 2
    },
    '[]()': {
      wrap: (text) => `[${text}]()`,
      placeholder: '[]()',
      cursorOffset: 1
    }
  },
  block: {
    '```\n代码块\n```': {
      wrap: (text, start, end) => {
        const beforeSelection = content.value.substring(0, start)
        const afterSelection = content.value.substring(end)
        const isInMiddleOfParagraph = (beforeSelection.trim() !== '' || afterSelection.trim() !== '')
        
        if (isInMiddleOfParagraph) {
          return `\n\`\`\`\n${text}\n\`\`\`\n`
        } else {
          return `\n\`\`\`\n${text}\n\`\`\``
        }
      },
      placeholder: '\n\`\`\`\n\`\`\`',
      cursorOffset: 4
    },
    '> 引用': {
      wrap: (text) => {
        const lines = text.split('\n')
        const quotedLines = lines.map(line => `> ${line}`).join('\n')
        return `\n${quotedLines}`
      }
    }
  }
}

// 处理选中文本插入
const handleSelectionInsert = (editorElement, start, end, selectedText, currentLine, markdown, type) => {
  const config = markdownConfig[type]?.[markdown]
  
  if (selectedText) {
    if (type === 'inline' && config) {
      insertInlineWithSelection(editorElement, start, end, selectedText, config)
    } else if (type === 'block' && config) {
      insertBlockWithSelection(editorElement, start, end, selectedText, config)
    } else if (type === 'list') {
      insertListWithSelection(editorElement, start, selectedText, markdown)
    }
  } else {
    if (type === 'inline' && config) {
      insertInlineWithoutSelection(editorElement, start, config)
    } else if (type === 'block') {
      if (markdown.includes('代码块') && config) {
        insertCodeBlockWithoutSelection(editorElement, start, config)
      } else if (markdown.startsWith('#')) {
        insertHeadingWithoutSelection(editorElement, start, currentLine, markdown)
      }
    } else if (type === 'list') {
      insertListWithoutSelection(editorElement, start, markdown)
    }
  }
}

// 插入带选择的内联元素
const insertInlineWithSelection = (editorElement, start, end, selectedText, config) => {
  const wrappedText = config.wrap(selectedText)
  content.value = content.value.substring(0, start) + wrappedText + content.value.substring(end)
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(start + wrappedText.length, start + wrappedText.length)
  }, 0)
}

// 插入带选择的块元素
const insertBlockWithSelection = (editorElement, start, end, selectedText, config) => {
  const wrappedText = config.wrap(selectedText, start, end)
  content.value = content.value.substring(0, start) + wrappedText + content.value.substring(end)
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(start + wrappedText.length, start + wrappedText.length)
  }, 0)
}

// 插入带选择的列表
const insertListWithSelection = (editorElement, start, selectedText, markdown) => {
  const lineStart = content.value.lastIndexOf('\n', start) + 1
  const lineContent = content.value.substring(lineStart, start)
  const prefix = markdown.includes('有序') ? '1. ' : '- '
  const newContent = content.value.substring(0, lineStart) + prefix + lineContent + content.value.substring(start)
  content.value = newContent
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(lineStart + prefix.length + lineContent.length, lineStart + prefix.length + lineContent.length)
  }, 0)
}

// 插入不带选择的内联元素
const insertInlineWithoutSelection = (editorElement, start, config) => {
  const placeholder = config.placeholder
  content.value = content.value.substring(0, start) + placeholder + content.value.substring(start)
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(start + config.cursorOffset, start + config.cursorOffset)
  }, 0)
}

// 插入不带选择的代码块
const insertCodeBlockWithoutSelection = (editorElement, start, config) => {
  const placeholder = config.placeholder
  content.value = content.value.substring(0, start) + placeholder + content.value.substring(start)
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(start + config.cursorOffset, start + config.cursorOffset)
  }, 0)
}

// 插入不带选择的标题
const insertHeadingWithoutSelection = (editorElement, start, currentLine, markdown) => {
  if (currentLine && currentLine.trim() !== '') {
    markdown = '\n' + markdown
  }
  content.value = content.value.substring(0, start) + markdown + content.value.substring(start)
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(start + markdown.length, start + markdown.length)
  }, 0)
}

// 插入不带选择的列表
const insertListWithoutSelection = (editorElement, start, markdown) => {
  const lineStart = content.value.lastIndexOf('\n', start) + 1
  const lineContent = content.value.substring(lineStart, start)
  const prefix = markdown.includes('有序') ? '1. ' : '- '
  const newContent = content.value.substring(0, lineStart) + prefix + lineContent + content.value.substring(start)
  content.value = newContent
  handleContentChange(content.value)
  
  setTimeout(() => {
    editorElement.focus()
    editorElement.setSelectionRange(lineStart + prefix.length + lineContent.length, lineStart + prefix.length + lineContent.length)
  }, 0)
}

// 插入Markdown
const insertMarkdown = (data) => {
  const { markdown, type } = data
  
  // 处理选中文本
  const editorElement = editorRef.value?.editorRef
  if (editorElement) {
    const start = editorElement.selectionStart
    const end = editorElement.selectionEnd
    const selectedText = content.value.substring(start, end)
    const currentLine = content.value.substring(0, start).split('\n').pop()
    
    handleSelectionInsert(editorElement, start, end, selectedText, currentLine, markdown, type)
    return
  }
  
  // 普通插入
  content.value += markdown
  handleContentChange(content.value)
}

// 导出HTML
const exportHtml = () => {
  const htmlContent = generateHtmlContent()
  downloadHtmlFile(htmlContent)
}

// 生成HTML内容
const generateHtmlContent = () => {
  return `
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Markdown导出</title>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; padding: 20px; max-width: 800px; margin: 0 auto; }
        h1, h2, h3, h4, h5, h6 { font-weight: bold; margin-top: 1.5em; margin-bottom: 0.5em; }
        h1 { font-size: 2em; }
        h2 { font-size: 1.5em; }
        h3 { font-size: 1.2em; }
        p { margin-bottom: 1em; }
        ul, ol { margin-left: 2em; margin-bottom: 1em; }
        code { background: #f4f4f4; padding: 2px 4px; border-radius: 3px; font-family: 'Courier New', monospace; }
        pre { background: #f4f4f4; padding: 10px; border-radius: 5px; overflow-x: auto; }
        pre code { background: none; padding: 0; }
        blockquote { border-left: 4px solid #ddd; padding-left: 10px; margin-left: 0; color: #666; }
        a { color: #0066cc; text-decoration: none; }
        a:hover { text-decoration: underline; }
      </style>
    </head>
    <body>
      ${content.value}
    </body>
    </html>
  `
}

// 下载HTML文件
const downloadHtmlFile = (htmlContent) => {
  const blob = new Blob([htmlContent], { type: 'text/html' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'markdown-export.html'
  a.click()
  URL.revokeObjectURL(url)
}

// 复制内容到剪贴板
const copyContent = async () => {
  try {
    await navigator.clipboard.writeText(content.value)
    alert('内容已复制到剪贴板')
  } catch (err) {
    console.error('复制失败:', err)
    alert('复制失败，请手动复制')
  }
}

// 滚动跟随功能（编辑器到预览）
const handleEditorScroll = (event) => {
  if (previewRef.value) {
    const editorElement = event.target
    const previewElement = previewRef.value.$el.querySelector('.preview-content')
    
    if (previewElement) {
      syncScroll(editorElement, previewElement)
    }
  }
}

// 滚动跟随功能（预览到编辑器）
const handlePreviewScroll = (event) => {
  if (editorRef.value) {
    const previewElement = event.target
    const editorElement = editorRef.value.editorRef
    
    if (editorElement) {
      syncScrollReverse(previewElement, editorElement)
    }
  }
}

// 同步滚动（编辑器到预览）
const syncScroll = (editorElement, previewElement) => {
  // 计算滚动比例
  const editorScrollTop = editorElement.scrollTop
  const editorScrollHeight = editorElement.scrollHeight - editorElement.clientHeight
  const scrollRatio = editorScrollTop / editorScrollHeight
  
  // 应用到预览区
  const previewScrollHeight = previewElement.scrollHeight - previewElement.clientHeight
  const previewScrollTop = scrollRatio * previewScrollHeight
  
  previewElement.scrollTop = previewScrollTop
}

// 同步滚动（预览到编辑器）
const syncScrollReverse = (previewElement, editorElement) => {
  // 计算滚动比例
  const previewScrollTop = previewElement.scrollTop
  const previewScrollHeight = previewElement.scrollHeight - previewElement.clientHeight
  const scrollRatio = previewScrollTop / previewScrollHeight
  
  // 应用到编辑器区
  const editorScrollHeight = editorElement.scrollHeight - editorElement.clientHeight
  const editorScrollTop = scrollRatio * editorScrollHeight
  
  editorElement.scrollTop = editorScrollTop
}

// 撤回功能
const undo = () => {
  if (historyIndex.value > 0) {
    historyIndex.value--
    content.value = history.value[historyIndex.value]
    localStorage.setItem('markdown-content', content.value)
  }
}

// 恢复功能
const redo = () => {
  if (historyIndex.value < history.value.length - 1) {
    historyIndex.value++
    content.value = history.value[historyIndex.value]
    localStorage.setItem('markdown-content', content.value)
  }
}

// 快捷键支持
const handleKeydown = (e) => {
  // Ctrl+Z 撤回
  if (e.ctrlKey && e.key === 'z' && !e.shiftKey) {
    e.preventDefault()
    undo()
  }
  // Ctrl+Y 恢复
  if (e.ctrlKey && e.key === 'y') {
    e.preventDefault()
    redo()
  }
  // Ctrl+Shift+Z 恢复
  if (e.ctrlKey && e.shiftKey && e.key === 'z') {
    e.preventDefault()
    redo()
  }
}
</script>

<style scoped>
.app {
  min-height: 100vh;
  background-color: #f8f5f0;
  color: #333;
  transition: all 0.3s ease;
}

.app.dark-mode {
  background-color: #1e1e1e;
  color: #e0e0e0;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: linear-gradient(135deg, #8B4513, #D2691E);
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.3);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.app-header::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20"><circle cx="10" cy="10" r="1" fill="rgba(255,255,255,0.1)"/></svg>') repeat;
  opacity: 0.3;
}

.app.dark-mode .app-header {
  background: linear-gradient(135deg, #2d2d2d, #444);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

.brand {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  position: relative;
  z-index: 1;
}

.brand-icon {
  font-size: 2rem;
  animation: pulse 3s infinite;
  filter: drop-shadow(2px 2px 4px rgba(0, 0, 0, 0.3));
}

.artistic-title {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 900;
  position: relative;
  z-index: 1;
  display: flex;
  align-items: baseline;
  gap: 0.3rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.nut-text {
  color: #fff;
  text-shadow: 3px 3px 0px #8B4513, 6px 6px 0px rgba(139, 69, 19, 0.3);
  font-size: 1.8rem;
  font-family: 'Impact', 'Arial Black', sans-serif;
  animation: pulse 2s infinite alternate;
}

.cn-text {
  color: #fff;
  text-shadow: 2px 2px 0px #D2691E, 4px 4px 0px rgba(210, 105, 30, 0.3);
  font-size: 1.5rem;
  font-family: 'Microsoft YaHei', 'SimHei', sans-serif;
  font-weight: 700;
  animation: pulse 2s infinite alternate 0.3s;
}

.md-text {
  color: #fff;
  text-shadow: 1px 1px 0px #CD853F, 2px 2px 0px rgba(205, 133, 63, 0.3);
  font-size: 1.2rem;
  font-family: 'Courier New', monospace;
  font-weight: 700;
  animation: pulse 2s infinite alternate 0.6s;
}

.artistic-title::after {
  content: 'Beta';
  font-size: 0.7rem;
  font-weight: 400;
  margin-left: 0.5rem;
  opacity: 0.8;
  background: rgba(255, 255, 255, 0.2);
  padding: 0.2rem 0.4rem;
  border-radius: 10px;
  backdrop-filter: blur(5px);
  font-family: 'Arial', sans-serif;
  text-shadow: none;
  animation: none;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  100% {
    transform: scale(1.05);
  }
}

.header-actions {
  display: flex;
  gap: 0.8rem;
  align-items: center;
  position: relative;
  z-index: 1;
}

.export-btn {
  padding: 0.5rem 1rem;
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
  font-weight: 500;
  backdrop-filter: blur(5px);
}

.export-btn:hover {
  background-color: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.theme-toggle {
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  width: 45px;
  height: 45px;
  font-size: 1.3rem;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
}

.theme-toggle:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.app.dark-mode .theme-toggle {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
}

.editor-container {
  display: flex;
  flex: 1;
  height: calc(100vh - 140px);
  margin: 1.5rem;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
  background: #fff;
}

.app.dark-mode .editor-container {
  background: #2d2d2d;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
}

@media (max-width: 768px) {
  .editor-container {
    flex-direction: column;
    height: calc(100vh - 180px);
    margin: 0.8rem;
  }
  
  .app-header {
    padding: 0.8rem 1rem;
  }
  
  .brand {
    gap: 0.6rem;
  }
  
  .brand-icon {
    font-size: 1.6rem;
  }
  
  .app-header h1 {
    font-size: 1.3rem;
  }
  
  .app-header h1::after {
    font-size: 0.6rem;
    padding: 0.1rem 0.3rem;
  }
  
  .header-actions {
    gap: 0.5rem;
  }
  
  .export-btn {
    padding: 0.4rem 0.6rem;
    font-size: 0.8rem;
  }
  
  .theme-toggle {
    width: 40px;
    height: 40px;
    font-size: 1.2rem;
  }
}

@media (max-width: 480px) {
  .editor-container {
    height: calc(100vh - 160px);
    margin: 0.5rem;
  }
  
  .app-header {
    padding: 0.6rem 0.8rem;
  }
  
  .brand {
    gap: 0.4rem;
  }
  
  .brand-icon {
    font-size: 1.4rem;
  }
  
  .app-header h1 {
    font-size: 1.1rem;
  }
  
  .app-header h1::after {
    font-size: 0.5rem;
    padding: 0.1rem 0.2rem;
  }
  
  .header-actions {
    gap: 0.3rem;
  }
  
  .export-btn {
    padding: 0.3rem 0.5rem;
    font-size: 0.7rem;
  }
  
  .theme-toggle {
    width: 36px;
    height: 36px;
    font-size: 1.1rem;
  }
}
</style>