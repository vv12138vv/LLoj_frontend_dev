<template>
    <div class="setting-main">
        <el-row :gutter="20">
            <el-col :lg="10" :md="10" :sm="24">
                <div class="left">
                    <p class="section-title">{{ $t('m.Change_Password') }}</p>
                    <el-form ref="formPassword" :model="formPassword" :rules="rulePassword" class="setting-content">
                        <el-form-item :label="$t('m.Old_Password')" prop="oldPassword">
                            <el-input v-model="formPassword.oldPassword" type="password" />
                        </el-form-item>
                        <el-form-item :label="$t('m.New_Password')" prop="newPassword">
                            <el-input v-model="formPassword.newPassword" type="password" />
                        </el-form-item>
                        <el-form-item :label="$t('m.Confirm_New_Password')" prop="againPassword">
                            <el-input v-model="formPassword.againPassword" type="password" />
                        </el-form-item>
                    </el-form>
                    <el-popover v-model="visible.passwordSlideBlock" placement="top" trigger="click" width="350">
                        <el-button slot="reference" :disabled="disabled.btnPassword" :loading="loading.btnPassword"
                            type="primary">{{ $t('m.Update_Password') }}
                        </el-button>
                        <slide-verify v-show="!verify.passwordSuccess" ref="passwordSlideBlock" :accuracy="3" :h="100"
                            :l="42" :r="10" :slider-text="$t('m.Slide_Verify')" :w="325" :imgs="verifyImgs"
                            @again="onAgain('password')" @success="changePassword" @refresh="refreshCaptcha">
                        </slide-verify>
                        <el-alert v-show="verify.passwordSuccess" :center="true" :closable="false"
                            :description="verify.passwordMsg" :title="$t('m.Slide_Verify_Success')" show-icon
                            type="success">
                        </el-alert>
                    </el-popover>
                </div>
                <el-alert v-show="visible.passwordAlert.show" :closable="false"
                    :description="visible.passwordAlert.description" :title="visible.passwordAlert.title"
                    :type="visible.passwordAlert.type" effect="dark" show-icon style="margin-top:15px">
                </el-alert>
            </el-col>
            <el-col :lg="4" :md="4">
                <div class="separator hidden-md-and-down"></div>
                <p></p>
            </el-col>
            <el-col :lg="10" :md="10" :sm="24">
                <div class="right">
                    <p class="section-title">{{ $t('m.Change_Email') }}</p>
                    <el-form ref="formEmail" :model="formEmail" :rules="ruleEmail" class="setting-content">
                        <el-form-item :label="$t('m.Current_Password')" prop="password">
                            <el-input v-model="formEmail.password" type="password" />
                        </el-form-item>
                        <el-form-item :label="$t('m.Old_Email')">
                            <el-input v-model="formEmail.oldEmail" disabled />
                        </el-form-item>
                        <el-form-item :label="$t('m.New_Email')" prop="newEmail">
                            <el-input v-model="formEmail.newEmail" />
                        </el-form-item>
                    </el-form>
                    <el-popover v-model="visible.emailSlideBlock" placement="top" trigger="click" width="350">
                        <el-button slot="reference" :disabled="disabled.btnEmail" :loading="loading.btnEmailLoading"
                            type="primary">{{ $t('m.Update_Email') }}
                        </el-button>
                        <slide-verify v-show="!verify.emailSuccess" ref="emailSlideBlock" :accuracy="3" :h="100" :l="42"
                            :r="10" :slider-text="$t('m.Slide_Verify')" :w="325" :imgs="verifyImgs"
                            @again="onAgain('email')" @success="changeEmail" @refresh="refreshCaptcha">
                        </slide-verify>
                        <el-alert v-show="verify.emailSuccess" :center="true" :closable="false"
                            :description="verify.emailMsg" :title="$t('m.Slide_Verify_Success')" show-icon
                            type="success">
                        </el-alert>
                    </el-popover>
                </div>
                <el-alert v-show="visible.emailAlert.show" :closable="false"
                    :description="visible.emailAlert.description" :title="visible.emailAlert.title"
                    :type="visible.emailAlert.type" effect="dark" show-icon style="margin-top:15px">
                </el-alert>
            </el-col>
        </el-row>
    </div>
