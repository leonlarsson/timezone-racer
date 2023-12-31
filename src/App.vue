<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from "vue";
import { allTimezones, defaultTimezones } from "./timezones";
const searchInput = ref("");
const searchInputIsValidTimezone = ref(false);
const timezones = ref(defaultTimezones);

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

const progressData = computed(() =>
  timezones.value
    .map(timezoneId => ({
      timezoneId,
      progress: getProgress(timezoneId),
      progressComplete: getProgress(timezoneId) < 0,
    }))
    .sort((a, b) => {
      if (a.progressComplete && !b.progressComplete) {
        return -1;
      } else if (!a.progressComplete && b.progressComplete) {
        return 1;
      } else {
        return b.progress - a.progress;
      }
    })
);
</script>

<template>
  <main class="container mx-auto p-8">
    <div class="min-h-[100svh] flex flex-col gap-3">
      <div>
        <h1 class="font-bold text-4xl">Timezone Racer</h1>
        <h3 class="text-xl">A race to the new year.</h3>
      </div>

      <button
        class="bg-neutral-200 border w-fit p-1 hover:bg-neutral-300"
        @click="
          timezones.length > 10
            ? (timezones = defaultTimezones)
            : (timezones = allTimezones)
        "
      >
        {{
          timezones.length > 10
            ? "Show default timezones"
            : "Show all timezones"
        }}
      </button>

      <span
        >{{ progressData.filter(x => x.progressComplete).length }}/{{
          progressData.length
        }}
        are in the new year.</span
      >

      <div class="flex flex-col gap-1">
        <div
          v-for="(
            { timezoneId, progress, progressComplete }, index
          ) in progressData"
          :key="index"
          :class="{
            '!bg-green-300': progressComplete,
            '!bg-blue-300': searchInput.trim() === timezoneId,
          }"
          class="p-1 bg-neutral-100 select-none border-l-4 border-transparent hover:border-black flex justify-between"
        >
          <div>
            {{ timezoneId }} is
            <span
              v-if="progress > 1 || progress < 0"
              class="underline font-medium"
            >
              already in {{ new Date().getFullYear() + 1 }}
            </span>

            <span v-else class="tabular-nums font-medium underline">
              {{
                new Intl.NumberFormat("en", {
                  style: "percent",
                  maximumFractionDigits: 5,
                }).format(progress)
              }}
            </span>

            <span v-if="progress < 1 && progress > 0">
              to {{ new Date().getFullYear() + 1 }}
            </span>
          </div>

          <button
            class="mr-2 font-bold hover:underline"
            @click="timezones = timezones.filter(x => x !== timezoneId)"
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
            '!border-l-blue-300': timezones.find(x => x === searchInput.trim()),
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
