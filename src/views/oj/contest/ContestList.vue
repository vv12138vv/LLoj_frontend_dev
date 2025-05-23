<template>
  <el-row justify="space-around" type="flex">
    <el-col :span="24">
      <el-card shadow>
        <div slot="header">
          <span class="panel-title"
          >{{
              query.type === '' ? $t('m.All') : parseContestType(query.type)
            }}
            {{ $t('m.Contests') }}</span
          >
          <div class="filter-row">
            <span>
              <el-dropdown
                  class="drop-menu"
                  placement="bottom"
                  trigger="hover"
                  @command="onRuleChange"
              >
                <span class="el-dropdown-link">
                  {{
                    query.type == ''
                        ? $t('m.Rule')
                        : parseContestType(query.type)
                  }}
                  <i class="el-icon-caret-bottom"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="">{{
                      $t('m.All')
                    }}</el-dropdown-item>
                  <el-dropdown-item command="0">ACM</el-dropdown-item>
                  <el-dropdown-item command="1">OI</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </span>

            <span>
              <el-dropdown
                  class="drop-menu"
                  placement="bottom"
                  trigger="hover"
                  @command="onStatusChange"
              >
                <span class="el-dropdown-link">
                  {{
                    query.status === ''
                        ? $t('m.Status')
                        : $t('m.' + CONTEST_STATUS_REVERSE[query.status]['name'])
                  }}
                  <i class="el-icon-caret-bottom"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="">{{
                      $t('m.All')
                    }}</el-dropdown-item>
                  <el-dropdown-item command="-1">{{
                      $t('m.Scheduled')
                    }}</el-dropdown-item>
                  <el-dropdown-item command="0">{{
                      $t('m.Running')
                    }}</el-dropdown-item>
                  <el-dropdown-item command="1">{{
                      $t('m.Ended')
                    }}</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </span>

            <span>
              <vxe-input
                  v-model="query.keyword"
                  :placeholder="$t('m.Enter_keyword')"
                  size="medium"
                  type="search"
                  @keyup.enter.native="filterByChange"
                  @search-click="filterByChange"
              ></vxe-input>
            </span>
          </div>
        </div>
        <div v-loading="loading">
          <p v-show="contests.length == 0" id="no-contest">
            <el-empty :description="$t('m.No_contest')"></el-empty>
          </p>
          <ol id="contest-list">
            <li
                v-for="contest in contests"
                :key="contest.title"
                :style="getborderColor(contest)"
            >
              <el-row align="middle" justify="space-between" type="flex">
                <el-col :lg="2" :md="3" :sm="4" :xs="10">
                  <img
                      v-show="contest.type == 0"
                      :src="acmSrc"
                      class="trophy"
                      width="95px"
                  />
                  <img
                      v-show="contest.type == 1"
                      :src="oiSrc"
                      class="trophy"
                      width="95px"
                  />
                </el-col>
                <el-col
                    :lg="20"
                    :md="19"
                    :sm="16"
                    :xs="10"
                    class="contest-main"
                >
                  <p class="title">
                    <a class="entry" @click.stop="toContest(contest)">
                      {{ contest.title }}
                    </a>
                    <template v-if="contest.auth == 1">
                      <i
                          class="el-icon-lock"
                          size="20"
                          style="color:#d9534f"
                      ></i>
                    </template>
                    <template v-if="contest.auth == 2">
                      <i
                          class="el-icon-lock"
                          size="20"
                          style="color:#f0ad4e"
                      ></i>
                    </template>
                  </p>
                  <ul class="detail">
                    <li>
                      <i
                          aria-hidden="true"
                          class="fa fa-calendar"
                          style="color: #3091f2"
                      ></i>
                      {{ contest.startTime | localtime }}
                    </li>
                    <li>
                      <i
                          aria-hidden="true"
                          class="fa fa-clock-o"
                          style="color: #3091f2"
                      ></i>
                      {{ getDuration(contest.startTime, contest.endTime) }}
                    </li>
                    <li>
                      <template v-if="contest.type == 0">
                        <el-button
                            :type="'primary'"
                            round
                            size="mini"
                            @click="onRuleChange(contest.type)"
                        ><i class="fa fa-trophy"></i>
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
                          <el-button
                              :type="'warning'"
                              round
                              size="mini"
                              @click="onRuleChange(contest.type)"
                          ><i class="fa fa-trophy"></i>
                            {{ contest.type | parseContestType }}
                          </el-button>
                        </el-tooltip>
                      </template>
                    </li>
                    <li>
                      <el-tooltip
                          :content="
                          $t('m.' + CONTEST_TYPE_REVERSE[contest.auth].tips)
                        "
                          effect="light"
                          placement="top"
                      >
                        <el-tag
                            :type="CONTEST_TYPE_REVERSE[contest.auth]['color']"
                            effect="plain"
                        >
                          {{
                            $t(
                                'm.' + CONTEST_TYPE_REVERSE[contest.auth]['name']
                            )
                          }}
                        </el-tag>
                      </el-tooltip>
                    </li>
                    <li v-if="contest.count != null">
                      <i
                          class="el-icon-user-solid"
                          style="color:rgb(48, 145, 242);"
                      ></i
                      >x{{ contest.count }}
                    </li>
                    <li v-if="contest.openRank">
                      <el-tooltip
                          :content="$t('m.Contest_Outside_ScoreBoard')"
                          effect="dark"
                          placement="top"
                      >
                        <el-button
                            :disabled="contest.status == CONTEST_STATUS.SCHEDULED"
                            circle
                            icon="el-icon-data-analysis"
                            size="small"
                            type="primary"
                            @click="
                            toContestOutsideScoreBoard(contest.id, contest.type)
                          "
                        ></el-button>
                      </el-tooltip>
                    </li>
                  </ul>
                </el-col>
                <el-col
                    :lg="2"
                    :md="2"
                    :sm="4"
                    :xs="4"
                    style="text-align: center"
                >
                  <el-tag
                      :color="CONTEST_STATUS_REVERSE[contest.status]['color']"
                      effect="dark"
                      size="medium"
                  >
                    <i aria-hidden="true" class="fa fa-circle"></i>
                    {{
                      $t('m.' + CONTEST_STATUS_REVERSE[contest.status]['name'])
                    }}
                  </el-tag>
                </el-col>
              </el-row>
            </li>
          </ol>
        </div>
      </el-card>
      <Pagination
          :current.sync="currentPage"
          :pageSize="limit"
          :total="total"
          @on-change="getContestList"
      ></Pagination>
    </el-col>
  </el-row>
