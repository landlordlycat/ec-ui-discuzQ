<template>
  <qui-page :data-qui-theme="theme">
    <view class="register-box">
      <view class="register-box-h">{{ i18n.t('user.register') }}</view>
      <view class="register-box-con">
        <input
          class="input"
          maxlength="15"
          :placeholder="i18n.t('user.username')"
          placeholder-style="color: #ddd"
          v-model="username"
        />
        <input
          class="input"
          type="password"
          maxlength="50"
          :placeholder="i18n.t('user.password')"
          placeholder-style="color: #ddd"
          v-model="password"
        />
        <input
          v-if="validate"
          class="input"
          maxlength="50"
          :placeholder="i18n.t('user.reason')"
          placeholder-style="color: #ddd"
          v-model="reason"
        />
      </view>
      <view class="register-box-btn register-box-btnOne" id="TencentCaptcha" @click="register">
        {{ i18n.t('user.register') }}
      </view>
      <view class="register-box-btn register-box-btnTwo" @click="jump2Login" v-if="code !== 'undefined'">
        {{ i18n.t('user.exist') }}
      </view>
    </view>
    <qui-registration-agreement></qui-registration-agreement>
  </qui-page>
</template>

<script>
import forums from '@/mixin/forums';
import user from '@/mixin/user';
// #ifdef H5
import tcaptchs from '@/utils/tcaptcha';
// #endif
import { SITE_PAY } from '@/common/const';

