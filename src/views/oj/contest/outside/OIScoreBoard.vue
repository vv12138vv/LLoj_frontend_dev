<template>
  <div class="scoreboard-body">
    <el-card v-loading="loading.info" shadow>
      <div class="contest-title">
        <div slot="header">
          <span class="panel-title">{{ contest.title }}</span>
        </div>
      </div>
      <el-row style="margin-top: 10px;">
        <el-col :span="12" class="text-align:left">
          <el-tooltip
              v-if="contest.auth != null && contest.auth != undefined"
              :content="$t('m.' + CONTEST_TYPE_REVERSE[contest.auth]['tips'])"
              placement="top"
          >
            <el-tag
                :type.sync="CONTEST_TYPE_REVERSE[contest.auth]['color']"
                effect="plain"
            >
              {{ $t('m.' + CONTEST_TYPE_REVERSE[contest.auth]['name']) }}
            </el-tag>
          </el-tooltip>
        </el-col>
        <el-col :span="12" style="text-align:right">
          <el-button v-if="contest.count != null" plain size="small">
            <i class="el-icon-user-solid" style="color:rgb(48, 145, 242);"></i
            >x{{ contest.count }}
          </el-button>

          <template v-if="contest.type == 0">
            <el-button :type="'primary'" size="small">
              <i class="fa fa-trophy"></i>
              {{ contest.type | parseContestType }}
            </el-button>
          </template>
          <template v-else>
            <el-tooltip
                :content="
                $t('m.Contest_Rank') +
                  '：' +
                  (contest.oiRankScoreType == 'Recent'
                    ? $t(
                        'm.Based_on_The_Recent_Score_Submitted_Of_Each_Problem'
                      )
                    : $t(
                        'm.Based_on_The_Highest_Score_Submitted_For_Each_Problem'
                      ))
              "
                placement="top"
            >
              <el-button :type="'warning'" size="small">
                <i class="fa fa-trophy"></i>
                {{ contest.type | parseContestType }}
              </el-button>
            </el-tooltip>
          </template>
        </el-col>
      </el-row>

      <div class="contest-time">
        <el-row>
          <el-col :md="12" :xs="24" class="left">
            <p>
              <i aria-hidden="true" class="fa fa-hourglass-start"></i>
              {{ $t('m.StartAt') }}：{{ contest.startTime | localtime }}
            </p>
          </el-col>
          <el-col :md="12" :xs="24" class="right">
            <p>
              <i aria-hidden="true" class="fa fa-hourglass-end"></i>
              {{ $t('m.EndAt') }}：{{ contest.endTime | localtime }}
            </p>
          </el-col>
        </el-row>
      </div>
      <div class="slider">
        <el-slider
            v-model="progressValue"
            :format-tooltip="formatTooltip"
            :step="timeStep"
        ></el-slider>
      </div>
      <el-row>
        <el-col :span="24" style="text-align:center">
          <el-tag :style="countdownColor" effect="dark" size="medium">
            <i aria-hidden="true" class="fa fa-circle"></i>
            {{ countdown }}
          </el-tag>
        </el-col>
      </el-row>
    </el-card>
    <el-card v-loading="loading.rank" shadow style="margin-top:15px;">
      <el-row style="margin-bottom: 10px">
        <el-col :span="16">
          <el-input
              style="width: 300px"
              prefix-icon="el-icon-search"
              v-model="keyword"
              :placeholder="$t('m.Contest_Rank_Search_Placeholder')"
              @keyup.enter.native="getContestOutsideScoreboard(1)"
          >
          </el-input>
        </el-col>
        <el-col :span="8">
          <div class="contest-rank-switch">
            <span style="float:right;">
              <span>{{ $t('m.Auto_Refresh') }}(30s)</span>
              <el-switch
                  v-model="autoRefresh"
                  :disabled="contestEnded"
                  @change="handleAutoRefresh"
              ></el-switch>
            </span>
                <span v-if="isContestAdmin" style="float:right;">
              <span>{{ $t('m.Force_Update') }}</span>
              <el-switch
                  v-model="forceUpdate"
                  @change="getContestOutsideScoreboard(page)"
              ></el-switch>
            </span>
                <span style="float:right;">
              <span>{{ $t('m.Star_User') }}</span>
              <el-switch
                  v-model="showStarUser"
                  @change="getContestOutsideScoreboard(page)"
              ></el-switch>
            </span>
          </div>
        </el-col>
      </el-row>

      <vxe-table
          ref="OIContestRank"
          :cell-class-name="cellClassName"
          :data="dataRank"
          align="center"
          auto-resize
          border
          round
          size="medium"
      >
        <vxe-table-column
            :title="$t('m.Contest_Rank_Seq')"
            field="rank"
            fixed="left"
            width="50"
        >
          <template v-slot="{ row }">
            {{ row.rank == -1 ? '*' : row.rank }}
          </template>
        </vxe-table-column>
        <vxe-table-column
            v-if="!isMobileView"
            :title="$t('m.User')"
            align="left"
            field="username"
            fixed="left"
            header-align="center"
            min-width="300"
        >
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
                <span class="contest-username"
                ><span v-if="row.rank == -1" class="contest-rank-flag"
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
        <vxe-table-column
            v-else
            :title="$t('m.User')"
            align="left"
            field="username"
            header-align="center"
            min-width="300"
        >
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
                <span class="contest-username"
                ><span v-if="row.rank == -1" class="contest-rank-flag"
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
        <vxe-table-column
            :title="$t('m.Total_Score')"
            field="totalScore"
            min-width="90"
        >
          <template v-slot="{ row }">
            <span
            ><a style="color:rgb(87, 163, 243);">{{ row.totalScore }}</a>
              <br/>
              <span class="problem-time">({{ row.totalTime }}ms)</span>
            </span>
          </template>
        </vxe-table-column>
        <vxe-table-column
            v-for="problem in contestProblems"
            :key="problem.displayId"
            min-width="80"
        >
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
              <a class="emphasis" style="color:#495060;">
                {{ problem.displayId }}
              </a>
            </span>
            <br/>
            <span>
              <el-tooltip effect="dark" placement="top">
                <div slot="content">
                  {{ problem.displayId + '. ' + problem.displayTitle }}
                  <br/>
                  {{
                    'Accepted: ' +
                    getProblemCount(problemACCountMap[problem.displayId])
                  }}
                  <br/>
                  {{
                    'Rejected: ' +
                    getProblemCount(problemErrorCountMap[problem.displayId])
                  }}
                </div>
                <span
                >({{
                    getProblemCount(problemACCountMap[problem.displayId])
                  }}/{{
                    getProblemCount(problemACCountMap[problem.displayId]) +
                    getProblemCount(problemErrorCountMap[problem.displayId])
                  }})
                </span>
              </el-tooltip>
            </span>
          </template>
          <template v-slot="{ row }">
            <div v-if="row.submissionInfo[problem.displayId]">
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
      <Pagination
          :current.sync="page"
          :layout="'prev, pager, next, sizes'"
          :page-size.sync="limit"
          :page-sizes="[10, 30, 50]"
          :total="total"
          @on-change="getContestOutsideScoreboard"
          @on-page-size-change="getContestOutsideScoreboard(1)"
      ></Pagination>
    </el-card>
  </div>
