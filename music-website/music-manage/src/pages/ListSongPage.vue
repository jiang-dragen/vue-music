<template>
  <div class="table">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-tickets"></i> 歌单歌曲信息
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-button type="primary" size="mini" class="handle-del mr10" @click="delAll">批量删除</el-button>
        <el-input v-model="select_word" size="mini" placeholder="筛选关键词" class="handle-input mr10"></el-input>
        <el-button type="primary" size="mini" @click="centerDialogVisible = true">添加歌曲</el-button>
      </div>
      <el-table
        :data="tableData"
        border
        size="mini"
        style="width: 100%"
        ref="multipleTable"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="50"></el-table-column>
        <el-table-column prop="name" label="歌手-歌曲"></el-table-column>
        <el-table-column label="操作" width="80">
          <template slot-scope="{ row }">
            <el-button size="small" type="danger" @click="handleDelete(row.id, row.songListId)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!--添加歌曲-->
    <el-dialog title="添加歌曲" :visible.sync="centerDialogVisible" width="400px" center>
      <el-form
        :model="registerForm"
        status-icon
        ref="registerForm"
        label-width="80px"
        class="demo-ruleForm"
        :rules="rules"
      >
        <el-form-item prop="singerName" label="歌手名字" size="mini">
          <el-input v-model="registerForm.singerName" placeholder="歌手名字"></el-input>
        </el-form-item>
        <el-form-item prop="songName" label="歌曲名字" size="mini">
          <el-input v-model="registerForm.songName" placeholder="歌曲名字"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button size="mini" @click="centerDialogVisible = false">取 消</el-button>
        <el-button type="primary" size="mini" @click="getSongId">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 删除提示框 -->
    <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
      <div class="del-dialog-cnt" align="center">删除不可恢复，是否确定删除？</div>
      <span slot="footer" class="dialog-footer">
        <el-button size="mini" @click="delVisible = false">取 消</el-button>
        <el-button type="primary" size="mini" @click="deleteRow">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { mixin } from '../mixins'
import { HttpManager } from '../api/index'

export default {
  name: 'list-song-page',
  mixins: [mixin],
  data () {
    return {
      registerForm: {
        singerName: '',
        songName: ''
      },
      tableData: [],
      tempDate: [],
      multipleSelection: [],
      centerDialogVisible: false,
      editVisible: false,
      delVisible: false,
      select_word: '',
      idx: -1,
      rules: {
        singerName: [
          {required: true, message: '请输入歌手名', trigger: 'blur'}
        ],
        songName: [
          {required: true, message: '请输入歌曲名', trigger: 'blur'}
        ]
      }
    }
  },
  watch: {
    select_word: function () {
      if (this.select_word === '') {
        this.tableData = this.tempDate
      } else {
        this.tableData = []
        console.log(this.tempDate)
        for (let item of this.tempDate) {
          if (item.name.includes(this.select_word)) {
            this.tableData.push(item)
          }
        }
      }
    }
  },
  created () {
    this.getData()
  },
  methods: {
    // 获取歌单
    getData () {
      this.tableData = []
      this.tempDate = []
      HttpManager.getListSongOfSongId(this.$route.query.id)
        .then(res => {
          for (let item of res) {
            // console.log(item.songId)
            this.getSong(item.songId)
            // console.log(this.tableData)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    // 获取歌单里对应的音乐
    getSong (id) {
      HttpManager.getSongOfId(id)
        .then(res => {
          console.log(res)
          this.tableData.push(res)
          // console.log(this.tableData)
          this.tempDate.push(res)
        })
        .catch(err => {
          console.log(err)
        })
    },
    // 获取要添加歌曲的ID
    getSongId () {
      let _this = this
      var id = _this.registerForm.singerName + '-' + _this.registerForm.songName
      console.log(id)
      HttpManager.songOfSongName(id)
        .then(res => {
          console.log(res)
          if (res.length === 0) {
            this.centerDialogVisible = false
            this.notify('未找到歌曲', 'error')
          } else {
            this.addSong(res[0].id)
          }
        })
    },
    // 添加歌曲
    addSong (id) {
      this.$refs['registerForm'].validate(valid => {
        if (valid) {
          let params = new URLSearchParams()
          params.append('songId', id)
          params.append('songListId', this.$route.query.id)
          HttpManager.setListSong(params)
            .then(res => {
              if (res.code === 1) {
                this.getData()
                this.notify('添加成功', 'success')
              } else if (res.code === 0) {
                this.notify('添加失败', 'error')
              } else {
                this.notify('歌曲已存在,请勿重复添加', 'error')
              }
            })
            .catch(err => {
              console.log(err)
            })
          this.centerDialogVisible = false
        }
      })
    },
    // 确定删除
    deleteRow () {
      HttpManager.deleteListSong(this.idx, this.$route.query.id)
        .then(res => {
          if (res) {
            this.getData()
            this.notify('删除成功', 'success')
          } else {
            this.notify('删除失败', 'error')
          }
        })
        .catch(err => {
          console.log(err)
        })
      this.delVisible = false
    },
    handleDelete (id) {
      this.delVisible = true
      this.idx = id
    }
  }
}
</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}
</style>
