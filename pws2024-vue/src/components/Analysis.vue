<script>
import common from '../mixins/common'

const projectEndpoint = '/api/project'

export default {
    props: ['session'],
    emits: ['display-message'],
    mixins: [common],
    data() {
        return {
            projects: [],
            selectedProjectId: null,
            currentDate: new Date().toISOString().substr(0, 10),
            projectsLoading: false,
            tasksLoading: false,
            projectDetails: null // Store the selected project details separately
        }
    },
    computed: {
        selectedProject() {
            // Use projectDetails if available, otherwise find in projects array
            if (this.projectDetails) {
                return this.projectDetails;
            }
            if (!this.selectedProjectId) return null;
            return this.projects.find(p => p._id === this.selectedProjectId);
        },
        projectGanttData() {
            if (!this.projects || this.projects.length === 0) return [];
            
            return this.projects.map(project => {
                const startDate = new Date(project.startDate);
                const endDate = project.endDate ? new Date(project.endDate) : new Date();
                const isCompleted = project.endDate ? true : false;
                
                return {
                    id: project._id,
                    name: project.name,
                    start: startDate,
                    end: endDate,
                    isCompleted: isCompleted
                };
            });
        },
        taskGanttData() {
            if (!this.selectedProject) return [];
            
            // Check if tasks exist and has items
            if (!this.selectedProject.tasks || !Array.isArray(this.selectedProject.tasks) || this.selectedProject.tasks.length === 0) {
                return [];
            }
            
            return this.selectedProject.tasks.map(task => {
                const startDate = new Date(task.startDate);
                const endDate = task.endDate ? new Date(task.endDate) : new Date();
                const isCompleted = task.endDate ? true : false;
                
                return {
                    id: task.name || `task-${Math.random().toString(36).substr(2, 9)}`,
                    name: task.name || 'Unnamed Task',
                    start: startDate,
                    end: endDate,
                    isCompleted: isCompleted
                };
            });
        },
        hasTasksToDisplay() {
            return this.selectedProject && 
                   this.selectedProject.tasks && 
                   Array.isArray(this.selectedProject.tasks) && 
                   this.selectedProject.tasks.length > 0;
        }
    },
    methods: {
        loadProjects() {
            this.projectsLoading = true;
            fetch(projectEndpoint)
                .then(res => res.json()
                    .then(facet => {
                        if (facet.data) {
                            this.projects = facet.data;
                            this.projectsLoading = false;
                        }
                    })
                )
                .catch(err => {
                    console.error('Error loading projects:', err);
                    this.projectsLoading = false;
                    this.$emit('display-message', 'Failed to load projects', 'error');
                });
        },
        formatDate(date) {
            if (!date) return 'N/A';
            return new Date(date).toLocaleDateString();
        },
        calculateWidth(startDate, endDate) {
            const start = new Date(startDate);
            const end = endDate ? new Date(endDate) : new Date();
            const totalDays = Math.ceil((end - start) / (1000 * 60 * 60 * 24));
            return `${Math.max(totalDays * 3, 30)}px`; // Min width of 30px
        },
        isOngoing(item) {
            const today = new Date();
            const start = new Date(item.start);
            const end = new Date(item.end);
            
            return start <= today && (!item.isCompleted && end >= today);
        },
        onProjectSelect() {
            // Reset if no selection
            if (!this.selectedProjectId) {
                this.projectDetails = null;
                return;
            }
            
            this.tasksLoading = true;
            
            // Fetch complete project data including tasks
            fetch(projectEndpoint + '?' + new URLSearchParams({ 
                _id: this.selectedProjectId
            }).toString())
                .then(res => res.json())
                .then(facet => {
                    this.tasksLoading = false;
                    
                    if (facet.data && facet.data.length > 0) {
                        // Store the selected project details separately
                        // instead of modifying the projects array
                        this.projectDetails = facet.data[0];
                    } else {
                        console.warn('No project data returned for ID:', this.selectedProjectId);
                        this.projectDetails = null;
                    }
                })
                .catch(err => {
                    this.tasksLoading = false;
                    console.error('Error fetching project details:', err);
                    this.$emit('display-message', 'Failed to load project tasks', 'error');
                });
        }
    },
    mounted() {
        this.loadProjects();
    }
}
</script>

