<template>
  <div class="index">
    <el-container>
      <el-aside width="300px">
        <div class="logo-box">
          <span class="logo iconfont icon-ziyuan143"></span>
          <div class="title">
            <h2>管理平台</h2>
            <p>honggu keji Project Management System</p>
          </div>
        </div>
        <!-- 侧边栏菜单 -->
        <el-menu
          unique-opened
          router
          :default-active="currentPath"
          class="el-menu-vertical-demo"
          @open="handleOpen"
          @close="handleClose"
          @select="handleSelect"
          background-color="transparent"
          text-color="#fff"
          active-text-color="#46A5FE"
        >
          <template v-for="item in menuData">
            <template v-if="item.children">
              <!-- 二级菜单 -->
              <el-submenu :index="item.key" :key="item.key">
                <template slot="title">
                  <i :class="'iconfont icon-' + item.icon"></i>
                  <span slot="title">{{ item.title }}</span>
                </template>
                <template v-for="subItem in item.children">
                  <el-menu-item :index="subItem.path" :key="subItem.key">
                    <i :class="'iconfont icon-' + subItem.icon"></i>
                    <span slot="title">{{ subItem.title }}</span>
                  </el-menu-item>
                </template>
              </el-submenu>
            </template>
            <template v-else>
              <!-- 一级菜单 -->
              <el-menu-item :index="item.path" :key="item.key">
                <i :class="'iconfont icon-' + item.icon"></i>
                <span slot="title">{{ item.title }}</span>
              </el-menu-item>
            </template>
          </template>
        </el-menu>
      </el-aside>
      <el-container>
        <!-- 顶部区域 -->
        <el-header>
          <div class="fl">
            <a href="/">
              <span class="iconfont icon-zhuye"></span>
            </a>
            <div class="tab-box">
              <!-- 动态tab栏 -->
              <ul class="tab-box">
                <!-- 动态生成的子项 -->
                <li class="tab-item" v-for="(item, index) in tabList" :key="index">
                  <i :class="'iconfont icon-' + item.icon"></i>
                  <span @click="clickTitleHandle(item.path)">{{ item.title }}</span>
                  <span class="iconfont icon-guanbi" :style="currentPath == item.path ? 'display:inline-block' : ''" @click.stop.self="closeTabHandle(item.path)"></span>
                </li>
              </ul>
            </div>
          </div>
          <div class="fr">
            <span>
              <img src="../../assets/images/menu/当前用户图标.png" alt="用户" />
            </span>
            <span class="user-name"
              >{{ username }}<i class="el-icon-arrow-down"></i>
              <div class="user-child">
                <a href="javascript:;" @click="outLoginHandle">退出登录</a>
              </div>
            </span>
          </div>
        </el-header>
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import menuList from '../../assets/menuconfig/menu-config'
import * as type from '../../store/type'
export default {
  data() {
    return {
      menuData: menuList || [],
      // 当前选中的路由页面链接
      currentPath: this.$route.path || '/index/home',
      // TAB栏生成数据
      tabList: [],
      username: this.$store.getters[type.USERNAME] || sessionStorage.getItem('username')
    }
  },
  created() {},
  mounted() {
    // 初始化时响应式设置选中的路径为对应tab添加关闭按钮
    this.currentPath = this.$route.path
    this.createTabHandle(this.$route.path)
  },
  methods: {
    // 打开菜单
    handleOpen(key, keyPath) {
      // console.log(key, keyPath)
    },
    // 关闭菜单
    handleClose(key, keyPath) {
      // console.log(key, keyPath)
    },
    // 选中菜单
    handleSelect(key) {
      // 响应式设置选中的路径为对应tab添加关闭按钮
      this.currentPath = key
      this.createTabHandle(key)
    },
    // tab关闭
    closeTabHandle(path) {
      let currentIndex = ''
      // 循环找出要删除的元素位置
      this.tabList.forEach((item, index) => {
        if (item.path === path) {
          currentIndex = index
        }
      })
      // 删除指定元素
      this.tabList.splice(currentIndex, 1)
    },
    // 点击tab标题跳转到对应路由页面
    clickTitleHandle(path) {
      // 响应式设置选中的路径为对应tab添加关闭按钮
      this.currentPath = path
      this.$router.push(path)
    },
    // 根据选中的菜单tab栏生成
    createTabHandle(currentPath) {
      let newMenuList = []
      // 判断菜单数据是否存在
      if (menuList) {
        // 查找到了对应的菜单数据
        newMenuList = this.indexOfMenuInfoHandle(menuList, currentPath)
        // 数组去重一下，防止生成太多同样的tab栏
        this.tabList.every(item => {
          return item.path !== newMenuList.path
        }) && this.tabList.push(newMenuList)
      }
    },
    // 查找对应的菜单数据
    indexOfMenuInfoHandle(menuData, currentPath) {
      let back = {}
      // 遍历菜单数据
      menuData.forEach(item => {
        if (item.path === currentPath) {
          back = item
        } else if (!item.path) {
          // 只有查找到对应路径的信息才返回
          if (item.children.filter(citem => citem.path === currentPath)[0]) {
            back = item.children.filter(citem => citem.path === currentPath)[0]
          }
        }
      })
      return back
    },
    // 退出登录
    outLoginHandle() {
      this.$cookies.set('token', '')
      this.$store.commit('saveUserName', '')
      sessionStorage.setItem('username', '')
      this.$router.push('/login')
    }
  }
}
</script>
