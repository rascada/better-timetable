<template lang='jade'>
.app
  timetable
</template>

<script>
import timetable from './components/timetable';

export default {
  components: {
    timetable,
  },
};
</script>

<style lang='stylus'>
@import '~flexstyl/flex';
*
  font-family 'Signika', sans-serif
body
  margin 0
  flex styl
  flex center
  background #eee
  min-height 100vh
  text-align center

</style>
