<script setup>
import axios from 'axios'
import { ref } from 'vue'
import { RouterLink, useRouter } from 'vue-router'
import Swal from 'sweetalert2'

const apiPath = 'https://todolist-api.hexschool.io'
const signInData = ref({})
const router = useRouter()

/**
 * swal 提示訊息
 * @param param0
 */
const swalMessage = ({ type, text }) => {
  Swal.fire({
    title: '訊息',
    text: text,
    icon: type,
    confirmButtonText: '關閉'
  })
}

/**
 * 登入
 */
const signIn = async () => {
  try {
    const response = await axios.post(`${apiPath}/users/sign_in`, {
      email: signInData.value.email,
      password: signInData.value.password
    })
    swalMessage({ type: 'success', text: '登入成功' })
    const token = response.data.token
    document.cookie = `todoListToken=${token}; expires=${new Date(response.data.exp * 1000)}; path=/`
    router.push('/todoList')
  } catch (error) {
    swalMessage({
      type: 'error',
      text: Array.isArray(error.response.data.message)
        ? error.response.data.message[0]
        : error.response.data.message
    })
  }
}
</script>

<template>
  <div id="loginPage" class="bg-yellow">
    <div class="container loginPage vhContainer">
      <div class="side">
        <a href="#"
          ><img
            class="logoImg"
            src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png"
            alt=""
        /></a>
        <img
          class="d-m-n"
          src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png"
          alt="workImg"
        />
      </div>
      <div>
        <div>
          <form class="formControls" action="index.html">
            <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>
            <label class="formControls_label" for="email_signIn">Email</label>
            <input
              class="formControls_input"
              type="text"
              id="email_signIn"
              name="email_signIn"
              placeholder="請輸入 email"
              required
              v-model="signInData.email"
            />
            <span class="d-none">此欄位不可留空</span>
            <label class="formControls_label" for="pwd_signIn">密碼</label>
            <input
              class="formControls_input"
              type="password"
              name="pwd_signIn"
              id="pwd_signIn"
              placeholder="請輸入密碼"
              required
              v-model="signInData.password"
            />
            <input
              class="formControls_btnSubmit"
              type="button"
              value="登入"
              @click.prevent="signIn"
            />
            <RouterLink to="/signUp" class="formControls_btnLink">註冊帳號</RouterLink>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.d-none {
  display: none;
}
</style>