</template>
<script>
import Avatar from 'vue-avatar';
import ScoreBoardMixin from './scoreBoardMixin';
const Pagination = () => import('@/components/oj/common/Pagination');
export default {
  name: 'OIContestRank',
  components: {
    Pagination,
    Avatar,
  },
  mixins: [ScoreBoardMixin],
  data() {
    return {
      total: 0,
      page: 1,
      limit: 50,
      keyword: '',
      contestID: '',
      dataRank: [],
      autoRefresh: false,
      loading: {
        info: false,
        rank: false,
      },
      timer: null,
      CONTEST_STATUS: {},
      CONTEST_STATUS_REVERSE: {},
      CONTEST_TYPE_REVERSE: {},
      RULE_TYPE: {},
      problemACCountMap: {},
      problemErrorCountMap: {},
    };
  },
  created() {
    this.init();
  },
  mounted() {
    this.getContestOutsideScoreboard(1);
  },
  computed: {
    isMobileView() {
      return window.screen.width < 768;
    },
  },
  methods: {
    cellClassName({row, rowIndex, column, columnIndex}) {
      if (column.property === 'username' && row.userCellClassName) {
        return row.userCellClassName;
      }

      if (
          column.property !== 'rank' &&
          column.property !== 'totalScore' &&
          column.property !== 'username'
      ) {
        return row.cellClassName[
            [this.contestProblems[columnIndex - 3].displayId]
            ];
      } else {
        if (row.isConcerned && column.property !== 'username') {
          return 'bg-concerned';
        }
      }
    },
    getUserHomeByUsername(uid, username) {
      this.$router.push({
        name: 'UserHome',
        query: {username: username, uid: uid},
      });
    },
    applyToTable(dataRank) {
      let acCountMap = {};
      let errorCountMap = {};
      dataRank.forEach((rank, i) => {
        let submissionInfo = rank.submissionInfo;
        let timeInfo = rank.timeInfo;
        let cellClass = {};
        if (this.concernedList.indexOf(rank.uid) != -1) {
          dataRank[i].isConcerned = true;
        }
        Object.keys(submissionInfo).forEach((problemID) => {
          dataRank[i][problemID] = submissionInfo[problemID];
          if (!acCountMap[problemID]) {
            acCountMap[problemID] = 0;
          }
          if (!errorCountMap[problemID]) {
            errorCountMap[problemID] = 0;
          }

          let score = submissionInfo[problemID];
          if (timeInfo != null && timeInfo[problemID] != undefined) {
            cellClass[problemID] = 'oi-100';
            acCountMap[problemID] += 1;
          } else if (score == 0) {
            cellClass[problemID] = 'oi-0';
            errorCountMap[problemID] += 1;
          } else {
            cellClass[problemID] = 'oi-between';
            errorCountMap[problemID] += 1;
          }
        });
        dataRank[i].cellClassName = cellClass;
        if (dataRank[i].rank == -1) {
          dataRank[i].userCellClassName = 'bg-star';
        }
        if (dataRank[i].gender == 'female') {
          dataRank[i].userCellClassName = 'bg-female';
        }
      });
      this.dataRank = dataRank;
      this.problemACCountMap = acCountMap;
      this.problemErrorCountMap = errorCountMap;
    },
  },
};
</script>
<style scoped>
@media screen and (min-width: 1050px) {
  .scoreboard-body {
    margin-left: -2%;
    margin-right: -2%;
  }
}

