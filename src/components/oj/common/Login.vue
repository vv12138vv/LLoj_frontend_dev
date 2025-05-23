<template>
    <div>
        <el-form ref="formLogin" :model="formLogin" :rules="rules" label-width="100px">
            <el-form-item prop="username">
                <el-input v-model="formLogin.username" :placeholder="$t('m.Login_Username')"
                    prefix-icon="el-icon-user-solid" width="100%" @keyup.enter.native="enterHandleLogin"></el-input>
            </el-form-item>
            <el-form-item prop="password">
                <el-input v-model="formLogin.password" :placeholder="$t('m.Login_Password')" prefix-icon="el-icon-lock"
                    type="password" @keyup.enter.native="enterHandleLogin"></el-input>
            </el-form-item>
        </el-form>
        <div class="footer">
            <el-button v-if="!needVerify" :loading="btnLoginLoading" type="primary" @click="handleLogin">{{
                $t('m.Login_Btn') }}
            </el-button>
            <el-popover v-else v-model="loginSlideBlockVisible" placement="bottom" trigger="click" width="350">
                <el-button slot="reference" :loading="btnLoginLoading" type="primary">{{
                    $t('m.Login_Btn')
                    }}
                </el-button>
                <slide-verify v-if="!verify.loginSuccess" ref="slideBlock" :accuracy="3" :h="100" :l="42" :r="10"
                    :slider-text="$t('m.Slide_Verify')" :w="325" @success="handleLogin" :imgs="verifyImgs"
                    @refresh="refreshCaptcha">
                </slide-verify>
                <el-alert v-show="verify.loginSuccess" :center="true" :closable="false" :description="verify.loginMsg"
                    :title="$t('m.Slide_Verify_Success')" show-icon type="success">
                </el-alert>
            </el-popover>
            <el-link v-if="websiteConfig.register" type="primary" @click="switchMode('Register')">{{
                $t('m.Login_No_Account') }}
            </el-link>
            <el-link style="float: right" type="primary" @click="switchMode('ResetPwd')">{{
                $t('m.Login_Forget_Password') }}
            </el-link>
        </div>
    </div>
</template>
<script>
import { mapActions, mapGetters, mapState } from 'vuex';
import api from '@/common/api';
import mMessage from '@/common/message';

export default {
    data() {
        return {
            allow_register: true, //是否允许注册
            btnLoginLoading: false,
            verify: {
                loginSuccess: false,
                loginMsg: '',
            },
            needVerify: false,
            formLogin: {
                username: '',
                password: '',
            },
            verifyImgs: ["https://picsum.photos/300/150?image=" + this.getRandomNumberByRange(0, 1081)],
            loginSlideBlockVisible: false,
            rules: {
                username: [
                    {
                        required: true,
                        message: this.$i18n.t('m.Username_Check_Required'),
                        trigger: 'blur',
                    },
                    {
                        max: 20,
                        message: this.$i18n.t('m.Username_Check_Max'),
                        trigger: 'blur',
                    },
                ],
                password: [
                    {
                        required: true,
                        message: this.$i18n.t('m.Password_Check_Required'),
                        trigger: 'blur',
                    },
                    {
                        min: 6,
                        max: 20,
                        message: this.$i18n.t('m.Password_Check_Between'),
                        trigger: 'blur',
                    },
                ],
            },
        };
    },
    methods: {
        ...mapActions(['changeModalStatus']),
        switchMode(mode) {
            this.changeModalStatus({
                mode,
                visible: true,
            });
        },
        getRandomNumberByRange(start, end) {
            return Math.round(Math.random() * (end - start) + start);
        },
        refreshCaptcha() {
            this.verifyImgs = ["https://picsum.photos/300/150?image=" + this.getRandomNumberByRange(0, 1081)];
        },
        enterHandleLogin() {
            if (this.needVerify) {
                this.visible.loginSlideBlock = true;
            } else {
                this.handleLogin();
            }
        },
        handleLogin(times) {
            if (this.needVerify) {
                this.verify.loginSuccess = true;
                let time = (times / 1000).toFixed(1);
                this.verify.loginMsg = 'Total time ' + time + 's';
                setTimeout(() => {
                    this.loginSlideBlockVisible = false;
                    this.verify.loginSuccess = false;
                }, 1000);
            }
            this.$refs['formLogin'].validate((valid) => {
                if (valid) {
                    this.btnLoginLoading = true;
                    let formData = Object.assign({}, this.formLogin);
                    api.login(formData).then(
                        (res) => {
                            this.btnLoginLoading = false;
                            this.changeModalStatus({ visible: false });

                            const jwt = res.headers['authorization'];
                            this.$store.commit('changeUserToken', jwt);
                            this.$store.dispatch('setUserInfo', res.data.data);
                            this.$store.dispatch('incrLoginFailNum', true);
                            mMessage.success(this.$i18n.t('m.Welcome_Back'));
                        },
                        (_) => {
                            this.$store.dispatch('incrLoginFailNum', false);
                            this.btnLoginLoading = false;
                        }
                    );
                }
            });
        },
    },
    computed: {
        ...mapState(['websiteConfig']),
        ...mapGetters(['modalStatus', 'loginFailNum']),
        visible: {
            get() {
                return this.modalStatus.visible;
            },
            set(value) {
                this.changeModalStatus({ visible: value });
            },
        },
    },
    watch: {
        loginFailNum(newVal, oldVal) {
            if (newVal >= 5) {
                this.needVerify = true;
            } else {
                this.needVerify = false;
            }
        },
    },
};
</script>
<style scoped>
.footer {
    overflow: auto;
    margin-top: 20px;
    margin-bottom: -15px;
    text-align: left;
}

::v-deep .el-button {
    margin: 0 0 15px 0;
    width: 100%;
}

::v-deep .el-form-item__content {
    margin-left: 0px !important;
}
</style>
