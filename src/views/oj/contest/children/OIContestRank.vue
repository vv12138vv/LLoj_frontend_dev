<template>
  <el-card shadow>
    <div slot="header">
      <span class="panel-title"
      >{{ $t('m.Contest_Rank') }}({{
          contest.oiRankScoreType == 'Recent'
              ? $t('m.Based_on_The_Recent_Score_Submitted_Of_Each_Problem')
              : $t('m.Based_on_The_Highest_Score_Submitted_For_Each_Problem')
        }})</span
      >
      <el-row style="margin-top: 10px; margin-bottom: 10px">
        <el-col :span="20">
          <el-input
              style="width: 300px"
              prefix-icon="el-icon-search"
              v-model="keyword"
              :placeholder="$t('m.Contest_Rank_Search_Placeholder')"
              @keyup.enter.native="getContestRankData(1)"
          >
          </el-input>
        </el-col>
        <el-col :span="4">
          <span style="float:right;font-size: 20px;margin-top: 8px;">
            <el-popover placement="left-start" trigger="hover">
              <i slot="reference" class="el-icon-s-tools"></i>
              <div id="switches">
                <p>
                  <span>{{ $t('m.Chart') }}</span>
                  <el-switch v-model="showChart"></el-switch>
                </p>
                <p>
                  <span>{{ $t('m.Table') }}</span>
                  <el-switch v-model="showTable"></el-switch>
                </p>
                <p>
                  <span>{{ $t('m.Star_User') }}</span>
                  <el-switch
                      v-model="showStarUser"
                      @change="getContestRankData(page)"
                  ></el-switch>
                </p>
                <p>
                  <span>{{ $t('m.Auto_Refresh') }}(10s)</span>
                  <el-switch
                      v-model="autoRefresh"
                      :disabled="refreshDisabled"
                      @change="handleAutoRefresh"
                  ></el-switch>
                </p>
                <template v-if="isContestAdmin">
                  <p>
                    <span>{{ $t('m.Force_Update') }}</span>
                    <el-switch
                        v-model="forceUpdate"
                        @change="getContestRankData(page)"
                    ></el-switch>
                  </p>
                  <el-button size="small" type="primary" @click="downloadRankCSV">{{
                      $t('m.Download_as_CSV')
                    }}</el-button>
                </template>
                <!--            <template>-->
                <!--              <el-button type="primary" size="small" @click="downloadRankCSV">{{-->
                <!--                $t('m.Download_as_CSV')-->
                <!--              }}</el-button>-->
                <!--            </template>-->
              </div>
            </el-popover>
          </span>
        </el-col>
      </el-row>

    </div>
    <div v-show="showChart" class="echarts">
      <ECharts ref="chart" :autoresize="true" :options="options"></ECharts>
    </div>
    <div v-show="showTable">
      <vxe-table
          ref="OIContestRank"
          :cell-class-name="cellClassName"
          :data="dataRank"
          align="center"
          auto-resize
          border
          round
          size="medium"
          @cell-click="getUserProblemSubmission"
      >
        <!--序号-->
        <vxe-table-column v-if="isContestAdmin" :title="$t('m.Order_Number')" field="seq" fixed="left" width="50">
          <template v-slot="{ row }">
            {{ row.seq }}
          </template>
        </vxe-table-column>

        <!--排名-->
        <vxe-table-column :title="$t('m.Contest_Rank_Seq')" field="rank" fixed="left" width="50">
          <template v-slot="{ row }">
            {{ row.rank == -1 ? '*' : row.rank }}
          </template>
        </vxe-table-column>

        <!--用户-->
        <vxe-table-column v-if="!isMobileView" :title="$t('m.User')" align="left" field="username" fixed="left"
                          header-align="center" min-width="300">
          <template v-slot="{ row }">
            <avatar
                :inline="true"
                :size="37"
                :src="row.avatar"
                :title="row[contest.rankShowName]"
                :username="row[contest.rankShowName]"
                color="#FFF"
            ></avatar>
            <el-tooltip placement="top">
              <div slot="content">
                {{
                  row.isConcerned ? $t('m.Unfollow') : $t('m.Top_And_Follow')
                }}
              </div>
              <span
                  class="contest-rank-concerned"
                  @click="updateConcernedList(row.uid, !row.isConcerned)"
              >
                <i
                    v-if="row.isConcerned"
                    class="fa fa-star"
                    style="color: red;"
                ></i>
                <i v-else class="el-icon-star-off"></i>
              </span>
            </el-tooltip>
            <span style="float:right;text-align:right">
              <a @click="getUserHomeByUsername(row.uid, row.username)">
                <span class="contest-username">
                  <span v-if="row.uid == userInfo.uid" class="contest-rank-flag"
                  >Me</span
                  >
                  <span v-if="row.rank == -1" class="contest-rank-flag"
                  >Star</span
                  >
                  <span v-if="row.gender == 'female'" class="contest-rank-flag"
                  >Girl</span
                  >
                  {{ row[contest.rankShowName] }}</span
                >
                <span v-if="row.school" class="contest-school">{{
                    row.school
                  }}</span>
              </a>
            </span>
          </template>
        </vxe-table-column>
        <vxe-table-column v-else :title="$t('m.User')" align="left" field="username" fixed="left" header-align="center"
                          min-width="300">
          <template v-slot="{ row }">
            <avatar
                :inline="true"
                :size="37"
                :src="row.avatar"
                :title="row[contest.rankShowName]"
                :username="row[contest.rankShowName]"
                color="#FFF"
            ></avatar>
            <el-tooltip placement="top">
              <div slot="content">
                {{
                  row.isConcerned ? $t('m.Unfollow') : $t('m.Top_And_Follow')
                }}
              </div>
              <span
                  class="contest-rank-concerned"
                  @click="updateConcernedList(row.uid, !row.isConcerned)"
              >
                <i
                    v-if="row.isConcerned"
                    class="fa fa-star"
                    style="color: red;"
                ></i>
                <i v-else class="el-icon-star-off"></i>
              </span>
            </el-tooltip>
            <span style="float:right;text-align:right">
              <a @click="getUserHomeByUsername(row.uid, row.username)">
                <span class="contest-username">
                  <span v-if="row.uid == userInfo.uid" class="contest-rank-flag"
                  >Me</span
                  >
                  <span v-if="row.rank == -1" class="contest-rank-flag"
                  >Star</span
                  >
                  <span v-if="row.gender == 'female'" class="contest-rank-flag"
                  >Girl</span
                  >
                  {{ row[contest.rankShowName] }}</span
                >
                <span v-if="row.school" class="contest-school">{{
                    row.school
                  }}</span>
              </a>
            </span>
          </template>
        </vxe-table-column>

        <!--真实姓名-->
        <vxe-table-column v-if="isContestAdmin" :title="$t('m.RealName')" field="realname" min-width="96" show-overflow>
        </vxe-table-column>

        <!--总分-->
        <vxe-table-column :title="$t('m.Total_Score')" field="totalScore" min-width="90">
          <template v-slot="{ row }">
            <span
            ><a
                style="color:rgb(87, 163, 243);"
                @click="getUserTotalSubmit(row.username)"
            >{{ row.totalScore }}</a
            >
              <br/>
              <span class="problem-time">({{ row.totalTime }}ms)</span>
            </span>
          </template>
        </vxe-table-column>

        <!--提交-->
        <vxe-table-column v-for="problem in contestProblems" :key="problem.displayId" :field="problem.displayId"
                          min-width="80">
          <template v-slot:header>
            <span v-if="problem.color" class="contest-rank-balloon">
              <svg
                  class="icon"
                  height="25"
                  p-id="5840"
                  t="1633685184463"
                  version="1.1"
                  viewBox="0 0 1088 1024"
                  width="25"
                  xmlns="http://www.w3.org/2000/svg"
              >
                <path
                    :fill="problem.color"
                    d="M575.872 849.408c-104.576 0-117.632-26.56-119.232-31.808-6.528-22.528 32.896-70.592 63.744-96.768l-1.728-2.624c137.6-42.688 243.648-290.112 243.648-433.472A284.544 284.544 0 0 0 478.016 0a284.544 284.544 0 0 0-284.288 284.736c0 150.4 116.352 415.104 263.744 438.336-25.152 29.568-50.368 70.784-39.104 108.928 12.608 43.136 62.72 63.232 157.632 63.232 7.872 0 11.52 9.408 4.352 19.52-21.248 29.248-77.888 63.424-167.68 63.424V1024c138.944 0 215.936-74.816 215.936-126.528a46.72 46.72 0 0 0-16.32-36.608 56.32 56.32 0 0 0-36.416-11.456zM297.152 297.472c0 44.032-38.144 25.344-38.144-38.656 0-108.032 85.248-195.712 190.592-195.712 62.592 0 81.216 39.232 38.08 39.232-105.152 0.064-190.528 87.04-190.528 195.136z"
                    p-id="5841"
                ></path>
              </svg>
            </span>
            <span>
              <a
                  class="emphasis"
                  style="color:#495060;"
                  @click="getContestProblemById(problem.displayId)"
              >
                {{ problem.displayId }}
              </a>
            </span>
            <br/>
            <span>
              <el-tooltip effect="dark" placement="top">
                <div slot="content">
                  {{ problem.displayId + '. ' + problem.displayTitle }}
                  <br/>
                  {{ 'Accepted: ' + problem.ac }}
                  <br/>
                  {{ 'Rejected: ' + (problem.total - problem.ac) }}
                </div>
                <span>({{ problem.ac }}/{{ problem.total }}) </span>
              </el-tooltip>
            </span>
          </template>
          <template v-slot="{ row }">
            <div
                v-if="row.submissionInfo[problem.displayId]"
                class="submission-hover"
            >
              <span>{{ row.submissionInfo[problem.displayId] }}</span>
              <br/>
              <span
                  v-if="row.timeInfo && row.timeInfo[problem.displayId] != null"
                  style="font-size:12px;"
              >({{ row.timeInfo[problem.displayId] }}ms)</span
              >
            </div>
          </template>
        </vxe-table-column>
      </vxe-table>
    </div>
    <Pagination
        :current.sync="page"
        :layout="'prev, pager, next, sizes'"
        :page-size.sync="limit"
        :page-sizes="[10, 30, 50]"
        :total="total"
        @on-change="getContestRankData"
        @on-page-size-change="getContestRankData(1)"
    ></Pagination>
  </el-card>
