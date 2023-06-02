<script setup lang="ts">
import { onMounted, ref, watch } from "vue";

import PageHeader from "./components/PageHeader.vue";
import CountryList from "./components/CountryList.vue";
import axiosClient from "./utils/axios";
import { Country } from "./models/country.model";

const countries = ref<Country[]>([]);
const search = ref("");
const filteredCountries = ref<Country[]>([]);
const page = ref(1);
const itemsPerPage = ref(12);
const paginatedCountries = ref<Country[]>([]);
const totalItems = ref(0);

const fetchCountries = async () => {
  try {
    const { data } = await axiosClient.get("/all");
    countries.value = data;
    totalItems.value = countries.value.length;
  } catch (error) {
    console.log(error);
  }
};

const filterCountries = () => {
  filteredCountries.value = countries.value.filter((country) =>
    country.name.common.toLowerCase().includes(search.value.toLowerCase())
  );
};

const sliceCountries = (currentCountries: Country[]) => {
  const start = (page.value - 1) * itemsPerPage.value;
  const end = page.value * itemsPerPage.value;
  paginatedCountries.value = currentCountries.slice(start, end);
};

const changePage = (newPage: number) => {
  page.value = newPage;
};

onMounted(() => {
  fetchCountries();
});

watch([countries, page, filteredCountries], () => {
  sliceCountries(
    filteredCountries.value.length <= 0 && search.value === ""
      ? countries.value
      : filteredCountries.value
  );
});
</script>

<template>
  <PageHeader />
  <div class="container max-w-screen-lg mx-auto px-6">
    <div class="mb-8">
      <input
        type="text"
        class="border border-gray-300 rounded w-full p-1 px-4"
        placeholder="Seacrh by country name"
        v-model="search"
        @input="filterCountries"
      />
    </div>
    <div class="mb-8 flex justify-center space-x-6">
      <button
        :disabled="page <= 1"
        :class="{ 'opacity-50': page <= 1 }"
        @click="changePage(page - 1)"
        class="border border-gray-300 rounded px-2 py-0.5 hover:bg-gray-200"
      >
        Previous
      </button>
      <button
        :disabled="page >= totalItems / itemsPerPage"
        :class="{ 'opacity-50': page >= totalItems / itemsPerPage }"
        @click="changePage(page + 1)"
        class="border border-gray-300 rounded px-2 py-0.5 hover:bg-gray-200"
      >
        Next
      </button>
    </div>
    <CountryList :countries="paginatedCountries" />
  </div>
</template>
