<template>
  <el-input :modelValue="userName" @update:modelValue="handleInput"></el-input>
  <el-button @click="handleSubmit">提交测试</el-button>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'VInput',
  props: ['modelValue', 'title'],
  setup(props, context) {
    let userName = ref(props.modelValue)
    const handleInput = (val) => {
      userName.value = val
      context.emit('update:modelValue', val)
    }
    const handleSubmit = () => {
      context.emit('handleSubmit', userName)
    }
    return {
      userName,
      handleInput,
      handleSubmit
    }
  }
}
</script>

<style>

</style>
