<template>
  <view class="my">
    <!-- #ifdef MP-WEIXIN -->
    <uni-nav-bar
      :title="i18n.t('profile.mine')"
      fixed="true"
      :color="currentTheme ? '#fff' : '#000'"
      :background-color="currentTheme ? '#2e2f30' : '#fff'"
      status-bar
    ></uni-nav-bar>
    <!-- #endif -->
    <scroll-view
      scroll-y="true"
      scroll-with-animation="true"
      show-scrollbar="false"
      class="scroll-y"
    >
      <view class="my-info">
        <view class="my-info__box">
          <view class="my-info__box__detail fbh">
            <qui-avatar :user="userInfo" :size="120" />
            <view class="fbv fbjc">
              <view class="my-info__box__detail-username">
                {{ userInfo.username }}
                <view
                  :class="['badge', [userInfo.groupsName]]"
                  v-if="userInfo.groupsName == '' ? !isBadge : isBadge"
                >
                  {{ userInfo.groupsName }}
                </view>
              </view>
              <view class="my-info__introduction" v-if="userInfo.signature">
                {{ userInfo.signature }}
              </view>
              <!-- <view class="thrtw-box">
                <view class="fbv ctw">
                  <text class="fw3 fs20 pt8">
                    {{ userInfo.signature || '该用户还未填写签名哦！' }}
                  </text>
                </view>
                <qui-icon
                  class="icon ctw thr-icon"
                  name="icon-icon-arrow-right2"
                  size="34"
                ></qui-icon>
              </view> -->
            </view>
            <!-- <qui-cell-item
              :title="userInfo.username || ''"
              :brief="userInfo.groupsName"
              :border="false"
              class="my-info__box__detail-username"
            >
              
            </qui-cell-item> -->
          </view>
        </view>
        <view class="my-tabs">
          <qui-tabs
            :hide-border="false"
            :values="items"
            @clickItem="onClickItem"
            :brief="true"
            :current="-1"
          ></qui-tabs>
        </view>
      </view>

      <!-- <view class="my-info">
        <view class="my-info__box">
          <view class="my-info__box__detail">
            <qui-avatar :user="userInfo" :is-real="userInfo.isReal" />
            <qui-cell-item
              :title="userInfo.username || ''"
              :brief="userInfo.groupsName"
              :border="false"
              class="my-info__box__detail-username"
            ></qui-cell-item>
          </view>
        </view>
        <view class="my-info__introduction" v-if="userInfo.signature">
          {{ userInfo.signature }}
        </view>
      </view>
      <view class="my-tabs">
        <qui-tabs :values="items" @clickItem="onClickItem" :brief="true" :current="-1"></qui-tabs>
      </view> -->
      <view class="my-items">
        <view class="my-items__wrap">
          <navigator url="/pages/my/profile" hover-class="none" class="fbh">
            <view class="fbh themlist">
              <qui-cell-item
                left-icon="icon-wenzhang_2"
                :title="i18n.t('profile.myprofile')"
                arrow
                icon-background-color=""
              ></qui-cell-item>
              <qui-icon class="fbh icon_right" name="icon-icon-arrow-right2"></qui-icon>
            </view>
          </navigator>
          <navigator url="/pages/my/wallet" hover-class="none" v-if="forums.paycenter.wxpay_close">
            <qui-cell-item
              left-icon="icon-qianbao"
              :title="i18n.t('profile.mywallet')"
              arrow
            ></qui-cell-item>
          </navigator>
          <navigator url="/pages/my/favorite" hover-class="none" class="fbh">
            <view class="fbh themlist">
              <qui-cell-item
                :title="i18n.t('profile.myfavorite')"
                arrow
                left-icon="icon-dingyue"
                :border="false"
              ></qui-cell-item>
              <qui-icon class="fbh icon_right" name="icon-icon-arrow-right2"></qui-icon>
            </view>
          </navigator>
        </view>
        <view class="my-items__wrap">
          <navigator url="/pages/site/index" hover-class="none" class="fbh">
            <view class="fbh themlist">
              <qui-cell-item
                left-icon="icon-ziyuan_2"
                :title="i18n.t('profile.circleinfo')"
                arrow
              ></qui-cell-item>
              <qui-icon class="fbh icon_right" name="icon-icon-arrow-right2"></qui-icon>
            </view>
          </navigator>
          <navigator url="/pages/site/search" hover-class="none" class="fbh">
            <view class="fbh themlist">
              <qui-cell-item
                :title="i18n.t('profile.search')"
                arrow
                left-icon="icon-sousuo_2"
                :border="forums.other && forums.other.can_create_invite ? true : false"
              ></qui-cell-item>
              <qui-icon class="fbh icon_right" name="icon-icon-arrow-right2"></qui-icon>
            </view>
          </navigator>
          <navigator
            v-if="forums.other && forums.other.can_create_invite"
            url="/pages/manage/index"
            hover-class="none"
          >
            <qui-cell-item
              :title="i18n.t('profile.circlemanagement')"
              arrow
              left-icon="icon-guanli"
              :border="false"
            ></qui-cell-item>
          </navigator>
        </view>

        <view class="my-items__wrap">
          <qui-cell-item
            left-icon="icon-kapianxingshi"
            :title="i18n.t('profile.theme')"
            slot-right
            :border="false"
          >
            <u-switch
              @change="changeCheck"
              v-model="currentTheme"
              active-color="#1E78F3"
            ></u-switch>
          </qui-cell-item>
        </view>
        <!-- <view class="my-items__wrap">
          <qui-cell-item
            left-icon="icon-zhongyingwenyuyan"
            :title="i18n.t('profile.enableEnLang')"
            slot-right
            :border="false"
          >
            <u-switch @change="changeLangs" v-model="lang" active-color="#1E78F3"></u-switch>
          </qui-cell-item>
        </view> -->
        <!-- #ifdef H5-->
        <!-- 微信内：无感模式不展示按钮，其他模式展示退出并解绑按钮，微信外：任何模式都展示退出登录按钮 -->
        <view class="logout">
          <qui-button
            size="large"
            type="warn"
            @click="handleClick"
            v-if="isWeixin && offiaccount_close && register_type !== 2"
          >
            {{ i18n.t('user.noBind') }}
          </qui-button>
          <qui-button
            size="large"
            type="warn"
            @click="handleClick"
            v-if="isWeixin && !offiaccount_close"
          >
            {{ i18n.t('user.logout') }}
          </qui-button>
          <qui-button size="large" type="warn" @click="handleClick" v-if="!isWeixin">
            {{ i18n.t('user.logout') }}
          </qui-button>
        </view>
        <!-- #endif -->
      </view>
    </scroll-view>
    <uni-popup ref="popup" type="center">
      <uni-popup-dialog
        type="warn"
        :content="i18n.t('user.loginOutTips')"
        :before-close="true"
        @close="handleClickCancel"
        @confirm="handleClickOk"
      ></uni-popup-dialog>
    </uni-popup>
  </view>
