<template>
    <div>
        <el-card class="training-header" shadow>
            <div slot="header">
                <span class="panel-title">{{ training.title }}</span>
            </div>
            <template v-if="isAuthenticated">
                <div>
                    <el-tooltip :content="training.acCount + '/' + training.problemCount" effect="dark" placement="top">
                        <el-progress :color="customColors" :percentage="getAcProblemPercent()" :stroke-width="20"
                            :text-inside="true"></el-progress>
                    </el-tooltip>
                </div>
                <div class="count">
                    {{ training.acCount + ' / ' + training.problemCount }}
                </div>
            </template>
        </el-card>
        <div class="card-top">
            <el-tabs v-model="route_name" @tab-click="tabClick">
                <el-tab-pane lazy name="TrainingDetails">
                    <span slot="label"><i class="el-icon-s-home"></i>&nbsp;{{
                        $t('m.Training_Introduction')
                    }}</span>
                    <el-row :gutter="30">
                        <el-col :md="7" :sm="24">
                            <el-card
                                v-if="trainingPasswordFormVisible && (!training.limitTime || training.limitTime && ifStart && !ifEnd)"
                                class="password-form-card">
                                <div slot="header">
                                    <span class="panel-title" style="color: #e6a23c;"><i class="el-icon-warning">
                                            {{ $t('m.Password_Required') }}</i></span>
                                </div>
                                <h3>
                                    {{ $t('m.To_Enter_Training_Need_Password') }}
                                </h3>
                                <el-form>
                                    <el-input v-model="trainingPassword"
                                        :placeholder="$t('m.Enter_the_training_password')" style="width:70%"
                                        type="password" @keydown.enter.native="checkPassword" />
                                    <el-button :loading="btnLoading" style="margin:5px" type="primary"
                                        @click="checkPassword">{{
                                            $t('m.OK') }}
                                    </el-button>
                                </el-form>
                            </el-card>
                            <el-card v-else class="status-card">
                                <div style="justify-content: center;">
                                    <span v-if="training.limitTime && ifEnd" class="status-tag status-ended">
                                        <span>该训练已结束</span>
                                    </span>
                                    <span v-else-if="training.limitTime && !ifStart"
                                        class="status-tag status-not-started">
                                        <span>该训练未开始</span>
                                    </span>
                                </div>
                            </el-card>
                            <el-card>
                                <div class="info-rows">
                                    <div>
                                        <span>
                                            <span>{{ $t('m.Training_Number') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.rank }}</span>
                                        </span>
                                    </div>
                                    <div>
                                        <span>
                                            <span>{{ $t('m.Training_Auth') }}</span>
                                        </span>
                                        <span v-if="training.auth">
                                            <el-tag :type="TRAINING_TYPE[training.auth]['color']" effect="dark">
                                                {{ $t('m.Training_' + training.auth) }}
                                            </el-tag>
                                        </span>
                                    </div>
                                    <div>
                                        <span>
                                            <span>{{ $t('m.Training_Category') }}</span>
                                        </span>
                                        <span>
                                            <span><el-tag :style="'color: #fff;background-color: ' +
                                                training.categoryName +
                                                ';background-color: ' +
                                                training.categoryColor
                                                " class="category-item" size="medium">{{ training.categoryName
                                                    }}</el-tag></span>
                                        </span>
                                    </div>

                                    <div>
                                        <span>
                                            <span>{{ $t('m.Training_Total_Problems') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.problemCount }}</span>
                                        </span>
                                    </div>
                                    <div>
                                        <span>
                                            <span>{{ $t('m.Author') }}</span>
                                        </span>
                                        <span>
                                            <span><el-link type="info" @click="goUserHome(training.author)">{{
                                                training.author
                                                    }}</el-link></span>
                                        </span>
                                    </div>
                                    <div v-if="training.auth == 'Team'">
                                        <span>
                                            <span>{{ $t('m.Team_Name') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.teamName }}</span>
                                        </span>
                                    </div>
                                    <div>
                                        <span>
                                            <span>{{ $t('m.Recent_Update') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.gmtModified | localtime }}</span>
                                        </span>
                                    </div>

                                    <div>
                                        <span>
                                            <span>{{ $t('m.Limit_Time') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.limitTime ? $t('m.Yes') : $t('m.No') }}</span>
                                        </span>
                                    </div>
                                    <div v-if="training.limitTime">
                                        <span>
                                            <span>{{ $t('m.Begin_Time') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.startTime | localtime }}</span>
                                        </span>
                                    </div>
                                    <div v-if="training.limitTime">
                                        <span>
                                            <span>{{ $t('m.End_Time') }}</span>
                                        </span>
                                        <span>
                                            <span>{{ training.endTime | localtime }}</span>
                                        </span>
                                    </div>
                                </div>
                            </el-card>
                        </el-col>
                        <el-col :md="17" :sm="24">
                            <el-card>
                                <div slot="header">
                                    <span class="panel-title">{{
                                        $t('m.Training_Introduction')
                                    }}</span>
                                </div>
                                <div v-highlight class="markdown-body" v-html="descriptionHtml"></div>
                            </el-card>
                        </el-col>
                    </el-row>
                </el-tab-pane>

                <el-tab-pane :disabled="trainingMenuDisabled" lazy name="TrainingProblemList">
                    <span slot="label"><i aria-hidden="true" class="fa fa-list"></i>&nbsp;{{
                        $t('m.Problem_List')
                    }}</span>
                    <transition name="el-collapse-transition">
                        <router-view v-if="route_name === 'TrainingProblemList'"></router-view>
                    </transition>
                </el-tab-pane>

                <el-tab-pane v-if="isTrainingAdmin && (isPrivateTraining || isTeamTraining)"
                    :disabled="trainingMenuDisabled" lazy name="TrainingRank">
                    <span slot="label"><i aria-hidden="true" class="fa fa-bar-chart"></i>&nbsp;{{
                        $t('m.Record_List')
                    }}</span>
                    <transition name="el-collapse-transition">
                        <router-view v-if="route_name === 'TrainingRank'"></router-view>
                    </transition>
                </el-tab-pane>

                <el-tab-pane v-if="isTrainingAdmin && (isPrivateTraining || isTeamTraining)"
                    :disabled="trainingMenuDisabled" lazy name="TrainingRegisters">
                    <span slot="label"><i class="fa fa-users" aria-hidden="true"></i>&nbsp;{{ $t('m.Training_Registers')
                    }}</span>
                    <transition name="el-collapse-transition">
                        <router-view v-if="route_name === 'TrainingRegisters'"></router-view>
                    </transition>
                </el-tab-pane>
            </el-tabs>
        </div>
    </div>
