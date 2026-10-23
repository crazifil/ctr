
<template>
  <div class="p-4 text-center">
    <div v-if="loading">
      Checking neighbourhood permissions...
    </div>

    <div v-else-if="!canAdmin">
      <div class="text-2xl font-bold text-green">
        Access Denied
      </div>
      <p class="mt-4">
        You do not have permission to update this neighbourhood.
      </p>
    </div>

    <div v-else>
      <div class="text-2xl font-bold text-green">
        Neighbourhood Update
      </div>
      <p class="mt-4">
        Manage the open and closed status of neighbourhood blocks.
      </p>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "NeighborhoodUpdatePage",

  data: () => ({
    loading: true,
    canAdmin: false,
  }),

  methods: {
    
    async loadNeighbourhood() {
      try {
        const response = await this.$http.get(
          `/hood/${this.$route.params.id}`,
        );

        const place = response.data.hood;
        place.hood = response.data.hood;
        place.colony = response.data.colony;

        this.$store.methods.setPlace(place);

        document.title = `${place.name} - Cybertown`;
      } catch (error) {
        console.error("Failed to load neighbourhood", error);
      }
    },

    async checkPermissions() {
      this.loading = true;
      this.canAdmin = false;

      try {
        await this.loadNeighbourhood();

        const response = await this.$http.get(
          `/hood/${this.$route.params.id}/can_admin`,
        );

        this.canAdmin = response.data.result === true;
      } catch (error) {
        this.canAdmin = false;
      } finally {
        this.loading = false;
      }
    },
  },

  mounted() {
    this.checkPermissions();
  },

  watch: {
    "$route.params.id"() {
      this.checkPermissions();
    },
  },
});
</script>
