<template>
  <div>
    <div v-if="!mobileNar">
      <el-menu
          :default-active="currentPath"
          :router="true"
          class="vertical_menu">
        <div class="logo">
          <img :src="imgUrl" alt="oj admin"/>
        </div>
        <el-menu-item index="/admin/">
          <i aria-hidden="true" class="fa fa-tachometer fa-size"/>
          {{ $t('m.Dashboard') }}
        </el-menu-item>
        <!-- <el-submenu v-if="isSuperAdmin" index="general"> -->
        <el-submenu v-if="isSuperAdmin" index="general">
          <template slot="title"><i class="el-icon-menu"/>{{ $t('m.General') }}
          </template>
          <el-menu-item index="/admin/user">
            {{$t('m.User_Admin') }}
          </el-menu-item>
          <el-menu-item index="/admin/announcement">
            {{$t('m.Announcement_Admin') }}
          </el-menu-item>
          <el-menu-item index="/admin/notice">
            {{$t('m.SysNotice') }}
          </el-menu-item>
          <el-menu-item index="/admin/conf">
            {{$t('m.System_Config') }}
          </el-menu-item>
          <el-menu-item index="/admin/switch">
            {{$t('m.System_Switch') }}
          </el-menu-item>
        </el-submenu>
        <el-submenu index="problem">
          <template slot="title">
            <i aria-hidden="true" class="fa fa-bars fa-size"/>
            {{ $t('m.Problem_Admin') }}
          </template>
          <el-menu-item index="/admin/problems">
            {{$t('m.Problem_List') }}
          </el-menu-item>
          <el-menu-item index="/admin/problem/create">
            {{$t('m.Create_Problem') }}
          </el-menu-item>
          <el-menu-item index="/admin/problem/tag">
            {{$t('m.Admin_Tag') }}
          </el-menu-item>
          <el-menu-item
              v-if="isSuperAdmin"
              index="/admin/problem/batch-operation">
            {{ $t('m.Export_Import_Problem') }}
          </el-menu-item>
        </el-submenu>

        <el-submenu index="contest">
          <template slot="title"><i aria-hidden="true" class="fa fa-trophy fa-size"/>
            {{ $t('m.Contest_Admin') }}
          </template>
          <el-menu-item index="/admin/contest">
            {{$t('m.Contest_List') }}
          </el-menu-item>
          <el-menu-item index="/admin/contest/create">
            {{$t('m.Create_Contest') }}
          </el-menu-item>
        </el-submenu>

        <el-submenu index="team">
          <template slot="title">
            <i aria-hidden="true"
              class="el-icon-user-solid"
              style="font-size: 20px;"/>
            {{ $t('m.Team_Admin') }}
          </template>
          <el-menu-item index="/admin/team">
            {{$t('m.Team_List') }}
          </el-menu-item>
          <el-menu-item index="/admin/team/create">
            {{$t('m.Create_Team')}}
          </el-menu-item>
        </el-submenu>
        
        <el-submenu index="training">
          <template slot="title">
            <i aria-hidden="true"
              class="el-icon-s-claim"
              style="font-size: 20px;"/>
            {{ $t('m.Training_Admin') }}
          </template>
          <el-menu-item index="/admin/training">
            {{$t('m.Training_List') }}
          </el-menu-item>
          <el-menu-item index="/admin/training/create">
            {{$t('m.Create_Training')}}
          </el-menu-item>
          <el-menu-item index="/admin/training/category">
            {{$t('m.Admin_Category') }}
          </el-menu-item>
        </el-submenu>


        <el-submenu index="discussion">
          <template slot="title">
            <i aria-hidden="true" class="fa fa-comments fa-size"/>
            {{ $t('m.Discussion') }}
          </template>
          <el-menu-item index="/admin/discussion">
            {{$t('m.Discussion_Admin') }}
          </el-menu-item>
        </el-submenu>
      </el-menu>
      <div id="header">
        <el-row>
          <el-col :span="20">
            <div class="breadcrumb-container">
              <el-breadcrumb separator-class="el-icon-arrow-right">
                <el-breadcrumb-item :to="{ path: '/admin/' }">
                  {{$t('m.Home_Page') }}
                </el-breadcrumb-item>
                <el-breadcrumb-item v-for="item in routeList" :key="item.path">
                  {{ $t('m.' + item.meta.title.replaceAll(' ', '_')) }}
                </el-breadcrumb-item>
              </el-breadcrumb>
            </div>
          </el-col>
          <el-col v-show="isAuthenticated" :span="4">
            <avatar
                :inline="true"
                :size="30"
                :src="userInfo.avatar"
                :username="userInfo.username"
                class="drop-avatar"
                color="#FFF"
            ></avatar>
            <el-dropdown
                style="vertical-align: middle;"
                @command="handleCommand"
            >
              <span>
                {{ userInfo.username }}<i class="el-icon-caret-bottom el-icon--right"/>
              </span>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item command="logout">{{
                    $t('m.Logout')
                  }}
                </el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </el-col>
        </el-row>
      </div>
    </div>
    <div v-else>
      <mu-appbar class="mobile-nav" color="primary">
        <mu-button slot="left" icon @click="opendrawer = !opendrawer">
          <i class="el-icon-s-unfold"></i>
        </mu-button>
        {{
          websiteConfig.shortName
              ? websiteConfig.shortName.toUpperCase() + ' ADMIN'
              : 'ADMIN'
        }}
        <mu-menu
            v-show="isAuthenticated"
            slot="right"
            :open.sync="openusermenu"
        >
          <mu-button flat>
            {{ userInfo.username }}<i class="el-icon-caret-bottom"></i>
          </mu-button>
          <mu-list slot="content" @change="handleCommand">
            <mu-list-item button value="logout">
              <mu-list-item-content>
                <mu-list-item-title>{{ $t('m.Logout') }}</mu-list-item-title>
              </mu-list-item-content>
            </mu-list-item>
          </mu-list>
        </mu-menu>
      </mu-appbar>

      <mu-drawer :docked="false" :open.sync="opendrawer" :right="false">
        <mu-list toggle-nested>
          <mu-list-item
              :ripple="true"
              active-class="mobile-menu-active"
              button
              nested
              to="/admin/dashboard"
              @click="opendrawer = !opendrawer"
          >
            <mu-list-item-action>
              <mu-icon size="24" value="dashboard"></mu-icon>
            </mu-list-item-action>
            <mu-list-item-title>{{ $t('m.Dashboard') }}</mu-list-item-title>
          </mu-list-item>

          <mu-list-item
              v-if="isSuperAdmin"
              :open="openSideMenu === 'general'"
              :ripple="false"
              button
              nested
              @toggle-nested="openSideMenu = arguments[0] ? 'general' : ''">
            <mu-list-item-action>
              <mu-icon value="view_list"></mu-icon>
            </mu-list-item-action>
            <mu-list-item-title>{{ $t('m.General') }}</mu-list-item-title>
            <mu-list-item-action>
              <mu-icon
                  class="toggle-icon"
                  size="24"
                  value="keyboard_arrow_down"
              ></mu-icon>
            </mu-list-item-action>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/user"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{ $t('m.User_Admin') }}</mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/announcement"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Announcement') }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/notice"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{ $t('m.SysNotice') }}</mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/conf"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{
                  $t('m.System_Config')
                }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                button
                :ripple="false"
                slot="nested"
                to="/admin/switch"
                @click="opendrawer = !opendrawer"
                active-class="mobile-menu-active">
              <mu-list-item-title>{{
                  $t('m.System_Switch')
                }}</mu-list-item-title>
            </mu-list-item>
          </mu-list-item>

          <mu-list-item
              :open="openSideMenu === 'problem'"
              :ripple="false"
              button
              nested
              @toggle-nested="openSideMenu = arguments[0] ? 'problem' : ''">
            <mu-list-item-action>
              <mu-icon value="menu"/>
            </mu-list-item-action>
            <mu-list-item-title>{{ $t('m.Problem_Admin') }}</mu-list-item-title>
            <mu-list-item-action>
              <mu-icon
                  class="toggle-icon"
                  size="24"
                  value="keyboard_arrow_down"/>
            </mu-list-item-action>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/problems"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Problem_List') }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/problem/create"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Create_Problem') }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                v-if="isSuperAdmin || isProblemAdmin"
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/problem/tag"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{ $t('m.Admin_Tag') }}</mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                v-if="isSuperAdmin"
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/problem/batch-operation"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Export_Import_Problem') }}
              </mu-list-item-title>
            </mu-list-item>
          </mu-list-item>

          <mu-list-item
              :open="openSideMenu === 'team'"
              :ripple="false"
              button
              nested
              @toggle-nested="openSideMenu = arguments[0] ? 'team' : ''">
            <mu-list-item-action>
              <mu-icon value=":el-icon-user-solid fa-size"></mu-icon>
            </mu-list-item-action>
            <mu-list-item-title>
              {{$t('m.Team_Admin') }}
            </mu-list-item-title>
            <mu-list-item-action>
              <mu-icon
                  class="toggle-icon"
                  size="24"
                  value="keyboard_arrow_down"
              ></mu-icon>
            </mu-list-item-action>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/team"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Team_List') }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/team/create"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{
                  $t('m.Create_Team')
                }}
              </mu-list-item-title>
            </mu-list-item>

            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/training/category"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{
                  $t('m.Admin_Category')
                }}
              </mu-list-item-title>
            </mu-list-item>
          </mu-list-item>


          <mu-list-item
              :open="openSideMenu === 'training'"
              :ripple="false"
              button
              nested
              @toggle-nested="openSideMenu = arguments[0] ? 'training' : ''">
            <mu-list-item-action>
              <mu-icon value=":el-icon-s-claim fa-size"></mu-icon>
            </mu-list-item-action>
            <mu-list-item-title>
              {{$t('m.Training_Admin') }}
            </mu-list-item-title>
            <mu-list-item-action>
              <mu-icon
                  class="toggle-icon"
                  size="24"
                  value="keyboard_arrow_down"
              ></mu-icon>
            </mu-list-item-action>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/training"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Training_List') }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/training/create"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{
                  $t('m.Create_Training')
                }}
              </mu-list-item-title>
            </mu-list-item>

            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/training/category"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>{{
                  $t('m.Admin_Category')
                }}
              </mu-list-item-title>
            </mu-list-item>
          </mu-list-item>

          <mu-list-item
              :open="openSideMenu === 'contest'"
              :ripple="false"
              button
              nested
              @toggle-nested="openSideMenu = arguments[0] ? 'contest' : ''">
            <mu-list-item-action>
              <mu-icon value=":fa fa-trophy fa-size"></mu-icon>
            </mu-list-item-action>
            <mu-list-item-title>{{ $t('m.Contest_Admin') }}</mu-list-item-title>
            <mu-list-item-action>
              <mu-icon
                  class="toggle-icon"
                  size="24"
                  value="keyboard_arrow_down"
              ></mu-icon>
            </mu-list-item-action>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/contest"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Contest_List') }}
              </mu-list-item-title>
            </mu-list-item>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/contest/create"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Create_Contest') }}
              </mu-list-item-title>
            </mu-list-item>
          </mu-list-item>

          <mu-list-item
              :open="openSideMenu === 'discussion'"
              :ripple="false"
              button
              nested
              @toggle-nested="openSideMenu = arguments[0] ? 'discussion' : ''">
            <mu-list-item-action>
              <mu-icon value=":fa fa-comments fa-size"></mu-icon>
            </mu-list-item-action>
            <mu-list-item-title>{{ $t('m.Discussion') }}</mu-list-item-title>
            <mu-list-item-action>
              <mu-icon
                  class="toggle-icon"
                  size="24"
                  value="keyboard_arrow_down"
              ></mu-icon>
            </mu-list-item-action>
            <mu-list-item
                slot="nested"
                :ripple="false"
                active-class="mobile-menu-active"
                button
                to="/admin/discussion"
                @click="opendrawer = !opendrawer">
              <mu-list-item-title>
                {{$t('m.Discussion_Admin') }}
              </mu-list-item-title>
            </mu-list-item>
          </mu-list-item>
        </mu-list>
      </mu-drawer>
    </div>
  </div>
