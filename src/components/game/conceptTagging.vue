<script setup>
import { ref, computed } from 'vue';
import { supabase } from '@/lib/supabaseClient.js';

const emit = defineEmits(['completed']);

const images = ['Image_0014.jpg', 'Image_0055.jpg', 'Image_0035.jpg'];
const terms = [
    { id: 1, text: 'Verwischte Texturen', selected: false },
    { id: 2, text: 'Inkonsistente Merkmale', selected: false },
    { id: 3, text: 'Unrealistische Darstellung', selected: false },
    { id: 4, text: 'Komisches Licht/Schatten', selected: false }
];

const termState = ref(terms);
const resolved = ref(false);

const imageUrls = computed(() => {
    return images.map(img => supabase.storage.from('Fake-Images').getPublicUrl(img).data.publicUrl);
});

const toggleTerm = (index) => {
    if (resolved.value) return;
    termState.value[index].selected = !termState.value[index].selected;
};

const resolve = () => {
    resolved.value = true;
    // Setze alle auf true für den visuellen Effekt "Alles ist richtig"
    termState.value.forEach(t => t.selected = true);
};
</script>

<template>
    <div class="game-card">
        <h2>Was fällt dir am Hintergrund auf?</h2>
        <p class="subtitle">Wähle alle passenden Begriffe aus.</p>

        <div class="gallery">
            <img v-for="url in imageUrls" :key="url" :src="url" />
        </div>

        <div class="terms-grid">
            <div 
                v-for="(term, idx) in termState" 
                :key="term.id"
                class="term-btn"
                :class="{ 'selected': term.selected, 'correct': resolved }"
                @click="toggleTerm(idx)"
            >
                {{ term.text }}
                <span v-if="resolved">✅</span>
            </div>
        </div>

        <button v-if="!resolved" class="primary-btn" @click="resolve">Überprüfen</button>
        
        <div v-if="resolved" class="feedback">
            <p><strong>Die Lösung ist: Alles trifft zu!</strong></p>
            <p>Diese Fehlerarten überschneiden sich oft. Achte im Hintergrund immer besonders auf verwischte Texturen und inkonsistente Lichtverhältnisse.</p>
            <button class="primary-btn" @click="$emit('completed')">Weiter</button>
        </div>
    </div>
</template>

<style scoped>
.game-card {
    background: var(--card-bg, #edc531);
    border: 2px solid #000;
    padding: 1.5rem;
    box-shadow: 0.375rem 0.375rem 0 #000;
    width: 100%;
    box-sizing: border-box;
}

h2 {
    font-size: clamp(1.2rem, 4vw, 1.5rem);
    margin-top: 0;
    line-height: 1.2;
}

.subtitle {
    text-align: center;
    margin-bottom: 1rem;
    font-style: italic;
    font-size: 0.9rem;
}

/* Mobile-First Gallery: Horizontales Scrollen */
.gallery {
    display: flex;
    gap: 0.75rem;
    margin-bottom: 1.5rem;
    overflow-x: auto; /* Scrollbar erlauben */
    padding-bottom: 0.5rem; /* Platz für Scrollbar */
    scroll-snap-type: x mandatory; /* Einrasten */
    -webkit-overflow-scrolling: touch;
}

.gallery img {
    flex: 0 0 80%; /* Bild nimmt 80% der Breite ein */
    width: 80%;
    height: 180px;
    object-fit: cover;
    border: 2px solid #000;
    scroll-snap-align: center; /* Zentriert beim Scrollen */
}

/* Ab Tablet wieder nebeneinander */
@media (min-width: 600px) {
    .gallery img {
        flex: 1;
        width: auto;
    }
}

.terms-grid {
    display: grid;
    grid-template-columns: 1fr; /* Mobil: 1 Spalte */
    gap: 0.75rem;
}

@media (min-width: 600px) {
    .terms-grid {
        grid-template-columns: 1fr 1fr; /* Tablet/Desktop: 2 Spalten */
    }
}

.term-btn {
    background: #fff;
    border: 2px solid #000;
    padding: 1rem;
    font-weight: bold;
    cursor: pointer;
    text-align: center;
    display: flex;
    justify-content: space-between;
    align-items: center;
    min-height: 3.5rem; /* Leicht treffbar */
    font-size: 0.95rem;
    user-select: none;
}

.term-btn.selected {
    background: #000;
    color: #fff;
    transform: scale(1.02);
}

.term-btn.correct {
    border-color: #00aa00;
    background: #dfffd6;
    color: #005500;
    pointer-events: none;
}

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

.feedback {
    margin-top: 1rem;
    padding: 1rem;
    background: #fff;
    border: 2px solid #000;
    text-align: center;
}
</style>