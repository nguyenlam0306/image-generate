<template>
  <div class="container">
    <h1 class="title">Image Generator</h1>
    <div class="input-section">
      <textarea
        v-model="prompt"
        placeholder="Enter your description in English (e.g., 'A peaceful sunset over a mountain lake')"
        class="prompt-input"
        rows="4"
      ></textarea>
      <button @click="summarizeAndGenerate" class="generate-btn" :disabled="!prompt.trim() || loading">
        Generate Image
      </button>
    </div>
    <div class="result-section" v-if="summary || imageUrl || error">
      <p v-if="summary" class="summary-text">Summary: {{ summary }}</p>
      <img v-if="imageUrl" :src="imageUrl" alt="Generated Image" class="generated-image" />
      <p v-if="error" class="error-message">{{ error }}</p>
    </div>
    <div v-if="loading" class="loading">
      <span class="spinner"></span> {{ loadingMessage }}
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      prompt: '',
      summary: null,
      imageUrl: null,
      error: null,
      loading: false,
      loadingMessage: '',
    };
  },
  methods: {
    async summarizeAndGenerate() {
      this.error = null;
      this.imageUrl = null;
      this.summary = null;
      this.loading = true;
      this.loadingMessage = 'Summarizing text...';

      try {
        const apiBaseUrl = 'https://3c3d-34-16-169-132.ngrok-free.app'; // Cập nhật URL nếu cần
        const summarizeResponse = await axios.post(`${apiBaseUrl}/summarize/`, {
          prompt: this.prompt,
        }, {
          headers: { 'Content-Type': 'application/json' }
        });
        if (summarizeResponse.data.error) throw new Error(summarizeResponse.data.error);
        this.summary = summarizeResponse.data.summary;

        this.loadingMessage = 'Generating image...';
        console.log('Sending request to /generate-image/ with summary:', this.summary);
        const imageResponse = await axios.post(
          `${apiBaseUrl}/generate-image/`,
          { summary: this.summary },
          {
            responseType: 'blob',
          
          }
        );
        console.log('Image response received:', imageResponse);
        if (imageResponse.headers['content-type'] !== 'image/png') {
          const errorText = await imageResponse.data.text();
          throw new Error(`Image generation failed: ${errorText}`);
        }
        this.imageUrl = URL.createObjectURL(imageResponse.data);
      } catch (err) {
        console.error('Request failed:', err);
        this.error = err.message || 'An error occurred while generating the image';
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>


<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.title {
  text-align: center;
  color: #333;
  font-size: 2.5rem;
  margin-bottom: 20px;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.input-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
}

.prompt-input {
  width: 100%;
  max-width: 600px;
  padding: 15px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  resize: none;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: border-color 0.3s;
}

.prompt-input:focus {
  border-color: #007bff;
  outline: none;
}

.generate-btn {
  padding: 12px 30px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s;
}

.generate-btn:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}

.generate-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  transform: none;
}

.result-section {
  margin-top: 30px;
  text-align: center;
}

.generated-image {
  max-width: 100%;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  margin-top: 20px;
}

.error-message {
  color: #dc3545;
  font-size: 1.1rem;
  margin-top: 20px;
}

.loading {
  text-align: center;
  margin-top: 20px;
  font-size: 1.2rem;
  color: #555;
}

.spinner {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 3px solid #007bff;
  border-top: 3px solid transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-right: 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>