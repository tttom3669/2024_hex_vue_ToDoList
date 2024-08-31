<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'
import Swal from 'sweetalert2'
import { useRouter, RouterLink } from 'vue-router'

const router = useRouter()
const apiPath = 'https://todolist-api.hexschool.io'

const token = getToken()
const apiKeyAuth = {
  headers: {
    Authorization: token
  }
}
const userName = ref('')
const tempTodo = ref('')
const tempTodoListData = ref([])
const todoListType = ref('all')
const toDoListCount = ref(0)
const filterToDoData = ref([])

/**
 * swal 提示訊息
 * @param param0
 */
function swalMessage({ type, text }) {
  Swal.fire({
    title: '訊息',
    text: text,
    icon: type,
    confirmButtonText: '關閉'
  })
}

/**
 * 取得 token
 */
function getToken() {
  return document.cookie.replace(
    new RegExp('(?:(?:^|.*;\\s*)todoListToken\\s*=\\s*([^;]*).*$)|^.*$'),
    '$1'
  )
}

/**
 * 登入驗證
 */
const checkOut = async () => {
  try {
    const response = await axios.get(`${apiPath}/users/checkout`, apiKeyAuth)
    userName.value = response.data.nickname
  } catch (error) {
    swalMessage({
      type: 'error',
      text: '登入驗證失效'
    })
    router.push('/')
  }
}

/**
 * 登出
 */
const signOut = async () => {
  try {
    const response = await axios.post(
      `${apiPath}/users/sign_out`,
      {},
      {
        headers: {
          Authorization: token
        }
      }
    )
    swalMessage({
      type: 'success',
      text: response.data.message
    })
    router.push('/')
  } catch (error) {
    swalMessage({
      type: 'error',
      text: '登出失敗'
    })
    router.push('/')
  }
}

/**
 * 新增待辦
 */
const addToDo = async () => {
  try {
    await axios.post(
      `${apiPath}/todos/`,
      {
        content: tempTodo.value
      },
      apiKeyAuth
    )
    tempTodo.value = ''
    getToDo(todoListType.value)
  } catch (error) {
    console.log(error)
  }
}

/**
 * 取得待辦
 */
async function getToDo(status = 'all') {
  try {
    const response = await axios.get(`${apiPath}/todos/`, apiKeyAuth)
    tempTodoListData.value = [...response.data.data]
    switchToDoStatus(status)
  } catch (error) {
    console.log(error)
  }
}

/**
 * 待辦事項切換完成狀態
 * @param 待辦事項id
 */
const toggleToDo = async (id) => {
  try {
    await axios.patch(`${apiPath}/todos/${id}/toggle`, {}, apiKeyAuth)
    getToDo(todoListType.value)
  } catch (error) {
    console.log(error)
  }
}

/**
 * 刪除待辦事項
 * @param 待辦事項id
 */
const deleteToDo = async (id) => {
  try {
    await axios.delete(`${apiPath}/todos/${id}`, apiKeyAuth)
    getToDo(todoListType.value)
  } catch (error) {
    console.log(error)
  }
}

/**
 * 切換待辦事項清單狀態
 * @param status
 */
const switchToDoStatus = (status) => {
  todoListType.value = status

  switch (todoListType.value) {
    case 'checked':
      filterToDoData.value = tempTodoListData.value.filter((todo) => todo.status)
      toDoListCount.value = filterToDoData.value.length
      break
    case 'unchecked':
      filterToDoData.value = tempTodoListData.value.filter((todo) => !todo.status)
      toDoListCount.value = filterToDoData.value.length
      break
    case 'all':
    default:
      filterToDoData.value = [...tempTodoListData.value]
      toDoListCount.value = tempTodoListData.value.filter((todo) => !todo.status).length
      break
  }
}

onMounted(() => {
  checkOut()
  getToDo(todoListType.value)
})
</script>

<template>
  <!-- ToDo List -->
  <div id="todoListPage" class="bg-half">
    <nav>
      <h1><a href="#">ONLINE TODO LIST</a></h1>
      <ul>
        <li class="todo_sm">
          <RouterLink to="/todoList" class="formControls_btnLink"
            ><span>{{ userName }}的代辦</span></RouterLink
          >
        </li>
        <li><a href="#loginPage" @click.prevent="signOut">登出</a></li>
      </ul>
    </nav>
    <div class="container todoListPage vhContainer">
      <div class="todoList_Content">
        <div class="inputBox">
          <input type="text" placeholder="請輸入待辦事項" v-model="tempTodo" />
          <a href="#" @click.prevent="addToDo">
            <i class="fa fa-plus"></i>
          </a>
        </div>
        <div class="todoList_list">
          <ul class="todoList_tab">
            <li>
              <a
                href="#"
                :class="todoListType === 'all' ? 'active' : ''"
                @click.prevent="switchToDoStatus('all')"
                >全部</a
              >
            </li>
            <li>
              <a
                href="#"
                :class="todoListType === 'unchecked' ? 'active' : ''"
                @click.prevent="switchToDoStatus('unchecked')"
                >待完成</a
              >
            </li>
            <li>
              <a
                href="#"
                :class="todoListType === 'checked' ? 'active' : ''"
                @click.prevent="switchToDoStatus('checked')"
                >已完成</a
              >
            </li>
          </ul>
          <div class="todoList_items">
            <ul class="todoList_item" v-if="tempTodoListData.length">
              <li v-for="todo in filterToDoData" :key="todo.id">
                <label class="todoList_label">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    :checked="todo.status"
                    @click="toggleToDo(todo.id)"
                  />
                  <span>{{ todo.content }}</span>
                </label>
                <a href="#" @click.prevent="deleteToDo(todo.id)">
                  <i class="fa fa-times"></i>
                </a>
              </li>
            </ul>
            <p class="todoList_empty" v-else>目前尚無待辦事項</p>
            <div class="todoList_statistics">
              <p>{{ toDoListCount }} 個{{ todoListType === 'checked' ? '已' : '待' }}完成項目</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
