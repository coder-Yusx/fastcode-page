<style scoped>
</style>
<template>
  <div ref="homePage">
    <el-row  style="height: 100%;">
      <el-col :span="3">
        <el-menu
          default-active="2"
          class="el-menu-vertical-demo"
          @open="handleOpen"
          @close="handleClose"
          @select="selectMenu"
          style="background: #dcdfe6;">
          <el-submenu v-for="(item,index) in this.menus" :index="item.id+''" :key="index">
            <template slot="title">
              <i class="el-icon-menu"></i>
              <span slot="title">{{item.name}}</span>
            </template>

              <el-menu-item v-for="(child,index) in item.children" :index="child.id+''" :key="index">
                <span slot="title">{{child.name}}</span>
              </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-col>

      <el-col :span="21" style="height: 100%;">
        <div style="background: #dcdfe6;width: 100%;height: 5.2%">1</div>
        <div style="background: aliceblue;width: 100%;height: 94.5%">
          <el-tabs v-model="editableTabsValue2" type="card" closable @tab-remove="removeTab">
            <el-tab-pane
              v-for="item in editableTabs2"
              :key="item.name"
              :label="item.title"
              :name="item.name"
            >
              <!--{{item.content}}-->
              <router-view/>
            </el-tab-pane>
          </el-tabs>
        </div>
        <!--<div style="background: aliceblue;width: 100%;height: 90%">1</div>-->
      </el-col>
    </el-row>

  </div>
</template>
<script>
  export default {
    data(){
      return {
        clientHeight:'',

        editableTabsValue2: '0',
        editableTabs2: [{
          title: '首页',
          name: '1',
          content: '首页'
        }],
        tabIndex: 2,
        menus:[],
        routers:[]
      }
    },
    methods:{
      getHeight(){
        console.log(window.innerHeight)
        this.clientHeight=window.innerHeight-20+'px';
        this.$refs.homePage.style.height = this.clientHeight
      },
      handleOpen(key, keyPath) {
        //console.log(key, keyPath);
      },
      handleClose(key, keyPath) {
        //console.log(key, keyPath);
      },
      selectMenu(key,keyPath){
        console.log(key, keyPath);
        for(var item of this.routers){
          if(key == item.id){
            this.addTab(item.routerName,item.routerPath)
          }
        }

        this.addTab(key)
      },

      addTab(key,path) {
        var flag = -1
        this.editableTabs2.forEach(function(tab,index){
          if(tab.name == key){
            flag = key
          }
        })
        if(flag != key){
          this.editableTabs2.push({
            title:key,
            name:key,
            content:key
          })
          //this.$router.push(path)
          this.$router.push('/user')
        }
        this.editableTabsValue2 = key+''
      },
      removeTab(targetName) {
        let tabs = this.editableTabs2;
        let activeName = this.editableTabsValue2;
        if (activeName === targetName) {
          tabs.forEach((tab, index) => {
            if (tab.name === targetName) {
              let nextTab = tabs[index + 1] || tabs[index - 1];
              if (nextTab) {
                activeName = nextTab.name;
              }
            }
          });
        }

        this.editableTabsValue2 = activeName;
        this.editableTabs2 = tabs.filter(tab => tab.name !== targetName);
      },
      /*menuToRouter:function(menus){
      if(!menus) return null;
      var children=[]
        for(var menu of menus){
          var router = {
            name:menu.routerName,
            path:menu.routerPath,
            component:function(){
              return import('@/components'+menu.routerComponenet)
            },
            children:this.menuToRouter(menu.getChildren)
          }
        }
        if(children) children.push(router)
        return children
      }*/
    },
    mounted: function () {
      window.addEventListener('resize', this.getHeight);
      this.getHeight()

      //获取菜单
      var _this = this
      this.$api.getMenus().then(function(data){
        //_this.$store.commit("initMenus",data.data)
        //_this.menus=this.$store.getters.getMenus
        //填充菜单
        //console.log(data.data)
        _this.menus = data.data.children

        //填充路由
        //var router=_this.menuToRouter(data.data.children);
        //console.log(router)

      })

      //获取路由
      this.$api.getRouter().then(function (data) {
        //console.log(data.data)
        var routers=[]
        for(var item of data.data){
          var router={
            path:item.routerPath,
            name:item.routerName,
            component:function () {
              return import('@/component'+item.routerComponent)
            }
          }
          routers.push(router)
        }
        //_this.$router.addRouters(routers)

        _this.$router.options.routes.push(routers[0])
        console.log(_this.$router.options.routes)
        _this.routers=routers
      })
    },
    destroyed(){
      window.removeEventListener('resize', this.getHeight)
    }
  }
</script>