</template>

<script>
import { TRAINING_TYPE } from '@/common/constants';
import { mapActions, mapGetters, mapState } from 'vuex';
import myMessage from '@/common/message';
import api from '@/common/api';

export default {
    data() {
        return {
            route_name: 'TrainingDetails',
            TRAINING_TYPE: {},
            trainingPassword: '',
            btnLoading: false,
            customColors: [
                { color: '#909399', percentage: 20 },
                { color: '#f56c6c', percentage: 40 },
                { color: '#e6a23c', percentage: 60 },
                { color: '#1989fa', percentage: 80 },
                { color: '#67c23a', percentage: 100 },
            ],
            ifStart: false,
            ifEnd: false
        };
    },
    created() {
        this.route_name = this.$route.name;
        if (this.route_name == 'TrainingProblemDetails') {
            this.route_name = 'TrainingProblemList';
        }
        this.TRAINING_TYPE = Object.assign({}, TRAINING_TYPE);
        this.$store.dispatch('getTraining').then((res) => {
            this.changeDomTitle({ title: res.data.data.title });
            const nowTime = new Date();
            const startTime = new Date(this.training.startTime);
            const endTime = new Date(this.training.endTime);
            this.ifStart = nowTime >= startTime;
            this.ifEnd = nowTime >= endTime;
        });


    },
    methods: {
        ...mapActions(['changeDomTitle']),
        tabClick(tab) {
            let name = tab.name;
            if (name !== this.$route.name) {
                this.$router.push({ name: name });
            }
        },
        checkPassword() {
            if (this.trainingPassword === '') {
                myMessage.warning(this.$i18n.t('m.Enter_the_training_password'));
                return;
            }
            this.btnLoading = true;
            api.registerTraining(this.training.id + '', this.trainingPassword).then(
                (res) => {
                    myMessage.success(this.$i18n.t('m.Register_training_successfully'));
                    this.$store.commit('trainingIntoAccess', { intoAccess: true });
                    this.btnLoading = false;
                },
                (res) => {
                    this.btnLoading = false;
                }
            );
        },
        goUserHome(username) {
            this.$router.push({
                name: 'UserHome',
                query: { username: username },
            });
        },
        getAcProblemPercent() {
            if (!this.training.problemCount) {
                return 100;
            }
            if (this.training.acCount == null) {
                this.training.acCount = 0;
            }
            return Number(((this.training.acCount / this.training.problemCount) *100).toFixed(2));
        },
    },
    computed: {
        ...mapState({
            training: (state) => state.training.training,
        }),
        ...mapGetters([
            'trainingPasswordFormVisible',
            'trainingMenuDisabled',
            'isPrivateTraining',
            'isTeamTraining',
            'isAuthenticated',
            'isTrainingAdmin'
        ]),
        descriptionHtml() {
            if (this.training.description) {
                return this.$markDown.render(this.training.description);
            }
        },
    },
    watch: {
        $route(newVal) {
            this.route_name = newVal.name;
            if (newVal.name == 'TrainingProblemDetails') {
                this.route_name = 'TrainingProblemList';
            }
            this.changeDomTitle({ title: this.training.title });
        },
    },
    beforeDestroy() {
        this.$store.commit('clearTraining');
    },
};
</script>

