<template>
    <div class="app-shell">
        <div class="ambient ambient-one" aria-hidden="true"></div>
        <div class="ambient ambient-two" aria-hidden="true"></div>
        <div class="page-grid" aria-hidden="true"></div>

        <main class="todo-card">
            <div class="card-shine" aria-hidden="true"></div>

            <header class="app-header">
                <div class="brand">
                    <div class="brand-mark" aria-hidden="true">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                            <path d="M7 12.5l3 3 7-7" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" />
                            <path d="M12 3.5a8.5 8.5 0 110 17 8.5 8.5 0 010-17z" stroke-width="1.8" />
                        </svg>
                    </div>
                    <div>
                        <p class="eyebrow">FOCUS BOARD</p>
                        <h1>Мои задачи</h1>
                        <p class="subtitle">Маленькие шаги ведут к большим результатам.</p>
                    </div>
                </div>

                <button
                    type="button"
                    @click="toggleTheme"
                    class="theme-toggle"
                    :aria-label="isDark ? 'Включить светлую тему' : 'Включить тёмную тему'"
                    :title="isDark ? 'Светлая тема' : 'Тёмная тема'"
                >
                    <svg v-if="isDark" viewBox="0 0 24 24" fill="none" stroke="currentColor" aria-hidden="true">
                        <circle cx="12" cy="12" r="4" stroke-width="2" />
                        <path d="M12 2v2m0 16v2M4.93 4.93l1.42 1.42m11.3 11.3l1.42 1.42M2 12h2m16 0h2M4.93 19.07l1.42-1.42m11.3-11.3l1.42-1.42" stroke-width="2" stroke-linecap="round" />
                    </svg>
                    <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" aria-hidden="true">
                        <path d="M20.2 15.1A8.5 8.5 0 118.9 3.8a7 7 0 0011.3 11.3z" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                    </svg>
                </button>
            </header>

            <section class="summary" aria-label="Статистика задач">
                <div class="summary-item">
                    <span class="status-dot"></span>
                    <span>В работе</span>
                    <strong>{{ activeCount }}</strong>
                </div>
                <div class="summary-divider"></div>
                <div class="summary-item summary-muted">
                    <span>Выполнено</span>
                    <strong>{{ completedCount }}</strong>
                </div>
                <div class="progress-track" :title="`Выполнено ${completionPercent}%`">
                    <span :style="{ width: `${completionPercent}%` }"></span>
                </div>
                <span class="progress-value">{{ completionPercent }}%</span>
            </section>

            <form @submit.prevent="addTodo" class="task-form">
                <label for="new-todo" class="sr-only">Новая задача</label>
                <div class="input-wrap">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" aria-hidden="true">
                        <path d="M12 5v14M5 12h14" stroke-width="2" stroke-linecap="round" />
                    </svg>
                    <input
                        id="new-todo"
                        name="newTodo"
                        v-model="newTodo"
                        @keydown.escape="newTodo = ''"
                        type="text"
                        placeholder="Добавьте новую задачу..."
                        autocomplete="off"
                        ref="inputRef"
                    >
                </div>
                <button type="submit" class="add-button">
                    <span>Добавить</span>
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" aria-hidden="true">
                        <path d="M5 12h14m-5-5l5 5-5 5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                    </svg>
                </button>
            </form>

            <Transition name="message">
                <p v-if="error" class="error-message" role="alert">
                    <span>!</span>{{ error }}
                </p>
            </Transition>

            <nav v-if="todos.length > 0" class="filters" aria-label="Фильтр задач">
                <button
                    v-for="f in filters"
                    :key="f.value"
                    type="button"
                    @click="filter = f.value"
                    :class="{ active: filter === f.value }"
                >
                    {{ f.label }}
                    <span>{{ f.count }}</span>
                </button>
            </nav>

            <ul class="task-list" aria-live="polite">
                <TransitionGroup name="task">
                    <li
                        v-for="todo in filteredTodos"
                        :key="todo.id"
                        class="task-item"
                        :class="{ completed: todo.completed }"
                    >
                        <label class="checkbox-wrap" :for="'todo-' + todo.id">
                            <input
                                :id="'todo-' + todo.id"
                                :name="'todo-' + todo.id"
                                v-model="todo.completed"
                                type="checkbox"
                                class="task-checkbox"
                                :aria-label="todo.completed ? 'Снять отметку' : 'Пометить выполненной'"
                            >
                            <svg viewBox="0 0 16 16" fill="none" stroke="currentColor" aria-hidden="true">
                                <path d="M3.5 8l3 3 6-6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                            </svg>
                        </label>

                        <input
                            v-if="editingId === todo.id"
                            v-model="editText"
                            @keyup.enter="saveEdit(todo)"
                            @keyup.esc="cancelEdit"
                            @blur="saveEdit(todo)"
                            :ref="el => { if (el) editInputRef = el; }"
                            type="text"
                            class="edit-input"
                            aria-label="Редактировать задачу"
                        >
                        <span
                            v-else
                            @dblclick="startEdit(todo)"
                            class="task-text"
                            title="Двойной клик для редактирования"
                        >
                            {{ todo.text }}
                        </span>

                        <button
                            type="button"
                            @click="removeTodo(todo.id)"
                            class="delete-button"
                            :aria-label="`Удалить задачу: ${todo.text}`"
                            title="Удалить"
                        >
                            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" aria-hidden="true">
                                <path d="M4 7h16m-10 4v6m4-6v6M9 7l1-3h4l1 3m3 0l-1 13H7L6 7" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
                            </svg>
                        </button>
                    </li>
                </TransitionGroup>
            </ul>

            <div v-if="todos.length === 0" class="empty-state">
                <div class="empty-illustration" aria-hidden="true">
                    <span class="empty-orbit orbit-one"></span>
                    <span class="empty-orbit orbit-two"></span>
                    <svg viewBox="0 0 48 48" fill="none" stroke="currentColor">
                        <rect x="12" y="9" width="24" height="31" rx="6" stroke-width="2.5" />
                        <path d="M19 9.5V8a5 5 0 0110 0v1.5M18 24l4 4 8-9" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" />
                    </svg>
                </div>
                <h2>Здесь пока тихо</h2>
                <p>Добавьте первую задачу — и начните двигаться вперёд.</p>
                <span class="empty-hint">Нажмите Enter для быстрого добавления</span>
            </div>

            <footer v-else class="task-footer">
                <p>
                    <span v-if="activeCount">Ещё немного — осталось {{ activeCount }}</span>
                    <span v-else>Всё готово! Отличная работа ✨</span>
                </p>
                <button
                    v-if="completedCount > 0"
                    type="button"
                    @click="clearCompleted"
                    class="clear-button"
                >
                    Очистить выполненные
                </button>
            </footer>
        </main>

        <p class="page-note">Планируйте спокойно · Выполняйте уверенно</p>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue';

