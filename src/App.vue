<script setup>
import { ref, onMounted, watch, computed } from "vue";

const title = ref("today's tasks");
const tasks = ref([]);
const newTask = ref("");
const weekdays = ref([
  { i: 0, name: "Sun", isSelected: false },
  { i: 1, name: "Mon", isSelected: false },
  { i: 2, name: "Tue", isSelected: false },
  { i: 3, name: "Wed", isSelected: false },
  { i: 4, name: "Thu", isSelected: false },
  { i: 5, name: "Fri", isSelected: false },
  { i: 6, name: "Sat", isSelected: false },
]);
const profileImage = ref(
  localStorage.getItem("profileImage") ||
    "https://avatar.iran.liara.run/public/boy"
);
const search = ref("");

const isEditing = ref(false);
const editText = ref("");
const editIndex = ref(null);

const isDarkMode = ref(localStorage.getItem("theme") === "dark");

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value;
  localStorage.setItem("theme", isDarkMode.value ? "dark" : "light");

  if (isDarkMode.value) {
    document.documentElement.classList.add("dark");
  } else {
    document.documentElement.classList.remove("dark");
  }
};

const uploadImage = (e) => {
  const file = e.target.files[0];
  const reader = new FileReader();
  reader.onload = (e) => {
    profileImage.value = e.target.result;
    localStorage.setItem("profileImage", profileImage.value);
  };
  reader.readAsDataURL(file);
};

onMounted(() => {
  if (localStorage.getItem("theme") === "dark") {
    document.documentElement.classList.add("dark");
  }

  const now = new Date();
  const day = now.getDay();
  weekdays.value[day].isSelected = true;

  const lastDayInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0);
  const lastDayInLastMonth = new Date(now.getFullYear(), now.getMonth(), 0);

  let dayNumber = now.getDate();
  weekdays.value[day].dayNum = dayNumber;

  // calc each day in the list
  for (let i = weekdays.value[day].i - 1; i >= 0; i--) {
    weekdays.value[i].dayNum =
      --dayNumber < 1 ? lastDayInLastMonth.getDate() : dayNumber;
  }
  dayNumber = now.getDate();
  for (let i = weekdays.value[day].i + 1; i <= 6; i++) {
    weekdays.value[i].dayNum =
      ++dayNumber > lastDayInMonth.getDate() ? 1 : dayNumber;
  }

  const savedTasks = localStorage.getItem("tasks");
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks);
  }
});

watch(
  tasks,
  (newTasks) => {
    localStorage.setItem("tasks", JSON.stringify(newTasks));
  },
  { deep: true }
);

const addTask = () => {
  if (newTask.value.trim() !== "") {
    const now = new Date();
    const formatedTime = now.toLocaleTimeString("en-US", {
      hour: "numeric",
      minute: "numeric",
      hour12: true,
    });
    tasks.value.push({
      text: newTask.value,
      time: formatedTime,
      completed: false,
    });
    newTask.value = "";
  }
};

const removeTask = (index) => {
  tasks.value.splice(index, 1);
};

const openEditModal = (index) => {
  editText.value = tasks.value[index].text;
  editIndex.value = index;
  isEditing.value = true;
};

const updateTask = () => {
  if (editText.value.trim() !== "") {
    tasks.value[editIndex.value].text = editText.value;
  }
  isEditing.value = false;
};

const filteredTasks = computed(() => {
  if (!search.value.trim()) return tasks.value;
  return tasks.value.filter((task) =>
    task.text.toLowerCase().includes(search.value.toLowerCase())
  );
});
</script>

<template>
  <div
    :class="{ dark: isDarkMode }"
    class="flex items-center justify-center h-screen"
  >
    <div
      class="bg-w-bg rounded-2xl p-10 w-100 shadow-lg h-[700px] flex flex-col border border-w-border dark:bg-d-bg"
    >
      <!-- Theme Toggle Button -->
      <button
        @click="toggleTheme"
        class="absolute top-4 right-4 p-2 rounded-md bg-w-border dark:bg-d-border"
      >
        {{ isDarkMode ? "🌞 Light Mode" : "🌙 Dark Mode" }}
      </button>

      <!-- Profile image -->
      <label for="imageUpload" class="cursor-pointer">
        <img
          :src="profileImage"
          alt="Profile"
          class="w-16 h-16 rounded-full border border-gray-300"
        />
      </label>
      <input
        type="file"
        id="imageUpload"
        class="hidden"
        accept="image/*"
        @change="uploadImage"
      />

      <!-- Weekdays -->
      <div class="flex justify-between mt-4 items-center">
        <div
          v-for="(day, index) in weekdays"
          :key="index"
          class="flex flex-col items-center text-white"
        >
          <div
            :class="{
              'bg-w-gold w-9 h-9': day.isSelected,
              'bg-w-brown w-6 h-6 text-sm': !day.isSelected,
            }"
            class="rounded-full flex items-center justify-center mb-2"
          >
            {{ day.dayNum }}
          </div>
          <div
            :class="{
              'text-w-gold': day.isSelected,
              'text-w-brown': !day.isSelected,
            }"
          >
            {{ day.name }}
          </div>
        </div>
      </div>

      <!-- Search -->
      <div class="relative mt-4">
        <input
          v-model="search"
          type="text"
          placeholder="Search"
          class="max-h-8 w-full px-10 py-2 bg-white rounded-full border border-w-border focus:outline-none text-xs"
        />
        <img
          src="./assets/searchicon.png"
          alt="Search Icon"
          class="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 pointer-events-none"
        />
      </div>

      <!-- Tasks -->
      <h3 class="text-xl font-bold mt-4">{{ title }}</h3>
      <div class="overflow-y-auto flex-1 mt-2 pr-2 max-h-70">
        <div
          v-for="(task, index) in filteredTasks"
          :key="index"
          class="bg-w-bg py-2 px-4 my-2 rounded-2xl border border-w-border flex items-center justify-between"
        >
          <div class="flex items-center space-x-2">
            <label class="flex items-center cursor-pointer relative">
              <input
                type="checkbox"
                v-model="task.completed"
                class="peer h-5 w-5 cursor-pointer transition-all appearance-none hover:shadow-md border border-w-gold checked:bg-w-gold checked:border-white"
              />
              <span
                class="absolute text-white opacity-0 peer-checked:opacity-100 top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 pointer-events-none"
              >
                ✔
              </span>
            </label>
            <span class="ml-2">
              <div class="text-w-border text-xs mb-2">{{ task.time }}</div>
              <div
                :class="{
                  'line-through text-gray-400': task.completed,
                  'text-gray-800': !task.completed,
                }"
                class="text-md"
              >
                {{ task.text }}
              </div>
            </span>
          </div>

          <div class="flex flex-col items-center space-y-2">
            <button @click="openEditModal(index)" class="text-gold">
              <img
                src="./assets/editIcon.png"
                alt="Edit Icon"
                class="w-4 h-4"
              />
            </button>
            <button @click="removeTask(index)" class="text-red-500">
              <img
                src="./assets/deleteIcon.png"
                alt="Delete Icon"
                class="w-4 h-4"
              />
            </button>
          </div>
        </div>
      </div>

      <!-- Add new task -->
      <div class="mt-4 flex">
        <input
          v-model="newTask"
          @keyup.enter="addTask"
          placeholder="Add a new task"
          class="flex-1 px-4 py-2 border border-w-border rounded-l-md focus:outline-none"
        />
        <button
          @click="addTask"
          class="bg-w-gold text-white border border-w-border px-4 py-2 rounded-r-md text-2xl"
        >
          +
        </button>
      </div>
    </div>
  </div>
</template>
