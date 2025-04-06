<template>
  <div class="min-h-screen bg-gray-50 p-4 md:p-6">
    <!-- Toast Notifications -->
    <div class="fixed top-4 right-4 z-50 flex flex-col gap-2">
      <div 
        v-for="(toast, index) in toasts" 
        :key="index" 
        :class="[
          'toast-notification px-4 py-3 rounded-lg shadow-lg transform transition-all duration-300 max-w-md',
          `toast-${toast.type}`
        ]"
        style="animation: slideIn 0.3s ease-out forwards"
      >
        <div class="flex items-center gap-2">
          <span v-if="toast.type === 'success'" class="text-green-500">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
            </svg>
          </span>
          <span v-if="toast.type === 'error'" class="text-red-500">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
            </svg>
          </span>
          <span v-if="toast.type === 'info'" class="text-blue-500">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2h-1V9a1 1 0 00-1-1z" clip-rule="evenodd" />
            </svg>
          </span>
          <div class="flex-1">{{ toast.message }}</div>
          <button 
            @click="removeToast(index)" 
            class="text-gray-400 hover:text-gray-600"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Task Detail Modal -->
    <div v-if="showModal" class="fixed inset-0 z-50 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
      <div class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
        <!-- Background overlay -->
        <div 
          class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" 
          aria-hidden="true"
          @click="closeModal"
        ></div>

        <!-- Modal panel -->
        <div class="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full">
          <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
            <div class="sm:flex sm:items-start">
              <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left w-full">
                <div class="flex justify-between items-center mb-4">
                  <h3 class="text-lg leading-6 font-medium text-gray-900" id="modal-title">
                    Task Details
                  </h3>
                  <span
                    :class="{
                      'status-badge in-progress': selectedTask.status?.toLowerCase() === 'in_progress',
                      'status-badge pending': selectedTask.status?.toLowerCase() === 'pending',
                      'status-badge completed': selectedTask.status?.toLowerCase() === 'completed'
                    }"
                  >
                    {{ selectedTask.status?.toLowerCase().replace("_", " ") }}
                  </span>
                </div>
                <div class="mt-4">
                  <h4 class="text-xl font-semibold text-gray-900">{{ selectedTask.title }}</h4>
                  
                  <div class="mt-4">
                    <h5 class="text-sm font-medium text-gray-700">Description</h5>
                    <p class="mt-1 text-gray-600">
                      {{ selectedTask.description || 'No description provided.' }}
                    </p>
                  </div>
                  
                  <div class="mt-4 grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <h5 class="text-sm font-medium text-gray-700">Due Date</h5>
                      <p class="mt-1 text-gray-600">{{ formatDateTime(selectedTask.dueDate) }}</p>
                    </div>
                    <div>
                      <h5 class="text-sm font-medium text-gray-700">Created</h5>
                      <p class="mt-1 text-gray-600">{{ new Date(selectedTask.createdAt).toUTCString() }}</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
            <button 
              type="button" 
              class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-600 text-base font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:ml-3 sm:w-auto sm:text-sm"
              @click="updateTaskStatus(selectedTask); closeModal();"
            >
              {{ getNextStatusButtonText(selectedTask.status?.toLowerCase()) }}
            </button>
            <button 
              type="button" 
              class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm"
              @click="closeModal"
            >
              Close
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="max-w-6xl mx-auto">
      <header class="mb-8">
        <h1 class="text-3xl font-bold text-gray-900">Caseworker Task Management</h1>
        <p class="text-gray-600 mt-2">Efficiently manage and track your casework tasks</p>
      </header>
  
      <!-- Task Creation Form -->
      <div class="bg-white rounded-lg shadow-md p-6 mb-8">
        <h2 class="text-xl font-semibold mb-4">{{ 'Create New Task' }}</h2>
        <form @submit.prevent="createTask()">
          <div class="mb-4">
            <label for="title" class="block text-sm font-medium text-gray-700 mb-1">Title</label>
            <input
              id="title"
              v-model="newTask.title"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              required
            />
          </div>
          <div class="mb-4">
            <label for="description" class="block text-sm font-medium text-gray-700 mb-1">Description (optional)</label>
            <textarea
              id="description"
              v-model="newTask.description"
              rows="3"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            ></textarea>
          </div>
          <div class="mb-4">
            <label for="dueDate" class="block text-sm font-medium text-gray-700 mb-1">Due Date/Time</label>
            <input
              id="dueDate"
              v-model="newTask.dueDate"
              type="datetime-local"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              required
            />
          </div>
          <!-- Status field only shown when editing -->
          
          <div class="flex justify-end gap-2">
            <button
              type="submit"
              class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
            >
              {{ 'Create Task' }}
            </button>
          </div>
        </form>
      </div>
  
      <!-- Task Filtering and Sorting -->
      <div class="bg-white rounded-lg shadow-md p-6 mb-8">
        <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
          <div>
            <label for="filterStatus" class="block text-sm font-medium text-gray-700 mb-1">Status</label>
            <select
              id="filterStatus"
              v-model="filters.status"
              class="px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option value="all">All</option>
              <option value="pending">Pending</option>
              <option value="in_progress">In Progress</option>
              <option value="completed">Completed</option>
            </select>
          </div>
          <div>
            <label for="sortBy" class="block text-sm font-medium text-gray-700 mb-1">Sort By</label>
            <select
              id="sortBy"
              v-model="sortBy"
              class="px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option value="dueDate">Due Date</option>
              <option value="title">Title</option>
              <option value="createdAt">Created Date</option>
            </select>
          </div>
        </div>
      </div>
  
      <!-- Task List -->
      <div class="bg-white rounded-lg shadow-md overflow-hidden">
        <div class="p-6 border-b border-gray-200">
          <h2 class="text-xl font-semibold">Tasks ({{ filteredTasks.length }})</h2>
        </div>
        <div v-if="filteredTasks.length === 0" class="p-6 text-center text-gray-500">
          No tasks found. Create a new task to get started.
        </div>
        <ul v-else class="divide-y divide-gray-200">
          <li v-for="task in filteredTasks" :key="task.id" class="task-item p-6 hover:bg-gray-50">
            <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
              <div class="flex-1 cursor-pointer" @click="viewTask(task)">
                <div class="flex items-center gap-2">
                  <span
                    :class="{
                      'status-badge in-progress': task.status.toLowerCase() === 'in_progress',
                      'status-badge pending': task.status.toLowerCase() === 'pending',
                      'status-badge completed': task.status.toLowerCase() === 'completed'
                    }"
                  >
                    {{ task.status.toLowerCase().replace("_", " ") }}
                  </span>
                </div>
                <h3 class="task-title text-lg font-medium text-gray-900 mt-2">{{ task.title }}</h3>
                <p v-if="task.description" class="task-description text-gray-600 mt-1">{{ task.description }}</p>
                <div class="mt-2 text-sm text-gray-500">
                  <span v-if="task.dueDate" class="due-date mr-4">Due: {{ formatDateTime(task.dueDate) }}</span>
                </div>
                <div class="mt-2 text-sm text-gray-500">
                  <span v-if="task.createdAt" class="due-date mr-4">Created At: {{ new Date(task.createdAt).toUTCString() }}</span>
                </div>
              </div>
              <div class="task-actions flex items-center gap-2 mt-4 md:mt-0">
                <button
                  @click="viewTask(task)"
                  class="view-button px-3 py-1.5 text-sm border border-blue-300 rounded-md text-blue-700 hover:bg-blue-50"
                >
                  View
                </button>
                <button
                  @click="updateTaskStatus(task)"
                  :class="[
                    'status-button px-3 py-1.5 text-sm border rounded-md hover:bg-gray-50',
                    {
                      'border-blue-300 text-blue-700': task.status.toLowerCase() === 'pending',
                      'border-green-300 text-green-700': task.status.toLowerCase() === 'in_progress',
                      'border-gray-300 text-gray-700': task.status.toLowerCase() === 'completed'
                    }
                  ]"
                >
                  {{ getNextStatusButtonText(task.status.toLowerCase()) }}
                </button>
                <button
                  @click="deleteTask(task.id)"
                  class="delete-button px-3 py-1.5 text-sm border border-red-300 rounded-md text-red-700 hover:bg-red-50"
                >
                  Delete
                </button>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
  