const STORAGE_KEY = 'vue-todo-list';
const THEME_KEY = 'vue-todo-theme';

const newTodo = ref('');
const error = ref('');
const todos = ref([]);
const filter = ref('all');
const editingId = ref(null);
const editText = ref('');
const isDark = ref(false);
const inputRef = ref(null);
let editInputRef = null;

const activeCount = computed(() => todos.value.filter(t => !t.completed).length);
const completedCount = computed(() => todos.value.filter(t => t.completed).length);
const completionPercent = computed(() => (
    todos.value.length ? Math.round((completedCount.value / todos.value.length) * 100) : 0
));

const filters = computed(() => [
    { value: 'all', label: 'Все', count: todos.value.length },
    { value: 'active', label: 'Активные', count: activeCount.value },
    { value: 'completed', label: 'Готовые', count: completedCount.value },
]);

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
    if (text && text !== todo.text) todo.text = text;
    editingId.value = null;
    editText.value = '';
};

const cancelEdit = () => {
    editingId.value = null;
    editText.value = '';
};

const applyTheme = () => {
    document.documentElement.classList.toggle('dark', isDark.value);
};

const toggleTheme = () => {
    isDark.value = !isDark.value;
    applyTheme();
    localStorage.setItem(THEME_KEY, isDark.value ? 'dark' : 'light');
};

