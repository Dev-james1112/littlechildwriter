<template>
  <div class="create">
    <h1>계정 만들기</h1>
    <p>계정이 있으면 <NLink to="/account/login"> 로그인 </NLink>하기</p>

    <v-divider />

    <br />

    <v-form>
      <v-text-field v-model="displayName" label="이름" />
      <v-text-field
        v-model="email"
        label="Email"
        placeholder="Email"
        outlined
        clearable
        prepend-inner-icon="mdi-email"
      />
      <v-text-field
        v-model="password"
        type="password"
        label="Password"
        placeholder="Password"
        outlined
        clearable
        prepend-inner-icon="mdi-key"
      />

      <v-btn color="primary" @click="Make">
        <v-icon left> mdi-account </v-icon> 게정 만들기
      </v-btn>
    </v-form>
  </div>
</template>

<script setup lang="ts">
import { auth, db } from '@/plugins/firebase'

const router = useRouter()
const email = ref<string>('')
const password = ref<string>('')
const displayName = ref<string>('')

const Make = () => {
  auth.createUserWithEmailAndPassword(email.value, password.value).then(() => {
    db.ref(`/users/${auth.currentUser?.uid}`).set({
      displayName,
      photoURL:
        'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTWM85l8_8g0csORwtfpgd5Dyv-f9twzcVuQ0v2FzawHw&s',
      libris: 0,
      bio: `새롭게 계정 만든 ${displayName}입니다. (바꾸고 깊다면 편집 버튼을 누르세요.)`,
      subscribe: [],
      subscriber: []
    })

    router.push('/account/account')
  })
}

useHead({
  title: '만들기 - LCW'
})
</script>

<style scoped>
.create {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: calc(100vh - 88px);
}
</style>
