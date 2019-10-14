<template lang='pug'>
#app
  transition(name='fade')
    //- 設置v-if讓data內有資料時才繪製整個畫面    
    .container(v-if="weatherEle.length!==0&&weekData.length!==0&&weekEle.length!==0")
      .head
        .city-info
          h3 目前城市
          h2.city {{allData.records.location[NOW.now_idx].locationName}}
          i.fas.fa-chevron-circle-down(@click="toggleList" :class="{turnDown:toggleObj.showList}")
          h4 {{NOW.now_en}}
          .list(v-if='toggleObj.showList')
            div(v-for="(loc,idx) in locations")
              label(@click="chooseCity(loc,idx)") {{loc.location}}
              i.fas.fa-star(@click="addFav(loc,idx)")
          .favList
            h2 我的最愛地區
            .favLoc(v-for='(i,idx) in favList' v-if="favList.length!==0")
              .favbtn(@click="chooseCity({'location':i.slice(14,17)},i.slice(-2,-1))") {{i.slice(14,17)}}
              i.fas.fa-times(@click="removeFav(i)")
          
        .now-info
          h2.now-tem  {{NOW.now_tem}}℃
          h2.now-sky  {{NOW.now_sky}}
          h2.now-rain 降雨機率 {{weatherEle[1].time[0].parameter.parameterName}}%
      .body
        h2.topic 今明預報
        .wrap
          table.showWeather(cellspacing='0')
            th.spe_line {{NOW.now_city}}
            th.spe_line(v-for="ele in elements") {{ ele }}
            tr(v-for="n in 3")
              td.spe_line {{nowtime[n-1]}} {{weatherEle[0].time[n-1].startTime.split(' ')[1].slice(0,5)}} - {{weatherEle[0].time[n-1].endTime.split(' ')[1].slice(0,5)}}
              td.spe_line {{weatherEle[2].time[n-1].parameter.parameterName}}° - {{weatherEle[4].time[n-1].parameter.parameterName}}°

              td.spe_line 
                img(:src="'https://meteorological-app-asset.s3.us-east-2.amazonaws.com/w-icon/w'+dayPic[n-1]+'.png'" :title='weatherEle[0].time[n-1].parameter.parameterName')

              td.spe_line {{weatherEle[3].time[n-1].parameter.parameterName}}
              td.spe_line {{weatherEle[1].time[n-1].parameter.parameterName}}%
            
        h2.topic 一週預報
        .wrap(v-if="weekEle.length!==0")
          table.showWeather(cellspacing='0')
            th.spe_line {{NOW.now_city}}
            th.spe_line(v-for="i in 7") {{NOW.now_time.getMonth()+1}}/{{NOW.now_time.getDate()+i}}
            tr
              td.spe_line 白天
              td.spe_line.arr(v-for="i in WEEK.week_d")
                img(:src="'https://meteorological-app-asset.s3.us-east-2.amazonaws.com/w-icon/w'+weekPic[i]+'.png'")
                |{{weekEle[0].time[i].elementValue[0].value.split('。').slice(-5,-4)[0].slice(4,9).split('至').map(x=>x+'°').join('-')}}
            tr
              td 晚上
              td(v-for="i in WEEK.week_n")
                img(:src="'https://meteorological-app-asset.s3.us-east-2.amazonaws.com/w-icon/wn'+weekPic[i]+'.png'")
                |{{weekEle[0].time[i].elementValue[0].value.split('。').slice(-5,-4)[0].slice(4,9).split('至').map(x=>x+'°').join('-')}}
    .loading(v-else)
      h2 loading...
    // -
  .footer
    div
      h4 
        |TW Weather Web App Demo
        br
        |© 2019 Made by 
        strong Evan
    div
      h4 SPECIAL THANKS
      p 
        |Photographer 
        strong 
          a(href='https://www.pexels.com/@arnie-chou-304906') Arnie Chou 
        | & 
        strong 
          a(href='https://www.pexels.com/@fabianwiktor') Fabian Wiktor
      p 
        |Icon's author 
        strong
          a(href='https://www.iconfinder.com/iconsets/weather-413')  Dooder 
        |from ICONFINDER
</template>
<script>
// import nowInfo from '@/views/nowInfo.vue'
var arr = ['fav1', 'fav2', 'fav3']
// 全台36小時內天氣預報 api
var URL = 'https://opendata.cwb.gov.tw/api/v1/rest/datastore/F-C0032-001?Authorization=CWB-5B34607D-7B96-4799-84A7-D711B21BCD98'

// 全台一週天氣預報 綜合描述 api
var twURL = 'https://opendata.cwb.gov.tw/api/v1/rest/datastore/F-D0047-091?Authorization=CWB-5B34607D-7B96-4799-84A7-D711B21BCD98&elementName=WeatherDescription'