</template>

<script>
import { THEME_DEFAULT, THEME_DARK } from '@/common/const';
import forums from '@/mixin/forums';
import appCommonH from '@/utils/commonHelper';
import uniPopupDialog from '@/components/uni-popup/uni-popup-dialog';
import { i18n, localeUse } from '@/locale';

export default {
  components: { uniPopupDialog },
  mixins: [forums, appCommonH],
  data() {
    return {
      items: [
        { title: this.i18n.t('profile.topic'), brief: '0' },
        { title: this.i18n.t('profile.following'), brief: '0' },
        { title: this.i18n.t('profile.followers'), brief: '0' },
        { title: this.i18n.t('profile.likes'), brief: '0' },
      ],
      current: 0,
      currentTheme: false, // 当前主题是否为暗色
      changeLang: false, //  当前多语言
      isWeixin: false, // 默认不是微信浏览器
      offiaccount_close: false, // 默认不开启微信公众号
      register_type: 0, // 注册模式
      site_mode: '', // 站点模式
      lang: false,
      isBadge: true,
      username: '',
      userGroupName: '',
    };
  },
  computed: {
    userId() {
      return this.$store.getters['session/get']('userId');
    },
    userInfo() {
      const userInfo = this.$store.getters['jv/get'](`users/${this.userId}`);
      userInfo.groupsName = userInfo.groups ? userInfo.groups[0].name : '';
      this.setNum(userInfo);
      return userInfo;
    },
  },
  // #ifdef H5
  created() {
    if (this.forums && this.forums.set_reg) {
      this.register_type = this.forums.set_reg.register_type;
    }
    if (this.forums && this.forums.set_site) {
      this.site_mode = this.forums.set_site.site_mode;
    }
    if (this.forums && this.forums.passport) {
      this.offiaccount_close = this.forums.passport.offiaccount_close;
    }
    const { isWeixin } = appCommonH.isWeixin();
    this.isWeixin = isWeixin;

    // 多语言默认值
    if (i18n.locale !== 'zh') this.lang = true;
  },
  onShow() {
    // ? NOTE: 当页面展示时，获取当前主题
    this.ontrueGetList();
  },
  // #endif
  methods: {
    // 切换多语言
    changeLangs(e) {
      this.$localeUse(e ? 'en' : 'zh');
      // window.location.reload();
      // localeUse(e ? 'en' : 'zh');
    },
    changeCheck(e) {
      getApp().globalData.themeChanged(e ? THEME_DARK : THEME_DEFAULT);
      // window.location.reload();
    },
    onClickItem(e) {
      uni.navigateTo({
        url: `/pages/profile/index?current=${e.currentIndex}&userId=${this.userId}`,
      });
    },
    // #ifdef H5
    handleClick() {
      if (this.isWeixin) {
        // 微信内
        if (this.offiaccount_close) {
          if (this.register_type !== 2) {
            this.$refs.popup.open();
          }
        } else {
          this.$store.dispatch('session/logout').then(() => window.location.reload());
        }
      } else {
        this.$store.dispatch('session/logout').then(() => window.location.reload());
      }
    },
    // #endif
    // #ifdef H5
    handleClickOk() {
      this.$store.dispatch('jv/delete', `users/${this.userId}/wechat`).then(() => {
        this.handleClickCancel();
        this.$store.dispatch('session/logout').then(() => window.location.reload());
      });
    },
    // #endif
    // #ifdef H5
    handleClickCancel() {
      this.$refs.popup.close();
    },
    // #endif
    // 设置粉丝点赞那些数字
    setNum(res) {
      this.items[0].brief = res.threadCount || 0;
      this.items[1].brief = res.followCount || 0;
      this.items[2].brief = res.fansCount || 0;
      this.items[3].brief = res.likedCount || 0;
    },
    // 组件初始化数据
    ontrueGetList() {
      this.currentTheme = this.theme !== THEME_DEFAULT;
    },
    // 获取最新主题数那些
    refreshNum() {
      const userId = this.$store.getters['session/get']('userId');
      const params = {
        include: 'groups,wechat',
      };
      this.$store.dispatch('jv/get', [`users/${userId}`, { params }]);
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/base/variable/global.scss';
@import '@/styles/base/theme/fn.scss';
/* #ifdef H5 */
$height: calc(100vh - 120rpx);
/* #endif */

/* #ifdef MP-WEIXIN */
$height: calc(100vh - 260rpx);
/* #endif */
.my-items__wrap {
  // padding-left: 20rpx;
  margin: 30rpx 20rpx 0;
  background: --color(--qui-BG-2);
  // border-bottom: 2rpx solid --color(--qui-BOR-ED);
  transition: $switch-theme-time;
  box-shadow: 0 0 10rpx #ccc3;
  border-radius: 10rpx;
}
.my-items {
  padding-bottom: 60rpx;
}
.my-info {
  justify-content: space-between;
  padding: 20rpx;
  padding-top: 60rpx;
  // height: 180rpx;
  margin-bottom: 100rpx;
  font-size: $fg-f28;
  // background: --color(--qui-BG-HIGH-LIGHT);
  // animation: identifier 100s infinite linear;
  // animation-direction: 5s;
  // animation-delay: 0s;
  // animation-iteration-count: 100;
  // animation-direction: alternate;
  // background: linear-gradient(
  //   101.35deg,
  //   --color(--qui-BG-HIGH-LIGHT) 14.66%,
  //   #0a9393 29.72972972972973%,
  //   #11e397 34%,
  //   #000066 100%
  // );
  background: linear-gradient(to right, rgb(62, 135, 250), rgb(95, 226, 226)); //背景颜色向右渐变
}
.my-info__box {
  display: flex;
  justify-content: space-between;
}
.my-info__introduction {
  position: relative;
  padding-top: 0.5rem;
  margin-left: 0.5rem;
  font-size: 24rpx;
  color: --color(--qui-FC-GRAY);
  word-break: break-all;
  transition: $switch-theme-time;
  text-align: justify; //两端对齐
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
.my-info__box__detail {
  position: relative;
  display: flex;
  flex-direction: row;
  width: 100%;
  font-size: $fg-f28;
  box-sizing: border-box;
}
.my-info__box__detail-username {
  display: flex;
  align-items: center;
  font-size: 45rpx;
  padding-left: 20rpx;
  color: #fff;
  // background: red;
}
.badge {
  font-size: 16rpx;
  line-height: 16rpx;
  align-items: center;
  background: #fff !important;
  // color: yellow;
  border: #ccc 1px solid;
  color: #ccc;
  border-radius: 20rpx;
  margin-left: 10rpx;
  padding: 8rpx 10rpx;
  transform: scale(0.8);
}
// .管理员 {
//   border: #1878f3 1px solid;
//   color: #1878f3;
// }
// .普通会员 {
//   border: #ccc 1px solid;
//   color: #ccc;
// }
.my-tabs {
  // background: --color(--qui-BG-2);
  background: --color(--qui-BG-FFF);
  border-radius: 20rpx;
  overflow: hidden;
  transform: translateY(50rpx);
  transition: $switch-theme-time;
}
.scroll-y {
  max-height: $height;
}
.logout {
  margin: 30rpx 30rpx 0;
  text-align: center;
  border-radius: 7rpx;
}

@keyframes identifier {
  0% {
    background-size: 100%;
  }
  50% {
    background-size: 200%;
  }
  100% {
    background-size: 100%;
  }
}
.themlist {
  display: flex;
  width: 100vw;
  justify-content: space-between;
}
.icon_right {
  display: flex;
  align-items: center;
  margin-right: 25rpx;
  color: #ccc;
}
.thrtw-box {
  display: flex;
  max-width: 11rem;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  color: #fff;
}
// 定义最多显示两行，多出的用...显示
.thrtw-box text {
  text-align: justify; //两端对齐
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
.thr-icon {
  // padding-left: 5.3rem;
  position: absolute;
  right: 1.8rem;
  // color: #fff;
}
</style>