</template>
<script>
import Avatar from 'vue-avatar';
import {mapActions} from 'vuex';
import ContestRankMixin from './contestRankMixin';
import utils from '@/common/utils';

const Pagination = () => import('@/components/oj/common/Pagination');
export default {
  name: 'OIContestRank',
  components: {
    Pagination,
    Avatar,
  },
  mixins: [ContestRankMixin],
  data() {
    return {
      total: 0,
      page: 1,
      limit: 50,
      keyword: '',
      contestID: '',
      dataRank: [],
      autoRefresh: false,
      options: {
        title: {
          text: this.$i18n.t('m.Top_10_Teams'),
          left: 'center',
        },
        tooltip: {
          trigger: 'axis',
        },
        toolbox: {
          show: true,
          feature: {
            dataView: {show: true, readOnly: true},
            magicType: {show: true, type: ['line', 'bar']},
            saveAsImage: {show: true, title: this.$i18n.t('m.save_as_image')},
          },
          right: '10%',
          top: '5%',
        },
        calculable: true,
        xAxis: [
          {
            type: 'category',
            data: ['root'],
            boundaryGap: true,
            axisLabel: {
              interval: 0,
              showMinLabel: true,
              showMaxLabel: true,
              align: 'center',
              formatter: (value, index) => {
                return utils.breakLongWords(value, 14);
              },
            },
            axisTick: {
              alignWithLabel: true,
            },
          },
        ],
        yAxis: [
          {
            type: 'value',
          },
        ],
        grid: {
          left: '11%',
        },
        series: [
          {
            name: this.$i18n.t('m.Score'),
            type: 'bar',
            barMaxWidth: '80',
            data: [0],
            markPoint: {
              data: [{type: 'max', name: 'max'}],
            },
          },
        ],
      },
    };
  },
  created() {
    this.initConcernedList();
  },
  mounted() {
    this.contestID = this.$route.params.contestID;
    this.getContestRankData(1);
    if (!this.refreshDisabled) {
      this.autoRefresh = true;
      this.handleAutoRefresh(true);
    }
  },
  computed: {
    contest() {
      return this.$store.state.contest.contest;
    },
    isMobileView() {
      return window.screen.width < 768;
    },
  },
  methods: {
    ...mapActions(['getContestProblems']),

    cellClassName({row, rowIndex, column, columnIndex}) {
      if (column.property === 'username' && row.registered === false) {
        return 'unregistered';
      }

      if (row.username === this.userInfo.username) {
        if (
            column.property === 'seq' ||
            column.property === 'rank' ||
            column.property === 'username' ||
            column.property === 'realname' ||
            column.property === 'totalScore'
        ) {
          return 'own-submit-row';
        }
      }

      if (column.property === 'username' && row.userCellClassName) {
        return row.userCellClassName;
      }

      if (
          column.property !== 'seq' &&
          column.property !== 'rank' &&
          column.property !== 'totalScore' &&
          column.property !== 'username' &&
          column.property !== 'realname'
      ) {
        if (this.isContestAdmin) {
          return row.cellClassName[
              [this.contestProblems[columnIndex - 5].displayId]
              ];
        } else {
          return row.cellClassName[
              [this.contestProblems[columnIndex - 3].displayId]
              ];
        }
      } else {
        if (row.isConcerned && column.property !== 'username') {
          return 'bg-concerned';
        }
      }
    },
    getUserTotalSubmit(username) {
      this.$router.push({
        name: 'ContestSubmissionList',
        query: {username: username},
      });
    },
    getUserHomeByUsername(uid, username) {
      this.$router.push({
        name: 'UserHome',
        query: {username: username, uid: uid},
      });
    },
    getContestProblemById(pid) {
      this.$router.push({
        name: 'ContestProblemDetails',
        params: {
          contestID: this.contestID,
          problemID: pid,
        },
      });
    },
    getUserProblemSubmission({row, column}) {
      if (
          column.property === 'seq' ||
          column.property === 'rank' ||
          column.property === 'totalScore' ||
          column.property === 'username' ||
          column.property === 'realname' ||
          row[column.property] === undefined
      ) {
        return;
      }
      this.$router.push({
        name: 'ContestSubmissionList',
        query: {username: row.username, problemID: column.property},
      });
    },
    applyToChart(rankData) {
      let [user, scores] = [[], []];
      let len = rankData.length;
      let topIndex = this.concernedList.length || 0;
      if (rankData.length > 0) {
        if (rankData[0].uid == this.userInfo.uid) {
          topIndex++;
        }
      }
      for (let i = topIndex; i < len && i < topIndex + 10; i++) {
        let ele = rankData[i];
        user.push(ele[this.contest.rankShowName]);
        scores.push(ele.totalScore);
      }
      this.options.xAxis[0].data = user;
      this.options.series[0].data = scores;
    },
    applyToTable(dataRank) {
      dataRank.forEach((rank, i) => {
        let submissionInfo = rank.submissionInfo;
        let timeInfo = rank.timeInfo;
        let cellClass = {};
        if (this.concernedList.indexOf(rank.uid) !== -1) {
          dataRank[i].isConcerned = true;
        }
        Object.keys(submissionInfo).forEach((problemID) => {
          dataRank[i][problemID] = submissionInfo[problemID];
          let score = submissionInfo[problemID];
          if (timeInfo != null && timeInfo[problemID] !== undefined) {
            cellClass[problemID] = 'oi-100';
          } else if (score === 0) {
            cellClass[problemID] = 'oi-0';
          } else {
            cellClass[problemID] = 'oi-between';
          }
        });
        dataRank[i].cellClassName = cellClass;
        if (dataRank[i].rank === -1) {
          dataRank[i].userCellClassName = 'bg-star';
        }
        if (dataRank[i].gender === 'female') {
          dataRank[i].userCellClassName = 'bg-female';
        }
      });
      this.dataRank = dataRank;
    },

    downloadRankCSV() {
      utils.downloadFile(
          `/api/file/download-contest-rank?cid=${
              this.$route.params.contestID
          }&forceRefresh=${this.forceUpdate ? true : false}`
      );
    },
  },
};
</script>
<style scoped>
.echarts {
  margin: 20px auto;
  height: 400px;
  width: 100%;
}

