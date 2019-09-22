<template>
  <v-app>
    <v-content>
      <v-container>
        <v-card class="mx-auto">
          <VueCompareImage :leftImage="leftImg" :rightImage="rightImg" />
          <v-card-text>
            <v-row>
              <v-col cols="12" sm="6">
                <v-text-field label="Left Image URL" :rules="[rules.imageUrlMatch]" v-model="leftImgURL" />
              </v-col>
              <v-col cols="12" sm="6">
                <v-text-field label="Right Image URL" :rules="[rules.imageUrlMatch]" v-model="rightImgURL" />
              </v-col>
              <v-col cols="12" md="7">
                <v-text-field color="warning" label="Share URL" readonly :value="shareURL"></v-text-field>
              </v-col>
              <v-col cols="4" md="2">
                <v-btn block depressed x-large color="warning" @click="doCopyClick">
                  Copy
                  <v-icon dark right v-if="copySuccess">mdi-checkbox-marked-circle</v-icon>
                </v-btn>
              </v-col>
              <v-col cols="8" md="3">
                <v-btn
                  block
                  depressed
                  x-large
                  color="success"
                  :loading="loading"
                  @click="doShorten"
                >
                  Copy as Shorten
                  <v-icon dark right v-if="shortenSuccess">mdi-checkbox-marked-circle</v-icon>
                  <v-icon dark right v-if="shortenFailed">mdi-cancel</v-icon>
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
import defaultLeftImage from "./assets/IMG_2214.jpeg";
import defaultRightImage from "./assets/IMG_2215.jpeg";

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
    loading: false,
    copySuccess: false,
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
    doCopyClick() {
      this.$copyText(this.shareURL).then(() => {
        this.copySuccess = true;
        setTimeout(() => (this.copySuccess = false), 2000);
      });
    },
    doShorten() {
      this.loading = true;
      this.axios
        .post("https://pews.dev/api/shorten", {
          url: this.shareURL
        })
        .then(response => {
          this.loading = false;
          if (response.status === 200 && response.data && response.data.url) {
            this.$copyText(response.data.url).then(
              () => {
                this.shortenSuccess = true;
                setTimeout(() => (this.shortenSuccess = false), 5000);
              },
              () => {
                this.shortenFailed = true;
                setTimeout(() => (this.shortenFailed = false), 5000);
              }
            );
          }
        })
        .catch(() => {
          this.loading = false;
          this.shortenFailed = true;
          setTimeout(() => (this.shortenFailed = false), 5000);
        });
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
