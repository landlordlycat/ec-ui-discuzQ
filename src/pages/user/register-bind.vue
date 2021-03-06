<template>
  <qui-page :data-qui-theme="theme">
    <view class="register-bind-box">
      <view class="register-bind-box-h">{{ i18n.t('user.registerBind') }}</view>
      <view class="register-bind-box-con">
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
      <view class="register-bind-box-btn" id="TencentCaptcha" @click="register">
        {{ i18n.t('user.registerBindId') }}
      </view>
      <view class="register-bind-box-login" @click="jump2LoginBind">
        {{ i18n.t('user.loginBindId') }}
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
      token: '', // token
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
    const { url, validate, token, commentId, code } = params;
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
    if (token) {
      this.token = token;
    }
    if (this.forums && this.forums.set_reg && this.forums.set_reg.register_captcha) {
      this.register_captcha = this.forums.set_reg.register_captcha;
    }
    if (this.forums && this.forums.set_site && this.forums.set_site.site_mode) {
      this.site_mode = this.forums.set_site.site_mode;
    }
    if (this.user && this.user.paid) {
      this.isPaid = this.user.paid;
    }
  },
  created() {
    uni.$on('logind', () => {
      let url = '';
      if (this.url) {
        url = this.url;
      }
      if (this.site_mode !== SITE_PAY) {
        uni.navigateTo({
          url,
        });
      }
      if (this.site_mode === SITE_PAY && !this.isPaid) {
        uni.navigateTo({
          url: '/pages/site/info',
        });
      }
    });
  },
  destroyed() {
    uni.$off('logind');
  },
  methods: {
    register() {
      if (this.username === '') {
        this.showDialog(this.i18n.t('user.usernameEmpty'));
      } else if (this.password === '') {
        this.showDialog(this.i18n.t('user.passwordEmpty'));
      } else if (this.register_captcha) {
        this.toTCaptcha();
      } else {
        this.registerBind();
      }
    },
    // 验证码
    toTCaptcha() {
      // #ifdef H5
      // eslint-disable-next-line no-undef
      this.captcha = new TencentCaptcha(this.forums.qcloud.qcloud_captcha_app_id, res => {
        if (res.ret === 0) {
          this.ticket = res.ticket;
          this.randstr = res.randstr;
          this.registerBind();
        }
        if (res.ret === 2) {
          uni.hideLoading();
        }
      });
      // 显示验证码
      this.captcha.show();
      // #endif
    },
    registerBind() {
      const params = {
        data: {
          attributes: {
            username: this.username,
            password: this.password,
            token: this.token,
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
              title: this.i18n.t('user.registerBindSuccess'),
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
        })
        .catch(err => {
          console.log(err);
        });
    },
    showDialog(title) {
      uni.showToast({
        icon: 'none',
        title,
        duration: 2000,
      });
    },
    jump2LoginBind() {
      uni.navigateTo({
        url: `/pages/user/login-bind?url=${this.url}&validate=${this.validate}&token=${this.token}&code=${this.code}`,
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
.register-bind-box {
  height: 100vh;
  font-size: $fg-f28;
  background-color: --color(--qui-BG-2);

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
    color: --color(--qui-FC-FFF);
    text-align: center;
    background-color: --color(--qui-MAIN);
    border-radius: 5rpx;
  }

  &-login {
    margin: 20rpx 0rpx 0rpx 40rpx;
    font-size: $fg-f28;
    color: --color(--qui-LINK);
  }
}
</style>
