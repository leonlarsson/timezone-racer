<!-- TODO: Get all time data from server instead? -->

<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from "vue";
import { allTimezones, defaultTimezones } from "./timezones";
import { TIME } from "./constants";
const timezoneAddInput = ref("");
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
    new Date().toLocaleString("en", {
      timeZone: timezoneId,
    });
    return true;
  } catch (error) {
    return false;
  }
};

const getNewYear = (timezoneId: string): Date => {
  const now = new Date().toLocaleString("en", { timeZone: timezoneId });
  const date = new Date(now);
  return new Date(date.getFullYear(), 0, 1);
};

const getProgress = (
  timezoneId: string
): {
  currentDate: string;
  progressDecimal: number;
  progressMilliseconds: number;
} => {
  const newYear = getNewYear(timezoneId);

  const currentDate = new Date().toLocaleString("en", {
    timeZone: timezoneId,
  });

  // Progress so far in milliseconds
  const progressMilliseconds =
    new Date(currentDate).getTime() - newYear.getTime();

  // Progress so far in decimal
  const progressDecimal = progressMilliseconds / TIME.YEAR;
  console.log(progressMilliseconds, TIME.YEAR);

  return { currentDate, progressDecimal, progressMilliseconds };
};

const progressData = computed(() =>
  timezones.value
    .map(timezoneId => {
      const { currentDate, progressDecimal, progressMilliseconds } =
        getProgress(timezoneId);

      return {
        timezoneId,
        currentDate,
        progressDecimal,
        progressPercentage: new Intl.NumberFormat("en", {
          style: "percent",
          maximumFractionDigits: 5,
        }).format(progressDecimal),
        progressMilliseconds: progressMilliseconds,
        currentYear: new Date().toLocaleString("en", {
          year: "numeric",
          timeZone: timezoneId,
        }),
        nextYear:
          new Date(
            new Date().toLocaleString("en", {
              year: "numeric",
              timeZone: timezoneId,
            })
          ).getFullYear() + 1,
        lessThanADayHasPassed: progressMilliseconds < TIME.DAY,
      };
    })
    .sort((a, b) => {
      if (a.progressDecimal === b.progressDecimal) {
        return a.timezoneId.localeCompare(b.timezoneId);
      } else {
        return b.progressDecimal - a.progressDecimal;
      }
    })
);
</script>

<template>
  <!-- Local progress -->
  <div
    class="h-1 bg-green-500"
    :style="{
      width:
        progressData.find(
          x => x.timezoneId === Intl.DateTimeFormat().resolvedOptions().timeZone
        )?.progressPercentage ?? '0%',
    }"
  />

  <main class="container mx-auto p-8">
    <div class="min-h-[100svh] flex flex-col gap-3">
      <div>
        <h1 class="font-bold text-4xl">Timezone Racer</h1>
        <h3 class="text-xl">
          A race to the new year. Counter to next year starts after it has been
          a day.
        </h3>
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

      <!-- TODO: Something to display how many timezones are in x years x in year1, y, in year2 etc -->
      <div class="flex gap-x-4 flex-wrap">
        <span v-for="year in ['2024', '2025'].sort()">
          <span class="font-mono">
            {{ progressData.filter(x => x.currentYear === year).length }}/{{
              progressData.length
            }}
          </span>
          in {{ year }}
        </span>
      </div>

      <div class="flex flex-col gap-1">
        <div
          v-for="(
            {
              timezoneId,
              currentDate,
              progressPercentage,
              progressMilliseconds,
              currentYear,
              nextYear,
              lessThanADayHasPassed,
            },
            index
          ) in progressData"
          :key="index"
          :class="{
            '!bg-green-300': progressMilliseconds < TIME.DAY,
            '!bg-blue-300':
              timezoneAddInput.trim().toLowerCase() ===
              timezoneId.toLowerCase(),
          }"
          class="group p-1 bg-neutral-100 border-l-4 border-transparent hover:border-black flex-col justify-between"
        >
          <div class="flex">
            <div class="flex justify-between w-full flex-wrap">
              <div>
                {{ timezoneId }}

                <!-- If less than a day has passed in the current year (we have just hit the new year) -->
                <span
                  v-if="lessThanADayHasPassed"
                  class="underline font-medium"
                >
                  made it to
                  {{ currentYear }}
                </span>

                <!-- If more than a day has passed in the current year (we have not hit the new year) -->
                <span v-else>
                  is
                  <span class="tabular-nums font-medium underline">
                    {{ progressPercentage }}
                  </span>
                </span>

                <span v-if="!lessThanADayHasPassed">
                  to
                  <span class="font-semibold">
                    {{ nextYear }}
                  </span>
                </span>
              </div>

              <span class="tabular-nums">{{ currentDate }}</span>
            </div>

            <button
              class="mx-3 font-bold hover:underline"
              @click="timezones = timezones.filter(x => x !== timezoneId)"
            >
              X
            </button>
          </div>

          <!-- Progress bar -->
          <!-- <div
            class="bg-transparent h-[2px] group-hover:bg-green-500"
            :style="{
              width: new Intl.NumberFormat('en', {
                style: 'percent',
                maximumFractionDigits: 5,
              }).format(progressDecimal),
            }"
          /> -->
        </div>

        <input
          type="text"
          class="p-1 bg-neutral-100 select-none border-l-4 border-transparent outline-none"
          :class="{
            'border-l-transparent': !timezoneAddInput,
            'border-l-green-500': searchInputIsValidTimezone,
            'border-l-red-500': !searchInputIsValidTimezone,
            '!border-l-blue-300': timezones.find(
              x => x.toLowerCase() === timezoneAddInput.trim().toLowerCase()
            ),
          }"
          placeholder="Add timezone"
          v-model="timezoneAddInput"
          @input="
            searchInputIsValidTimezone = isValidTimezone(
              timezoneAddInput.trim()
            )
          "
          @keydown="
            e => {
              if (e.key === 'Enter' && timezoneAddInput.trim()) {
                if (
                  !isValidTimezone(timezoneAddInput.trim()) ||
                  timezones.find(
                    x =>
                      x.toLowerCase() === timezoneAddInput.trim().toLowerCase()
                  )
                )
                  return;

                timezones.push(timezoneAddInput.trim());
                timezoneAddInput = '';
              }
            }
          "
        />
      </div>
    </div>
  </main>
</template>