// 全台一週天氣預報 天氣描述 api
var twURLwx = 'https://opendata.cwb.gov.tw/api/v1/rest/datastore/F-D0047-091?Authorization=CWB-5B34607D-7B96-4799-84A7-D711B21BCD98&elementName=Wx'

var store = localStorage

export default {
  data(){
    return{
      // 今明預報資料
      allData: {},
      weatherEle: [], // 5項 分別對應於 elements
      // 一週預報資料
      weekData: [],
      weekEle: [],
      // 一週預報 天氣描述資料 換置圖片用
      weekWxData: [],
      weekWx: [],
      // 其他資料
      elements: ['溫度', '天氣狀況', '舒適度', '降雨機率(％)'],
      NOW: {
        now_time: {},
        now_tem: 0,
        now_sky: '',
        now_idx: 5,
        now_city: '臺北市',
        now_en: 'taipei_city'
      },
      WEEK: {
        week_d: [0, 2, 4, 6, 8, 10, 12],
        week_n: [1, 3, 5, 7, 9, 11, 13],
        picD: [],
        picN: []
      },
      locations: [
        {
          location: '臺北市',
          enLocation: 'taipei_city',
          val: 5,
          wVal: 9,
        }, {
          location: '新北市',
          enLocation: 'new_taipei_city',
          val: 1,
          wVal: 15,
        }, {
          location: '臺中市',
          enLocation: 'taichung_city',
          val: 11,
          wVal: 17,
        }, {
          location: '臺南市',
          enLocation: 'tainan_city',
          val: 6,
          wVal: 18,
        }, {
          location: '高雄市',
          enLocation: 'kaohsiung_city',
          val: 15,
          wVal: 12,
        }, {
          location: '基隆市',
          enLocation: 'keelung_city',
          val: 18,
          wVal: 16,
        }, {
          location: '桃園市',
          enLocation: 'taoyuan_country',
          val: 13,
          wVal: 19
        }, {
          location: '新竹市',
          enLocation: 'hsinchu_city',
          val: 4,
          wVal: 14
        }, {
          location: '新竹縣',
          enLocation: 'hsinchu_country',
          val: 3,
          wVal: 10
        }, {
          location: '苗栗縣',
          enLocation: 'miaoli_country',
          val: 8,
          wVal: 7
        }, {
          location: '彰化縣',
          enLocation: 'changhua_country',
          val: 20,
          wVal: 17
        }, {
          location: '南投縣',
          enLocation: 'nantou_country',
          val: 14,
          wVal: 1
        }, {
          location: '雲林縣',
          enLocation: 'yunlin_country',
          val: 9,
          wVal: 0
        }, {
          location: '嘉義市',
          enLocation: 'chiayi_city',
          val: 2,
          wVal: 21
        }, {
          location: '嘉義縣',
          enLocation: 'chiayi_country',
          val: 0,
          wVal: 20
        }, {
          location: '屏東縣',
          enLocation: 'pingtung_country',
          val: 17,
          wVal: 6
        }, {
          location: '宜蘭縣',
          enLocation: 'yilan_country',
          val: 7,
          wVal: 5
        }, {
          location: '花蓮縣',
          enLocation: 'hualien_country',
          val: 10,
          wVal: 11
        }, {
          location: '臺東縣',
          enLocation: 'taitung_country',
          val: 12,
          wVal: 3
        }, {
          location: '澎湖縣',
          enLocation: 'penghu_country',
          val: 19,
          wVal: 8
        }, {
          location: '金門縣',
          enLocation: 'kinmen_country',
          val: 16,
          wVal: 4
        }, {
          location: '連江縣',
          enLocation: 'lienchiang_country',
          val: 21,
          wVal: 2
        }],
      toggleObj: {
        showList: false
      },
      favList: []
    }
  },
  computed: {
    nowtime() {
      // 三元判斷式 判斷現在時間    
      return this.NOW.now_time.getHours() >= 6 && this.NOW.now_time.getHours() < 18 ? ['今日白天', '今晚明晨', '明日白天'] : ['今晚明晨', '明日白天', '明日晚上']
    },
    dayPic() {
      let w = []
      if(this.weatherEle.length!==0)
        for (let n = 0;n < 3;n++)
          w.push(this.weatherEle[0].time[n].parameter.parameterValue)
      // 呼叫函式判斷
      setPic(w);
      return w;
    },
    weekPic() {
      let dw = []
      let res = []
      if(this.weekWx.length!==0)
        for (let m = 0;m < 14;m++)
          dw.push(parseInt(this.weekWx[m].elementValue[1].value, 10))
      setPic(dw);
      return dw;
    },
  },
  watch: {
    weatherEle: {
      handler: function (newV, oldV) {
        if (newV.length !== 0) {
          let temH = this.weatherEle[4].time[0].parameter.parameterName
          let temL = this.weatherEle[2].time[0].parameter.parameterName
          this.NOW.now_tem = ((parseInt(temH, 10) + parseInt(temL, 10)) / 2).toFixed(1)

          let sky = this.weatherEle[0].time[0].parameter.parameterName
          let feel = this.weatherEle[3].time[0].parameter.parameterName
          this.NOW.now_sky = `${sky} · ${feel}`
        }
      }
    },
  },

  //   生命週期
  
  mounted() {
    this.favList = setFavList()
    // 抓取當前時間 判斷一週預報的白天晚上
    var now = new Date()
    this.NOW.now_time = now
    if (now.getHours() >= 6 && now.getHours() < 18) {
      this.WEEK.week_d = [0, 2, 4, 6, 8, 10, 12]
      this.WEEK.week_n = [1, 3, 5, 7, 9, 11, 13]
    } else {
      this.WEEK.week_d = [1, 3, 5, 7, 9, 11, 13]
      this.WEEK.week_n = [0, 2, 4, 6, 8, 10, 12]
    }

    // 取 今明預報
    fetch(URL)
      .then(res => res.json())
      .then(data => {
        this.allData = data
        this.weatherEle = data.records.location[5].weatherElement
      })
    // 取 一週預報
    fetch(twURL)
      .then(res => res.json())
      .then(data => {
        this.weekData = data.records.locations[0].location
        this.weekEle = data.records.locations[0].location[9].weatherElement
      })
    // 取 一週天氣描述 套用圖示
    fetch(twURLwx)
      .then(res => res.json())
      .then(data => {
        this.weekWxData = data.records.locations[0].location
        this.weekWx = data.records.locations[0].location[9].weatherElement[0].time
      })

  },
  methods: {
    // 切換選單
    toggleList() {
      this.toggleObj.showList = !this.toggleObj.showList
    },
    // 新增我的最愛地點
    addFav(loc, idx) {
      let save = JSON.stringify([loc, idx])
      this.toggleObj.showList = !this.toggleObj.showList
      if (store.fav1 === save ||
        store.fav2 === save ||
        store.fav3 === save) {
        alert('您選擇的城市已加入我的最愛')
        return
      }
      if (store.fav1 !== undefined &&
        store.fav2 !== undefined &&
        store.fav3 !== undefined) {
        alert('最多儲存三個城市，請先移除不需要的城市')
        return
      }
      if (!Boolean(store.fav1))
        store.setItem("fav1", save)
      else if (!Boolean(store.fav2))
        store.setItem("fav2", save)
      else if (!Boolean(store.fav3))
        store.setItem("fav3", save)

      this.favList = setFavList()
    },
    removeFav(i) {
      for (let n = 0;n < 3;n++) {
        if (store.getItem(arr[n]) !== null)
          store.getItem(arr[n]) === i ? store.removeItem(arr[n]) : ''
      }
      this.favList = setFavList()
    },
    // 選擇縣市的時候 帶入loc,idx更改所有資訊
    chooseCity(loc, idx) {
      let NL = this.locations[idx]
      this.NOW.now_idx = NL.val
      this.NOW.now_city = loc.location
      this.NOW.now_en = NL.enLocation
      this.toggleObj.showList = false
      this.weatherEle = this.allData.records.location[NL.val].weatherElement
      this.weekEle = this.weekData[NL.wVal].weatherElement
      this.weekWx = this.weekWxData[NL.wVal].weatherElement[0].time
    }
  }
}

