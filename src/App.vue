<template>
    <div id="app">
        <header class="header">
            <router-link class="header-left" to="/">
                <svg class="header-logo">
                    <use xlink:href="#icon-logo"></use>
                </svg>
            </router-link>
            <div class="header-right">
                <svg @click="loginDialogShow = true" v-if="JSON.stringify($store.state.user)==='{}'" width="100%"
                     height="100%"
                     fill="currentColor">
                    <use xlink:href="#icon-login"></use>
                </svg>
                <svg v-else-if="$route.name === 'user'" @click="logoutDialogShow = true" width="100%" height="100%"
                     fill="rgba(0,0,0,.54)">
                    <use xlink:href="#icon-logout"></use>
                </svg>
                <router-link class="header-avatar" v-else to="/user">
                    <img :src="$store.state.user.avatar_url" alt="" width="100%" height="100%">
                </router-link>
            </div>
        </header>
        <keep-alive>
            <router-view/>
        </keep-alive>
        <my-dialog v-model="loginDialogShow" :before-close="beforeLoginDialogClose"
                   :confirm-text="loginDialogConfirmText">
            <svg class="dialog-header-logo" slot="header">
                <use xlink:href="#icon-login"></use>
            </svg>
            <span slot="header">绑定 GitHub 信息</span>
            <my-input class="login-input" v-model="usernameValue" :label="usernameLabel"
                      :placeholder="usernamePlaceholder" ref="usernameInputRef"></my-input>
            <my-input class="login-input" type="password" v-model="tokenValue" :label="passwordLabel"
                      :placeholder="passwordPlaceholder" ref="passwordInputRef"></my-input>
        </my-dialog>
        <my-dialog v-model="logoutDialogShow" @on-confirm="clickLogoutDialogConfirm">
            <svg class="dialog-header-logo" slot="header">
                <use xlink:href="#icon-logout"></use>
            </svg>
            <span slot="header">注销账号</span>
            <div class="logout-main">确定要注销吗？</div>
        </my-dialog>
    </div>
</template>

<script>
  import Dialog from './components/Dialog'
  import Input from './components/Input'

  export default {
    components: {
      'my-dialog': Dialog,
      'my-input': Input
    },
    data() {
      return {
        loginDialogShow: false,
        logoutDialogShow: false,
        loginDialogConfirmText: '绑定',
        usernameLabel: '用户名',
        usernamePlaceholder: '请输入GitHub用户名',
        passwordLabel: 'Token',
        passwordPlaceholder: '请输入有效Token',
        usernameValue: '',
        tokenValue: ''
      }
    },
    methods: {
      clickLogoutDialogConfirm() {
        this.$store.commit('SET_USER', {})
        this.$store.commit('SET_TOKEN', '')
        this.$tips('注销成功', 'correct', 2000);
        this.$router.push('/read')
      },
      beforeLoginDialogClose(action, done) {
        if (action === 'on-confirm') {
          if (!(this.usernameValue && this.tokenValue)) {
            done(false);
          } else {
            this.$tips('正在绑定', 'loading');
            this.$ajax(`https://api.github.com/user?access_token=${this.tokenValue}`).then((res) => {
              if (this.usernameValue === res.data.login) {
                this.$store.commit('SET_USER', res.data);
                this.$store.commit('SET_TOKEN', this.tokenValue);
                this.$tips('绑定成功', 'correct', 2000);
                done()
                this.$refs.usernameInputRef.doExit()
                this.$refs.passwordInputRef.doExit()
                this.$router.push('/');
              } else {
                this.$tips('用户名与Token不一致', 'wrong', 2000);
                done(false)
              }
            }).catch(() => {
              this.$tips('绑定失败', 'wrong', 2000);
              done(false)
            })
          }
        } else {
          this.$refs.usernameInputRef.doExit()
          this.$refs.passwordInputRef.doExit()
          done()
        }
      }
    }
  }
</script>

<style lang="scss">
    #app {
        width: 100%;
        height: 100%;

        //采用特殊的布局方式，可视区域为绝对定位，页面不会出现滚动条，滚动条在可视区域里
        .header {
            position: relative;

            z-index: 2;
            display: flex;
            justify-content: space-between;
            align-items: center;

            padding: 0 15px;

            height: 46px;

            color: $theme-color;

            @include border(bottom, rgba(0, 0, 0, .2));

            .header-left {
                .header-logo {
                    width: 83px;
                    height: 30px;

                    fill: $theme-color;
                }
            }

            .header-right {
                width: 30px;
                height: 30px;

                font-size: 0;

                .header-avatar {
                    display: block;
                    width: 28px;
                    height: 28px;
                    border-radius: 50%;

                    border: 1px solid $theme-color;

                    overflow: hidden;
                }
            }
        }

        .dialog-header-logo {
            width: 30px;
            height: 30px;

            margin-right: 13px;

            fill: $theme-color;
        }
    }
</style>
