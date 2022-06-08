<script setup>
import { ref, onMounted, watch } from "vue";
import axios from "axios";

const loading = ref(true);
const error = ref(null);
const languages = ref(null);
const from = ref("en");
const to = ref("ar");
const text = ref(null);
const translation = ref(null);

const GetLanguages = () => {
  const options = {
    method: "GET",
    headers: {
      "X-RapidAPI-Host": "deep-translate1.p.rapidapi.com",
      "X-RapidAPI-Key": import.meta.env.VITE_RAPID_API_KEY,
    },
  };

  axios
    .request(
      "https://deep-translate1.p.rapidapi.com/language/translate/v2/languages",
      options
    )
    .then((res) => {
      languages.value = res.data.languages;
      loading.value = false;
    })
    .catch((err) => {
      error.value = err;
      loading.value = false;
    });
};

const GetTranslation = (data, source, target) => {
  const options = {
    method: "POST",
    headers: {
      "content-type": "application/json",
      "X-RapidAPI-Host": "deep-translate1.p.rapidapi.com",
      "X-RapidAPI-Key": import.meta.env.VITE_RAPID_API_KEY,
    },
    data: `{"q":"${data}","source":"${source}","target":"${target}"}`,
  };

  axios
    .request(
      "https://deep-translate1.p.rapidapi.com/language/translate/v2",
      options
    )
    .then((res) => {
      translation.value = res.data.data.translations.translatedText;
      loading.value = false;
    })
    .catch((err) => {
      error.value = err;
      loading.value = false;
    });
};

const Translate = () => {
  GetTranslation(text.value, from.value, to.value);
};

const Speech = (check) => {
  let speech;
  if (check) {
    speech = new SpeechSynthesisUtterance(text.value);
    speech.lang = from.value;
  } else {
    speech = new SpeechSynthesisUtterance(translation.value);
    speech.lang = to.value;
  }
  speechSynthesis.speak(speech);
};

const SwitchLanguages = () => {
  const fromFlag = from.value;
  const textFlag = text.value;
  from.value = to.value;
  to.value = fromFlag;
  text.value = translation.value;
  translation.value = textFlag;
};

onMounted(() => {
  GetLanguages();
});

watch(from, (val) => (from.value = val));
watch(to, (val) => (to.value = val));
watch(text, (val) => (text.value = val));
</script>

