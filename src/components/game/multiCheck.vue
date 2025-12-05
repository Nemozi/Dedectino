<script setup>
import { ref, onMounted } from 'vue';
import { supabase } from '@/lib/supabaseClient.js';

const emit = defineEmits(['completed']);

const img71Url = ref('');
const img21Url = ref('');
const selectedOption = ref(null);
const resolved = ref(false);

const options = [
    { id: 'none', label: 'Keins ist KI' },
    { id: 'both', label: 'Beide sind KI' }, // Korrekt
    { id: 'left', label: 'Nur Links (71)' },
    { id: 'right', label: 'Nur Rechts (21)' }
];

onMounted(() => {
    img71Url.value = supabase.storage.from('Fake-Images').getPublicUrl('Image_0071.jpg').data.publicUrl;
    img21Url.value = supabase.storage.from('Fake-Images').getPublicUrl('Image_0021.jpg').data.publicUrl;
});

const checkAnswer = () => {
    resolved.value = true;
};

const finish = () => {
    // Logic: User lag richtig, wenn er "Beide" gewählt hat.
    // Aber wir wollen wissen, welches Bild er "verpasst" hat, falls er nur eins gewählt hat.
    
    let result = { image71Correct: false, image21Correct: false };

    if (selectedOption.value === 'both') {
        result.image71Correct = true;
        result.image21Correct = true;
    } else if (selectedOption.value === 'left') {
        result.image71Correct = true; // 71 erkannt
        result.image21Correct = false; // 21 verpasst
    } else if (selectedOption.value === 'right') {
        result.image71Correct = false; // 71 verpasst
        result.image21Correct = true; // 21 erkannt
    }
    // 'none' -> beide falsch

    emit('completed', result);
};
</script>

<template>
    <div class="game-card">
        <h2>Sind diese Bilder KI-generiert?</h2>
        <div class="compare-grid">
            <div class="img-wrap"><img :src="img71Url" /><span>Nr. 71</span></div>
            <div class="img-wrap"><img :src="img21Url" /><span>Nr. 21</span></div>
        </div>

        <div class="options-list" :class="{ 'disabled': resolved }">
            <label v-for="opt in options" :key="opt.id" class="option-btn" :class="{ 'selected': selectedOption === opt.id }">
                <input type="radio" :value="opt.id" v-model="selectedOption">
                {{ opt.label }}
            </label>
        </div>

        <button v-if="!resolved" class="primary-btn" @click="checkAnswer" :disabled="!selectedOption">Auflösen</button>
        
        <div v-if="resolved" class="feedback">
            <p v-if="selectedOption === 'both'" class="success">Perfekt! Beide sind KI-generiert.</p>
            <p v-else class="fail">Nicht ganz richtig. Tatsächlich sind <strong>beide</strong> Bilder KI-generiert.</p>
            <button class="primary-btn" @click="finish">Erklärung ansehen</button>
        </div>
    </div>
</template>

<style scoped>
.game-card {
    background: var(--card-bg, #edc531);
    border: 2px solid #000;
    padding: 1.25rem; /* Weniger Padding auf Mobile */
    box-shadow: 0.375rem 0.375rem 0 #000;
    width: 100%;
    box-sizing: border-box;
}

h2 {
    font-size: clamp(1.1rem, 4vw, 1.5rem);
    margin-top: 0;
}

.compare-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.5rem; /* Enger */
    margin-bottom: 1.5rem;
}

.img-wrap {
    border: 2px solid #000;
    background: #000;
    position: relative;
    aspect-ratio: 1; /* Quadratisch erzwingen */
}

.img-wrap img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
}

.img-wrap span {
    position: absolute;
    bottom: 0;
    left: 0;
    background: #000;
    color: #fff;
    padding: 2px 6px;
    font-size: 0.75rem;
    font-weight: bold;
}

.options-list {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
}

.option-btn {
    background: #fff;
    border: 2px solid #000;
    padding: 1rem;
    cursor: pointer;
    font-weight: bold;
    font-size: 1rem;
    transition: all 0.1s;
    /* Touch Target Optimierung */
    min-height: 3rem; 
    display: flex;
    align-items: center;
}

.option-btn:hover { background: #eee; }

.option-btn.selected {
    background: #000;
    color: #fff;
    box-shadow: 4px 4px 0 rgba(0,0,0,0.2);
    transform: translate(-2px, -2px);
}

.option-btn input { display: none; }

.primary-btn {
    width: 100%;
    margin-top: 1.5rem;
    padding: 1.125rem;
    background: #000;
    color: #fff;
    border: none;
    font-weight: bold;
    text-transform: uppercase;
    cursor: pointer;
    font-size: 1rem;
}

.primary-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

.feedback {
    background: #fff;
    border: 2px solid #000;
    padding: 1rem;
    margin-top: 1rem;
    text-align: center;
}

.success { color: #00aa00; font-weight: bold; }
.fail { color: #aa0000; font-weight: bold; }
</style>