<script setup>
import { ref, computed, onMounted } from 'vue';
const baseUrl = process.env.VUE_APP_API_BASE_URL;
// Task state
const tasks = ref([]);
const newTask = ref({
  title: '',
  description: '',
  status: 'pending', // Default status is 'pending'
  dueDate: '2025-04-03T08:36',
  createdAt: ''
});

// Modal state
const showModal = ref(false);
const selectedTask = ref({});

const viewTask = (task) => {
  selectedTask.value = { ...task };
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
};

// Store tasks
const filters = ref({ status: 'all' }); // Filters for task status
const sortBy = ref('dueDate'); // Sort by due date or created date

// Fetch tasks when component is mounted
onMounted(async () => {
  await fetchTasks();
});

// Function to fetch tasks from the API
const fetchTasks = async () => {
  try {
    const response = await fetch(baseUrl + '/tasks/');
    
    if (response.ok) {
      const data = await response.json();
      tasks.value = data.data;  // Store the fetched tasks
    } else {
      console.error('Failed to fetch tasks');
    }
  } catch (error) {
    console.error('Error fetching tasks:', error);
  }
};

// Computed property to filter and sort tasks
const filteredTasks = computed(() => {
  let result = [...tasks.value]; // Copy tasks to avoid mutating original array

  // Apply filters
  if (filters.value.status !== 'all') {
    result = result.filter(task => task.status.toLowerCase() === filters.value.status);
  }

  // Apply sorting
  result.sort((a, b) => {
    const dateA = new Date(a.createdAt);
    const dateB = new Date(a.createdAt);

    if (sortBy.value === 'dueDate') {
      return dateA - dateB; // Sort by due date
    } else if (sortBy.value === 'createdAt') {
      const createdAtA = new Date(a.createdAt);
      const createdAtB = new Date(a.createdAt);
      return createdAtB - createdAtA; // Sort by created date
    } else {
      return a[sortBy.value].localeCompare(b[sortBy.value]); // Default sort (lexical)
    }
  });

  return result;
});
  
