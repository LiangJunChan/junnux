<template>
  <div :style="wrapperStyle" @click="handleClickWrapper">
    <div ref="terminalRef" class="jun-terminal" :style="mainStyle">
      <template v-for="(output, index) in outputList" :key="index">
        <template v-if="output.type === 'command'">
          <div class="terminal-row">
            <span style="user-select: none; margin-right: 10px">
              {{ prompt }}
            </span>
            <span>{{ output.text }}</span>
          </div>
          <div v-for="(result, idx) in output?.resultList" :key="idx" class="terminal-row">
            <ContentOutput :output="result" />
          </div>
        </template>
        <!-- 打印信息 -->
        <template v-else>
          <div class="terminal-row">
            <ContentOutput :output="output" />
          </div>
        </template>
      </template>
      <div class="terminal-row">
        <a-input
          ref="commandInputRef"
          class="command-input"
          v-model:value="inputCommand.text"
          :bordered="false"
          autofocus
          @press-enter="doSubmitCommand"
        >
          <template #addonBefore>
            <span class="command-input-prompt">{{ prompt }}</span>
          </template>
        </a-input>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { useTerminalConfigStore } from '@/stores/terminalConfigStore'
import ContentOutput from './ContentOutput.vue'
import type {
  CommandInputType,
  CommandOutputType,
  OutputStatusType,
  OutputType,
  TerminalType,
  TextOutputType
} from './type'
import { registerShortcuts } from './shortcuts'
const props = defineProps({
  height: {
    type: String,
    default: '400px'
  },
  fullScreen: {
    type: Boolean,
    default: true
  },
  user: {
    type: String,
    default: 'local'
  },
  onSubmitCommand: {
    type: Function
  }
})

const terminalRef = ref()
// 输出列表
const outputList = ref<OutputType[]>([])

const commandInputRef = ref()

// 命令是否运行
const isRunning = ref(false)

// 引入终端配置状态
const configStore = useTerminalConfigStore()

/**
 * 初始命令
 */
const initCommand = {
  text: '',
  placeholder: ''
}

/**
 * 待输入的命令
 */
const inputCommand = ref<CommandInputType>({
  ...initCommand
})

/**
 * 全局记录当前命令，便于写入结果
 */
let currentNewCommand

const prompt = computed(() => {
  return `[${props.user}]$`
})

/**
 * 当点击空白聚焦输入框
 */
function handleClickWrapper(event: Event) {
  //@ts-ignore
  if (event.target.className === 'jun-terminal') {
    focusInput()
  }
}

/**
 * 输入框聚焦
 */
const focusInput = () => {
  commandInputRef.value.focus()
}

/**
 * 终端主样式
 */
const mainStyle: any = computed(() => {
  const fullScreenStyle = {
    position: 'fixed',
    top: 0,
    bottom: 0,
    left: 0,
    right: 0
  }
  return props.fullScreen
    ? fullScreenStyle
    : {
        height: props.height
      }
})

/**
 * 终端包装类主样式
 */
const wrapperStyle = computed(() => {
  const { background } = configStore
  const style = {
    ...mainStyle.value
  }
  if (background?.startsWith('http')) {
    style.background = `url(${background})`
  } else {
    style.background = background
  }
  return style
})

/**
 * 提交命令
 */
async function doSubmitCommand() {
  isRunning.value = true
  let inputText = inputCommand.value.text
  // 执行命令
  const newCommand: CommandOutputType = {
    text: inputText,
    type: 'command',
    resultList: []
  }
  // 记录当前命令，便于写入结果
  currentNewCommand = newCommand

  await props.onSubmitCommand?.(inputText)

  outputList.value.push(newCommand)

  // console.log('aaa', inputText)
  // 自动滚到底部
  setTimeout(() => {
    terminalRef.value.scrollTop = terminalRef.value.scrollHeight
  }, 50)
  isRunning.value = false
}

/**
 * 清空所有输出
 */
const clear = () => {
  outputList.value = []
}

/**
 * 立即输出文本
 * @param text
 * @param status
 */
const writeTextOutput = (text: string, status?: OutputStatusType) => {
  const newOutput: TextOutputType = {
    text,
    type: 'text',
    status
  }
  outputList.value.push(newOutput)
}

/**
 * 操作终端的对象
 */
const terminal: TerminalType = {
  clear,
  focusInput,
  writeTextOutput
}

/**
 * 只执行一次
 */
onMounted(() => {
  registerShortcuts(terminal)
  const { welcomeTexts } = configStore
  if (welcomeTexts?.length > 0) {
    welcomeTexts.forEach((welcomeText) => {
      terminal.writeTextOutput(welcomeText)
    })
  } else {
    terminal.writeTextOutput(
      `Welcome to Junnux, coolest browser index for geeks!` +
        `<a href="//github.com/LiangJunChan/junnux" target='_blank'> GitHub Open Source</a>`
    )
    terminal.writeTextOutput(
      `Author <a href="//docs.qq.com/doc/DUFFRVWladXVjeUxW" target="_blank">chenliangjun</a>` +
        `: please input 'help' to enjoy`
    )
    terminal.writeTextOutput('<br/>')
  }
})
</script>
<style lang="less">
.jun-terminal-wrapper {
  background: black;
}

.jun-terminal {
  background: rgba(0, 0, 0, 1);
  padding: 20px;
  overflow: scroll;
}

.jun-terminal::-webkit-scrollbar {
  display: none;
}

.jun-terminal span {
  font-size: 16px;
}

.command-input {
  caret-color: white;
}

.command-input :deep(input) {
  color: white !important;
  font-size: 16px;
  //   padding: 0 10px;
}

.command-input .ant-input-group-addon {
  background: none;
  border: none;
  padding: 0;
}

.command-input-prompt {
  color: white;
  background: transparent;
}

.terminal-row {
  color: white;
  font-size: 16px;
  font-family: courier-new, courier, monospace;
  .ant-input-group-addon {
    background: transparent;
    border: none;
  }
  .ant-input {
    color: #fff;
  }
}
</style>
