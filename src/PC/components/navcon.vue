<template>
  <div>
    <el-menu
      default-active="2"
      mode="horizontal"
      background-color="#7779c7"
      text-color="#ffffff">
      <!--主要网页信息和链接-->
      <el-menu-item index="1" @click="openLoginMenu">
        <img src="@/assets/pictures/minami.png" class="minami-logo" alt="网页出错了请重新加载_(´ཀ`」 ∠)_"/>
      </el-menu-item>
      <el-menu-item index="2" @click="jumpToPublic">美波七海歌单</el-menu-item>
      <el-menu-item index="3"><a href="https://space.bilibili.com/692283831" target="_blank">b站主页</a></el-menu-item>
      <el-menu-item index="4"><a href="https://live.bilibili.com/22571958" target="_blank">直播间</a></el-menu-item>
      <!--当前用户信息和状态（登录后显示）-->
      <el-submenu index="5" v-if="loginState" class="submenu">
        <template slot="title">{{ currentUser }}</template>
        <el-menu-item index="5-1" @click="logoutDialogVisible = true">退出登录</el-menu-item>
      </el-submenu>
    </el-menu>

    <!--退出登录提示框-->
    <el-dialog
      title="提示"
      :visible.sync="logoutDialogVisible"
      width="30%">
      <span>确定要退出登录吗？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="logoutDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="logout">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex'
import { getSingleMenu } from '@/api/menu'

export default {
  inject: ['reload'],
  data () {
    return {
      // 退出登录对话框的显示和隐藏
      logoutDialogVisible: false
    }
  },
  computed: {
    ...mapState(['loginState', 'currentUser'])
  },
  created () {
    const currentUser = window.sessionStorage.getItem('user')
    if (this.currentUser === '' && currentUser !== 'null') {
      this.setCurrentUser(currentUser)
    }
  },
  methods: {
    ...mapMutations(['setLoginDialogVisible', 'setLoginState', 'setCurrentUser']),
    // 打开登录菜单
    openLoginMenu() {
      getSingleMenu(1).then(res => {
        if (res.data.code === 200) {
          this.setLoginState(true)
          this.setLoginDialogVisible(false)
          if (this.$route.path.search('song-list') === -1) {
            this.$router.push('/pc/show/song-list')
          }
        } else {
          if (this.$route.path.search('public') === -1) {
            this.$router.push('/pc/public')
          }

          this.setLoginState(false)
          this.setLoginDialogVisible(true)
        }
      })
    },
    // 跳转回普通用户页面
    jumpToPublic() {
      if (this.$route.path.search('public') === -1) {
        this.$router.push('/pc/public')
      } else {
        this.reload()
      }
    },
    // 退出登录
    logout() {
      // 登录状态的监测数据全部初始化
      this.setLoginState(false)
      this.setCurrentUser('')
      window.localStorage.setItem('token', undefined)

      // 跳转回默认页面
      this.logoutDialogVisible = false
      this.$message.success('退出登录成功！')
      if (this.$route.path.search('public') === -1) {
        this.$router.push('/pc/public')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.minami-logo {
  height: 50px;
  width: 50px;
}

.submenu {
  float: right;
}
</style>
