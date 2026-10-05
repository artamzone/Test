<template>
    <div class="bg-gray-100 dark:bg-gray-900 min-h-screen flex items-center justify-center p-4 transition-colors">
        <div class="bg-white dark:bg-gray-800 p-8 rounded-lg shadow-lg w-full max-w-md transition-colors">
            <div class="flex items-center justify-between mb-6">
                <h1 class="text-2xl font-bold text-gray-800 dark:text-gray-100">Мои задачи</h1>
                <button
                    @click="toggleTheme"
                    class="text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200 transition-colors p-1"
                    :aria-label="isDark ? 'Включить светлую тему' : 'Включить тёмную тему'"
                    :title="isDark ? 'Светлая тема' : 'Тёмная тема'"
                >
                    <svg v-if="isDark" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M10 2a1 1 0 011 1v1a1 1 0 11-2 0V3a1 1 0 011-1zm4 8a4 4 0 11-8 0 4 4 0 018 0zm-.464 4.95l.707.707a1 1 0 001.414-1.414l-.707-.707a1 1 0 00-1.414 1.414zm2.12-10.607a1 1 0 010 1.414l-.706.707a1 1 0 11-1.414-1.414l.707-.707a1 1 0 011.414 0zM17 11a1 1 0 100-2h-1a1 1 0 100 2h1zm-7 4a1 1 0 011 1v1a1 1 0 11-2 0v-1a1 1 0 011-1zM5.05 6.464A1 1 0 106.465 5.05l-.708-.707a1 1 0 00-1.414 1.414l.707.707zm1.414 8.486l-.707.707a1 1 0 01-1.414-1.414l.707-.707a1 1 0 011.414 1.414zM4 11a1 1 0 100-2H3a1 1 0 000 2h1z" clip-rule="evenodd" />
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path d="M17.293 13.293A8 8 0 016.707 2.707a8.001 8.001 0 1010.586 10.586z" />
                    </svg>
                </button>
            </div>
            
            <form @submit.prevent="addTodo" class="flex mb-2">
                <label for="new-todo" class="sr-only">Новая задача</label>
                <input 
                    id="new-todo"
                    name="newTodo"
                    v-model="newTodo" 
                    @keydown.escape="newTodo = ''"
                    type="text" 
                    placeholder="Что нужно сделать? Нажмите Enter..." 
                    class="flex-1 px-4 py-2 border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-800 dark:text-gray-100 rounded-l-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                    autocomplete="off"
                    ref="inputRef"
                >
                <button 
                    type="submit"
                    class="bg-blue-500 text-white px-4 py-2 rounded-r-lg hover:bg-blue-600 transition-colors"
                >
                    Добавить
                </button>
            </form>
            <p v-if="error" class="text-red-500 text-sm mb-2">{{ error }}</p>

            <div v-if="todos.length > 0" class="flex gap-2 mb-4 text-sm">
                <button
                    v-for="f in filters"
                    :key="f.value"
                    @click="filter = f.value"
                    :class="[
                        'px-3 py-1 rounded transition-colors',
                        filter === f.value
                            ? 'bg-blue-500 text-white'
                            : 'bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300 hover:bg-gray-300 dark:hover:bg-gray-600'
                    ]"
                >
                    {{ f.label }} ({{ f.count }})
                </button>
            </div>

            <ul class="space-y-2">
                <TransitionGroup name="task">
                    <li 
                        v-for="todo in filteredTodos" 
                        :key="todo.id"
                        class="flex items-center justify-between p-3 bg-gray-50 dark:bg-gray-700 rounded-lg border border-gray-200 dark:border-gray-600 group transition-all hover:bg-gray-100 dark:hover:bg-gray-600"
                    >
                        <div class="flex items-center gap-3 flex-1 min-w-0">
                            <input 
                                :id="'todo-' + todo.id"
                                :name="'todo-' + todo.id"
                                v-model="todo.completed" 
                                type="checkbox" 
                                class="w-4 h-4 text-blue-600 rounded focus:ring-blue-500 cursor-pointer flex-shrink-0"
                                :aria-label="todo.completed ? 'Снять отметку' : 'Пометить выполненной'"
                            >
                            <input
                                v-if="editingId === todo.id"
                                v-model="editText"
                                @keyup.enter="saveEdit(todo)"
                                @keyup.esc="cancelEdit"
                                @blur="saveEdit(todo)"
                                :ref="el => { if (el) editInputRef = el; }"
                                type="text"
                                class="flex-1 px-2 py-0.5 border border-blue-500 rounded focus:outline-none bg-white dark:bg-gray-600 text-gray-800 dark:text-gray-100"
                            >
                            <span 
                                v-else
                                @dblclick="startEdit(todo)"
                                :class="{
                                    'line-through text-gray-400 dark:text-gray-500': todo.completed,
                                    'text-gray-700 dark:text-gray-200': !todo.completed,
                                    'cursor-pointer': true
                                }"
                                class="transition-all break-words flex-1 min-w-0"
                                :title="'Двойной клик для редактирования'"
                            >
                                {{ todo.text }}
                            </span>
                        </div>
                        <button 
                            @click="removeTodo(todo.id)"
                            class="text-red-400 hover:text-red-600 dark:text-red-500 dark:hover:text-red-400 md:opacity-0 md:group-hover:opacity-100 transition-opacity p-1 flex-shrink-0"
                            aria-label="Удалить задачу"
                            title="Удалить"
                        >
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                            </svg>
                        </button>
                    </li>
                </TransitionGroup>
            </ul>

            <div v-if="todos.length === 0" class="text-center text-gray-500 dark:text-gray-400 mt-6">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto mb-2 text-gray-300 dark:text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4" />
                </svg>
                <p>Список задач пуст</p>
                <p class="text-sm mt-1">Напишите первую задачу и нажмите Enter</p>
            </div>

            <div v-else class="mt-4 flex items-center justify-between text-sm text-gray-600 dark:text-gray-400">
                <span>Осталось: <strong class="text-gray-800 dark:text-gray-200">{{ activeCount }}</strong></span>
                <button
                    v-if="completedCount > 0"
                    @click="clearCompleted"
                    class="text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 transition-colors"
                >
                    Очистить выполненные ({{ completedCount }})
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue';

