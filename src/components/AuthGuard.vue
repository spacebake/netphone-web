<template>
  <div class="auth-guard fill-parent-height">
    <transition name="component-fade" mode="out-in">
      <div class="auth-guard-overlay green-bg" v-if="!isReady">
        <v-container fluid fill-height>
          <v-row justify="center">
            <v-col align="center">
              <v-progress-linear
                style="width: 300px"
                height="13"
                indeterminate
                color="white"
              ></v-progress-linear>
            </v-col>
            <br />
          </v-row>
        </v-container>
      </div>
    </transition>
    <slot v-if="isReady" name="default"> </slot>
    <slot v-if="hasError" name="error">
      <div
        style="
          bottom: -4px;
          right: 14px;
          position: absolute;
          z-index: 10000;
          max-width: 100%;
          width: 300px;
        "
        align="center"
      >
        <v-alert type="error" align="center"> Error! </v-alert>
      </div>
    </slot>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Watch } from 'vue-property-decorator';
import meQuery from '@/gql/me.gql';

@Component
export default class AuthGuard extends Vue {
  private isReady = false;

  private hasError = false;

  private async mounted() {
    this.$auth.initialize();
    this.$auth.on('login', async () => {
      this.guardAauth();
    });

    this.$auth.on('logout', async () => {
      this.guardAauth();
      //   await cache.reset();
      //   await proxies.User.refresh();
    });
  }

  private beforeDestroy() {
    //
  }

  private update() {
    //
  }

  private async guardAauth(): Promise<void> {
    this.isReady = false;
    this.$io.disconnect();
    try {
      await this.$auth.awaitInit();

      const validationVersion = localStorage.getItem('validation');
      if (validationVersion !== '1.0.0') {
        localStorage.setItem('validation', '1.0.0');
        await this.$auth.signOut();
        window.location.reload();
      }

      if (this.$auth.isAuthorized) {
        await this.$apollo.query({
          query: meQuery,
          fetchPolicy: 'network-only',
        });
      }

      // await new Promise((r) => setTimeout(r, 300));

      this.$io.connect(this.$auth.token);

      while (!this.$io.isConnected) {
        await new Promise((resolve) => setTimeout(resolve, 100));
      }
      this.isReady = true;
    } catch (err) {
      console.error(err);
      this.hasError = true;
    }
  }
}
</script>

<style lang="scss" scoped>
.auth-guard {
  // min-height: 100%;
  // overflow: scroll;

  // border: solid 5px red;

  &.--hidden {
    display: none;
  }

  .auth-guard-overlay {
    width: 100%;
    height: 100%;
    position: absolute;
    z-index: 10000;
    top: 0;
  }

  .component-fade-enter-active,
  .component-fade-leave-active {
    transition: opacity 0.8s ease;
  }
  .component-fade-enter, .component-fade-leave-to
/* .component-fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }
}
</style>
