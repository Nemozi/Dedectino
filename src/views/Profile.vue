<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { supabase } from '@/lib/supabaseClient.js';

const router = useRouter();
const loading = ref(true);
const profileData = ref(null);
const email = ref('');

onMounted(async () => {

    const { data: { user } } = await supabase.auth.getUser();
    
    if (!user) {
        router.push('/login');
        return;
    }
    email.value = user.email;

    const { data, error } = await supabase
        .from('spielerprofile')
        .select('*')
        .eq('user_id', user.id)
        .single();

    if (data) profileData.value = data;
    loading.value = false;
});

const handleLogout = async () => {
    await supabase.auth.signOut();
    router.push('/');
};
</script>

<template>
    <div class="content-wrapper">
        <div class="profile-card">
            <h1>Dein Profil</h1>
            
            <div v-if="loading">Lade Daten...</div>
            
            <div v-else class="profile-details">
                <div class="detail-row">
                    <span class="label">Email:</span>
                    <span class="value">{{ email }}</span>
                </div>
                
                <div v-if="profileData" class="stats-box">
                    <h3>Deine Werte</h3>
                    <p>Alter: {{ profileData.alter }}</p>
                    <p>Internet-Affinität: {{ profileData.internet_affinitaet }}/10</p>
                    <p>Erkennungs-Skill: {{ profileData.erkennung_skill }}%</p>
                </div>
            </div>

            <button @click="handleLogout" class="primary-btn logout">
                Abmelden
            </button>
        </div>
    </div>
</template>

<style scoped>
/* Dein Standard-Design anwenden... */
.profile-card {
    background: var(--card-bg, #edc531);
    border: 0.0625rem solid #000;
    box-shadow: 0.375rem 0.375rem 0 rgba(0,0,0,1);
    padding: 2.5rem;
    max-width: 35rem;
    width: 100%;
    margin: 2rem auto;
}

h1 { text-transform: uppercase; border-bottom: 2px solid #000; }
.detail-row { margin-bottom: 1rem; font-size: 1.1rem; }
.label { font-weight: bold; margin-right: 0.5rem; }
.stats-box { background: #fff; padding: 1rem; border: 1px solid #000; margin-top: 1rem; }

</style>