const STORAGE_KEY = 'vue-todo-list';
const THEME_KEY = 'vue-todo-theme';

const newTodo = ref('');
const error = ref('');
const todos = ref([]);
const filter = ref('all'); // all | active | completed
const editingId = ref(null);
const editText = ref('');
const isDark = ref(false);
const inputRef = ref(null);
let editInputRef = null;

const filters = computed(() => [
    { value: 'all', label: 'Все', count: todos.value.length },
    { value: 'active', label: 'Активные', count: activeCount.value },
    { value: 'completed', label: 'Выполненные', count: completedCount.value },
]);

const activeCount = computed(() => todos.value.filter(t => !t.completed).length);
const completedCount = computed(() => todos.value.filter(t => t.completed).length);

const filteredTodos = computed(() => {
    if (filter.value === 'active') return todos.value.filter(t => !t.completed);
    if (filter.value === 'completed') return todos.value.filter(t => t.completed);
    return todos.value;
});

const addTodo = () => {
    const text = newTodo.value.trim();
    if (!text) {
        error.value = 'Введите текст задачи';
        return;
    }
    if (text.length > 200) {
        error.value = 'Слишком длинная задача (макс. 200 символов)';
        return;
    }
    todos.value.push({ id: Date.now() + Math.random(), text, completed: false });
    newTodo.value = '';
    error.value = '';
};

const removeTodo = (id) => {
    todos.value = todos.value.filter(t => t.id !== id);
};

const clearCompleted = () => {
    todos.value = todos.value.filter(t => !t.completed);
};

const startEdit = async (todo) => {
    editingId.value = todo.id;
    editText.value = todo.text;
    await nextTick();
    editInputRef?.focus();
    editInputRef?.select();
};

const saveEdit = (todo) => {
    if (editingId.value !== todo.id) return;
    const text = editText.value.trim();
    if (text && text !== todo.text) {
        todo.text = text;
    }
    editingId.value = null;
    editText.value = '';
};

const cancelEdit = () => {
    editingId.value = null;
    editText.value = '';
};

const toggleTheme = () => {
    isDark.value = !isDark.value;
    applyTheme();
    localStorage.setItem(THEME_KEY, isDark.value ? 'dark' : 'light');
};

const applyTheme = () => {
    if (isDark.value) {
        document.documentElement.classList.add('dark');
    } else {
        document.documentElement.classList.remove('dark');
    }
};

// Watch with error auto-clear
watch(newTodo, () => {
    if (error.value) error.value = '';
});

// Persist todos
watch(todos, (val) => {
    try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(val));
    } catch (e) {
        console.warn('Не удалось сохранить задачи', e);
    }
}, { deep: true });

onMounted(() => {
    try {
        const saved = localStorage.getItem(STORAGE_KEY);
        if (saved) {
            const parsed = JSON.parse(saved);
            if (Array.isArray(parsed)) {
                todos.value = parsed.filter(t => t && typeof t.text === 'string');
            }
        }
        const theme = localStorage.getItem(THEME_KEY);
        if (theme === 'dark' || (!theme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
            isDark.value = true;
        }
        applyTheme();
        inputRef.value?.focus();
    } catch (e) {
        console.warn('Ошибка инициализации', e);
    }
});
</script>

<style>
/* Tailwind v4 uses @import in CSS, dark mode via class strategy */
@custom-variant dark (&:where(.dark, .dark *));

/* TransitionGroup animations */
.task-enter-active,
.task-leave-active {
    transition: all 0.3s ease;
}
.task-enter-from {
    opacity: 0;
    transform: translateX(-20px);
}
.task-leave-to {
    opacity: 0;
    transform: translateX(20px);
}
.task-move {
    transition: transform 0.3s ease;
}
</style>
