<template>
    <div 
        :style="wrapperStyle"
        @click="handleClickWrapper"
    >
        <div ref="terminalRef" class="jun-terminal" :style="mainStyle">
            <template v-for="(output, index) in outputList" :key="index">
                <div class="terminal-row">
                    <span style="user-select: none; margin-right: 10px; padding: 4px 11px">
                        {{ prompt }}
                    </span>
                    <span>{{ output.text }}</span>
                </div>
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
<script setup>

import {computed, ref} from 'vue';
import { useTerminalConfigStore } from '@/stores/terminalConfigStore'

const props = defineProps({
  height: {
    type: String,
    default: "400px"
  },
  fullScreen: {
    type: Boolean,
    default: true
  },
  user: {
    type: String,
    default: 'local'
  }
})

const terminalRef = ref()
// 输出列表
const outputList = ref([]);

const commandInputRef = ref()

// 命令是否运行
const isRunning = ref(false);

// 引入终端配置状态
const configStore = useTerminalConfigStore();

/**
 * 初始命令
 */
 const initCommand = {
  text: "",
  placeholder: "",
};

/**
 * 待输入的命令
 */
const inputCommand = ref({
  ...initCommand,
});

/**
 * 全局记录当前命令，便于写入结果
 */
let currentNewCommand

const prompt = computed(() => {
  return `[${props.user}]$`;
});

/**
 * 当点击空白聚焦输入框
 */
 function handleClickWrapper(event) {
  //@ts-ignore
  if (event.target.className === "jun-terminal") {
    focusInput();
  }
}

/**
 * 输入框聚焦
 */
 const focusInput = () => {
  commandInputRef.value.focus();
};

/**
 * 终端主样式
 */
 const mainStyle = computed(() => {
  const fullScreenStyle = {
    position: "fixed",
    top: 0,
    bottom: 0,
    left: 0,
    right: 0,
  };
  return props.fullScreen
    ? fullScreenStyle
    : {
        height: props.height,
      };
});

/**
 * 终端包装类主样式
 */
 const wrapperStyle = computed(() => {
  const { background } = configStore;
  const style = {
    ...mainStyle.value,
  };
  if (background?.startsWith("http")) {
    style.background = `url(${background})`;
  } else {
    style.background = background;
  }
  return style;
});

/**
 * 提交命令
*/
function doSubmitCommand() {
    isRunning.value = true
    let inputText = inputCommand.value.text;
    // 执行命令
    const newCommand = {
        text: inputText,
        type: "command",
        resultList: [],
    };
    // 记录当前命令，便于写入结果
    currentNewCommand = newCommand;

    outputList.value.push(newCommand);

    console.log('aaa', inputText)
    // 自动滚到底部
    setTimeout(() => {
        terminalRef.value.scrollTop = terminalRef.value.scrollHeight;
    }, 50);
    isRunning.value = false
}

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
  padding: 0 10px;
}

.command-input :deep(.ant-input-group-addon) {
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