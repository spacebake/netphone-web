<template>
  <v-container fluid class="fill-parent-height">
    <v-row class="fill-parent-height">
      <v-col cols="2">
        <room-details-sidebar />
      </v-col>
      <v-col>
        <canvas-parent />
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import GqlMe from '@/components/GqlMe.vue';
import CanvasParent from '@/components/Canvas/CanvasParent.vue';
import RoomDetailsSidebar from '@/components/RoomDetailsSidebar.vue';


@Component({
  components: {
    GqlMe,
    CanvasParent,
    RoomDetailsSidebar,
  },
})
export default class Home extends Vue {
  private async mounted() {
    //
  }
}
</script>
