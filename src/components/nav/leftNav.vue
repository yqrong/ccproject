<template>
  <el-col :span="24" class="main">
    <!--左侧导航-->
    <aside :class="{showSidebar:!collapsed}">
      <!--展开折叠开关-->
      <div class="menu-toggle" @click.prevent="collapse">
        <i class="iconfont icon-outdent" v-show="!collapsed" title="收起"></i>
        <i class="iconfont icon-indent" v-show="collapsed" title="展开"></i>
      </div>
      <!--导航菜单-->
      <el-menu :default-active="$route.path" router :collapse="collapsed" ref="leftNavigation">
        <template v-for="(issue,index) in $router.options.routes">
          <template v-if="issue.name === $store.state.leftNavState"><!-- 注意：这里就是leftNavState状态作用之处，当该值与router的根路由的name相等时加载相应菜单组 -->
            <template v-for="(item,index) in issue.children">
              <el-submenu v-if="!item.leaf" :index="index+''" v-show="item.menuShow">
                <template slot="title"><i :class="item.iconCls"></i><span slot="title">{{item.name}}</span></template>
                <el-menu-item v-for="term in item.children" :key="term.path" :index="term.path" v-if="term.menuShow"
                              :class="$route.path==term.path?'is-active':''">
                  <i :class="term.iconCls"></i><span slot="title">{{term.name}}</span>
                </el-menu-item>
              </el-submenu>
              <el-menu-item v-else-if="item.leaf&&item.children&&item.children.length" :index="item.children[0].path"
                            :class="$route.path==item.children[0].path?'is-active':''" v-show="item.menuShow">
                <i :class="item.iconCls"></i><span slot="title">{{item.children[0].name}}</span>
              </el-menu-item>
            </template>
          </template>
        </template>
      </el-menu>
    </aside>
    <!--右侧内容区-->
    <section class="content-container">
      <div class="grid-content bg-purple-light">
        <el-col :span="24" class="content-wrapper">
          <transition name="fade" mode="out-in">
            <router-view></router-view>
          </transition>
        </el-col>
      </div>
    </section>
  </el-col>
</template>
<script>
  export default {
    name: 'leftNav',
    data () {
      return {
        collapsed: this.$store.state.collapsed
      }
    },
    methods: {
      //折叠导航栏
      collapse: function () {
        this.collapsed = !this.collapsed;
        this.$store.state.collapsed = this.collapsed;
      },
      // 左侧导航栏根据当前路径默认打开子菜单（如果当前路由是三级，则二级子菜单默认打开）
      defaultLeftNavOpened () {
        let cur_path = this.$route.path; //获取当前路由
        let routers = this.$router.options.routes; // 获取路由对象
        let subMenuIndex = 0, needOpenSubmenu = false;
        for (let i = 0; i < routers.length; i++) {
          let children = routers[i].children;
          if(children){
            for (let j = 0; j < children.length; j++) {
              if (children[j].path === cur_path) {
                break;
              }
              // 如果该菜单下有子菜单(个数>1且设置的leaf为false才有下拉子菜单)
              if(children[j].children && !children[j].leaf) {
                let grandChildren = children[j].children;
                for(let z=0; z<grandChildren.length; z++) {
                  if(grandChildren[z].path === cur_path) {
                    subMenuIndex = j;
                    needOpenSubmenu = true;
                    break;
                  }
                }
              }
            }
          }
        }
        if(this.$refs['leftNavigation'] && needOpenSubmenu) {
          this.$refs['leftNavigation'].open(subMenuIndex);
        }
      },
    },
    mounted() {
      this.defaultLeftNavOpened();
    }
  }
</script>
