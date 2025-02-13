<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const selectedOption = ref(0); // Default to no option selected
const blink = ref(true);

const showSecondaryMenu = ref(false);

const primaryMenuOptions = [
  { id: 1, text: "Boot from Hard Disk" },
  { id: 2, text: "Boot from CD-ROM" },
];

const secondaryMenuOptions = [
  { id: 1, text: "Start Windoge 98 Setup from CD-ROM." },
  { id: 2, text: "Start computer with CD-ROM support." },
  { id: 3, text: "Start computer without CD-ROM support." },
];

const menuOptions = computed(() =>
  showSecondaryMenu.value ? secondaryMenuOptions : primaryMenuOptions,
);

const maxOptionId = computed(() =>
  showSecondaryMenu.value
    ? secondaryMenuOptions.length
    : primaryMenuOptions.length,
);

function selectOption(optionId: number) {
  selectedOption.value = optionId;

  setTimeout(() => {
    if (!showSecondaryMenu.value && optionId === 2) {
      showSecondaryMenu.value = true;
      selectedOption.value = 1;
    } else {
      setTimeout(navigateToLoadingView, 250);
    }
  }, 250);
}

function toggleBlink() {
  if (selectedOption.value === 0) {
    blink.value = !blink.value;
  } else {
    blink.value = false;
  }
}

let intervalId: number | undefined;

onMounted(() => {
  intervalId = window.setInterval(toggleBlink, 500);
  window.addEventListener("keydown", handleKeyDown);
  window.addEventListener("contextmenu", handleContextMenu);
});

onUnmounted(() => {
  if (intervalId !== undefined) {
    window.clearInterval(intervalId);
  }
  window.removeEventListener("keydown", handleKeyDown);
  window.removeEventListener("contextmenu", handleContextMenu);
});

function handleKeyDown(event: KeyboardEvent) {
  const maxId = maxOptionId.value;
  if (event.key === "ArrowUp") {
    selectedOption.value =
      selectedOption.value > 1 ? selectedOption.value - 1 : maxId;
  } else if (event.key === "ArrowDown") {
    selectedOption.value =
      selectedOption.value < maxId ? selectedOption.value + 1 : 1;
  } else if (event.key === "Enter") {
    if (!showSecondaryMenu.value && selectedOption.value === 2) {
      showSecondaryMenu.value = true;
      selectedOption.value = 1; // Reset to the first option in the secondary menu
    } else {
      setTimeout(navigateToLoadingView, 250);
    }
  }
}

function handleContextMenu(event: MouseEvent) {
  event.preventDefault();
}

function navigateToLoadingView() {
  router.push("/loading");
}
</script>

<template>
  <div class="boot-container">
    <div class="boot-loader">
      <h1 v-if="!showSecondaryMenu">Microdao Windoge 98 CD-ROM Startup Menu</h1>
      <h1 v-else>Microdao Windoge 98 Startup Menu</h1>
      <ul>
        <ul
          v-for="option in menuOptions"
          :key="option.id"
          :class="{ selected: selectedOption == option.id }"
          @click="selectOption(option.id)"
        >
          {{
            option.id + ". " + option.text
          }}
        </ul>
      </ul>
      <p>
        Enter your choice:
        <span v-if="selectedOption">{{ selectedOption }}</span>
        <span v-if="blink && selectedOption === 0" class="blink"></span>
      </p>
    </div>
  </div>
</template>

<style scoped>
.boot-container {
  font-family: "Courier New", monospace;
  background-color: black;
  color: rgba(255, 255, 255, 0.75);
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.boot-loader {
  text-align: left;
}

.boot-loader h1 {
  font-size: 32px;
  margin: 0 0 20px 0;
  border-bottom: 6px double rgba(255, 255, 255, 0.75);
  padding-bottom: 8px;
}

.boot-loader ul {
  margin: 0 0 20px 0;
}

.boot-loader ul,
p,
span {
  font-size: 24px;
}

.boot-loader ul.selected {
  color: black;
  background-color: rgba(255, 255, 255, 0.75);
}

.blink {
  display: inline-block;
  width: 10px;
  border-bottom: 2px solid white;
  animation: blinker 1s step-end infinite;
}

@keyframes blinker {
  50% {
    border-bottom: 2px solid transparent;
  }
}
</style>
