<template>
  <div class="preview-wrapper" :class="`theme-${previewTheme}`">
    <div class="preview-header">
      <h3>预览区</h3>
      <div class="theme-selector">
        <button 
          v-for="theme in themes" 
          :key="theme.value"
          :class="['theme-btn', { active: previewTheme === theme.value }]"
          :title="theme.name"
          @click="changeTheme(theme.value)"
        >
          {{ theme.icon }}
        </button>
      </div>
    </div>
    <div class="preview-content" v-html="renderedContent" @scroll="handleScroll"></div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { marked } from 'marked'
import hljs from 'highlight.js'
import DOMPurify from 'dompurify'
import katex from 'katex'
import 'katex/dist/katex.min.css'
// 导入常用语言支持
import javascript from 'highlight.js/lib/languages/javascript'
import typescript from 'highlight.js/lib/languages/typescript'
import html from 'highlight.js/lib/languages/xml'
import css from 'highlight.js/lib/languages/css'
import json from 'highlight.js/lib/languages/json'
import python from 'highlight.js/lib/languages/python'
import java from 'highlight.js/lib/languages/java'
import csharp from 'highlight.js/lib/languages/csharp'
import cpp from 'highlight.js/lib/languages/cpp'
import bash from 'highlight.js/lib/languages/bash'
import sql from 'highlight.js/lib/languages/sql'
import 'highlight.js/styles/github.css'

const props = defineProps({
  content: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['scroll'])

// 处理预览区滚动事件
const handleScroll = (event) => {
  emit('scroll', event)
}

// 主题配置
const themes = [
  { value: 'default', name: '默认主题', icon: '🎨' },
  { value: 'simple', name: '简约主题', icon: '📝' },
  { value: 'dark', name: '暗黑主题', icon: '🌙' },
  { value: 'github', name: 'GitHub主题', icon: '📘' }
]

// 预览主题状态
const previewTheme = ref('default')

// 从本地存储加载主题偏好
onMounted(() => {
  const savedTheme = localStorage.getItem('preview-theme')
  if (savedTheme && themes.some(theme => theme.value === savedTheme)) {
    previewTheme.value = savedTheme
  }
})

// 注册语言
hljs.registerLanguage('javascript', javascript)
hljs.registerLanguage('typescript', typescript)
hljs.registerLanguage('html', html)
hljs.registerLanguage('css', css)
hljs.registerLanguage('json', json)
hljs.registerLanguage('python', python)
hljs.registerLanguage('java', java)
hljs.registerLanguage('csharp', csharp)
hljs.registerLanguage('cpp', cpp)
hljs.registerLanguage('bash', bash)
hljs.registerLanguage('sql', sql)

// 配置marked以支持数学公式
const markedOptions = {
  highlight: function(code, lang) {
    if (lang === 'math') {
      try {
        return katex.renderToString(code, {
          throwOnError: false
        })
      } catch (error) {
        return code
      }
    }
    // 普通代码块
    const language = hljs.getLanguage(lang) ? lang : 'plaintext'
    const highlightedCode = hljs.highlight(code, { language }).value
    return highlightedCode
  },
  breaks: true,
  gfm: true
}

// 处理行内数学公式的函数
const processInlineMath = (src) => {
  return src.replace(/\$(.*?)\$/g, function(match, formula) {
    try {
      return katex.renderToString(formula, {
        throwOnError: false
      })
    } catch (error) {
      return match
    }
  })
}

const renderedContent = computed(() => {
  // 处理行内数学公式
  const contentWithMath = processInlineMath(props.content)
  // 使用正确的marked API
  const html = marked.parse(contentWithMath, markedOptions)
  return DOMPurify.sanitize(html)
})

// 切换主题
const changeTheme = (theme) => {
  previewTheme.value = theme
  localStorage.setItem('preview-theme', theme)
}
</script>

<style scoped>
.preview-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: #fff;
  transition: all 0.3s ease;
  position: relative;
}

.app.dark-mode .preview-wrapper {
  background-color: #2d2d2d;
}

