<template>
  <el-row justify="space-around" type="flex">
    <el-col :span="24">
<!--      <el-card :padding="10">-->
<!--        <div slot="header">-->
<!--          <span class="panel-title">{{ $t('m.OI_Ranklist') }}</span>-->
<!--        </div>-->
<!--        <div class="echarts">-->
<!--          <ECharts ref="chart" :options="options" auto-resize></ECharts>-->
<!--        </div>-->
<!--      </el-card>-->
      <el-card :padding="10">
        <div slot="header">
          <span class="panel-title">{{ $t('m.OI_Ranklist') }}</span>
        </div>
        <div style="text-align: center;">
          <el-input
              v-model="searchUser"
              :placeholder="$t('m.Rank_Search_Placeholder')"
              @keyup.enter.native="getRankData(1)"
          >
            <el-button
                slot="append"
                class="search-btn"
                icon="el-icon-search"
                @click="getRankData(1)"
            ></el-button>
          </el-input>
        </div>
      </el-card>
      <vxe-table
          :data="dataRank"
          :loading="loadingTable"
          :seq-config="{ seqMethod }"
          align="center"
          auto-resize
          highlight-hover-row
          style="font-weight: 500;"
      >
        <vxe-table-column min-width="50" type="seq"></vxe-table-column>
        <vxe-table-column
            :title="$t('m.User')"
            align="left"
            field="username"
            min-width="150"
            show-overflow
        >
          <template v-slot="{ row }">
            <avatar
                :inline="true"
                :size="25"
                :src="row.avatar"
                :username="row.username"
                class="user-avatar"
                color="#FFF"
            ></avatar>
            <a
                style="color:#2d8cf0;"
                @click="getInfoByUsername(row.uid, row.username)"
            >{{ row.username }}</a
            >
          </template>
        </vxe-table-column>
        <vxe-table-column
            :title="$t('m.Nickname')"
            field="nickname"
            width="160"
        >
          <template v-slot="{ row }">
            <el-tag
                v-if="row.nickname"
                :type="nicknameColor(row.nickname)"
                effect="plain"
                size="small"
            >
              {{ row.nickname }}
            </el-tag>
          </template>
        </vxe-table-column>
        <vxe-table-column :title="$t('m.Score')" min-width="80">
          <template v-slot="{ row }">
            <span>{{ row.score }}</span>
          </template>
        </vxe-table-column>
        <vxe-table-column
            :title="$t('m.AC') + '/' + $t('m.Total')"
            min-width="100"
        >
          <template v-slot="{ row }">
            <span>
              <a
                  style="color:rgb(87, 163, 243);"
                  @click="goUserACStatus(row.username)"
              >{{ row.ac }}</a
              >
              <span>/{{ row.total }}</span>
            </span>
          </template>
        </vxe-table-column>
        <vxe-table-column :title="$t('m.Rating')" min-width="80">
          <template v-slot="{ row }">
            <span>{{ getACRate(row.ac, row.total) }}</span>
          </template>
        </vxe-table-column>
        <vxe-table-column
            :title="$t('m.Signature')"
            align="left"
            min-width="300"
            show-overflow="ellipsis"
        >
          <template v-slot="{ row }">
            <span v-if="row.signature" v-katex class="rank-signature-body">{{
                row.signature
              }}</span>
          </template>
        </vxe-table-column>
      </vxe-table>
      <Pagination
          :current.sync="page"
          :layout="'prev, pager, next, sizes'"
          :page-size.sync="limit"
          :total="total"
          show-sizer
          @on-change="getRankData"
          @on-page-size-change="getRankData(1)"
      ></Pagination>
    </el-col>
  </el-row>
</template>

<script>
import api from '@/common/api';
import utils from '@/common/utils';
import {RULE_TYPE} from '@/common/constants';
import {mapGetters} from 'vuex';
import Avatar from 'vue-avatar';

