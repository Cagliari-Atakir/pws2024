<!-- TasksModal.vue -->
<template>
    <div class="modal-overlay">
      <div class="modal-content">
        <h2>Manage Tasks</h2>
        <div v-for="(task, index) in tasks" :key="index" class="task-item">
          <v-text-field
            label="Task Name"
            v-model="task.name"
          ></v-text-field>
          <v-text-field
            label="Start Date"
            type="date"
            v-model="task.startDate"
          ></v-text-field>
          <v-text-field
            label="End Date"
            type="date"
            v-model="task.endDate"
          ></v-text-field>
          <v-autocomplete
            v-model="task.workers"
            :items="persons"
            label="Workers"
            multiple
            chips
            :item-title="person => person.firstName + ' ' + person.lastName"
            item-value="_id"
          ></v-autocomplete>
          
          <v-btn color="error" variant="elevated" @click="deleteTask(index)">Delete</v-btn>
        </div>
  
        <v-btn color="primary" variant="elevated" @click="addTask">Add Task</v-btn>
        <v-btn color="success" variant="elevated" @click="saveTasks">Save & Close</v-btn>
        <v-btn variant="elevated" @click="$emit('cancel')">Cancel</v-btn>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      initialTasks: {
        type: Array,
        default: () => []
      },
      persons: {
        type: Array,
        default: () => []
      }
    },
    data() {
      return {
        tasks: JSON.parse(JSON.stringify(this.initialTasks)) // make a local copy
      }
    },
    methods: {
      addTask() {
        this.tasks.push({
          name: '',
          startDate: '',
          endDate: '',
          workers: []
        })
      },
      deleteTask(index) {
        this.tasks.splice(index, 1)
      },
      saveTasks() {
        // Emit the updated tasks to the parent
        this.$emit('close', this.tasks)
      }
    }
  }
  </script>
  
  <style scoped>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0,0,0,0.4);
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .modal-content {
    background: #fff;
    padding: 20px;
    border-radius: 6px;
    max-width: 600px;
    width: 90%;
  }
  .task-item {
    margin-bottom: 20px;
    border: 1px solid #eee;
    padding: 10px;
    border-radius: 4px;
  }
  </style>
  