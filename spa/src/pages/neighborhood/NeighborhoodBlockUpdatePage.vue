
<template>
  <div class="p-4 text-center">
    <div v-if="loading">
      Loading block...
    </div>

    <div v-else-if="!canAdmin">
      <div class="text-2xl font-bold text-green">
        Access Denied
      </div>
      <p class="mt-4">
        You do not have permission to manage this block.
      </p>
    </div>

    <div v-else-if="!block">
      <div class="text-2xl font-bold text-green">
        Block Not Found
      </div>
    </div>

    <div v-else>
      <div class="text-2xl font-bold text-green">
        {{ block.name }}
      </div>

      <p class="mt-4">
        Current status:
        <span :class="block.status === 1 ? 'text-green' : 'text-red-500'">
          {{ block.status === 1 ? "OPEN" : "CLOSED" }}
        </span>
      </p>

      <p class="mt-4">
        Block management controls will be added next.
      </p>

      <router-link
        :to="{ name: 'neighborhoodUpdate', params: { id: $route.params.id } }"
        class="btn-ui inline-block mt-6"
      >
        Back
      </router-link>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

interface Block {
  id: number;
  name: string;
  status: number;
  location: number;
}

export default Vue.extend({
  name: "NeighborhoodBlockUpdatePage",

  data: () => ({
    loading: true,
    canAdmin: false,
    block: null as Block | null,
  }),

  methods: {
    async loadBlock() {
      this.loading = true;
      this.canAdmin = false;
      this.block = null;

      try {
        const hoodId = this.$route.params.id;
        const blockId = Number(this.$route.params.blockId);

        const hoodResponse = await this.$http.get(`/hood/${hoodId}`);

        const place = hoodResponse.data.hood;
        place.hood = hoodResponse.data.hood;
        place.colony = hoodResponse.data.colony;

        this.$store.methods.setPlace(place);

        const permissionResponse = await this.$http.get(
          `/hood/${hoodId}/can_admin`,
        );

        this.canAdmin = permissionResponse.data.result === true;

        if (!this.canAdmin) return;

        const blocksResponse = await this.$http.get(
          `/hood/${hoodId}/blocks`,
        );

        this.block =
          blocksResponse.data.blocks.find(
            (block: Block) => block.id === blockId,
          ) || null;
      } catch (error) {
        console.error("Failed to load block management page", error);
      } finally {
        this.loading = false;
      }
    },
  },

  mounted() {
    this.loadBlock();
  },

  watch: {
    "$route.params.blockId"() {
      this.loadBlock();
    },
    "$route.params.id"() {
      this.loadBlock();
    },
  },
});
</script>