watch(newTodo, () => {
    if (error.value) error.value = '';
});

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
:root {
    --page-bg: #f4f6fb;
    --card-bg: rgba(255, 255, 255, 0.88);
    --surface: #f7f8fc;
    --surface-hover: #f1f3f9;
    --text: #182033;
    --muted: #7b849b;
    --border: rgba(120, 130, 160, 0.16);
    --primary: #6c5ce7;
    --primary-strong: #5546d9;
    --primary-soft: #eeeafe;
    --cyan: #26c6da;
    --danger: #ef5b70;
    --shadow: 0 28px 80px rgba(50, 45, 100, 0.14), 0 8px 24px rgba(31, 38, 63, 0.06);
}

.dark {
    --page-bg: #0e1120;
    --card-bg: rgba(24, 28, 48, 0.9);
    --surface: #20253c;
    --surface-hover: #272d48;
    --text: #f2f3fb;
    --muted: #9ca5bf;
    --border: rgba(181, 190, 225, 0.12);
    --primary: #8b7cf6;
    --primary-strong: #a397ff;
    --primary-soft: rgba(121, 103, 238, 0.16);
    --shadow: 0 32px 90px rgba(0, 0, 0, 0.34), 0 8px 24px rgba(0, 0, 0, 0.18);
}

* { box-sizing: border-box; }

button, input { font: inherit; }
button { -webkit-tap-highlight-color: transparent; }

.app-shell {
    position: relative;
    min-height: 100vh;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 18px;
    padding: 48px 20px 28px;
    color: var(--text);
    background:
        radial-gradient(circle at 12% 15%, rgba(108, 92, 231, 0.11), transparent 28%),
        radial-gradient(circle at 88% 85%, rgba(38, 198, 218, 0.11), transparent 28%),
        var(--page-bg);
    transition: background-color .4s ease, color .3s ease;
}

.page-grid {
    position: absolute;
    inset: 0;
    opacity: .35;
    background-image: linear-gradient(var(--border) 1px, transparent 1px), linear-gradient(90deg, var(--border) 1px, transparent 1px);
    background-size: 48px 48px;
    mask-image: radial-gradient(circle at center, black, transparent 72%);
    pointer-events: none;
}

.ambient {
    position: absolute;
    border-radius: 999px;
    filter: blur(2px);
    opacity: .72;
    pointer-events: none;
    animation: drift 10s ease-in-out infinite alternate;
}

.ambient-one {
    width: 280px;
    height: 280px;
    top: -120px;
    right: 8%;
    background: linear-gradient(135deg, rgba(108, 92, 231, .22), rgba(169, 155, 255, .04));
}

.ambient-two {
    width: 220px;
    height: 220px;
    bottom: -100px;
    left: 8%;
    background: linear-gradient(135deg, rgba(38, 198, 218, .2), rgba(105, 240, 174, .04));
    animation-delay: -4s;
}

.todo-card {
    position: relative;
    z-index: 1;
    width: min(100%, 680px);
    overflow: hidden;
    padding: 34px;
    border: 1px solid var(--border);
    border-radius: 28px;
    background: var(--card-bg);
    box-shadow: var(--shadow);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    transition: background-color .3s ease, border-color .3s ease, box-shadow .3s ease;
}

.card-shine {
    position: absolute;
    top: 0;
    left: 12%;
    width: 76%;
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(255,255,255,.85), transparent);
}

.app-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 24px;
    margin-bottom: 25px;
}

