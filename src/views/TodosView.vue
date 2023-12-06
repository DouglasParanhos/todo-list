<template>
  <main>
    <h1>Create Todo</h1>
    <TodoCreator @create-todo="createTodo">
      <template #button-content>Create</template>
    </TodoCreator>
    <draggable v-if="todoList.length > 0" v-model="todoList" :item-key="uid()" :animation="300" tag="ul" class="todo-list"> 
      <template #item="{ element: todo }">
        <TodoItem
          :todo="todo"
          @edit-todo="toggleEditTodo"
          @update-todo="updateTodo"
          @toggle-complete="toggleTodoComplete"
          @delete-todo="deleteTodo"
        />
      </template>
    </draggable>
    <ul v-if="todoList.length > 0" class="todo-list">
        
    </ul>
    <p v-else class="todos-msg">
      <Icon icon="noto-v1:sad-but-relieved-face" />
      <span>You have no todo's to complete! Add one!</span>
    </p>
    <p v-if="todosCompleted && todoList.length > 0" class="todos-msg">
      <Icon icon="noto-v1:party-popper" />
      <span>You have completed all your todos!</span>
    </p>
  </main>
</template>

<script setup>
import { Icon } from "@iconify/vue";
import { uid } from "uid";
import { ref, computed, watch } from "vue";
import draggable from 'vuedraggable';
import TodoCreator from "../components/TodoCreator.vue";
import TodoItem from "../components/TodoItem.vue";

const todoList = ref([]);

const todosCompleted = computed(() => {
  return todoList.value.every((todo) => todo.isCompleted);
});

const fetchTodoList = () => {
  const savedTodoList = JSON.parse(localStorage.getItem("todoList"));
  if (savedTodoList) {
    todoList.value = savedTodoList;
    todoList.value.map(todo => todo.isEditing = false)
  }
};

// Fetch Todo's on page load
fetchTodoList();

const setTodoListLocalStorage = () => {
  localStorage.setItem("todoList", JSON.stringify(todoList.value));
};

const createTodo = (todo) => {
  todoList.value.push({
    id: uid(),
    todo,
    isCompleted: false,
    isEditing: null,
  });
};

const toggleEditTodo = (todo) => {
  const todoEdited = todoList.value.filter((t) => t.id === todo.id)[0]
  
  todoEdited.isEditing = !todoEdited.isEditing;
};

const updateTodo = (todoVal, todo) => {
  const todoUpdated = todoList.value.filter((t) => t.id === todo.id)[0]
  todoUpdated.todo = todoVal
};

const toggleTodoComplete = (todo) => {
  const todoCompleted = todoList.value.filter((t) => t.id === todo.id)[0]
  todoCompleted.isCompleted = !todoCompleted.isCompleted;
};

const deleteTodo = (todo) => {
  todoList.value = todoList.value.filter(
    (todoFilter) => todoFilter.id !== todo.id
  );
};

watch(todoList, () => {
    setTodoListLocalStorage();
  }, 
  { deep: true }
  )
</script>

<style scoped lang="scss">
main {
  display: flex;
  flex-direction: column;
  max-width: 500px;
  width: 100%;
  margin: 0 auto;
  padding: 40px 16px;

  h1 {
    margin-bottom: 16px;
    text-align: center;
  }

  .todo-list {
    display: flex;
    flex-direction: column;
    list-style: none;
    margin-top: 24px;
    gap: 20px;
  }

  .todos-msg {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    margin-top: 24px;
  }
}
</style>