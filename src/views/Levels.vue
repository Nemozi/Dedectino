<script setup>
import { ref, onMounted } from 'vue';
import { supabase } from '@/lib/supabaseClient.js';
import { useRouter } from 'vue-router';

const router = useRouter();
const loading = ref(true);
const completedLevels = ref([]); // Array von Level-IDs, z.B. [1, 2]

// Konfiguration der Levels
const levels = [
    { id: 1, title: "Einstufungstest", route: "/test-user-level" }, 
    { id: 2, title: "Deepfakes Basics", route: "/level/2" },
    { id: 3, title: "Hände & Finger", route: "/level/3" },
    { id: 4, title: "Hintergründe", route: "/level/4" },
];

onMounted(async () => {
    const { data: { user } } = await supabase.auth.getUser();
    if (!user) {
        router.push('/login');
        return;
    }

    // Fortschritt laden
    const { data } = await supabase
        .from('level_fortschritt')
        .select('level_id');
    
    if (data) {
        completedLevels.value = data.map(entry => entry.level_id);
    }
    loading.value = false;
});

const openLevel = (levelId, route) => {
    // Level ist offen, wenn es Level 1 ist ODER das vorherige Level erledigt ist
    const isUnlocked = levelId === 1 || completedLevels.value.includes(levelId - 1);
    
    if (isUnlocked) {
        router.push(route);
    }
};

const getStatusClass = (id) => {
    if (completedLevels.value.includes(id)) return 'completed';
    // Wenn Vorgänger fertig oder Level 1 -> aktiv
    if (id === 1 || completedLevels.value.includes(id - 1)) return 'active';
    return 'locked';
};
</script>

<template>
    <div class="content-wrapper">
        <div class="roadmap-card">
            <h1>Deine Reise</h1>
            
            <div v-if="loading">Lade Map...</div>

            <div v-else class="timeline">
                <div 
                    v-for="level in levels" 
                    :key="level.id" 
                    class="level-node"
                    :class="getStatusClass(level.id)"
                    @click="openLevel(level.id, level.route)"
                >
                    <div class="circle">
                        <span v-if="completedLevels.includes(level.id)">✓</span>
                        <span v-else-if="getStatusClass(level.id) === 'locked'">🔒</span>
                        <span v-else>{{ level.id }}</span>
                    </div>
                    <span class="level-title">{{ level.title }}</span>
                </div>
                
                <!-- Verbindungslinie (optisch im CSS gelöst) -->
                <div class="line"></div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.roadmap-card {
    background: var(--card-bg, #edc531);
    border: 0.0625rem solid #1a1a1a;
    box-shadow: 0.375rem 0.375rem 0 rgba(0,0,0,1);
    padding: 2rem;
    width: 100%;
    max-width: 30rem;
    position: relative;
}

h1 { text-align: center; text-transform: uppercase; border-bottom: 2px solid #000; padding-bottom: 1rem; }

.timeline {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2rem;
    position: relative;
    margin-top: 2rem;
}

/* Die durchgehende Linie im Hintergrund */
.line {
    position: absolute;
    top: 20px;
    bottom: 20px;
    width: 4px;
    background: #000;
    z-index: 0;
}

.level-node {
    z-index: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    cursor: pointer;
    transition: transform 0.2s;
    background: var(--card-bg); /* Überdeckt die Linie */
    padding: 10px;
}

.level-node:hover:not(.locked) { transform: scale(1.1); }

.circle {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    border: 3px solid #000;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: 800;
    font-size: 1.5rem;
    background: #fff;
    box-shadow: 4px 4px 0 rgba(0,0,0,1);
}

.level-title {
    margin-top: 0.5rem;
    font-weight: 700;
    text-transform: uppercase;
    font-size: 0.9rem;
    background: #fff;
    padding: 2px 5px;
    border: 1px solid #000;
}

/* Status Styles */
.completed .circle {
    background: #d4d4d4; /* Grau wie gewünscht */
    color: #555;
    box-shadow: none;
}
.completed .level-title { opacity: 0.7; }

.active .circle {
    background: #ff4757; /* Signalfarbe für aktuelles Level */
    color: #fff;
    animation: pulse 2s infinite;
}

.locked { cursor: not-allowed; opacity: 0.6; }
.locked .circle { background: #555; color: #aaa; border-color: #333; }

@keyframes pulse {
    0% { box-shadow: 0 0 0 0 rgba(255, 71, 87, 0.4); }
    70% { box-shadow: 0 0 0 10px rgba(255, 71, 87, 0); }
    100% { box-shadow: 0 0 0 0 rgba(255, 71, 87, 0); }
}
</style>