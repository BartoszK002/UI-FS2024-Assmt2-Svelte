<script>
  import JournalEntry from './lib/JournalEntry.svelte';
  import { onMount } from 'svelte';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  import { writable } from 'svelte/store';

  let entries = [];
  $: averageSleep = calculateAverageSleep(entries);
  $: moodCounts = calculateMoodCounts(entries);
  $: averageWater = calculateAverageWater(entries);
  $: waterIntakeStore = writable(0);
  $: waterIntakeTweened = tweened(0, {
    duration: 500,
    easing: cubicOut
  });

  $: {
    averageSleepStore.set(averageSleep);
    averageSleepTweened.set(averageSleep, { duration: 0 }); // Set duration to 0 for immediate update
    waterIntakeStore.set(averageWater);
    waterIntakeTweened.set(averageWater);
  }

  function calculateAverageSleep(entries) {
    const validEntries = entries.filter(entry => entry.sleep !== undefined && entry.sleep !== null);
    if (validEntries.length === 0) return 0;
    const totalSleep = validEntries.reduce((sum, entry) => sum + entry.sleep, 0);
    return Math.min(totalSleep / validEntries.length, 16);
  }

  function calculateMoodCounts(entries) {
    return entries.reduce((counts, entry) => {
      (entry.moods || []).forEach(mood => {
        counts[mood] = (counts[mood] || 0) + 1;
      });
      return counts;
    }, {});
  }

  function calculateAverageWater(entries) {
    const validEntries = entries.filter(entry => entry.waterIntake !== undefined && entry.waterIntake !== null);
    if (validEntries.length === 0) return 0;
    const totalWater = validEntries.reduce((sum, entry) => sum + entry.waterIntake, 0);
    return totalWater / validEntries.length;
  }

  function calculateBarHeight(count, maxCount) {
    const minHeight = 5; // Minimum height in percentage
    const maxHeight = 95; // Maximum height in percentage
    const sqrtMax = Math.sqrt(maxCount);
    const sqrtValue = Math.sqrt(count);
    return minHeight + (maxHeight - minHeight) * (sqrtValue / sqrtMax);
  }

  const dummyData = [
    { date: '2024-03-01', sleep: 7, moods: ['energetic', 'motivated'], journalText: 'Great day!', imageUrl: '', quote: '', waterIntake: 6 },
    { date: '2024-03-02', sleep: 6, moods: ['anxious', 'tired'], journalText: 'Tough day.', imageUrl: '', quote: '', waterIntake: 4 },
    { date: '2024-03-03', sleep: 8, moods: ['motivated', 'happy'], journalText: 'Productive day!', imageUrl: '', quote: '', waterIntake: 8 },
    { date: '2024-03-04', sleep: 7.5, moods: ['energetic', 'happy'], journalText: 'Feeling good!', imageUrl: '', quote: '', waterIntake: 7 },
    { date: '2024-03-05', sleep: 6.5, moods: ['tired', 'anxious'], journalText: 'Busy day ahead.', imageUrl: '', quote: '', waterIntake: 5 },
    { date: '2024-03-06', sleep: 7, moods: ['motivated', 'creative'], journalText: 'Focused and determined.', imageUrl: '', quote: '', waterIntake: 6 },
    { date: '2024-03-07', sleep: 8, moods: ['happy', 'energetic'], journalText: 'Wonderful day!', imageUrl: '', quote: '', waterIntake: 7 },
    { date: '2024-03-08', sleep: 5, moods: ['tired', 'frustrated'], journalText: 'Stressful day.', imageUrl: '', quote: '', waterIntake: 3 },
    { date: '2024-03-09', sleep: 7.5, moods: ['motivated', 'creative'], journalText: 'Accomplished a lot today.', imageUrl: '', quote: '', waterIntake: 6 },
    { date: '2024-03-10', sleep: 9, moods: ['happy', 'calm'], journalText: 'Relaxing weekend.', imageUrl: '', quote: '', waterIntake: 5 },
    { date: '2024-03-11', sleep: 6, moods: ['anxious', 'frustrated'], journalText: 'Challenging day at work.', imageUrl: '', quote: '', waterIntake: 4 },
    { date: '2024-03-12', sleep: 7, moods: ['calm', 'creative'], journalText: 'Peaceful and productive.', imageUrl: '', quote: '', waterIntake: 6 },
    { date: '2024-03-13', sleep: 8, moods: ['energetic', 'motivated'], journalText: 'Feeling unstoppable!', imageUrl: '', quote: '', waterIntake: 7 },
    { date: '2024-03-14', sleep: 6.5, moods: ['tired', 'bored'], journalText: 'Uneventful day.', imageUrl: '', quote: '', waterIntake: 5 }
  ];

  onMount(() => {
    const today = new Date().toISOString().split('T')[0];
    const existingEntries = dummyData.filter(entry => entry.date !== today);
    entries = [...existingEntries, createTodayEntry()];
  });

  function createTodayEntry() {
    return {
      date: new Date().toISOString().split('T')[0],
      sleep: 0,
      moods: [],
      journalText: '',
      imageUrl: '',
      quote: '',
      waterIntake: 0
    };
  }

  function handleEntryUpdate(event) {
    const updatedEntry = event.detail;
    const index = entries.findIndex(e => e.date === updatedEntry.date);
    if (index !== -1) {
      entries[index] = updatedEntry;
      entries = [...entries]; // Create a new array to trigger reactivity
    }
  }

  const averageSleepStore = writable(0);
  const averageSleepTweened = tweened(0, {
    duration: 500, // Reduced from 1000 to 500 milliseconds
    easing: cubicOut
  });

  function getColorForMood(mood) {
    const colors = {
      energetic: '#ff9f43',
      anxious: '#ee5253',
      motivated: '#54a0ff',
      tired: '#5f27cd',
      happy: '#10ac84',
      calm: '#26de81',
      frustrated: '#fc5c65',
      creative: '#45aaf2',
      bored: '#778ca3'
    };
    return colors[mood] || '#636e72';
  }

  function getShortMoodName(mood) {
    const shortNames = {
      energetic: 'Energy',
      anxious: 'Anxiety',
      motivated: 'Motiv.',
      tired: 'Tired',
      happy: 'Happy',
      calm: 'Calm',
      frustrated: 'Frustr.',
      creative: 'Create',
      bored: 'Bored'
    };
    return shortNames[mood] || mood;
  }

  $: sleepColor = `hsl(${Math.min($averageSleepTweened * 15, 120)}, 100%, 45%)`;
  // Remove the sleepTextColor variable

  const GLASS_TO_ML = 250;
  const MAX_WATER_ML = 3000;

  $: averageWaterMl = averageWater * GLASS_TO_ML;
  $: waterFillPercentage = (averageWaterMl / MAX_WATER_ML) * 100;
