<template>
  <div id="app">
    <div v-if="!isAdminView">
      <el-backtop :right="10"></el-backtop>
      <NavBar></NavBar>
      <div id="oj-content">
        <transition name="el-collapse-transition">
          <router-view></router-view>
        </transition>
      </div>
      <footer>
        <div class="mundb-footer">
          <a :href="websiteConfig.recordUrl" target="_blank">{{ websiteConfig.recordName }}</a>
          <span style="margin-left:10px">
            <el-dropdown placement="top" @command="changeLanguage">
              <span class="el-dropdown-link">
                <i aria-hidden="true" class="fa fa-globe">
                  {{ this.webLanguage == 'zh-CN' ? '简体中文' : 'English' }}</i>
                <i class="el-icon-arrow-up el-icon--right"/>
              </span>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item command="zh-CN">简体中文</el-dropdown-item>
                <el-dropdown-item command="en-US">English</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </span>
        </div>
      </footer>
    </div>
    <div v-else>
      <div id="admin-content">
        <transition name="el-collapse-transition">
          <router-view></router-view>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
import NavBar from '@/components/oj/common/NavBar';
import {mapActions, mapGetters, mapState} from 'vuex';
import {LOGO, MOTTO} from '@/common/logo';

export default {
  name: 'App',
  components: {
    NavBar,
  },
  data() {
    return {
      isAdminView: false
    };
  },
  methods: {
    ...mapActions(['changeDomTitle', 'getWebsiteConfig']),
    goRoute(path) {
      this.$router.push({
        path: path,
      });
    },
    toUpper(str) {
      if (str) {
        return str.toUpperCase();
      }
    },
    changeLanguage(language) {
      this.$store.commit('changeWebLanguage', {language: language});
    },
  },
  watch: {
    $route(to, from) {
      this.changeDomTitle();
      if (to !== from && to.path.split('/')[1] == 'admin') {
        this.isAdminView = true;
      } else {
        this.isAdminView = false;
      }
    },
    websiteConfig() {
      this.changeDomTitle();
    },
  },
  computed: {
    ...mapState(['websiteConfig']),
    ...mapGetters(['webLanguage']),
  },
  created: function () {
    this.$nextTick(function () {
      try {
        document.body.removeChild(document.getElementById('app-loader'));
      } catch (e) {
      }
    });

    if (this.$route.path.split('/')[1] != 'admin') {
      this.isAdminView = false;
    } else {
      this.isAdminView = true;
    }
  },
  mounted() {
    this.getWebsiteConfig();
  },
};
</script>

<style>
* {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

body {
  background-color: #eff3f5 !important;
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB',
  'Microsoft YaHei', '微软雅黑', Arial, sans-serif !important;
  color: #495060 !important;
  font-size: 12px !important;
}

code,
kbd,
pre,
samp {
  font-family: Consolas, Menlo, Courier, monospace;
}

::-webkit-scrollbar {
  width: 10px;
  height: 12px;
  background-color: #fff;
}

::-webkit-scrollbar-thumb {
  display: block;
  min-height: 12px;
  min-width: 10px;
  border-radius: 6px;
  background-color: rgb(217, 217, 217);
}

::-webkit-scrollbar-thumb:hover {
  display: block;
  min-height: 12px;
  min-width: 10px;
  border-radius: 6px;
  background-color: rgb(159, 159, 159);
}

#admin-content {
  background-color: #1e9fff;
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}

.mobile-menu-active {
  background-color: rgba(0, 0, 0, 0.1);
}

.mobile-menu-active .mu-item-title {
  color: #2d8cf0 !important;
}

.mobile-menu-active .mu-icon {
  color: #2d8cf0 !important;
}

#particles-js {
  position: fixed;
  z-index: 0;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

a {
  text-decoration: none;
  background-color: transparent;
  color: #495060;
  outline: 0;
  cursor: pointer;
  transition: color 0.2s ease;
}

a:hover {
  color: #2196f3 !important;
}

.markdown-body a {
  color: #2196f3;
  cursor: pointer;
  text-decoration: none;
  transition: all 0.28s ease;
  -moz-transition: all 0.28s ease;
  -webkit-transition: all 0.28s ease;
  -o-transition: all 0.28s ease;
}

.markdown-body a:hover {
  color: #ff5722 !important;
  text-decoration: underline;
}

.panel-title {
  font-size: 21px;
  font-weight: 500;
  padding-top: 10px;
  padding-bottom: 20px;
  line-height: 30px;
}

.home-title {
  color: #409eff;
  font-family: 'Raleway';
}

