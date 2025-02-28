<!-- TasksModal.vue with improved Vuetify styling -->
<template>
  <div class="modal-overlay">
    <v-card class="modal-content">
      <v-card-title class="text-h5">Manage Tasks</v-card-title>
      
      <v-card-text>
        <v-container>
          <v-card v-for="(task, index) in tasks" :key="index" class="mb-4 pa-2" variant="outlined">
            <v-row>
              <v-col cols="12">
                <v-text-field
                  label="Task Name"
                  v-model="task.name"
                  variant="outlined"
                  density="comfortable"
                  placeholder="Enter task name"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
            
            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  label="Start Date"
                  type="date"
                  v-model="task.startDate"
                  variant="outlined"
                  density="comfortable"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12" md="6">
                <v-text-field
                  label="End Date"
                  type="date"
                  v-model="task.endDate"
                  variant="outlined"
                  density="comfortable"
                ></v-text-field>
              </v-col>
            </v-row>
            
            <v-row>
              <v-col cols="12">
                <v-autocomplete
                  v-model="task.workers"
                  :items="persons"
                  label="Workers"
                  multiple
                  chips
                  closable-chips
                  variant="outlined"
                  density="comfortable"
                  :item-title="person => person.firstName + ' ' + person.lastName"
                  item-value="_id"
                ></v-autocomplete>
              </v-col>
            </v-row>
            
            <v-row>
              <v-col class="text-right">
                <v-btn color="error" variant="elevated" @click="deleteTask(index)">
                  <v-icon left>mdi-delete</v-icon> Delete
                </v-btn>
              </v-col>
            </v-row>
          </v-card>
          
          <v-row v-if="tasks.length === 0">
            <v-col class="text-center">
              <p>No tasks yet. Click "Add Task" to create a new task.</p>
            </v-col>
          </v-row>
        </v-container>
      </v-card-text>
      
      <v-divider></v-divider>
      
      <v-card-actions>
        <v-btn color="primary" variant="elevated" @click="addTask">
          <v-icon left>mdi-plus</v-icon> Add Task
        </v-btn>
        <v-spacer></v-spacer>
        <v-btn color="success" variant="elevated" @click="saveTasks">
          Save & Close
        </v-btn>
        <v-btn color="grey" variant="elevated" @click="$emit('cancel')">
          Cancel
        </v-btn>
      </v-card-actions>
    </v-card>
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
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  max-width: 800px;
  width: 90%;
  max-height: 85vh;
  overflow-y: auto;
}
</style>