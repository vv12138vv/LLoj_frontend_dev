<template>
  <div>
    <el-card>
      <div slot="header">
        <span class="panel-title home-title">{{ $t('m.SysNotice') }}</span>
        <div style="font-size:13px;margin-top: 5px;color: red;">
          {{ $t('m.Push_System_Notification_Every_Hour') }}
        </div>
      </div>
      <div class="create">
        <el-button
            icon="el-icon-plus"
            size="small"
            type="primary"
            @click="openNoticeDialog(null)"
        >{{ $t('m.Create') }}
        </el-button
        >
      </div>
      <div class="list">
        <vxe-table
            ref="table"
            :data="noticeList"
            :loading="loading"
            auto-resize
            stripe
        >
          <vxe-table-column field="id" min-width="50" title="ID">
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Notice_Title')"
              field="title"
              min-width="150"
              show-overflow
          >
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Created_Time')"
              field="gmtCreate"
              min-width="150"
          >
            <template v-slot="{ row }">
              {{ row.gmtCreate | localtime }}
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Modified_Time')"
              field="gmtModified"
              min-width="150"
          >
            <template v-slot="{ row }">
              {{ row.gmtModified | localtime }}
            </template>
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Author')"
              field="adminUsername"
              min-width="150"
              show-overflow
          >
          </vxe-table-column>
          <vxe-table-column
              :title="$t('m.Notice_Push')"
              field="state"
              min-width="100"
          >
          </vxe-table-column>
          <vxe-table-column :title="$t('m.Option')" min-width="150">
            <template v-slot="row">
              <el-tooltip
                  :content="$t('m.Edit_Notice')"
                  class="item"
                  effect="dark"
                  placement="top"
              >
                <el-button
                    icon="el-icon-edit-outline"
                    size="mini"
                    type="primary"
                    @click.native="openNoticeDialog(row.row)"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                  :content="$t('m.Delete_Notice')"
                  class="item"
                  effect="dark"
                  placement="top"
              >
                <el-button
                    icon="el-icon-delete-solid"
                    size="mini"
                    type="danger"
                    @click.native="deleteNotice(row.row.id)"
                ></el-button>
              </el-tooltip>
            </template>
          </vxe-table-column>
        </vxe-table>

        <div class="panel-options">
          <el-pagination
              :page-size="pageSize"
              :total="total"
              class="page"
              layout="prev, pager, next"
              @current-change="currentChange"
          >
          </el-pagination>
        </div>
      </div>
    </el-card>

    <!--编辑通知对话框-->
    <el-dialog
        :fullscreen="true"
        :title="noticeDialogTitle"
        :visible.sync="showEditNoticeDialog"
        @open="onOpenEditDialog"
    >
      <el-form :model="notice" label-position="top">
        <el-form-item :label="$t('m.Notice_Title')" required>
          <el-input
              v-model="notice.title"
              :placeholder="$t('m.Notice_Title')"
              class="title-input"
          >
          </el-input>
        </el-form-item>
        <el-form-item :label="$t('m.Notice_Content')" required>
          <Editor :value.sync="notice.content"></Editor>
        </el-form-item>
        <div class="visible-box">
          <span>{{ $t('m.Notice_Recipient') }}</span>
          <span>
            <el-radio v-model="notice.type" label="All">{{
                $t('m.All_User')
              }}</el-radio>
            <!-- <el-radio v-model="notice.type" disabled label="Single">{{
                $t('m.Designated_User')
              }}</el-radio>
            <el-radio v-model="notice.type" disabled label="Admin">{{
                $t('m.All_Admin')
              }}</el-radio> -->
          </span>
        </div>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="danger" @click.native="showEditNoticeDialog = false">{{
            $t('m.Cancel')
          }}</el-button>
        <el-button type="primary" @click.native="submitNotice">{{
            $t('m.OK')
          }}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import api from '@/common/api';
import myMessage from '@/common/message';
import {mapGetters} from 'vuex';