.contest-username {
  display: block;
  overflow: hidden;
  color: black;
  font-size: 13.5px;
  font-weight: 550;
}

.contest-school {
  font-size: 12px;
  font-weight: normal;
  color: dimgrey;
}

.contest-rank-flag {
  margin-right: 20px !important;
  background-color: rgb(255, 193, 10);
  border-radius: 4px;
  color: rgb(73, 36, 0);
  padding: 1px 3px !important;
}

.bg-female {
  background-color: rgb(255, 153, 203);
}

.bg-star {
  background-color: #ffffcc;
}

.bg-concerned {
  background-color: lightyellow;
}

.contest-rank-balloon {
  vertical-align: top;
  margin-left: -10px !important;
  margin-right: -7px !important;
}

.oi-100 {
  background-color: #19be6b;
  color: #fff;
  font-weight: 700;
}

.oi-0 {
  color: #a94442;
  background-color: #f2dede;
}

.oi-between {
  background-color: #2d8cf0;
  color: #fff;
}

.first-ac {
  background-color: #1daa1d;
}

.ac {
  background-color: #60e760;
}

.wa {
  background-color: #e87272;
}

.try {
  background-color: #ff9800;
}

.status-green {
  background-color: #19be6b !important;
  color: #fff !important;
}

.status-red {
  background-color: #ed3f14 !important;
  color: #fff !important;
}

.status-yellow {
  background-color: #f90 !important;
  color: #fff !important;
}

.status-blue {
  background-color: #2d8cf0 !important;
  color: #fff !important;
}

.status-gray {
  background-color: #909399 !important;
  color: #fff !important;
}

.own-submit-row {
  background: rgb(230, 255, 223) !important;
}

.unregistered {
  background-color: #C0C4CC !important;
  color: #fff !important;
}

.submission-hover:hover {
  cursor: pointer;
}

.vxe-table {
  color: #000 !important;
  font-size: 12px !important;
  font-weight: 500 !important;
}

.row--hover {
  cursor: pointer;
  background-color: #ebf7ff !important;
}

.vxe-table .vxe-body--column:not(.col--ellipsis),
.vxe-table .vxe-footer--column:not(.col--ellipsis),
.vxe-table .vxe-header--column:not(.col--ellipsis) {
  padding: 9px 0 !important;
}

#nprogress .bar {
  background: #66b1ff !important;
}

@media screen and (min-width: 1050px) {
  #oj-content {
    margin-top: 80px;
    padding: 0 3%;
  }
}

.markdown-body img {
  max-width: 100%;
}

.contest-description img {
  max-width: 100%;
}

@media screen and (max-width: 1050px) {
  #oj-content {
    margin-top: 20px;
    padding: 0 5px;
  }

  .el-row {
    margin-left: 0px !important;
    margin-right: 0px !important;
  }

  .el-col {
    padding-left: 0px !important;
    padding-right: 0px !important;
  }

  .el-message-box {
    width: 70% !important;
  }
}

#problem-content .sample pre {
  -ms-flex: 1 1 auto;
  flex: 1 1 auto;
  -ms-flex-item-align: stretch;
  align-self: stretch;
  border-style: solid;
  background: #fafafa;
  border-left: 2px solid #3498db;
}

.markdown-body pre {
  padding: 5px 10px;
  white-space: pre-wrap;
  margin-top: 15px;
  margin-bottom: 15px;
  background: #f8f8f9;
  border: 1px dashed #e9eaec;
}

.el-menu--popup {
  min-width: 120px !important;
  text-align: center;
}

.panel-options {
  margin-top: 10px;
  text-align: center;
}

.el-tag--dark {
  border-color: #fff !important;
}

.v-note-wrapper .v-note-panel {
  height: 460px !important;
}

.tex-formula {
  font-family: times new roman, sans-serif;
  vertical-align: middle;
  margin: 0;
  border: medium none;
  position: relative;
  bottom: 2px;
}

.tex-span {
  font-size: 125%;
  font-family: times new roman, sans-serif;
  white-space: nowrap;
}

.tex-font-size-tiny {
  font-size: 70%;
}

.tex-font-size-script {
  font-size: 75%;
}

.tex-font-size-footnotes {
  font-size: 85%;
}

.tex-font-size-small {
  font-size: 85%;
}

.tex-font-size-normal {
  font-size: 100%;
}

.tex-font-size-large-1 {
  font-size: 115%;
}

.tex-font-size-large-2 {
  font-size: 130%;
}

.tex-font-size-large-3 {
  font-size: 145%;
}

.tex-font-size-huge-1 {
  font-size: 175%;
}

.tex-font-size-huge-2 {
  font-size: 200%;
}

