<template>
  <div>{{ readersNumber }} {{ book.title }}</div>
</template>

<script>
import { ref, reactive } from 'vue'

export default {
  props: {
    title: {
      type: String,
      default: 'haha'
    }
  },
  setup(props) {
    const readersNumber = ref(0)
    const book = reactive({ title: 'Vue 3 Guide' })
    console.log(props.title)
    // expose to template
    return {
      readersNumber,
      book
    }
  }
}
</script>
