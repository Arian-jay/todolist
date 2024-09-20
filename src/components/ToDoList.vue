<template>
    <v-container>
      <v-card>
        <v-card-title>
          <v-text-field
            v-model="newTodo"
            @keyup.enter="addTodo"
            label="Add a new task"
            outlined
          ></v-text-field>
        </v-card-title>
  
        <v-list>
          <v-list-item-group v-if="todos.length">
            <v-list-item v-for="todo in todos" :key="todo.id">
              <v-list-item-content class="d-flex align-center">
                <v-checkbox
                  v-model="todo.completed"
                  class="mr-2"
                  hide-details
                ></v-checkbox>
                <v-list-item-title :class="{ completed: todo.completed }" class="flex-grow-1">
                  {{ todo.text }}
                </v-list-item-title>
                <v-icon
                  @click="removeTodo(todo.id)"
                  class="ml-2"
                  style="cursor: pointer;"
                >
                  mdi-delete
                </v-icon>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
          <v-list-item v-else>No tasks available</v-list-item>
        </v-list>
      </v-card>
    </v-container>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newTodo: '',
        todos: []
      };
    },
    methods: {
      addTodo() {
        if (this.newTodo.trim() === '') return;
        this.todos.push({
          id: Date.now(),
          text: this.newTodo,
          completed: false
        });
        this.newTodo = '';
      },
      removeTodo(id) {
        this.todos = this.todos.filter(todo => todo.id !== id);
      }
    }
  };
  </script>
  
  <style scoped>
  .v-list-item-content {
    display: flex;
    align-items: center;
  }
  
  .v-list-item-title {
    flex-grow: 1;
  }
  
  .v-checkbox {
    margin-right: 16px;
  }
  
  .v-icon {
    margin-left: 16px;
  }
  
  .completed {
    color: gray;
    text-decoration: line-through;
  }
  </style>
  