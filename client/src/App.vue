<template>
  <div class="app" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <aside class="sidebar" :aria-expanded="!sidebarCollapsed">
      <button
        class="sidebar-toggle"
        @click="sidebarCollapsed = !sidebarCollapsed"
        :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        :aria-label="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'"
      >
        <svg v-if="!sidebarCollapsed" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M15.41 16.59L10.83 12l4.58-4.59L14 6l-6 6 6 6z"/></svg>
        <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6z"/></svg>
      </button>
      <div class="sidebar-logo">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M3 3h8v8H3zm10 0h8v8h-8zM3 13h8v8H3zm10 0h8v8h-8z"/></svg>
          <span>{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M20 7H4a1 1 0 00-1 1v11a1 1 0 001 1h16a1 1 0 001-1V8a1 1 0 00-1-1zM9 17H7v-2h2v2zm4 0h-2v-2h2v2zm4 0h-2v-2h2v2zM20 5H4a1 1 0 000 2h16a1 1 0 000-2zM9 3h6v2H9z"/></svg>
          <span>{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M8 6h13M8 12h13M8 18h13M3 6h.01M3 12h.01M3 18h.01"/></svg>
          <span>{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/></svg>
          <span>{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M16 6l2.29 2.29-4.88 4.88-4-4L2 16.59 3.41 18l6-6 4 4 6.3-6.29L22 12V6z"/></svg>
          <span>{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/restocking" :class="{ active: $route.path === '/restocking' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M17.65 6.35A7.958 7.958 0 0012 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0112 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg>
          <span>{{ t('nav.restocking') }}</span>
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8l-6-6zm-1 1.5L18.5 9H13V3.5zM8 17v-2h8v2H8zm0-4v-2h8v2H8zm0-4V7h4v2H8z"/></svg>
          <span>Reports</span>
        </router-link>
      </nav>
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>
    <div class="main-wrapper">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
      <ProfileDetailsModal
        :is-open="showProfileDetails"
        @close="showProfileDetails = false"
      />
      <TasksModal
        :is-open="showTasks"
        :tasks="tasks"
        @close="showTasks = false"
        @add-task="addTask"
        @delete-task="deleteTask"
        @toggle-task="toggleTask"
      />
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])
    const sidebarCollapsed = ref(false)

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* Global accessible focus indicator */
:focus-visible {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}

body {
  font-family: 'Nunito', sans-serif;
  background: #f5f0ff;
  color: #111827;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: 240px;
  background: linear-gradient(180deg, #7c3aed 0%, #4f46e5 60%, #2563eb 100%);
  display: flex;
  flex-direction: column;
  z-index: 100;
  overflow-y: auto;
  /* Animate width when collapsing/expanding */
  transition: width 0.25s ease;
}

.sidebar-logo {
  padding: 1.5rem 1.25rem 1rem;
  border-bottom: 1px solid rgba(255,255,255,0.15);
}

.sidebar-logo h1 {
  font-family: 'Fredoka One', cursive;
  font-size: 1.25rem;
  color: #ffffff;
  letter-spacing: 0.01em;
  line-height: 1.2;
}

.sidebar-subtitle {
  display: block;
  font-size: 0.75rem;
  color: #e0d9ff;
  margin-top: 0.25rem;
  font-weight: 600;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0;
  display: flex;
  flex-direction: column;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1.25rem;
  border-radius: 12px;
  margin: 0.2rem 0.75rem;
  font-weight: 700;
  font-size: 0.938rem;
  text-decoration: none;
  color: #e0d9ff;
  transition: all 0.2s ease;
}

.sidebar-nav a svg {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
  opacity: 0.8;
}

.sidebar-nav a:hover {
  background: rgba(255,255,255,0.12);
  color: #ffffff;
}

.sidebar-nav a:hover svg {
  opacity: 1;
}

.sidebar-nav a.active {
  background: rgba(255,255,255,0.22);
  color: #ffffff;
}

.sidebar-nav a.active svg {
  opacity: 1;
}

.sidebar-footer {
  padding: 1rem 1rem 1.25rem;
  border-top: 1px solid rgba(255,255,255,0.15);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* ── Sidebar toggle button ── */
.sidebar-toggle {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  width: 100%;
  padding: 0.75rem 1rem 0.5rem;
  background: none;
  border: none;
  color: rgba(255,255,255,0.6);
  cursor: pointer;
  transition: color 0.2s;
}

.sidebar-toggle:hover { color: #ffffff; }

.sidebar-toggle svg { width: 20px; height: 20px; }

.sidebar-toggle:focus-visible {
  outline: 2px solid rgba(255,255,255,0.6);
  outline-offset: 2px;
}

/* ── Collapsed sidebar states ── */
.app.sidebar-collapsed .sidebar { width: 64px; }

.app.sidebar-collapsed .main-wrapper { margin-left: 64px; }

.app.sidebar-collapsed .sidebar-logo {
  padding: 1rem 0;
  display: flex;
  justify-content: center;
}

.app.sidebar-collapsed .sidebar-logo h1 { display: none; }

.app.sidebar-collapsed .sidebar-subtitle { display: none; }

.app.sidebar-collapsed .sidebar-nav a {
  justify-content: center;
  padding: 0.75rem;
  margin: 0.2rem 0.5rem;
}

.app.sidebar-collapsed .sidebar-nav a span { display: none; }

.app.sidebar-collapsed .sidebar-nav a svg { opacity: 1; }

.app.sidebar-collapsed .sidebar-footer {
  flex-direction: column;
  gap: 0.5rem;
  padding: 0.75rem 0;
  align-items: center;
}

/* Center the toggle icon when collapsed */
.app.sidebar-collapsed .sidebar-toggle { justify-content: center; }

/* ── Main wrapper ── */
.main-wrapper {
  margin-left: 240px;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  /* Animate margin when sidebar collapses/expands */
  transition: margin-left 0.25s ease;
}

.main-content {
  flex: 1;
  max-width: 1400px;
  width: 100%;
  margin: 0 auto;
  padding: 1.5rem 2rem;
}

/* ── Page headers ── */
.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-family: 'Fredoka One', cursive;
  font-size: 2rem;
  font-weight: 400;
  color: #2e1065;
  margin-bottom: 0.25rem;
  letter-spacing: 0.01em;
}

.page-header p {
  color: #4b5563;
  font-size: 0.938rem;
}

/* ── Stats grid ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 1.25rem;
  border-radius: 18px;
  border: 1px solid #ede9fe;
  box-shadow: 0 4px 28px rgba(124,58,237,0.12);
  transition: all 0.2s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 36px rgba(124,58,237,0.18);
}

.stat-label {
  color: #4b5563;
  font-size: 0.813rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 800;
  color: #2e1065;
  letter-spacing: -0.025em;
}

/* Accessible semantic color variants — all pass WCAG AA on white */
.stat-card.success .stat-value    { color: #059669; }
.stat-card.warning .stat-value    { color: #d97706; }
.stat-card.danger .stat-value     { color: #e11d48; }
.stat-card.info .stat-value       { color: #0891b2; }
.stat-card.restocking .stat-value { color: #0284c7; }

/* ── Cards ── */
.card {
  background: white;
  border-radius: 18px;
  padding: 1.25rem;
  border: 1px solid #ede9fe;
  box-shadow: 0 4px 28px rgba(124,58,237,0.12);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #ede9fe;
}

.card-title {
  font-family: 'Fredoka One', cursive;
  font-size: 1.25rem;
  font-weight: 400;
  color: #111827;
  letter-spacing: 0.01em;
}

/* ── Tables ── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f5f0ff;
  border-top: 1px solid #ede9fe;
  border-bottom: 2px solid #ede9fe;
}

th {
  text-align: left;
  padding: 0.625rem 0.75rem;
  font-weight: 700;
  /* Dark blue — passes AA on light table header bg */
  color: #7c3aed;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.625rem 0.75rem;
  border-top: 1px solid #ede9fe;
  color: #111827;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  /* Very light blue — visible but not distracting */
  background: #f5f0ff;
}

/* ── Badges ── */
.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

/* All badge color pairs chosen for WCAG AA contrast and color-blind safety */
.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fef3c7; color: #92400e; }
.badge.danger     { background: #ffe4e6; color: #9f1239; }
.badge.info       { background: #e0f2fe; color: #164e63; }
.badge.restocking { background: #e0f2fe; color: #075985; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #ffe4e6; color: #9f1239; }
.badge.stable     { background: #ede9fe; color: #5b21b6; }
.badge.high       { background: #fee2e2; color: #7f1d1d; }
.badge.medium     { background: #fef3c7; color: #78350f; }
.badge.low        { background: #e0f2fe; color: #0c4a6e; }

/* ── Loading / Error ── */
.loading {
  text-align: center;
  padding: 3rem;
  color: #4b5563;
  font-size: 0.938rem;
  font-weight: 600;
}

.error {
  background: #fee2e2;
  border: 1px solid #fca5a5;
  color: #7f1d1d;
  padding: 1rem;
  border-radius: 12px;
  margin: 1rem 0;
  font-size: 0.938rem;
  font-weight: 600;
}
</style>
