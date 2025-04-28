<template>
    <div class="view">
        <el-card>
            <div slot="header">
                <span class="panel-title home-title">
                    {{ title }}
                </span>
            </div>
            <el-form label-position="top">
                <el-row :gutter="20">
                    <el-col :span="24">
                        <el-form-item :label="$t('m.Team_Name')" required>
                            <el-input v-model="team.name" :placeholder="$t('m.Team_Name')"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item :label="$t('m.Team_Description')" required>
                            <Editor :value.sync="team.description"></Editor>
                        </el-form-item>
                    </el-col>

                </el-row>
            </el-form>
            <div v-if="$route.name === 'admin-edit-team'">
                <p class="panel-title" style="text-align: center;">{{ $t('m.Team_Member_Manage') }}</p>
                <vxe-toolbar :tools="teamMemberToolbarTools" @tool-click="teamMemberToolClickEvent"></vxe-toolbar>
                <vxe-table ref="xTable" :data="teamMembers" :loading="teamMemberLoading" align="center" auto-resize
                    stripe @checkbox-change="handleTeamMemberSelectChange"
                    @checkbox-all="handleTeamMemberSelectChangeAll">
                    <vxe-column type="checkbox" width="60"></vxe-column>
                    <vxe-table-column :title="$t('m.Team_Member_Name')" field="username" width="80">
                    </vxe-table-column>
                    <vxe-table-column :title="$t('m.Team_Member_Student_Id')" field="number" min-width="150"
                        show-overflow>
                    </vxe-table-column>
                    <vxe-table-column :title="$t('m.Team_Member_RealName')" field="realname" min-width="150"
                        show-overflow>
                    </vxe-table-column>
                    <vxe-table-column :title="$t('m.Option')" min-width="150">
                        <template v-slot="{ row }">
                            <el-tooltip v-if="isSuperAdmin || user_info.uid == team.owner_id"
                                :content="$t('m.Remove_Member')" effect="dark" placement="top">
                                <el-button icon="el-icon-minus" size="mini" type="danger"
                                    @click.native="removeUserFromTeam(row.id, row.uid)">
                                </el-button>
                            </el-tooltip>
                        </template>
                    </vxe-table-column>
                </vxe-table>
                <div class="panel-options">
                    <el-pagination :page-size="teamMemberPageSize" :total="teamMemberTotal" class="page"
                        layout="prev, pager, next" @current-change="teamMemberPageChange">
                    </el-pagination>
                </div>

                <p class="panel-title" style="text-align: center;">{{ $t('m.Team_Add_Member') }}</p>
                <vxe-toolbar :tools="userToolbarTools" @tool-click="userToolClickEvent"></vxe-toolbar>
                <vxe-table ref="usersTable" :data="users" :loading="usersLoading" align="center" auto-resize stripe
                    @checkbox-change="handleUserSelectChange" @checkbox-all="handleUserSelectChangeAll">
                    <vxe-column type="checkbox" width="60"></vxe-column>
                    <vxe-table-column :title="$t('m.Show_Username')" field="username" width="80">
                    </vxe-table-column>
                    <vxe-table-column :title="$t('m.Team_Member_Student_Id')" field="number" min-width="150"
                        show-overflow>
                    </vxe-table-column>
                    <vxe-table-column :title="$t('m.Team_Member_RealName')" field="realname" min-width="150"
                        show-overflow>
                    </vxe-table-column>
                    <vxe-table-column :title="$t('m.Option')" min-width="150">
                        <template v-slot="{ row }">
                            <template v-if="isSuperAdmin || userInfo.uid == team.owner_id">
                                <div style="margin-bottom:10px">
                                    <el-tooltip :content="$t('m.Add')" effect="dark" placement="top">
                                        <el-button icon="el-icon-plus" size="mini" type="primary"
                                            @click.native="addUserToTeam(row.uid)">
                                        </el-button>
                                    </el-tooltip>
                                </div>
                            </template>
                        </template>
                    </vxe-table-column>
                </vxe-table>
                <div class="panel-options">
                    <el-pagination :page-size="usersPageSize" :total="usersTotal" class="page"
                        layout="prev, pager, next" @current-change="usersPageChange">
                    </el-pagination>
                </div>
            </div>

            <el-button type="primary" @click.native="saveTeam">{{
                $t('m.Save')
            }}
            </el-button>
        </el-card>
    </div>
</template>

<script>
import api from '@/common/api';
import { mapGetters } from 'vuex';
import myMessage from '@/common/message';
import { duration } from 'moment';

