<template>
  <div>
    <el-card>
      <div slot="header">
        <span class="panel-title home-title">{{
            query.contestId ? $t('m.Contest_Problem_List') : $t('m.Problem_List')
          }}</span>
        <div class="filter-row">
          <span>
            <el-button
                icon="el-icon-plus"
                size="small"
                type="primary"
                @click="goCreateProblem"
            >{{ $t('m.Create') }}
            </el-button>
          </span>
          <span v-if="query.contestId">
            <el-button
                icon="el-icon-plus"
                size="small"
                type="primary"
                @click="addProblemDialogVisible = true"
            >{{ $t('m.Add_From_Public_Problem') }}
            </el-button>
          </span>
          <!-- <span>
            <el-button
                icon="el-icon-plus"
                size="small"
                type="success"
                @click="AddRemoteOJProblemDialogVisible = true"
            >{{ $t('m.Add_Remote_OJ_Problem') }}
            </el-button>
          </span> -->
          <span>
            <vxe-input
                v-model="query.keyword"
                :placeholder="$t('m.Enter_keyword')"
                size="medium"
                type="search"
                @search-click="filterByKeyword"
                @keyup.enter.native="filterByKeyword"
            ></vxe-input>
          </span>

          <span>
            <el-select
                v-model="query.oj"
                size="small"
                style="width: 180px;"
                @change="ProblemListChangeFilter"
            >
              <el-option
                  :label="$t('m.All_Problem')"
                  :value="'All'"
              ></el-option>
              <el-option :label="$t('m.My_OJ')" :value="'LOCAL'"></el-option>
              <el-option
                  v-for="(remoteOj, index) in REMOTE_OJ"
                  :key="index"
                  :label="remoteOj.name"
                  :value="remoteOj.key"
              ></el-option>
            </el-select>
          </span>

          <span v-if="!query.contestId">
            <el-select
                v-model="query.problemListAuth"
                size="small"
                style="width: 180px;"
                @change="ProblemListChangeFilter"
            >
              <el-option :label="$t('m.All_Problem')" :value="0"></el-option>
              <el-option :label="$t('m.Public_Problem')" :value="1"></el-option>
              <el-option
                  :label="$t('m.Private_Problem')"
                  :value="2"
              ></el-option>
              <el-option
                  :label="$t('m.Contest_Problem')"
                  :value="3"
              ></el-option>
            </el-select>
          </span>
        </div>
      </div>
      <vxe-table
          ref="adminProblemList"
          :data="problemList"
          :loading="loading"
          align="center"
          auto-resize
          stripe
      >
        <vxe-table-column field="id" min-width="64" title="ID">
        </vxe-table-column>
        <vxe-table-column
            v-if="!isContest"
            :title="$t('m.Display_ID')"
            field="problemId"
            min-width="120"
            show-overflow
        >
        </vxe-table-column>

        <vxe-table-column
            v-else
            :title="$t('m.Original_Display')"
            align="left"
            min-width="120"
        >
          <template v-slot="{ row }">
            <p v-if="query.contestId">
              {{ $t('m.Display_ID') }}：{{ row.problemId }}
            </p>
            <p v-if="query.contestId">{{ $t('m.Title') }}：{{ row.title }}</p>
            <span v-else>{{ row.problemId }}</span>
          </template>
        </vxe-table-column>

        <vxe-table-column
            v-if="!isContest"
            :title="$t('m.Title')"
            field="title"
            min-width="150"
            show-overflow
        >
        </vxe-table-column>

        <vxe-table-column
            v-else
            :title="$t('m.Contest_Display')"
            align="left"
            min-width="150"
        >
          <template v-slot="{ row }">
            <p v-if="contestProblemMap[row.id]">
              {{ $t('m.Display_ID') }}：{{
                contestProblemMap[row.id]['displayId']
              }}
            </p>
            <p v-if="contestProblemMap[row.id]">
              {{ $t('m.Title') }}：{{
                contestProblemMap[row.id]['displayTitle']
              }}
            </p>
            <span v-if="contestProblemMap[row.id]">
              {{ $t('m.Balloon_Color') }}：<el-color-picker
                v-model="contestProblemMap[row.id].color"
                :predefine="predefineColors"
                show-alpha
                size="small"
                style="vertical-align: middle;"
                @change="
                  changeContestProblemColor(
                    contestProblemMap[row.id].id,
                    contestProblemMap[row.id].color
                  )
                "
            >
              </el-color-picker>
            </span>
            <span v-else>{{ row.title }}</span>
          </template>
        </vxe-table-column>

        <vxe-table-column
            :title="$t('m.Author')"
            field="author"
            min-width="120"
            show-overflow
        >
        </vxe-table-column>
        <vxe-table-column :title="$t('m.Created_Time')" min-width="130">
          <template v-slot="{ row }">
            {{ row.gmtCreate | localtime }}
          </template>
        </vxe-table-column>
        <vxe-table-column
            :title="$t('m.Modified_User')"
            field="modifiedUser"
            min-width="96"
            show-overflow
        >
        </vxe-table-column>
        <vxe-table-column :title="$t('m.Auth')" min-width="120">
          <template v-slot="{ row }">
            <el-select
                v-model="row.auth"
                :disabled="!isSuperAdmin && !isProblemAdmin && !query.contestId"
                size="small"
                @change="changeProblemAuth(row)"
            >
              <el-option
                  :disabled="!isSuperAdmin && !isProblemAdmin"
                  :label="$t('m.Public_Problem')"
                  :value="1"
              ></el-option>
              <el-option
                  :label="$t('m.Private_Problem')"
                  :value="2"
              ></el-option>
              <el-option
                  :disabled="!query.contestId"
                  :label="$t('m.Contest_Problem')"
                  :value="3"
              ></el-option>
            </el-select>
          </template>
        </vxe-table-column>
        <vxe-table-column min-width="200" title="Option">
          <template v-slot="{ row }">
            <el-tooltip
                v-if="
                isSuperAdmin ||
                  isProblemAdmin ||
                  row.author == userInfo.username
              "
                :content="$t('m.Edit')"
                effect="dark"
                placement="top"
            >
              <el-button
                  icon="el-icon-edit-outline"
                  size="mini"
                  type="primary"
                  @click.native="goEdit(row.id)"
              >
              </el-button>
            </el-tooltip>

            <el-tooltip
                v-if="isSuperAdmin || isProblemAdmin"
                :content="$t('m.Download_Testcase')"
                effect="dark"
                placement="top"
            >
              <el-button
                  icon="el-icon-download"
                  size="mini"
                  type="success"
                  @click.native="downloadTestCase(row.id)"
              >
              </el-button>
            </el-tooltip>

            <el-tooltip
                v-if="query.contestId"
                :content="$t('m.Remove')"
                effect="dark"
                placement="top"
            >
              <el-button
                  icon="el-icon-close"
                  size="mini"
                  type="warning"
                  @click.native="removeProblem(row.id)"
              >
              </el-button>
            </el-tooltip>

            <el-tooltip
                v-if="!isContest && (isSuperAdmin || isProblemAdmin)"
                :content="$t('m.Delete')"
                effect="dark"
                placement="top"
            >
              <el-button
                  icon="el-icon-delete-solid"
                  size="mini"
                  type="danger"
                  @click.native="deleteProblem(row.id)"
              >
              </el-button>
            </el-tooltip>
          </template>
        </vxe-table-column>
      </vxe-table>

      <div class="panel-options">
        <el-pagination
            v-if="showPagination"
            :current-page.sync="query.currentPage"
            :page-size="query.pageSize"
            :page-sizes="[10, 30, 50, 100]"
            :total="total"
            class="page"
            layout="prev, pager, next, sizes"
            @current-change="currentChange"
            @size-change="onPageSizeChange"
        >
        </el-pagination>
      </div>
    </el-card>

    <el-dialog
        v-if="query.contestId"
        :title="$t('m.Add_Contest_Problem')"
        :visible.sync="addProblemDialogVisible"
        width="90%"
        :close-on-click-modal="false"
    >
      <AddPublicProblem
          :contestID="query.contestId"
          @on-change="getProblemList"
          @getTips="getContestProblemTips"
      ></AddPublicProblem>
    </el-dialog>

    <el-dialog
        :title="$t('m.Add_Remote_OJ_Problem')"
        :visible.sync="AddRemoteOJProblemDialogVisible"
        width="350px"
        :close-on-click-modal="false"
    >
      <el-form>
        <el-form-item :label="$t('m.Remote_OJ')">
          <el-select v-model="otherOJName" size="small">
            <el-option
                v-for="(remoteOj, index) in REMOTE_OJ"
                :key="index"
                :label="remoteOj.name"
                :value="remoteOj.key"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('m.Problem_ID')" required>
          <el-input v-model="otherOJProblemId" size="small"></el-input>
        </el-form-item>