</template>

<script>
import api from '@/common/api';
import {mapGetters} from 'vuex';
import utils from '@/common/utils';
import time from '@/common/time';
import {CONTEST_STATUS, CONTEST_STATUS_REVERSE, CONTEST_TYPE_REVERSE,} from '@/common/constants';
import myMessage from '@/common/message';

const Pagination = () => import('@/components/oj/common/Pagination');
const limit = 10;

export default {
  name: 'ContestList',
  components: {
    Pagination,
  },
  data() {
    return {
      currentPage: 1,
      query: {
        status: '',
        keyword: '',
        type: '',
      },
      limit: limit,
      total: 0,
      rows: '',
      contests: [],
      CONTEST_STATUS_REVERSE: {},
      CONTEST_STATUS: {},
      CONTEST_TYPE_REVERSE: {},
      acmSrc: require('@/assets/acm.jpg'),
      oiSrc: require('@/assets/oi.jpg'),
      loading: true,
    };
  },
  mounted() {
    this.CONTEST_STATUS_REVERSE = Object.assign({}, CONTEST_STATUS_REVERSE);
    this.CONTEST_TYPE_REVERSE = Object.assign({}, CONTEST_TYPE_REVERSE);
    this.CONTEST_STATUS = Object.assign({}, CONTEST_STATUS);
    this.init();
  },
  methods: {
    init() {
      let route = this.$route.query;
      this.query.status = route.status || '';
      this.query.type = route.type || '';
      this.query.keyword = route.keyword || '';
      this.currentPage = parseInt(route.currentPage) || 1;
      this.getContestList();
    },
    getContestList(page = 1) {
      this.loading = true;
      api.getContestList(page, this.limit, this.query).then(
          (res) => {
            this.contests = res.data.data.records;
            this.total = res.data.data.total;
            this.loading = false;
          },
          (err) => {
            this.loading = false;
          }
      );
    },
    filterByChange() {
      let query = Object.assign({}, this.query);
      query.currentPage = this.currentPage;
      this.$router.push({
        name: 'ContestList',
        query: utils.filterEmptyValue(query),
      });
    },

    parseContestType(type) {
      if (type == 0) {
        return 'ACM';
      } else if (type == 1) {
        return 'OI';
      }
    },
    onRuleChange(rule) {
      this.query.type = rule;
      this.currentPage = 1;
      this.filterByChange();
    },
    onStatusChange(status) {
      this.query.status = status;
      this.currentPage = 1;
      this.filterByChange();
    },
    toContest(contest) {
      if (!this.isAuthenticated) {
        myMessage.warning(this.$i18n.t('m.Please_login_first'));
        this.$store.dispatch('changeModalStatus', {visible: true});
      } else {
        this.$router.push({
          // name: 'ContestDetails',
          name: 'ContestProblemList',
          params: {contestID: contest.id},
        });
      }
    },
    toContestOutsideScoreBoard(cid, type) {
      if (type == 0) {
        this.$router.push({
          name: 'ACMScoreBoard',
          params: {contestID: cid},
        });
      } else if (type == 1) {
        this.$router.push({
          name: 'OIScoreBoard',
          params: {contestID: cid},
        });
      }
    },
    getDuration(startTime, endTime) {
      return time.formatSpecificDuration(startTime, endTime);
    },
    getborderColor(contest) {
      return (
          'border-left: 4px solid ' +
          CONTEST_STATUS_REVERSE[contest.status]['color']
      );
    },
  },
  computed: {
    ...mapGetters(['isAuthenticated', 'userInfo']),
  },
  watch: {
    $route(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.init();
      }
    },
  },
};
</script>
<style scoped>
#no-contest {
  text-align: center;
  font-size: 16px;
  padding: 20px;
}

