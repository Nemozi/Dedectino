<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { supabase } from '@/lib/supabaseClient.js'; 

const router = useRouter();
const isMenuOpen = ref(false);
const user = ref(null); 

const toggleMenu = () => {
    isMenuOpen.value = !isMenuOpen.value;
};

// Logout Funktion
const handleLogout = async () => {
    await supabase.auth.signOut();
    isMenuOpen.value = false; 
    router.push('/'); 
};

onMounted(() => {
    
    supabase.auth.getSession().then(({ data: { session } }) => {
        user.value = session?.user || null;
    });

    supabase.auth.onAuthStateChange((_event, session) => {
        user.value = session?.user || null;
    });
});
</script>

<template>
  <nav class="navbar">
    <div class="navbar-section navbar-left">
      <router-link to="/" class="navbar-logo">
        <img src="/Dedectino.png" alt="Dedectino Logo" class="logo-image">
        <span class="logo-text">Dedectino</span>
      </router-link>
    </div>

    <button class="menu-toggle" @click="toggleMenu" aria-label="Toggle navigation">
        &#9776;
    </button>

    <div class="navbar-links" :class="{ 'is-open': isMenuOpen }">

        <div class="navbar-section navbar-right">
           
           <router-link v-if="!user" to="/login" class="navbar-button">
                Login
           </router-link>

           <template v-else>
               <router-link to="/profile" class="navbar-button">
                    Profil
               </router-link>
               
               <button @click="handleLogout" class="navbar-button">
                    Logout
               </button>
           </template>
        
            <router-link to="/info" class="navbar-button">
                Info
            </router-link>
        </div>
    </div>
  </nav>
</template>

<style lang="css" scoped>
.navbar {
    display: flex; 
    justify-content: space-between; 
    align-items: center;
    height: 60px;
    padding: 0 20px;
    background-color: var(--card-bg, #edc531); 
    border-bottom: black 2px solid;
    position: relative;
    z-index: 100;
}

.navbar-section {
    display: flex;
    align-items: center;
}

.navbar-links {
    display: flex;
    flex-grow: 1;
    justify-content: flex-end;
}

.navbar-left {
    flex-grow: 0;
    min-width: 200px;
}
.navbar-logo {
    text-decoration: none;
    font-size: 1.4em;
    font-weight: bold;
    display: flex;
    align-items: center;
    color: black;
}
.navbar-logo:hover {
   scale: 1.05;
}
.logo-image {
    max-height: 3rem;
    width: auto;
    margin-right: 0.5rem; 
}

.navbar-right {
    display: flex;
    justify-content: flex-end; 
    gap: 1rem; 
    min-width: 200px;
}

.navbar-button {
    background-color: black; 
    color: white;
    border: none;
    font-size: 1em;
    cursor: pointer;
    padding: 0.5em 1em;
    box-shadow: 1rem;
    text-decoration: none; 
    display: inline-block;
}

.navbar-button:hover {
    background-color: white;
    transform: translate(0.125rem, 0.125rem);
    box-shadow: 0.25rem 0.25rem 0 rgba(0,0,0,.2);
    color: black;
}

.menu-toggle {
    display: none;
    background: none;
    border: none;
    font-size: 2em;
    cursor: pointer;
    color: black;
}

@media (max-width: 768px) {
    .menu-toggle {
        display: block; 
    }
    
    .navbar {
        justify-content: space-between;
        height: 50px;
    }

    .navbar-links {
        display: none;
        flex-direction: column;
        width: 100%;
        position: absolute;
        top: 50px;
        left: 0;
        background-color: black; 
        box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
    }

    .navbar-links.is-open {
        display: flex;
    }

    .navbar-section {
        width: 100%;
        padding: 10px 0;
        border-top: 1px solid rgba(0, 0, 0, 0.1);
        flex-direction: column;
        gap: 10px;
    }
    
    .navbar-right {
        flex-direction: column; 
        gap: 0;
        width: 100%;
    }

    .navbar-button {
        width: 90%; 
        margin: 5px auto;
        text-align: center;
    }
}
</style>