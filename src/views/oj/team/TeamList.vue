<template>
    <div class="container">
        <el-row :gutter="20">
            <el-col v-loading="loading">
                <!-- <div class="discussion-header">
                    <span style="padding: 16px;float:left;">
                        <el-breadcrumb separator-class="el-icon-arrow-right">
                        </el-breadcrumb>
                    </span>
                </div> -->
                <template v-if="teamList.length > 0">
                    <div v-for="(team, index) in teamList" :key="index" class="title-article">
                        <el-card class="list-card" shadow="hover">
                            <h1 class="article-hlink">
                                <a>{{
                                    team.name
                                    }}</a>
                            </h1>
                            <a class="article-hlink2">
                                <p>{{ team.description }}</p>
                            </a>
                            <div class=" title-msg">
                                <span>
                                    <a :title="team.ownerName" @click="getInfoByUsername(team.ownerId, team.ownerName)">
                                        <avatar :inline="true" :size="24" :src="team.ownerAvatar"
                                            :username="team.ownerName" class="user-avatar" color="#FFF"></avatar>
                                        <span class="pl">{{ $t('m.Owner_Name') }}: {{ team.ownerName }}</span>
                                        <span v-if="team.ownerId == userInfo.uid" class="role-admin role"
                                            title="Self">Self</span>
                                    </a>
                                </span>
                                <span>
                                    <span class="pl">{{ $t('m.Team_Member_Count') }}: {{ team.count }}</span>
                                </span>
                                <el-dropdown v-show="isAuthenticated && team.ownerId != userInfo.uid"
                                    class="hidden-xs-only" style="float:right;" @command="handleCommand">
                                    <span class="el-dropdown-link">
                                        <i class="el-icon-more"></i>
                                    </span>
                                    <el-dropdown-menu slot="dropdown" @command="handleCommand">
                                        <el-dropdown-item :command="'quit:' + team.uuid" icon="el-icon-delete">{{
                                            $t('m.Quit_Team') }}
                                        </el-dropdown-item>
                                    </el-dropdown-menu>
                                </el-dropdown>
                            </div>
                        </el-card>
                    </div>
                </template>
                <template v-else>
                    <el-empty :description="$t('m.No_Data')"></el-empty>
                </template>
                <div class="panel-options">
                    <el-pagination :page-size="pageSize" :total="total" class="page" layout="prev, pager, next"
                        @current-change="currentChange">
                    </el-pagination>
                </div>
            </el-col>
        </el-row>
    </div>
</template>
<script>
import Avatar from 'vue-avatar';
import api from '@/common/api';
import myMessage from '@/common/message';
import { mapActions, mapGetters } from 'vuex';
import 'element-ui/lib/theme-chalk/display.css';
import Pagination from '@/components/oj/common/Pagination';

const Editor = () => import('@/components/admin/Editor.vue');
export default {
    components: {
        Avatar,
        Editor,
        Pagination,
    },
    data() {
        return {
            total: 0,
            teamList: [],
            routeName: '',
            pageSize: 10,
            currentPage: 1,
            loading: false
        };
    },
    mounted() {

        this.getMyTeamList(1);
    },
    methods: {
        ...mapActions(['changeDomTitle']),
        currentChange(page) {
            this.currentChange = page;
            this.getMyTeamList(page);
        },
        getMyTeamList(page) {
            this.loading = true;
            api.getMyTeamList(page, this.pageSize, this.userInfo.uid).then((res) => {
                this.loading = false;
                this.total = res.data.data.total;
                this.teamList = res.data.data.records;
            }, (res) => {
            })
        },
        getInfoByUsername(uid, username) {
            this.$router.push({
                path: '/user-home',
                query: { uid, username },
            });
        },
        handleCommand(command) {
            let tmpArr = command.split(':');
            switch (tmpArr[0]) {
                case 'quit':
                    const id = tmpArr[1];
                    this.quitTeam(id);
            }
        },
        quitTeam(teamId) {
            this.loading = true;
            api.quitTeam(teamId, this.userInfo.uid).then((res) => {
                this.loading = false;
                myMessage.success('退出成功');
                this.currentChange(1);
            }, (res) => {
                this.loading = false;
                myMessage.error('退出失败');

            })
        }
    },
    watch: {
        $route(newVal, oldVal) {
            if (newVal != oldVal) {
                this.init();
            }
        },
    },

    computed: {
        ...mapGetters(['isAuthenticated', 'userInfo', 'isAdminRole']),
    },
};
</script>
<style>
.role-root {
    background-color: #f9d681 !important;
    color: #ff503f !important;
    font-weight: 600;
}

.role-admin {
    background-color: #409eff !important;
    color: #fff !important;
}

.role {
    display: inline-block;
    font-size: 0.75rem;
    padding: 0.1875rem 0.25rem;
    line-height: 1;
    vertical-align: middle;
}
</style>
<style scoped>
::v-deep .el-card__body {
    padding: 0 !important;
}

.discussion-header {
    background-color: #fff;
    border-radius: 6px;
    overflow: hidden;
    margin-bottom: 10px;
    height: 41px;
}

.discussion-header .search {
    margin-top: 3px;
    margin-right: 6px;
    float: right;
}

.list-card {
    border-radius: 6px;
    margin-bottom: 10px;
    padding: 15px;
    text-align: left;
    position: relative;
}

.list-card p {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

.list-card .article-hlink {
    overflow: hidden;
    display: block;
}

.svg-top {
    position: absolute;
    top: 0px;
    right: 0px;
}

.article-hlink {
    margin: 0;
    padding: 0;
}

.article-hlink a {
    font-size: 16px;
    font-weight: 600;
    color: #34495e;
    margin-top: 5px;
}

a {
    color: #34495e;
    text-decoration: none;
}

.article-hlink2 p {
    margin-bottom: 10px;
    color: #888;
    font-size: 12px;
    margin: 0;
    padding: 0;
}

.title-article .title-msg {
    margin-top: 15px;
    font-size: 12px;
    color: #999 !important;
}

.title-article .title-msg a {
    color: #999;
    text-decoration: none;
}

.user-avatar {
    vertical-align: middle;
}

.title-article .title-msg span {
    margin-right: 3px;
}

.title-article .title-msg .pl {
    padding-left: 0.3rem !important;
}

.title-article .title-msg .pr {
    padding-right: 0.3rem !important;
}

.category-body {
    background: #fff;
    padding: 15px;
    margin-bottom: 15px;
    border-radius: 6px;
    overflow: hidden;
    margin-top: 12px;
}

.category-body .title-sidebar {
    border-bottom: 1px solid #eee;
    width: 100%;
    color: #34495e;
    font-size: 14px;
    font-weight: 600;
    padding-bottom: 10px;
    margin-bottom: 10px;
    overflow: hidden;
}

.category-body .title-sideba a {
    color: #34495e;
}

.category-body h3 {
    margin: 0;
    padding: 0;
}

.category-item {
    height: 30px;
    font-size: 14px;
    padding: 3px 10px;
    margin-bottom: 5px;
}

.category-item a {
    color: #34495e;
}

.category-item:hover {
    background-color: #eff3f5 !important;
    font-weight: bold;
    color: #222;
}
</style>
