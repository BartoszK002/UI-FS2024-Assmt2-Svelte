<script>
  import { createEventDispatcher, onMount } from 'svelte';

  export let entry = {}; // Provide a default empty object
  const dispatch = createEventDispatcher();

  let userName = "Bartosz Kawalkowski";
  let imageUrl = "";
  let quote = "";
  let currentTime = "";

  const moods = [
    { name: 'energetic', emoji: '⚡', color: '#ff9f43' },
    { name: 'anxious', emoji: '😰', color: '#ee5253' },
    { name: 'motivated', emoji: '💪', color: '#54a0ff' },
    { name: 'tired', emoji: '😴', color: '#5f27cd' },
    { name: 'happy', emoji: '😊', color: '#10ac84' },
    { name: 'calm', emoji: '😌', color: '#26de81' },
    { name: 'frustrated', emoji: '😤', color: '#fc5c65' },
    { name: 'creative', emoji: '🎨', color: '#45aaf2' },
    { name: 'bored', emoji: '🥱', color: '#778ca3' }
  ];

  const GLASS_TO_ML = 250; // Assuming 1 glass is 250ml
  const MAX_WATER_ML = 3000;
  const MAX_GLASSES = MAX_WATER_ML / GLASS_TO_ML;

  $: journalText = entry.journalText || '';
  $: sleep = entry.sleep ?? 0;
  $: moodsArray = entry.moods || [];
  $: waterIntake = entry.waterIntake || 0;
  $: waterIntakeMl = waterIntake * GLASS_TO_ML;

  // Function to fetch a random image
  function fetchRandomImage() {
    const width = 800;
    const height = 600;
    imageUrl = `https://picsum.photos/${width}/${height}?random=${Date.now()}`;
    updateEntry();
  }

  async function fetchRandomQuote() {
    try {
      const response = await fetch('https://api.quotable.io/random');
      const data = await response.json();
      quote = `"${data.content}" - ${data.author}`;
    } catch (error) {
      console.error('Error fetching quote:', error);
      quote = "The only way to do great work is to love what you do. - Steve Jobs";
    }
    updateEntry();
  }

  function updateTime() {
    const now = new Date();
    currentTime = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  }

  onMount(() => {
    fetchRandomImage();
    fetchRandomQuote();
    updateTime();
    const timer = setInterval(updateTime, 60000); // Update time every minute

    return () => clearInterval(timer); // Cleanup on component destroy
  });

  function updateEntry() {
    if (entry && typeof entry === 'object') {
      const updatedEntry = {
        ...entry,
        journalText,
        sleep,
        moods: [...moodsArray],
        imageUrl,
        quote,
        waterIntake
      };
      dispatch('update', updatedEntry);
    }
  }

  function handleSleepChange(event) {
    sleep = Math.max(0, Math.min(16, parseFloat(event.target.value)));
    updateEntry();
  }

  function toggleMood(mood) {
    if (moodsArray.includes(mood)) {
      moodsArray = moodsArray.filter(m => m !== mood);
    } else {
      moodsArray = [...moodsArray, mood];
    }
    updateEntry();
  }

  function handleWaterChange(event) {
    const newValue = Math.max(0, Math.min(MAX_GLASSES, parseFloat(event.target.value)));
    waterIntake = Math.round(newValue * 2) / 2; // Round to nearest 0.5
    updateEntry();
  }

  $: sleepColor = `hsl(${Math.min(sleep * 15, 120)}, 100%, 45%)`;
  $: textColor = sleep > 8 ? 'black' : 'white';
  $: waterFillPercentage = (waterIntakeMl / MAX_WATER_ML) * 100;
</script>

