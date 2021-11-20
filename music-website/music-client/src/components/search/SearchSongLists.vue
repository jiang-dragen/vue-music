<template>
  <div class="search-song-Lists">
    <content-list :contentList="albumDatas" path="song-list-album"></content-list>
  </div>
</template>

<script>
import mixin from '../../mixins'
import ContentList from '../ContentList'
import { HttpManager } from '../../api/index'

export default {
  name: 'search-song-Lists',
  mixins: [mixin],
  components: {
    ContentList
  },
  data () {
    return {
      albumDatas: []
    }
  },
  mounted () {
    this.getSearchList()
  },
  methods: {
    getSearchList () {
      if (!this.$route.query.keywords) {
        this.notify('您输入的内容为空', 'warning')
      } else {
        HttpManager.getSongListOfLikeTitle(this.$route.query.keywords)
          .then(res => {
            if (!res.length) {
              this.notify('暂无该歌单内容', 'warning')
            } else {
            // eslint-disable-next-line indent
            this.albumDatas = res
            }
          })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../../assets/css/search-song-Lists.scss';
</style>
