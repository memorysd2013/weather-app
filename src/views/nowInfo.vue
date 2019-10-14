<template lang="pug">
  .now-info
    h2.now-tem  {{now_tem}}℃
    h2.now-sky  {{now_sky}}
    h2.now-rain
</template>

<script>
export default {
  data() {

  },
  props:['allData'],
  computed: {
    now_tem(){
      let temH=this.allData.records.location[0].weatherElement[4].time[0].parameter.parameterName
      let temL=this.allData.records.location[0].weatherElement[2].time[0].parameter.parameterName
      return (parseInt(temH,10)+parseInt(temL,10))/2
    },
    now_sky(){
      let sky=this.allData.records.location[0].weatherElement[0].time[0].parameter.parameterName
      let feel=this.allData.records.location[0].weatherElement[3].time[0].parameter.parameterName
      return `${sky} · ${feel}`
    },
  }
}
</script>