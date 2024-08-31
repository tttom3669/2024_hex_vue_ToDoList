<script setup>
import axios from 'axios'
import { ref } from 'vue'
import { RouterLink, useRouter } from 'vue-router'
import Swal from 'sweetalert2'

const signUpData = ref({})
const apiPath = 'https://todolist-api.hexschool.io'
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
 * 註冊
 */
const signUp = async () => {
  try {
    // 再次輸入密碼的錯誤判斷
    if (signUpData.value.password && signUpData.value.password.length >= 6) {
      if (!signUpData.value.password_checked) {
        swalMessage({ type: 'error', text: '請再次輸入密碼' })
        return
      }
      if (signUpData.value.password_checked !== signUpData.value.password) {
        swalMessage({ type: 'error', text: '輸入密碼不同' })
        return
      }
    }

    await axios.post(`${apiPath}/users/sign_up`, {
      email: signUpData.value.email,
      password: signUpData.value.password,
      nickname: signUpData.value.nickname
    })

    swalMessage({ type: 'success', text: '註冊成功' })
    router.push('/')
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
  <div id="singUpPage" class="bg-yellow">
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
            <h2 class="formControls_txt">註冊帳號</h2>
            <label class="formControls_label" for="email">Email</label>
            <input
              class="formControls_input"
              type="text"
              id="email"
              name="email"
              placeholder="請輸入 email"
              required
              v-model="signUpData.email"
            />
            <label class="formControls_label" for="name">您的暱稱</label>
            <input
              class="formControls_input"
              type="text"
              name="name"
              id="name"
              placeholder="請輸入您的暱稱"
              v-model="signUpData.nickname"
            />
            <label class="formControls_label" for="pwd">密碼</label>
            <input
              class="formControls_input"
              type="password"
              name="pwd"
              id="pwd"
              placeholder="請輸入密碼"
              required
              v-model="signUpData.password"
            />
            <label class="formControls_label" for="pwd_checked">再次輸入密碼</label>
            <input
              class="formControls_input"
              type="password"
              name="pwd_checked"
              id="pwd_checked"
              placeholder="請再次輸入密碼"
              required
              v-model="signUpData.password_checked"
            />
            <input
              class="formControls_btnSubmit"
              type="button"
              @click.prevent="signUp"
              value="註冊帳號"
            />
            <!-- <a class="formControls_btnLink" href="#loginPage"></a> -->
            <RouterLink to="/" class="formControls_btnLink">登入</RouterLink>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style></style>
