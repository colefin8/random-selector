<template>
  <div class="app">
    <div v-if="!showSpinner">
        <input hidden type="file" ref="file" @change="uploadLegacy"/>
        <div class="file-form" @click='$refs.file.click()' @dragover.prevent @drop.prevent="uploadDrag">
          <span class="drop-files">Drop csv here or click to select a file!</span>
        </div>
    </div>
    <spinner :names="nameArray" v-else></spinner>
    <!-- <div v-for="(e, i) in nameArray" :key='i'>
      {{e}}
    </div> -->
  </div>
</template>

<script>
import Spinner from './Spinner.vue'

export default {
  name: 'App',
  components: {Spinner},
  data(){
    return {
      showUploadModal: false,
      file: null,
      array: [],
      showSpinner: false,
    }
  },
  methods: {
    upload(file) {
      const reader = new FileReader()
      let text;
      reader.onload = (f) => {
        text = f.target.result
        let arr = text.split('\n')
        this.array = arr.map(e => {
          if(e.length > 0) {
            return e.trim().split(',')
          } else return []
        })
      }
      this.showUploadModal = false;
      this.showSpinner = true
      reader.readAsText(file)
    },
    uploadDrag(e){
      this.upload(e.dataTransfer.files[0])
    },
    uploadLegacy(e) {
      this.upload(e.target.files[0])
    }
  },
  computed: {
    lastNameIndex() {
      return this.array[0]?.findIndex(e => e.toLowerCase() === 'last name')
    },
    firstNameIndex() {
      return this.array[0]?.findIndex(e => e.toLowerCase() === 'first name')
    },
    nameIndex() {
      return this.array[0]?.length === 1 ? 0 : -1
    },
    nameArray() {
      if(this.firstNameIndex >= 0 && this.lastNameIndex >= 0){
        return this.array.slice(1).map(e => `${e[this.firstNameIndex]} ${e[this.lastNameIndex]}`)
      } else if (this.nameIndex >= 0){
        return this.array.slice(1).map(e => `${e[this.nameIndex]}`)
      }
      return [];
    }
  },
  created() {
    console.log('created')
    window.addEventListener('keydown', (e => {
      console.log(e)
      if(e.key === "Escape") {
        this.showSpinner = false;
      }
    }))
  }
}
</script>

<style>
@font-face {
    font-family: 'Sofia';
    src: url('./assets/fonts/sofiapro-regular-webfont.woff2') format('woff2'), /* Super Modern Browsers */ url('./assets/fonts/sofiapro-regular-webfont.woff') format('woff'); /* Pretty Modern Browsers */
    font-weight: 400;
}

@font-face {
    font-family: 'Sofia';
    src: url('./assets/fonts/sofiapro-bold-webfont.woff2') format('woff2'), /* Super Modern Browsers */ url('./assets/fonts/sofiapro-bold-webfont.woff') format('woff'); /* Pretty Modern Browsers */
    font-weight: 600;
}

@font-face {
    font-family: 'Sofia';
    src: url('./assets/fonts/sofiapro-light-webfont.woff2') format('woff2'), /* Super Modern Browsers */ url('./assets/fonts/sofiapro-light-webfont.woff') format('woff'); /* Pretty Modern Browsers */
    font-weight: 200;
}
body, html {
  padding: 0;
  margin: 0;
}
.app {
  padding: 10vh 0;
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

.file-form {
  margin: 0px 25vw;
  padding: 48px;
  font-size: 36px;
  border: 4px dashed #111111;
}

.button {
    background-color: #20C4F4;
    border: 1px solid #20C4F4;
    box-sizing: border-box;
    border-radius: 40px;
    color: #fff;
    cursor: pointer;
    display: inline-block;
    font-family: 'Sofia', arial, sans-serif;
    font-size: 12px;
    font-weight: bold;
    height: 40px;
    line-height: 37px;
    padding: 0 20px;
    text-decoration: none;
}
.modal {
  display: flex;
  justify-content:center;
  align-items: center;
  height: 50vh;
  width: 50vw;
  background: #F3F4F6;
  position: fixed;
  left: 25vw;
  top: 25vh;
  border-radius: 12px;
}
</style>
