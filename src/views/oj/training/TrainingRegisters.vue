<template>
    <div style="margin-top:5px">
        <el-card shadow>
            <div slot="header" class="rank-title">
                <span class="panel-title">{{ $t('m.Training_Registers') }}</span>
            </div>
            <vxe-table ref="TrainingRegisters" :cell-class-name="cellClassName" :data="dataTrainingRegisters"
                :seq-config="{ startIndex: (this.page - 1) * this.limit }" align="center" auto-resize border round
                size="medium">
                <vxe-table-column field="rank" fixed="left" type="seq" width="50"></vxe-table-column>
                <vxe-table-column v-if="!isMobileView" :title="$t('m.User')" align="left" field="username" fixed="left"
                    header-align="center" min-width="300">
                    <template v-slot="{ row }">
                        <avatar :inline="true" :size="37" :src="row.avatar" :title="row.username"
                            :username="row.username" color="#FFF"></avatar>
                        <span style="float:right;text-align:right">
                            <a @click="getUserHomeByUsername(row.uid, row.username)">
                                <span class="contest-username"><span v-if="row.gender == 'female'"
                                        class="contest-rank-flag">Girl</span>{{ row.username }}</span>
                                <span v-if="row.school" class="contest-school">{{
                                    row.school
                                }}</span>
                            </a>
                        </span>
                    </template>
                </vxe-table-column>

                <vxe-table-column v-else :title="$t('m.User')" align="left" field="username" header-align="center"
                    min-width="300">
                    <template v-slot="{ row }">
                        <avatar :inline="true" :size="37" :src="row.avatar" :title="row.username"
                            :username="row.username" color="#FFF"></avatar>
                        <span style="float:right;text-align:right">
                            <a @click="getUserHomeByUsername(row.uid, row.username)">
                                <span class="contest-username"><span v-if="row.gender == 'female'"
                                        class="contest-rank-flag">Girl</span>{{ row.username }}</span>
                                <span v-if="row.school" class="contest-school">{{
                                    row.school
                                }}</span>
                            </a>
                        </span>
                    </template>
                </vxe-table-column>
                <vxe-table-column v-if="isTrainingAdmin" :title="$t('m.RealName')" field="realname" min-width="96"
                    show-overflow>
                </vxe-table-column>

                <vxe-table-column :title="$t('m.Option')" min-width="150" v-if="isTrainingAdmin">
                    <template v-slot="{ row }">
                        <template>
                            <div style="margin-bottom:10px">
                                <el-tooltip :content="$t('m.Notify')" effect="dark" placement="top">
                                    <el-button icon="el-icon-tickets" size="mini" type="warning"
                                        @click.native="notifyRegisters(row.uid)">
                                    </el-button>
                                </el-tooltip>
                                <el-tooltip :content="$t('m.Delete')" effect="dark" placement="top">
                                    <el-button icon="el-icon-delete" size="mini" type="danger"
                                        @click.native="deleteTrainingRegister(row.uid)">
                                    </el-button>
                                </el-tooltip>
                            </div>
                        </template>
                    </template>
                </vxe-table-column>
            </vxe-table>
            <Pagination :current.sync="page" :layout="'prev, pager, next, sizes'" :page-size.sync="limit" :total="total"
                @on-change="getTrainingRegistersData" @on-page-size-change="getTrainingRegistersData(1)"></Pagination>
        </el-card>
    </div>
</template>

<script>
import Avatar from 'vue-avatar';
import { mapActions, mapGetters, mapState } from 'vuex';
import { JUDGE_STATUS } from '@/common/constants';
import api from '@/common/api';
import time from '@/common/time';
import myMessage from '@/common/message';

const Pagination = () => import('@/components/oj/common/Pagination');

export default {
    name: 'TrainingRegisters',
    components: {
        Pagination,
        Avatar,
    },
    data() {
        return {
            total: 0,
            page: 1,
            limit: 30,
            trainingID: '',
            dataTrainingRegisters: [],
            JUDGE_STATUS: {},
        };
    },
    mounted() {
        this.JUDGE_STATUS = Object.assign({}, JUDGE_STATUS);
        this.trainingID = this.$route.params.trainingID;
        this.getTrainingRegistersData();
    },
    methods: {
        ...mapActions(['getTrainingProblemList']),

        getTrainingRegistersData() {
            let param = {
                tid: this.trainingID,
                limit: this.limit,
                currentPage: this.page,
            };
            api.getTrainingRegisters(param).then(
                (res) => {
                    this.total = res.data.data.total;
                    this.applyToTable(res.data.data.records);
                },
                (err) => {
                }
            );
        },

        getUserACSubmit(username) {
            this.$router.push({
                name: 'SubmissionList',
                query: { username: username, status: 0 },
            });
        },
        getUserHomeByUsername(uid, username) {
            this.$router.push({
                name: 'UserHome',
                query: { username: username, uid: uid },
            });
        },
        cellClassName({ row, rowIndex, column, columnIndex }) {
            if (column.property === 'username' && row.userCellClassName) {
                return row.userCellClassName;
            }
        },
        applyToTable(registers) {
            registers.forEach((rank, i) => {
                if (registers[i].gender == 'female') {
                    registers[i].userCellClassName = 'bg-female';
                }
            });
            this.dataTrainingRegisters = registers;
        },
        parseTotalTime(totalTime) {
            return time.secondFormat(totalTime);
        },
        getPassingRate(ac, total) {
            if (!total) {
                return 0;
            }
            return ((ac / total) * 100).toFixed(2);
        },
        notifyRegisters(uid) {
            let param = {
                tid: this.trainingID,
                uid: uid
            };
            api.notifyTrainingRegister(param).then((res) => {
                if(res.data.data==true){
                    myMessage.success('通知成功!');
                    return;
                }
                myMessage.warn('通知失败，请稍后再试!');
            }, (error) => {
                myMessage.warn('通知失败，请稍后再试!');
             });
        },
        deleteTrainingRegister(uid) {
            let param = {
                tid: this.trainingID,
                uid: uid
            };
            api.removeTrainingRegister(param).then((res) => {
                this.getTrainingRegistersData();
            }, (error) => { })
        }
    },
    computed: {
        ...mapGetters(['isTrainingAdmin']),
        training() {
            return this.$store.state.training.training;
        },
        isMobileView() {
            return window.screen.width < 768;
        },
    },
};
</script>
<style scoped>
.rank-title {
    text-align: center;
}

::v-deep .el-card__body {
    padding: 20px !important;
}

.vxe-cell p,
.vxe-cell span {
    margin: 0;
    padding: 0;
}

@media screen and (max-width: 768px) {
    ::v-deep .el-card__body {
        padding: 0 !important;
    }
}

a.emphasis {
    color: #495060 !important;
}

a.emphasis:hover {
    color: #2d8cf0 !important;
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
    height: 51px !important;
    box-sizing: border-box;
    text-align: left;
    text-overflow: ellipsis;
    vertical-align: middle;
}

::v-deep .vxe-table .vxe-cell {
    padding-left: 5px !important;
    padding-right: 5px !important;
}

.judge-status {
    font-size: 16px;
    font-weight: bold;
}

.judge-time {
    color: rgba(0, 0, 0, 0.45);
    font-size: 12px;
}
</style>