<!--        <el-form-item-->
<!--            v-if="query.contestId"-->
<!--            :label="$t('m.Enter_The_Problem_Display_ID_in_the_Contest')"-->
<!--            required-->
<!--        >-->
<!--          <el-input v-model="displayId" size="small"></el-input>-->
<!--        </el-form-item>-->

        <el-form-item style="text-align:center">
          <el-button
              :loading="addRemoteOJProblemLoading"
              icon="el-icon-plus"
              type="primary"
              @click="addRemoteOJProblem"
          >{{ $t('m.Add') }}
          </el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import api from '@/common/api';
import utils from '@/common/utils';
import AddPublicProblem from '@/components/admin/AddPublicProblem.vue';
import myMessage from '@/common/message';
import {REMOTE_OJ} from '@/common/constants';
import {mapGetters} from 'vuex';

export default {
  name: 'ProblemList',
  components: {
    AddPublicProblem,
  },
  data() {
    return {
      total: 0,
      query: {
        problemListAuth: 0,
        oj: 'All',
        pageSize: 10,
        keyword: '',
        currentPage: 1,
        contestId: null,
      },
      problemList: [],
      contestProblemMap: {},
      loading: false,
      routeName: '',
      // for make public use
      currentProblemID: '',
      currentRow: {},
      addProblemDialogVisible: false,
      AddRemoteOJProblemDialogVisible: false,
      addRemoteOJProblemLoading: false,
      otherOJName: REMOTE_OJ[0].key,
      otherOJProblemId: '',
      REMOTE_OJ: {},
      displayId: '',
      contestProblemTips: '',
      showPagination: false,

      predefineColors: [
        '#ff4500',
        '#ff8c00',
        '#ffd700',
        '#90ee90',
        '#00ced1',
        '#1e90ff',
        '#c71585',
      ],
    };
  },
  mounted() {
    this.init();
  },
  computed: {
    ...mapGetters(['userInfo', 'isSuperAdmin', 'isProblemAdmin']),
    isContest() {
      return !(this.routeName == 'admin-problem-list' && !this.query.contestId);
    },
  },
  methods: {
    init() {
      this.routeName = this.$route.name;
      let query = this.$route.query;
      this.query.currentPage = parseInt(query.currentPage) || 1;
      this.query.pageSize = parseInt(query.pageSize) || 10;
      this.query.keyword = query.keyword;
      this.query.problemListAuth = query.problemListAuth
          ? parseInt(query.problemListAuth)
          : 0;
      this.query.oj = query.oj || 'All';
      this.query.contestId = this.$route.params.contestId;
      this.contestProblemMap = {};
      this.getProblemList();
      this.REMOTE_OJ = Object.assign({}, REMOTE_OJ);
    },
    getContestProblemTips(data){
      this.contestProblemTips = data
    },
    goEdit(problemId) {
      if (this.routeName === 'admin-problem-list') {
        this.$router.push({
          name: 'admin-edit-problem',
          params: {problemId},
          query: {
            back: this.$route.fullPath,
          },
        });
      } else if (this.routeName === 'admin-contest-problem-list') {
        this.$router.push({
          name: 'admin-edit-contest-problem',
          params: {problemId: problemId, contestId: this.query.contestId},
        });
      }
    },
    goCreateProblem() {
      if (this.routeName === 'admin-problem-list') {
        this.$router.push({
          name: 'admin-create-problem',
          query: {
            back: this.$route.fullPath,
          },
        });
      } else if (this.routeName === 'admin-contest-problem-list') {
        this.$router.push({
          name: 'admin-create-contest-problem',
          params: {contestId: this.query.contestId},
        });
      }
    },

    pushRouter() {
      if (this.query.contestId) {
        this.$router.push({
          name: 'admin-contest-problem-list',
          query: this.query,
          params: {
            contestId: this.query.contestId,
          },
        });
      } else {
        this.$router.push({
          name: 'admin-problem-list',
          query: this.query,
        });
      }
    },

    // 切换页码回调
    currentChange(page) {
      this.query.currentPage = page;
      this.pushRouter();
    },
    onPageSizeChange(pageSize) {
      this.query.pageSize = pageSize;
      this.pushRouter();
    },
    getProblemList() {
      let params = {
        limit: this.query.pageSize,
        currentPage: this.query.currentPage,
        keyword: this.query.keyword,
        cid: this.query.contestId,
        oj: this.query.oj,
      };
      if (this.problemListAuth != 0) {
        params['auth'] = this.query.problemListAuth;
      }
      this.loading = true;
      if (this.routeName === 'admin-problem-list') {
        api.admin_getProblemList(params).then(
            (res) => {
              this.loading = false;
              this.showPagination = true;
              this.total = res.data.data.total;
              this.problemList = res.data.data.records;
              this.contestProblemMap = {};
            },
            (err) => {
              this.loading = false;
              this.showPagination = true;
            }
        );
      } else {
        api.admin_getContestProblemList(params).then(
            (res) => {
              this.loading = false;
              this.showPagination = false;
              this.total = res.data.data.problemList.total;
              this.problemList = res.data.data.problemList.records;
              this.contestProblemMap = res.data.data.contestProblemMap;
            },
            (err) => {
              this.loading = false;
              this.showPagination = false;
            }
        );
      }
    },

    changeProblemAuth(row) {
      api.admin_changeProblemAuth(row).then((res) => {
        myMessage.success(this.$i18n.t('m.Update_Successfully'));
      });
    },

    deleteProblem(id) {
      this.$confirm(this.$i18n.t('m.Delete_Problem_Tips'), 'Tips', {
        type: 'warning',
      }).then(
          () => {
            let funcName =
                this.routeName === 'admin-problem-list'
                    ? 'admin_deleteProblem'
                    : 'admin_deleteContestProblem';
            api[funcName](id, null)
                .then((res) => {
                  myMessage.success(this.$i18n.t('m.Delete_successfully'));
                  this.getProblemList();
                })
                .catch(() => {
                });
          },
          () => {
          }
      );
    },
    removeProblem(pid) {
      this.$confirm(this.$i18n.t('m.Remove_Contest_Problem_Tips'), 'Tips', {
        type: 'warning',
      }).then(
          () => {
            api
                .admin_deleteContestProblem(pid, this.query.contestId)
                .then((res) => {
                  myMessage.success('success');
                  this.getProblemList();
                })
                .catch(() => {
                });
          },
          () => {
          }
      );
    },
    updateProblem(row) {
      let data = Object.assign({}, row);
      let funcName = '';
      if (this.query.contestId) {
        data.contest_id = this.query.contestId;
        funcName = 'admin_editContestProblem';
      } else {
        funcName = 'admin_editProblem';
      }
      api[funcName](data)
          .then((res) => {
            myMessage.success(this.$i18n.t('m.Update_Successfully'));
            this.getProblemList();
          })
          .catch(() => {
          });
    },
    downloadTestCase(problemID) {
      let url = '/api/file/download-testcase?pid=' + problemID;
      utils.downloadFile(url).then(() => {
        this.$alert(this.$i18n.t('m.Download_Testcase_Success'), 'Tips');
      });
    },
    ProblemListChangeFilter() {
      this.pushRouter();
    },
    filterByKeyword() {
      this.query.currentPage = 1;
      this.pushRouter();
    },
    addRemoteOJProblem() {
      if (!this.otherOJProblemId) {
        myMessage.error(this.$i18n.t('m.Problem_ID_is_required'));
        return;
      }

      // if (!this.displayId && this.query.contestId) {
      //   myMessage.error(
      //       this.$i18n.t('m.The_Problem_Display_ID_in_the_Contest_is_required')
      //   );
      //   return;
      // }

      this.addRemoteOJProblemLoading = true;
      let funcName = '';
      if (this.query.contestId) {
        funcName = 'admin_addContestRemoteOJProblem';
      } else {
        funcName = 'admin_addRemoteOJProblem';
      }
      api[funcName](
          this.otherOJName,
          this.otherOJProblemId,
          this.query.contestId,
          this.displayId
      ).then(
          (res) => {
            this.addRemoteOJProblemLoading = false;
            this.AddRemoteOJProblemDialogVisible = false;
            myMessage.success(this.$i18n.t('m.Add_Successfully'));
            this.currentChange(1);
          },
          (err) => {
            this.addRemoteOJProblemLoading = false;
          }
      );
    },
    changeContestProblemColor(id, color) {
      let data = {
        id: id,
        color: color,
      };
      api.admin_setContestProblemInfo(data).then((res) => {
        myMessage.success(this.$i18n.t('m.Update_Balloon_Color_Successfully'));
      });
    },
  },
  watch: {
    $route(newVal, oldVal) {
      this.init();
    },
  },
};
</script>

<style scoped>
.filter-row span button {
  margin-top: 5px;
  margin-bottom: 5px;
}

.filter-row span div {
  margin-top: 8px;
}

@media screen and (max-width: 768px) {
  .filter-row span {
    margin-right: 5px;
  }

  .filter-row span div {
    width: 80%;
  }
}

@media screen and (min-width: 768px) {
  .filter-row span {
    margin-right: 20px;
  }
}
</style>
