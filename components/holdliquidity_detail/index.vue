<template>
  <div slot="main">
    <div
      v-loading="loading"
      class="card-container"
    >
      <no-content-prompt :list="pointLog.list">
        <liquidityCard
          v-for="(item, index) in pointLog.list"
          :key="index"
          :card="item"
        />
      </no-content-prompt>
    </div>
    <div
      v-if="loading || total > pointLog.params.pagesize"
      class="pagination"
    >
      <user-pagination
        v-show="!loading"
        :current-page="currentPage"
        :params="pointLog.params"
        :api-url="pointLog.apiUrl"
        :page-size="5"
        :total="total"
        :need-access-token="true"
        :small="true"
        :select-class="'user-pagination-light'"
        class="pagination"
        @paginationData="paginationData"
        @togglePage="togglePage"
      />
    </div>
  </div>
</template>

<script>
import liquidityCard from '@/components/liquidity_card/index.vue'
import userPagination from '@/components/user/user_pagination.vue'

export default {
  components: {
    liquidityCard,
    userPagination
  },
  props: {
    id: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      pointLog: {
        params: {
          tokenId: this.id,
          pagesize: 5
        },
        apiUrl: 'liquidityLogsDetail',
        list: []
      },
      currentPage: Number(this.$route.query['detailH' + this.id]) || 1,
      loading: true, // 加载数据
      total: 0,
      assets: {
      },
      viewStatus: 0, // 0 1
      amount: 0,
      tokenSymbol: ''
    }
  },
  created() {
    if (!this.id) this.$router.go(-1)
  },
  methods: {
    paginationData(res) {
      // console.log(res)
      this.pointLog.list = res.data.list
      this.assets = res.data
      this.total = res.data.count || 0
      this.amount = res.data.amount || 0
      this.tokenSymbol = res.data.tokenDetail.symbol
      this.loading = false
    },
    togglePage(i) {
      this.loading = true
      this.pointLog.list = []
      this.currentPage = i
      const query = { ...this.$route.query }
      query['detailH' + this.id] = i
      this.$router.push({
        query
      })
    }
  }
}
</script>

<style lang="less" scoped>
.card-container {
  margin: 0;
}
.point-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
  margin: 10px 0;
  background-color: #fff;
  padding: 16px 20px;
  border-bottom: 1px solid #DBDBDB;
  .title {
    font-size: 18px;
    font-weight: 400;
    color: #000000;
    line-height: 28px;
  }
  .point-pricing {
    padding: 0;
    margin: 0;
    font-size: 30px;
    font-weight: 700;
    color: #000000;
    line-height: 28px;
  }
}
.tag-title {
  font-weight: bold;
  font-size: 20px;
  padding-left: 10px;
  margin: 0;
}
.pagination {
  margin: 20px 0;
}
</style>
