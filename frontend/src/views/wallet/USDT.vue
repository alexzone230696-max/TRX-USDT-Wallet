<template>
  <van-notice-bar color="#1989fa" background="#ecf9ff" left-icon="info-o">
    暂时只支持查看<span style="font-weight: bold"> 转账 </span>类型的交易记录
  </van-notice-bar>
  <div class="wrapper" v-if="wallet">
    <van-row class="listWapper">
      <div class="item">
        <div class="left">
          <van-image :src="tether" class="logo">
            <template #error>加载失败</template>
          </van-image>
          <div class="name">{{ 'Tether' }}</div>
        </div>
        <div class="right">
          <div class="amount">{{ wallet.balance }}</div>
          <div class="token">USDT</div>
        </div>
      </div>
    </van-row>
    <div class="divider" />
    <van-pull-refresh v-model="loading" @refresh="onRefresh" class="transations">
      <van-list :loading="loading" :finished="finished" finished-text="没有更多了" @load="onLoad">
        <div
          class="transation"
          v-for="(transaction, index) in transactions"
          :key="index"
          :transaction="transaction"
          @click="viewDetail(transaction)"
        >
          <div class="icon">
            <van-icon name="guide-o" v-if="transaction.from === wallet.address" />
            <van-icon name="down" v-else />
          </div>
          <div class="detail">
            <div class="direction">
              <div>{{ transaction.from === wallet.address ? '转出' : '收到' }}</div>
              <div class="amount">
                {{
                  utils.numFormat(
                    parseInt(transaction.value) / 10 ** transaction.token_info.decimals
                  )
                }}
              </div>
              <div class="date">
                {{ utils.dateFormat('yyyy-MM-dd', new Date(transaction.block_timestamp)) }}
              </div>
            </div>
            <div class="address_info">
              <div class="address">
                {{ transaction.from === wallet.address ? '到: ' : '从: '
                }}{{
                  transaction.from === wallet.address
                    ? truncate(transaction.to)
                    : truncate(transaction.from)
                }}
              </div>
              <div class="time">
                {{ utils.dateFormat('hh:mm:ss', new Date(transaction.block_timestamp)) }}
              </div>
            </div>
          </div>
          <div class="more"><van-icon name="arrow" color="#c0c0c0" /></div>
        </div>
      </van-list>
    </van-pull-refresh>
  </div>
</template>

<script lang="js" setup>
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import request from '@/common/request'
import { showToast } from 'vant'
import tether from '@/assets/icons/tether.svg'
import utils from '@/common/utils'

const router = useRouter()
const wallet = ref(null)
const transactions = ref([])
const loading = ref(false)
const finished = ref(false)

function truncate(str) {
  if (str.length > 30) {
    return str.slice(0, 10) + '...' + str.slice(-10)
  }
  return str
}

onMounted(() => {
  wallet.value = localStorage.getItem('wallet') ? JSON.parse(localStorage.getItem('wallet')) : null
  loadUSDT()
  onLoad()
})

let page = 1
let limit = 20
let fingerprint = undefined

const onRefresh = () => {
  page = 1
  loading.value = false
  finished.value = false
  fingerprint = undefined
  loadUSDT()
  onLoad()
}

const onLoad = async () => {
  try {
    loading.value = true
    const res = await request('/api/wallet/tron/usdt/history', 'POST', {
      address: wallet.value.address,
      fingerprint,
      page,
      limit
    })
    loading.value = false
    if (res.code && res.type == 'login') return router.replace({ name: 'Login' })
    if (!res.code) {
      const res_transactions = res.data.transactions.filter(
        t => t.type === 'Transfer' && parseInt(t.value) >= 1 * 10 ** 6
      )
      if (page === 1) {
        transactions.value = res_transactions
      } else {
        transactions.value = [...transactions.value, ...res_transactions]
      }
      if (!res.data.transactions.length || !res.data.fingerprint) finished.value = true
      fingerprint = res.data.fingerprint
      page++
    } else {
      showToast({ message: res.message, position: 'bottom' })
    }
  } catch (error) {
    loading.value = false
    finished.value = true
    showToast({ message: error.message, position: 'bottom' })
  }
}

const viewDetail = (transaction) => {
  let url = process.env.NODE_ENV === 'development'
    ? `https://nile.tronscan.org/#/transaction/${transaction.transaction_id}`
    : `https://tronscan.org/#/transaction/${transaction.transaction_id}`
  window.open(url, '_blank')
}

const loadUSDT = async () => {
  try {
    const res = await request('/api/wallet/tron/usdt/balance', 'POST', {
      address: wallet.value.address
    })
    if (res.code && res.type == 'login') return router.replace({ name: 'Login' })
    if (!res.code) wallet.value.balance = utils.numFormat(res.data.balance)
    else showToast({ message: res.message, position: 'bottom' })
  } catch (error) {
    console.log(error)
  }
}
</script>