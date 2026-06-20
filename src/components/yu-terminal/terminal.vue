<template>
  <div class="terminal">
    <div class="output" ref="outputRef">
      <div>欢迎使用 YuIndex 终端！输入 help 查看命令。</div>
      <div class="show" v-for="(item, index) in history" :key="index">
        <div class="showInput" v-if="item.type === 'input'">
          {{ '[local]$ ' + item.content }}
        </div>
        <div class="showOutput" v-else>
          {{ item.content }}
        </div>
      </div>
      <div ref="bottomAnchorRef"></div>
    </div>

    <div class="input">
      <span class="prompt">[local]$</span>
      <input class="import" type="text" v-model="inputText" @keydown.enter="handleCommand(), handleKeyDown($event)">
    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, nextTick } from 'vue'
import { getHelp, getCurrentTime, getUnKnow, getShortcut } from '../../core/commands/summary'

const inputText = ref('')
const history = ref<historyItem[]>([])
// bottomAnchorRef在这里有两种可能的值null或者div
const bottomAnchorRef = ref<HTMLElement | null>(null)
const outputRef = ref<HTMLElement | null>(null)
// 定义接口 用来表示内容type
interface historyItem {
  type: 'output' | 'input'
  content: string
}




const handleKeyDown = async (event: KeyboardEvent) => {
  if (event.ctrlKey && event.key === 'l') {
    history.value = []
    return
  } else if (event.ctrlKey && event.key === 'v') {
    //阻止浏览器默认行为
    event.preventDefault()
    try {
      const text = await navigator.clipboard.readText()
      if (!text) return
      history.value.push({
        content: text.trim(),
        type: 'input'
      })
    } catch (err) {
      alert('自定义粘贴失败，请检查权限或浏览器安全策略')
    }
  }
}

const handleCommand = async () => {
  const input = inputText.value.trim()
  if (!input) return
  history.value.push({
    content: input,
    type: 'input'
  })
  let output = ''
  if (input === 'date') {
    output = getCurrentTime()
  } else if (input == 'clear') {
    history.value = []
    inputText.value = ''
  } else if (input === 'help') {
    output = getHelp()
  } else if (input === 'shortcut') {
    output = getShortcut()
  }
  else {
    output = getUnKnow()
  }

  if (output) {
    history.value.push({
      content: output,
      type: 'output'
    })
  }

  inputText.value = ''
  await nextTick()
  const screenHeight = window.innerHeight
  const contentHeight = outputRef.value?.scrollHeight || 0
  //scrollIntoView()：浏览器原生滚动方法
  if (contentHeight > screenHeight && bottomAnchorRef.value) {
    bottomAnchorRef.value.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

<style scoped>
.terminal {
  display: flex;
  background-color: black;
  color: white;
  flex-direction: column;
  text-align: left;
  height: 100vh;
  width: 100vw;

  font-size: 16px;
}

.terminal>div,
.terminal>span {
  margin: 5px 10px;
}

input {
  margin: 0px 10px;
  color: white;
  background: transparent;
}

.input {
  display: flex;
}

.output {
  overflow-y: auto;
}

.prompt {
  white-space: nowrap;
}

.import {
  border: none;
  outline: none;
  flex: 1;
  line-height: 1.5;
  font-size: 16px;
}

.showInput {
  display: flex;
  color: greenyellow;

  align-items: center;
}

.showOutput {
  display: flex;
  align-items: center;
  white-space: pre-wrap;
}
</style>