<template>
  <v-container :style="{ backgroundColor: darkMode ? '#1E1E2F' : '#F7F7F7' }" fluid>
    <v-card class="pa-4 my-card" :color="darkMode ? '#2C2E43' : '#FAF9F9'" elevation="5">
      <v-card-title class="justify-space-between">
        <div class="title" :style="{ color: darkMode ? '#FFBE0B' : '#2C2E43' }">My To-Do List</div>
        <v-switch v-model="darkMode" label="Dark Mode" class="ml-auto"></v-switch>
      </v-card-title>

      <v-card-text>
        <v-row>
          <v-col cols="12" sm="8">
            <v-text-field
              v-model="newTodo.text"
              @keyup.enter="addTodo"
              label="Add a new task"
              outlined
              dense
              :color="darkMode ? '#FFBE0B' : '#3A86FF'"
              class="task-input"
            ></v-text-field>
          </v-col>
          <v-col cols="12" sm="4">
            <v-select
              v-model="newTodo.priority"
              :items="['Least', 'Normal', 'Most']"
              label="Priority"
              outlined
              dense
              :color="darkMode ? '#FF006E' : '#3A86FF'"
            ></v-select>
          </v-col>
          <v-col cols="12" class="text-right mt-2">
            <v-btn @click="addTodo" :color="darkMode ? '#FF006E' : '#3A86FF'" elevation="2" class="add-task-btn">Add Task</v-btn>
          </v-col>
        </v-row>
      </v-card-text>

      <v-row>
        <v-col cols="12" class="d-flex justify-end mb-3">
          <v-select
            v-model="filter"
            :items="['All', 'Completed', 'Pending']"
            label="Filter tasks"
            outlined
            dense
            :color="darkMode ? '#FFBE0B' : '#3A86FF'"
            class="mx-3"
            style="max-width: 150px;"
          ></v-select>
        </v-col>

        <v-col cols="12">
          <v-progress-linear
            :value="(completedTasks / todos.length) * 100"
            height="8"
            :color="progressBarColor"
            background-color="grey lighten-2"
            v-if="todos.length"
          ></v-progress-linear>
        </v-col>

        <v-col cols="12">
          <div class="cards-background">
            <v-list>
              <v-list-item-group v-if="filteredTodos.length">
                <v-row no-gutters>
                  <v-col
                    v-for="todo in filteredTodos"
                    :key="todo.id"
                    cols="12" sm="6" md="4" lg="3"
                    class="d-flex justify-center mb-4 mx-2"
                    style="overflow: hidden;"
                  >
                    <v-card
                      :color="darkMode ? '#3B3D58' : '#FFFFFF'"
                      elevation="4"
                      class="task-card pa-2"
                      style="width: 100%;"
                    >
                      <v-list-item-content>
                        <v-list-item-title
                          :class="{ completed: todo.completed }"
                          :style="getPriorityStyle(todo.priority)"
                        >
                          {{ todo.text }}
                        </v-list-item-title>
                        <v-list-item-subtitle>{{ todo.priority }} Priority</v-list-item-subtitle>
                      </v-list-item-content>

                      <v-card-actions class="d-flex justify-between">
                        <v-btn icon @click="editTodo(todo)">
                          <v-icon color="primary">mdi-pencil</v-icon>
                        </v-btn>
                        <v-btn icon @click="confirmDelete(todo)">
                          <v-icon color="red">mdi-delete</v-icon>
                        </v-btn>
                        <v-btn icon @click="toggleCompletion(todo)">
                          <v-icon :color="todo.completed ? 'green' : 'grey'">
                            mdi-check-circle
                          </v-icon>
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-col>
                </v-row>
              </v-list-item-group>
              <v-list-item v-else>No tasks available</v-list-item>
            </v-list>
          </div>
        </v-col>
      </v-row>
    </v-card>

    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>Edit Task</v-card-title>
        <v-card-text>
          <v-text-field v-model="editedTodo.text" label="Task"></v-text-field>
          <v-select v-model="editedTodo.priority" :items="['Least', 'Normal', 'Most']" label="Priority"></v-select>
        </v-card-text>
        <v-card-actions>
          <v-btn color="primary" @click="saveTodo">Save</v-btn>
          <v-btn text @click="dialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="deleteDialog" max-width="400px">
      <v-card>
        <v-card-title>Are you sure?</v-card-title>
        <v-card-text>Do you really want to delete this task?</v-card-text>
        <v-card-actions>
          <v-btn color="red" @click="deleteTodo">Yes</v-btn>
          <v-btn text @click="deleteDialog = false">No</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      newTodo: {
        text: '',
        priority: 'Normal',
      },
      todos: [],
      dialog: false,
      editedTodo: {},
      darkMode: false,
      deleteDialog: false,
      todoToDelete: null,
      filter: 'All',
    };
  },
  mounted() {
    this.loadTodos();
    this.darkMode = JSON.parse(localStorage.getItem('darkMode')) || false;
  },
  computed: {
    filteredTodos() {
      if (this.filter === 'Completed') return this.todos.filter(todo => todo.completed);
      if (this.filter === 'Pending') return this.todos.filter(todo => !todo.completed);
      return this.todos;
    },
    completedTasks() {
      return this.todos.filter(todo => todo.completed).length;
    },
    progressBarColor() {
      const completionRate = (this.completedTasks / this.todos.length) * 100;
      if (completionRate === 0) return 'red';
      if (completionRate > 0 && completionRate < 100) return 'orange';
      if (completionRate === 100) return 'green';
      return 'grey';
    },
  },
  methods: {
    loadTodos() {
      const todos = JSON.parse(localStorage.getItem('todos'));
      this.todos = todos || [];
    },
    addTodo() {
      if (!this.newTodo.text.trim()) return;
      this.todos.push({
        id: Date.now(),
        text: this.newTodo.text,
        completed: false,
        priority: this.newTodo.priority,
      });
      this.newTodo.text = '';
      this.newTodo.priority = 'Normal';
    },
    editTodo(todo) {
      this.editedTodo = { ...todo };
      this.dialog = true;
    },
    saveTodo() {
      const index = this.todos.findIndex(t => t.id === this.editedTodo.id);
      if (index !== -1) {
        this.todos.splice(index, 1, { ...this.editedTodo });
      }
      this.dialog = false;
    },
    confirmDelete(todo) {
      this.todoToDelete = todo;
      this.deleteDialog = true;
    },
    deleteTodo() {
      this.todos = this.todos.filter(todo => todo.id !== this.todoToDelete.id);
      this.deleteDialog = false;
    },
    toggleCompletion(todo) {
      todo.completed = !todo.completed;
    },
    getPriorityStyle(priority) {
      if (priority === 'Least') return { color: '#00FF7F' };
      if (priority === 'Normal') return { color: '#FF7F50' };
      if (priority === 'Most') return { color: '#FFD700' };
      return {};
    },
  },
  watch: {
    todos: {
      handler(newTodos) {
        localStorage.setItem('todos', JSON.stringify(newTodos));
      },
      deep: true,
    },
    darkMode(val) {
      localStorage.setItem('darkMode', JSON.stringify(val));
      document.body.classList.toggle('theme--dark', val);
    },
  },
};
</script>

<style scoped>
.my-card {
  border-radius: 12px;
  transition: box-shadow 0.3s;
}

.my-card:hover {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.cards-background {
  background-color: #ffffff;
  border-radius: 8px;
  padding: 20px;
  margin-top: 16px;
}

.title {
  font-size: 1.5rem;
  font-weight: bold;
}

.task-input {
  background: transparent;
}

.add-task-btn {
  border-radius: 20px;
}

.task-card {
  border-radius: 8px;
  transition: box-shadow 0.3s;
}

.task-card:hover {
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.15);
}

.completed {
  text-decoration: line-through;
  color: grey;
}

.v-btn {
  transition: background-color 0.3s;
}

.v-btn:hover {
  background-color: rgba(58, 134, 255, 0.1);
}
</style>