<style scoped>
.card-top {
    margin-top: 15px;
}

.training-header {
    text-align: center;
}

.count {
    margin-top: 10px;
    font-size: 18px;
    font-weight: 700;
}

.password-form-card {
    text-align: center;
    margin-bottom: 15px;
}

.info-rows>* {
    margin-bottom: var(--info-row-margin-bottom, 1em);
    display: flex;
    align-items: center;
    font-size: 16px;
    line-height: 1.5;
    color: rgba(0, 0, 0, 0.75);
}

.info-rows>*>*:first-child {
    flex: 1 0 auto;
    text-align: left;
}

.info-rows> :last-child {
    margin-bottom: 0;
}

::v-deep .el-card__header {
    border-bottom: 0px;
    padding-bottom: 0px;
}

::v-deep .el-tabs__nav-wrap {
    background: #fff;
    border-radius: 3px;
}

::v-deep .el-tabs--top .el-tabs__item.is-top:nth-child(2) {
    padding-left: 20px;
}

.status-card {
    padding: 12px;
    /* 增加卡片内边距 */
    justify-content: center;
    display: flex;
    align-items: center;
}

.status-tag {
    display: inline-flex;
    align-items: center;
    padding: 6px 12px;
    border-radius: 20px;
    /* 圆角 */
    font-weight: 500;
    /* 加粗文字 */
    transition: all 0.3s ease;
}

/* 已结束状态 */
.status-ended {
    background-color: #f8d7da;
    /* 浅红色背景 */
    color: #721c24;
    /* 深红色文字 */
    border: 1px solid #f5c6cb;
    /* 浅红边框 */

}

/* 未开始状态 */
.status-not-started {
    background-color: #fff3cd;
    /* 浅黄色背景 */
    color: #856404;
    /* 深黄色文字 */
    border: 1px solid #ffeeba;
    /* 浅黄边框 */

}

/* 悬停效果 */
.status-tag:hover {
    transform: translateY(-2px);
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
}

/* 文字排版 */
.status-tag span {
    margin-right: 8px;
    /* 图标与文字间距（如果有图标时使用） */
    font-size: 16px;
}
</style>