<div class="journal-entry">
  <header>
    <div class="header-content">
      <h2>{userName}'s Journal</h2>
      <span class="date">{entry.date}</span>
    </div>
    <span class="current-time">{currentTime}</span>
  </header>
  
  <div class="image-quote-container">
    <div class="image-container">
      <img src={imageUrl} alt="Daily inspiration" />
      <button on:click={fetchRandomImage}>New Image</button>
    </div>

    <blockquote>
      {quote}
      <button class="quote-button" on:click={fetchRandomQuote}>New Quote</button>
    </blockquote>
  </div>

  <textarea 
    bind:value={journalText}
    placeholder="Write your journal entry here..."
    on:input={updateEntry}
  ></textarea>

  <div class="journal-meta">
    <div class="sleep-input">
      <h3>Hours of Sleep:</h3>
      <div class="sleep-slider-container">
        <input 
          type="range" 
          id="sleep" 
          value={sleep}
          min="0" 
          max="16" 
          step="0.5"
          on:input={handleSleepChange}
        >
        <div class="sleep-value" style="--sleep-color: {sleepColor}; --text-color: {textColor}">
          <span class="sleep-emoji">{sleep <= 4 ? '😴' : sleep <= 8 ? '😊' : '😃'}</span>
          <span class="sleep-hours">{sleep.toFixed(1)}</span>
        </div>
      </div>
    </div>

    <div class="water-input">
      <h3>Daily Water Intake:</h3>
      <div class="water-tracker">
        <input 
          type="range" 
          id="water" 
          value={waterIntake}
          min="0" 
          max={MAX_GLASSES}
          step="0.5"
          on:input={handleWaterChange}
        >
        <div class="water-visualization">
          <div class="water-container">
            <div class="water-fill" style="height: {waterFillPercentage}%"></div>
          </div>
          <div class="water-stats">
            <span class="water-value">{waterIntake.toFixed(1)} glasses</span>
            <span class="water-value-ml">({waterIntakeMl.toFixed(0)} ml)</span>
          </div>
        </div>
      </div>
    </div>

    <div class="mood-section">
      <h3>Today's Moods:</h3>
      <div class="mood-selectors">
        {#each moods as mood}
          <button 
            class="mood-button" 
            class:selected={moodsArray.includes(mood.name)}
            on:click={() => toggleMood(mood.name)}
            style="--mood-color: {mood.color}"
          >
            <span class="mood-emoji">{mood.emoji}</span>
            <span class="mood-name">{mood.name}</span>
          </button>
        {/each}
      </div>
    </div>
  </div>
</div>

<style>
  .journal-entry {
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 1.5rem;
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 1rem;
  }

  .header-content {
    display: flex;
    flex-direction: column;
  }

  h2 {
    color: #2c3e50;
    margin: 0;
    font-size: 1.5rem;
  }

  .date {
    font-size: 0.9rem;
    color: #7f8c8d;
    margin-top: 0.25rem;
  }

  .current-time {
    font-size: 1rem;
    color: #7f8c8d;
  }

  .image-quote-container {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .image-container {
    flex: 1;
    text-align: center;
  }

  .image-container img {
    max-width: 75%; /* Increased from 60% to 75% */
    height: auto;
    border-radius: 8px;
    margin-bottom: 0.5rem;
    display: block;
    margin-left: auto;
    margin-right: auto;
  }

  blockquote {
    flex: 1;
    font-style: italic;
    color: #34495e;
    margin: 0;
    padding: 1rem;
    background-color: #ecf0f1;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }

  textarea {
    width: 100%;
    height: 150px;
    margin-bottom: 1rem;
    padding: 0.5rem;
    border: 1px solid #bdc3c7;
    border-radius: 8px;
    resize: vertical;
    font-family: inherit;
  }

  .journal-meta {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .sleep-input {
    margin-bottom: 0.5rem;
  }

  .sleep-input h3 {
    margin-bottom: 0.25rem;
    color: #2c3e50;
  }

  .sleep-slider-container {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  input[type="range"] {
    flex-grow: 1;
    -webkit-appearance: none;
    width: 100%;
    height: 10px;
    border-radius: 5px;
    background: #d7dcdf;
    outline: none;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #3498db;
    cursor: pointer;
    transition: background .15s ease-in-out;
  }

  input[type="range"]::-moz-range-thumb {
    width: 20px;
    height: 20px;
    border: 0;
    border-radius: 50%;
    background: #3498db;
    cursor: pointer;
    transition: background .15s ease-in-out;
  }

  .sleep-value {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 70px;
    height: 70px;
    border-radius: 50%;
    background-color: var(--sleep-color);
    color: var(--text-color);
    font-weight: bold;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .sleep-emoji {
    font-size: 1.5rem;
    margin-bottom: 1px; /* Reduced from 2px to 1px (50% reduction) */
  }

  .sleep-hours {
    font-size: 1.2rem;
  }

  .mood-section {
    margin-top: 0.5rem;
  }

  .mood-section h3 {
    margin-bottom: 0.25rem;
    color: #2c3e50;
  }

  .mood-selectors {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .mood-button {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 0.5rem;
    border: 2px solid var(--mood-color);
    background-color: transparent;
    color: var(--mood-color);
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .mood-button:hover {
    background-color: var(--mood-color);
    color: white;
  }

  .mood-button.selected {
    background-color: var(--mood-color);
    color: white;
  }

  .mood-emoji {
    font-size: 1.5rem;
    margin-bottom: 0.25rem;
  }

  .mood-name {
    font-size: 0.8rem;
    text-transform: capitalize;
  }

  .water-input {
    margin-bottom: 0.5rem;
  }

  .water-input h3 {
    margin-bottom: 0.25rem;
    color: #2c3e50;
  }

  .water-tracker {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .water-visualization {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .water-container {
    width: 40px;
    height: 100px;
    background-color: #ecf0f1;
    border-radius: 20px;
    overflow: hidden;
    position: relative;
  }

  .water-fill {
    position: absolute;
    bottom: 0;
    width: 100%;
    background-color: #3498db;
    transition: height 0.3s ease-in-out;
  }

  .water-stats {
    display: flex;
    flex-direction: column;
  }

  .water-value {
    font-size: 1.2em;
    color: #3498db;
    font-weight: bold;
  }

  .water-value-ml {
    font-size: 0.9em;
    color: #7f8c8d;
  }

  input[type="number"] {
    width: 60px;
    padding: 0.5rem;
    border: 1px solid #bdc3c7;
    border-radius: 4px;
    font-size: 1rem;
  }
</style>