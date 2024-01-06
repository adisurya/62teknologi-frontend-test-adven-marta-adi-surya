<template>
  <q-page>
    <div class="row q-pa-md">
      <div class="col">
        <q-input v-model="term" label="Search" :dense="dense">
          <template v-slot:before>
            <q-btn round dense flat icon="filter_list" @click="toggleFilter" />
          </template>

          <template v-slot:append>
            <q-icon
              v-if="term !== ''"
              name="close"
              @click="clear"
              class="cursor-pointer"
            />
          </template>

          <template v-slot:after>
            <q-btn round dense flat icon="search" @click="getBusinesses" />
          </template>
        </q-input>
      </div>
    </div>
    <div class="q-pa-md row items-start q-gutter-md">
      <q-card v-for="business in businesses" :key="business.id" class="my-card">
        <q-img :src="business.image_url" />
        <q-card-section>
          <q-btn
            fab
            color="primary"
            icon="place"
            class="absolute"
            style="top: 0; right: 12px; transform: translateY(-50%)"
          />

          <div class="row no-wrap items-center">
            <div class="col text-h6 ellipsis">{{ business.name }}</div>
            <div
              class="col-auto text-grey text-caption q-pt-md row no-wrap items-center"
            >
              <q-icon name="place" />
              {{ Math.round(business.distance) }} m
            </div>
          </div>

          <q-rating v-model="business.rating" :max="5" size="32px" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="text-subtitle2">
            {{ business.price }}・
            <span v-for="(category, index) in business.categories" :key="index">
              {{ category.title }}
              <span v-if="index != business.categories.length - 1">, </span>
            </span>
          </div>
          <div class="text-caption text-grey">
            {{ business.location.display_address.join(" ") }}
          </div>
        </q-card-section>

        <q-separator />

        <q-card-actions>
          <q-btn flat color="primary" :to="'/' + business.id">Detail</q-btn>
        </q-card-actions>
      </q-card>
    </div>
    <div class="row q-pa-lg flex-center">
      <div class="col">&nbsp;</div>
      <div class="col col-auto">
        <q-pagination
          v-model="currentPage"
          :max="totalPage"
          :max-pages="6"
          direction-links
          boundary-links
        />
      </div>
      <div class="col">&nbsp;</div>
    </div>
    <q-drawer v-model="rightDrawerOpen" bordered show-if-above>
      <q-list>
        <q-item-label header> Filter </q-item-label>
        <q-separator spaced></q-separator>
        <q-item>
          <q-item-section>
            <q-input
              v-model="location"
              label="Location"
              :dense="dense"
            ></q-input>
          </q-item-section>
        </q-item>
        <q-item>
          <q-item-section>
            <q-input v-model="radius" label="Radius" :dense="dense"></q-input>
          </q-item-section>
        </q-item>
        <q-item tag="label" v-ripple>
          <q-item-section side top>
            <q-checkbox v-model="openNow" />
          </q-item-section>

          <q-item-section>
            <q-item-label>Open Now</q-item-label>
          </q-item-section>
        </q-item>
        <q-item>
          <q-item-section>
            <q-select
              filled
              v-model="sortBy"
              :options="sortByOptions"
              label="Sort By"
            />
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <MyDialog v-model="showDialog" :message="messageDialog"></MyDialog>
  </q-page>
</template>

<script setup>
import { ref, watch, getCurrentInstance } from "vue";
import MyDialog from "src/components/MyDialog.vue";

import { onMounted } from "vue";
import { useQuasar } from "quasar";

const $q = useQuasar();
const $api = getCurrentInstance().appContext.config.globalProperties.$api;
const showDialog = ref(false);
const messageDialog = ref("");
const businesses = ref([]);
const totalBusinesses = ref(0);
const totalPage = ref(1);
const currentPage = ref(1);
const limit = ref(25);
const rightDrawerOpen = ref(false);
const term = ref("");
const location = ref("NYC");
const radius = ref("");
const openNow = ref(false);
const sortBy = ref({ label: "Best Match", value: "best_match" });
const sortByOptions = [
  { label: "Best Match", value: "best_match" },
  { label: "Rating", value: "rating" },
  { label: "Review Count", value: "review_count" },
  { label: "Distance", value: "distance" },
];
// const categories = ref([]);

function getcategories() {
  console.log("get categories");
  $api
    .get("/categories")
    .then((res) => {
      const { data } = res;
      const cat = data.categories || [];
      categories.value = cat.map((c) => {
        return { label: c.title, value: c.alias };
      });
    })
    .catch((e) => {
      messageDialog.value = e.message;
      showDialog.value = true;
    });
}
function getBusinesses() {
  console.log("get business");
  $q.loading.show({
    delay: 400, // ms
  });
  const offset = (currentPage.value - 1) * limit.value;
  $api
    .get("/businesses/search", {
      params: {
        sort_by: "best_match",
        limit: limit.value,
        location: location.value,
        open_now: 1,
        offset,
        term: term.value,
        radius: radius.value,
        open_now: openNow.value,
        sort_by: sortBy.value.value,
      },
    })
    .then((res) => {
      const { data } = res;
      businesses.value = data.businesses || [];
      totalBusinesses.value = data.total || 0;
      totalPage.value = Math.ceil(data.total / limit.value);
    })
    .catch((e) => {
      messageDialog.value = e.message;
      showDialog.value = true;
    })
    .finally(() => {
      $q.loading.hide();
    });
}

function clear() {
  term.value = "";
  getBusinesses();
}

function toggleFilter() {
  rightDrawerOpen.value = !rightDrawerOpen.value;
}

onMounted(() => {
  // getcategories();
  getBusinesses();
});

watch(currentPage, (newCurrentPage, oldQuestion) => {
  getBusinesses();
});

watch(location, (newLocation, oldLocation) => {
  getBusinesses();
});

watch(radius, (newRadius, oldRadius) => {
  getBusinesses();
});

watch(openNow, (newOpenNow, oldOpenNow) => {
  getBusinesses();
});

watch(sortBy, (newSortBy, oldSortBy) => {
  getBusinesses();
});
</script>
<style scoped>
.my-card {
  width: 100%;
  max-width: 300px;
}
</style>
