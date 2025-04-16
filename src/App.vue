<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { useToast } from 'vue-toastification'

const message = ref('')
const loading = ref(false)
const toast = useToast()

const BOT_TOKEN = import.meta.env.VITE_TELEGRAM_BOT_TOKEN
const GROUP_IDS = import.meta.env.VITE_TELEGRAM_GROUP_IDS.split(',')

const sendMessage = async () => {
  if (!message.value.trim()) {
    toast.error('Please enter a message')
    return
  }

  loading.value = true
  try {
    await Promise.all(
      GROUP_IDS.map(async (groupId: string | number) => {
        try {
          const response = await axios.post(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`, {
            chat_id: groupId,
            text: message.value,
            parse_mode: 'HTML'
          })
          return response
        } catch (error: any) {
          if (error.response) {
            console.error(`Telegram API Error for group ${groupId}:`, {
              status: error.response.status,
              data: error.response.data,
              headers: error.response.headers
            })
            throw new Error(`Failed to send to group ${groupId}: ${error.response.data.description || error.message}`)
          }
          throw error
        }
      })
    )
    toast.success('Messages sent successfully!')
    message.value = ''
  } catch (error: any) {
    const errorMessage = error.message || 'Failed to send messages'
    toast.error(errorMessage)
    console.error('Error:', error)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="absolute inset-0 bg-gradient-to-r from-blue-400 to-indigo-500 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <div class="max-w-md mx-auto">
          <div class="divide-y divide-gray-200">
            <div class="py-8 text-base leading-6 space-y-4 text-gray-700 sm:text-lg sm:leading-7">
              <div class="text-center mb-8">
                <h1 class="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-blue-500 to-indigo-600">
                  JDA Telegram Broadcast to Group
                </h1>
              </div>
              <div class="mb-6">
                <textarea
                  v-model="message"
                  class="w-full px-4 py-3 text-gray-700 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
                  rows="10"
                  cols="100"
                  placeholder="Enter your message..."
                ></textarea>
              </div>
              <button
                @click="sendMessage"
                :disabled="loading"
                class="w-full bg-gradient-to-r from-blue-500 to-indigo-600 text-white font-medium py-3 px-4 rounded-lg hover:from-blue-600 hover:to-indigo-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transform transition duration-200 ease-in-out hover:scale-[1.02] disabled:opacity-50 disabled:cursor-not-allowed disabled:transform-none"
              >
                <span class="flex items-center justify-center">
                  <svg v-if="loading" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                  </svg>
                  {{ loading ? 'Sending...' : 'Send Broadcast' }}
                </span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>