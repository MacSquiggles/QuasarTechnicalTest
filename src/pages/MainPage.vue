<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <PestAnnotator />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import PestAnnotator from "../components/PestAnnotator.vue";

export default defineComponent({
  name: "MainPage",

  components: {
    PestAnnotator,
  },
});
</script>
