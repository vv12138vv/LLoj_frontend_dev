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
        };
    },
    mounted() {
        this.init();
    },
    watch: {
        $route() {
            if (this.$route.name === 'admin-edit-team') {
                this.title = this.$i18n.t('m.Edit_Team');
                this.getTeam();
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
        ...mapGetters(['userInfo']),
    },
    methods: {
        init() {
            if (this.$route.name === 'admin-edit-team') {
                this.title = this.$i18n.t('m.Edit_Team');
                this.getTeam();
            } else {
                this.title = this.$i18n.t('m.Create_Team');
            }
        },
        getTeam() {
            api.admin_getTeam(this.$route.params.teamId)
                .then((res) => {
                    const data = res.data.data;
                    this.team = data.team || {};
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
        }
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
