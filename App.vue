<template>
  <div>
      <DatePicker v-model="now"></DatePicker>
      
  </div>
</template>
xxxxxx
<script>
import DatePicker from './date-picker';
export default {
  data() {
    return {
        now: new Date()
    }
  },
  methods: {

  },
  components: {
      DatePicker
  }
}
</script>
