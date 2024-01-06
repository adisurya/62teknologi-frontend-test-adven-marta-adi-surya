<template>
  <q-page>
    <div class="row q-pa-md">
      <div class="col">
        <q-carousel animated v-model="slide" arrows navigation infinite>
          <q-carousel-slide
            v-for="(photo, index) in business.photos"
            :key="index"
            :name="index"
            :img-src="photo"
          ></q-carousel-slide>
        </q-carousel>
      </div>
    </div>
    <div class="row q-pa-md">
      <div class="col col-8">
        <h4>{{ business.name }}</h4>
        <q-rating v-model="business.rating" :max="5" size="32px" />
      </div>
      <div class="col">
        <div class="row">
          <div class="col">
            <q-list bordered padding>
              <q-item-label header>Categories</q-item-label>

              <div
                v-for="(category, index) in business.categories"
                :key="category.alias"
              >
                <q-item>
                  <q-item-section>
                    <q-item-label>{{ category.title }}</q-item-label>
                  </q-item-section>
                </q-item>
                <q-separator
                  v-if="index != business.categories.length - 1"
                ></q-separator>
              </div>
            </q-list>
          </div>
        </div>
      </div>
    </div>
    <div class="q-pa-md row items-start q-gutter-md"></div>
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
const $route = getCurrentInstance().appContext.config.globalProperties.$route;

const showDialog = ref(false);
const messageDialog = ref("");
const business = ref({});
const slide = ref(1);

function getBusinessDetail(id) {
  console.log("get business");
  $q.loading.show({
    delay: 400, // ms
  });

  $api
    .get(`/businesses/${id}`)
    .then((res) => {
      const { data } = res;
      business.value = data || {};
      console.log(business.value);
    })
    .catch((e) => {
      messageDialog.value = e.message;
      showDialog.value = true;
    })
    .finally(() => {
      $q.loading.hide();
    });
}

onMounted(() => {
  getBusinessDetail($route.params.id);
});
</script>
<style scoped>
.my-card {
  width: 100%;
  max-width: 300px;
}
</style>
