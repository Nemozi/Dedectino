<script setup>
import { computed } from 'vue';
import { supabase } from '@/lib/supabaseClient.js';

const props = defineProps(['image', 'title', 'text', 'buttonText']);
const emit = defineEmits(['next']);

// Hilfsfunktion für URLs (kann auch Array sein)
const imageUrls = computed(() => {
    const list = Array.isArray(props.image) ? props.image : [props.image];
    return list.map(img => {
        const { data } = supabase.storage.from('Fake-Images').getPublicUrl(img);
        return data.publicUrl;
    });
});
</script>

<template>
    <div class="analysis-card">
        <h2>{{ title }}</h2>
        <div class="images-container">
            <img v-for="url in imageUrls" :key="url" :src="url" alt="Analyse" />
        </div>
        <div class="text-content">
            <p>{{ text }}</p>
        </div>
        <button class="primary-btn" @click="$emit('next')">{{ buttonText || 'Weiter' }}</button>
    </div>
</template>

<style scoped>
.analysis-card {
    background: #fff;
    border: 2px solid #000;
    padding: 1.5rem;
    box-shadow: 0.375rem 0.375rem 0 #000;
    width: 100%;
    box-sizing: border-box;
}

h2 {
    margin-top: 0;
    font-size: clamp(1.2rem, 5vw, 1.6rem);
    text-transform: uppercase;
}

.images-container {
    display: flex;
    gap: 1rem;
    margin: 1.5rem 0;
    justify-content: center;
    flex-wrap: wrap; /* WICHTIG: Bilder brechen um */
}

img {
    max-width: 100%;
    height: auto;
    max-height: 300px;
    border: 2px solid #000;
    object-fit: contain;
    /* Auf Mobile volle Breite */
    flex: 1 1 200px; 
}

.text-content p {
    font-size: 1rem;
    line-height: 1.6;
    font-weight: 500;
    color: #333;
}

.primary-btn {
    width: 100%;
    margin-top: 1.5rem;
    padding: 1.125rem; /* Größere Touch-Area */
    background: #000;
    color: #fff;
    border: none;
    font-weight: 700;
    cursor: pointer;
    text-transform: uppercase;
    font-size: 1rem;
    letter-spacing: 0.05em;
}

.primary-btn:active {
    background: #333;
    transform: translateY(2px);
}
</style>