<template>
  <div class="min-h-screen bg-sky-900 p-5 text-center sm:p-14">
    <h1 class="text-4xl font-bold text-white sm:text-6xl">Translator</h1>

    <div
      class="m-auto my-10 flex w-11/12 flex-col items-center justify-between gap-4 sm:w-4/5 md:flex-row"
    >
      <select
        class="w-full rounded p-2 focus:outline-none md:w-1/4"
        v-model="from"
      >
        <option
          v-for="lang in languages"
          :key="lang.name"
          :value="lang.language"
        >
          {{ lang.name }}
        </option>
      </select>

      <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 512 512"
        class="w-5 cursor-pointer fill-white transition duration-500 ease-in-out hover:fill-blue-300"
        @click="SwitchLanguages"
      >
        <path
          d="M480 256c-17.67 0-32 14.31-32 32c0 52.94-43.06 96-96 96H192L192 344c0-9.469-5.578-18.06-14.23-21.94C169.1 318.3 159 319.8 151.9 326.2l-80 72C66.89 402.7 64 409.2 64 416s2.891 13.28 7.938 17.84l80 72C156.4 509.9 162.2 512 168 512c3.312 0 6.615-.6875 9.756-2.062C186.4 506.1 192 497.5 192 488L192 448h160c88.22 0 160-71.78 160-160C512 270.3 497.7 256 480 256zM160 128h159.1L320 168c0 9.469 5.578 18.06 14.23 21.94C337.4 191.3 340.7 192 343.1 192c5.812 0 11.57-2.125 16.07-6.156l80-72C445.1 109.3 448 102.8 448 95.1s-2.891-13.28-7.938-17.84l-80-72c-7.047-6.312-17.19-7.875-25.83-4.094C325.6 5.938 319.1 14.53 319.1 24L320 64H160C71.78 64 0 135.8 0 224c0 17.69 14.33 32 32 32s32-14.31 32-32C64 171.1 107.1 128 160 128z"
        />
      </svg>
      <select
        class="w-full rounded p-2 focus:outline-none md:w-1/4"
        v-model="to"
      >
        <option
          v-for="lang in languages"
          :key="lang.name"
          :value="lang.language"
        >
          {{ lang.name }}
        </option>
      </select>
    </div>

    <div
      class="m-auto mb-10 flex w-11/12 flex-col justify-between gap-6 md:w-4/5 md:flex-row"
    >
      <div class="relative h-56 w-full md:w-2/5">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 640 512"
          class="absolute right-2 bottom-2 w-5 cursor-pointer fill-blue-600 transition duration-300 ease-in-out hover:fill-blue-800"
          @click="Speech(true)"
        >
          <path
            d="M412.6 182c-10.28-8.334-25.41-6.867-33.75 3.402c-8.406 10.24-6.906 25.35 3.375 33.74C393.5 228.4 400 241.8 400 255.1c0 14.17-6.5 27.59-17.81 36.83c-10.28 8.396-11.78 23.5-3.375 33.74c4.719 5.806 11.62 8.802 18.56 8.802c5.344 0 10.75-1.779 15.19-5.399C435.1 311.5 448 284.6 448 255.1S435.1 200.4 412.6 182zM473.1 108.2c-10.22-8.334-25.34-6.898-33.78 3.34c-8.406 10.24-6.906 25.35 3.344 33.74C476.6 172.1 496 213.3 496 255.1s-19.44 82.1-53.31 110.7c-10.25 8.396-11.75 23.5-3.344 33.74c4.75 5.775 11.62 8.771 18.56 8.771c5.375 0 10.75-1.779 15.22-5.431C518.2 366.9 544 313 544 255.1S518.2 145 473.1 108.2zM534.4 33.4c-10.22-8.334-25.34-6.867-33.78 3.34c-8.406 10.24-6.906 25.35 3.344 33.74C559.9 116.3 592 183.9 592 255.1s-32.09 139.7-88.06 185.5c-10.25 8.396-11.75 23.5-3.344 33.74C505.3 481 512.2 484 519.2 484c5.375 0 10.75-1.779 15.22-5.431C601.5 423.6 640 342.5 640 255.1S601.5 88.34 534.4 33.4zM301.2 34.98c-11.5-5.181-25.01-3.076-34.43 5.29L131.8 160.1H48c-26.51 0-48 21.48-48 47.96v95.92c0 26.48 21.49 47.96 48 47.96h83.84l134.9 119.8C272.7 477 280.3 479.8 288 479.8c4.438 0 8.959-.9314 13.16-2.835C312.7 471.8 320 460.4 320 447.9V64.12C320 51.55 312.7 40.13 301.2 34.98z"
          />
        </svg>
        <textarea
          class="h-full w-full rounded p-3 text-center shadow-lg focus:outline-none"
          v-model="text"
          placeholder="Enter text to translate"
        ></textarea>
      </div>

      <div class="relative h-56 w-full md:w-2/5">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 640 512"
          class="absolute right-2 bottom-2 w-5 cursor-pointer fill-blue-600 transition duration-300 ease-in-out hover:fill-blue-800"
          @click="Speech(false)"
        >
          <path
            d="M412.6 182c-10.28-8.334-25.41-6.867-33.75 3.402c-8.406 10.24-6.906 25.35 3.375 33.74C393.5 228.4 400 241.8 400 255.1c0 14.17-6.5 27.59-17.81 36.83c-10.28 8.396-11.78 23.5-3.375 33.74c4.719 5.806 11.62 8.802 18.56 8.802c5.344 0 10.75-1.779 15.19-5.399C435.1 311.5 448 284.6 448 255.1S435.1 200.4 412.6 182zM473.1 108.2c-10.22-8.334-25.34-6.898-33.78 3.34c-8.406 10.24-6.906 25.35 3.344 33.74C476.6 172.1 496 213.3 496 255.1s-19.44 82.1-53.31 110.7c-10.25 8.396-11.75 23.5-3.344 33.74c4.75 5.775 11.62 8.771 18.56 8.771c5.375 0 10.75-1.779 15.22-5.431C518.2 366.9 544 313 544 255.1S518.2 145 473.1 108.2zM534.4 33.4c-10.22-8.334-25.34-6.867-33.78 3.34c-8.406 10.24-6.906 25.35 3.344 33.74C559.9 116.3 592 183.9 592 255.1s-32.09 139.7-88.06 185.5c-10.25 8.396-11.75 23.5-3.344 33.74C505.3 481 512.2 484 519.2 484c5.375 0 10.75-1.779 15.22-5.431C601.5 423.6 640 342.5 640 255.1S601.5 88.34 534.4 33.4zM301.2 34.98c-11.5-5.181-25.01-3.076-34.43 5.29L131.8 160.1H48c-26.51 0-48 21.48-48 47.96v95.92c0 26.48 21.49 47.96 48 47.96h83.84l134.9 119.8C272.7 477 280.3 479.8 288 479.8c4.438 0 8.959-.9314 13.16-2.835C312.7 471.8 320 460.4 320 447.9V64.12C320 51.55 312.7 40.13 301.2 34.98z"
          />
        </svg>

        <textarea
          class="h-full w-full rounded p-3 text-center shadow-lg focus:outline-none"
          v-model="translation"
          placeholder="Translation"
        ></textarea>
      </div>
    </div>
    <button
      class="m-auto w-4/5 rounded bg-blue-600 p-2 text-white shadow-lg transition duration-500 ease-in-out hover:bg-blue-800 focus:outline-none"
      @click="Translate"
    >
      Translate
    </button>
  </div>
</template>
