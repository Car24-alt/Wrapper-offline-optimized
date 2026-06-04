<template>
  <div class="export-modal" v-if="isOpen" @click="closeOnBackdrop">
    <div class="export-container" @click.stop>
      <div class="header">
        <div class="icon">🎬</div>
        <h1>Export Video</h1>
      </div>

      <div class="info-box">
        <strong>⚡ Tip:</strong> Direct export is much faster than screen recording and produces better quality!
      </div>

      <div class="status-message" :class="[statusType, { show: statusMessage }]" v-if="statusMessage">
        {{ statusMessage }}
      </div>

      <!-- Format Selection -->
      <div class="section">
        <div class="section-title">Output Format</div>
        <div class="format-grid">
          <div
            class="format-option"
            :class="{ active: exportConfig.format === 'mp4' }"
            @click="exportConfig.format = 'mp4'"
          >
            <div>📦 MP4</div>
            <div style="font-size: 11px; color: currentColor; opacity: 0.7; margin-top: 4px;">Best Compatibility</div>
          </div>
          <div
            class="format-option"
            :class="{ active: exportConfig.format === 'webm' }"
            @click="exportConfig.format = 'webm'"
          >
            <div>📦 WebM</div>
            <div style="font-size: 11px; color: currentColor; opacity: 0.7; margin-top: 4px;">Better Compression</div>
          </div>
        </div>
      </div>

      <!-- Quality Settings -->
      <div class="section">
        <div class="section-title">Quality</div>
        <input
          type="range"
          class="quality-slider"
          v-model="exportConfig.quality"
          min="30"
          max="100"
        >
        <div class="quality-info">
          <span>Lower Quality</span>
          <span class="quality-value">{{ exportConfig.quality }}%</span>
          <span>Higher Quality</span>
        </div>
      </div>

      <!-- Frame Rate -->
      <div class="section">
        <div class="section-title">Frame Rate</div>
        <div class="framerate-grid">
          <div
            class="framerate-option"
            :class="{ active: exportConfig.fps === 24 }"
            @click="exportConfig.fps = 24"
          >
            24 FPS
          </div>
          <div
            class="framerate-option"
            :class="{ active: exportConfig.fps === 30 }"
            @click="exportConfig.fps = 30"
          >
            30 FPS
          </div>
          <div
            class="framerate-option"
            :class="{ active: exportConfig.fps === 60 }"
            @click="exportConfig.fps = 60"
          >
            60 FPS
          </div>
        </div>
      </div>

      <!-- Additional Options -->
      <div class="section">
        <div class="section-title">Options</div>
        <div style="display: flex; flex-direction: column; gap: 10px;">
          <div class="checkbox-group">
            <input type="checkbox" id="audioCheckbox" class="checkbox" v-model="exportConfig.audio">
            <label for="audioCheckbox" class="checkbox-label">Include Audio</label>
          </div>
          <div class="checkbox-group">
            <input type="checkbox" id="subtitlesCheckbox" class="checkbox" v-model="exportConfig.subtitles">
            <label for="subtitlesCheckbox" class="checkbox-label">Add Subtitles</label>
          </div>
        </div>
      </div>

      <!-- Progress Bar -->
      <div class="progress-section" :class="{ active: isExporting }" v-if="isExporting">
        <div class="section-title">Export Progress</div>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: progress + '%' }"></div>
        </div>
        <div class="progress-text">
          <strong>{{ Math.floor(progress) }}%</strong> - <span>{{ progressStatus }}</span>
        </div>
      </div>

      <!-- Buttons -->
      <div class="button-group">
        <button class="btn-cancel" @click="cancel" :disabled="isExporting">Cancel</button>
        <button class="btn-export" @click="startExport" :disabled="isExporting">
          {{ isExporting ? 'Exporting...' : 'Export Video' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import { exportVideo } from '../services/exportService';

export default defineComponent({
  name: 'ExportDialog',
  props: {
    isOpen: {
      type: Boolean,
      default: false,
    },
  },
  emits: ['close', 'export-start', 'export-complete'],
  data() {
    return {
      exportConfig: {
        format: 'mp4',
        quality: 75,
        fps: 30,
        audio: true,
        subtitles: false,
      },
      isExporting: false,
      progress: 0,
      progressStatus: 'Initializing...',
      statusMessage: '',
      statusType: 'info',
    };
  },
  methods: {
    async startExport() {
      if (this.isExporting) return;

      this.isExporting = true;
      this.progress = 0;
      this.$emit('export-start', this.exportConfig);

      try {
        const result = await exportVideo(
          this.exportConfig,
          (progress, status) => {
            this.progress = progress;
            this.progressStatus = status;
          }
        );

        this.showMessage('✓ Video exported successfully!', 'success');
        this.$emit('export-complete', result);

        setTimeout(() => {
          this.close();
        }, 2000);
      } catch (error) {
        this.showMessage(`✗ Export failed: ${error.message}`, 'error');
        console.error('Export error:', error);
      } finally {
        this.isExporting = false;
      }
    },

    cancel() {
      if (!this.isExporting) {
        this.close();
      }
    },

    close() {
      this.$emit('close');
      this.resetForm();
    },

    closeOnBackdrop(event) {
      if (event.target === event.currentTarget && !this.isExporting) {
        this.close();
      }
    },

    resetForm() {
      this.exportConfig = {
        format: 'mp4',
        quality: 75,
        fps: 30,
        audio: true,
        subtitles: false,
      };
      this.progress = 0;
      this.progressStatus = 'Initializing...';
      this.statusMessage = '';
    },

    showMessage(message, type) {
      this.statusMessage = message;
      this.statusType = type;
      setTimeout(() => {
        this.statusMessage = '';
      }, 5000);
    },
  },
});
</script>

<style scoped>
.export-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.export-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-width: 500px;
  width: 100%;
  padding: 40px;
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.header {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 30px;
}

.icon {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
}

.header h1 {
  font-size: 24px;
  color: #2d3748;
  font-weight: 600;
}

.section {
  margin-bottom: 25px;
}

.section-title {
  font-size: 13px;
  font-weight: 700;
  text-transform: uppercase;
  color: #718096;
  margin-bottom: 12px;
  letter-spacing: 0.5px;
}

.format-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-bottom: 15px;
}

.format-option {
  padding: 12px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
  text-align: center;
  font-weight: 500;
  color: #4a5568;
  user-select: none;
}

.format-option:hover {
  border-color: #667eea;
  background: #f7fafc;
}

.format-option.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
  color: white;
}

