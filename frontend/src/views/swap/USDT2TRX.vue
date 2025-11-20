<template>
  <van-notice-bar left-icon="info-o" color="#1989fa" background="#ecf9ff">
    <template #default>
      <div class="notify">
        <span class="notify_left" @click="showApprove = true"
          >您已授权兑换 {{ approved }} 个USDT, 点这里提升</span
        >
        <span class="notify_right" @click="refreshApprove" v-if="!loading">刷新</span>
        <span class="notify_right" v-if="loading">
          <van-loading size="24" color="#1989fa" />
        </span>
      </div>
    </template>
  </van-notice-bar>

  <div class="wrapper">
    <van-row class="listWapper">
      <div class="item">
        <div class="left">
          <van-image :src="tether" class="logo">
            <template #error>加载失败</template>
          </van-image>
          <div class="name">{{ 'Tether' }}</div>
        </div>
        <div class="right">
          <div class="amount">{{ usdtBalance }}</div>
          <div class="token">USDT</div>
        </div>
      </div>
    </van-row>
    <van-form @submit="onSwap">
      <van-field
        v-model="swapAmount"
        type="number"
        :disabled="loading"
        name="数量"
        label="数量"
        placeholder="请输入要兑换的USDT数量"
        :rules="[{ required: true, message: '请输入要兑换的USDT数量' }]"
      >
        <template #button>
          <span @click="swapAmount = approved">全部</span>
        </template>
      </van-field>
      <van-notice-bar color="#1989fa" background="#ecf9ff" v-if="swap_etimated">
        大概需要消耗 {{ swap_fee }} 个TRX, 获得 {{ tokenGot }} 个TRX
      </van-notice-bar>

      <div style="margin: 16px 0">
        <van-button
          block
          type="primary"
          native-type="submit"
          :loading="loading"
          :disabled="swapAmount == 0 || swapAmount > realUsdtBalance || swapAmount > approved"
        >
          {{ swap_etimated ? '兑换成TRX' : '估算兑换费用' }}
        </van-button>
      </div>
    </van-form>
  </div>

  <van-overlay :show="showApprove">
    <div class="approve" @click.stop>
      <div class="block">
        <div class="dlg_title">
          <div class="title">更新授权额度</div>
          <div @click="closeAppove"><van-icon name="cross" /></div>
        </div>
        <van-form @submit="onUpdateApprove">
          <van-cell-group inset>
            <van-field
              v-model="newApproved"
              type="number"
              placeholder="请填写新的授权额度"
              :rules="[{ required: true, message: '请填写新的授权额度' }]"
            />
            <van-notice-bar color="#1989fa" background="#ecf9ff" v-if="approve_etimated">
              大概需要消耗 {{ approve_fee }} 个TRX
            </van-notice-bar>
          </van-cell-group>
          <div style="margin: 16px">
            <van-button block type="primary" native-type="submit" :loading="loading">
              {{ approve_etimated ? '授权' : '估算授权费用' }}
            </van-button>
          </div>
        </van-form>
      </div>
    </div>
  </van-overlay>

  <van-popup
    v-if="showPassword"
    v-model:show="showPassword"
    position="bottom"
    :style="{ height: '80%' }"
  >
    <Password @password="onPassword" @cancel="cancelPassword" />
  </van-popup>

  <van-popup
    v-if="show2FACode"
    v-model:show="show2FACode"
    position="bottom"
    :style="{ height: '80%' }"
  >
    <Password @password="on2FACode" info="请输入验证器上的数字" @cancel="cancel2FACode" />
  </van-popup>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import request from '@/common/request'
import utils from '@/common/utils'
import { useRouter } from 'vue-router'
import { showToast } from 'vant'
import Password from '@/components/wallet/Password.vue'
import tether from '@/assets/icons/tether.svg'

const approved = ref(0)
const wallet = ref(null)
const router = useRouter()
const showApprove = ref(false)
const loading = ref(false)
const newApproved = ref(null)
const showPassword = ref(false)
let code2fa = ref(null)
let password = ref(null)
const show2FACode = ref(false)
const approve_fee = ref(0)
const approve_etimated = ref(false)
const swap_etimated = ref(false)
const trxBalance = ref(0)
const realTrxBalance = ref(0)
const usdtBalance = ref(0)
const realUsdtBalance = ref(0)
const swapAmount = ref(null)
const swap_fee = ref(0)
const tokenGot = ref(0)
let verifyType = ''

// Инициализация без backButton
onMounted(() => {
  wallet.value = localStorage.getItem('wallet') ? JSON.parse(localStorage.getItem('wallet')) : null
  refreshApprove()
  loadTron()
  loadUSDT()
})

const loadTron = async () => {
  try {
    let res = await request('/api/wallet/tron/trx/balance', 'POST', {
      address: wallet.value.address
    })
    if (res.code && res.type == 'login') return router.replace({ name: 'Login' })
    trxBalance.value = utils.numFormat(res.data.balance)
    realTrxBalance.value = parseFloat(res.data.balance)
  } catch (error) {
    console.log(error)
  }
}

const loadUSDT = async () => {
  try {
    let res = await request('/api/wallet/tron/usdt/balance', 'POST', {
      address: wallet.value.address
    })
    if (res.code && res.type == 'login') return router.replace({ name: 'Login' })
    usdtBalance.value = utils.numFormat(res.data.balance)
    realUsdtBalance.value = parseFloat(res.data.balance)
  } catch (error) {
    console.log(error)
  }
}

// Остальной код логики swap/approve/password/2FA без изменений
// ...
</script>

<style lang="scss" scoped>
.notify { display: flex; width: 100%; }
.notify_left { flex-grow: 1; }

.wrapper { padding: 10px; display: flex; flex-direction: column; height: calc(100vh - 80px); }
.listWapper { background: #fff; border-radius: 5px; padding: 5px 15px; display: flex; flex-direction: column; }
.item { display: flex; width: 100%; padding: 10px 0; }
.left { display: flex; flex-grow: 1; align-items: center; }
.logo { width: 42px; height: 42px; }
.name { padding-left: 12px; font-size: 18px; }
.right { display: flex; flex-direction: column; justify-content: center; align-items: end; }
.amount { font-size: 24px; }
.token { font-size: 12px; color: #c9c9c9; }

.approve { display: flex; align-items: center; justify-content: center; height: 50vh; }
.block { width: 80vw; padding: 15px; background-color: #eff2f5; border-radius: 5px; }
.dlg_title { font-size: 14px; display: flex; align-items: center; justify-content: center; padding-bottom: 15px; }
.title { display: flex; align-items: center; justify-content: center; flex-grow: 1; font-weight: bold; }
</style>