export default {
  mixins: [
    forums,
    user,
    // #ifdef H5
    tcaptchs,
    // #endif
  ],
  data() {
    return {
      username: '', // 用户名
      password: '', // 密码
      reason: '', // 注册原因
      url: '', // 上一个页面的路径
      validate: false, // 默认不开启注册审核
      code: '', // 注册邀请码
      register_captcha: false, // 默认不开启注册验证码
      site_mode: '', // 站点模式
      isPaid: false, // 是否付费
      captcha: null, // 腾讯云验证码实例
      captcha_ticket: '', // 腾讯云验证码返回票据
      captcha_rand_str: '', // 腾讯云验证码返回随机字符串
      ticket: '',
      randstr: '',
      captchaResult: {},
    };
  },
  onLoad(params) {
    const { url, validate, commentId, code } = params;
    if (url) {
      let pageUrl;
      if (url.substr(0, 1) !== '/') {
        pageUrl = `/${url}`;
      } else {
        pageUrl = url;
      }
      if (commentId) {
        this.url = `${pageUrl}&commentId=${commentId}`;
      } else {
        this.url = pageUrl;
      }
    }
    if (validate) {
      this.validate = JSON.parse(validate);
    }
    if (code !== 'undefined') {
      this.code = code;
    }
    if (this.forums && this.forums.set_reg && this.forums.set_reg.register_captcha) {
      this.register_captcha = this.forums.set_reg.register_captcha;
    }
    if (this.forums && this.forums.set_site && this.forums.set_site.site_mode) {
      this.site_mode = this.forums.set_site.site_mode;
    }
    this.$u.event.$on('logind', () => {
      if (this.user && this.user.paid) {
        this.isPaid = this.user.paid;
      }
      if (this.site_mode !== SITE_PAY) {
        uni.navigateTo({
          url: this.url,
        });
      }
      if (this.site_mode === SITE_PAY && !this.isPaid) {
        uni.navigateTo({
          url: '/pages/site/info',
        });
      }
    });
  },
  methods: {
    register() {
      if (this.username === '') {
        this.showDialog(this.i18n.t('user.usernameEmpty'));
      } else if (this.password === '') {
        this.showDialog(this.i18n.t('user.passwordEmpty'));
      } else if (this.forums && this.forums.set_reg && this.forums.set_reg.register_captcha) {
        this.toTCaptcha();
      } else {
        this.registerClick();
      }
    },
    // 验证码
    toTCaptcha() {
      // #ifdef H5
      if (this.forums && this.forums.qcloud && this.forums.qcloud.qcloud_captcha_app_id) {
        // eslint-disable-next-line no-undef
        this.captcha = new TencentCaptcha(this.forums.qcloud.qcloud_captcha_app_id, res => {
          if (res.ret === 0) {
            this.ticket = res.ticket;
            this.randstr = res.randstr;
            this.registerClick();
          }
          if (res.ret === 2) {
            uni.hideLoading();
          }
        });
        // 显示验证码
        this.captcha.show();
      }
      // #endif
    },
    registerClick() {
      if (this.forums && this.forums.set_reg && !this.forums.set_reg.register_close) {
        this.$store
          .dispatch('forum/setError', {
            code: 'register_close',
            status: 500,
          })
          .then(res => {
            console.log(res);
            uni.navigateTo({
              url: '/pages/home/index',
            });
          });
      } else {
        const params = {
          data: {
            attributes: {
              username: this.username,
              password: this.password,
            },
          },
        };
        if (this.register_captcha && this.validate) {
          params.data.attributes.register_reason = this.reason;
          params.data.attributes.captcha_ticket = this.ticket;
          params.data.attributes.captcha_rand_str = this.randstr;
        }
        if (this.validate) {
          params.data.attributes.register_reason = this.reason;
        }
        if (this.register_captcha) {
          params.data.attributes.captcha_ticket = this.ticket;
          params.data.attributes.captcha_rand_str = this.randstr;
        }
        if (this.code !== '') {
          params.data.attributes.code = this.code;
        }
        this.$store
          .dispatch('session/h5Register', params)
          .then(result => {
            if (result && result.data && result.data.data && result.data.data.id) {
              this.logind();
              uni.showToast({
                title: this.i18n.t('user.registerSuccess'),
                duration: 2000,
              });
            }
            if (
              result &&
              result.data &&
              result.data.errors &&
              result.data.errors[0].status === '422'
            ) {
              uni.showToast({
                icon: 'none',
                title: result.data.errors[0].detail[0],
                duration: 2000,
              });
            }
            if (
              result &&
              result.data &&
              result.data.errors &&
              result.data.errors[0].code === 'register_validate'
            ) {
              this.$store
                .dispatch('forum/setError', {
                  code: 'register_validate',
                  status: 500,
                })
                .then(res => {
                  console.log(res);
                  uni.navigateTo({
                    url: '/pages/home/index',
                  });
                });
            }
          })
          .catch(err => {
            console.log(err);
          });
      }
    },
    jump2Login() {
      uni.navigateTo({
        url: `/pages/user/login?url=${this.url}&validate=${this.validate}&code=${this.code}`,
      });
    },
    showDialog(title) {
      uni.showToast({
        icon: 'none',
        title,
        duration: 2000,
      });
    },
  },
  onUnload() {
    this.$u.event.$off('logind');
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/base/variable/global.scss';
@import '@/styles/base/theme/fn.scss';
.register-box {
  height: 100vh;
  font-size: $fg-f28;
  background-color: --color(--qui-BG-2);
  background: linear-gradient(206.53deg, #ffffff 0%, #ffffff 32%, #eefff9 51%, #fff0f0 72%, #c6e8ff 100%);

  &-h {
    padding: 60rpx 0rpx 80rpx 40rpx;
    font-size: 50rpx;
    font-weight: bold;
    color: --color(--qui-FC-333);
  }

  &-con {
    margin: 0rpx 40rpx;

    .input {
      width: 100%;
      height: 100rpx;
      padding: 0rpx 0rpx 0rpx 20rpx;
      font-size: $fg-f34;
      line-height: 100rpx;
      text-align: left;
      border-bottom: 2rpx solid --color(--qui-BOR-ED);
    }
  }

  &-btn {
    width: 670rpx;
    height: 90rpx;
    margin: 50rpx auto 0rpx;
    line-height: 90rpx;
    text-align: center;
    border-radius: 5rpx;
  }

  &-btnOne {
    color: --color(--qui-FC-FFF);
    background-color: --color(--qui-BG-BTN);
  }

  &-btnTwo {
    color: --color(--qui-FC-FFF);
    background-color: --color(--qui-FC-TAG);
  }

  &-exist {
    margin: 20rpx 0rpx 0rpx 40rpx;
    color: --color(--qui-LINK);
  }
}
</style>
