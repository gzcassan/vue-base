<template>
    <transition name="bounce" enter-active-class="fadeInDown" leave-active-class="bounceOutUp">
        <div class="home-container" v-show="$store.state.toLogin">
            <div class="home-login-container">
                <Card>
                    <p slot="title" @click="current = 2">
                        <Icon type="log-in"></Icon>
                        {{ current === 1 ? '欢迎登录' : '欢迎注册' }}
                    </p>
                    <transition name="bounce" enter-active-class="fadeInLeft" leave-active-class="fadeOutLeft" mode="out-in">
                        <Form class="login-form" ref="loginForm" :model="loginForm" :rules="loginRules"  v-if="current === 1">
                            <FormItem prop="account" label="用户名">
                                <Input type="text" v-model="infoForm.account" placeholder="请输入，必填"></Input>
                            </FormItem>
                            <FormItem prop="password" label="密码">
                                <Input type="password" v-model="infoForm.password" placeholder="请输入，必填"></Input>
                                <!--Input type="password" v-model="loginForm.password" placeholder="密码">
                                    <span slot="prepend">
                                        <Icon :size="14" type="locked"></Icon>
                                    </span>
                                </Input-->
                            </FormItem>
                            <Form-item>
                                <Button type="primary"  :loading="loginLoading" @click="login('loginForm')" long>确认</Button>
                            </Form-item>
                        </Form>
                    </transition>
                    <transition name="bounce" enter-active-class="fadeInRight" leave-active-class="fadeOutRight" mode="out-in">
                        <Form  class="login-form" ref="infoForm" :model="infoForm" :rules="infoRules" v-if="current === 2">
                            <Form-item prop="userName" label="用户名">
                                <Input type="text" v-model="infoForm.userName" placeholder="请输入，必填"></Input>
                            </Form-item>
                            <Form-item prop="allId" label="部门信息">
                                    <Cascader v-model="infoForm.allId" :data="deptList" change-on-select placeholder="请选择，必选"></Cascader>
                            </Form-item>

                            <Form-item class="text-center">
                                <ButtonGroup shape="circle">
                                    <Button :loading="loginLoading" @click="current = 1">返回</Button>
                                    <Button type="primary" :loading="loginLoading" @click="perfectInfo('loginForm')">开启</Button>
                                </ButtonGroup>
                            </Form-item>
                        </Form>
                    </transition>
                </Card>
            </div>
        </div>
        </div>
    </transition>
</template>

<script>
import _isEmpty from 'lodash/isEmpty'
let md = require('Utils/md5')
export default {
    name: 'Login',
    data() {
        return {
            current: 2,
            deptList: [],
            loginForm: {
                account: '',
                password: '',
            },
            loginLoading: false,
            loginRules: {
                account: [{
                    required: true,
                    message: '请填写手机号',
                    trigger: 'blur'
                }],
                password: [{
                    required: true,
                    message: '请填写密码',
                    trigger: 'blur'
                }, {
                    type: 'string',
                    min: 6,
                    message: '密码长度不能小于6位',
                    trigger: 'blur'
                }]
            },
            infoForm: {
                userName: '',
                allId: [],
            },
            infoRules: {
                userName: [{
                    required: true,
                    message: '请填写您的姓名',
                }],
                allId: [{
                    required: true,
                    message: '请选择所属部门和组',
                }]
            }
        }
    },
    methods: {
        login() {
            this.$refs.loginForm.validate((valid) => {
                // if (valid) {
                    this.loginAndVerify()
                // } else {
                    // this.$Message.error('表单验证失败!');
                // }
            })
        },
        async loginAndVerify() {
            // try {
                // this.loginForm.password = md.hex_md5(this.loginForm.password)
                // this.loginLoading = true
                // this.uidInfo = await this.$apis['other/login'](this.loginForm)
                // this.userInfo = await this.$apis['user/info']()
                // if (this.userInfo.status === 'FRESH') {
                //     // 新注册用户
                //     this.current = 2
                //     let _deptList = await this.$apis['other/allGroup']()
                //     this.deptList = this.handleDeptList(_deptList)
                // } else {
                    // 已经完善过信息的用户
                    this.gotoIndex()
                // }
            // } catch (e) {}
            // this.loginLoading = false
        },
        handleDeptList(deptList) {
            return deptList.map(dept => {
                let children = dept.groupList.map(group => {
                    return {
                        label: group.name,
                        value: group.groupId,
                    }
                })
                return {
                    label: dept.name,
                    value: dept.groupId,
                    children
                }
            })
        },
        perfectInfo() {
             this.$refs.infoForm.validate((valid) => {
                if (valid) {
                    let params = {
                        uid: this.uidInfo.uid,
                        userName: this.infoForm.userName,
                        deptId: this.infoForm.allId[0],
                        projectId: this.infoForm.allId[1]
                    }
                    this.$apis['user/save'](params).then(() => {
                        this.$store.commit('SET_USER_INFO', this.userInfo)
                        this.$Message.info('登录成功!');
                        this.gotoIndex()
                    })
                } else {
                    this.$Message.error('表单验证失败!');
                }
            })
        },
        gotoIndex() {
            this.$router.push(this.$store.state.toLoginPath)
            this.$store.commit('SET_TO_LOGIN', false)
        }
    }
}
</script>