</template>

<script>
import {mapGetters} from 'vuex';
import api from '@/common/api';
import mMessage from '@/common/message';
import Avatar from 'vue-avatar';

export default {
  name: 'NavBar',
  mounted() {
    this.currentPath = this.$route.path;
    this.getBreadcrumb();
    window.onresize = () => {
      this.page_width();
    };
    this.page_width();
  },
  data() {
    return {
      openusermenu: false,
      openSideMenu: '',
      // katexVisible: false,
      opendrawer: false,
      mobileNar: false,
      currentPath: '',
      routeList: [],
      imgUrl: require('@/assets/management.svg'),
    };
  },
  components: {
    Avatar,
  },
  methods: {
    handleCommand(command) {
      if (command === 'logout') {
        api.logout().then((res) => {
          this.$router.push({path: '/admin/login'});
          mMessage.success(res.data.msg);
          this.$store.commit('clearUserInfoAndToken');
        });
      }
    },
    page_width() {
      let screenWidth = window.screen.width;
      if (screenWidth < 1150) {
        this.mobileNar = true;
      } else {
        this.mobileNar = false;
      }
    },
    getBreadcrumb() {
      let matched = this.$route.matched.filter((item) => item.meta.title); //获取路由信息，并过滤保留路由标题信息存入数组
      this.routeList = matched;
    }
  },
  computed: {
    ...mapGetters([
      'userInfo',
      'isSuperAdmin',
      'isProblemAdmin',
      'isAuthenticated',
      'websiteConfig',
    ]),
    'window.screen.width'(newVal, oldVal) {
      if (newVal < 1150) {
        this.mobileNar = true;
      } else {
        this.mobileNar = false;
      }
    },
  },
  watch: {
    $route() {
      this.getBreadcrumb(); //监听路由变化
    },
  },
};
</script>

<style scoped>
.vertical_menu {
  overflow: auto;
  width: 205px;
  height: 100%;
  position: fixed !important;
  z-index: 100;
  top: 0;
  bottom: 0;
  left: 0;
}

.vertical_menu .logo {
  margin: 10px 0;
  text-align: center;
}

.vertical_menu .logo img {
  width: 100px;
  height: 100px;
}

.fa-size {
  text-align: center;
  font-size: 18px;
  vertical-align: middle;
  margin-right: 5px;
  width: 24px;
}

a {
  background-color: transparent;
}

a:active,
a:hover {
  outline-width: 0;
}

img {
  border-style: none;
}

.breadcrumb-container {
  padding: 17px;
  background-color: #fff;
}

* {
  box-sizing: border-box;
}

#header {
  text-align: right;
  padding-left: 210px;
  padding-right: 30px;
  line-height: 50px;
  height: 50px;
  background: #f9fafc;
}

.drop-avatar {
  vertical-align: middle;
  margin-right: 10px;
}
</style>
