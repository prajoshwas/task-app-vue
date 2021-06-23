<template>
<div>
    <div>
        <AddTask v-show="showAddTask" @add-task="addTask" />
         <Tasks
            @toggle-reminder="toggleReminder"
            @delete-task="deleteTask"
            v-bind:tasks="tasks" />
    </div>
</div>
</template>

<script>

import Tasks from '../components/Tasks.vue';
import AddTask from '../components/AddTask.vue';

export default {
  name: 'Home',
  components: {
    Tasks,
    AddTask,
  },
  props: {
    showAddTask: Boolean,
  },
  data() {
    return {
      tasks: [],
    };
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
  methods: {
    async addTask(task) {
      const response = await fetch('api/tasks', {
        method: 'POST',
        headers: { 'Content-type': 'application/json' },
        body: JSON.stringify(task),
      });
      const data = await response.json();
      this.tasks = [...this.tasks, data];
    },
    async deleteTask(id) {
      if (window.confirm('Are you sure?')) {
        const response = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        });
        if (response.status === 200) {
          this.tasks = this.tasks.filter((task) => task.id !== id);
        } else {
          alert('Error in deleting task');
        }
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updatedTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

      const response = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updatedTask),
      });

      const data = await response.json();
      this.tasks = this.tasks.map((task) => (task.id === id
        ? { ...task, reminder: data.reminder } : task));
    },
    async fetchTasks() {
      const response = await fetch('api/tasks');
      const data = response.json();
      return data;
    },
    async fetchTask(id) {
      const response = await fetch(`api/tasks/${id}`);
      const data = response.json();
      return data;
    },
  },
};
</script>