</script>

<main class="app-container">
  <div class="content-wrapper">
    <h1>Daily Journal</h1>
    <div class="journal-stats-container">
      <div class="journal-container">
        {#if entries.length > 0}
          <JournalEntry entry={entries[entries.length - 1]} on:update={handleEntryUpdate} />
        {:else}
          <p>Loading...</p>
        {/if}
      </div>
      <div class="stats-container">
        <h2>Statistics</h2>
        <div class="stat-box">
          <span class="stat-label">Average Sleep</span>
          <div class="sleep-visualization">
            <svg width="100" height="100" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="45" fill="none" stroke="#ecf0f1" stroke-width="10" />
              <circle cx="50" cy="50" r="45" fill="none" stroke={sleepColor} stroke-width="10"
                      stroke-dasharray="282.7"
                      stroke-dashoffset={282.7 - (282.7 * $averageSleepTweened / 16)}
                      transform="rotate(-90 50 50)" />
              <text x="50" y="50" text-anchor="middle" dominant-baseline="central" font-size="20" fill="#2c3e50">
                {$averageSleepStore.toFixed(1)}
              </text>
            </svg>
            <span class="stat-value">hours</span>
          </div>
        </div>
        <div class="stat-box water-stat-box">
          <span class="stat-label">Average Water Intake</span>
          <div class="water-visualization">
            <div class="water-container">
              <div class="water-fill" style="height: {waterFillPercentage}%"></div>
            </div>
            <div class="water-stats">
              <span class="water-value">{$waterIntakeStore.toFixed(1)} glasses</span>
              <span class="water-value-ml">({averageWaterMl.toFixed(0)} ml)</span>
            </div>
          </div>
        </div>
        <h3>Mood Trends</h3>
        <div class="mood-visualization">
          {#each Object.entries(moodCounts) as [mood, count]}
            {@const color = getColorForMood(mood)}
            {@const maxCount = Math.max(...Object.values(moodCounts))}
            {@const heightPercentage = calculateBarHeight(count, maxCount)}
            <div class="mood-bar">
              <div class="mood-bar-wrapper">
                <div class="mood-bar-fill" style="--mood-color: {color}; height: {heightPercentage}%;">
                  <span class="mood-count">{count}</span>
                </div>
              </div>
              <span class="mood-label">{getShortMoodName(mood)}</span>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f5f7fa;
    color: #333;
  }

  .app-container {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding: 2rem;
  }

  .content-wrapper {
    max-width: 1200px;
    width: 100%;
    margin: 0 auto; /* Center the content */
  }

  h1 {
    color: #2c3e50;
    text-align: center;
    margin-bottom: 2rem;
    font-size: 2.5rem;
  }

  .journal-stats-container {
    display: flex;
    justify-content: space-between;
    gap: 2rem;
  }

  .journal-container {
    flex: 1;
    max-width: 60%;
  }

  .stats-container {
    flex: 0 0 35%;
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 1.5rem;
    align-self: flex-start;
    position: sticky;
    top: 2rem;
  }

  h2, h3 {
    color: #2c3e50;
    margin-bottom: 1rem;
  }

  .stat-box {
    background-color: #ecf0f1;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1rem;
  }

  .stat-label {
    font-weight: bold;
    color: #34495e;
  }

  .stat-value {
    font-size: 1.2em;
    color: #27ae60;
    float: right;
  }

  .mood-list {
    list-style-type: none;
    padding: 0;
  }

  .mood-list li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 0;
    border-bottom: 1px solid #ecf0f1;
  }

  .mood-label {
    color: #34495e;
  }

  .mood-count {
    background-color: #3498db;
    color: white;
    border-radius: 12px;
    padding: 0.2rem 0.5rem;
    font-size: 0.8em;
  }

  .sleep-visualization {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 1rem;
  }

  .sleep-visualization svg {
    width: 100px;
    height: 100px;
  }

  .sleep-visualization circle {
    transition: stroke-dashoffset 0.5s ease-in-out, stroke 0.5s ease-in-out;
  }

  .sleep-visualization text {
    transition: fill 0.5s ease-in-out;
  }

  .sleep-visualization .stat-value {
    margin-left: 10px; /* Add some space between the SVG and "hours" text */
  }

  .mood-visualization {
    display: flex;
    justify-content: space-around;
    height: 200px;
    margin-top: 1rem;
  }

  .mood-bar {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 60px; /* Increased width for better spacing */
    height: 100%;
  }

  .mood-bar-wrapper {
    flex-grow: 1;
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
  }

  .mood-bar-fill {
    width: 100%;
    background-color: var(--mood-color);
    border-radius: 4px 4px 0 0;
    transition: height 0.3s ease-in-out;
    display: flex;
    justify-content: center;
    align-items: flex-end;
    padding-bottom: 5px;
  }

  .mood-label {
    margin-top: 10px;
    text-align: center;
    font-size: 0.8em;
    color: #34495e;
    white-space: nowrap;
  }

  .mood-count {
    font-size: 0.9em;
    font-weight: bold;
    color: white; /* Changed from black to white */
    text-shadow: 0 0 2px black; /* Changed from white to black for better visibility on lighter backgrounds */
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  .stat-box, .mood-visualization {
    animation: fadeIn 1s ease-out;
  }

  @media (max-width: 1200px) {
    .journal-stats-container {
      flex-direction: column;
    }

    .journal-container,
    .stats-container {
      max-width: 100%;
      width: 100%;
    }

    .stats-container {
      position: static;
    }
  }

  .water-stat-box {
    background-color: #e3f2fd;
    border: 1px solid #bbdefb;
  }

  .water-visualization {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 1rem;
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
    transition: height 0.5s ease-in-out;
  }

  .water-stats {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
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
</style>