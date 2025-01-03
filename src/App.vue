<template>
  <div class="app">
    <div v-if="!showSpinner" class="upload-box">
      <div>
        <input hidden type="file" ref="file" @change="uploadLegacy" />
        <div
          class="file-form"
          @click="$refs.file.click()"
          @dragover.prevent
          @drop.prevent="uploadDrag"
        >
          <span class="drop-files"
            >Drop csv here or click to select a file!</span
          >
        </div>
      </div>
      <div>
        <label for="delete" style="font-size: 24px"
          >Remove Winner after spinning?</label
        >
        <input
          name="delete"
          type="checkbox"
          style="width: 25px; height: 25px"
          v-model="deleteWinner"
        />
      </div>
    </div>
    <div v-else>
      <div class="back-arrow" role="button" @click="back"></div>
      <spinner :names="nameArray" :delete-winner="deleteWinner"></spinner>
    </div>
  </div>
</template>

<script>
import Spinner from "./Spinner.vue";
export default {
  name: "App",
  components: { Spinner },
  data() {
    return {
      showUploadModal: false,
      file: null,
      array: [],
      showSpinner: false,
      deleteWinner: true,
    };
  },
  methods: {
    back() {
      this.showSpinner = false;
      this.array = [];
    },
    upload(file) {
      const reader = new FileReader();
      let text;
      reader.onload = (f) => {
        text = f.target.result;
        let arr = text.split("\n");
        this.array = arr.map((e) => {
          if (e.length > 0) {
            return e.trim().split(",");
          } else return [];
        });
      };
      this.showUploadModal = false;
      this.showSpinner = true;
      reader.readAsText(file);
    },
    uploadDrag(e) {
      this.upload(e.dataTransfer.files[0]);
    },
    uploadLegacy(e) {
      this.upload(e.target.files[0]);
    },
  },
  computed: {
    lastNameIndex() {
      return this.array[0]?.findIndex((e) => e.toLowerCase() === "last name");
    },
    firstNameIndex() {
      return this.array[0]?.findIndex((e) => e.toLowerCase() === "first name");
    },
    nameIndex() {
      return this.array[0]?.length === 1 ? 0 : -1;
    },
    nameArray() {
      if (this.firstNameIndex >= 0 && this.lastNameIndex >= 0) {
        return this.array
          .slice(1)
          .map((e) => `${e[this.firstNameIndex]} ${e[this.lastNameIndex]}`);
      } else if (this.nameIndex >= 0) {
        return this.array.slice(1).map((e) => `${e[this.nameIndex]}`);
      }
      return [];
    },
  },
  created() {
    window.addEventListener("keydown", (e) => {
      if (e.key === "Escape") {
        this.showSpinner = false;
      }
    });
  },
};
</script>

<style>
@font-face {
  font-family: "Sofia";
  src: url("./assets/fonts/sofiapro-regular-webfont.woff2") format("woff2"),
    url("./assets/fonts/sofiapro-regular-webfont.woff") format("woff");
  font-weight: 400;
}

@font-face {
  font-family: "Sofia";
  src: url("./assets/fonts/sofiapro-bold-webfont.woff2") format("woff2"),
    url("./assets/fonts/sofiapro-bold-webfont.woff") format("woff");
  font-weight: 600;
}

@font-face {
  font-family: "Sofia";
  src: url("./assets/fonts/sofiapro-light-webfont.woff2") format("woff2"),
    url("./assets/fonts/sofiapro-light-webfont.woff") format("woff");
  font-weight: 200;
}
body,
html {
  padding: 0;
  margin: 0;
}
.app {
  height: 100vh;
  width: 100%;
  font-family: Sofia, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #111827;
  box-sizing: border-box;
  background-image: url(./assets/background.svg);
  background-size: cover;
  /* background: black; */
}

.upload-box {
  height: 100%;
  width: 100%;
  gap: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.file-form {
  padding: 100px;
  font-size: 48px;
  border: 4px dashed #111111;
}

.back-arrow {
  position: fixed;
  left: 20px;
  top: 20px;
  height: 50px;
  width: 50px;
}

.back-arrow:hover {
  background-image: url(./assets/arrow-circle-left.svg);
  background-size: cover;
  background-position: center;
}
</style>
