<template>
  <div class="min-h-screen bg-gray-900 text-green-500 flex flex-col" dir="rtl">
    <div
      class="flex-grow flex items-center justify-center px-4 sm:px-6 lg:px-8"
    >
      <div class="w-full max-w-7xl">
        <h1 class="text-4xl font-bold text-center mb-12">کامنت نکن!</h1>

        <!-- Search input -->
        <div class="mb-8 flex justify-center">
          <UInput
            v-model="q"
            name="q"
            placeholder="لینک ریلز را اینجا وارد کنید..."
            icon="i-heroicons-magnifying-glass-20-solid"
            autocomplete="off"
            :ui="{ icon: { trailing: { pointer: '' } } }"
            class="w-full max-w-md"
            @input="searchCards"
          >
            <template #trailing>
              <UButton
                v-show="q !== ''"
                color="gray"
                variant="link"
                icon="i-heroicons-x-mark-20-solid"
                :padded="false"
                @click="clearSearch"
              />
            </template>
          </UInput>
        </div>

        <!-- Cards grid -->
        <div class="w-full flex justify-center items-center">
          <div
            class="w-[80vw] grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"
          >
            <div
              v-for="card in filteredCards"
              :key="card.id"
              class="bg-gray-800 p-6 rounded-lg shadow-lg hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold mb-3 text-green-400">جواب ریلز</h2>
              <p class="text-sm text-gray-400 mb-4 truncate">
                {{ shortenLink(card.link) }}
              </p>
              <div class="flex justify-between items-center mb-4">
                <UButton
                  size="sm"
                  color="blue"
                  icon="i-heroicons-arrow-top-right-on-square-20-solid"
                  @click="openLink(card.link)"
                >
                  باز کردن لینک
                </UButton>
                <UButton
                  size="sm"
                  :color="card.showAnswer ? 'yellow' : 'gray'"
                  :icon="
                    card.showAnswer
                      ? 'i-heroicons-eye-slash-20-solid'
                      : 'i-heroicons-eye-20-solid'
                  "
                  @click="card.showAnswer = !card.showAnswer"
                >
                  {{ card.showAnswer ? "مخفی کردن پاسخ" : "نمایش پاسخ" }}
                </UButton>
              </div>
              <transition name="fade">
                <p
                  v-if="card.showAnswer"
                  class="text-sm text-white bg-gray-700 p-4 rounded-md mt-4"
                >
                  {{ card.answer }}
                </p>
              </transition>
            </div>
          </div>
        </div>

        <!-- Add New Card Button -->
        <div class="mt-12 text-center">
          <UButton @click="isModalOpen = true" color="green" size="lg"
            >افزودن ریلز جدید</UButton
          >
        </div>
      </div>
    </div>

    <!-- Footer -->
    <div class="mt-12 py-4 text-center text-sm text-gray-400">
      <p>
        قدرت گرفته از
        <a
          href="https://instagram.com/rezasmind"
          class="text-green-500 hover:text-green-400 transition-colors duration-300"
        >
          Rezasmind
        </a>
      </p>
    </div>

    <!-- Modal -->
    <UModal v-model="isModalOpen">
      <UCard :ui="{ base: 'rtl text-right' }">
        <template #header>
          <div class="flex justify-between items-center">
            <div>
              <h3 class="text-lg font-bold">افزودن ریلز جدید</h3>
              <p class="text-sm text-gray-500">
                لطفاً فرم زیر را برای افزودن ریلز جدید پر کنید
              </p>
            </div>
            <UButton
              color="gray"
              variant="ghost"
              icon="i-heroicons-x-mark-20-solid"
              @click="isModalOpen = false"
            />
          </div>
        </template>

        <form @submit.prevent="addNewCard" class="space-y-4">
          <UFormGroup label="لینک ویدیو" required>
            <UInput
              v-model="newCard.link"
              placeholder="https://www.example.com/video..."
            />
          </UFormGroup>
          <UFormGroup label="پاسخ" required>
            <UTextarea v-model="newCard.answer" rows="4" />
          </UFormGroup>
        </form>

        <template #footer>
          <div class="flex justify-start gap-2">
            <UButton type="submit" color="green" @click="addNewCard">
              ثبت
            </UButton>
            <UButton color="gray" variant="soft" @click="isModalOpen = false">
              لغو
            </UButton>
          </div>
        </template>
      </UCard>
    </UModal>
  </div>
</template>

<script setup>
const { $supabase } = useNuxtApp();
const q = ref("");
const isModalOpen = ref(false);
const newCard = ref({ link: "", answer: "" });
const cards = ref([]);
const filteredCards = ref([]);

// Fetch cards from Supabase
async function fetchCards() {
  const { data, error } = await $supabase
    .from("video")
    .select("*")
    .order("created_at", { ascending: false });

  if (error) {
    console.error("خطا در دریافت کارت‌ها:", error);
  } else {
    cards.value = data.map((card) => ({
      ...card,
      showAnswer: false,
    }));
    filteredCards.value = cards.value;
  }
}

// Add new card to Supabase
async function addNewCard() {
  const { data, error } = await $supabase
    .from("video")
    .insert([{ link: newCard.value.link, answer: newCard.value.answer }])
    .select();

  if (error) {
    console.error("خطا در افزودن کارت جدید:", error);
  } else {
    cards.value.unshift({
      ...data[0],
      showAnswer: false,
    });
    filteredCards.value = cards.value;
    newCard.value = { link: "", answer: "" };
    isModalOpen.value = false;
  }
}

// Function to shorten link
function shortenLink(link) {
  try {
    const url = new URL(link);
    return (
      url.hostname +
      url.pathname.slice(0, 15) +
      (url.pathname.length > 15 ? "..." : "")
    );
  } catch (e) {
    return link.slice(0, 30) + (link.length > 30 ? "..." : "");
  }
}

// Function to open link in new window
function openLink(link) {
  window.open(link, "_blank");
}

// Function to search cards
function searchCards() {
  if (q.value.trim() === "") {
    filteredCards.value = cards.value;
  } else {
    filteredCards.value = cards.value.filter((card) =>
      card.link.toLowerCase().includes(q.value.toLowerCase())
    );
  }
}

// Function to clear search
function clearSearch() {
  q.value = "";
  filteredCards.value = cards.value;
}

// Fetch cards on component mount
onMounted(() => {
  fetchCards();
});
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
