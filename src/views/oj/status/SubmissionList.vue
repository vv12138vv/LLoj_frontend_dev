<template>
  <el-row class="flex-container">
    <div id="main">
      <el-card shadow>
        <div slot="header">
          <el-row :gutter="18">
            <el-col :lg="2" :md="4">
              <span class="panel-title hidden-md-and-down">{{
                  $t('m.Status')
                }}</span>
            </el-col>
            <el-col :lg="4" :md="4" :sm="8" :xs="10">
              <el-switch
                  v-model="formFilter.onlyMine"
                  :active-text="$t('m.Mine')"
                  :inactive-text="$t('m.All')"
                  :width="40"
                  style="display: block"
                  @change="handleOnlyMine"
              >
              </el-switch>
            </el-col>

            <el-col :lg="4" :md="5" :sm="8" :xs="10" style="padding-top: 5px;">
              <el-dropdown
                  class="drop-menu"
                  placement="bottom"
                  trigger="hover"
                  @command="handleStatusChange"
              >
                <span class="el-dropdown-link">
                  {{ status }}
                  <i class="el-icon-caret-bottom"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="All">{{
                      $t('m.All')
                    }}
                  </el-dropdown-item>
                  <el-dropdown-item
                      v-for="result in Object.keys(JUDGE_STATUS_LIST)"
                      :key="result"
                      :command="result"
                  >
                    {{ JUDGE_STATUS_LIST[result].name }}
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </el-col>

            <el-col :lg="4" :md="5" :sm="8" class="hidden-xs-only">
              <el-button
                  icon="el-icon-refresh"
                  round
                  size="small"
                  type="primary"
                  @click="getSubmissions"
              >{{ $t('m.Refresh') }}
              </el-button
              >
            </el-col>
            <el-col :xs="4" class="hidden-sm-and-up">
              <el-button
                  circle
                  icon="el-icon-refresh"
                  size="small"
                  type="primary"
                  @click="getSubmissions"
              ></el-button>
            </el-col>

            <el-col :lg="5" :md="5" :sm="12" :xs="24" class="search">
              <vxe-input
                  v-model="formFilter.problemID"
                  :placeholder="$t('m.Enter_Problem_ID')"
                  size="medium"
                  type="search"
                  @keyup.enter.native="handleQueryChange('problemID')"
                  @search-click="handleQueryChange('problemID')"
              ></vxe-input>
            </el-col>
            <el-col :lg="5" :md="5" :sm="12" :xs="24" class="search">
              <vxe-input
                  v-model="formFilter.username"
                  :disabled="formFilter.onlyMine"
                  :placeholder="$t('m.Enter_Author')"
                  size="medium"
                  type="search"
                  @keyup.enter.native="handleQueryChange('username')"
                  @search-click="handleQueryChange('username')"
              ></vxe-input>
            </el-col>
          </el-row>
        </div>
        <vxe-table
            ref="xTable"
            :data="submissions"
            :loading="loadingTable"
            :row-class-name="tableRowClassName"
            align="center"
            auto-resize
            border="inner"
            highlight-current-row
            highlight-hover-row
            keep-source
            stripe
        >
          <vxe-table-column
              :title="$t('m.Run_ID')"
              field="submitId"
              min-width="96"
          ></vxe-table-column>
          <vxe-table-column
              :title="$t('m.Problem')"
              field="pid"
              min-width="150"
              show-overflow
          >
            <template v-slot="{ row }">
              <span
                  v-if="contestID"
                  style="color: rgb(87, 163, 243)"
                  @click="getProblemUri(row.displayId, true)"
              >{{ row.displayId + ' ' + row.title }}
              </span>
              <span
                  v-else
                  style="color: rgb(87, 163, 243)"
                  @click="getProblemUri(row.displayPid, false)"
              >{{ row.displayPid + ' ' + row.title }}
              </span>
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Status')"
              field="status"
              min-width="160"
          >
            <template v-slot="{ row }">
              <span :class="getStatusColor(row.status)">
                <i
                    v-if="
                    row.status == JUDGE_STATUS_RESERVE['Pending'] ||
                      row.status == JUDGE_STATUS_RESERVE['Compiling'] ||
                      row.status == JUDGE_STATUS_RESERVE['Judging']
                  "
                    class="el-icon-loading"
                ></i>
                <i
                    v-if="
                    row.status == JUDGE_STATUS_RESERVE['sf'] &&
                      row.uid == userInfo.uid
                  "
                    class="el-icon-refresh"
                    @click="reSubmit(row)"
                ></i>
                {{ JUDGE_STATUS[row.status].name }}
              </span>
            </template>
          </vxe-table-column>
          <vxe-table-column
              v-if="scoreColumnVisible"
              :title="$t('m.Score')"
              field="score"
              min-width="64"
          >
            <template v-slot="{ row }">
              <template v-if="contestID && row.score != null">
                <el-tag
                    :type="JUDGE_STATUS[row.status]['type']"
                    effect="plain"
                    size="medium"
                >{{ row.score }}
                </el-tag>
              </template>
              <template v-else-if="row.score != null">
                <el-tag
                    :type="JUDGE_STATUS[row.status]['type']"
                    effect="plain"
                    size="medium"
                >{{ row.score }}
                </el-tag>
