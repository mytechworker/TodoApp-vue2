<template>
  <div class="todo-app">
    <h1>Todo App in Vue2</h1>
    <input
      v-model="newTask"
      class="todo_title"
      @keyup.enter="addTask"
      placeholder="Add a new task and Press Enter"
    />
    <table class="todo-table">
      <thead>
        <tr>
          <th>Status</th>
          <th>Task</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="task in tasks" :key="task.id">
          <td>
            <input
              type="checkbox"
              :checked="task.completed"
              @click="toggleTask(task)"
            />
          </td>
          <td>
            <span :class="{ completed: task.completed }">{{ task.text }}</span>
          </td>
          <td>
            <button @click="removeTask(task)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      newTask: "",
      tasks: [],
    };
  },
  created() {
    this.fetchTasks();
  },
  methods: {
    async fetchTasks() {
      try {
        const response = await axios.get("http://localhost:3000/tasks");
        this.tasks = response.data;
      } catch (error) {
        console.error("Error fetching tasks:", error);
      }
    },
    async addTask() {
      if (this.newTask.trim() === "") return;
      try {
        const response = await axios.post("http://localhost:3000/tasks", {
          text: this.newTask,
          completed: false,
        });
        this.tasks.push(response.data);
        this.newTask = "";
      } catch (error) {
        console.error("Error adding task:", error);
      }
    },
    async toggleTask(task) {
      try {
        task.completed = !task.completed;
        const response = await axios.patch(
          `http://localhost:3000/tasks/${task.id}`,
          {
            completed: task.completed,
          }
        );
        task.completed = response.data.completed;
      } catch (error) {
        console.error("Error toggling task:", error);
      }
    },
    async removeTask(task) {
      try {
        await axios.delete(`http://localhost:3000/tasks/${task.id}`);
        const index = this.tasks.findIndex((t) => t.id === task.id);
        if (index !== -1) {
          this.tasks.splice(index, 1);
        }
      } catch (error) {
        console.error("Error removing task:", error);
      }
    },
  },
};
</script>

<style scoped>

/* todo_title style */
.todo-app h1{
   text-align:center;
}
.todo_title{
  max-width: 400px;
  width: 100%;
  margin: 50px auto 30px;
  padding: 15px;
  display: flex;
}
@media (max-width:425px){
  .todo_title{
    max-width:80%;
  }
}
/* start table style */
.todo-table {
  max-width:1300px;
  width: 100%;
  margin:0 auto;
  border-collapse: collapse;
  margin-top: 20px;
}
.todo-table :is(thead,tbody) tr{
  width:100%;
}
.todo-table :is(th,td){
  width:100%;
}
.todo-table :is(thead,tbody) :is(th:first-child,td:first-child),
.todo-table :is(thead,tbody) :is(th:last-child,td:last-child){
   width:50px;
}


.todo-table th,
.todo-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
}

.todo-table th {
  background-color: #f0f0f0;
}

.completed {
  text-decoration: line-through;
}
</style>
