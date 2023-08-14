<script>
import $ from 'jquery'

export default {
  data() {
    return {
      // 整個List
      todoList: [],
      // 呈現在畫面的List
      showingList: [],
      // 新增、修改單一Todo時存放的地方
      tempTodo: {
        content: '',
        date: '',
        time: '',
        importance: '',
        id: 0,
        isDone: false,
        isNew: true
      },
      // 用於下拉選單的Filter
      currentFilterType: 'all'
    }
  },
  computed: {
    todoListLength() {
      let isDoneArray = []
      this.todoList.forEach((item) => {
        if (!item.isDone) {
          isDoneArray.push(item);
        }
      })
      return isDoneArray.length;
    }
  },
  methods: {
    // 根據傳入的element開啟特定的Modal
    openModal(element) {
      $(element).fadeIn(300);
      $(element).css('display', 'flex');
    },
    // 根據傳入的element關閉特定的Modal
    closeModal(element) {
      $(element).fadeOut(300);
    },
    // 確認新增 OR 確認修改的函數
    confirmModal() {
      const timestamp = new Date().getTime();
      // 如果是新的Todo才會push進todoList
      if (this.tempTodo.isNew) {
        this.tempTodo.id = timestamp;
        this.todoList.push(this.tempTodo);
        this.tempTodo.isNew = false;
      }
      // 初始化暫存的todo
      this.tempTodo = {
        content: '',
        date: '',
        time: '',
        importance: '',
        id: 0,
        isDone: false,
        isNew: true
      };
      this.filterByType(this.currentFilterType);
      this.closeModal(".modalContainer");
    },
    // 透過id filter出特定的todo並放入暫存的todo
    editTodoitem(id) {
      this.todoList.filter((item) => {
        if (item.id === id) {
          this.tempTodo = item
        }
      })

      this.openModal(".modalContainer");
    },
    deleteTodoitem(index) {
      this.todoList.splice(index, 1);
    },
    // 在todoList有東西的情況才可以開啟alert
    deleteAllAlert() {
      if (this.todoList[0]) {
        this.openModal(".alertContainer");
      }
    },
    // 清空List
    deleteAll() {
      this.todoList = [];
      this.showingList = [];
      this.closeModal(".alertContainer");
    },
    // 根據currentFilterType 決定showingList的內容
    filterByType(value) {
      if (value === 'processing') {
        this.todoList.forEach((item) => {
          if (!item.isDone) {
            this.showingList.push(item);
          }
        })
      } else if (value === 'finish') {
        this.todoList.forEach((item) => {
          if (item.isDone) {
            this.showingList.push(item);
          }
        })
      } else {
        this.showingList = this.todoList
      }
    }
  },
  watch: {
    currentFilterType: {
      handler(value) {
        this.showingList = [];
        this.filterByType(value);
      },
      immediate: true
    },
    // 當todoList有任何變動都上傳到localStorage
    todoList: {
      handler() {
        localStorage.setItem('todoList', JSON.stringify(this.todoList));
      },
      deep: true
    }
  },
  mounted() {
    if (JSON.parse(localStorage.getItem('todoList')) == null) {
      this.todoList = [];
      this.showingList = [];
    } else {
      this.todoList = JSON.parse(localStorage.getItem('todoList'));
      this.showingList = JSON.parse(localStorage.getItem('todoList'));
    }
  }
}
</script>

<template>
  <div class="container">
    <h1 class="title">Peter's Todo List</h1>

    <div class="tableContainer">
      <div class="firstbar">
        <select id="" v-model="currentFilterType">
          <option value="all" selected>全部</option>
          <option value="processing">處理中</option>
          <option value="finish">已完成</option>
        </select>
        <button class="addBtn" @click="openModal('.modalContainer')"><img src="@/assets/img/icon/plus.png" alt="">
          新增
        </button>
      </div>

      <table>
        <thead>
          <th>
            <p>狀態</p>
          </th>
          <th>
            <p>代辦事項</p>
          </th>
          <th>
            <p>時間</p>
          </th>
          <th>
            <p>重要性</p>
          </th>
          <th>
            <p>編輯</p>
          </th>
          <th>
            <p>刪除</p>
          </th>
        </thead>

        <tbody>
          <tr v-for="(item, index) in showingList" :key="item.id">
            <td>
              <label :for="item.id">
                <div>
                  <img src="@/assets/img/icon/check.png" alt="" v-if="item.isDone">
                </div>
              </label>
              <input type="checkbox" :id="item.id" v-model="item.isDone">
            </td>
            <td>
              <p>{{ item.content }}</p>
            </td>
            <td>
              <p>{{ item.date }}</p>
              <p>{{ item.time }}</p>
            </td>
            <td>
              <p v-if="item.importance === 'low'" class="low">低</p>
              <p v-else-if="item.importance === 'middle'" class="middle">中</p>
              <p v-else-if="item.importance === 'high'" class="high">高</p>
            </td>
            <td>
              <button type="button" class="editBtn">
                <img src="@/assets/img/icon/pen.png" alt="" @click="editTodoitem(item.id)">
              </button>
            </td>
            <td>
              <button type="button" class="deleteBtn" @click="deleteTodoitem(index)">
                <img src="@/assets/img/icon/trashcan.png" alt="">
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      <div class="lastbar">
        <p>還有{{ todoListLength }}項任務尚未完成</p>
        <button type="button" @click="deleteAllAlert">清除所有任務</button>
      </div>
    </div>


    <div class="modalContainer" @keyup.enter="confirmModal">
      <div>
        <h2>代辦事項</h2>
        <img src="@/assets/img/icon/cancel.png" alt="" @click="closeModal('.modalContainer')">
        <div>
          <label for="content">代辦事項</label>
          <input type="text" id="content" v-model="tempTodo.content">
          <label for="date">日期</label>
          <input type="date" id="date" v-model="tempTodo.date"> <!--2023-08-21 -->
          <label for="time">時間</label>
          <input type="time" id="time" v-model="tempTodo.time"> <!--20:21 -->
          <label for="importance">重要性</label>
          <select name="" id="importance" v-model="tempTodo.importance">
            <option value="low">低</option>
            <option value="middle">中</option>
            <option value="high">高</option>
          </select>
          <div>
            <button type="button" class="deleteBtn" @click="closeModal('.modalContainer')">取消</button>
            <button type="button" class="confirmBtn" @click="confirmModal">確認</button>
          </div>
        </div>
      </div>
    </div>

    <div class="alertContainer">
      <div>
        <h2>警告</h2>
        <img src="@/assets/img/icon/cancel.png" alt="" @click="closeModal('.alertContainer')">
        <h2>您確定要全部刪除嗎?</h2>
        <div>
          <button type="button" class="deleteBtn" @click="closeModal('.alertContainer')">取消</button>
          <button type="button" class="confirmBtn" @click="deleteAll">確認</button>
        </div>
      </div>
    </div>

  </div>
</template>