.contest-title {
  text-align: center;
}

.panel-title {
  font-size: 1.5rem !important;
  font-weight: 500;
}

.contest-time {
  width: 100%;
  font-size: 16px;
}

@media screen and (min-width: 768px) {
  .contest-time .left {
    text-align: left;
  }

  .contest-time .right {
    text-align: right;
  }
}

@media screen and (max-width: 768px) {
  .contest-time .left,
  .contest-time .right {
    text-align: center;
  }
}

::v-deep .el-slider__button {
  width: 20px !important;
  height: 20px !important;
  background-color: #409eff !important;
}

::v-deep .el-slider__button-wrapper {
  z-index: 500;
}

::v-deep .el-slider__bar {
  height: 10px !important;
  background-color: #09be24 !important;
}

.el-tag--dark {
  border-color: #fff;
}

.el-tag {
  color: rgb(25, 190, 107);
  background: #fff;
  border: 1px solid #e9eaec;
  font-size: 18px;
}

::v-deep .el-card__body {
  padding: 15px !important;
  padding-top: 20px !important;
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
  padding: 4px 0 !important;
  line-height: 20px !important;
}

::v-deep .vxe-table .vxe-body--column:not(.col--ellipsis) {
  line-height: 20px !important;
  padding: 0 !important;
}

::v-deep .vxe-body--column {
  min-width: 0;
  height: 48px;
  box-sizing: border-box;
  text-align: left;
  text-overflow: ellipsis;
  vertical-align: middle;
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