// Methods
const createTask = async () => {
  const task = {
    //   ...newTask.value,
    title: newTask.value.title,
    description: newTask.value.description,
    dueDate: newTask.value.dueDate
  };
  
  try {
    const response = await fetch(baseUrl + '/tasks/create', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(task),
    });

    if (response.ok) {
      const data = await response.json();
      await fetchTasks();
      // Show success toast
      showToast(`${data.message}`, 'success');
      resetForm();
    } else {
      // Show error toast
      showToast("Failed to create task", 'error');
    }
  } catch (error) {
    showToast(`${error}`, 'error');
  }
  resetForm();
};
  
  
const deleteTask = async (id) => {
  if (confirm('Are you sure you want to delete this task?')) {
    try {
      const response = await fetch(`${baseUrl}/tasks/${id}`, {
        method: 'DELETE',
      });

      if (response.ok) {
        // Remove the task from the local list if deletion is successful
        tasks.value = tasks.value.filter(task => task.id !== id);

        showToast("Task deleted successfully", 'success');
      } else {
        console.error('Failed to delete task');
      }
    } catch (error) {
      console.error('Error deleting task:', error);
    }
  }
};
  
const updateTaskStatus = async (task) => {
  const index = tasks.value.findIndex(t => t.id === task.id);
  let status = null;
  if (index !== -1) {
    if (tasks.value[index].status.toLowerCase() === 'completed') {
      // tasks.value[index].status = 'pending';
      status = "IN_PROGRESS";
    } else if(tasks.value[index].status.toLowerCase() === 'in_progress'){
      // tasks.value[index].status = 'completed';
      status = "COMPLETED";
    }
    else {
      // tasks.value[index].status = 'completed';
      status = "IN_PROGRESS";
    }
  }

  const body = {
    status: status
  }
  try {
    const response = await fetch(`${baseUrl}/tasks/${task.id}`,{
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(body),
    });
    if (response.ok) {
      const data = await response.json();
      await fetchTasks();
      // Show success toast
      showToast(`${data.message}`, 'success');
    } else {
      showToast("Failed to fetch tasks", "error");
    }
  } catch (error) {
    showToast(`${error}`, "error");
  }
};
  