// 判斷一週預報 天氣描述代號 將代號換成圖片
function setPic(w){
  for(let i=0;i<w.length;i++){
    if([1,24,25].some(x=>x==w[i]))w[i]='01'
    else if(w[i]==2)w[i]='02'
    else if(w[i]==3)w[i]='03'
    else if([4,5,6,7,26,27,28].some(x=>x==w[i]))w[i]='04'
    else if([15,16,17,18,21,22,33,34,35,36,41].some(x=>x==w[i]))w[i]='15'
    else if([23,37,42].some(x=>x==w[i]))w[i]='23'
    else{w[i]='08'}
      }
  return w
}

function setFavList(){
  let res=[]
  if(store.fav1===undefined&&
     store.fav2===undefined&&
     store.fav3===undefined) return res
  for(let n=0;n<3;n++){
    if(store.getItem(arr[n])!==null)
      res.push(store.getItem(arr[n]))
  }
  return res
}
</script>

<style lang="sass">
@mixin allWidth
  width: 100%
@mixin now-font
  margin: 0
  font-weight: lighter
  text-shadow: 0 0 6px #aaa
@mixin card-style
  box-sizing: border-box
  background-color: rgba(0,0,0,.7)
  border: 1px solid rgba(255,255,255,.7)
  border-radius: 4px
  margin: 10px 0
