<template>
  <div class="all-pic">
    <p class="back"><a href="javascript:void(0);" @click.stop="goBack">返回</a></p>
    <ul>
      <li v-for="(item, index) in list">
        <p>{{item.name}}</p>
        <img class="previewer-demo-img" :src="item.src" width="100%" @click="onClick(index)">
      </li>
    </ul>
    <previewer :list="list" ref="previewer" :options="options"></previewer>
  </div>
</template>
<script>
import {
  Previewer
} from 'cvux/src/components'
import rest from '../rest'
// import ajax from '../common/ajax'

export default {
  components: {
    Previewer
  },
  methods: {
    getTraceImages: rest.zs_used_car.getTraceImages,
    // getImageInfo: ajax.getImageInfo(),
    goBack() {
      this.$router.go(-1)
    },
    onClick(index) {
      this.$refs.previewer.show(index)
    }
  },
  data() {
    return {
      trace_id: 0,
      list: [],
      options: {
        // fullscreenEl: true,
        getThumbBoundsFn(index) {
          // find thumbnail element
          let thumbnail = document.querySelectorAll('.previewer-demo-img')[index]
            // get window scroll Y
          let pageYScroll = window.pageYOffset || document.documentElement.scrollTop
            // optionally get horizontal scroll
            // get position of element relative to viewport
          let rect = thumbnail.getBoundingClientRect()
            // w = width
          return {
            x: rect.left,
            y: rect.top + pageYScroll,
            w: rect.width
          }
          // Good guide on how to get element coordinates:
          // http://javascript.info/tutorial/coordinates
        }
      }
    }
  },
  created() {
    this.trace_id = this.$route.params.traceId
    if (this.trace_id) {
      this.getTraceImages(null, this.trace_id).then((res) => {
        let self = this
        let list = []
        if (res.code === 0) {
          this.list = res.data.images
          list = res.data.images
          list.forEach((v, i) => {
            let img = new window.Image()
            let j = i
            img.src = v.src
            // 如果图片被缓存，则直接返回缓存数据
            if (img.complete) {
              v.w = img.width
              v.h = img.height
            } else {
              // 完全加载完毕的事件
              img.onload = function () {
                v.w = img.width
                v.h = img.height
                if (j === list.length - 1) {
                  self.list = list
                }
              }
            }
          })
        } else {
          this.showToast(res.message)
        }
      })
    } else {
      this.showToast('trace_id不存在')
    }
  }
}
</script>
