<template>
  <div>
    <!-- 新增 input -->
    <input
      type="text"
      v-model="newTodo"
      @keyup.enter="addTodo"
      autofoucs
      placeholder="新增今日待办"
      autocomplete="off"
    />

    <!-- todo 列表 -->
    <ul>
      <li
        v-for="todo in filterTodos"
        :key="todo.id"
        :class="{ completed: todo.completed, editing: todo === editedTodo }"
      >
        <!-- 绑定完成状态 -->
        <div class="view">
          <input type="checkbox" v-model="todo.completed" />
          <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
          <button @click="removeTodo(todo)">X</button>
        </div>

        <!-- 编辑待办 -->
        <input
          type="text"
          class="edit"
          v-model="todo.title"
          v-inputFocus="todo === editedTodo"
          @blur="doneEdit(todo)"
          @keyup.enter="doneEdit(todo)"
          @keyup.esc="cancelEdit(todo)"
        />
      </li>
    </ul>

    <!-- 状态栏 -->
    <div class="filters">
      <span
        @click="visibility = 'all'"
        :class="{ selected: visibility === 'all' }"
        >All</span
      >
      <span
        @click="visibility = 'active'"
        :class="{ selected: visibility === 'active' }"
        >Active</span
      >
      <span
        @click="visibility = 'completed'"
        :class="{ selected: visibility === 'completed' }"
        >Completed</span
      >
    </div>
  </div>
</template>

<script>
import { computed, reactive, toRefs, watchEffect } from 'vue'

const filters = {
  all(todos) {
    return todos
  },
  active(todos) {
    return todos.filter((todo) => !todo.completed)
  },
  completed(todos) {
    return todos.filter((todo) => todo.completed)
  },
}

const todoStorage = {
  fetch() {
    let todos = JSON.parse(localStorage.getItem('todos') || '[]')
    todos.forEach((todo, index) => {
      todo.id = index + 1
    })
    return todos
  },
  save(todos) {
    localStorage.setItem('todos', JSON.stringify(todos))
  },
}

export default {
  setup() {
    const state = reactive({
      newTodo: '',
      todos: todoStorage.fetch(),
      beforeEditCache: '', // 缓存编辑之前的 title
      editedTodo: null,
      visibility: 'all',
      filterTodos: computed(() => {
        return filters[state.visibility](state.todos)
      }),
    })

    function addTodo() {
      state.todos.push({
        id: state.todos.length + 1,
        title: state.newTodo,
        completed: false,
      })
      state.newTodo = ''
    }

    function removeTodo(todo) {
      state.todos.splice(state.todos.indexOf(todo), 1)
    }

    function editTodo(todo) {
      state.beforeEditCache = todo.title
      state.editedTodo = todo
    }

    function cancelEdit(todo) {
      todo.title = state.beforeEditCache
      state.editedTodo = null
    }

    function doneEdit(todo) {
      state.editedTodo = null
    }

    watchEffect(() => {
      todoStorage.save(state.todos)
    })

    return {
      ...toRefs(state),
      addTodo,
      removeTodo,
      editTodo,
      cancelEdit,
      doneEdit,
    }
  },
  directives: {
    inputFocus: (el, { value }) => {
      if (value) {
        el.focus()
      }
    },
  },
}
</script>

<style scoped>
.completed label {
  text-decoration: line-through;
}

li {
  list-style: none;
}

.edit,
.editing .view {
  display: none;
}

.view,
.editing .edit {
  display: block;
}

.filters > span {
  padding: 2px 4px;
  margin-right: 4px;
  border: 1px solid transparent;
}

.filters > span.selected {
  border-color: #ccc;
}
</style>