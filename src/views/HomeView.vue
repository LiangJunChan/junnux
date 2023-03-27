<template>
  <JunnuxTerminal
    ref="terminalRef"
    :on-submit-command="onSubmitCommand"
  />

</template>
<script setup lang="ts">
import JunnuxTerminal from "@/components/terminal/JunnuxTerminal.vue"
import { ref } from "vue";

const terminalRef = ref();

const onSubmitCommand = async (inputText: string) => {
  if (!inputText) {
    return;
  }
  const terminal = terminalRef.value.terminal;
  console.warn('===',inputText, terminalRef.value)
};
</script>
<style scoped lang="less"></style>
