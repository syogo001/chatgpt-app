<template>
  <div>
    <div v-for="message in messages" :key="message.id">
      <div v-if="message.role === 'system'" class="system-message">{{ message.content }}</div>
      <div v-else-if="message.role === 'user'" class="user-message">{{ message.content }}</div>
      <div v-else class="assistant-message">{{ message.content }}</div>
    </div>
    <div class="user-input">
      <input v-model="userMessage" @keydown.enter="sendMessage" placeholder="Type your message..." />
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      messages: [], // チャットメッセージの配列
      userMessage: '' // ユーザーの入力したメッセージ
    };
  },
  methods: {
    async sendMessage() {
      // ユーザーのメッセージを追加
      this.messages.push({ role: 'user', content: this.userMessage });

      // ChatGPTにメッセージを送信してレスポンスを取得
      const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${process.env.VUE_APP_API_KEY}` // ChatGPTのAPIキーを設定
        },
        body: JSON.stringify({
          model: 'gpt-3.5-turbo', // ChatGPTのモデルを指定
          messages: [
            { role: 'system', content: 'You are a helpful assistant.' }, // システムメッセージを追加
            { role: 'user', content: this.userMessage } // ユーザーのメッセージを追加
          ]
        })
      });

      const data = await response.json();

      // ChatGPTのレスポンスの形式に合わせて取得する
      const chatResponse = data.choices[0].message.content;

      // チャットメッセージにChatGPTのレスポンスを追加
      this.messages.push({ role: 'assistant', content: chatResponse });

      // ユーザーの入力をリセット
      this.userMessage = '';
    }
  }
}
</script>

<style scoped>
.system-message {
  color: gray;
}

.user-message {
  color: blue;
}

.assistant-message {
  color: green;
}

.user-input {
  margin-top: 16px;
}
</style>