</template>

<script>
import api from '@/common/api';
import myMessage from '@/common/message';
import 'element-ui/lib/theme-chalk/display.css';
export default {
    data() {
        const oldPasswordCheck = [
            {
                required: true,
                trigger: 'blur',
            },
            {
                trigger: 'blur',
                min: 6,
                max: 20,
                message: this.$i18n.t('m.Password_Check_Between'),
            },
        ];
        const CheckAgainPassword = (rule, value, callback) => {
            if (value !== this.formPassword.newPassword) {
                callback(new Error(this.$i18n.t('m.Password_does_not_match')));
            }
            callback();
        };
        const CheckNewPassword = (rule, value, callback) => {
            if (this.formPassword.oldPassword !== '') {
                if (this.formPassword.oldPassword === this.formPassword.newPassword) {
                    callback(
                        new Error(this.$i18n.t('m.The_new_password_does_not_change'))
                    );
                } else {
                    // 对第二个密码框再次验证
                    this.$refs.formPassword.validateField('again_password');
                }
            }
            callback();
        };
        const CheckEmail = (rule, value, callback) => {
            if (this.formEmail.oldEmail !== '') {
                if (this.formEmail.oldEmail === this.formEmail.newEmail) {
                    callback(new Error(this.$i18n.t('m.The_new_email_does_not_change')));
                }
            }
            callback();
        };
        return {
            loading: {
                btnPassword: false,
                btnEmail: false,
            },
            disabled: {
                btnPassword: false,
                btnEmail: false,
            },
            verify: {
                passwordSuccess: false,
                passwordMsg: '',
                emailSuccess: false,
                emailMsg: '',
            },
            visible: {
                passwordAlert: {
                    type: 'success',
                    show: false,
                    title: '',
                    description: '',
                },
                emailAlert: {
                    type: 'success',
                    show: false,
                    title: '',
                    description: '',
                },
                passwordSlideBlock: false,
                emailSlideBlock: false,
            },
            formPassword: {
                oldPassword: '',
                newPassword: '',
                againPassword: '',
            },
            formEmail: {
                password: '',
                oldEmail: '',
                newEmail: '',
            },
            rulePassword: {
                oldPassword: oldPasswordCheck,
                newPassword: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                    {
                        trigger: 'blur',
                        min: 6,
                        max: 20,
                        message: this.$i18n.t('m.Password_Check_Between'),
                    },
                    { validator: CheckNewPassword, trigger: 'blur' },
                ],
                againPassword: [
                    {
                        required: true,
                        trigger: 'blur',
                        message: this.$i18n.t('m.Password_Again_Check_Required'),
                    },
                    { validator: CheckAgainPassword, trigger: 'blur' },
                ],
            },
            ruleEmail: {
                password: oldPasswordCheck,
                newEmail: [
                    {
                        required: true,
                        message: this.$i18n.t('m.Email_Check_Required'),
                        trigger: 'blur',
                    },
                    {
                        type: 'email',
                        trigger: 'change',
                        message: this.$i18n.t('m.Email_Check_Format'),
                    },
                    { validator: CheckEmail, trigger: 'blur' },
                ],
            },
            verifyImgs: ["https://picsum.photos/300/150?image=" + this.getRandomNumberByRange(0, 1081)]
        };
    },
    mounted() {
        this.formEmail.oldEmail = this.$store.getters.userInfo.email || '';
    },
    methods: {
        getRandomNumberByRange(start, end) {
            return Math.round(Math.random() * (end - start) + start);
        },
        refreshCaptcha(){
            this.verifyImgs=["https://picsum.photos/300/150?image=" + this.getRandomNumberByRange(0, 1081)];
        },
        changePassword(times) {
            this.verify.passwordSuccess = true;
            let time = (times / 1000).toFixed(1);
            this.verify.passwordMsg = 'Total time ' + time + 's';
            setTimeout(() => {
                this.visible.passwordSlideBlock = false;
                this.verify.passwordSuccess = false;
                // 无论后续成不成功，验证码滑动都要刷新
                this.$refs.passwordSlideBlock.reset();
            }, 1000);

            this.$refs['formPassword'].validate((valid) => {
                if (valid) {
                    this.loading.btnPassword = true;
                    let data = Object.assign({}, this.formPassword);
                    delete data.againPassword;
                    api.changePassword(data).then(
                        (res) => {
                            this.loading.btnPassword = false;
                            if (res.data.data.code == 200) {
                                myMessage.success(this.$i18n.t('m.Update_Successfully'));
                                this.visible.passwordAlert = {
                                    show: true,
                                    title: this.$i18n.t('m.Update_Successfully'),
                                    type: 'success',
                                    description: res.data.data.msg,
                                };
                                setTimeout(() => {
                                    this.visible.passwordAlert = false;
                                    this.$router.push({ name: 'Logout' });
                                }, 5000);
                            } else {
                                myMessage.error(res.data.msg);
                                this.visible.passwordAlert = {
                                    show: true,
                                    title: this.$i18n.t('m.Update_Failed'),
                                    type: 'warning',
                                    description: res.data.data.msg,
                                };
                                if (res.data.data.code == 403) {
                                    this.visible.passwordAlert.type = 'error';
                                    this.disabled.btnPassword = true;
                                }
                            }
                        },
                        (err) => {
                            this.loading.btnPassword = false;
                        }
                    );
                }
            });
        },
        changeEmail(times) {
            this.verify.emailSuccess = true;
            let time = (times / 1000).toFixed(1);
            this.verify.emailMsg = 'Total time ' + time + 's';
            setTimeout(() => {
                this.visible.emailSlideBlock = false;
                this.verify.emailSuccess = false;
                // 无论后续成不成功，验证码滑动都要刷新
                this.$refs.emailSlideBlock.reset();
            }, 1000);
            this.$refs['formEmail'].validate((valid) => {
                if (valid) {
                    this.loading.btnEmail = true;
                    let data = Object.assign({}, this.formEmail);
                    api.changeEmail(data).then(
                        (res) => {
                            this.loading.btnEmail = false;
                            if (res.data.data.code == 200) {
                                myMessage.success(this.$i18n.t('m.Update_Successfully'));
                                this.visible.emailAlert = {
                                    show: true,
                                    title: this.$i18n.t('m.Update_Successfully'),
                                    type: 'success',
                                    description: res.data.data.msg,
                                };
                                // 更新本地缓存
                                this.$store.dispatch('setUserInfo', res.data.data.userInfo);
                                this.$refs['formEmail'].resetFields();
                                this.formEmail.oldEmail = res.data.data.userInfo.email;
                            } else {
                                myMessage.error(res.data.msg);
                                this.visible.emailAlert = {
                                    show: true,
                                    title: this.$i18n.t('m.Update_Failed'),
                                    type: 'warning',
                                    description: res.data.data.msg,
                                };
                                if (res.data.data.code == 403) {
                                    this.visible.emailAlert.type = 'error';
                                    this.disabled.btnEmail = true;
                                }
                            }
                        },
                        (err) => {
                            this.loading.btnEmail = false;
                        }
                    );
                }
            });
        },
        onAgain(type) {
            if ((type = 'password')) {
                this.$refs.passwordSlideBlock.reset();
            } else {
                this.$refs.emailSlideBlock.reset();
            }
            myMessage.warning(this.$i18n.t('m.Guess_robot'));
        },
    },
};
</script>

<style scoped>
.section-title {
    font-size: 21px;
    font-weight: 500;
    padding-top: 10px;
    padding-bottom: 20px;
    line-height: 30px;
}

.left {
    text-align: center;
}

.right {
    text-align: center;
}

::v-deep .el-input__inner {
    height: 32px;
}

::v-deep .el-form-item__label {
    font-size: 12px;
    line-height: 20px;
}

.separator {
    display: block;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 50%;
    border: 1px dashed #eee;
}
</style>
