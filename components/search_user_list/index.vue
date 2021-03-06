<template>
  <div class="user-list">
    <n-link :to="{name: 'user-id', params: {id: card.id}}">
      <avatar
        :src="avatarSrc"
        size="80px"
        class="avatar"
      />
    </n-link>
    <div class="user-info">
      <n-link :to="{name: 'user-id', params: {id: card.id}}">
        <p
          class="user-title"
          v-html="userTitle"
        />
      </n-link>
      <p class="user-num">
        <span>{{ $t('follow') }}: {{ card && card.follows }}</span>
        &nbsp;
        <span>{{ $t('fans') }}: {{ card && card.fans }}</span>
      </p>
      <p class="user-des">
        {{ card && card.introduction || $t('notProfile') }}
      </p>
    </div>
    <template v-if="!isMe(card.id)">
      <el-button
        :class="!card.is_follow && 'black'"
        class="follow"
        @click.stop="followOrUnFollow"
      >
        <i
          v-if="!card.is_follow"
          class="el-icon-plus"
        />
        {{ followBtnText }}
      </el-button>
    </template>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import avatar from '@/components/avatar/index.vue'
import { xssFilter } from '@/utils/xss'

export default {
  components: {
    avatar
  },
  props: {
    card: {
      type: Object,
      required: true
    }
  },
  computed: {
    ...mapGetters(['currentUserInfo', 'isLogined', 'isMe']),
    avatarSrc() {
      if (this.card.avatar) return this.$ossProcess(this.card.avatar)
      return ''
    },
    userTitle() {
      return xssFilter(this.card.nickname || this.card.username)
    },
    followBtnText() {
      return this.card.is_follow ? this.$t('following') : this.$t('follow')
    }
  },
  methods: {
    followOrUnFollow() {
      if (this.card.is_follow) {
        this.$confirm(this.$t('p.confirmUnFollowMessage'), this.$t('promptTitle'), {
          confirmButtonText: this.$t('confirm'),
          cancelButtonText: this.$t('cancel'),
          type: 'warning'
        }).then(() => {
          this.followOrUnfollowUser(this.card.id, 0)
        })
      } else {
        this.followOrUnfollowUser(this.card.id, 1)
      }
    },
    async followOrUnfollowUser(id, type) {
      if (!this.isLogined) return this.$store.commit('setLoginModal', true)
      const message = type === 1 ? this.$t('follow') : this.$t('unFollow')

      try {
        let res = null
        if (type === 1) res = await this.$API.follow(id)
        else res = await this.$API.unfollow(id)
        if (res.code === 0) {
          this.$message({ showClose: true, message: `${message}${this.$t('success.success')}`, type: 'success'})

          this.$emit('updateList')
          this.card.is_follow = type === 1
        } else {
          this.$message({ showClose: true, message: `${message}${this.$t('error.fail')}`, type: 'error' })
        }
      } catch (error) {
        this.$message({ showClose: true, message: `${message}${this.$t('error.fail')}`, type: 'error' })
      }
    }
  }
}
</script>

<style lang="less" scoped>
.avatar {
  margin-right: 16px;
  cursor: pointer;
}
.user-list {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 1);
  border-radius: @br10;
  margin: 20px 0;
  padding: 20px;
  transition: all 0.3s;

  &:hover {
    transform: translate(0, -2px);
    box-shadow: 0 0 30px 0 rgba(0, 0, 0, 0.1);
  }
}
.user-info {
  flex: 1;
  height: 80px;
  .user-title {
    padding: 0;
    margin: 4px 0;
    font-size: 20px;
    font-weight: 500;
    color: #333;
    cursor: pointer;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
    word-break: break-all;
  }
  .user-num,
  .user-des {
    padding: 0;
    margin: 4px 0;
    font-size: 16px;
    font-weight: 400;
    color: rgba(178, 178, 178, 1);
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
    word-break: break-all;
  }
  .user-num {
    margin: 6px 0;
  }
}

.follow {
  margin-left: 10px;
  &.black {
    background: #333;
    color: #fff;
    border: 1px solid #333;
  }
}
</style>

<style lang="less">
.user-title em {
  font-size: 20px;
  font-style: inherit;
}
</style>
