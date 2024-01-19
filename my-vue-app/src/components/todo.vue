
<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

//declarations for creating a todo
const taskname = ref('');
const taskcomplete = ref(false);

//declarations for filtering todos
const taskcomplete1 = ref(false);
const taskcomplete2 = ref(false);
const searchQuery = ref('');
const searchButtonClicked = ref(false);

//declarations for sorting alphabetical 
const sortBy = ref('');

//declaring todo
const todos = ref<Array<{ _id: string;
  todoName: string;
  isComplete: boolean;
   }>>([]);

// fetching data on onMounted state
   onMounted(async () => {
  await fetchTodos(); 
  });

  //fetching data
const fetchTodos = async () => {
  try {
    const response = await fetch('https://calm-plum-jaguar-tutu.cyclic.app/todos');
    const data = await response.json();
    todos.value = data.data;
  } catch (error) {
    console.error('Error fetching todos', error);
  }
};

//creating a todo
const createtodo = async () => {
  try {
      await axios.post('https://calm-plum-jaguar-tutu.cyclic.app/todos', {
      todoName: taskname.value,
      isComplete: taskcomplete.value,
    });
    await fetchTodos();
    taskname.value = '';
    taskcomplete.value = false;
  } catch (error) {
    console.error('Error while creating', error);
  }
}
//editing a todo
const editTodo = async (todoId: string) => {
  try {
    const todo = todos.value.find((t) => t._id === todoId);
    if (todo) {
      todo.isComplete = !todo.isComplete;
      await axios.put(`https://calm-plum-jaguar-tutu.cyclic.app/todos/${todoId}`, {
        isComplete: todo.isComplete,
    });
  }
  } catch (error) {
    console.error('Error editing a post', error);
  }
}

//Deleting a todo
const deleteTodo = async (todoId: string) => {
  try {
    await axios.delete(`https://calm-plum-jaguar-tutu.cyclic.app/todos/${todoId}`);
    todos.value = todos.value.filter((todo) => todo._id !== todoId);

    console.log('Todo --> deleted');
  } catch (error) {
    console.error('Error --> deleting', error);
  }
};

// Filtering and Sorting todos
const filteredAndSortedTodos = computed(() => {
  const filteredTodos = todos.value.filter((todo) => {
    const condition1 = !taskcomplete1.value || todo.isComplete;
    const condition2 = !taskcomplete2.value || !todo.isComplete;

    // both checkboxes are checked, it shows all the todos
    if (taskcomplete1.value && taskcomplete2.value) {
      return true;
    }

    // Check search query
    const matchesSearch = todo.todoName.toLowerCase().includes(searchQuery.value.toLowerCase());

    return condition1 && condition2 && matchesSearch;
  });

  // Sort todos based on the sorting order
  if (sortBy.value === 'alphabetical') {
    return [...filteredTodos].sort((a, b) => a.todoName.localeCompare(b.todoName));
  } else if (sortBy.value === 'reverseAlphabetical') {
    return [...filteredTodos].sort((a, b) => b.todoName.localeCompare(a.todoName));
  } else {
    return filteredTodos; 
  }
});
const sortTodos = () => {
  sortBy.value = sortBy.value === 'alphabetical' ? 'reverseAlphabetical' : 'alphabetical';
};

</script>
<template>
   <div>
    <h1>Todos</h1>
    <div class="form-section">
      <div class="form-row">
        <div class="checkbox-group">
          <label>
            Show Completed:
            <input type="checkbox" v-model="taskcomplete1" />
          </label>
          <label>
            Show Incomplete:
            <input type="checkbox" v-model="taskcomplete2" />
          </label>
        </div>
        <div class="search-group">
          <input v-model="searchQuery" placeholder="Search Todos" />
          <button @click="searchButtonClicked = true" class="search-button">Search</button>
        </div>
      </div>
    </div>
      <table class="todo-table">
        <thead>
          <tr>
            <th @click="sortTodos">Name (sort)</th>
            <th>Complete</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="todo in filteredAndSortedTodos" :key="todo._id">
            <td>{{ todo.todoName }}</td>
            <td>{{ todo.isComplete ? 'Yes' : 'No' }}</td>
            <td>
              <button @click="editTodo(todo._id)">Edit</button>
              <button @click="deleteTodo(todo._id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
      <div class="input-group">
          <label>
            Task Name:
            <input v-model="taskname" id="taskname" required>
          </label>
          <label>
            Task completed?
            <input v-model="taskcomplete" id="taskcomplete" required>
          </label>
          <button @click="createtodo" class="create-todo-button">Create a Todo</button>
        </div>
    </div>
</template>
<style>
.form-section {
  margin-bottom: 20px;
}

.form-row {
  display: flex;
  justify-content: space-between;
}

.input-group{
  border: 2px solid #4caf50; 
  padding: 20px;
  border-radius: 8px; 
}

.checkbox-group,
.search-group {
  flex: 1;
}
.checkbox-group {
  display: flex;
  flex-direction: column;
}
.todo-table {
  width: 100%;
  border-collapse: collapse;
}
.todo-table th, .todo-table td {
  border: 2px solid #0d0808;
  padding: 20px;
  text-align: left;
}
.todo-table th {
  background-color: #c5b5b5;
}
.create-todo-button {
  background-color: #4b7c4c;
  color: white;
  border: 1px solid #4caf50;
  padding: 10px 15px;
  cursor: pointer;
  transition: background-color 0.3s;
}
.search-button{
  background-color: #4b7c4c;
  color: white;
  border: 1px solid #4caf50;
  padding: 10px 15px;
  cursor: pointer;
  transition: background-color 0.3s;
}
</style>