.tex-font-style-sf {
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}

.tex-font-style-tt {
  font-size: 110%;
  font-family: courier new, monospace;
}

.tex-font-style-bf {
  font-weight: bold;
}

.tex-font-style-it {
  font-style: italic;
}

.tex-font-style-sl {
  font-style: italic;
}

.tex-font-style-sc {
  text-transform: uppercase;
}

.tex-font-style-striked {
  text-decoration: line-through;
}

.tex-font-style-underline {
  text-decoration: underline;
}

.tex-graphics {
  display: block;
}

footer {
  margin-top: 2rem;
  color: #555 !important;
  background-color: #fff;
  text-align: center;
}

footer a {
  color: #555;
}

footer a:hover {
  color: #409eff;
  text-decoration: none;
}

footer h1 {
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, PingFang SC,
  Hiragino Sans GB, Microsoft YaHei, Helvetica Neue, Helvetica, Arial,
  sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol;
  font-weight: 300;
  color: #3d3d3d;
  line-height: 1.1;
  font-size: 1.5rem;
}

.mundb-footer {
  padding: 1rem 2.5rem;
  width: 100%;
  font-weight: 400;
  font-size: 1rem;
  line-height: 1;
}

@media (min-width: 768px) {
  .hr-none {
    display: none !important;
  }
}

.el-empty {
  max-width: 256px;
  margin: 0 auto;
}

.el-empty__description {
  text-align: center;
  color: #3498db;
  font-size: 13px;
}
</style>
<style>
.markdown-body pre {
  display: block;
  border-radius: 3px !important;
  border: 1px solid #c3ccd0;
  padding: 0 16px 0 50px !important;
  position: relative !important;
  overflow-y: hidden !important;
  font-size: 1rem !important;
  background: #f8f8f9 !important;
  white-space: pre !important;
}

.markdown-body pre code {
  line-height: 26px !important;
}

.markdown-body pre ol.pre-numbering {
  position: absolute;
  top: 0;
  left: 0;
  line-height: 26px;
  margin: 0;
  padding: 0;
  list-style-type: none;
  counter-reset: sectioncounter;
  background: #f1f1f1;
  color: #777;
  font-size: 12px;
}

.markdown-body pre ol.pre-numbering li {
  margin-top: 0 !important;
}

.markdown-body pre ol.pre-numbering li:before {
  content: counter(sectioncounter) '';
  counter-increment: sectioncounter;
  display: inline-block;
  width: 40px;
  text-align: center;
}

.markdown-body pre i.code-copy {
  position: absolute;
  top: 0;
  right: 0;
  background-color: #2196f3;
  display: none;
  padding: 5px;
  margin: 5px 5px 0 0;
  font-size: 11px;
  border-radius: inherit;
  color: #fff;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
}

.markdown-body pre:hover i.code-copy {
  display: block;
}

.markdown-body pre i.code-copy:hover i.code-copy {
  display: block;
}

.markdown-body blockquote {
  color: #666;
  border-left: 4px solid #8bc34a;
  padding: 10px;
  margin-left: 0;
  font-size: 14px;
  background: #f8f8f8;
}

.markdown-body h1,
.markdown-body h2,
.markdown-body h3,
.markdown-body h4,
.markdown-body h5,
.markdown-body h6 {
  position: relative;
  margin-top: 1em;
  margin-bottom: 16px;
  font-weight: bold;
  line-height: 1.4;
}

.markdown-body h1 {
  padding-bottom: 0.3em;
  font-size: 2.25em;
  line-height: 1.2;
  border-bottom: 1px solid #eee;
}

.markdown-body h2 {
  font-size: 1.45em;
  line-height: 1.425;
  border-bottom: 1px solid #eee;
  background: #cce5ff;
  padding: 8px 10px;
  color: #545857;
  border-radius: 3px;
}

.markdown-body h3 {
  font-size: 1.3em;
  line-height: 1.43;
}

.markdown-body h3:before {
  content: '';
  border-left: 4px solid #03a9f4;
  padding-left: 6px;
}

.markdown-body h4 {
  font-size: 1.12em;
}

.markdown-body h4:before {
  content: '';
  border-left: 4px solid #bbb;
  padding-left: 6px;
}

.markdown-body img {
  border: 0;
  background: #ffffff;
  padding: 15px;
  margin: 5px 0;
  box-shadow: inset 0 0 12px rgb(219 219 219);
}

.markdown-body p {
  font-size: 15px;
  word-wrap: break-word;
  word-break: break-word;
  line-height: 1.8;
}

.hljs {
  padding: 0 !important;
}
</style>
