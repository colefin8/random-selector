<template>
  <div class="app">
    <div v-if="currentView === 'upload'" class="upload-box">
      <div class="upload-columns">
        <div class="upload-column">
          <h2>Upload CSV File</h2>
          <input hidden type="file" ref="file" @change="uploadLegacy" />
          <div
            class="file-form"
            @click="$refs.file.click()"
            @dragover.prevent
            @drop.prevent="uploadDrag"
          >
            <span class="drop-files"
              >Drop file here or click to select a file!</span
            >
          </div>
          <div class="csv-format-info">
            CSV upload should be formatted with one of the following:
            <ul>
              <li>one header "Name"</li>
              <li>two headers "First Name" and "Last Name"</li>
              <li>no headers at all, just a list of names in one column</li>
            </ul>
          </div>
        </div>

        <div class="upload-column">
          <h2>Or Paste Names</h2>
          <div class="name-input-container">
            <textarea
              v-model="nameInput"
              class="name-input"
              placeholder="Enter names, one per line"
            ></textarea>
            <button class="submit-names-button" @click="processNameInput">
              Use These Names
            </button>
          </div>
        </div>
      </div>

      <div class="common-options">
        <label for="delete" style="font-size: 24px"
          >Remove winner after spinning?</label
        >
        <input
          name="delete"
          type="checkbox"
          style="width: 25px; height: 25px"
          v-model="deleteWinner"
        />
      </div>
    </div>

    <div v-else-if="currentView === 'validation'" class="validation-view">
      <div class="back-arrow" role="button" @click="back" title="Go back to upload screen">
        <div class="back-icon">←</div>
      </div>

      <div class="validation-container">
        <h2>Participant Summary</h2>
        <div class="file-info">Source: {{ fileName }}</div>
        <div class="winner-removal-info removal-dialog">
          <span v-if="deleteWinner">Winners will be removed from the participant pool after selection.</span>
          <span v-else>Winners will remain in the participant pool after selection.</span>
        </div>

        <div v-if="validationErrors.length > 0" class="validation-errors">
          <h3>Issues Found:</h3>
          <ul>
            <li v-for="(error, index) in validationErrors" :key="index" class="error-message">
              {{ error }}
            </li>
          </ul>
          <button class="try-again-button" @click="back">Try Again with Different Input</button>
        </div>

        <div class="validation-stats">
          <div class="stat-item">
            <div class="stat-value">{{ nameArray.length }}</div>
            <div class="stat-label">Total Participants</div>
          </div>
        </div>

        <div class="name-sample" v-if="nameArray.length > 0">
          <h3>Sample Names:</h3>
          <ul>
            <li v-for="(name, index) in nameArray.slice(0, 5)" :key="index">{{ name }}</li>
            <li v-if="nameArray.length > 5">...</li>
          </ul>
        </div>

        <button
          class="proceed-button"
          @click="startSpinner"
          :disabled="nameArray.length === 0"
        >
          Start Random Selection
        </button>
      </div>
    </div>

    <div v-else-if="currentView === 'spinner'">
      <div class="back-arrow" role="button" @click="back" title="Go back to upload screen">
        <div class="back-icon">←</div>
      </div>
      <div class="participant-counter">Remaining Participants: {{ remainingParticipants }}</div>
      <div class="machine-container">
        <spinner
          :names="nameArray"
          :delete-winner="deleteWinner"
          @winner-selected="updateRemainingCount"
          ref="spinnerComponent"
        ></spinner>
      </div>
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
      currentView: 'upload',
      deleteWinner: true,
      validationErrors: [],
      fileName: '',
      nameInput: '',
      remainingParticipants: 0,
    };
  },
  methods: {
    back() {
      if (this.currentView === 'spinner') {
        this.currentView = 'validation';
      } else {
        this.currentView = 'upload';
        this.array = [];
        this.validationErrors = [];
      }
    },
    startSpinner() {
      this.remainingParticipants = this.nameArray.length;
      this.currentView = 'spinner';
    },

    updateRemainingCount() {
      // Update the remaining participants count
      if (this.$refs.spinnerComponent) {
        this.remainingParticipants = this.$refs.spinnerComponent.names.length;
      }
    },
    upload(file) {
      this.fileName = file.name;
      this.validationErrors = [];
      if (!file.name.toLowerCase().endsWith('.csv')) {
        this.validationErrors.push('Please upload a CSV file');
        this.currentView = 'validation';
        return;
      }
      const MAX_FILE_SIZE_MB = 5;
      if (file.size > MAX_FILE_SIZE_MB * 1024 * 1024) {
        this.validationErrors.push(`File size exceeds ${MAX_FILE_SIZE_MB}MB limit. Please use a smaller file.`);
        this.currentView = 'validation';
        return;
      }      const reader = new FileReader();
      let text;

      reader.onerror = () => {
        this.validationErrors.push('Error reading file: The file might be corrupted or inaccessible');
        this.currentView = 'validation';
      };

      reader.onload = (f) => {
        try {
          text = f.target.result;

          if (!text || text.trim().length === 0) {
            this.validationErrors.push('The file is empty');
            this.currentView = 'validation';
            return;
          }

          let arr = text.split("\n");
          this.array = arr.map((e) => {
            if (e.length > 0) {
              return e.trim().split(",");
            } else return [];
          });
          if (arr.length === 0) {
            this.validationErrors.push('CSV file appears to be empty');
          }

          if (this.nameArray.length === 0) {
            this.validationErrors.push('No valid names found in the CSV');
          }

          const rowLengths = this.array.map(row => row.length).filter(len => len > 0);
          const inconsistentRows = [];

          if (inconsistentRows.length > 0) {
            this.validationErrors.push(`Found ${inconsistentRows.length} rows with inconsistent column counts. Some data may be missing or malformed.`);
          }
          this.currentView = 'validation';
        } catch (error) {
          this.validationErrors.push(`Error processing file: ${error.message || 'Unknown error'}`);
          this.currentView = 'validation';
        }
      };
      reader.readAsText(file);
    },
    uploadDrag(e) {
      if (e.dataTransfer.files && e.dataTransfer.files.length > 0) {
        this.upload(e.dataTransfer.files[0]);
      } else {
        this.validationErrors.push('No file was provided');
        this.currentView = 'validation';
      }
    },
    uploadLegacy(e) {
      if (e.target.files && e.target.files.length > 0) {
        this.upload(e.target.files[0]);
      } else {
        this.validationErrors.push('No file was selected');
        this.currentView = 'validation';
      }
    },

    processNameInput() {
      this.validationErrors = [];
      this.fileName = 'Manual Entry';

      if (!this.nameInput.trim()) {
        this.validationErrors.push('No names were entered');
        this.currentView = 'validation';
        return;
      }

      try {
        const names = this.nameInput.split('\n')
          .map(name => name.trim())
          .filter(name => name.length > 0);

        if (names.length === 0) {
          this.validationErrors.push('No valid names were found');
          this.currentView = 'validation';
          return;
        }

        this.array = [['name'], ...names.map(name => [name])];

        this.currentView = 'validation';
      } catch (error) {
        this.validationErrors.push(`Error processing names: ${error.message || 'Unknown error'}`);
        this.currentView = 'validation';
      }
    },
  },
  computed: {
    lastNameIndex() {
      return this.array[0]?.findIndex((e) => e?.toLowerCase?.() === "last name");
    },
    firstNameIndex() {
      return this.array[0]?.findIndex((e) => e?.toLowerCase?.() === "first name");
    },
    nameIndex() {
      return this.array[0]?.length === 1 ? 0 : -1;
    },
    nameArray() {
      if (this.firstNameIndex >= 0 && this.lastNameIndex >= 0) {
        return this.array
          .slice(1)
          .map((e) => `${e[this.firstNameIndex]} ${e[this.lastNameIndex]}`)
          .filter(name => name.trim() !== ' ');
      }
      else if (this.nameIndex >= 0) {
        return this.array.slice(1).map((e) => `${e[this.nameIndex]}`).filter(name => name.trim() !== '');
      }
      else {
        const allRows = [...this.array];

        return allRows
          .filter(row => row.length > 0)
          .map(row => {
            if (row.length > 1) {
              return row.join(' ').trim();
            } else if (row.length === 1) {

              return row[0].trim();
            }
            return '';
          })
          .filter(name => name !== '');
      }
    },
  },
  created() {
    window.addEventListener("keydown", (e) => {
      if (e.key === "Escape") {
        this.back();
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
  color: #f9fafb;
  box-sizing: border-box;
  background: linear-gradient(135deg, #1f2937 0%, #111827 100%);
  background-size: cover;

}

.upload-box {
  height: 100%;
  width: 100%;
  gap: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  box-sizing: border-box;
}

.upload-columns {
  display: flex;
  width: 100%;
  max-width: 1200px;
  gap: 40px;
  margin-bottom: 30px;
}

.upload-column {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-column h2 {
  margin-bottom: 20px;
  color: #f9fafb;
  font-size: 28px;
}

.common-options {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-top: 20px;
}

.file-form {
  padding: 60px 40px;
  font-size: 32px;
  border: 4px dashed #f9fafb;
  background-color: rgba(31, 41, 55, 0.7);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 500px;
  height: 300px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
}

.file-form:hover {
  background-color: rgba(31, 41, 55, 0.9);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
  border-color: #FA5959;
}

.drop-files {
  font-weight: 600;
}

.file-form::before {
  content: "📄";
  display: block;
  font-size: 60px;
  margin-bottom: 20px;
}

.csv-format-info {
  margin-top: 20px;
  padding: 15px 20px;
  background-color: rgba(31, 41, 55, 0.6);
  border-radius: 8px;
  font-size: 16px;
  color: #d1d5db;
  text-align: left;
  border: 1px solid rgba(249, 250, 251, 0.2);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.back-arrow {
  position: fixed;
  left: 20px;
  top: 20px;
  height: 50px;
  width: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(31, 41, 55, 0.8);
  border-radius: 50%;
  cursor: pointer;
  z-index: 100;
  transition: all 0.2s ease;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5);
}

.back-arrow:hover {
  transform: scale(1.1);
  background-color: #1f2937;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.6);
}

.back-icon {
  height: 100%;
  width: 100%;
  font-size: 28px;
  font-weight: bold;
  color: #f9fafb;
}

.participant-counter {
  position: fixed;
  top: 20px;
  right: 20px;
  background-color: rgba(31, 41, 55, 0.9);
  border-radius: 20px;
  padding: 10px 18px;
  font-size: 18px;
  font-weight: 600;
  color: #f9fafb;
  z-index: 100;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
  transition: all 0.3s ease;
}

.machine-container {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  position: relative;
}


.validation-view {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.validation-container {
  background-color: rgba(31, 41, 55, 0.9);
  border-radius: 16px;
  padding: 40px;
  max-width: 600px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
  text-align: left;
}

.validation-container h2 {
  font-size: 32px;
  color: #f9fafb;
  margin-top: 0;
  margin-bottom: 10px;
  text-align: center;
}

.file-info {
  text-align: center;
  font-style: italic;
  color: #9CA3AF;
  margin-bottom: 16px;
}

.winner-removal-info {
  text-align: center;
  padding: 10px 16px;
  border-radius: 6px;
  margin-bottom: 24px;
  font-size: 16px;
  font-weight: 500;
}

.removal-dialog {
  background-color: #1e3a8a;
  color: #93c5fd;
  border: 1px solid #3b82f6;
}

.validation-errors {
  background-color: #450a0a;
  border-left: 4px solid #FA5959;
  padding: 15px 20px;
  margin-bottom: 24px;
  border-radius: 8px;
}

.validation-errors h3 {
  color: #FA5959;
  margin-top: 0;
  font-size: 18px;
}

.error-message {
  color: #fca5a5;
  margin-bottom: 8px;
}

.try-again-button {
  display: block;
  margin-top: 16px;
  padding: 10px 16px;
  background-color: #F87171;
  color: white;
  font-size: 14px;
  font-weight: 500;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.try-again-button:hover {
  background-color: #EF4444;
}

.validation-stats {
  display: flex;
  justify-content: space-around;
  margin: 30px 0;
}

.stat-item {
  text-align: center;
}

.stat-value {
  font-size: 48px;
  font-weight: bold;
  color: #60a5fa;
}

.stat-label {
  font-size: 16px;
  color: #9CA3AF;
}

.name-sample {
  background-color: #374151;
  padding: 15px 20px;
  border-radius: 8px;
  margin-bottom: 30px;
}

.name-sample h3 {
  font-size: 18px;
  margin-top: 0;
  color: #f9fafb;
}

.name-sample ul {
  margin: 0;
  padding-left: 20px;
}

.name-sample li {
  margin-bottom: 5px;
}

.proceed-button {
  display: block;
  width: 100%;
  padding: 14px 24px;
  background-color: #60a5fa;
  color: #1f2937;
  font-size: 18px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.proceed-button:hover {
  background-color: #3b82f6;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.proceed-button:disabled {
  background-color: #4B5563;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.name-input-container {
  width: 100%;
  max-width: 500px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.name-input {
  width: 100%;
  height: 300px;
  padding: 20px;
  font-size: 18px;
  font-family: Sofia, sans-serif;
  border: 4px dashed #f9fafb;
  background-color: rgba(31, 41, 55, 0.7);
  color: #f9fafb;
  border-radius: 12px;
  border-style: solid;
  transition: all 0.3s ease;
  resize: none;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  box-sizing: border-box;
}

.name-input:focus {
  outline: none;
  border-color: #60a5fa;
  background-color: rgba(31, 41, 55, 0.9);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
}

.name-input:hover {
  background-color: rgba(31, 41, 55, 0.9);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
  border-color: #60a5fa;
}

.submit-names-button {
  padding: 14px 24px;
  background-color: #60a5fa;
  color: #1f2937;
  font-size: 18px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.submit-names-button:hover {
  background-color: #3b82f6;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.name-input::placeholder {
  color: #9CA3AF;
}

@media screen and (max-width: 900px) {
  .upload-columns {
    flex-direction: column;
    align-items: center;
  }

  .upload-column {
    width: 100%;
    max-width: 500px;
    margin-bottom: 30px;
  }

  .file-form {
    padding: 40px 30px;
    font-size: 24px;
    height: 200px;
  }

  .name-input {
    height: 200px;
  }
}

@media screen and (max-height: 500px) {
  .participant-counter {
    visibility: hidden;
  }
}
</style>
