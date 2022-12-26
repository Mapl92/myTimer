<template>
  <div class="flex flex-col items-center">
     <div v-text="title + '|'" class="blinking-cursor inline bg-gradient-to-r from-indigo-200 via-sky-400 to-indigo-200 bg-clip-text font-display text-5xl tracking-tight text-transparent"></div>
    <div class="flex">
      <input v-model="name" list="names" class="border rounded px-2 py-1" />
      <datalist id="names">
        <option v-for="name in names" :key="name">{{ name }}</option>
      </datalist>
    </div>
    <div class="text-4xl font-bold mt-4">{{ formattedTimer }}</div>
    <div class="flex mt-4">
      <button @click="startTimer" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full">Start</button>
      <button @click="stopTimer" class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-full ml-4">Stop</button>
      <button @click="resetHistory" class="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded-full ml-4">Reset History</button>
      <button @click="saveTimer" class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-full ml-4">Save</button>
    </div>
    <div class="mt-4">
      <div v-for="(history, index) in timerHistory" :key="index" class="text-lg font-bold border-b border-gray-600 py-2">
        {{ history.name }}: {{ history.formattedTimer }}
        <!-- Add a delete button here -->
        <button @click="deleteHistory(index)" class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded-full ml-4">Delete</button>
        <button @click="continueTimer(history)" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-2 rounded-full ml-4">Continue</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed} from 'vue';
import { TodoistApi } from "@doist/todoist-api-typescript"
export default {
  setup() {
    // Declare variables
    const timerHistory = ref([]);
    const name = ref('');
    const timer = ref(0);
    const startDate = ref(null);
    const endDate = ref(null);
    let interval = null;
    const api = new TodoistApi('441e3150ec84cd6d3d314f17b8cc9441feeb41d1');

    const names = ref([]); // Declare a names reactive variable


    // Use the Todoist API client to get the project names
    async function getTasksWithLabelZE() {
  const taskList = await api.getTasks();

  // Use filter to create a new array with only the tasks that have a label that contains "ZE"
  const tasksWithLabelZE = taskList.filter(task => task.labels.includes("ZE"));

  // Set the value of names to the list of tasks with label "ZE"
  names.value = tasksWithLabelZE.map(task => task.content);
}

// Call the getTasksWithLabelZE function to populate the names reactive variable
getTasksWithLabelZE();


    // load the timerHistory from the local storage
    const storedTimerHistory = window.localStorage.getItem('timerHistory');
    if (storedTimerHistory) {
      timerHistory.value = JSON.parse(storedTimerHistory);
    }

    const formattedTimer = computed(() => {
      const hours = Math.floor(timer.value / 3600);
      const minutes = Math.floor((timer.value % 3600) / 60);
      const seconds = timer.value % 60;
      return `${hours}:${minutes}:${seconds}`;
    });

    const startDateString = computed(() => {
      return startDate.value?.toLocaleTimeString();
    });
    const endDateString = computed(() => {
      return endDate.value?.toLocaleTimeString();
    });

    function startTimer() {
      startDate.value = new Date();
      interval = setInterval(() => {
        timer.value++;
      }, 1000);
    }

    function stopTimer() {
      endDate.value = new Date();
      clearInterval(interval);
    }

  function resetHistory() {
  timerHistory.value = []; // Clear the timer history array
  window.localStorage.removeItem('timerHistory'); // Remove the timer history from local storage
}

function deleteHistory(index) {
  timerHistory.value.splice(index, 1); // remove the history element at the specified index
  window.localStorage.setItem('timerHistory', JSON.stringify(timerHistory.value)); // update the local storage
}
function continueTimer(history) {
  // Set the timer variables to the values from the history element
  timer.value = 0;
  startDate.value = history.startDate;
  endDate.value = history.endDate;
  name.value = history.name; // set the name variable to the value from 
  // Start the timer
  startTimer();
}

function saveTimer() {
  // Check if the name already exists in the timerHistory
  const existingHistoryIndex = timerHistory.value.findIndex(history => history.name === name.value);

  if (existingHistoryIndex !== -1) {
    // If the name already exists, update the end date and timer of the existing history element
    timerHistory.value[existingHistoryIndex].endDate = endDate.value;
    timerHistory.value[existingHistoryIndex].timer += timer.value;
    // Recompute the computed properties of the existing history element
    timerHistory.value[existingHistoryIndex].formattedTimer = computed(() => {
      const hours = Math.floor(timerHistory.value[existingHistoryIndex].timer / 3600);
      const minutes = Math.floor((timerHistory.value[existingHistoryIndex].timer % 3600) / 60);
      const seconds = timerHistory.value[existingHistoryIndex].timer % 60;
      return `${hours}:${minutes}:${seconds}`;
    }).value;
    timerHistory.value[existingHistoryIndex].endDateString = computed(() => {
      return timerHistory.value[existingHistoryIndex].endDate.toLocaleTimeString();
    }).value;
  } else {
    // If the name does not exist, add a new history element to the timerHistory array
    timerHistory.value.push({
      name: name.value,
      timer: timer.value,
      startDate: startDate.value,
      endDate: endDate.value,
      formattedTimer: formattedTimer.value,
      startDateString: startDateString.value,
      endDateString: endDateString.value
    });
  }

  // Update the local storage
  window.localStorage.setItem('timerHistory', JSON.stringify(timerHistory.value));
  // Reset the timer
  timer.value = 0;
}




    return {
      timerHistory,
      name,
      names,
      timer,
      startDate,
      endDate,
      interval,
      formattedTimer,
      startDateString,
      endDateString,
      deleteHistory,
      startTimer,
      stopTimer,
      resetHistory,
      saveTimer,
      continueTimer,
    };
  },
      data() {
      return {
        title: ''
      }
    },
   // Add the mounted hook here
   mounted() {
  // Add your typing animation code here
  let title = 'Todoist Time Tracker'
  let i = 0

  const interval = setInterval(() => {
    this.title += title[i]
    i++
    if (i === title.length) {
      clearInterval(interval)
      setTimeout(() => {
        this.mounted()
      }, 15000)
    }
  }, 150)
}
};


</script>

<style>
 .blinking-cursor {
    animation: blink 1s infinite;
  }

  @keyframes blink {
    50% {
      visibility: hidden;
    }
  }
  @import '../index.css';

  
</style>