<!--                <el-tooltip placement="top">-->
<!--                  <div slot="content">-->
<!--                    {{ $t('m.Problem_Score') }}：{{-->
<!--                      row.score != null ? row.score : $t('m.Unknown')-->
<!--                    }}<br/>{{ $t('m.OI_Rank_Score') }}：{{-->
<!--                      row.oiRankScore != null-->
<!--                          ? row.oiRankScore-->
<!--                          : $t('m.Unknown')-->
<!--                    }}<br/>-->
<!--                    {{-->
<!--                      $t('m.OI_Rank_Calculation_Rule')-->
<!--                    }}：(score*0.1+difficulty*2)*(ac_cases/sum_cases)-->
<!--                  </div>-->
<!--                  <el-tag-->
<!--                      :type="JUDGE_STATUS[row.status]['type']"-->
<!--                      effect="plain"-->
<!--                      size="medium"-->
<!--                  >{{ row.score }}-->
<!--                  </el-tag>-->
<!--                </el-tooltip>-->
              </template>
              <template
                  v-else-if="
                  row.status == JUDGE_STATUS_RESERVE['Pending'] ||
                    row.status == JUDGE_STATUS_RESERVE['Compiling'] ||
                    row.status == JUDGE_STATUS_RESERVE['Judging']
                "
              >
                <el-tag
                    :type="JUDGE_STATUS[row.status]['type']"
                    effect="plain"
                    size="medium"
                >
                  <i class="el-icon-loading"></i>
                </el-tag>
              </template>
              <template v-else>
                <el-tag
                    :type="JUDGE_STATUS[row.status]['type']"
                    effect="plain"
                    size="medium"
                >--
                </el-tag>
              </template>
            </template>
          </vxe-table-column>
          <vxe-table-column :title="$t('m.Time')" field="time" min-width="90">
            <template v-slot="{ row }">
              <span>{{ submissionTimeFormat(row.time) }}</span>
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Memory')"
              field="memory"
              min-width="90"
          >
            <template v-slot="{ row }">
              <span>{{ submissionMemoryFormat(row.memory) }}</span>
            </template>
          </vxe-table-column>

          <vxe-table-column
              :title="$t('m.Length')"
              field="length"
              min-width="90"
          >
            <template v-slot="{ row }">
              <span>{{ submissionLengthFormat(row.length) }}</span>
            </template>
          </vxe-table-column>

          <vxe-table-column
              :title="$t('m.Language')"
              field="language"
              min-width="90"
              show-overflow
          >
            <template v-slot="{ row }">
              <el-tooltip
                  :content="$t('m.View_submission_details')"
                  class="item"
                  effect="dark"
                  placement="top"
              >
                <el-button type="text" @click="showSubmitDetail(row)">{{
                    row.language
                  }}
                </el-button>
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Judger')"
              field="judger"
              min-width="90"
              show-overflow
          >
            <template v-slot="{ row }">
              <span v-if="row.judger">{{ row.judger }}</span>
              <span v-else>--</span>
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Author')"
              field="username"
              min-width="90"
              show-overflow
          >
            <template v-slot="{ row }">
              <a
                  style="color: rgb(87, 163, 243)"
                  @click="goUserHome(row.username, row.uid)"
              >{{ row.username }}</a
              >
            </template>
          </vxe-table-column>
          <vxe-table-column
              v-if="contestID && isAdminRole"
              :title="$t('m.RealName')"
              field="realname"
              min-width="90"
              show-overflow
          >
            <template v-slot="{ row }">
              {{ row.realname }}
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Submit_Time')"
              field="submitTime"
              min-width="90"
          >
            <template v-slot="{ row }">
              <span
              ><el-tooltip
                  :content="row.submitTime | localtime"
                  placement="top"
              >
                  <span>{{ row.submitTime | fromNow }}</span>
                </el-tooltip></span
              >
            </template>
          </vxe-table-column>
          <!-- 非比赛提交记录，超级管理员可以对提交进行重判 -->
          <vxe-table-column
              v-if="isSuperAdmin"
              :title="$t('m.Option')"
              min-width="90"
          >
            <template v-slot="{ row }">
              <vxe-button
                  :loading="row.loading"
                  size="mini"
                  status="primary"
                  @click="handleRejudge(row)"
              >{{ $t('m.Rejudge') }}
              </vxe-button
              >
            </template>
          </vxe-table-column>
        </vxe-table>
      </el-card>
      <Pagination
          :current.sync="currentPage"
          :layout="'prev, pager, next, sizes'"
          :page-size="limit"
          :total="total"
          @on-change="changeRoute"
          @on-page-size-change="onPageSizeChange"
      ></Pagination>
    </div>
  </el-row>