.brand { display: flex; align-items: center; gap: 15px; min-width: 0; }

.brand-mark {
    display: grid;
    flex: 0 0 auto;
    width: 49px;
    height: 49px;
    place-items: center;
    color: white;
    border-radius: 16px;
    background: linear-gradient(135deg, var(--primary), #8d7df6);
    box-shadow: 0 10px 24px rgba(108, 92, 231, .28);
    transform: rotate(-3deg);
}

.brand-mark svg { width: 29px; height: 29px; }
.eyebrow { margin: 0 0 3px; color: var(--primary); font-size: 10px; font-weight: 800; letter-spacing: .19em; }
.app-header h1 { margin: 0; font-size: clamp(25px, 4vw, 31px); line-height: 1.1; font-weight: 800; letter-spacing: -.035em; }
.subtitle { margin: 6px 0 0; color: var(--muted); font-size: 13px; line-height: 1.45; }

.theme-toggle {
    display: grid;
    flex: 0 0 auto;
    width: 42px;
    height: 42px;
    place-items: center;
    color: var(--muted);
    border: 1px solid var(--border);
    border-radius: 13px;
    background: var(--surface);
    cursor: pointer;
    transition: transform .2s ease, color .2s ease, background .2s ease, box-shadow .2s ease;
}

.theme-toggle:hover { color: var(--primary); transform: translateY(-2px) rotate(4deg); box-shadow: 0 8px 18px rgba(45, 48, 80, .1); }
.theme-toggle svg { width: 20px; height: 20px; }

.summary {
    display: flex;
    align-items: center;
    gap: 11px;
    min-height: 44px;
    margin-bottom: 18px;
    padding: 0 14px;
    color: var(--muted);
    border: 1px solid var(--border);
    border-radius: 14px;
    background: rgba(127, 132, 160, .055);
    font-size: 12px;
}

.summary-item { display: flex; align-items: center; gap: 7px; white-space: nowrap; }
.summary-item strong { color: var(--text); font-size: 13px; }
.summary-divider { width: 1px; height: 15px; background: var(--border); }
.status-dot { width: 7px; height: 7px; border-radius: 50%; background: #38cf90; box-shadow: 0 0 0 4px rgba(56, 207, 144, .12); }
.progress-track { flex: 1; min-width: 45px; height: 5px; overflow: hidden; border-radius: 99px; background: var(--border); }
.progress-track span { display: block; height: 100%; border-radius: inherit; background: linear-gradient(90deg, var(--primary), var(--cyan)); transition: width .45s cubic-bezier(.22,1,.36,1); }
.progress-value { width: 32px; text-align: right; font-weight: 700; color: var(--primary); }

.task-form {
    display: flex;
    gap: 10px;
    padding: 7px;
    border: 1px solid var(--border);
    border-radius: 17px;
    background: var(--surface);
    transition: border-color .2s ease, box-shadow .2s ease, background .3s ease;
}

.task-form:focus-within { border-color: rgba(108, 92, 231, .46); box-shadow: 0 0 0 4px rgba(108, 92, 231, .09); }
.input-wrap { display: flex; flex: 1; min-width: 0; align-items: center; gap: 10px; padding-left: 9px; }
.input-wrap svg { flex: 0 0 auto; width: 19px; height: 19px; color: var(--primary); }
.input-wrap input { width: 100%; min-width: 0; padding: 10px 0; color: var(--text); border: 0; outline: 0; background: transparent; font-size: 14px; }
.input-wrap input:focus-visible { outline: 0; }
.input-wrap input::placeholder { color: var(--muted); opacity: .82; }

.add-button {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    min-height: 43px;
    padding: 0 17px;
    color: white;
    border: 0;
    border-radius: 12px;
    background: linear-gradient(135deg, var(--primary), #7c69ef);
    box-shadow: 0 7px 18px rgba(108, 92, 231, .25);
    cursor: pointer;
    font-size: 13px;
    font-weight: 700;
    transition: transform .2s ease, box-shadow .2s ease, filter .2s ease;
}

.add-button:hover { transform: translateY(-1px); filter: brightness(1.06); box-shadow: 0 10px 22px rgba(108, 92, 231, .32); }
.add-button:active { transform: translateY(1px) scale(.98); }
.add-button svg { width: 16px; height: 16px; transition: transform .2s ease; }
.add-button:hover svg { transform: translateX(2px); }

.error-message { display: flex; align-items: center; gap: 8px; margin: 10px 5px 0; color: var(--danger); font-size: 12px; }
.error-message span { display: grid; width: 17px; height: 17px; place-items: center; color: white; border-radius: 50%; background: var(--danger); font-size: 10px; font-weight: 800; }

.filters { display: flex; gap: 6px; margin: 20px 0 14px; padding: 4px; border-radius: 13px; background: var(--surface); }
.filters button { display: flex; flex: 1; align-items: center; justify-content: center; gap: 7px; padding: 8px 10px; color: var(--muted); border: 0; border-radius: 10px; background: transparent; cursor: pointer; font-size: 12px; font-weight: 650; transition: all .2s ease; }
.filters button:hover { color: var(--text); }
.filters button.active { color: var(--primary); background: var(--card-bg); box-shadow: 0 3px 10px rgba(40, 42, 70, .07); }
.filters button span { display: grid; min-width: 20px; height: 20px; padding: 0 5px; place-items: center; border-radius: 7px; background: var(--primary-soft); font-size: 10px; }

.task-list { display: flex; flex-direction: column; gap: 8px; margin: 0; padding: 0; list-style: none; }
.task-item { display: flex; align-items: center; gap: 12px; min-height: 54px; padding: 9px 10px 9px 13px; border: 1px solid var(--border); border-radius: 15px; background: rgba(127, 132, 160, .035); transition: transform .2s ease, background .2s ease, border-color .2s ease, opacity .2s ease; }
.task-item:hover { transform: translateX(3px); border-color: rgba(108, 92, 231, .24); background: var(--surface-hover); }
.task-item.completed { opacity: .7; }

.checkbox-wrap { position: relative; display: grid; flex: 0 0 auto; width: 23px; height: 23px; place-items: center; cursor: pointer; }
.task-checkbox { width: 23px; height: 23px; margin: 0; appearance: none; border: 1.5px solid rgba(123, 132, 155, .45); border-radius: 8px; background: transparent; cursor: pointer; transition: all .2s ease; }
.task-checkbox:hover { border-color: var(--primary); }
.task-checkbox:checked { border-color: transparent; background: linear-gradient(135deg, var(--primary), #8272f1); box-shadow: 0 5px 12px rgba(108, 92, 231, .22); }
.checkbox-wrap svg { position: absolute; width: 14px; height: 14px; color: white; opacity: 0; transform: scale(.5) rotate(-15deg); pointer-events: none; transition: all .2s ease; }
.checkbox-wrap:has(.task-checkbox:checked) svg { opacity: 1; transform: scale(1) rotate(0); }

.task-text { flex: 1; min-width: 0; overflow-wrap: anywhere; color: var(--text); font-size: 14px; line-height: 1.45; cursor: text; transition: color .2s ease; }
.completed .task-text { color: var(--muted); text-decoration: line-through; text-decoration-color: rgba(123, 132, 155, .55); }
.edit-input { flex: 1; min-width: 0; padding: 6px 9px; color: var(--text); border: 1px solid var(--primary); border-radius: 8px; outline: 0; background: var(--card-bg); font-size: 14px; box-shadow: 0 0 0 3px var(--primary-soft); }

.delete-button { display: grid; flex: 0 0 auto; width: 34px; height: 34px; place-items: center; color: var(--muted); border: 0; border-radius: 10px; background: transparent; cursor: pointer; opacity: 0; transform: translateX(4px); transition: all .2s ease; }
.task-item:hover .delete-button, .delete-button:focus-visible { opacity: 1; transform: translateX(0); }
.delete-button:hover { color: var(--danger); background: rgba(239, 91, 112, .1); }
.delete-button svg { width: 18px; height: 18px; }

.empty-state { display: flex; flex-direction: column; align-items: center; padding: 42px 20px 22px; text-align: center; }
.empty-illustration { position: relative; display: grid; width: 92px; height: 92px; margin-bottom: 17px; place-items: center; color: var(--primary); border-radius: 30px; background: linear-gradient(145deg, var(--primary-soft), rgba(38, 198, 218, .1)); animation: float 4s ease-in-out infinite; }
.empty-illustration svg { width: 49px; height: 49px; }
.empty-orbit { position: absolute; border-radius: 50%; background: var(--cyan); box-shadow: 0 0 0 5px rgba(38, 198, 218, .1); }
.orbit-one { top: 10px; right: 7px; width: 7px; height: 7px; }
.orbit-two { bottom: 12px; left: 3px; width: 5px; height: 5px; background: var(--primary); }
.empty-state h2 { margin: 0 0 7px; font-size: 18px; letter-spacing: -.02em; }
.empty-state p { max-width: 330px; margin: 0; color: var(--muted); font-size: 13px; line-height: 1.55; }
.empty-hint { margin-top: 15px; padding: 6px 10px; color: var(--muted); border: 1px solid var(--border); border-radius: 8px; font-size: 10px; }

.task-footer { display: flex; align-items: center; justify-content: space-between; gap: 15px; margin-top: 17px; padding-top: 17px; border-top: 1px solid var(--border); }
.task-footer p { margin: 0; color: var(--muted); font-size: 12px; }
.clear-button { padding: 6px 9px; color: var(--danger); border: 0; border-radius: 8px; background: transparent; cursor: pointer; font-size: 11px; font-weight: 650; transition: background .2s ease; }
.clear-button:hover { background: rgba(239, 91, 112, .09); }
.page-note { position: relative; z-index: 1; margin: 0; color: var(--muted); font-size: 10px; letter-spacing: .08em; text-transform: uppercase; opacity: .72; }

.task-enter-active, .task-leave-active { transition: all .32s cubic-bezier(.22,1,.36,1); }
.task-enter-from { opacity: 0; transform: translateY(-8px) scale(.97); }
.task-leave-to { opacity: 0; transform: translateX(24px) scale(.97); }
.task-move { transition: transform .32s ease; }
.message-enter-active, .message-leave-active { transition: all .2s ease; }
.message-enter-from, .message-leave-to { opacity: 0; transform: translateY(-5px); }

@keyframes float { 0%, 100% { transform: translateY(0) rotate(-1deg); } 50% { transform: translateY(-7px) rotate(1deg); } }
@keyframes drift { from { transform: translate3d(-10px, -5px, 0) scale(1); } to { transform: translate3d(16px, 13px, 0) scale(1.08); } }

@media (max-width: 600px) {
    .app-shell { justify-content: flex-start; padding: 20px 12px; }
    .todo-card { padding: 23px 17px; border-radius: 23px; }
    .app-header { gap: 12px; }
    .brand-mark { width: 43px; height: 43px; border-radius: 14px; }
    .subtitle { max-width: 230px; }
    .summary-muted, .summary-divider { display: none; }
    .task-form { gap: 6px; }
    .add-button { width: 45px; padding: 0; }
    .add-button span { display: none; }
    .input-wrap { padding-left: 6px; }
    .filters button { padding-inline: 5px; }
    .delete-button { opacity: 1; transform: none; }
    .task-footer { align-items: flex-start; flex-direction: column; gap: 7px; }
}

@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { scroll-behavior: auto !important; animation-duration: .01ms !important; animation-iteration-count: 1 !important; transition-duration: .01ms !important; }
}
</style>
