<template>
  <v-app>
    <v-content>
      <v-container>
        <v-card class="mx-auto">
          <VueCompareImage :leftImage="leftImg" :rightImage="rightImg" />
          <v-card-text>
            <v-row>
              <v-col cols="12" sm="6">
                <v-text-field
                  label="Left Image URL"
                  :rules="[rules.imageUrlMatch]"
                  v-model="leftImgURL"
                />
              </v-col>
              <v-col cols="12" sm="6">
                <v-text-field
                  label="Right Image URL"
                  :rules="[rules.imageUrlMatch]"
                  v-model="rightImgURL"
                />
              </v-col>
              <v-col cols="12" md="7">
                <v-text-field color="warning" label="Share URL" readonly :value="shareURL"></v-text-field>
              </v-col>
              <v-col cols="4" md="2">
                <v-btn block depressed x-large color="warning" @click="onCopyClick">
                  Copy
                  <v-icon dark right v-if="copySuccess">mdi-checkbox-marked-circle</v-icon>
                  <v-icon dark right v-if="copyFailed">mdi-close</v-icon>
                </v-btn>
              </v-col>
              <v-col cols="8" md="3">
                <v-btn
                  block
                  depressed
                  x-large
                  color="success"
                  :loading="loading"
                  @click="genShortenClick"
                >
                  Shorten Link!
                  <v-icon dark right v-if="shortenSuccess">mdi-checkbox-marked-circle</v-icon>
                  <v-icon dark right v-if="shortenFailed">mdi-close</v-icon>
                </v-btn>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import isImageUrl from "is-image-url";
import VueCompareImage from "vue-compare-image";
import defaultLeftImage from "./assets/IMG_2214.jpg";
import defaultRightImage from "./assets/IMG_2215.jpg";

export default {
  name: "App",
  components: {
    VueCompareImage
  },
  data: () => ({
    sheet: false,
    leftImgURL: "",
    rightImgURL: "",
    shareURL: "",
    shortenURL: "",
    loading: false,
    copySuccess: false,
    copyFailed: false,
    shortenSuccess: false,
    shortenFailed: false,
    rules: {
      imageUrlMatch: val => {
        if (!isImageUrl(val)) {
          return "Invalid Image URL";
        }
        return true;
      }
    }
  }),
  mounted() {
    this.load();
  },
  computed: {
    leftImg() {
      if (!this.leftImgURL) {
        return defaultLeftImage;
      } else {
        return this.leftImgURL;
      }
    },
    rightImg() {
      if (!this.rightImgURL) {
        return defaultRightImage;
      } else {
        return this.rightImgURL;
      }
    }
  },
  methods: {
    onCopyClick() {
      this.$copyText(this.shareURL).then(
        () => {
          this.copySuccess = true;
          setTimeout(() => (this.copySuccess = false), 3000);
        },
        () => {
          this.copyFailed = true;
          setTimeout(() => (this.copyFailed = false), 3000);
        }
      );
    },
    async genShortenClick() {
      this.loading = true;
      let response = {};
      try {
        response = await this.axios.post("https://pews.dev/api/shorten", {
          url: this.shareURL
        });
      } catch (e) {
        this.shortenFailed = true;
        setTimeout(() => (this.shortenFailed = false), 3000);
      }

      this.loading = false;

      if (response.status === 200 && response.data && response.data.url) {
        this.shareURL = response.data.url.replace("http", "https");
        this.shortenSuccess = true;
        setTimeout(() => (this.shortenSuccess = false), 3000);
      } else {
        this.shortenFailed = true;
        setTimeout(() => (this.shortenFailed = false), 3000);
      }
    },
    load() {
      let urlParams = new URLSearchParams(window.location.search);

      let base64_json = urlParams.get("json");
      if (!base64_json) {
        this.shareURL = window.location.href;
        return;
      }

      let decoded_jsonstr = atob(base64_json);
      let params = JSON.parse(decoded_jsonstr);
      if (!params.leftImg || !params.leftImg) {
        return;
      }

      this.leftImgURL = params.leftImg;
      this.rightImgURL = params.rightImg;
    },
    getShareURL() {
      if (!isImageUrl(this.leftImgURL) || !isImageUrl(this.rightImgURL)) {
        return window.location.href;
      }
      let result = "";
      let location = `${window.location.origin}${window.location.pathname}`;
      let param = {
        leftImg: this.leftImgURL,
        rightImg: this.rightImgURL
      };
      let base64_json = btoa(JSON.stringify(param));
      result = `${location}?json=${base64_json}`;
      return result;
    }
  },
  watch: {
    leftImgURL() {
      this.shareURL = this.getShareURL();
    },
    rightImgURL() {
      this.shareURL = this.getShareURL();
    }
  }
};
</script>