const Editor = () => import('@/components/admin/Editor.vue');
export default {
    name: 'Team',
    components: {
        Editor,
    },
    data() {
        return {
            title: 'Create Team',
            team: {
                name: '',
                description: '',
            },
            teamMembers: [],
            teamMemberLoading: false,
            teamMemberCurrentPage: 1,
            teamMemberPageSize: 10,
            teamMemberTotal: 0,
            teamMemberSelected: [],
            users: [],
            usersLoading: false,
            usersCurrentPage: 1,
            usersPageSize: 10,
            usersTotal: 0,
            usersSelected: [],
            teamMemberToolbarTools: [
                { name: '批量移出', code: 'removeBatch', status: 'error' },
            ],
            userToolbarTools: [
                { name: '批量添加', code: 'addBatch', status: 'primary' },
            ]
        };
    },
    mounted() {
        this.init();
    },
    watch: {
        async $route() {
            if (this.$route.name === 'admin-edit-team') {
                this.title = this.$i18n.t('m.Edit_Team');
                await this.getTeam();
                this.getTeamMembers(1);
                this.getAddUser(1);
            } else {
                this.title = this.$i18n.t('m.Create_Team');
                this.team = {
                    name: '',
                    description: '',
                };
            }
        },
    },
    computed: {
        ...mapGetters(['isSuperAdmin', 'userInfo']),
    },
    methods: {
        async init() {
            if (this.$route.name === 'admin-edit-team') {
                this.title = this.$i18n.t('m.Edit_Team');
                await this.getTeam();
                this.getTeamMembers(1);
                this.getAddUser(1);
            } else {
                this.title = this.$i18n.t('m.Create_Team');
            }
        },
        // 切换页码回调
        teamMemberPageChange(page) {
            this.teamMemberCurrentPage = page;
            this.getTeamMembers(page);
        },
        usersPageChange(page) {
            this.usersCurrentPage = page;
            this.getAddUser(page);
        },
        teamMemberToolClickEvent({ code }) {
            if (code == 'removeBatch') {
                if (this.teamMemberSelected.length == 0) {
                    myMessage.error('请先选中');
                    return;
                }
                this.removeUsersFromTeam(this.teamMemberSelected);
            }
        },
        userToolClickEvent({ code }) {
            if (code == 'addBatch') {
                if (this.usersSelected.length == 0) {
                    myMessage.error('请先选中');
                    return;
                }
                this.addUsersToTeam(this.usersSelected);

            }
        },
        getTeam() {
            api.admin_getTeam(this.$route.params.teamId)
                .then((res) => {
                    const data = res.data.data;
                    this.team = data;
                }).catch(() => {

                });
        },
        saveTeam() {
            if (!this.team.name) {
                myMessage.error(
                    this.$i18n.t('m.Team_Name') + ' ' + this.$i18n.t('m.is_required')
                );
                return;
            }
            if (!this.team.description) {
                myMessage.error(
                    this.$i18n.t('m.Team_Description') +
                    ' ' +
                    this.$i18n.t('m.is_required')
                );
                return;
            }
            let data = {
                uuid: this.team.uuid,
                name: this.team.name,
                description: this.team.description
            };
            api.admin_editOrCreateTeam(data).then((res) => {
                myMessage.success('success');
                this.$router.push({
                    name: 'admin-team-list',
                    query: { refresh: 'true' },
                });
            }).catch(() => {

            })
        },
        getTeamMembers(page) {
            this.teamMemberLoading = true;
            api.admin_getTeamMembers(page, this.teamMemberPageSize, this.$route.params.teamId).then((res) => {
                this.teamMemberLoading = false;
                this.teamMemberTotal = res.data.data.total;
                this.teamMembers = res.data.data.records;
            }, (res) => {
                this.loading = false;
            })
        },
        getAddUser(page) {
            this.usersLoading = true;
            api.admin_getAddUsers(page, this.usersPageSize, this.$route.params.teamId).then((res) => {
                this.usersLoading = false;
                this.usersTotal = res.data.data.total;
                this.users = res.data.data.records;
            }, (res) => {
                this.usersLoading = false;
            })
        },
        addUserToTeam(userId) {
            const data = {
                teamId: this.$route.params.teamId,
                memberId: [userId]
            };
            api.admin_addUserToTeam(data).then((res) => {
                myMessage.success('success');
                this.teamMemberPageChange(1);
                this.usersPageChange(1);
            }, (res) => {
                myMessage.error('failed');
            })
        },
        addUsersToTeam(userIds) {
            const data = {
                teamId: this.$route.params.teamId,
                memberId: userIds
            };
            api.admin_addUserToTeam(data).then((res) => {
                myMessage.success('success');
                this.teamMemberPageChange(1);
                this.usersPageChange(1);
            }, (res) => {
                myMessage.error('failed');
            })
        },
        removeUserFromTeam(id, memberId) {
            const data = {
                teamId: this.$route.params.teamId,
                memberId: [memberId]
            };
            api.admin_removeUserFromTeam(data).then((res) => {
                myMessage.success('success');
                this.usersPageChange(1);
                this.teamMemberPageChange(1);
            }, (res) => {
                myMessage.error('failed');
            })
        },
        removeUsersFromTeam(memberIds) {
            const data = {
                teamId: this.$route.params.teamId,
                memberId: memberIds
            };
            api.admin_removeUserFromTeam(data).then((res) => {
                myMessage.success('success');
                this.usersPageChange(1);
                this.teamMemberPageChange(1);
            }, (res) => {
                myMessage.error('failed');
            })
        },
        handleTeamMemberSelectChange({ records }) {
            this.teamMemberSelected = [];
            for (let i = 0; i < records.length; i += 1) {
                this.teamMemberSelected.push(records[i].uid);
            }
        },
        handleTeamMemberSelectChangeAll() {
            const records = this.$refs.xTable.getCheckboxRecords();
            this.teamMemberSelected = [];
            for (let i = 0; i < records.length; i += 1) {
                this.teamMemberSelected.push(records[i].uid);
            }
        },
        handleUserSelectChange({ records }) {
            this.usersSelected = [];
            for (let i = 0; i < records.length; i++) {
                this.usersSelected.push(records[i].uid);
            }
        },
        handleUserSelectChangeAll() {
            const records = this.$refs.usersTable.getCheckboxRecords();
            this.usersSelected = [];
            for (let i = 0; i < records.length; i += 1) {
                this.usersSelected.push(records[i].uid);
            }
        },
    },
};
</script>
<style scoped>
.userPreview {
    padding-left: 10px;
    padding-top: 20px;
    padding-bottom: 20px;
    color: red;
    font-size: 16px;
    margin-bottom: 10px;
}
</style>
