
<script setup>
defineProps({
  search: { type: String, default: '' },
  status: { type: String, default: 'all' },
  priority: { type: String, default: 'all' },
  sort: { type: String, default: 'newest' },
  onlyUnfinished: { type: Boolean, default: false },
})

const emit = defineEmits([
  'update:search',
  'update:status',
  'update:priority',
  'update:sort',
  'update:onlyUnfinished',
  'reset',
])
</script>

<template>
  <section class="tf5-panel" aria-label="Пошук і фільтри">
    <h2>Пошук і впорядкування</h2>

    <div class="tf5-controls-grid">
      <label class="tf5-search">
        Пошук за назвою та описом
        <input
          type="search"
          :value="search"
          placeholder="Наприклад: інтерфейс"
          @input="emit('update:search', $event.target.value)"
        />
      </label>

      <label>
        Статус
        <select
          :value="status"
          @change="emit('update:status', $event.target.value)"
        >
          <option value="all">Усі статуси</option>
          <option value="todo">Заплановано</option>
          <option value="progress">У роботі</option>
          <option value="done">Виконано</option>
        </select>
      </label>

      <label>
        Пріоритет
        <select
          :value="priority"
          @change="emit('update:priority', $event.target.value)"
        >
          <option value="all">Усі пріоритети</option>
          <option value="low">Низький</option>
          <option value="medium">Середній</option>
          <option value="high">Високий</option>
        </select>
      </label>

      <label>
        Сортування
        <select
          :value="sort"
          @change="emit('update:sort', $event.target.value)"
        >
          <option value="newest">Спочатку нові</option>
          <option value="title">За назвою (А–Я)</option>
          <option value="priority">
            За пріоритетом (високий перший)
          </option>
          <option value="dueDate">
            За терміном (найближчий перший)
          </option>
          <option value="status">
            За статусом
          </option>
          <option value="dueDateDesc">
            За терміном (найдальший перший)
          </option>
        </select>
      </label>
    </div>

    <div class="tf5-actions" style="margin-top: 16px">
      <button
        type="button"
        class="tf5-btn"
        @click="emit('reset')"
      >
        Скинути фільтри
      </button>

      <button
        v-if="search.trim()"
        type="button"
        class="tf5-btn"
        @click="emit('update:search', '')"
      >
        Очистити пошук
      </button>

      <button
        type="button"
        class="tf5-btn"
        :aria-pressed="onlyUnfinished"
        @click="emit('update:onlyUnfinished', !onlyUnfinished)"
      >
        {{ onlyUnfinished
          ? 'Показати всі статуси'
          : 'Тільки невиконані' }}
      </button>
    </div>
  </section>
</template>