@mixin flex-btw
  display: flex
  justify-content: space-between
@mixin flex-center
  display: flex
  flex-direction: column
  align-items: center
  justify-content: center
@mixin hover
  cursor: pointer
  transition: all .3s
  
  // width 425px - 768px 
  
*
  // outline: 1px #aaf solid

  // animation
.fade-enter-active, .fade-leave-active
  transition: opacity 1s

.fade-enter, .fade-leave-to
  opacity: 0


body, h1, h2
  margin: 0
  padding: 0
#app
  color: #eeeeee
  font-family: 'Noto Serif TC'
  background-image: url('https://meteorological-app-asset.s3.us-east-2.amazonaws.com/app-bg3.jpg')
  background-position: center
  background-attachment: fixed
  width: 100%
.container
  +flex_center
  padding: 12px
.head
  +flex-center
  +allWidth
img
  filter: contrast(200%) invert(1) drop-shadow(0px 0px 1px #000) drop-shadow(0px 0px 3px #fff)
  width: 50px
.city-info
  +card-style
  width: 100%
  height: 220px
  padding: 20px
  position: relative
  h3
    +now-font
    font-size: 34px
  h4
    +now-font
    font-size: 22px
    padding: 6px 0
  .city
    +now-font
    font-size: 52px
    display: inline-block
  .fa-chevron-circle-down
    +hover
    font-size: 30px
    margin: 0 0 0 10px
    transition: all .5s
  .turnDown
    transform: rotate(1.5turn)
    transition: all .5s
  .list
    position: absolute
    height: 300px
    overflow: auto
    left: 100px
    top: 150px
    padding: 3px 0
    border-radius: 5px
    background: #fff
    label
      display: inline-block
      padding: 2px 10px 2px 20px
      border-right: 1px solid #aaa
      color: #333
      &:hover
        +hover
        background: #eee
    .fa-star
      padding: 0 8px
      transition: all .2s
      &:hover
        +hover
        color: #ffd000
.favList
  position: absolute
  right: 15px
  top: 24px
  border-left: 1px solid #999
  height: 170px
  padding-left: 15px
  h2
    +now-font
    font-size: 20px
    margin-bottom: 5px 
  .favLoc
    i:hover
      +hover
      color: #f66
    .favbtn
      display: inline-block
      width: 60px
      text-align: center
      padding: 3px 0
      margin: 6px 6px 6px 24px
      color: #000
      background: #fff
      border-radius: 4px
      &:hover
        +hover
        background: #ffd944
      
.now-info
  +card-style
  width: 100%
  height: 220px
  padding: 15px
  .now-tem,.now-sky,.now-rain
    +now-font
    text-align: center
  .now-tem
    font-size: 70px
.body
  +allWidth
  +flex-center
.wrap
  +allWidth
  overflow-y: hidden
  overflow-x: scroll
.topic
  font-size: 40px
  font-weight: 900
  letter-spacing: 5px
  text-shadow: 3px 3px 2px #000
.showWeather
  +card-style
  // +allWidth
  width: 760px
  box-sizing: border-box
  padding: 0 10px
  .spe_line
    border-bottom: 1px solid rgba(255,255,255,.3)
  th,td
    padding: 20px 0
    text-align: center
  th
    font-weight: bolder
    width: 45px
  td
    // text-align: center

.loading
  +flex_center
  +now-font
  width: 100%
  height: 500px
  h2
    color: #000

    
.footer
  +now-font
  +flex-btw
  box-sizing: border-box
  width: 100%
  background: rgba(0,0,0,.4)
  margin-top: 50px
  padding: 30px 50px 50px
  h4,p
    font-weight: normal
    margin: 5px 0
  p
    font-size: 12px
  a
    text-decoration: none
    color: #fff

    
@media screen and (max-width:425px)
  .city-info
    height: 300px
  .favList
    top: 200px
    left: 18px
    padding: 10px 0 0 0
    border-top: 1px solid #999
    border-left: none
    .favLoc
      display: inline
      .favbtn
        margin: 6px
  .footer
    +flex-center
    div
      padding-bottom: 20px
      
        
@media screen and (min-width:768px)
  .head
    +flex-btw
    flex-direction: row
    max-width: 760px
  .city-info
    width: 55%
  .now-info
    width: 42%
  .wrap
    +flex-center
  
    
  
@media screen and (max-width:1024px)

  
</style>
