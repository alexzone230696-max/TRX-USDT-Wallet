<template>
  <div class="wrapper" v-if="wallet">
    <Card
      :wallet="wallet"
      :editable="true"
      :copyable="true"
      :clickable="false"
      :info="true"
      :show-stake="true"
      :timestamp="timestamp"
    />

    <div class="actions">
      <van-grid direction="vertical" :column-num="4">
        <van-grid-item icon="guide-o" text="发送" @click="router.push({ name: 'Send' })" />
        <van-grid-item icon="down" text="接收" @click="router.push({ name: 'Receive' })" />
        <van-grid-item icon="exchange" text="兑换" @click="showSwap = true" />
        <van-grid-item icon="replay" text="刷新" @click="onRefresh" />
      </van-grid>
    </div>

    <van-row class="listWapper">
      <div class="item" @click="enterHistory('trx')">
        <div class="left">
          <van-image :src="tron" class="logo">
            <template #error>加载失败</template>
          </van-image>
          <div class="name">TRX</div>
        </div>
        <div class="right">
          <div class="amount">{{ trx }}</div>
        </div>
      </div>

      <div class="divider" />

      <div class="item" @click="enterHistory('usdt')">
        <div class="left">
          <van-image :src="tether" class="logo">
            <template #error>加载失败</template>
          </van-image>
          <div class="name">USDT</div>
        </div>
        <div class="right">
          <div class="amount">{{ usdt }}</div>
        </div>
      </div>
    </van-row>
  </div>

  <van-action-sheet
    v-model:show="showSwap"
    :actions="swapActions"
    cancel-text="取消"
    close-on-click-action
    @select="onSwapAction"
  />
</template>

<script setup>
import { onMounted, ref } from 'vue'
import { TronWeb } from 'tronweb'
import { useRouter } from 'vue-router'
import request from '@/common/request'
import utils from '@/common/utils'
import Card from '@/components/wallet/Card.vue'
import { closeToast, showLoadingToast, showToast } from 'vant'
import tron from '@/assets/icons/tron.svg'
import tether from '@/assets/icons/tether.svg'

const router = useRouter()

const wallet = ref(null)
const trx = ref(0)
const usdt = ref(0)
const loading = ref(false)
const timestamp = ref(0)

onMounted(() => {
  // Загружаем кошелёк
  wallet.value =
    localStorage.getItem('wallet') ?
    JSON.parse(localStorage.getItem('wallet')) :
    null

  // Загружаем балансы TRX + USDT
  loadTron()
  loadUSDT()
})

const onRefresh = async () => {
  showLoadingToast({
    duration: 0,
    forbidClick: true,
    message: '正在刷新...'
  })

  try {
    loading.value = true
    timestamp.value = Date.now()
    await loadTron()
    await loadUSDT()
    loading.value = false
    closeToast()
  } catch {
    loading.value = false
    closeToast()
  }
}

const enterHistory = (subtype) => {
  const tronWeb = new TronWeb({
    fullHost: 'https://api.trongrid.io'
  })

  wallet.value.trxAddr = tronWeb.address.toHex(wallet.value.address)
  wallet.value.subtype = subtype
  wallet.value.balance = subtype === 'trx' ? trx.value : usdt.value

  localStorage.setItem('wallet', JSON.stringify(wallet.value))

  if (subtype === 'trx') {
    router.push({ name: 'TRX' })
  } else {
    router.push({ name: 'USDT' })
  }
}

const loadTron = async () => {
  try {
    const res = await request('/api/wallet/tron/trx/balance', 'POST', {
      address: wallet.value.address
    })

    if (res.code && res.type === 'login') {
      return router.replace({ name: 'Login' })
    }

    if (!res.code) {
      trx.value = utils.numFormat(res.data.balance)
    } else {
      showToast({
        message: res.message,
        position: 'bottom'
      })
    }
  } catch (e) {
    console.log(e)
  }
}

const loadUSDT = async () => {
  try {
    const res = await request('/api/wallet/tron/usdt/balance', 'POST', {
      address: wallet.value.address
    })

    if (res.code && res.type === 'login') {
      return router.replace({ name: 'Login' })
    }

    if (!res.code) {
      usdt.value = utils.numFormat(res.data.balance)
    } else {
      showToast({
        message: res.message,
        position: 'bottom'
      })
    }
  } catch (e) {
    console.log(e)
  }
}

const showSwap = ref(false)
const swapActions = [
  { name: 'TRX转USDT', val: 'trx2usdt' },
  { name: 'USDT转TRX', val: 'usdt2trx' }
]

const onSwapAction = (action) => {
  if (action.val === 'usdt2trx') {
    router.push({ name: 'USDT2TRX' })
  } else if (action.val === 'trx2usdt') {
    router.push({ name: 'TRX2USDT' })
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
  padding: 10px 10px;

  .actions {
    padding-bottom: 10px;
    :deep(.van-grid-item__content) {
      border-radius: 5px;
    }
  }
}

.listWapper {
  background-color: #fff;
  border-radius: 5px;
  padding: 5px 15px;
  display: flex;
  flex-direction: column;

  .item {
    display: flex;
    width: 100%;
    padding: 10px 0;

    .left {
      display: flex;
      flex-grow: 1;
      align-items: center;

      .logo {
        width: 42px;
        height: 42px;
      }

      .name {
        padding-left: 12px;
        font-size: 18px;
      }
    }

    .right {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: end;

      .amount {
        font-size: 24px;
      }

      .token {
        font-size: 12px;
        color: #c9c9c9;
      }
    }
  }

  .divider {
    height: 1px;
    background-color: #f9f9f9;
  }
}
</style>