<template>
  <div class="app">
    <div v-if="currentView === 'upload'" class="upload-box">
      <div class="confetti-container">
        <div class="confetti" v-for="(style, index) in confettiStyles" :key="index" :style="style"></div>
      </div>
      <h1 class="main-title">Awardco Random Selector</h1>
      <h2> Two Ways to Add Participants:</h2>
      <div class="upload-columns">
        <div class="upload-column">
          <h3>Upload CSV File</h3>
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
          <div class="csv-download-container">
            <button class="csv-template-button" @click="downloadTemplate">
              Download CSV Template
            </button>
          </div>
        </div>
        <div class="upload-column">
          <h3> Paste Names</h3>
          <div class="name-input-container">
            <textarea
              v-model="nameInput"
              class="name-input"
              placeholder="Enter names, one per line&#10;For multiple entries use: Name, entries&#10;Example:&#10;John Doe, 3&#10;Jane Smith, 1&#10;Bob Wilson"
            ></textarea>
            <button class="submit-names-button" @click="processNameInput">
              Use These Names
            </button>
            <div class="text-input-info">
              You can enter just names (one per line) or use "Name, entries" format for multiple entries per person.
            </div>
          </div>
        </div>
      </div>

      <div class="common-options">
        <div class="option-row">
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
        
        <div class="option-row">
          <label for="deleteAll" style="font-size: 24px"
            >Remove all copies of winner's name? <span class="dependency-note">(requires first option)</span></label
          >
          <input
            name="deleteAll"
            type="checkbox"
            style="width: 25px; height: 25px"
            v-model="deleteAllCopies"
          />
        </div>
      </div>
    </div>

    <div v-else-if="currentView === 'validation'" class="validation-view">
      <div class="back-arrow" role="button" @click="back" title="Go back to upload screen">
        <div class="back-icon">←</div>
      </div>

      <div class="validation-container">
        <h3>Participant Summary</h3>
        <div class="file-info">Source: {{ fileName }}</div>
        <div class="winner-removal-info removal-dialog">
          <span v-if="deleteWinner && deleteAllCopies">Winners will be removed from the participant pool after selection. All copies of the winner's name will be removed.</span>
          <span v-else-if="deleteWinner">Winners will be removed from the participant pool after selection. Only one copy of the winner's name will be removed.</span>
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

        <div class="name-table-container" v-if="nameArray.length > 0">
          <h3>Participant Entry Counts:</h3>
          <div class="table-wrapper">
            <table class="name-entry-table">
              <thead>
                <tr>
                  <th>Name</th>
                  <th>Entries</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, index) in nameEntryTable" :key="index">
                  <td>{{ item.name }}</td>
                  <td class="entry-count">{{ item.entries }}</td>
                </tr>
              </tbody>
            </table>
            <div v-if="nameEntryTable.length === 100" class="table-note">
              Showing first 100 unique names...
            </div>
          </div>
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
          :delete-all-copies="deleteAllCopies"
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
      deleteAllCopies: false,
      validationErrors: [],
      fileName: '',
      nameInput: '',
      remainingParticipants: 0,
      confettiStyles: [],
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
        const lines = this.nameInput.split('\n')
          .map(line => line.trim())
          .filter(line => line.length > 0);

        if (lines.length === 0) {
          this.validationErrors.push('No valid names were found');
          this.currentView = 'validation';
          return;
        }

        // Check if any line contains a comma (name, entries format)
        const hasCommaFormat = lines.some(line => line.includes(','));
        
        if (hasCommaFormat) {
          // Process as "Name, Entries" format
          const processedData = [['Name', 'Entries']]; // Header row
          
          lines.forEach((line, idx) => {
            if (line.includes(',')) {
              const parts = line.split(',').map(part => part.trim());
              if (parts.length >= 2) {
                const name = parts[0];
                const entriesRaw = parts[1];
                const entries = parseInt(entriesRaw, 10);
                if (!isNaN(entries) && entries > 0) {
                  processedData.push([name, String(entries)]);
                } else {
                  this.validationErrors.push(
                    `Line ${idx + 1}: Invalid entry count "${entriesRaw}" for "${name}". Must be a positive integer.`
                  );
                }
              } else {
                // Single name on a line with comma but no entry count
                processedData.push([parts[0], '1']);
              }
            } else {
              // Single name without comma
              processedData.push([line, '1']);
            }
          });
          
          this.array = processedData;
        } else {
          // Legacy format: just names, one per line
          this.array = [['name'], ...lines.map(name => [name])];
        }

        this.currentView = 'validation';
      } catch (error) {
        this.validationErrors.push(`Error processing names: ${error.message || 'Unknown error'}`);
        this.currentView = 'validation';
      }
    },

    downloadTemplate() {
      const csvContent = "Name,Entries\n";
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      const link = document.createElement('a');
      const url = URL.createObjectURL(blob);
      link.setAttribute('href', url);
      link.setAttribute('download', 'name_entry_template.csv');
      link.style.visibility = 'hidden';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
    },
  },
  computed: {
    confettiStyles() {
      const styles = [];
      const colors = ["#FA5959", "#20C4F4", "#FDAF08", "#44DDB4"];
      
      for (let i = 0; i < 30; i++) {
        const randomColor = colors[Math.floor(Math.random() * colors.length)];
        const randomLeft = Math.random() * 100;
        const randomDelay = Math.random() * 5;
        const randomDuration = 3 + Math.random() * 4;
        const randomSize = 8 + Math.random() * 8;

        styles.push({
          backgroundColor: randomColor,
          left: randomLeft + '%',
          animationDelay: randomDelay + 's',
          animationDuration: randomDuration + 's',
          width: randomSize + 'px',
          height: randomSize + 'px'
        });
      }
      
      return styles;
    },
    lastNameIndex() {
      return this.array[0]?.findIndex((e) => e?.toLowerCase?.() === "last name");
    },
    firstNameIndex() {
      return this.array[0]?.findIndex((e) => e?.toLowerCase?.() === "first name");
    },
    nameIndex() {
      return this.array[0]?.length === 1 ? 0 : -1;
    },
    entriesIndex() {
      return this.array[0]?.findIndex((e) => e?.toLowerCase?.() === "entries");
    },
    nameColumnIndex() {
      return this.array[0]?.findIndex((e) => e?.toLowerCase?.() === "name");
    },
    nameArray() {
      // New format: Name and Entries columns
      if (this.nameColumnIndex >= 0 && this.entriesIndex >= 0) {
        const result = [];
        this.array.slice(1).forEach((row) => {
          const name = row[this.nameColumnIndex]?.trim();
          const entries = Math.max(1, parseInt(row[this.entriesIndex]?.trim(), 10) || 1);
          if (name) {
            for (let i = 0; i < entries; i++) {
              result.push(name);
            }
          }
        });
        return result;
      }
      // Legacy format: First Name and Last Name
      else if (this.firstNameIndex >= 0 && this.lastNameIndex >= 0) {
        return this.array
          .slice(1)
          .map((e) => `${e[this.firstNameIndex]} ${e[this.lastNameIndex]}`)
          .filter(name => name.trim() !== ' ');
      }
      // Legacy format: Single Name column (header present but no Entries column)
      else if (this.nameColumnIndex >= 0 && this.entriesIndex < 0) {
        return this.array.slice(1).map((e) => `${e[this.nameColumnIndex]}`).filter(name => name.trim() !== '');
      }
      // Legacy format: Single column without header
      else if (this.nameIndex >= 0) {
        return this.array.slice(1).map((e) => `${e[this.nameIndex]}`).filter(name => name.trim() !== '');
      }
      // Legacy format: No headers at all
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
    nameEntryTable() {
      if (this.nameArray.length === 0) return [];
      
      // Count occurrences of each name
      const nameCounts = {};
      this.nameArray.forEach(name => {
        nameCounts[name] = (nameCounts[name] || 0) + 1;
      });
      
      // Convert to array of objects and sort by name
      return Object.entries(nameCounts)
        .map(([name, count]) => ({ name, entries: count }))
        .sort((a, b) => a.name.localeCompare(b.name))
        .slice(0, 100); // Show first 100 unique names
    },
  },
  watch: {
    deleteAllCopies(newVal) {
      // Automatically check the first checkbox when the second one is checked
      if (newVal === true) {
        this.deleteWinner = true;
      }
    },
    deleteWinner(newVal) {
      // Automatically uncheck the second checkbox when the first one is unchecked
      if (newVal === false) {
        this.deleteAllCopies = false;
      }
    }
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

h2 {
  font-size: 32px;
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
  height: 100vh;
  width: 100%;
  gap: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  box-sizing: border-box;
  position: relative;
  overflow: hidden;
}

.main-title {
  font-size: 36px;
  font-weight: 600;
  color: #f9fafb;
  margin: 0;
  text-align: center;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  position: relative;
  z-index: 2;
  flex-shrink: 0;
}

.confetti-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: hidden;
  z-index: 1;
}

.confetti {
  position: absolute;
  top: -20px;
  animation: fall linear infinite;
  pointer-events: none;
}

@keyframes fall {
  0% {
    transform: translateY(-100vh) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translateY(100vh) rotate(360deg);
    opacity: 0;
  }
}

.upload-columns {
  display: flex;
  width: 100%;
  max-width: 1200px;
  gap: 30px;
  margin-bottom: 0;
  position: relative;
  z-index: 2;
  flex: 1;
  min-height: 0;
}

.upload-column {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-column h3 {
  margin-bottom: 20px;
  color: #f9fafb;
  font-size: 28px;
}

.common-options {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  margin-top: 0;
  position: relative;
  z-index: 2;
  flex-shrink: 0;
}

.option-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.option-row label {
  font-size: 20px;
}

.dependency-note {
  font-size: 16px;
  color: #9CA3AF;
  font-weight: 400;
  font-style: italic;
}

.file-form {
  padding: 40px 30px;
  font-size: 28px;
  border: 4px dashed #f9fafb;
  background-color: rgba(31, 41, 55, 0.7);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 500px;
  height: 220px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
  flex-shrink: 0;
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

.csv-download-container {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.csv-template-button {
  padding: 12px 20px;
  background-color: #60a5fa;
  color: #1f2937;
  font-size: 16px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: Sofia, sans-serif;
}

.csv-template-button:hover {
  background-color: #3b82f6;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.text-input-info {
  margin-top: 15px;
  padding: 12px 16px;
  background-color: rgba(31, 41, 55, 0.6);
  border-radius: 8px;
  font-size: 14px;
  color: #d1d5db;
  text-align: center;
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

.validation-container h3 {
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

.name-table-container {
  background-color: #374151;
  padding: 15px 20px;
  border-radius: 8px;
  margin-bottom: 30px;
}

.name-table-container h3 {
  font-size: 18px;
  margin-top: 0;
  color: #f9fafb;
  margin-bottom: 15px;
}

.table-wrapper {
  max-height: 300px;
  overflow-y: auto;
  border-radius: 6px;
}

.name-entry-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #1f2937;
  border-radius: 6px;
  overflow: hidden;
}

.name-entry-table th {
  background-color: #111827;
  color: #f9fafb;
  padding: 12px 16px;
  text-align: left;
  font-weight: 600;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.name-entry-table td {
  padding: 10px 16px;
  border-bottom: 1px solid #374151;
  color: #d1d5db;
  font-size: 14px;
}

.name-entry-table tbody tr:hover {
  background-color: #2d3748;
}

.name-entry-table tbody tr:last-child td {
  border-bottom: none;
}

.entry-count {
  text-align: center;
  font-weight: 600;
  color: #60a5fa !important;
}

.table-note {
  text-align: center;
  font-style: italic;
  color: #9CA3AF;
  margin-top: 10px;
  font-size: 14px;
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
  height: 220px;
  padding: 15px;
  font-size: 16px;
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
  flex-shrink: 0;
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
  .upload-box {
    padding: 10px;
    gap: 10px;
  }

  .upload-columns {
    flex-direction: column;
    align-items: center;
    gap: 15px;
    flex: 1;
    justify-content: center;
  }

  .upload-column {
    width: 100%;
    max-width: 500px;
    margin-bottom: 0;
  }

  .file-form {
    padding: 25px 20px;
    font-size: 22px;
    height: 160px;
  }

  .name-input {
    height: 160px;
    font-size: 15px;
    padding: 12px;
  }

  .main-title {
    font-size: 26px;
    margin: 0;
  }

  .option-row {
    flex-direction: row;
    text-align: center;
    gap: 8px;
    flex-wrap: wrap;
  }

  .option-row label {
    font-size: 18px !important;
  }

  .common-options {
    gap: 8px;
    margin-top: 0;
  }

  .csv-template-button, .submit-names-button {
    padding: 8px 16px;
    font-size: 14px;
  }
}

@media screen and (max-height: 500px) {
  .participant-counter {
    visibility: hidden;
  }
}

@media screen and (max-width: 600px) {
  .main-title {
    font-size: 22px;
    margin: 0;
  }

  .upload-box {
    padding: 8px;
    gap: 8px;
    height: 100vh;
  }

  .file-form {
    padding: 20px 15px;
    font-size: 18px;
    height: 130px;
  }

  .name-input {
    height: 130px;
    font-size: 14px;
    padding: 10px;
  }

  .option-row label {
    font-size: 16px !important;
  }

  .dependency-note {
    font-size: 13px !important;
  }

  .csv-template-button, .submit-names-button {
    padding: 6px 12px;
    font-size: 13px;
  }

  .upload-columns {
    gap: 10px;
  }

  .common-options {
    gap: 6px;
  }
}

@media screen and (max-height: 700px) {
  .upload-box {
    gap: 8px;
    padding: 8px;
  }

  .main-title {
    font-size: 28px;
    margin: 0;
  }

  .upload-columns {
    gap: 12px;
  }

  .file-form, .name-input {
    height: 140px;
  }

  .file-form {
    padding: 20px;
    font-size: 24px;
  }

  .name-input {
    padding: 12px;
    font-size: 15px;
  }

  .common-options {
    gap: 6px;
    margin-top: 0;
  }
}

@media screen and (max-height: 600px) {
  .main-title {
    font-size: 24px;
  }

  .file-form, .name-input {
    height: 120px;
  }

  .file-form {
    padding: 15px;
    font-size: 20px;
  }

  .option-row label {
    font-size: 16px !important;
  }

  .upload-columns {
    gap: 8px;
  }
}
</style>
