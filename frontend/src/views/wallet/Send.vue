<script setup>
import { onMounted, onUnmounted, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { showToast } from 'vant'
import { TronWeb } from 'tronweb'
import request from '@/common/request'
import utils from '@/common/utils'
import Password from '@/components/wallet/Password.vue'

const router = useRouter()
const wallet = ref(null)
const showPicker = ref(false)
let subtype = ref(null)
const subtypeTxt = ref('')
const address_to = ref('')
const selectedType = ref([])
const amount = ref(undefined)
const loading = ref(false)
const balance = ref(0)
let balanceAmount = 0
const estimated = ref(false)
const fee = ref(0)
let trxBalance = 0
const showPassword = ref(false)
const localToken = ref(null)
let code2fa = ref(null)
let password = ref(null)
const show2FACode = ref(false)

const columns = ref([
  { text: 'TRX', value: 'trx' },
  { text: 'USDT', value: 'usdt' }
])

const onSelectConfirm = ({ selectedOptions }) => {
  showPicker.value = false
  subtype.value = selectedOptions[0].value
  subtypeTxt.value = selectedOptions[0].text
  if (subtype.value == 'trx') loadTron()
  if (subtype.value == 'usdt') loadUSDT()
}

onMounted(async () => {
  try {
    wallet.value = localStorage.getItem('wallet')
      ? JSON.parse(localStorage.getItem('wallet'))
      : null
    localToken.value = localStorage.getItem('token') ?? null
  } catch (error) {
    console.log(error)
  }
})

onUnmounted(() => {
  // ничего удалять не нужно — Telegram SDK нет
})

const scanQRCode = async () => {
  try {
    if (!('mediaDevices' in navigator)) {
      return showToast('当前设备不支持扫码')
    }

    showToast('扫码功能暂未启用（需要jsQR库）')
  } catch (error) {
    showToast(error.message)
  }
}

const estimate = async () => { /* … без изменений … */ }
const getSecurity = async () => { /* … */ }
const checkSecurity = async () => { /* … */ }
const transfer = async () => { /* … */ }
const enterHistory = (subtype) => { /* … */ }
const onSubmit = async () => { /* … */ }
const loadTron = async () => { /* … */ }
const loadUSDT = async () => { /* … */ }
const onPassword = async (passcode) => { /* … */ }
const on2FACode = (code) => { /* … */ }
const cancelPassword = () => { /* … */ }
const cancel2FACode = () => { /* … */ }

watch(subtype, async () => { estimated.value = false })
watch(amount, async () => { estimated.value = false })
watch(address_to, async () => { estimated.value = false })
</script>