.preview-header {
  padding: 0.8rem 1.2rem;
  border-bottom: 1px solid #e0d4c0;
  background: linear-gradient(135deg, #f8f5f0, #f0e6d2);
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
}

.theme-selector {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.theme-btn {
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  width: 32px;
  height: 32px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.theme-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.theme-btn.active {
  background: rgba(210, 105, 30, 0.8);
  border-color: #D2691E;
  color: white;
  box-shadow: 0 2px 8px rgba(210, 105, 30, 0.4);
}

.app.dark-mode .theme-btn {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
}

.app.dark-mode .theme-btn:hover {
  background: rgba(255, 255, 255, 0.15);
}

.app.dark-mode .theme-btn.active {
  background: rgba(255, 152, 0, 0.8);
  border-color: #ff9800;
  box-shadow: 0 2px 8px rgba(255, 152, 0, 0.4);
}

.app.dark-mode .preview-header {
  background: linear-gradient(135deg, #333, #3a3a3a);
  border-bottom-color: #444;
}

.preview-header h3 {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: #8B4513;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.preview-header h3::before {
  content: '👁️';
  font-size: 1.1rem;
}

.app.dark-mode .preview-header h3 {
  color: #ccc;
}

.preview-content {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  line-height: 1.8;
  color: inherit;
  max-width: 100%;
}

/* 预览区样式 */
.preview-content :deep(h1),
.preview-content :deep(h2),
.preview-content :deep(h3),
.preview-content :deep(h4),
.preview-content :deep(h5),
.preview-content :deep(h6) {
  font-weight: 700;
  margin-top: 2em;
  margin-bottom: 1em;
  color: inherit;
  position: relative;
  padding-bottom: 0.5rem;
}

.preview-content :deep(h1) {
  font-size: 2.2em;
  border-bottom: 2px solid #e0d4c0;
  padding-bottom: 0.8rem;
  color: #8B4513;
}

.preview-content :deep(h2) {
  font-size: 1.8em;
  border-bottom: 1px solid #e0d4c0;
  color: #D2691E;
}

.preview-content :deep(h3) {
  font-size: 1.4em;
  color: #CD853F;
}

.app.dark-mode .preview-content :deep(h1) {
  border-bottom-color: #444;
  color: #ff9800;
}

.app.dark-mode .preview-content :deep(h2) {
  border-bottom-color: #444;
  color: #ffb74d;
}

.app.dark-mode .preview-content :deep(h3) {
  color: #ffcc80;
}

.preview-content :deep(p) {
  margin-bottom: 1.5em;
  text-align: justify;
}

.preview-content :deep(ul),
.preview-content :deep(ol) {
  margin-left: 2.5em;
  margin-bottom: 1.5em;
}

.preview-content :deep(li) {
  margin-bottom: 0.8em;
  position: relative;
}

.preview-content :deep(ul li) {
  list-style-type: disc;
}

.preview-content :deep(ol li) {
  list-style-type: decimal;
}

.preview-content :deep(code) {
  background: #f0e6d2;
  padding: 3px 6px;
  border-radius: 4px;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 0.9em;
  color: #8B4513;
  border: 1px solid #e0d4c0;
}

.app.dark-mode .preview-content :deep(code) {
  background: #444;
  border-color: #555;
  color: #e0e0e0;
}

.preview-content :deep(pre) {
  background: linear-gradient(135deg, #f8f5f0, #f0e6d2);
  padding: 1.5rem;
  border-radius: 8px;
  overflow-x: auto;
  margin-bottom: 1.5em;
  border: 1px solid #e0d4c0;
  box-shadow: 0 2px 8px rgba(139, 69, 19, 0.1);
  position: relative;
}

.preview-content :deep(pre)::before {
  content: '';
  position: absolute;
  top: 10px;
  right: 10px;
  width: 12px;
  height: 12px;
  background: #ff6b6b;
  border-radius: 50%;
  box-shadow: 18px 0 0 #ffd93d, 36px 0 0 #6bcb77;
}

.app.dark-mode .preview-content :deep(pre) {
  background: linear-gradient(135deg, #333, #3a3a3a);
  border-color: #444;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.preview-content :deep(pre code) {
  background: none;
  padding: 0;
  border: none;
  color: inherit;
}

.preview-content :deep(blockquote) {
  border-left: 4px solid #D2691E;
  padding: 1rem 1.5rem;
  margin-left: 0;
  color: #666;
  font-style: italic;
  margin-bottom: 1.5em;
  background: rgba(210, 105, 30, 0.05);
  border-radius: 0 8px 8px 0;
}

.app.dark-mode .preview-content :deep(blockquote) {
  border-left-color: #ff9800;
  color: #aaa;
  background: rgba(255, 152, 0, 0.1);
}

.preview-content :deep(a) {
  color: #8B4513;
  text-decoration: none;
  position: relative;
  padding-bottom: 2px;
  transition: all 0.3s ease;
}

.preview-content :deep(a::after) {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 2px;
  background: #D2691E;
  transition: width 0.3s ease;
}

.preview-content :deep(a:hover) {
  color: #D2691E;
}

.preview-content :deep(a:hover::after) {
  width: 100%;
}

.app.dark-mode .preview-content :deep(a) {
  color: #ff9800;
}

.app.dark-mode .preview-content :deep(a::after) {
  background: #ffcc80;
}

.app.dark-mode .preview-content :deep(a:hover) {
  color: #ffcc80;
}

.preview-content :deep(img) {
  max-width: 100%;
  height: auto;
  border-radius: 8px;
  margin: 1.5em 0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  transition: transform 0.3s ease;
}

.preview-content :deep(img:hover) {
  transform: scale(1.02);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
}

.preview-content :deep(table) {
  width: 100%;
  border-collapse: collapse;
  margin: 1.5em 0;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.preview-content :deep(th),
.preview-content :deep(td) {
  padding: 0.8rem 1rem;
  border: 1px solid #e0d4c0;
  text-align: left;
  transition: background-color 0.3s ease;
}

.preview-content :deep(th) {
  background: linear-gradient(135deg, #8B4513, #D2691E);
  font-weight: bold;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.preview-content :deep(tr:nth-child(even)) {
  background-color: rgba(240, 230, 210, 0.3);
}

.preview-content :deep(tr:hover) {
  background-color: rgba(210, 105, 30, 0.1);
}

.app.dark-mode .preview-content :deep(th),
.app.dark-mode .preview-content :deep(td) {
  border-color: #555;
}

.app.dark-mode .preview-content :deep(th) {
  background: linear-gradient(135deg, #444, #555);
  color: #e0e0e0;
}

.app.dark-mode .preview-content :deep(tr:nth-child(even)) {
  background-color: rgba(255, 255, 255, 0.05);
}

.app.dark-mode .preview-content :deep(tr:hover) {
  background-color: rgba(255, 152, 0, 0.1);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .preview-header {
    padding: 0.6rem 1rem;
  }
  
  .preview-header h3 {
    font-size: 0.9rem;
  }
  
  .theme-btn {
    width: 28px;
    height: 28px;
    font-size: 0.9rem;
  }
  
  .preview-content {
    padding: 1.2rem;
    font-size: 14px;
  }
  
  .preview-content :deep(h1) {
    font-size: 1.8em;
  }
  
  .preview-content :deep(h2) {
    font-size: 1.5em;
  }
  
  .preview-content :deep(h3) {
    font-size: 1.2em;
  }
  
  .preview-content :deep(ul),
  .preview-content :deep(ol) {
    margin-left: 2em;
  }
}

/* 主题样式 */

/* 简约主题 */
.preview-wrapper.theme-simple {
  background-color: #fff;
}

.preview-wrapper.theme-simple .preview-header {
  background: #f5f5f5;
  border-bottom-color: #e0e0e0;
}

.preview-wrapper.theme-simple .preview-header h3 {
  color: #333;
}

.preview-wrapper.theme-simple .preview-content {
  background-color: #fff;
  color: #333;
  font-family: 'Helvetica Neue', Arial, sans-serif;
}

.preview-wrapper.theme-simple .preview-content :deep(h1),
.preview-wrapper.theme-simple .preview-content :deep(h2),
.preview-wrapper.theme-simple .preview-content :deep(h3) {
  color: #333;
  border-bottom-color: #e0e0e0;
}

.preview-wrapper.theme-simple .preview-content :deep(code) {
  background: #f0f0f0;
  border-color: #e0e0e0;
  color: #333;
}

.preview-wrapper.theme-simple .preview-content :deep(pre) {
  background: #f5f5f5;
  border-color: #e0e0e0;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.preview-wrapper.theme-simple .preview-content :deep(blockquote) {
  border-left-color: #ccc;
  background: #f9f9f9;
  color: #666;
}

.preview-wrapper.theme-simple .preview-content :deep(a) {
  color: #0066cc;
}

.preview-wrapper.theme-simple .preview-content :deep(a::after) {
  background: #0066cc;
}

.preview-wrapper.theme-simple .preview-content :deep(th) {
  background: #f5f5f5;
  color: #333;
  border-color: #e0e0e0;
}

.preview-wrapper.theme-simple .preview-content :deep(tr:nth-child(even)) {
  background-color: #f9f9f9;
}

.preview-wrapper.theme-simple .preview-content :deep(tr:hover) {
  background-color: #f0f0f0;
}

/* 暗黑主题 */
.preview-wrapper.theme-dark {
  background-color: #1e1e1e;
}

.preview-wrapper.theme-dark .preview-header {
  background: #2d2d2d;
  border-bottom-color: #444;
}

.preview-wrapper.theme-dark .preview-header h3 {
  color: #e0e0e0;
}

.preview-wrapper.theme-dark .preview-content {
  background-color: #1e1e1e;
  color: #e0e0e0;
}

.preview-wrapper.theme-dark .preview-content :deep(h1),
.preview-wrapper.theme-dark .preview-content :deep(h2),
.preview-wrapper.theme-dark .preview-content :deep(h3) {
  color: #e0e0e0;
  border-bottom-color: #444;
}

.preview-wrapper.theme-dark .preview-content :deep(code) {
  background: #333;
  border-color: #444;
  color: #e0e0e0;
}

.preview-wrapper.theme-dark .preview-content :deep(pre) {
  background: #2d2d2d;
  border-color: #444;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.preview-wrapper.theme-dark .preview-content :deep(blockquote) {
  border-left-color: #555;
  background: rgba(255, 255, 255, 0.05);
  color: #aaa;
}

.preview-wrapper.theme-dark .preview-content :deep(a) {
  color: #66b3ff;
}

.preview-wrapper.theme-dark .preview-content :deep(a::after) {
  background: #66b3ff;
}

.preview-wrapper.theme-dark .preview-content :deep(th) {
  background: #333;
  color: #e0e0e0;
  border-color: #444;
}

.preview-wrapper.theme-dark .preview-content :deep(tr:nth-child(even)) {
  background-color: rgba(255, 255, 255, 0.05);
}

.preview-wrapper.theme-dark .preview-content :deep(tr:hover) {
  background-color: rgba(255, 255, 255, 0.1);
}

/* GitHub主题 */
.preview-wrapper.theme-github {
  background-color: #fff;
}

.preview-wrapper.theme-github .preview-header {
  background: #f6f8fa;
  border-bottom-color: #e1e4e8;
}

.preview-wrapper.theme-github .preview-header h3 {
  color: #24292e;
}

.preview-wrapper.theme-github .preview-content {
  background-color: #fff;
  color: #24292e;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
}

.preview-wrapper.theme-github .preview-content :deep(h1),
.preview-wrapper.theme-github .preview-content :deep(h2),
.preview-wrapper.theme-github .preview-content :deep(h3) {
  color: #24292e;
  border-bottom-color: #eaecef;
}

.preview-wrapper.theme-github .preview-content :deep(h1) {
  font-size: 2em;
}

.preview-wrapper.theme-github .preview-content :deep(h2) {
  font-size: 1.5em;
}

.preview-wrapper.theme-github .preview-content :deep(h3) {
  font-size: 1.25em;
}

.preview-wrapper.theme-github .preview-content :deep(code) {
  background: #f6f8fa;
  border: 1px solid #eaecef;
  border-radius: 3px;
  color: #24292e;
  padding: 0.2em 0.4em;
}

.preview-wrapper.theme-github .preview-content :deep(pre) {
  background: #f6f8fa;
  border: 1px solid #eaecef;
  border-radius: 6px;
  padding: 16px;
  box-shadow: none;
}

.preview-wrapper.theme-github .preview-content :deep(blockquote) {
  border-left: 4px solid #dfe2e5;
  background: #f6f8fa;
  color: #6a737d;
  padding: 16px;
  border-radius: 0 6px 6px 0;
}

.preview-wrapper.theme-github .preview-content :deep(a) {
  color: #0366d6;
  text-decoration: none;
}

.preview-wrapper.theme-github .preview-content :deep(a:hover) {
  text-decoration: underline;
}

.preview-wrapper.theme-github .preview-content :deep(a::after) {
  display: none;
}

.preview-wrapper.theme-github .preview-content :deep(img) {
  border-radius: 6px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.preview-wrapper.theme-github .preview-content :deep(table) {
  border: 1px solid #eaecef;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: none;
}

.preview-wrapper.theme-github .preview-content :deep(th) {
  background: #f6f8fa;
  color: #24292e;
  border: 1px solid #eaecef;
  padding: 6px 13px;
  font-weight: 600;
}

.preview-wrapper.theme-github .preview-content :deep(td) {
  border: 1px solid #eaecef;
  padding: 6px 13px;
}

.preview-wrapper.theme-github .preview-content :deep(tr:nth-child(even)) {
  background-color: #f6f8fa;
}

.preview-wrapper.theme-github .preview-content :deep(tr:hover) {
  background-color: #f6f8fa;
}
</style>