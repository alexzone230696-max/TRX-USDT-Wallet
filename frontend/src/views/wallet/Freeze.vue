<script setup>
import { onMounted, onUnmounted, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { showToast } from 'vant'
import request from '@/common/request'
import utils from '@/common/utils'
import Password from '@/components/wallet/Password.vue'

const router = useRouter()
const wallet = ref(null)

const showStakePicker = ref(false)
const showPurposePicker = ref(false)
const stakeType = ref([])
const stakeTypeText = ref('')
const purposeType = ref([])
const purposeTypeText = ref('')
const amount = ref(undefined)
const loading = ref(false)
const balance = ref(0)
let balanceAmount = 0

const estimated = ref(false)
const fee = ref(0)
let trxBalance = 0

const showPassword = ref(false)
let code2fa = ref(null)
let password = ref(null)
const show2FACode = ref(false)

const benifit = ref(0)
const deduct = ref(0)

const stakeColumns = ref([
  { text: '质押', value: 'freeze' },
  { text: '解押', value: 'unfreeze' }
])

const purposeColumns = ref([
  { text: '带宽', value: 'BANDWIDTH' },
  { text: '能量', value: 'ENERGY' }
])

onMounted(() => {
  wallet.value = localStorage.getItem('wallet')
    ? JSON.parse(localStorage.getItem('wallet'))
    : null
})