const Editor = () => import('@/components/admin/Editor.vue');
export default {
  name: 'Notice',
  components: {
    Editor,
  },
  data() {
    return {
      contestID: '',
      // 显示编辑通知对话框
      showEditNoticeDialog: false,
      // 通知列表
      noticeList: [],
      // 一页显示的通知数
      pageSize: 10,
      // 总通知数
      total: 0,
      mode: 'create',
      // 通知 (new | edit) model

      notice: {
        id: null,
        title: '',
        content: '',
        type: '',
        adminId: '',
      },
      // 对话框标题
      noticeDialogTitle: 'Edit Notice',
      // 是否显示loading
      loading: false,
      // 当前页码
      currentPage: 1,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.getNoticeList(1);
    },
    // 切换页码回调
    currentChange(page) {
      this.currentPage = page;
      if (this.contestID) {
        this.getContestNoticeList(page);
      } else {
        this.getNoticeList(page);
      }
    },

    getNoticeList(page) {
      this.loading = true;
      api.admin_getNoticeList(page, this.pageSize, 'All').then(
          (res) => {
            this.loading = false;
            this.total = res.data.data.total;
            this.noticeList = res.data.data.records;
          },
          (res) => {
            this.loading = false;
          }
      );
    },
    // 打开编辑对话框的回调
    onOpenEditDialog() {
      // todo 优化
      // 暂时解决 文本编辑器显示异常bug
      setTimeout(() => {
        if (document.createEvent) {
          let event = document.createEvent('HTMLEvents');
          event.initEvent('resize', true, true);
          window.dispatchEvent(event);
        } else if (document.createEventObject) {
          window.fireEvent('onresize');
        }
      }, 0);
    },
    // 提交编辑
    // 默认传入MouseEvent
    submitNotice(data = undefined) {
      if (!data.id) {
        data = this.notice;
      }
      let funcName =
          this.mode === 'edit' ? 'admin_updateNotice' : 'admin_createNotice';
      let requestData = data;

      api[funcName](requestData)
          .then((res) => {
            this.showEditNoticeDialog = false;
            myMessage.success(this.$i18n.t('m.Post_successfully'));
            this.init();
          })
          .catch();
    },

    // 删除通知
    deleteNotice(noticeId) {
      this.$confirm(this.$i18n.t('m.Delete_Notice_Tips'), 'Warning', {
        confirmButtonText: this.$i18n.t('m.OK'),
        cancelButtonText: this.$i18n.t('m.Cancel'),
        type: 'warning',
      })
          .then(() => {
            // then 为确定
            this.loading = true;
            let funcName = 'admin_deleteNotice';
            api[funcName](noticeId).then((res) => {
              this.loading = true;
              myMessage.success(res.data.msg);
              this.init();
            });
          })
          .catch(() => {
            // catch 为取消
            this.loading = false;
          });
    },

    openNoticeDialog(row) {
      this.showEditNoticeDialog = true;
      if (row !== null) {
        this.noticeDialogTitle = this.$i18n.t('m.Edit_Notice');
        this.notice = Object.assign({}, row);
        this.mode = 'edit';
      } else {
        this.noticeDialogTitle = this.$i18n.t('m.Create_Notice');
        this.notice.title = '';
        this.notice.content = '';
        this.notice.type = 'All';
        this.notice.adminId = this.userInfo.uid;
        this.mode = 'create';
      }
    },
  },
  watch: {
    $route() {
      this.init();
    },
  },
  computed: {
    ...mapGetters(['userInfo']),
  },
};
</script>

<style scoped>
.title-input {
  margin-bottom: 20px;
}

.visible-box {
  margin-top: 10px;
  float: left;
}

.visible-box span {
  margin-right: 10px;
}

.el-form-item {
  margin-bottom: 2px !important;
}

::v-deep .el-dialog__body {
  padding-top: 0 !important;
}

.create {
  margin-bottom: 5px;
}
</style>
