<template>
  <div class="todo-container">
    <h1 class="title">Todo List</h1>
    <button @click="showAddModal = true" class="add-btn">+ Add Todo</button>

    <table class="todo-table">
      <thead>
        <tr>
          <th>No</th>
          <th>Todo</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(todo, index) in todos" :key="todo.id">
          <td>{{ index + 1 }}</td>
          <td>{{ todo.title }}</td>
          <td>
            <button class="action-btn edit-btn" @click="openEditModal(todo)">✏️ Edit</button>
            <button class="action-btn delete-btn" @click="deleteTodo(todo.id)">🗑️ Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <button class="load-more-btn" @click="loadMore">Load More</button>

    <!-- Add Todo Modal -->
    <div v-if="showAddModal" class="modal-overlay" @click.self="closeAddModal">
      <div class="modal-content">
        <h2>Add Todo</h2>
        <TodoForm :todo="newTodo" :buttonText="'Add Todo'" @submit="addTodo" />
        <button class="close-btn" @click="closeAddModal">Close</button>
      </div>
    </div>

    <!-- Edit Todo Modal -->
    <div v-if="showEditModal" class="modal-overlay" @click.self="closeEditModal">
      <div class="modal-content">
        <h2>Edit Todo</h2>
        <TodoForm :todo="newTodo" :buttonText="'Update Todo'" @submit="updateTodo" />
        <button class="close-btn" @click="closeEditModal">Close</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import TodoForm from './TodoForm.vue';

export default {
  name: 'TodoList',
  components: {
    TodoForm
  },
  data() {
    return {
      todos: [],
      newTodo: { title: '' },
      page: 1,
      limit: 10,
      editIndex: -1,
      showAddModal: false,
      showEditModal: false,
    };
  },
  methods: {
    fetchTodos() {
      axios.get(`https://jsonplaceholder.typicode.com/todos?_page=${this.page}&_limit=${this.limit}`)
        .then(response => {
          this.todos = [...this.todos, ...response.data];
        });
    },
    addTodo(todo) {
      if (!todo || !todo.title || !todo.title.trim()) {
        return;
      }
      axios.post('https://jsonplaceholder.typicode.com/todos', todo)
        .then(response => {
          this.todos.unshift(response.data);
          this.newTodo = { title: '' };
          this.showAddModal = false; 
        })
        .catch(error => {
          console.error('Error during creation:', error.response || error.message);
          alert('Failed to create todo. Please try again.');
        });
    },
    openEditModal(todo) {
      this.newTodo = { ...todo };
      this.editIndex = this.todos.indexOf(todo);
      this.showEditModal = true;
    },
    updateTodo(todo) {
      if (!todo || !todo.id || !todo.title.trim()) {
        return;
      }

      axios.delete(`https://jsonplaceholder.typicode.com/todos/${todo.id}`)
        .then(() => {
          return axios.post('https://jsonplaceholder.typicode.com/todos', todo);
        })
        .then(response => {
          this.todos.splice(this.editIndex, 1, response.data);
          this.newTodo = { title: '' };
          this.editIndex = -1;
          this.showEditModal = false; 
        })
        .catch(error => {
          console.error('Error during update:', error.response || error.message);
          alert('Failed to update todo. Please try again.');
        });
    },
    deleteTodo(id) {
      axios.delete(`https://jsonplaceholder.typicode.com/todos/${id}`)
        .then(() => {
          this.todos = this.todos.filter(todo => todo.id !== id);
        })
        .catch(error => {
          console.error('Error during deletion:', error.response || error.message);
          alert('Failed to delete todo. Please try again.');
        });
    },
    loadMore() {
      this.page++;
      this.fetchTodos();
    },
    closeAddModal() {
      this.showAddModal = false;
      this.newTodo = { title: '' }; 
    },
    closeEditModal() {
      this.showEditModal = false;
      this.newTodo = { title: '' }; 
      this.editIndex = -1;
    }
  },
  mounted() {
    this.fetchTodos();
  }
};
</script>

<style scoped>
.todo-container {
  width: 90%;
  max-width: 1200px;
  margin: 40px auto;
  background-color: #ffffff;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2.5em;
  color: #333;
  margin-bottom: 30px;
}

.todo-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 30px;
}

.todo-table th, .todo-table td {
  padding: 15px;
  border: 1px solid #e0e0e0;
  text-align: center;
}

.todo-table th {
  background-color: #f9f9f9;
  color: #555;
  font-weight: 600;
}

.todo-table td {
  background-color: #fff;
  color: #666;
}

.add-btn {
  display: block;
  width: 180px;
  margin: 0 auto 20px;
  padding: 12px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1.1em;
  text-transform: uppercase;
  transition: background-color 0.3s ease;
}

.add-btn:hover {
  background-color: #45a049;
}

.action-btn {
  padding: 8px 14px;
  margin: 0 4px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9em;
}

.edit-btn {
  background-color: #fbc02d;
  color: white;
}

.delete-btn {
  background-color: #d32f2f;
  color: white;
}

.edit-btn:hover {
  background-color: #f9a825;
}

.delete-btn:hover {
  background-color: #c62828;
}

.load-more-btn {
  display: block;
  width: 100%;
  padding: 14px;
  background-color: #1976D2;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1.1em;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.load-more-btn:hover {
  background-color: #1565C0;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 30px;
  border-radius: 12px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.close-btn {
  background-color: #d32f2f;
  color: white;
  margin-top: 20px;
  padding: 10px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1em;
  width: 100%;
  transition: background-color 0.3s ease;
}

.close-btn:hover {
  background-color: #c62828;
}
</style>