const resetForm = () => {
  newTask.value = {
    title: '',
    description: '',
    status: 'pending', // Reset status to 'pending'
    dueDate: '2025-04-03T08:36',
    createdAt: ''
  };
};
  
const formatDateTime = (dateTimeArray) => {
  if (!dateTimeArray) return '';

  const date = new Date(dateTimeArray);

  const options = { 
    year: 'numeric', 
    month: 'short', 
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  };
  return date.toLocaleString(undefined, options);
};

// Get the next status button text based on current status
const getNextStatusButtonText = (currentStatus) => {
  switch (currentStatus) {
    case 'pending':
      return 'Start Progress';
    case 'in_progress':
      return 'Complete';
    case 'completed':
      return 'Reopen';
    default:
      return 'Update Status';
  }
};

// Toast notifications
const toasts = ref([]);

const showToast = (message, type = 'info', duration = 3000) => {
  const toast = { message, type };
  toasts.value.push(toast);
  
  // Auto-remove toast after duration
  setTimeout(() => {
    removeToast(toasts.value.indexOf(toast));
  }, duration);
};

const removeToast = (index) => {
  if (index > -1) {
    toasts.value.splice(index, 1);
  }
};
</script>
  
<style>
/* Custom styles for the task management application */

/* General styles */
:root {
  --primary-color: #3b82f6;
  --primary-hover: #2563eb;
  --danger-color: #ef4444;
  --danger-hover: #dc2626;
  --pending-color: #9ca3af;
  --in-progress-color: #3b82f6;
  --completed-color: #10b981;
}

/* Form styles */
input:focus, select:focus, textarea:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
}

/* Task item styles */
.task-item {
  transition: all 0.2s ease;
  border-left: 4px solid transparent;
}

.task-item:hover {
  border-left-color: var(--primary-color);
}

.task-title {
  font-weight: 600;
  transition: color 0.2s ease;
}

.task-description {
  line-height: 1.5;
  max-height: 4.5em;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.due-date {
  font-weight: 500;
}

/* Status badge styles */
.status-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: capitalize;
}

.status-badge.pending {
  background-color: #f3f4f6;
  color: #4b5563;
}

.status-badge.in-progress {
  background-color: #dbeafe;
  color: #1e40af;
}

.status-badge.completed {
  background-color: #d1fae5;
  color: #065f46;
}

/* Button styles */
button {
  transition: all 0.2s ease;
}

button:hover {
  transform: translateY(-1px);
}

button:active {
  transform: translateY(0);
}

.edit-button:hover {
  border-color: var(--primary-color);
  color: var(--primary-color);
}

.status-button:hover {
  border-color: var(--primary-color);
  color: var(--primary-color);
}

.delete-button:hover {
  background-color: var(--danger-color);
  border-color: var(--danger-color);
  color: white;
}

.view-button:hover {
  background-color: var(--primary-color);
  border-color: var(--primary-color);
  color: white;
}

/* Toast notification styles */
.toast-notification {
  opacity: 0;
  transform: translateX(100%);
}

.toast-success {
  background-color: #ecfdf5;
  border-left: 4px solid #10b981;
}

.toast-error {
  background-color: #fef2f2;
  border-left: 4px solid #ef4444;
}

.toast-info {
  background-color: #eff6ff;
  border-left: 4px solid #3b82f6;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slideOut {
  from {
    opacity: 1;
    transform: translateX(0);
  }
  to {
    opacity: 0;
    transform: translateX(100%);
  }
}

/* Modal animation */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(10px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .task-actions {
    display: flex;
    justify-content: flex-start;
    width: 100%;
  }
  
  .toast-notification {
    max-width: 90vw;
  }
}

/* Animation for new tasks */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.task-item {
  animation: fadeIn 0.3s ease-out;
}
</style>