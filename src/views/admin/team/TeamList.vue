<template>
    <div>
        <el-card>
            <div slot="header">
                <span class="panel-title home-title">{{ $t('m.Team_List') }}</span>
                <div class="filter-row">
                    <span>
                        <vxe-input v-model="keyword" :placeholder="$t('m.Enter_keyword')" size="medium" type="search"
                            @search-click="filterByKeyword" @keyup.enter.native="filterByKeyword"></vxe-input>
                    </span>
                    <span>
                        <el-button icon="el-icon-plus" size="small" type="primary" @click="goCreateTeam">{{
                            $t('m.Create') }}
                        </el-button>
                    </span>
                </div>
            </div>
            <vxe-table ref="xTable" :data="teamList" :loading="loading" align="center" auto-resize stripe>
                <vxe-table-column :title="$t('m.Team_Name')" field="name" width="80">
                </vxe-table-column>
                <vxe-table-column :title="$t('m.Owner_Name')" field="ownerName" width="120" show-overflow>
                </vxe-table-column>
                <vxe-table-column :title="$t('m.Owner_RealName')" field="ownerRealName" width="120">
                </vxe-table-column>
                <vxe-table-column :title="$t('m.Team_Member_Count')" field="count" width="80">
                </vxe-table-column>
                <vxe-table-column :title="$t('m.Team_Description')" field="description" min-width="150">
                </vxe-table-column>
                <vxe-table-column :title="$t('m.Info')" min-width="150">
                    <template v-slot="{ row }">
                        <p>Created Time: {{ row.gmtCreate | localtime }}</p>
                        <p>Update Time: {{ row.gmtModified | localtime }}</p>
                    </template>
                </vxe-table-column>
                <vxe-table-column :title="$t('m.Option')" min-width="150">
                    <template v-slot="{ row }">
                        <template v-if="isSuperAdmin || userInfo.uid == row.ownerId">
                            <div style="margin-bottom:10px">
                                <el-tooltip :content="$t('m.Edit')" effect="dark" placement="top">
                                    <el-button icon="el-icon-edit" size="mini" type="primary"
                                        @click.native="goEdit(row.uuid)">
                                    </el-button>
                                </el-tooltip>
                                <!-- <el-tooltip :content="$t('m.View_Training_Problem_List')" effect="dark" placement="top">
                                    <el-button icon="el-icon-tickets" size="mini" type="success"
                                        @click.native="goTrainingProblemList(row.id)">
                                    </el-button>
                                </el-tooltip> -->
                            </div>
                        </template>
                        <el-tooltip v-if="isSuperAdmin||userInfo.uid == row.ownerId" :content="$t('m.Delete')" effect="dark" placement="top">
                            <el-button icon="el-icon-delete" size="mini" type="danger"
                                @click.native="deleteTeam(row.uuid)">
                            </el-button>
                        </el-tooltip>
                    </template>
                </vxe-table-column>
            </vxe-table>
            <div class="panel-options">
                <el-pagination :page-size="pageSize" :total="total" class="page" layout="prev, pager, next"
                    @current-change="currentChange">
                </el-pagination>
            </div>
        </el-card>
    </div>
</template>

<script>
import api from '@/common/api';
import { TRAINING_TYPE } from '@/common/constants';
import { mapGetters } from 'vuex';
import myMessage from '@/common/message';

export default {
    name: 'TeamList',
    data() {
        return {
            pageSize: 10,
            total: 0,
            teamList: [],
            keyword: '',
            loading: false,
            excludeAdmin: true,
            currentPage: 1,
            currentId: 1,
            downloadDialogVisible: false,
        };
    },
    mounted() {
        this.getTeamList(this.currentPage);
    },
    watch: {
        $route() {
            let refresh = this.$route.query.refresh == 'true' ? true : false;
            if (refresh) {
                this.getTeamList(1);
            }
        },
    },
    computed: {
        ...mapGetters(['isSuperAdmin', 'userInfo']),
    },
    methods: {
        // 切换页码回调
        currentChange(page) {
            this.currentPage = page;
            this.getTeamList(page);
        },
        getTeamList(page) {
            this.loading = true;
            api.admin_getTeamList(page, this.pageSize, this.keyword).then(
                (res) => {
                    this.loading = false;
                    this.total = res.data.data.total;
                    this.teamList = res.data.data.records;
                },
                (res) => {
                    this.loading = false;
                }
            );
        },
        goEdit(teamId) {
            this.$router.push({
                name: 'admin-edit-team',
                params: { teamId },
            });
        },
        deleteTeam(teamId) {
            this.$confirm(this.$i18n.t('m.Delete_Team_Tips'), 'Tips', {
                confirmButtonText: this.$i18n.t('m.OK'),
                cancelButtonText: this.$i18n.t('m.Cancel'),
                type: 'warning',
            }).then(() => {
                api.admin_deleteTeam(teamId).then((res)=>{
                    myMessage.success(this.$i18n.t('m.Delete_successfully'));
                    this.currentChange(1);
                })
            });
        },
        filterByKeyword() {
            this.currentChange(1);
        },
        goCreateTeam() {
            this.$router.push({ name: 'admin-create-team' });
        },
    },
};
</script>
<style scoped>
.filter-row {
    margin-top: 10px;
}

@media screen and (max-width: 768px) {
    .filter-row span {
        margin-right: 5px;
    }

    .filter-row span div {
        width: 80% !important;
    }
}

@media screen and (min-width: 768px) {
    .filter-row span {
        margin-right: 20px;
    }
}

.el-tag--dark {
    border-color: #fff;
}
</style>