<template>
    <div>
        <h1>Project Analysis</h1>
        
        <v-card class="mb-6">
            <v-card-title>Projects Timeline</v-card-title>
            <v-card-text>
                <div v-if="projectsLoading" class="d-flex justify-center">
                    <v-progress-circular indeterminate color="primary"></v-progress-circular>
                </div>
                <div v-else-if="projects.length === 0" class="text-center">
                    No projects available.
                </div>
                <div v-else class="gantt-container">
                    <div class="gantt-chart">
                        <!-- Projects Gantt -->
                        <div v-for="item in projectGanttData" :key="item.id" class="gantt-row">
                            <div class="gantt-label">{{ item.name }}</div>
                            <div class="gantt-timeline">
                                <div 
                                    class="gantt-bar" 
                                    :class="{ 
                                        'completed': item.isCompleted,
                                        'ongoing': isOngoing(item)
                                    }"
                                    :style="{ 
                                        width: calculateWidth(item.start, item.end)
                                    }"
                                >
                                    {{ formatDate(item.start) }} - {{ formatDate(item.end) }}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </v-card-text>
        </v-card>
        
        <v-card>
            <v-card-title>Task Analysis</v-card-title>
            <v-card-text>
                <v-select
                    v-model="selectedProjectId"
                    :items="projects"
                    item-title="name"
                    item-value="_id"
                    label="Select Project"
                    variant="outlined"
                    density="comfortable"
                    @update:model-value="onProjectSelect"
                ></v-select>
                
                <div v-if="tasksLoading" class="d-flex justify-center my-4">
                    <v-progress-circular indeterminate color="primary"></v-progress-circular>
                </div>
                <div v-else-if="!selectedProject" class="text-center my-4">
                    Select a project to view tasks.
                </div>
                <div v-else-if="!hasTasksToDisplay" class="text-center my-4">
                    <p>No tasks available for this project.</p>
                    <p class="text-body-2 mt-2">
                        Try adding tasks to this project by editing it in the Projects section.
                    </p>
                </div>
                <div v-else class="gantt-container mt-4">
                    <div class="gantt-chart">
                        <!-- Tasks Gantt -->
                        <div v-for="item in taskGanttData" :key="item.id" class="gantt-row">
                            <div class="gantt-label">{{ item.name }}</div>
                            <div class="gantt-timeline">
                                <div 
                                    class="gantt-bar" 
                                    :class="{ 
                                        'completed': item.isCompleted,
                                        'ongoing': isOngoing(item)
                                    }"
                                    :style="{ 
                                        width: calculateWidth(item.start, item.end)
                                    }"
                                >
                                    {{ formatDate(item.start) }} - {{ formatDate(item.end) }}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </v-card-text>
        </v-card>
    </div>
</template>

<style scoped>
.gantt-container {
    overflow-x: auto;
    margin-top: 1rem;
}

.gantt-chart {
    display: flex;
    flex-direction: column;
    min-width: 100%;
}

.gantt-row {
    display: flex;
    margin-bottom: 0.5rem;
    align-items: center;
}

.gantt-label {
    width: 150px;
    font-weight: bold;
    padding-right: 1rem;
    text-align: right;
}

.gantt-timeline {
    flex-grow: 1;
    position: relative;
    height: 35px;
    background-color: #f5f5f5;
    border-radius: 4px;
}

.gantt-bar {
    position: relative;
    height: 100%;
    background-color: #1976D2;
    color: white;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.8rem;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    padding: 0 8px;
}

.gantt-bar.completed {
    background-color: #4CAF50;
}

.gantt-bar.ongoing {
    background-color: #FF9800;
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% {
        opacity: 0.8;
    }
    50% {
        opacity: 1;
    }
    100% {
        opacity: 0.8;
    }
}
</style>