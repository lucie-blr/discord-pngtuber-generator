<template>
  <div class="container">
    <h1>Discord PNGTuber Generator</h1>

    <p>
      This tool helps you generate custom CSS for <a href="https://streamkit.discord.com/overlay">Discord streamkit overlay</a> to replace user avatars with PNG images that change based on their speaking status. Configure global settings and add users with their respective idle and speaking images to create your personalized CSS.
    </p>

    <div class="card settings">
      <h2>Global Settings</h2>
      <div class="row">
        <label>
          Bounce Height (px):
          <input type="number" v-model="settings.bounceHeight">
        </label>
        <label>
          Dim Idle Users:
          <select v-model="settings.dimIdle">
            <option :value="true">Yes (50% Brightness)</option>
            <option :value="false">No (100% Brightness)</option>
          </select>
        </label>
      </div>
      <div class="row">
        <label>
          Hide Names:
          <input type="checkbox" v-model="settings.hideNames">
        </label>
        <label>
          Hide Users Not in List:
          <input type="checkbox" v-model="settings.hideNotInList">
        </label>
        <label>
          Hide Myself:
          <input type="text" v-model="settings.myId" placeholder="e.g. 123456789012345678">
        </label>
      </div>
    </div>

    <div class="users-section">
      <h2>Users</h2>
      <div v-for="(user, index) in users" :key="index" class="card user-card">
        <div class="card-header">
          <h3>User {{ index + 1 }}</h3>
          <button @click="removeUser(index)" class="btn-delete" v-if="users.length > 1">Remove</button>
        </div>

        <div class="row">
          <div class="input-group">
            <label>Discord User ID</label>
            <input type="text" v-model="user.id" placeholder="e.g. 123456789012345678">
            <small>Right click user in Discord -> Copy ID</small>
          </div>
        </div>

        <div class="row">
          <div class="input-group">
            <label>Idle Image URL</label>
            <input type="text" v-model="user.idleUrl" placeholder="https://...">
          </div>
          <div class="input-group">
            <label>Speaking Image URL</label>
            <input type="text" v-model="user.speakingUrl" placeholder="https://...">
          </div>
        </div>
      </div>

      <button @click="addUser" class="btn-add">+ Add Another User</button>
    </div>

    <div class="output-section">
      <h2>Generated CSS</h2>
      <textarea readonly :value="generatedCss" rows="15"></textarea>
      <button @click="copyToClipboard" class="btn-copy">
        {{ copied ? 'Copied!' : 'Copy CSS' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import {ref, reactive, computed, watch, onMounted} from 'vue';

// --- State ---
const settings = reactive({
  bounceHeight: 10,
  dimIdle: true,
  hideNames: true,
  hideNotInList: false,
  myId: ""
});

const users = ref([
  { id: '', idleUrl: '', speakingUrl: '' }
]);

const STORAGE_KEY = 'discord-pngtuber-v1';

onMounted(() => {
  const savedData = localStorage.getItem(STORAGE_KEY);
  if (savedData) {
    try {
      const parsed = JSON.parse(savedData);
      // Merge saved settings with current reactive object
      if (parsed.settings) Object.assign(settings, parsed.settings);
      // Load saved users
      if (parsed.users) users.value = parsed.users;
    } catch (e) {
      console.error("Failed to load saved data", e);
    }
  }
});

// Watch for changes in settings OR users and save immediately
watch([settings, users], () => {
  const dataToSave = {
    settings: settings,
    users: users.value
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(dataToSave));
}, { deep: true }); // 'deep: true' is important to detect changes inside objects/arrays


const copied = ref(false);

// --- Actions ---
const addUser = () => {
  users.value.push({ id: '', idleUrl: '', speakingUrl: '' });
};

const removeUser = (index) => {
  users.value.splice(index, 1);
};

const copyToClipboard = () => {
  navigator.clipboard.writeText(generatedCss.value);
  copied.value = true;
  setTimeout(() => copied.value = false, 2000);
};

// --- CSS Generation Logic ---
const generatedCss = computed(() => {
  // 1. Base Styles (Layout, Animation, Hiding defaults)
  let css = `/* Generated PNGTuber CSS */
body {
  background-color: rgba(0, 0, 0, 0);
  margin: 0px auto;
  overflow: hidden;
}

ul {
  margin: 0;
  margin-top: ${settings.bounceHeight}px;
  padding: 0;
  position: relative;
  list-style: none;
  display: flex;
}

/* Animation Definition */
@keyframes speak-now {
  0% { bottom: 0px; }
  15% { bottom: ${settings.bounceHeight}px; }
  30% { bottom: 0px; }
}

/* Hide all default avatars first */

${settings.hideNotInList ?
      '/* Hide Users Not in List */\n' +
      'li:not(:has(img[src*="' + users.value.map(u => u.id).filter(id => id).join('"]), li:not(:has(img[src*="') + '"])) { display: none; }' : ''}
img[class*="avatar"] {
  opacity: 1 !important;
  border-radius: 0% !important;
  border: none;
  margin: 0;
  padding: 0;
  height: 200px;
  width: 200px;
  filter: brightness(${settings.dimIdle ? '50%' : '100%'});
}

img[class*="Speaking"] {
  filter: brightness(100%);
  position: relative;
  border-color: rgba(0,0,0,0) !important;
  animation-name: speak-now;
  animation-duration: 1s;
  animation-fill-mode: forwards;
}

${settings.myId !== '' ? `/* Hide Myself */
li[data-userid*="${settings.myId}"] { display: none; }` : ''}

${settings.hideNames ? '/* Hide Names */\nspan[class*="name"]{ display: none; }' : ''}
`;

  // 2. Loop through users to generate specific overrides
  users.value.forEach((user, index) => {
    // Skip empty users to prevent broken CSS
    if (!user.id) return;

    css += `
/* --- User ${index + 1} (${user.id}) --- */

/* Reveal this specific user */

/* Idle State */
img[class*="avatar"][src*="${user.id}"] {
  content: url("${user.idleUrl}");
  // filter: brightness(${settings.dimIdle ? '50%' : '100%'});
}

/* Speaking State */
img[class*="Speaking"][src*="${user.id}"] {
  content: url("${user.speakingUrl !== null && user.speakingUrl !== '' ? user.speakingUrl : user.idleUrl}");
}
`;
  });

  // 3. Optional: Hide users NOT in the list
  // We construct a selector that hides the list item if it doesn't contain one of our IDs
  // Note: This is tricky in CSS without :has(), usually just opacity:0 on the image is enough.

  return css;
});
</script>

<style scoped>
body {
  min-height: 100vh;
  align-content: center;
}
/* Simple styling for the Vue App itself */
.container { max-width: 800px; margin: 20px auto; padding: 20px; font-family: sans-serif;
  background-color: var(--color-background-mute);
  border-radius: 2rem;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  box-shadow: 0 4px 30px rgba(67, 67, 67, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.3);
  display: flex;
  flex-direction: column;
  gap: 20px;
}
.card { background: var(--color-background-soft); padding: 15px; margin-bottom: 15px; border-radius: 8px; }
.row { display: flex; gap: 15px; margin-bottom: 10px; flex-wrap: wrap; align-items: center }
.input-group { display: flex; flex-direction: column; flex: 1; }
input, select { padding: 8px; margin-top: 5px; border: 1px solid #ccc; border-radius: 4px; }
textarea { width: 100%; font-family: monospace; border: 1px solid #333; background: #222; color: #0f0; padding: 10px; }
.btn-add { background: #4caf50; color: white; border: none; padding: 10px 20px; cursor: pointer; border-radius: 4px; }
.btn-delete { background: #ff5252; color: white; border: none; padding: 5px 10px; cursor: pointer; border-radius: 4px; }
.btn-copy { background: #2196f3; color: white; border: none; padding: 10px; margin-top: 10px; cursor: pointer; width: 100%; }
.card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
</style>