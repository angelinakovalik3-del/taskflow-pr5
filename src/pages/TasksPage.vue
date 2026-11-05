
<script setup>
import { ref, computed } from 'vue'

import AppTabbar from '../components/AppTabbar.vue'
import TaskCard from '../components/TaskCard.vue'
import TaskForm from '../components/TaskForm.vue'
import TaskListControls from '../components/TaskListControls.vue'

import {
  taskState,
  addTask,
  updateTask,
  deleteTask,
  setTaskStatus,
  setTaskPriority,
} from '../store/tasks'

// Стилі наявного проєкту
import '../css/app.css'
import '../css/task-form.css'
import '../css/task-list.css'

// Пошук, фільтрація та сортування
const searchText = ref('')
const selectedStatus = ref('all')
const selectedPriority = ref('all')
const sortOrder = ref('newest')

// Додаткове завдання 1: тільки невиконані
const onlyUnfinished = ref(false)

// Керування формою створення та редагування
const showForm = ref(false)
const editingId = ref(null)

const editingTask = computed(() =>
  taskState.tasks.find(
    (task) => task.id === editingId.value
  ) ?? null
)

// Додаткове завдання 3:
// Відсоток виконаних завдань
const completedPercent = computed(() => {
  if (taskState.tasks.length === 0) return 0

  const completed = taskState.tasks.filter(
    (task) => task.status === 'done'
  ).length

  return Math.round(
    (completed / taskState.tasks.length) * 100
  )
})

// Пошук, комбінована фільтрація та сортування
const visibleTasks = computed(() => {
  const term = searchText.value
    .trim()
    .toLocaleLowerCase('uk-UA')

  const priorityWeight = {
    high: 3,
    medium: 2,
    low: 1,
  }

  // Додаткове завдання 2:
  // У роботі -> Заплановано -> Виконано
  const statusWeight = {
    progress: 0,
    todo: 1,
    done: 2,
  }

  // Фільтрація створює новий масив
  const result = taskState.tasks.filter((task) => {
    const matchesText =
      !term ||
      `${task.title} ${task.description ?? ''}`
        .toLocaleLowerCase('uk-UA')
        .includes(term)

    const matchesStatus =
      selectedStatus.value === 'all' ||
      task.status === selectedStatus.value

    const matchesPriority =
      selectedPriority.value === 'all' ||
      task.priority === selectedPriority.value

    const matchesUnfinished =
      !onlyUnfinished.value ||
      task.status !== 'done'

    return (
      matchesText &&
      matchesStatus &&
      matchesPriority &&
      matchesUnfinished
    )
  })

  // Сортуємо лише результат filter(),
  // не змінюючи спільний реактивний масив
  return result.sort((a, b) => {

    // Сортування за назвою
    if (sortOrder.value === 'title') {
      return (
        a.title.localeCompare(b.title, 'uk-UA') ||
        a.id - b.id
      )
    }

    // Сортування за пріоритетом
    if (sortOrder.value === 'priority') {
      return (
        priorityWeight[b.priority] -
        priorityWeight[a.priority] ||
        b.id - a.id
      )
    }

    // Додаткове завдання 2:
    // Сортування за статусом
    if (sortOrder.value === 'status') {
      return (
        statusWeight[a.status] -
        statusWeight[b.status] ||
        b.id - a.id
      )
    }

    // Сортування за найближчим терміном
    if (sortOrder.value === 'dueDate') {
      const first = a.dueDate || '9999-12-31'
      const second = b.dueDate || '9999-12-31'

      return (
        first.localeCompare(second) ||
        b.id - a.id
      )
    }

    // Додаткове завдання 6:
    // Зворотне сортування за датою
    if (sortOrder.value === 'dueDateDesc') {
      if (!a.dueDate && !b.dueDate) {
        return b.id - a.id
      }

      if (!a.dueDate) return 1
      if (!b.dueDate) return -1

      return (
        b.dueDate.localeCompare(a.dueDate) ||
        b.id - a.id
      )
    }

    // За замовчуванням: спочатку нові
    return b.id - a.id
  })
})

