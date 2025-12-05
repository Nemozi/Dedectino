<script setup>
import { ref, reactive, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { supabase } from '@/lib/supabaseClient.js';

// Komponenten
import spotTheFake from '@/components/game/spotTheFake.vue';
import analysis from '@/components/game/analysis.vue';
import multiCheck from '@/components/game/multiCheck.vue';
import conceptTagging from '@/components/game/conceptTagging.vue';

const router = useRouter();
const currentStep = ref(0);
const realImagePool = ref([]);
const alreadySeenImages = ref([]);

// Speichert, welche Erklärungen nötig sind
const needsRemediation = reactive({
    image71: false,
    image21: false
});

onMounted(async () => {
    const { data: { user } } = await supabase.auth.getUser();
    if (!user) return router.push('/login');

    const { data: storageData } = await supabase.storage.from('Real-Images').list();
    if (storageData) {
        realImagePool.value = storageData
            .filter(f => f.name.endsWith('.jpg') || f.name.endsWith('.png'))
            .map(f => f.name);
    }

    const { data: dbData } = await supabase
        .from('gesehene_bilder')
        .select('image_name')
        .eq('user_id', user.id);

    if (dbData) {
        alreadySeenImages.value = dbData.map(entry => entry.image_name);
    }
});

const saveSeenImagesToDB = async (images) => {
    alreadySeenImages.value.push(...images);
    const { data: { user } } = await supabase.auth.getUser();
    if (!user || images.length === 0) return;

    const inserts = images.map(img => ({
        user_id: user.id,
        image_name: img,
        bucket_name: 'Real-Images'
    }));
    await supabase.from('gesehene_bilder').insert(inserts);
};

const getUniqueRealImages = (count) => {
    const available = realImagePool.value.filter(img => !alreadySeenImages.value.includes(img));
    const shuffled = available.sort(() => 0.5 - Math.random());
    const selection = shuffled.slice(0, count);
    saveSeenImagesToDB(selection);
    return selection;
};

// --- NAVIGATION LOGIK ---

const nextStep = () => {
    currentStep.value++;
    window.scrollTo(0, 0);
};

// Spezial-Logik nach dem Multi-Check (Schritt 2)
const handleMultiCheckResult = (result) => {
    needsRemediation.image71 = !result.image71Correct;
    needsRemediation.image21 = !result.image21Correct;

    // Entscheidungsbaum: Wohin geht es als nächstes?
    if (needsRemediation.image71) {
        currentStep.value = 3; // Zeige Erklärung für Bild 71
    } else if (needsRemediation.image21) {
        currentStep.value = 4; // 71 war richtig, zeige Erklärung für Bild 21
    } else {
        currentStep.value = 5; // Alles richtig, weiter zum nächsten Spiel
    }
    window.scrollTo(0, 0);
};

// Spezial-Logik nach Erklärung 71 (Schritt 3)
const afterRemediation71 = () => {
    if (needsRemediation.image21) {
        currentStep.value = 4; // Zeige Erklärung für Bild 21
    } else {
        currentStep.value = 5; // Weiter zum nächsten Spiel
    }
    window.scrollTo(0, 0);
};

const finishLevel = async () => {
    const { data: { user } } = await supabase.auth.getUser();
    if (user) {
        await supabase.from('level_fortschritt').upsert({
            user_id: user.id,
            level_id: 2,
            score: 100
        }, { onConflict: 'user_id, level_id' });
    }
    router.push('/levels');
};
</script>

<template>
    <div class="content-wrapper">
        <div class="level-container">
            <div class="level-progress">
                Schritt {{ currentStep + 1 }} 
            </div>

            <!-- SCHRITT 0: Spot the Fake (Image 24) -->
            <spotTheFake 
                v-if="currentStep === 0"
                aiImage="Image_0024.jpg"
                :realImages="getUniqueRealImages(3)"
                @completed="nextStep"
            />

            <!-- SCHRITT 1: Analyse Image 24 -->
            <analysis 
                v-if="currentStep === 1"
                image="Image_0024.jpg"
                title="Analyse: Maßstab & Konsistenz"
                text="Hier sehen wir ein typisches Beispiel für inkonsistente Hintergründe. Der Mann wirkt echt, aber der Maßstab des Labyrinths passt nicht zu seiner Größe. Zudem zeigt der Bereich hinter dem Labyrinth plötzlich eine völlig andere Landschaftsskalierung."
                buttonText="Verstanden"
                @next="nextStep"
            />

            <!-- SCHRITT 2: Multi Check (71 & 21) -->
            <!-- Hier wird entschieden, ob Schritte 3 und 4 nötig sind -->
            <multiCheck 
                v-if="currentStep === 2"
                @completed="handleMultiCheckResult"
            />

            <!-- SCHRITT 3: Remediation Image 71 (Nur wenn nötig) -->
            <analysis 
                v-if="currentStep === 3"
                image="Image_0071.jpg"
                title="Fehleranalyse: Geteilter Hintergrund"
                text="Das war knapp! Achte hier auf den Hintergrund: Er wird durch das Objekt im Zentrum (die Person) quasi 'zwei-geteilt'. Links ist der Wald total verwischt, rechts sieht die Struktur ganz anders aus. Ein klassischer KI-Fehler."
                buttonText="Weiter"
                @next="afterRemediation71"
            />

            <!-- SCHRITT 4: Remediation Image 21 (Nur wenn nötig) -->
            <analysis 
                v-if="currentStep === 4"
                image="Image_0021.jpg"
                title="Fehleranalyse: Texturen"
                text="Schau dir die Mauer genau an. Sie ist viel zu unscharf dafür, wie nah der Mann an ihr lehnt. Diese Unschärfe sieht nicht wie ein echter 'Bokeh-Effekt' (Tiefenschärfe) einer Kamera aus, sondern einfach matschig und verwischt."
                buttonText="Weiter"
                @next="nextStep"
            />

            <!-- SCHRITT 5: Spot the Fake (Image 73) -->
            <spotTheFake 
                v-if="currentStep === 5"
                aiImage="Image_0073.jpg"
                :realImages="getUniqueRealImages(3)"
                customSuccessText="Richtig! Der Mann trennt den Hintergrund: Links eine Hütte, rechts plötzlich Wald. Das passt nicht zusammen."
                @completed="nextStep"
            />

            <!-- SCHRITT 6: Concept Tagging -->
            <conceptTagging 
                v-if="currentStep === 6"
                @completed="nextStep"
            />

            <!-- SCHRITT 7: Zusammenfassung -->
            <analysis 
                v-if="currentStep === 7"
                :image="['Image_0001.jpg', 'Image_0012.jpg']"
                title="Level Abschluss"
                text="Du hast gelernt, auf den Hintergrund zu achten: Suche nach verwischten Texturen, inkonsistenten Objekten links/rechts und unlogischen Schatten. Diese Fehler verraten die KI oft, auch wenn das Gesicht perfekt aussieht."
                buttonText="Level Beenden"
                @next="finishLevel"
            />

        </div>
    </div>
</template>

<style scoped>
.level-container {
    width: 100%;
    max-width: 50rem; /* Desktop Begrenzung */
    margin: 0 auto;
    /* Mobil-Optimierung: */
    padding: 0 1rem 2rem 1rem; /* Platz für Schatten unten */
    box-sizing: border-box;
}

.level-progress {
    text-align: center;
    font-weight: 800;
    text-transform: uppercase;
    margin-bottom: 1.5rem;
    border-bottom: 3px solid #000;
    padding-bottom: 0.5rem;
    font-size: 1.2rem;
}

/* Falls globale Styles stören: */
:deep(.content-wrapper) {
    padding-top: 1rem;
    align-items: flex-start; /* Auf Mobile besser oben startend als mittig */
}
</style>