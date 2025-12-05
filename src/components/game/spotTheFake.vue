<script setup>
import { ref, onMounted } from 'vue';
import { supabase } from '@/lib/supabaseClient.js';

const props = defineProps(['aiImage', 'realImages', 'customSuccessText']);
const emit = defineEmits(['completed']);

const images = ref([]);
const loading = ref(true);
const resolved = ref(false);
const message = ref('');

onMounted(() => {
    // Bilder vorbereiten
    const aiObj = { src: props.aiImage, type: 'ai', bucket: 'Fake-Images' };
    const realObjs = props.realImages.map(img => ({ src: img, type: 'real', bucket: 'Real-Images' }));
    
    // Mischen
    const combined = [aiObj, ...realObjs].sort(() => 0.5 - Math.random());
    
    // URLs laden
    images.value = combined.map(img => {
        const { data } = supabase.storage.from(img.bucket).getPublicUrl(img.src);
        return { ...img, url: data.publicUrl };
    });
    loading.value = false;
});

const selectImage = (img) => {
    if (resolved.value) return;
    
    resolved.value = true;
    if (img.type === 'ai') {
        message.value = props.customSuccessText || "Korrekt! Das ist das KI-Bild.";
    } else {
        message.value = "Das war ein echtes Foto. Schau dir das KI-Bild (rot markiert) nochmal an.";
    }
};
</script>

<template>
    <div class="game-card">
        <h2>Welches Bild ist KI?</h2>
        <div class="grid">
            <div 
                v-for="img in images" 
                :key="img.src"
                class="img-box"
                :class="{ 
                    'correct': resolved && img.type === 'ai',
                    'faded': resolved && img.type === 'real'
                }"
                @click="selectImage(img)"
            >
                <img :src="img.url" />
            </div>
        </div>
        <div v-if="resolved" class="feedback-box">
            <p>{{ message }}</p>
            <button class="primary-btn" @click="$emit('completed')">Weiter</button>
        </div>
    </div>
</template>

<style scoped>
.game-card {
    background: var(--card-bg, #edc531);
    border: 2px solid #000; /* Dickerer Rahmen für Mobile-Look */
    padding: 1.5rem;
    box-shadow: 0.375rem 0.375rem 0 #000; /* ca 6px */
    width: 100%;
    box-sizing: border-box;
}

h2 {
    margin-top: 0;
    font-size: clamp(1.2rem, 5vw, 1.5rem); /* Skaliert mit */
    line-height: 1.2;
}

.grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem; /* Kleinerer Abstand auf Mobile */
    margin-top: 1rem;
}

.img-box {
    border: 2px solid #000;
    aspect-ratio: 1;
    cursor: pointer;
    background: #000;
    position: relative;
    /* Touch-Feedback entfernen bei Mobile */
    -webkit-tap-highlight-color: transparent; 
}

/* Desktop Hover Effekt */
@media (hover: hover) {
    .img-box:hover:not(.faded) { transform: scale(1.02); }
}

.img-box img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block; /* Entfernt Lücken unten */
}

.correct {
    border: 4px solid #fff;
    z-index: 2;
    transform: scale(1.02); /* Leichter Zoom */
    box-shadow: 0 4px 10px rgba(0,0,0,0.3);
}

.correct::after {
    content: "KI";
    position: absolute;
    top: 0;
    right: 0;
    background: #000;
    color: #fff;
    padding: 4px 8px;
    font-weight: bold;
    font-size: 0.8rem;
}

.faded { opacity: 0.3; pointer-events: none; }

.feedback-box {
    background: #fff;
    border: 2px solid #000;
    padding: 1rem;
    margin-top: 1.5rem;
    text-align: center;
    font-weight: bold;
}

/* Globale Button Klasse oder lokal */
.primary-btn {
    width: 100%;
    margin-top: 0.5rem;
    padding: 1rem;
    background: #000;
    color: #fff;
    border: none;
    font-weight: bold;
    cursor: pointer;
    text-transform: uppercase;
    font-size: 1rem;
}
</style>