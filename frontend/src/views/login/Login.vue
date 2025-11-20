<template>
  <div class="login-page">
    <h1>Локальный кошелёк (offline)</h1>

    <div class="card">
      <h3>Create new wallet</h3>
      <button @click="createWallet">Создать новый кошелёк</button>
      <p v-if="newSeed">Сохраните seed/ключ в надёжном месте:</p>
      <textarea v-if="newSeed" readonly rows="3">{{ newSeed }}</textarea>
      <button v-if="newSeed" @click="finishCreate">Я сохранил(а)</button>
    </div>

    <div class="card">
      <h3>Import wallet</h3>
      <label>Seed / Private key / JSON</label>
      <textarea v-model="importText" rows="3"></textarea>
      <input v-model="walletName" placeholder="Имя кошелька (опционально)" />
      <button @click="importWallet">Импортировать</button>
    </div>

    <div class="note">
      <p>Это оффлайн-режим: данные хранятся в вашем браузере (localStorage). Нет серверной авторизации.</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const importText = ref('')
const walletName = ref('')
const newSeed = ref('')

function randomSeed() {
  // очень простая заглушка — при желании заменить на библиотеку bip39/ethereum/tron
  const r = () => Math.random().toString(36).slice(2, 10)
  return `${r()}-${r()}-${r()}`
}

function createWallet() {
  const seed = randomSeed()
  // простая структура кошелька — ты можешь заменить на реальную генерацию
  newSeed.value = seed
  // временно не сохраняем, пользователь должен подтвердить что сохранил
}

function finishCreate() {
  const seed = newSeed.value
  const wallet = {
    address: 'local-' + Math.random().toString(36).slice(2,10),
    seed,
    name: 'Local wallet'
  }
  localStorage.setItem('wallet', JSON.stringify(wallet))
  router.push({ name: 'Home' })
}

function importWallet() {
  const txt = importText.value && importText.value.trim()
  if (!txt) {
    alert('Введите seed / private key / JSON')
    return
  }
  let wallet
  try {
    // если это JSON
    wallet = JSON.parse(txt)
    if (!wallet.address) wallet.address = 'local-' + Math.random().toString(36).slice(2,10)
  } catch (e) {
    // иначе считаем это seed/privateKey
    wallet = {
      address: 'local-' + Math.random().toString(36).slice(2,10),
      seed: txt,
      name: walletName.value || 'Imported wallet'
    }
  }
  localStorage.setItem('wallet', JSON.stringify(wallet))
  router.push({ name: 'Home' })
}
</script>

<style scoped>
.login-page { padding: 20px; }
.card { border: 1px solid #eee; padding: 12px; margin-bottom: 12px; border-radius: 8px; }
.note { color: #666; font-size: 14px; }
</style>