const Pagination = () => import('@/components/oj/common/Pagination');
export default {
  name: 'acm-rank',
  components: {
    Pagination,
    Avatar,
  },
  data() {
    return {
      page: 1,
      limit: 50,
      total: 0,
      searchUser: null,
      dataRank: [],
      loadingTable: false,
      screenWidth: 768,
      options: {
        tooltip: {
          trigger: 'axis',
        },
        legend: {
          data: ['Score'],
        },
        grid: {
          x: '3%',
          x2: '3%',
          left: '8%',
          right: '8%',
        },
        toolbox: {
          show: true,
          feature: {
            dataView: {show: true, readOnly: true},
            magicType: {show: true, type: ['line', 'bar']},
            saveAsImage: {show: true},
          },
          right: '8%',
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
                if (this.screenWidth < 768) {
                  if (this.isAuthenticated && this.userInfo.username == value) {
                    return utils.breakLongWords(value, 14);
                  } else {
                    return '';
                  }
                } else {
                  return utils.breakLongWords(value, 14);
                }
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
            axisLabel: {
              rotate: 50,
              textStyle: {
                fontSize: '12em',
              },
            },
          },
        ],
        series: [
          {
            name: this.$i18n.t('m.Score'),
            type: 'bar',
            data: [0],
            barMaxWidth: '80',
            markPoint: {
              data: [{type: 'max', name: 'max'}],
            },
          },
        ],
      },
    };
  },
  created() {
    this.screenWidth = window.screen.width;
    const that = this;
    window.onresize = () => {
      return (() => {
        that.screenWidth = document.documentElement.clientWidth;
      })();
    };
  },
  mounted() {
    this.getRankData(1);
  },
  methods: {
    getRankData(page) {
      // let bar = this.$refs.chart;
      // bar.showLoading({maskColor: 'rgba(250, 250, 250, 0.8)'});
      this.loadingTable = true;
      api.getUserRank(page, this.limit, RULE_TYPE.OI, this.searchUser).then(
          (res) => {
            if (page === 1) {
              this.changeCharts(res.data.data.records.slice(0, 10));
            }
            this.total = res.data.data.total;
            this.dataRank = res.data.data.records;
            this.loadingTable = false;
            bar.hideLoading();
          },
          (err) => {
            this.loadingTable = false;
            bar.hideLoading();
          }
      );
    },
    seqMethod({rowIndex}) {
      return this.limit * (this.page - 1) + rowIndex + 1;
    },
    changeCharts(rankData) {
      let [usernames, scores] = [[], []];
      rankData.forEach((ele) => {
        usernames.push(ele.username);
        scores.push(ele.score);
      });
      this.options.xAxis[0].data = usernames;
      this.options.series[0].data = scores;
    },
    getInfoByUsername(uid, username) {
      this.$router.push({
        path: '/user-home',
        query: {uid, username},
      });
    },
    goUserACStatus(username) {
      this.$router.push({
        path: '/status',
        query: {username, status: 0},
      });
    },
    getACRate(ac, total) {
      return utils.getACRate(ac, total);
    },
    nicknameColor(nickname) {
      let typeArr = ['', 'success', 'info', 'danger', 'warning'];
      let index = nickname.length % 5;
      return typeArr[index];
    },
  },
  computed: {
    ...mapGetters(['isAuthenticated', 'userInfo']),
  },
};
</script>

<style scoped>
.echarts {
  margin: 0 auto;
  width: 100%;
  height: 400px;
}

@media screen and (max-width: 768px) {
  ::v-deep .el-card__body {
    padding: 0 !important;
  }
}

.user-avatar {
  margin-right: 5px !important;
  vertical-align: middle;
}

@media screen and (min-width: 768px) {
  .el-input-group {
    width: 50%;
  }
}

@media screen and (min-width: 1050px) {
  .el-input-group {
    width: 30%;
  }
}
</style>