.quality-slider {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: #e2e8f0;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
}

.quality-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
}

.quality-slider::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
  border: none;
}

.quality-info {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  font-size: 12px;
  color: #718096;
}

.quality-value {
  font-weight: 600;
  color: #2d3748;
}

.framerate-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 10px;
}

.framerate-option {
  padding: 10px;
  border: 2px solid #e2e8f0;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  text-align: center;
  font-weight: 500;
  color: #4a5568;
  font-size: 14px;
}

.framerate-option:hover {
  border-color: #667eea;
  background: #f7fafc;
}

.framerate-option.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
  color: white;
}

.checkbox-group {
  display: flex;
  gap: 12px;
  align-items: center;
}

.checkbox {
  width: 18px;
  height: 18px;
  cursor: pointer;
  accent-color: #667eea;
}

.checkbox-label {
  font-size: 14px;
  color: #4a5568;
  cursor: pointer;
  flex: 1;
}

.progress-section {
  display: none;
  margin-bottom: 25px;
}

.progress-section.active {
  display: block;
}

.progress-bar {
  width: 100%;
  height: 6px;
  background: #e2e8f0;
  border-radius: 3px;
  overflow: hidden;
  margin-bottom: 8px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  transition: width 0.3s ease;
  border-radius: 3px;
}

.progress-text {
  font-size: 13px;
  color: #718096;
  text-align: center;
}

.progress-text strong {
  color: #2d3748;
}

.button-group {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-top: 30px;
}

button {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.btn-cancel {
  background: #e2e8f0;
  color: #2d3748;
}

.btn-cancel:hover:not(:disabled) {
  background: #cbd5e0;
}

.btn-cancel:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-export {
  grid-column: 1 / -1;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-export:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.4);
}

.btn-export:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.info-box {
  background: #f7fafc;
  border-left: 4px solid #667eea;
  padding: 12px;
  border-radius: 4px;
  font-size: 12px;
  color: #4a5568;
  margin-bottom: 20px;
}

.info-box strong {
  color: #2d3748;
}

.status-message {
  padding: 12px;
  border-radius: 6px;
  margin-bottom: 15px;
  font-size: 13px;
  display: none;
}

.status-message.show {
  display: block;
}

.status-message.success {
  background: #c6f6d5;
  border: 1px solid #9ae6b4;
  color: #22543d;
}

.status-message.error {
  background: #fed7d7;
  border: 1px solid #fc8181;
  color: #742a2a;
}

.status-message.info {
  background: #bee3f8;
  border: 1px solid #90cdf4;
  color: #2c5282;
}

@media (max-width: 480px) {
  .export-container {
    padding: 30px 20px;
  }

  .header h1 {
    font-size: 20px;
  }

  .format-grid {
    grid-template-columns: 1fr;
  }

  .button-group {
    grid-template-columns: 1fr;
  }
}
</style>
