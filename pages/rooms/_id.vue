<template>
  <div class="relative">
    <div class="px-4 pb-32"></div>
    <form
      @submit.prevent="onSubmit"
      class="fixed bottom-0 bg-white w-full max-w-sm flex py-4 border-t border-gray-300"
    >
      <textarea
        v-model="form.message.val"
        placeholder="発言してみよう"
        class="block appearance-none w-full ml-2 py-3 px-4 rounded-lg border border-gray-400 text-darkGray focus:outline-none focus:bg-white overflow-hidden bg-blue-100"
        name="form.body"
      />
      <button
        :disable="isValidateError"
        :class="{ 'text-blue': !isValidateError }"
        class="w-2/12 flex items-start justify-center text-gray font-semibold"
      >
        送信
      </button>
    </form>
  </div>
</template>

<script>
export default {
  middleware: ['checkAuth'],
  data() {
    return {
      form: {
        message: {
          val: null
        }
      }
    }
  },

  computed: {
    isValidateError() {
      return !this.form.message.val
    }
  },

  methods: {
    async onSubmit() {

      // まず未入力チェックを行い、未入力出ない場合は自分のユーザー情報を取得 
      // user 情報が存在しない場合ログイン画面にリダイレクト
      if (this.isValidateError) return
      const user = await this.$user()
      if (!user) this.$router.push('/login')

      // まず、 this.$route.params.id で開いているチャットルームの ID を取得
      // このチャットページは先程 pages/rooms/_id.vue という名前で作成
      // _からファイル名やディレクトリ名が始まる場合は、http://3000/rooms/ルームID 
      const roomId = this.$route.params.id
      const chat = {
        userId: user.uid,
        name: user.name,
        iconImageUrl: user.iconImageUrl,
        body: this.form.message.val,
        createdAt: this.$firebase.firestore.FieldValue.serverTimestamp()
      }
      // 以下でfirestoreに登録。チャットの UID は Firestore で自動生成する。setではなくaddとすることでfirestoreでUIDを自動生成してデータを追加する。
      try {
        await this.$firestore
          .collection('rooms')
          .doc(roomId)
          .collection('chats')
          .add(chat)
        this.resetForm()　
        this.scrollBottom()
      } catch (e) {
        this.setMessage({ message: '登録に失敗しました。'})
      }
    },

    scrollBottom() {
      const element = document.documentElement
      const bottom = element.scrollHeight - element.clientHeight
      window.scroll(0, bottom)
    },

    resetForm() {
      this.form.message.val = null
    }
  }
}
</script>