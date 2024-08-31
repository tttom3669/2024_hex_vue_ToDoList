<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'
import Swal from 'sweetalert2'
import { useRouter, RouterLink } from 'vue-router'

const router = useRouter()
const apiPath = 'https://todolist-api.hexschool.io'

const swalMessage = ({ type, text }) => {
  Swal.fire({
    title: '訊息',
    text: text,
    icon: type,
    confirmButtonText: '關閉'
  })
}
const token = getToken()
const apiKeyAuth = {
  headers: {
    Authorization: token
  }
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
    await axios.get(`${apiPath}/users/checkout`, apiKeyAuth)
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

const tempTodo = ref('')
/**
 * 新增待辦
 */
const addToDo = async () => {
  try {
    const response = await axios.post(
      `${apiPath}/todos/`,
      {
        content: tempTodo.value
      },
      apiKeyAuth
    )
    console.log(response)
    getToDo()
  } catch (error) {
    console.log(error)
  }
}

let tempTodoListData = ref([])
let todoListType = ref('')
let toDoListCount = ref(0)
let filterToDoData = ref([])
/**
 * 取得待辦
 */
async function getToDo(status = 'all') {
  try {
    const response = await axios.get(`${apiPath}/todos/`, apiKeyAuth)
    tempTodoListData.value = [...response.data.data]
    // checkedToDoCount.value = tempTodoListData.value.filter((todo) => !todo.status).length
    switchToDoStatus(status)

    console.log(response)
  } catch (error) {
    console.log(error)
  }
}

const toggleToDo = async (id) => {
  try {
    const response = await axios.patch(`${apiPath}/todos/${id}/toggle`, {}, apiKeyAuth)
    console.log(response)
    getToDo()
  } catch (error) {
    console.log(error)
  }
}

const deleteToDo = async (id) => {
  try {
    const response = await axios.delete(`${apiPath}/todos/${id}`, apiKeyAuth)
    console.log(response)
    getToDo()
  } catch (error) {
    console.log(error)
  }
}

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

// function changeToDoListCount(){
//   switch(todoListType){
//     case 'checked':

//     break
//     case 'unchecked':
//     case 'all':
//     break
//   }
// }

onMounted(() => {
  checkOut()
  getToDo()
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
            ><span>王小明的代辦</span></RouterLink
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
