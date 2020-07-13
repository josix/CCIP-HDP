<template>
  <div
    class="landing landing-container"
    :class="{ passed: status === true, failed: status === false }"
  >
    <h1>掃描 QR Code<br />以開始填寫健康聲明書</h1>
    <Scanner />
    <div
      v-if="status !== null"
      :class="{ passed: status }"
      class="landing status-icon"
    >
      <font-awesome-icon v-if="status" :icon="['fas', 'check']" />
      <font-awesome-icon v-else :icon="['fas', 'times']" />
    </div>
    <h1>Scan QR Code<br />to start filling the Healty Declaration</h1>
  </div>
</template>

<script lang="ts">
import { Component, Watch, Vue } from "vue-property-decorator";
import { Action, Getter } from "vuex-class";

import { AppState } from "@/store/types/app.type";
import { FormState, FormResponse } from "@/store/types/form.type";

import Scanner from "@/components/Scanner.vue";

@Component({
  components: {
    Scanner
  }
})
export default class Landing extends Vue {
  @Action("setToken", { namespace: "form" })
  private setToken!: (token: FormState["token"]) => void;

  @Action("verifyToken", { namespace: "form" })
  private verifyToken!: (args: {
    endpoint: AppState["endpoint"];
    token: FormState["token"];
  }) => Promise<FormResponse>;

  @Action("resetToken", { namespace: "form" })
  private resetToken!: () => void;

  @Getter("endpoint", { namespace: "app" })
  private endpoint!: string;

  @Getter("token", { namespace: "form" })
  private token!: string;

  @Watch("token")
  private async onTokenChange(token: string) {
    if (token) {
      this.status = null;

      const res = (await this.verifyToken({
        endpoint: this.endpoint,
        token: token
      })) as { fill: string; status: string };

      if (!res.fill) {
        this.$router.push({ name: "Fill" });
      } else if (res.status) {
        this.status = true;
        this.resetScanner();
      } else {
        this.status = false;
        this.resetScanner();
      }
    }
  }

  private mounted() {
    this.detectRouterToken();
  }

  private status: boolean | null = null;

  private resetScanner(): void {
    setTimeout(() => {
      this.status = null;
      this.resetToken();
    }, 2000);
  }

  private detectRouterToken(): void {
    if (this.$route.query && this.$route.query.rtoken) {
      this.setToken(this.$route.query.rtoken as string);
    }
  }
}
</script>