</template>

<script>
import {mapActions, mapGetters} from 'vuex';
import api from '@/common/api';
import {CONTEST_STATUS, JUDGE_STATUS, JUDGE_STATUS_RESERVE, RULE_TYPE,} from '@/common/constants';
import utils from '@/common/utils';
import Pagination from '@/components/oj/common/Pagination';
import myMessage from '@/common/message';
import 'element-ui/lib/theme-chalk/display.css';

export default {
  name: 'SubmissionList',
  components: {
    Pagination,
  },
  data() {
    return {
      formFilter: {
        onlyMine: false,
        status: '',
        username: '',
        problemID: '',
      },
      loadingTable: false,
      submissions: [],
      needCheckSubmitIds: {}, // 当前状态为6和7的提交记录Id 需要重新检查更新
      total: 30,
      limit: 15,
      currentPage: 1,
      contestID: '',
      routeName: '',
      checkStatusNum: 0,
      JUDGE_STATUS: '',
      JUDGE_STATUS_LIST: '',
      autoCheckOpen: false,
      JUDGE_STATUS_RESERVE: {},
      CONTEST_STATUS: {},
      RULE_TYPE: {},
    };
  },
  created() {
    this.init();
  },
  mounted() {
    this.JUDGE_STATUS = Object.assign({}, JUDGE_STATUS);
    this.JUDGE_STATUS_LIST = Object.assign({}, JUDGE_STATUS);
    this.JUDGE_STATUS_RESERVE = Object.assign({}, JUDGE_STATUS_RESERVE);
    this.CONTEST_STATUS = Object.assign({}, CONTEST_STATUS);
    this.RULE_TYPE = Object.assign({}, RULE_TYPE);
    // 去除下拉框选择中的Compiling,Judging,Submitting,Not Submitted,Submitted Unknown Result 五种状态
    delete this.JUDGE_STATUS_LIST['6'];
    delete this.JUDGE_STATUS_LIST['7'];
    delete this.JUDGE_STATUS_LIST['9'];
    delete this.JUDGE_STATUS_LIST['-5'];
    delete this.JUDGE_STATUS_LIST['-10'];
    this.getData();
  },
  methods: {
    init() {
      this.checkStatusNum = 0;
      this.contestID = this.$route.params.contestID;
      let query = this.$route.query;
      this.formFilter.problemID = query.problemID;
      this.formFilter.username = query.username || '';
      this.formFilter.onlyMine = query.onlyMine + '' == 'true'; // 统一换成字符串判断
      this.formFilter.status = query.status;
      this.formFilter.completeProblemID = query.completeProblemID || false;
      if (this.formFilter.onlyMine) {
        // 当前为搜索自己的提交 那么不可搜索别人的提交
        this.formFilter.username = '';
      }
      this.currentPage = parseInt(query.currentPage) || 1;
      if (this.currentPage < 1) {
        this.currentPage = 1;
      }
      this.limit = parseInt(query.limit) || 15;
      this.routeName = this.$route.name;
    },

    getData() {
      this.getSubmissions();
    },

    buildQuery() {
      return {
        onlyMine: this.formFilter.onlyMine,
        status: this.formFilter.status,
        username: this.formFilter.username,
        problemID: this.formFilter.problemID,
        currentPage: this.currentPage,
        limit: this.limit,
        completeProblemID: this.formFilter.completeProblemID,
      };
    },

    submissionTimeFormat(time) {
      return utils.submissionTimeFormat(time);
    },

    submissionMemoryFormat(memory) {
      return utils.submissionMemoryFormat(memory);
    },

    submissionLengthFormat(length) {
      return utils.submissionLengthFormat(length);
    },
    reSubmit(row) {
      api.reSubmitRemoteJudge(row.submitId).then((res) => {
        let xTable = this.$refs.xTable;
        // 重新提交开始，需要将该提交的部分参数初始化
        row.status = res.data.data.status;
        row.time = res.data.data.time;
        row.memory = res.data.data.memory;
        row.errorMessage = res.data.data.errorMessage;
        row.judger = res.data.data.judger;
        // 重新加载该行数据到view
        xTable.reloadRow(row, null, null);

        this.submissions[row.index] = res.data.data;
        myMessage.success(res.data.msg);

        // 加入待重判列表
        this.needCheckSubmitIds[row.submitId] = row.index;

        this.checkStatusNum = 0;
        if (!this.autoCheckOpen) {
          // 如果当前未开启自动检查提交状态的定时任务，则开启
          this.checkSubmissionsStatus();
        }
      });
    },
    getSubmissions() {
      let params = this.buildQuery();
      params.contestID = this.contestID;
      if (this.contestID) {
        if (this.contestStatus == CONTEST_STATUS.SCHEDULED) {
          params.beforeContestSubmit = true;
        } else {
          params.beforeContestSubmit = false;
        }
      }
      if (this.formFilter.onlyMine) {
        // 需要判断是否为登陆状态
        if (this.isAuthenticated) {
          params.username = ''; // 如果是搜索当前用户的提交记录，那么用户名搜索应该无效
          this.formFilter.username = '';
        } else {
          this.formFilter.onlyMine = false;
          myMessage.error(this.$i18n.t('m.Please_login_first'));
          return;
        }
      }

      this.loadingTable = true;
      this.submissions = [];
      this.needCheckSubmitIds = {};
      let func = this.contestID
          ? 'getContestSubmissionList'
          : 'getSubmissionList';
      api[func](this.limit, utils.filterEmptyValue(params))
          .then((res) => {
            let data = res.data.data;
            let index = 0;
            for (let v of data.records) {
              if (
                  v.status == JUDGE_STATUS_RESERVE['Pending'] ||
                  v.status == JUDGE_STATUS_RESERVE['Compiling'] ||
                  v.status == JUDGE_STATUS_RESERVE['Judging']
              ) {
                this.needCheckSubmitIds[v.submitId] = index;
              }
              v.loading = false;
              v.index = index;
              index += 1;
            }
            this.loadingTable = false;
            this.submissions = data.records;
            this.total = data.total;
            if (Object.keys(this.needCheckSubmitIds).length > 0) {
              this.checkSubmissionsStatus();
            }
          })
          .catch(() => {
            this.loadingTable = false;
          });
    },
    // 对当前提交列表 状态为Pending（6）和Judging（7）的提交记录每2秒查询一下最新结果
    checkSubmissionsStatus() {
      // 使用setTimeout避免一些问题
      if (this.refreshStatus) {
        // 如果之前的提交状态检查还没有停止, 则停止, 否则将会失去timeout的引用造成无限请求
        clearTimeout(this.refreshStatus);
        this.autoCheckOpen = false;
      }
      const checkStatus = () => {
        let submitIds = this.needCheckSubmitIds;
        let func = this.contestID
            ? 'checkContestSubmissionsStatus'
            : 'checkSubmissionsStatus';
        api[func](Object.keys(submitIds), this.contestID).then(
            (res) => {
              let result = res.data.data;
              if (!this.$refs.xTable) {
                // 避免请求一半退出view保错
                return;
              }
              let viewData = this.$refs.xTable.getTableData().tableData;
              for (let key in submitIds) {
                let submitId = parseInt(key);
                if (!result[submitId]) {
                  continue;
                }
                // 更新数据列表
                this.submissions[submitIds[key]] = result[submitId];
                // 更新view中的结果，f分数，耗时，空间消耗，判题机ip
                viewData[submitIds[key]].status = result[submitId].status;
                viewData[submitIds[key]].score = result[submitId].score;
                viewData[submitIds[key]].time = result[submitId].time;
                viewData[submitIds[key]].memory = result[submitId].memory;
                viewData[submitIds[key]].judger = result[submitId].judger;
                // 重新加载这行数据到view中
                this.$refs.xTable.reloadRow(viewData[submitIds[key]], null, null);

                if (
                    result[submitId].status != JUDGE_STATUS_RESERVE['Pending'] &&
                    result[submitId].status != JUDGE_STATUS_RESERVE['Compiling'] &&
                    result[submitId].status != JUDGE_STATUS_RESERVE['Judging']
                ) {
                  delete this.needCheckSubmitIds[key];
                }
              }
              // 当前提交列表的提交都判题结束或者检查结果60s（2s*30）还没判题结束，为了避免无用请求加重服务器负担，直接停止检查的请求。
              if (
                  Object.keys(this.needCheckSubmitIds).length == 0 ||
                  this.checkStatusNum == 30
              ) {
                clearTimeout(this.refreshStatus);
                this.autoCheckOpen = false;
              } else {
                this.checkStatusNum += 1;
                this.refreshStatus = setTimeout(checkStatus, 2000);
              }
            },
            (res) => {
              clearTimeout(this.refreshStatus);
              this.autoCheckOpen = false;
            }
        );
      };
      // 设置每2秒检查一下提交结果
      this.checkStatusNum += 1;
      this.refreshStatus = setTimeout(checkStatus, 2000);
      this.autoCheckOpen = true;
    },
    onPageSizeChange(pageSize) {
      this.limit = pageSize;
      this.changeRoute();
    },
    // 改变route， 通过监听route变化请求数据，这样可以产生route history， 用户返回时就会保存之前的状态
    changeRoute() {
      let query = this.buildQuery();
      query.contestID = this.contestID;
      let queryParams = utils.filterEmptyValue(query);
      // 判断新路径请求参数与当前路径请求的参数是否一致，避免重复访问路由报错
      let equal = true;
      for (let key in queryParams) {
        if (queryParams[key] != this.$route.query[key]) {
          equal = false;
          break;
        }
      }
      if (equal) {
        // 判断请求参数的长短
        if (
            Object.keys(queryParams).length !=
            Object.keys(this.$route.query).length
        ) {
          equal = false;
        }
      }

      if (!equal) {
        // 避免重复同个路径请求导致报错
        let routeName = queryParams.contestID
            ? 'ContestSubmissionList'
            : 'SubmissionList';
        this.$router.push({
          name: routeName,
          query: queryParams,
        });
      }
    },
    goRoute(route) {
      this.$router.push(route);
    },
    goUserHome(username, uid) {
      this.$router.push({
        path: '/user-home',
        query: {uid, username},
      });
    },
    handleStatusChange(status) {
      if (status == 'All') {
        this.formFilter.status = '';
      } else {
        this.formFilter.status = status;
      }
      this.currentPage = 1;
      this.changeRoute();
    },
    handleQueryChange(searchParam) {
      if (searchParam == 'problemID') {
        this.formFilter.completeProblemID = false; // 并非走完全检索displayID了
      }
      this.currentPage = 1;
      this.changeRoute();
    },
    handleRejudge(row) {
      this.submissions[row.index].loading = true;
      api.submissionRejudge(row.submitId).then(
          (res) => {
            let xTable = this.$refs.xTable;
            // 重判开始，需要将该提交的部分参数初始化
            row.status = res.data.data.status;
            row.score = null;
            row.time = res.data.data.time;
            row.memory = res.data.data.memory;
            row.errorMessage = res.data.data.errorMessage;
            row.judger = res.data.data.judger;
            row.loading = false;
            // 重新加载该行数据到view
            xTable.reloadRow(row, null, null);

            this.submissions[row.index] = res.data.data;
            this.submissions[row.index].loading = false;
            myMessage.success(this.$i18n.t('m.Rejudge_successfully'));

            // 加入待重判列表
            this.needCheckSubmitIds[row.submitId] = row.index;
            this.checkStatusNum = 0;
            if (!this.autoCheckOpen) {
              // 如果当前未开启自动检查提交状态的定时任务，则开启
              this.checkSubmissionsStatus();
            }
          },
          () => {
            this.submissions[row.index].loading = false;
          }
      );
    },
    handleOnlyMine() {
      if (this.formFilter.onlyMine) {
        // 需要判断是否为登陆状态
        if (this.isAuthenticated) {
          this.formFilter.username = '';
        } else {
          this.formFilter.onlyMine = false;
          myMessage.error(this.$i18n.t('m.Please_login_first'));
          return;
        }
      }
      this.currentPage = 1;
      this.changeRoute();
    },
    ...mapActions(['changeModalStatus']),

    showSubmitDetail(row) {
      if (row.cid != 0) {
        // 比赛提交详情
        this.$router.push({
          name: 'ContestSubmissionDetail',
          params: {
            contestID: this.$route.params.contestID,
            problemID: row.displayId,
            submitID: row.submitId,
          },
        });
      } else {
        this.$router.push({
          name: 'SubmissionDetail',
          params: {submitID: row.submitId},
        });
      }
    },
    getProblemUri(pid, isContest) {
      if (isContest) {
        this.$router.push({
          name: 'ContestProblemDetails',
          params: {
            contestID: this.$route.params.contestID,
            problemID: pid,
          },
        });
      } else {
        this.$router.push({
          name: 'ProblemDetails',
          params: {
            problemID: pid,
          },
        });
      }
    },
    getStatusColor(status) {
      return 'el-tag el-tag--medium status-' + JUDGE_STATUS[status]['color'];
    },
    tableRowClassName({row, rowIndex}) {
      if (row.username == this.userInfo.username && this.isAuthenticated) {
        return 'own-submit-row';
      }
    }
  },
  computed: {
    ...mapGetters([
      'isAuthenticated',
      'userInfo',
      'isSuperAdmin',
      'isAdminRole',
      'contestRuleType',
      'contestStatus',
      'ContestRealTimePermission',
    ]),
    title() {
      if (!this.contestID) {
        return 'Status';
      } else if (this.problemID) {
        return 'Problem Submissions';
      } else {
        return 'Submissions';
      }
    },
    status() {
      return this.formFilter.status === ''
          ? this.$i18n.t('m.Status')
          : JUDGE_STATUS[this.formFilter.status]
              ? JUDGE_STATUS[this.formFilter.status].name
              : this.$i18n.t('m.Status');
    },
    scoreColumnVisible() {
      return (
          (this.contestID && this.contestRuleType == this.RULE_TYPE.OI) ||
          !this.contestID
      );
    },
  },
  watch: {
    $route(newVal, oldVal) {
      if (newVal !== oldVal) {
        if (this.autoCheckOpen) {
          clearInterval(this.refreshStatus);
        }
        this.init();
        this.getData();
      }
    },
    isAuthenticated() {
      this.init();
      this.getData();
    },
  },
  beforeRouteLeave(to, from, next) {
    // 防止切换组件后仍然不断请求
    clearInterval(this.refreshStatus);
    next();
  },
};
</script>

<style scoped>
@media only screen and (max-width: 767px) {
  .search {
    margin-top: 20px;
  }
}

.flex-container #main {
  flex: auto;
}

.flex-container .filter {
  margin-right: -10px;
}

.flex-container #contest-menu {
  flex: none;
  width: 210px;
}

::v-deep .el-card__header {
  border-bottom: 0px;
  padding-bottom: 0px;
  text-align: center;
}

::v-deep .el-dialog {
  border-radius: 6px !important;
  text-align: center;
}

::v-deep .el-switch {
  padding-top: 6px;
}

@media only screen and (min-width: 768px) and (max-width: 992px) {
  .el-col-sm-12 {
    padding-top: 10px;
  }
}

@media screen and (min-width: 1050px) {
  ::v-deep .vxe-table--body-wrapper {
    overflow-x: hidden !important;
  }
}
</style>