.filter-row {
  float: right;
}

@media screen and (max-width: 768px) {
  .filter-row span {
    margin-right: 2px;
  }

  ol {
    padding-inline-start: 5px;
  }

  ::v-deep .el-card__header {
    margin-bottom: 5px;
  }
}

@media screen and (min-width: 768px) {
  .filter-row span {
    margin-right: 20px;
  }
}

::v-deep .el-card__header {
  border-bottom: 0px;
}

#contest-list > li {
  padding: 5px;
  margin-left: -20px;
  margin-top: 10px;
  width: 100%;
  border-bottom: 1px solid rgba(187, 187, 187, 0.5);
  list-style: none;
}

#contest-list .trophy {
  height: 70px;
  margin-left: 10px;
  margin-right: -20px;
}

#contest-list .contest-main {
  text-align: left;
}

#contest-list .contest-main .title {
  font-size: 1.25rem;
  padding-left: 8px;
  margin-bottom: 0;
}

#contest-list .contest-main .title a.entry {
  color: #495060;
}

#contest-list .contest-main .title a:hover {
  color: #2d8cf0;
  border-bottom: 1px solid #2d8cf0;
}

#contest-list .contest-main .detail {
  font-size: 0.875rem;
  padding-left: 0;
  padding-bottom: 10px;
}

#contest-list .contest-main li {
  display: inline-block;
  padding: 10px 0 0 10px;
}
</style>