// Скидання всіх фільтрів
function resetFilters() {
  searchText.value = ''
  selectedStatus.value = 'all'
  selectedPriority.value = 'all'
  sortOrder.value = 'newest'
  onlyUnfinished.value = false
}

// Відкриття форми створення
function startCreate() {
  editingId.value = null
  showForm.value = true
}

// Відкриття форми редагування
function startEdit(id) {
  editingId.value = id
  showForm.value = true
}

// Закриття форми
function cancelForm() {
  editingId.value = null
  showForm.value = false
}

// Створення або оновлення завдання
function saveTask(values) {
  const saved =
    values.id === null
      ? addTask(values)
      : updateTask(values)

  if (saved) {
    cancelForm()
  }
}

// Видалення з підтвердженням
function handleDelete(id) {
  const task = taskState.tasks.find(
    (item) => item.id === id
  )

  if (!task) return

  const confirmed = window.confirm(
    `Видалити завдання «${task.title}»?`
  )

  if (!confirmed) return

  if (deleteTask(id) && editingId.value === id) {
    cancelForm()
  }
}
</script>

<template>
  <f7-page name="tasks" class="tf3-page">
    <f7-navbar title="Мої завдання" />

    <div class="tf3-container">

      <!-- Заголовок сторінки -->
      <section class="tf3-hero">
        <div class="tf3-eyebrow">
          TASKFLOW / TASK MANAGER
        </div>

        <h1>Керування завданнями</h1>

        <p>
          Створюйте, редагуйте, знаходьте,
          сортуйте та видаляйте завдання.
        </p>

        <button
          type="button"
          class="tf5-btn tf5-btn--primary tf5-new"
          @click="startCreate"
        >
          + Додати завдання
        </button>
      </section>

      <!-- Форма створення та редагування -->
      <TaskForm
        v-if="showForm"
        :key="editingId ?? 'new'"
        :task="editingTask"
        @save="saveTask"
        @cancel="cancelForm"
      />

      <!-- Пошук, фільтрація, сортування -->
      <TaskListControls
        v-model:search="searchText"
        v-model:status="selectedStatus"
        v-model:priority="selectedPriority"
        v-model:sort="sortOrder"
        v-model:onlyUnfinished="onlyUnfinished"
        @reset="resetFilters"
      />

      <!-- Список завдань -->
      <section class="tf3-section">

        <div class="tf3-section-heading">

          <div>
            <div class="tf3-eyebrow">
              TASK LIST
            </div>

            <h2>Список завдань</h2>
          </div>

          <!-- Лічильник і відсоток -->
          <div class="tf3-counter">
            Показано:
            {{ visibleTasks.length }}
            із
            {{ taskState.tasks.length }}

            <span class="tf5-percent">
              | Виконано: {{ completedPercent }}%
            </span>
          </div>

        </div>

        <!-- Картки завдань -->
        <div
          v-if="visibleTasks.length"
          class="tf3-task-grid"
        >
          <TaskCard
            v-for="task in visibleTasks"
            :key="task.id"
            :task="task"
            @status-change="setTaskStatus"
            @priority-change="setTaskPriority"
            @edit="startEdit"
            @delete="handleDelete"
          />
        </div>

        <!-- Додаткове завдання 5:
             усі завдання видалені -->
        <div
          v-else-if="taskState.tasks.length === 0"
          class="tf5-empty"
        >
          <p>
            Список завдань порожній.
            Усі завдання видалено.
          </p>

          <button
            type="button"
            class="tf5-btn"
            @click="startCreate"
          >
            Створити перше завдання
          </button>
        </div>

        <!-- Порожній результат пошуку -->
        <div v-else class="tf5-empty">
          <p>
            За поточними умовами
            завдань не знайдено.
          </p>

          <button
            type="button"
            class="tf5-btn"
            @click="resetFilters"
          >
            Показати всі завдання
          </button>
        </div>

      </section>
    </div>

    <!-- Нижня навігація -->
    <AppTabbar active="tasks" />

  </f7-page>
</template>