::v-deep .el-card__body {
  padding: 20px !important;
  padding-top: 0 !important;
}

@media screen and (max-width: 768px) {
  ::v-deep .el-card__body {
    padding: 0 !important;
  }
}

.screen-full {
  margin-right: 8px;
}

#switches p {
  margin-top: 5px;
}

#switches p:first-child {
  margin-top: 0;
}

#switches p span {
  margin-left: 8px;
  margin-right: 4px;
}

.vxe-cell p,
.vxe-cell span {
  margin: 0;
  padding: 0;
}

::v-deep .vxe-table .vxe-header--column:not(.col--ellipsis) {
  padding: 4px 0 !important;
}

::v-deep .vxe-table .vxe-body--column {
  line-height: 20px !important;
  padding: 0px !important;
}

::v-deep .vxe-body--column {
  min-width: 0;
  height: 48px;
  box-sizing: border-box;
  text-align: left;
  text-overflow: ellipsis;
  vertical-align: middle;
}

a.emphasis {
  color: #495060 !important;
}

a.emphasis:hover {
  color: #2d8cf0 !important;
}

.problem-time {
  color: rgba(0, 0, 0, 0.45);
  font-size: 12px;
}

::v-deep .vxe-table .vxe-cell {
  padding-left: 5px !important;
  padding-right: 5px !important;
}
</style>
