<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from "vue";
import timezonesImport from "./timezones";
const searchInput = ref("");
const searchInputIsValidTimezone = ref(false);
const timezones = ref(timezonesImport);

let intervalId: number;

onMounted(() => {
  intervalId = setInterval(() => {
    // This will force Vue to re-render the component every second
    timezones.value = [...timezones.value];
  }, 1000);
});

onUnmounted(() => {
  clearInterval(intervalId);
});

// This will return false if the timezoneId is invalid
const isValidTimezone = (timezoneId: string): boolean => {
  try {
    new Date().toLocaleString(undefined, {
      timeZone: timezoneId,
    });
    return true;
  } catch (error) {
    return false;
  }
};

const getNewYear = (timezoneId: string): Date => {
  const now = new Date().toLocaleString(undefined, { timeZone: timezoneId });
  const date = new Date(now);
  return new Date(date.getFullYear(), 0, 1);
};

const getProgress = (timezoneId: string): number => {
  const newYear = getNewYear(timezoneId);
  const currentTime = new Date().toLocaleString(undefined, {
    timeZone: timezoneId,
  });

  // Next year
  const nextYear = new Date(newYear.getFullYear() + 1, 0, 1);

  // Time until next year in milliseconds
  const timeLeftMilliseconds =
    nextYear.getTime() - new Date(currentTime).getTime();

  // Progress until next year
  const timeLeftDecimal =
    1 - timeLeftMilliseconds / (1000 * 60 * 60 * 24 * 365);

  return timeLeftDecimal;
};

const progress = computed(() =>
  timezones.value.map(timezoneId => getProgress(timezoneId))
);
</script>

<template>
  <main class="container mx-auto py-8">
    <div class="min-h-[100svh] flex flex-col gap-5">
      <div class="">
        <h1 class="font-bold text-4xl">Timezone Racer</h1>
        <h3 class="text-xl">A race to the new year.</h3>
      </div>

      <div class="flex flex-col gap-1">
        <div
          v-for="(timezoneId, index) in timezones"
          :key="index"
          :class="{
            'bg-green-300': progress[index] < 0,
            'bg-blue-300': searchInput.trim() === timezoneId,
          }"
          class="p-1 bg-neutral-100 select-none border-l-4 border-transparent hover:border-black flex justify-between"
        >
          <div>
            {{ timezoneId }} is
            <span
              v-if="progress[index] > 1 || progress[index] < 0"
              class="underline font-medium"
            >
              already in {{ new Date().getFullYear() + 1 }}
            </span>

            <span v-else class="tabular-nums font-medium underline">
              {{
                new Intl.NumberFormat("en", {
                  style: "percent",
                  maximumFractionDigits: 5,
                }).format(progress[index])
              }}
            </span>

            <span v-if="progress[index] < 1 && progress[index] > 0">
              to {{ new Date().getFullYear() + 1 }}
            </span>
          </div>

          <button
            class="mr-2 font-bold hover:underline"
            @click="timezones.splice(index, 1)"
          >
            X
          </button>
        </div>

        <input
          type="text"
          class="p-1 bg-neutral-100 select-none border-l-4 border-transparent outline-none"
          :class="{
            'border-l-transparent': !searchInput,
            'border-l-green-500': searchInputIsValidTimezone,
            'border-l-red-500': !searchInputIsValidTimezone,
            'border-l-blue-300': timezones.find(x => x === searchInput.trim()),
          }"
          placeholder="Add timezone"
          v-model="searchInput"
          @input="
            searchInputIsValidTimezone = isValidTimezone(searchInput.trim())
          "
          @keydown="
            e => {
              if (e.key === 'Enter' && searchInput.trim()) {
                if (
                  !isValidTimezone(searchInput.trim()) ||
                  timezones.find(x => x === searchInput.trim())
                )
                  return;

                timezones.push(searchInput.trim());
                searchInput = '';
              }
            }
          "
        />
      </div>
    </div>
  </main>
</template>
